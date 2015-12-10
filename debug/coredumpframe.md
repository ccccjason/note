# core dump 之前的 frame




每一個 C/C++ 的 programmer，應該都有遇過程式 core dump 或 segmentation fault 之後，透過 GDB (debugger) 卻無法指出錯位址的窘境。 (應該沒有人例外吧!) 這通常是因為 process 或 thread 的 stack 已經被弄亂了，或者 frame pointer 已經被改亂了，指向無名之處。 而 debugger 需要正確的 stack 資訊，才能分析出正確的 call frame。 遇到這種問題，通常的做法是乖乖去看程式碼，憑感覺推斷一些可能出錯的點， 並在出錯之前做 break point，再一步步的 step 過去，看是否正確。 又或者，一層又一層的二分法，縮小範圍，直到找到病源。 這樣的做法，對於小程式還可以，大程式就要花費非常多的功夫。 但又有什麼解法呢?

過去我也遇過無數次這類的情況，一直沒找到解法。 但我一直有一個想法，如果能在 stack 爛掉之前，把內容記錄下來，應該就能解決這個 問題。 這當然是癈話，如果知道什麼時侯爛掉，問題就不會這麼複雜。 於是，一直都沒有進一步的想法。 直到前幾天，在 irc 上看到另一個朋友在抱怨這個問題，突然間靈機一動。 其實可以這樣解。

gcc 有一個功能，叫做 instrument functions，能產生特別的 binary，在進入 和離開 function 時，各自呼叫特定的 function。 如果能在這兩個特定的 function，將 return address 記錄下來，那麼在程式爛掉時， 就可以透過該份記錄得知整個呼叫的過程。 如此就能很快的鎖定出錯的範圍。 知道範圍之後，就能使用 gdb 輕鬆的在出錯之前停下來，檢查整個狀態，找出問題 的根源。

為了避免印出一堆亂七八雜的訊息，影嚮程式的正常執行。 我使用一個 thread local variable，存放每個 thread 的 stack 資訊，主要是 return address。 平常時，我們不會看到任何差異。 但程式產生 core dump 之後，我們就能用 gdb 檢視 core file，並讀出 thread local variable 取得每一個 thread 的 call flow。 如此我們就可以很快縮小出錯的範圍。

以下程式叫 before_die.c，宣告了兩個特殊的程式，讓 gcc 在進入和離開每個 function 的前後進行呼叫。這兩個 function 會將 function 的 return address 記錄在 __before_die_frame_info 這個變數裡。

- before_die.c

```c
#include <stdio.h>
#include <stdlib.h>

struct bd_frame_info {
    int max;
    int top;
    void *stack[1];
};

#define BDFI_MAX_DEPTH 128
#define BDFI_SIZE (sizeof(void *) * (BDFI_MAX_DEPTH - 1) + \
            sizeof(struct bd_frame_info))
#define SHOW_CORRUPTED_INFO 1
#define SHOW_THREAD_INFO 1

__thread struct bd_frame_info *__before_die_frame_info = NULL;

static void
__before_die_corrupted() {
#ifdef SHOW_CORRUPTED_INFO
    int i;
    struct bd_frame_info *bdfi;

    fprintf(stderr, "Stack is corrupted:\n");
    bdfi = __before_die_frame_info;
    for(i = bdfi->top - 1; i >= 0; i--) {
        fprintf(stderr, "\t%p\n", bdfi->stack[i]);
    }
#endif /* SHOW_CORRUPTED_INFO */
}

static void
__before_die_new_thread(struct bd_frame_info *bdfi) {
#ifdef SHOW_THREAD_INFO
    fprintf(stderr, "Thread ID = 0x%x, bdfi @ 0x%p\n", pthread_self(), bdfi);
#endif /* SHOW_THREAD_INFO */
}

void
__cyg_profile_func_enter(void *this_fn, void *call_site) {
    struct bd_frame_info *bdfi;
    void *pparent;

    bdfi = __before_die_frame_info;
    if(bdfi == NULL) {
        /* First frame of this thread */
        bdfi = (struct bd_frame_info *)malloc(BDFI_SIZE);
        bdfi->max = BDFI_MAX_DEPTH;
        bdfi->top = 0;
        __before_die_frame_info = bdfi;
        __before_die_new_thread(bdfi);
    }

    if(bdfi->top >= bdfi->max)
        return;

    pparent = __builtin_return_address(2);
    if(bdfi->top > 0 && pparent != bdfi->stack[bdfi->top - 1]) {
        /* Corrupted stack? */
        __before_die_corrupted();
        return;
    }

    bdfi->stack[bdfi->top++] = call_site;
}

void
__cyg_profile_func_exit(void *this_fn, void *call_site) {
    struct bd_frame_info *bdfi;
    void *parent;

    bdfi = __before_die_frame_info;
    if(bdfi == NULL)
        return;

    parent = __builtin_return_address(1);
    if(parent != bdfi->stack[bdfi->top - 1]) {
        /* Corrupted stack? */
        __before_die_corrupted();
        return;
    }

    if(bdfi->top == 1) {
        /* Last frame of this thread */
        free(bdfi);
        __before_die_frame_info = NULL;
        return;
    }

    bdfi->top--;
}


```

