# 管理應用的內存

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/articles/memory.html>

Random Access Memory(RAM)在任何軟件開發環境中都是一個很寶貴的資源。這一點在物理內存通常很有限的移動操作系統上，顯得尤為突出。儘管Android的Dalvik虛擬機扮演了常規的垃圾回收的角色，但這並不意味著你可以忽視app的內存分配與釋放的時機與地點。

為了GC能夠從app中及時回收內存，我們需要注意避免內存洩露(通常由於在全局成員變量中持有對象引用而導致)並且在適當的時機(下面會講到的lifecycle callbacks)來釋放引用對象。對於大多數app來說，Dalvik的GC會自動把離開活動線程的對象進行回收。

這篇文章會解釋Android是如何管理app的進程與內存分配，以及在開發Android應用的時候如何主動的減少內存的使用。關於Java的資源管理機制，請參考其它書籍或者線上材料。如果你正在尋找如何分析你的內存使用情況的文章，請參考這裡[Investigating Your RAM Usage](http://developer.android.com/tools/debugging/debugging-memory.html)。

## 第1部分: Android是如何管理內存的

Android並沒有為內存提供交換區(Swap space)，但是它有使用[paging](http://en.wikipedia.org/wiki/Paging)與[memory-mapping(mmapping)](http://en.wikipedia.org/wiki/Memory-mapped_files)的機制來管理內存。這意味著任何你修改的內存(無論是通過分配新的對象還是去訪問mmaped pages中的內容)都會貯存在RAM中，而且不能被paged out。因此唯一完整釋放內存的方法是釋放那些你可能hold住的對象的引用，當這個對象沒有被任何其他對象所引用的時候，它就能夠被GC回收了。只有一種例外是：如果系統想要在其他地方重用這個對象。

### 1) 共享內存

Android通過下面幾個方式在不同的進程中來實現共享RAM:

* 每一個app的進程都是從一個被叫做**Zygote**的進程中fork出來的。Zygote進程在系統啟動並且載入通用的framework的代碼與資源之後開始啟動。為了啟動一個新的程序進程，系統會fork Zygote進程生成一個新的進程，然後在新的進程中加載並運行app的代碼。這使得大多數的RAM pages被用來分配給framework的代碼，同時使得RAM資源能夠在應用的所有進程中進行共享。

* 大多數static的數據被mmapped到一個進程中。這不僅僅使得同樣的數據能夠在進程間進行共享，而且使得它能夠在需要的時候被paged out。例如下面幾種static的數據:
	* Dalvik 代碼 (放在一個預鏈接好的 .odex 文件中以便直接mapping)
	* App resources (通過把資源表結構設計成便於mmapping的數據結構，另外還可以通過把APK中的文件做aligning的操作來優化)
	* 傳統項目元素，比如 .so 文件中的本地代碼.
* 在很多情況下，Android通過顯式的分配共享內存區域(例如ashmem或者gralloc)來實現一些動態RAM區域能夠在不同進程間進行共享。例如，window surfaces在app與screen compositor之間使用共享的內存，cursor buffers在content provider與client之間使用共享的內存。

關於如何查看app所使用的共享內存，請查看[Investigating Your RAM Usage](http://developer.android.com/tools/debugging/debugging-memory.html)

### 2) 分配與回收內存

這裡有下面幾點關於Android如何分配與回收內存的事實：

* 每一個進程的Dalvik heap都有一個受限的虛擬內存範圍。這就是邏輯上講的heap size，它可以隨著需要進行增長，但是會有一個系統為它所定義的上限。
* 邏輯上講的heap size和實際物理上使用的內存數量是不等的，Android會計算一個叫做Proportional Set Size(PSS)的值，它記錄了那些和其他進程進行共享的內存大小。（假設共享內存大小是10M，一共有20個Process在共享使用，根據權重，可能認為其中有0.3M才能真正算是你的進程所使用的）
* Dalvik heap與邏輯上的heap size不吻合，這意味著Android並不會去做heap中的碎片整理用來關閉空閒區域。Android僅僅會在heap的尾端出現不使用的空間時才會做收縮邏輯heap size大小的動作。但是這並不是意味著被heap所使用的物理內存大小不能被收縮。在垃圾回收之後，Dalvik會遍歷heap並找出不使用的pages，然後使用madvise(系統調用)把那些pages返回給kernal。因此，成對的allocations與deallocations大塊的數據可以使得物理內存能夠被正常的回收。然而，回收碎片化的內存則會使得效率低下很多，因為那些碎片化的分配頁面也許會被其他地方所共享到。

### 3) 限制應用的內存

為了維持多任務的功能環境，Android為每一個app都設置了一個硬性的heap size限制。準確的heap size限制會因為不同設備的不同RAM大小而各有差異。如果你的app已經到了heap的限制大小並且再嘗試分配內存的話，會引起`OutOfMemoryError`的錯誤。

在一些情況下，你也許想要查詢當前設備的heap size限制大小是多少，然後決定cache的大小。可以通過`getMemoryClass()`來查詢。這個方法會返回一個整數，表明你的應用的heap size限制是多少Mb(megabates)。

### 4) 切換應用

Android並不會在用戶切換不同應用時候做交換內存的操作。Android會把那些不包含foreground組件的進程放到LRU cache中。例如，當用戶剛開始啟動了一個應用，系統會為它創建了一個進程，但是當用戶離開這個應用，此進程並不會立即被銷燬。系統會把這個進程放到cache中，如果用戶後來再回到這個應用，此進程就能夠被完整恢復，從而實現應用的快速切換。

如果你的應用中有一個被緩存的進程，這個進程會佔用暫時不需要使用到的內存，這個暫時不需要使用的進程，它被保留在內存中，這會對系統的整體性能有影響。因此當系統開始進入低內存狀態時，它會由系統根據LRU的規則與其他因素選擇綜合考慮之後決定殺掉某些進程，為了保持你的進程能夠儘可能長久的被緩存，請參考下面的章節學習何時釋放你的引用。

對於那些不在foreground的進程，Android是如何決定kill掉哪一類進程的問題，請參考[Processes and Threads](http://developer.android.com/guide/components/processes-and-threads.html).

## 第2部分: 你的應用該如何管理內存

你應該在開發過程的每一個階段都考慮到RAM的有限性，甚至包括在開始編寫代碼之前的設計階段就應該考慮到RAM的限制性。我們可以使用多種設計與實現方式，他們有著不同的效率，即使這些方式只是相同技術的不斷組合與演變。

為了使得你的應用性能效率更高，你應該在設計與實現代碼時，遵循下面的技術要點。

### 1) 珍惜Services資源

如果你的應用需要在後臺使用service，除非它被觸發並執行一個任務，否則其他時候service都應該是停止狀態。另外需要注意當這個service完成任務之後因為停止service失敗而引起的內存洩漏。

當你啟動一個service，系統會傾向為了保留這個service而一直保留service所在的進程。這使得進程的運行代價很高，因為系統沒有辦法把service所佔用的RAM空間騰出來讓給其他組件，另外service還不能被paged out。這減少了系統能夠存放到LRU緩存當中的進程數量，它會影響app之間的切換效率。它甚至會導致系統內存使用不穩定，從而無法繼續保持住所有目前正在運行的service。

限制你的service的最好辦法是使用[IntentService](http://developer.android.com/reference/android/app/IntentService.html)， 它會在處理完交代給它的intent任務之後儘快結束自己。更多信息，請閱讀[Running in a Background Service](http://developer.android.com/training/run-background-service/index.html).

當一個Service已經不再需要的時候還繼續保留它，這對Android應用的內存管理來說是**最糟糕的錯誤之一**。因此千萬不要貪婪的使得一個Service持續保留。不僅僅是因為它會使得你的應用因為RAM空間的不足而性能糟糕，還會使得用戶發現那些有著常駐後臺行為的應用並且可能卸載它。

### 2) 當UI隱藏時釋放內存

當用戶切換到其它應用並且你的應用 UI不再可見時，你應該釋放你的應用UI上所佔用的所有內存資源。在這個時候釋放UI資源可以顯著的增加系統緩存進程的能力，它會對用戶體驗有著很直接的影響。

為了能夠接收到用戶離開你的UI時的通知，你需要實現Activtiy類裡面的`onTrimMemory()`回調方法。你應該使用這個方法來監聽到`TRIM_MEMORY_UI_HIDDEN`級別的回調，此時意味著你的UI已經隱藏，你應該釋放那些僅僅被你的UI使用的資源。

請注意：你的應用僅僅會在所有UI組件的被隱藏的時候接收到`onTrimMemory()`的回調並帶有參數`TRIM_MEMORY_UI_HIDDEN`。這與onStop()的回調是不同的，onStop會在activity的實例隱藏時會執行，例如當用戶從你的app的某個activity跳轉到另外一個activity時前面activity的onStop()會被執行。因此你應該實現onStop回調，並且在此回調裡面釋放activity的資源，例如釋放網絡連接，註銷監聽廣播接收者。除非接收到[onTrimMemory(TRIM_MEMORY_UI_HIDDEN)](http://developer.android.com/reference/android/content/ComponentCallbacks2.html#onTrimMemory(int))的回調，否者你不應該釋放你的UI資源。這確保了用戶從其他activity切回來時，你的UI資源仍然可用，並且可以迅速恢復activity。

### 3) 當內存緊張時釋放部分內存

在你的app生命週期的任何階段，onTrimMemory的回調方法同樣可以告訴你整個設備的內存資源已經開始緊張。你應該根據onTrimMemory回調中的內存級別來進一步決定釋放哪些資源。

* [TRIM_MEMORY_RUNNING_MODERATE](http://developer.android.com/reference/android/content/ComponentCallbacks2.html#TRIM_MEMORY_RUNNING_MODERATE)：你的app正在運行並且不會被列為可殺死的。但是設備此時正運行於低內存狀態下，系統開始觸發殺死LRU Cache中的Process的機制。
* [TRIM_MEMORY_RUNNING_LOW](http://developer.android.com/reference/android/content/ComponentCallbacks2.html#TRIM_MEMORY_RUNNING_LOW)：你的app正在運行且沒有被列為可殺死的。但是設備正運行於更低內存的狀態下，你應該釋放不用的資源用來提升系統性能（但是這也會直接影響到你的app的性能）。
* [TRIM_MEMORY_RUNNING_CRITICAL](http://developer.android.com/reference/android/content/ComponentCallbacks2.html#TRIM_MEMORY_RUNNING_CRITICAL)：你的app仍在運行，但是系統已經把LRU Cache中的大多數進程都已經殺死，因此你應該立即釋放所有非必須的資源。如果系統不能回收到足夠的RAM數量，系統將會清除所有的LRU緩存中的進程，並且開始殺死那些之前被認為不應該殺死的進程，例如那個包含了一個運行態Service的進程。

同樣，當你的app進程正在被cached時，你可能會接受到從onTrimMemory()中返回的下面的值之一:

* [TRIM_MEMORY_BACKGROUND](http://developer.android.com/reference/android/content/ComponentCallbacks2.html#TRIM_MEMORY_BACKGROUND): 系統正運行於低內存狀態並且你的進程正處於LRU緩存名單中**最不容易殺掉的位置**。儘管你的app進程並不是處於被殺掉的高危險狀態，系統可能已經開始殺掉LRU緩存中的其他進程了。你應該釋放那些容易恢復的資源，以便於你的進程可以保留下來，這樣當用戶回退到你的app的時候才能夠迅速恢復。
* [TRIM_MEMORY_MODERATE](http://developer.android.com/reference/android/content/ComponentCallbacks2.html#TRIM_MEMORY_MODERATE): 系統正運行於低內存狀態並且你的進程已經已經接近LRU名單的**中部位置**。如果系統開始變得更加內存緊張，你的進程是有可能被殺死的。
* [TRIM_MEMORY_COMPLETE](http://developer.android.com/reference/android/content/ComponentCallbacks2.html#TRIM_MEMORY_COMPLETE): 系統正運行與低內存的狀態並且你的進程正處於LRU名單中**最容易被殺掉的位置**。你應該釋放任何不影響你的app恢復狀態的資源。

因為onTrimMemory()的回調是在**API 14**才被加進來的，對於老的版本，你可以使用[onLowMemory](http://developer.android.com/reference/android/content/ComponentCallbacks.html#onLowMemory())回調來進行兼容。onLowMemory相當與`TRIM_MEMORY_COMPLETE`。

> **Note:** 當系統開始清除LRU緩存中的進程時，儘管它首先按照LRU的順序來操作，但是它同樣會考慮進程的內存使用量。因此消耗越少的進程則越容易被留下來。

### 4) 檢查你應該使用多少的內存

正如前面提到的，每一個Android設備都會有不同的RAM總大小與可用空間，因此不同設備為app提供了不同大小的heap限制。你可以通過調用[getMemoryClass()](http://developer.android.com/reference/android/app/ActivityManager.html#getMemoryClass())來獲取你的app的可用heap大小。如果你的app嘗試申請更多的內存，會出現`OutOfMemory`的錯誤。

在一些特殊的情景下，你可以通過在manifest的application標籤下添加`largeHeap=true`的屬性來聲明一個更大的heap空間。如果你這樣做，你可以通過[getLargeMemoryClass()](http://developer.android.com/reference/android/app/ActivityManager.html#getLargeMemoryClass())來獲取到一個更大的heap size。

然而，能夠獲取更大heap的設計本意是為了一小部分會消耗大量RAM的應用(例如一個大圖片的編輯應用)。**不要輕易的因為你需要使用大量的內存而去請求一個大的heap size。**只有當你清楚的知道哪裡會使用大量的內存並且為什麼這些內存必須被保留時才去使用large heap. 因此請儘量少使用large heap。使用額外的內存會影響系統整體的用戶體驗，並且會使得GC的每次運行時間更長。在任務切換時，系統的性能會變得大打折扣。

另外, large heap並不一定能夠獲取到更大的heap。在某些有嚴格限制的機器上，large heap的大小和通常的heap size是一樣的。因此即使你申請了large heap，你還是應該通過執行getMemoryClass()來檢查實際獲取到的heap大小。

### 5) 避免bitmaps的浪費

當你加載一個bitmap時，僅僅需要保留適配當前屏幕設備分辨率的數據即可，如果原圖高於你的設備分辨率，需要做縮小的動作。請記住，增加bitmap的尺寸會對內存呈現出2次方的增加，因為X與Y都在增加。

> **Note:**在Android 2.3.x (API level 10)及其以下, bitmap對象的pixel data是存放在native內存中的，它不便於調試。然而，從Android 3.0(API level 11)開始，bitmap pixel data是分配在你的app的Dalvik heap中, 這提升了GC的工作效率並且更加容易Debug。因此如果你的app使用bitmap並在舊的機器上引發了一些內存問題，切換到3.0以上的機器上進行Debug。

### 6) 使用優化的數據容器

利用Android Framework裡面優化過的容器類，例如[SparseArray](http://developer.android.com/reference/android/util/SparseArray.html), [SparseBooleanArray](http://developer.android.com/reference/android/util/SparseBooleanArray.html), 與 [LongSparseArray](http://developer.android.com/reference/android/support/v4/util/LongSparseArray.html)。 通常的HashMap的實現方式更加消耗內存，因為它需要一個額外的實例對象來記錄Mapping操作。另外，SparseArray更加高效在於他們避免了對key與value的autobox自動裝箱，並且避免了裝箱後的解箱。

### 7) 請注意內存開銷

對你所使用的語言與庫的成本與開銷有所瞭解，從開始到結束，在設計你的app時謹記這些信息。通常，表面上看起來無關痛癢(innocuous)的事情也許實際上會導致大量的開銷。例如：

* Enums的內存消耗通常是static constants的2倍。你應該儘量避免在Android上使用enums。
* 在Java中的每一個類(包括匿名內部類)都會使用大概500 bytes。
* 每一個類的實例花銷是12-16 bytes。
* 往HashMap添加一個entry需要額一個額外佔用的32 bytes的entry對象。

### 8) 請注意代碼“抽象”

通常，開發者使用抽象作為"好的編程實踐"，因為抽象能夠提升代碼的靈活性與可維護性。然而，抽象會導致一個顯著的開銷：通常他們需要同等量的代碼用於可執行。那些代碼會被map到內存中。因此如果你的抽象沒有顯著的提升效率，應該儘量避免他們。

### 9) 為序列化的數據使用nano protobufs

[Protocol buffers](https://developers.google.com/protocol-buffers/docs/overview)是由Google為序列化結構數據而設計的，一種語言無關，平臺無關，具有良好擴展性的協議。類似XML，卻比XML更加輕量，快速，簡單。如果你需要為你的數據實現協議化，你應該在客戶端的代碼中總是使用nano protobufs。通常的協議化操作會生成大量繁瑣的代碼，這容易給你的app帶來許多問題：增加RAM的使用量，顯著增加APK的大小，更慢的執行速度，更容易達到DEX的字符限制。

關於更多細節，請參考[protobuf readme](https://android.googlesource.com/platform/external/protobuf/+/master/java/README.txt)的"Nano version"章節。

### 10) 避免使用依賴注入框架

使用類似[Guice](https://code.google.com/p/google-guice/)或者[RoboGuice](https://github.com/roboguice/roboguice)等framework injection包是很有效的，因為他們能夠簡化你的代碼。

> Notes：RoboGuice 2 通過依賴注入改變代碼風格，讓Android開發時的體驗更好。你在調用 `getIntent().getExtras()` 時經常忘記檢查 null 嗎？RoboGuice 2 可以幫你做。你認為將 `findViewById()` 的返回值強制轉換成 TextView 是本不必要的工作嗎？ RoboGuice 2 可以幫你。RoboGuice 把這些需要猜測性的工作移到Android開發以外去了。RoboGuice 2 會負責注入你的 View, Resource, System Service或者其他對象等等類似的細節。

然而，那些框架會通過掃描你的代碼執行許多初始化的操作，這會導致你的代碼需要大量的RAM來mapping代碼，而且mapped pages會長時間的被保留在RAM中。

### 11) 謹慎使用第三方libraries

很多開源的library代碼都不是為移動網絡環境而編寫的，如果運用在移動設備上，，這樣的效率並不高。當你決定使用一個第三方library的時候，你應該針對移動網絡做繁瑣的遷移與維護的工作。

即使是針對Android而設計的library，也可能是很危險的，因為每一個library所做的事情都是不一樣的。例如，其中一個lib使用的是nano protobufs, 而另外一個使用的是micro protobufs。那麼這樣，在你的app裡面就有2種protobuf的實現方式。這樣的衝突同樣可能發生在輸出日誌，加載圖片，緩存等等模塊裡面。

同樣不要陷入為了1個或者2個功能而導入整個library的陷阱。如果沒有一個合適的庫與你的需求相吻合，你應該考慮自己去實現，而不是導入一個大而全的解決方案。

### 12) 優化整體性能

官方有列出許多優化整個app性能的文章：[Best Practices for Performance](http://developer.android.com/training/best-performance.html)。這篇文章就是其中之一。有些文章是講解如何優化app的CPU使用效率，有些是如何優化app的內存使用效率。

你還應該閱讀[optimizing your UI](http://developer.android.com/tools/debugging/debugging-ui.html)來為layout進行優化。同樣還應該關注lint工具所提出的建議，進行優化。

### 13) 使用ProGuard來剔除不需要的代碼

[ProGuard](http://developer.android.com/tools/help/proguard.html)能夠通過移除不需要的代碼，重命名類，域與方法等方對代碼進行壓縮，優化與混淆。使用ProGuard可以使得你的代碼更加緊湊，這樣能夠使用更少mapped代碼所需要的RAM。

### 14) 對最終的APK使用zipalign

在編寫完所有代碼，並通過編譯系統生成APK之後，你需要使用[zipalign](http://developer.android.com/tools/help/zipalign.html)對APK進行重新校準。如果你不做這個步驟，會導致你的APK需要更多的RAM，因為一些類似圖片資源的東西不能被mapped。

> **Notes: **Google Play不接受沒有經過zipalign的APK。

### 15) 分析你的RAM使用情況

一旦你獲取到一個相對穩定的版本後，需要分析你的app整個生命週期內使用的內存情況，並進行優化，更多細節請參考[Investigating Your RAM Usage](http://developer.android.com/tools/debugging/debugging-memory.html).

### 16) 使用多進程

如果合適的話，有一個更高級的技術可以幫助你的app管理內存使用：通過把你的app組件切分成多個組件，運行在不同的進程中。這個技術必須謹慎使用，大多數app都不應該運行在多個進程中。因為如果使用不當，它會顯著增加內存的使用，而不是減少。當你的app需要在後臺運行與前臺一樣的大量的任務的時候，可以考慮使用這個技術。

一個典型的例子是創建一個可以長時間後臺播放的Music Player。如果整個app運行在一個進程中，當後臺播放的時候，前臺的那些UI資源也沒有辦法得到釋放。類似這樣的app可以切分成2個進程：一個用來操作UI，另外一個用來後臺的Service.

你可以通過在manifest文件中聲明'android:process'屬性來實現某個組件運行在另外一個進程的操作。

```xml
<service android:name=".PlaybackService"
         android:process=":background" />
```

更多關於使用這個技術的細節，請參考原文，鏈接如下。
<http://developer.android.com/training/articles/memory.html>
