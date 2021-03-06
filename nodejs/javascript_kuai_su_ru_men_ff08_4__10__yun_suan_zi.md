# JavaScript 快速入門（4/10）－ 運算子


程式是一連串運算式的組合，我們也看到了最簡單的變數設值運算式如下：

```js
x=100 ; 
```

如你所見，一段運算式除了變數 x ，另外還有一個要素便是運算子，其中的 = 即是一種運算子，它的功能是將右邊的值設定給左邊的變數，而這段運算式是一種設值運算式。
根據套用的運算子，JavaScript 有數種不同的運算式，包含算術運算式、比較運算式與運算式等等，JavaScript 提供了大量的運算子以支援各種程式運算作業。

##》基本算術運算子

數學四則運算加（+）、減（-）、乘（*）、除（/）為四個最基本的運算子，另外還有一個模數（%）運算子，此運算子取回兩個數字相除的的餘數。考慮以下的程式片段：

```js
<script>
    var a = 100;
    var b = 16;
    console.log(a + b);
    console.log(a - b);
    console.log(a * b);
    console.log(a / b);
    console.log(a % b);
</script>
```

首先宣告兩個變數，進行四則運算，輸出結果如下：

```js
116
84
1600
6.25
4
```

四則運算很容理解，最後一個值要特別注意，運算子 % 回傳的是兩個數字相除的餘數，因此 100/16 的最後結果是 4 ，除此之外，運算子 + 運算在字串的場合會進行合併運算，如果是數字則會進行加總。緊接著來看數值的合併：

```js
<script>
    var a = '100';
    var b = '200';
    var x = 100;
    var y = 200;
    console.log(a + b);
    console.log(x + y);
</script>
```

此段程式碼分別以字串格式與數字格式宣告四個變數，並且分別進行+運算，輸出結果如下：

```js
100200
300
100100
```


除了第二行的輸出是將兩個數值加線之外，其它第一行與第三行的輸出由於包含了字串，因此將以合併的方式進行輸出。如果在運算式中包含了布林值（true/false），true被當作 1 處理，而 false 則被當作 0 處理，例如以下的程式片段：

```js
true + true      // 輸出 2
false + true 　  // 輸出 1
false + false 　 // 輸出 0
true + 100　     // 輸出 101
false + 100　    // 輸出 100
```

除非兩個運算元均是數值，否則會出現非數字相加的結果。

##》運算後設值

四則與模數運算子，可以進一步結合設值運算子，執行設值後運算，考慮以下的運算式：

```js
var a=200 ;
a=a+100 ; 
```

其中將 a 變數的值加上 100 ，然後再設定給 a ，因此 a 最後的結果為自己本身的值加上 100 ，因此最後a的值是 300 。現在利用設值後運算子進行相同的運算如下：

```js
a += 100 ; 
```

這一行執行完畢之後，a的結果同樣是 300 ，其它數個運算子，包含減（-）、乘（*）、除（/）與模數（%）意義相同。

##》一元運算子

加（＋）與減（-）同時可作為一元運算子，針對單一運算元，進行數值的轉換，考慮以下的運算式：

```js
var x = '100'; 
var y = +x ;    // y 是數值100
var y = -y ;    // z 是數值-100
```

如果運算元可以轉換為數值，則 + 將完成正數值轉換，而 – 則完成負數值轉換，而運算元若是無法轉成字串，則回傳一個 NaN ，例如將上述的 x 調整如下：

```js
var x = 'Hello';  
```

接下來的 + 與 – 輸出結果均是 NaN。

另外還有遞增（++）與遞減運算子（--），這兩組運算子針對指定的數值，進行加1（遞增）減1（遞減）的運算。假設宣告一個y變數如下：

```js
var y = 100 ;  
```

如果針對 y 進行遞增運算，例如 ++y ，則結果為 101 ，如果是遞減運算 –-y ，則結果為 99 。無論遞增或是遞減運算，均可配置於運算元前方或後方。如果配置於前方，則是先運算，運算元會先執行運算，取出的將是遞增/遞減運算的值，而配置於後方則是後運算，運算元的值會先被取出，然後才進行遞增/遞減運算。現在考慮以下的程式片段：

```js
<script>
    var x = 100;
    var y = 100;
    var a = ++x;
    var b = y++;
    console.log(a + ',' + x);
    console.log(b + ',' + y);
</script>
```

變數 x 進行前置運算，然後將結果值指定給 x，而變數 y則進行後置運算，將值指定給 b 。

```js
101,101
100,101 
```


由於前置運算會完成運算再設值，因此 x 完成遞增運算結果為 101 ，再進一步設定給 a 變數，如此一來，a 的值亦為 101 。後置運算會先進行設值再執行遞增，因此設定給變數的值是 100 原來的值，而 y 最後完成遞增運算，因此為 101 。