請將 before_die.c 用下列指令編譯。

```c
gcc -c -g before_die.c
```

下面是一個測試程式(test_corruption.c)，在 depth3() 裡面會破壞 stack 的內容，造成程式無法正確的 執行。


- test_corruption.c


```c
#include <stdio.h>
#include <pthread.h>

void
depth3() {
    int *ptr;
    int i;

    ptr = (int *)&ptr;
    ptr++;
    for(i = 0; i < 1024; i++) {
        *ptr++ = 0xff;
    }
}

void
depth2() {
    depth3();
}

void *
depth1(void *dummy) {
    depth2();
}

int
main(int argc, const char *argv[]) {
    pthread_t pth;
    void *value;

    pthread_create(&pth, NULL, depth1, NULL);
    pthread_join(pth, &value);
    return 0;
}

```
請用下面指令編譯

```c
gcc -O0 -g -pthread -finstrument-functions -c test_corruption.c
gcc -o test test_corruption.o before_die.o -pthread
```
於是，執行 test 時，會產生 core dump。假設檔名是 test.core

```c
src$ ./test$
Thread ID = 0x28404300, bdfi @ 0x0x28404600
Thread ID = 0x28404c00, bdfi @ 0x0x28416500
Segmentation fault: 11 (core dumped)
```
於是我們用 gdb 檢視該 core file


```c
src$ gdb ./test test.core 
GNU gdb 6.1.1 [FreeBSD]
Copyright 2004 Free Software Foundation, Inc.
GDB is free software, covered by the GNU General Public License, and you are
welcome to change it and/or distribute copies of it under certain conditions.
Type "show copying" to see the conditions.
There is absolutely no warranty for GDB.  Type "show warranty" for details.
This GDB was configured as "i386-marcel-freebsd"...
Core was generated by `test'.
Program terminated with signal 11, Segmentation fault.
Reading symbols from /lib/libthr.so.3...done.
Loaded symbols for /lib/libthr.so.3
Reading symbols from /lib/libc.so.7...done.
Loaded symbols for /lib/libc.so.7
Reading symbols from /libexec/ld-elf.so.1...done.
Loaded symbols for /libexec/ld-elf.so.1
#0  0x08048654 in depth3 () at test_corruption.c:12
12              *ptr++ = 0xff;
[New Thread 28404c00 (LWP 114183/test)]
[New Thread 28404300 (LWP 100776/test)]
(gdb) info threads
  2 Thread 28404300 (LWP 100776/test)  0x2809e793 in pthread_kill ()
   from /lib/libthr.so.3
* 1 Thread 28404c00 (LWP 114183/test)  0x08048654 in depth3 ()
    at test_corruption.c:12
(gdb) bt
#0  0x08048654 in depth3 () at test_corruption.c:12
#1  0x000000ff in ?? ()
#2  0x000000ff in ?? ()
#3  0x000000ff in ?? ()
#4  0x000000ff in ?? ()
#5  0x000000ff in ?? ()
#6  0x000000ff in ?? ()
#7  0x000000ff in ?? ()
#8  0x000000ff in ?? ()
#9  0x000000ff in ?? ()
#10 0x000000ff in ?? ()
#11 0x000000ff in ?? ()
#12 0x000000ff in ?? ()
#13 0x000000ff in ?? ()
#14 0x000000ff in ?? ()
#15 0x000000ff in ?? ()
#16 0x000000ff in ?? ()
#17 0x000000ff in ?? ()
#18 0x000000ff in ?? ()
#19 0x000000ff in ?? ()
#20 0x000000ff in ?? ()
#21 0x000000ff in ?? ()
#22 0x000000ff in ?? ()
#23 0x000000ff in ?? ()
#24 0x000000ff in ?? ()
#25 0x000000ff in ?? ()
#26 0x000000ff in ?? ()
#27 0x000000ff in ?? ()
#28 0x000000ff in ?? ()
#29 0x000000ff in ?? ()
#30 0x000000ff in ?? ()
#31 0x000000ff in ?? ()
#32 0x000000ff in ?? ()
#33 0x000000ff in ?? ()
#34 0x000000ff in ?? ()
#35 0x000000ff in ?? ()
#36 0x000000ff in ?? ()
#37 0x000000ff in ?? ()
Cannot access memory at address 0xbf9ff000
(gdb) 

```

stack 已經被破壞了。 透過剛才執行印出來的資訊得知，出錯的 thread 的 thread local variable 位址在 0x28416500。我們能檢視該變數取得 call flow。


```c
(gdb) p ((struct bd_frame_info *)0x28416500)->top
$1 = 3
(gdb) p ((struct bd_frame_info *)0x28416500)->stack@3
$2 = {{0x2809646a}, {0x80486ee}, {0x80486ae}}
(gdb) info symbol 0x80486ae
depth2 + 30 in section .text
(gdb) info symbol 0x80486ee
depth1 + 30 in section .text
(gdb) info symbol 0x2809646a
pthread_getprio + 394 in section .text
```

我們現在得知，出錯的位址，是透過以下幾個 function 依序進行槽狀呼叫而來。
```c
depth1() @ 0x80486ee
depth2() @ 0x80486ae
```
於是我們就能縮小範圍到這樣的呼叫結構。

打完收工!!