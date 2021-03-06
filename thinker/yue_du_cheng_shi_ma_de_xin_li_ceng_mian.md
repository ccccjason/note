# 閱讀程式碼的心理層面

linkedin 上面，朋友會 endorse 我各式技能，像是 Linux、embedded system、 python、open source ... etc。 老實說，這些都不是我真實的技能。 如果要我說明技能為何，我會說「讀程式碼」。 我曾在 resume 裡，將「閱讀程式碼」列為我的重要技能，但被對方笑說這 算什麼技能。 然而，讀別人的程式碼是一種技藝，不是想看就能看的懂。 要看的快，理解的快，可不是那麼容易。

如果要比諭，「閱讀程式碼」這項技能就像「吸星大法」或是「鬥換星移」 這兩門功夫。 看到任何系統，只要有程式碼，就能很快的拆解、修改，納為己用。 因此有「吸星大法」之效。 因為「閱讀」的能力好，在久經各式系統之後，能很快的猜出系統的運作方式， 可以在不全盤瞭解時，就進行修改。 改出問題也能快速的找出問題，並修掉。 因此有「鬥換星移」之效。

##見林不見樹

心法之一，必需能自由的在不同層次的抽象層移動，而不會被細節影嚮。 經驗不足的 developer，通常會程式一行一行的讀，以為這樣才能真的瞭解 程式的運作。 但，有經驗的 developer 會區塊區塊的讀。 他們會大略的掃視一下程式碼，約略猜一下那些區塊是什麼功能， 並簡單的測試一下自己的假設。 這種方式能讓 developer 快速的瞭解系統的大架構，方便之後有需要時， 再深入閱那些區塊的細節。

當 developer 注意細節時，會將心智限縮在小範圍，以致於摸不透大架構。 透過大架構的瞭解，能快速的瞭解各檔案、目錄、模組間的作用。 在有需要時，就能快速的定位出需要精讀的部分。

原開發者也是人，因此他們會用一定的邏輯去組織他們的程式碼。 大區塊的閱讀的目的就是要先摸清原開發者的邏輯為何，以更瞭解 系統的運作。

##符號的提示

有很多人會問，是否有文件? 或註解? 我寧可直接看程式碼，比文件正確。 讀程式碼時，function name 或變數名稱也能發揮很有效的指引。 很多人不注意符號 (symbol) 在寫些什麼，反而一頭鑽入一行行的程式碼。 就像前面說的，原作者也是人，他自己也得讀自己的程式碼，因此會 取一些有意義的符號名稱。 與其讀文件或註解，我會選擇先去解理符號本身想要表達什麼。

##Header Files

以 C/C++ 為例，會有 header file，宣告 function 、 data type、 struct 、 class 等。 要了解一個模組，我會先讀 header file，先看該模組提供什麼樣的介面。 透過 function name、class name 、 class/struct members，能幫你在心裡描繪出 一個大概的輪廓。

##Data structure 經常比指令清楚

和讀指令比較起來，讀 data structure 往往更能掌握系統/模組的結構。 data structure 往往決定了指令輪廓。 如果一頭栽入指令裡，而不先看 data structure，往往會一頭霧水。 先觀察 data structure，往往你大概就能猜出程式碼的外觀。

##註解

註解有時也佷有幫助，但註解也常會誤導你。 因此，我不會把註解放在優先。 反而註解會是最後被注意到的東西。

##筆記

閱讀程式碼時，筆記是必需要。 筆記就像地圖，能幫你確認目前的位置，你是怎麼到達目前的位置，和其它 程式之間的關聯。 筆記能幫你快速切換到系統的不同位置，特別是你需要回到之前看過的位置， 重新做一些確認時。

筆記最好只是紀錄輪廓，大略的樣貌。 細節留給程式碼本身就好了。

##把持自己的慾望

大多數人，會有鑽進細節裡的慾望。 我認為閱讀程式碼需要一種修行，剋制自己的慾望。 閱讀過程必需時時清楚目前的階段目標是什麼，並檢視目前的行為、動作是否 能有效率的帶你往目標前進。 因此，你必需問自己，現在是否該先看看 data structure 長什麼樣? 是否該看 .cpp 的內容，或者先看 header file 就好? 是否該繼續順著 function call 更深入的瞭解? 或者先停在這裡?