##》關係運算子

關係運算子測試兩個運算元的關係，包含相等性與大小比較，最後的結果則是一個布林值（true/false）值。

###- 相等性運算子 -

比較兩個運算元是否相等的運算子有 == 與 === ，一般的相等性比較使用 == 即可，如果是嚴格的相等性比較則使用 === 。考慮以下的 a 與 b 變數：

```js
var a = 0 ;
var b = false ;
```

由於 false 轉換為數值是 0 ，因此a與b進行不嚴格的相等性比較時，會得到相等的結果，不過兩個值實際上並不相同，因此若是進行嚴格檢查時，回傳的結果將是否定的。

```js
a == b ;   //相等因此回傳 true
a === b ;  //不相等因此回傳 flase 
```

null 與undefined ，透過相等性運算，可以更進一步看出其差異，如下式：


```js
null == undefined     //相等因此回傳 true
null === undefined    //不相等因此回傳 false
```


比較不嚴格的相等性運算，透過 == 比較得到的是相等（true）的結果，而 === 則是相反的結果。讀者必須注意的是，當兩個運算元是不同型別，則此兩運算元無法通過嚴格比較，因此 === 運算子一定會回傳 false ，而 == 則不一定，如果經過型態轉換之後，兩個運算元具有相同的值，則還是會相等，如下式：

```js
console.log('123' == 123);
console.log('123' === 123);
```

另外比較特別的是NaN，當你將其與另外一個NaN作比較，會得到一個false 的結果，因為 NaN 包含自己在內容完全沒有相等的值。

如果要測試不相等性，則使用 != 與 !== 這一組運算子即可，相較於前述的 == 與 === 邏輯剛好相反，當比較的兩個運算元相等，則回傳 false ，否則回傳 true 。

##- 比較運算子 -

比較運算子比較兩個運算元的大小，有大於（>）、小於（<）、大於等於（>=）、小於等於（<=）等四種，這一組運算子針對字串與數字型別運算元作比較，字串根據組成的字元Unicode值進行比較，以字母順序排列，而對於相同的字母而言，大寫字母小於小寫字母。

```js
'Abc' < 'abc'　//回傳 true
'a' < 'b'       //回傳 true
```

其中大寫 A 小於小寫 a ，因此回傳 true ，而小寫 a 排在小寫 b 前面，因此兩行程式碼的判斷式回傳結果均是 true 。 數字則依大小進行比較，在比較的運算上，Infinity 是最大的值，-Infinity 則是最小的值，NaN無法作大小比較，因此任何一個運算元中出現 NaN 的比較結果都是 false 。 

##》邏輯運算子

邏輯運算子進一步串聯布林值進行邏輯運算，相關的運算子有 AND （&&）與OR（||）。

AND 運算子必須兩個運算元都是 true 時才會回傳 true，否則一律回傳 false 。而OR運算只要有一個運算元是 true，回傳結果為 true ，只有兩個運算都是 flase 才會回傳 false 。

```js
//AND 邏輯運算
false && false 　//回傳 false
true && false    //回傳 false
true && true     //回傳 true
//OR 邏輯運算
false || false 　//回傳 false
true || false    //回傳 true
true || true     //回傳 true 
```

邏輯運算子通常不會直接以布林值為運算元，比較常見的用途與前述的關係運算子結合，進行更複雜的運算，例如以下的程式碼：

```js
(x == y) || (x == z)
```

其中根據 x==y 與 x==z 兩個相等性的比較結果，進行邏輯 AND 運算。
相等性運算子、比較運算子以及邏輯運算子的運用場合常見於條件判斷或是迴圈等敘述句，稍後讀者會看到進一步的應用。

##》三元運算子

三元運算子（?:）是一個特殊的運算子，它有三個運算元，可以進行簡易的true/false判斷運算，並且根據運算結果，決定最後的值，也被稱為條件運算子，語法如下：

```js
test ? exp1 : exp2
```

test 為一條件運算式，如果其結果值為true，則回傳exp1，若是結果值為false，則回傳exp2。考慮以下的程式碼片段：

```js
<script>
    function positive(x) {
        var n = x > 0 ? x : 0;
        return n;
    }
    console.log(positive(-1000));
    console.log(positive(1000));
</script>
```

函式positive 針對參數x進行判斷，如果這個值大於零，直接將其回傳，否則的話一律回傳0，緊接著測試此函式，分別傳入一個負數與一個正數，得到以下的輸出結果。

```js
0
1000
```


由於-1000小於零，因此回傳值是0，而1000大於零因此輸出1000。