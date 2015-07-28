# JNI Tips

> 編寫:[pedant](https://github.com/pedant) - 原文:<http://developer.android.com/training/articles/perf-jni.html>

JNI全稱Java Native Interface。它為託管代碼（使用Java編程語言編寫）與本地代碼（使用C/C++編寫）提供了一種交互方式。它是<font color='red'>與廠商無關的（vendor-neutral）</font>,支持從動態共享庫中加載代碼，雖然這樣會稍顯麻煩，但有時這是相當有效的。

如果你對JNI還不是太熟悉，可以先通讀[Java Native Interface Specification](http://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/jniTOC.html)這篇文章來對JNI如何工作以及哪些特性可用有個大致的印象。這種接口的一些方面不能立即一讀就顯而易見，所以你會發現接下來的幾個章節很有用處。

# JavaVM 及 JNIEnv

JNI定義了兩種關鍵數據結構，“JavaVM”和“JNIEnv”。它們本質上都是指向函數表指針的指針（在C++版本中，它們被定義為類，該類包含一個指向函數表的指針，以及一系列可以通過這個函數表間接地訪問對應的JNI函數的成員函數）。JavaVM提供“調用接口（invocation interface）”函數, 允許你創建和銷燬一個JavaVM。理論上你可以在一個進程中擁有多個JavaVM對象，但安卓只允許一個。

JNIEnv提供了大部分JNI功能。你定義的所有本地函數都會接收JNIEnv作為第一個參數。

JNIEnv是用作線程局部存儲。因此，**你不能在線程間共享一個JNIEnv變量**。如果在一段代碼中沒有其它辦法獲得它的JNIEnv，你可以共享JavaVM對象，使用GetEnv來取得該線程下的JNIEnv（如果該線程有一個JavaVM的話；見下面的AttachCurrentThread）。

JNIEnv和JavaVM的在C聲明是不同於在C++的聲明。頭文件“jni.h”根據它是以C還是以C++模式包含來提供不同的類型定義（typedefs）。因此，不建議把JNIEnv參數放到可能被兩種語言引入的頭文件中（換一句話說：如果你的頭文件需要#ifdef __cplusplus，你可能不得不在任何涉及到JNIEnv的內容處都要做些額外的工作）。

#線程

所有的線程都是Linux線程，由內核統一調度。它們通常從託管代碼中啟動（使用Thread.start），但它們也能夠在其他任何地方創建，然後連接（attach）到JavaVM。例如，一個用pthread_create啟動的線程能夠使用JNI AttachCurrentThread 或 AttachCurrentThreadAsDaemon函數連接到JavaVM。在一個線程成功連接（attach）之前，它沒有JNIEnv，**不能夠調用JNI函數**。

連接一個本地環境創建的線程會觸發構造一個java.lang.Thread對象，然後其被添加到主線程群組（main ThreadGroup）,以讓調試器可以探測到。對一個已經連接的線程使用AttachCurrentThread不做任何操作（no-op）。

安卓不能中止正在執行本地代碼的線程。如果正在進行垃圾回收，或者調試器已發出了中止請求，安卓會在下一次調用JNI函數的時候中止線程。

連接過的（attached）線程在它們退出之前**必須通過JNI調用DetachCurrentThread**。如果你覺得直接這樣編寫不太優雅，在安卓2.0（Eclair）及以上， 你可以使用pthread_key_create來定義一個析構函數，它將會在線程退出時被調用，你可以在那兒調用DetachCurrentThread （使用生成的key與pthread_setspecific將JNIEnv存儲到線程局部空間內；這樣JNIEnv能夠作為參數傳入到析構函數當中去）。

#jclass, jmethodID, jfieldID

如果你想在本地代碼中訪問一個對象的字段（field）,你可以像下面這樣做：

- 對於類，使用FindClass獲得類對象的引用
- 對於字段，使用GetFieldId獲得字段ID
- 使用對應的方法（例如GetIntField）獲取字段下面的值

類似地，要調用一個方法，你首先得獲得一個類對象的引用，然後是方法ID（method ID）。這些ID通常是指向運行時內部數據結構。查找到它們需要些字符串比較，但一旦你實際去執行它們獲得字段或者做方法調用是非常快的。

如果性能是你看重的，那麼一旦查找出這些值之後在你的本地代碼中緩存這些結果是非常有用的。因為每個進程當中的JavaVM是存在限制的，存儲這些數據到本地靜態數據結構中是非常合理的。

類引用（class reference），字段ID（field ID）以及方法ID（method ID）在類被卸載前都是有效的。如果與一個類加載器（ClassLoader）相關的所有類都能夠被垃圾回收，但是這種情況在安卓上是罕見甚至不可能出現，只有這時類才被卸載。注意雖然jclass是一個類引用，但是**必須要調用NewGlobalRef保護起來**（見下個章節）。

當一個類被加載時如果你想緩存些ID，而後當這個類被卸載後再次載入時能夠自動地更新這些緩存ID，正確做法是在對應的類中添加一段像下面的代碼來初始化這些ID：

``` java

/*
 * 我們在一個類初始化時調用本地方法來緩存一些字段的偏移信息
 * 這個本地方法查找並緩存你感興趣的class/field/method ID
 * 失敗時拋出異常
 */
private static native void nativeInit();

static {
    nativeInit();
}
```

在你的C/C++代碼中創建一個nativeClassInit方法以完成ID查找的工作。當這個類被初始化時這段代碼將會執行一次。當這個類被卸載後而後再次載入時，這段代碼將會再次執行。

#局部和全局引用

每個傳入本地方法的參數，以及大部分JNI函數返回的每個對象都是“局部引用”。這意味著它只在當前線程的當前方法執行期間有效。**即使這個對象本身在本地方法返回之後仍然存在，這個引用也是無效的**。

這同樣適用於所有jobject的子類，包括jclass，jstring，以及jarray（當JNI擴展檢查是打開的時候，運行時會警告你對大部分對象引用的誤用）。

如果你想持有一個引用更長的時間，你就必須使用一個全局（“global”）引用了。NewGlobalRef函數以一個局部引用作為參數並且返回一個全局引用。全局引用能夠保證在你調用DeleteGlobalRef前都是有效的。

這種模式通常被用在緩存一個從FindClass返回的jclass對象的時候，例如：

``` java

jclass localClass = env->FindClass("MyClass");
jclass globalClass = reinterpret_cast<jclass>(env->NewGlobalRef(localClass));
```

所有的JNI方法都接收局部引用和全局引用作為參數。相同對象的引用卻可能具有不同的值。例如，用相同對象連續地調用NewGlobalRef得到返回值可能是不同的。**為了檢查兩個引用是否指向的是同一個對象，你必須使用IsSameObject函數**。絕不要在本地代碼中用==符號來比較兩個引用。

得出的結論就是你**絕不要在本地代碼中假定對象的引用是常量或者是唯一的**。代表一個對象的32位值從方法的一次調用到下一次調用可能有不同的值。在連續的調用過程中兩個不同的對象卻可能擁有相同的32位值。不要使用jobject的值作為key.

開發者需要“不過度分配”局部引用。在實際操作中這意味著如果你正在創建大量的局部引用，或許是通過對象數組，你應該使用DeleteLocalRef手動地釋放它們，而不是寄希望JNI來為你做這些。實現上只預留了16個局部引用的空間，所以如果你需要更多，要麼你刪掉以前的，要麼使用EnsureLocalCapacity/PushLocalFrame來預留更多。

注意jfieldID和jmethodID是<font color='red'>映射類型（opaque types）</font>，不是對象引用，不應該被傳入到NewGlobalRef。原始數據指針，像GetStringUTFChars和GetByteArrayElements的返回值，也都不是對象（它們能夠在線程間傳遞，並且在調用對應的Release函數之前都是有效的）。

還有一種不常見的情況值得一提，如果你使用AttachCurrentThread連接（attach）了本地進程，正在運行的代碼在線程分離（detach）之前決不會自動釋放局部引用。你創建的任何局部引用必須手動刪除。通常，任何在循環中創建局部引用的本地代碼可能都需要做一些手動刪除。

#UTF-8、UTF-16 字符串

Java編程語言使用UTF-16格式。為了便利，JNI也提供了支持[變形UTF-8（Modified UTF-8）](http://en.wikipedia.org/wiki/UTF-8#Modified_UTF-8)的方法。這種變形編碼對於C代碼是非常有用的，因為它將\u0000編碼成0xc0 0x80，而不是0x00。最愜意的事情是你能在具有C風格的以\0結束的字符串上計數，同時兼容標準的libc字符串函數。不好的一面是你不能傳入隨意的UTF-8數據到JNI函數而還指望它正常工作。

如果可能的話，直接操作UTF-16字符串通常更快些。安卓當前在調用GetStringChars時不需要拷貝，而GetStringUTFChars需要一次分配並且轉換為UTF-8格式。注意**UTF-16字符串不是以零終止字符串**，\u0000是被允許的，所以你需要像對jchar指針一樣地處理字符串的長度。

**不要忘記Release你Get的字符串**。這些字符串函數返回jchar*或者jbyte*，都是指向基本數據類型的C格式的指針而不是局部引用。它們在Release調用之前都保證有效，這意味著當本地方法返回時它們並不主動釋放。

**傳入NewStringUTF函數的數據必須是變形UTF-8格式**。一種常見的錯誤情況是，從文件或者網絡流中讀取出的字符數據，沒有過濾直接使用NewStringUTF處理。除非你確定數據是7位的ASCII格式，否則你需要剔除超出7位ASCII編碼範圍（high-ASCII）的字符或者將它們轉換為對應的變形UTF-8格式。如果你沒那樣做，UTF-16的轉換結果可能不會是你想要的結果。JNI擴展檢查將會掃描字符串，然後警告你那些無效的數據，但是它們將不會發現所有潛在的風險。

#原生類型數組

JNI提供了一系列函數來訪問數組對象中的內容。對象數組的訪問只能<font color='red'>一次一條</font>，但如果原生類型數組以C方式聲明，則能夠直接進行讀寫。

為了讓接口更有效率而不受VM實現的制約，Get<PrimitiveType>ArrayElements系列調用允許運行時返回一個指向實際元素的指針，或者是分配些內存然後拷貝一份。不論哪種方式，返回的原始指針在相應的Release調用之前都保證有效（這意味著，如果數據沒被拷貝，實際的數組對象將會受到牽制，不能重新成為整理堆空間的一部分）。**你必須釋放（Release）每個你通過Get得到的數組**。同時，如果Get調用失敗，你必須確保你的代碼在之後不會去嘗試調用Release來釋放一個空指針（NULL pointer）。

你可以用一個非空指針作為isCopy參數的值來決定數據是否會被拷貝。這相當有用。

Release類的函數接收一個mode參數，這個參數的值可選的有下面三種。而運行時具體執行的操作取決於它返回的指針是指向真實數據還是拷貝出來的那份。

- 0
    - 真實的：實際數組對象不受到牽制
    - 拷貝的：數據將會複製回去，備份空間將會被釋放。
- JNI_COMMIT
    - 真實的：不做任何操作
    - 拷貝的：數據將會複製回去，備份空間將**不會被釋放**。
- JNI_ABORT
    - 真實的：實際數組對象不受到牽制.之前的寫入**不會**被取消。
    - 拷貝的：備份空間將會被釋放；裡面所有的變更都會丟失。

檢查isCopy標識的一個原因是對一個數組做出變更後確認你是否需要傳入JNI_COMMIT來調用Release函數。如果你交替地執行變更和讀取數組內容的代碼，你也許可以跳過無操作（no-op）的JNI_COMMIT。檢查這個標識的另一個可能的原因是使用JNI_ABORT可以更高效。例如，你也許想得到一個數組，適當地修改它，傳入部分到其他函數中，然後丟掉這些修改。如果你知道JNI是為你做了一份新的拷貝，就沒有必要再創建另一份“可編輯的（editable）”的拷貝了。如果JNI傳給你的是原始數組，這時你就需要創建一份你自己的拷貝了。

另一個常見的錯誤（在示例代碼中出現過）是認為當isCopy是false時你就可以不調用Release。實際上是沒有這種情況的。如果沒有分配備份空間，那麼初始的內存空間會受到牽制，位置不能被垃圾回收器移動。

另外注意JNI_COMMIT標識**沒有**釋放數組，你最終需要使用一個不同的標識再次調用Release。

#區間數組

當你想做的只是拷出或者拷進數據時，可以選擇調用像Get<Type>ArrayElements和GetStringChars這類非常有用的函數。想想下面：

``` JAVA

jbyte* data = env->GetByteArrayElements(array, NULL);
if (data != NULL) {
    memcpy(buffer, data, len);
    env->ReleaseByteArrayElements(array, data, JNI_ABORT);
}
```

這裡獲取到了數組，從當中拷貝出開頭的len個字節元素，然後釋放這個數組。根據代碼的實現，Get函數將會牽制或者拷貝數組的內容。上面的代碼拷貝了數據（為了可能的第二次），然後調用Release；這當中JNI_ABORT確保不存在第三份拷貝了。

另一種更簡單的實現方式：

``` JAVA

env->GetByteArrayRegion(array, 0, len, buffer);
```

這種方式有幾個優點：

- 只需要調用一個JNI函數而是不是兩個，減少了開銷。
- 不需要指針或者額外的拷貝數據。
- 減少了開發人員犯錯的風險-在某些失敗之後忘記調用Release不存在風險。

類似地，你能使用Set<Type>ArrayRegion函數拷貝數據到數組，使用GetStringRegion或者GetStringUTFRegion從String中拷貝字符。

#異常

**當異常發生時你一定不能調用大部分的JNI函數**。你的代碼收到異常（通過函數的返回值，ExceptionCheck，或者ExceptionOccurred），然後返回，或者清除異常，處理掉。

當異常發生時你被允許調用的JNI函數有：

- DeleteGlobalRef
- DeleteLocalRef
- DeleteWeakGlobalRef
- ExceptionCheck
- ExceptionClear
- ExceptionDescribe
- ExceptionOccurred
- MonitorExit
- PopLocalFrame
- PushLocalFrame
- Release<PrimitiveType>ArrayElements
- ReleasePrimitiveArrayCritical
- ReleaseStringChars
- ReleaseStringCritical
- ReleaseStringUTFChars

許多JNI調用能夠拋出異常，但通常提供一種簡單的方式來檢查失敗。例如，如果NewString返回一個非空值，你不需要檢查異常。然而，如果你調用一個方法（使用一個像CalllObjectMethod的函數），你必須一直檢查異常，因為當一個異常拋出時它的返回值將不會是有效的。

注意中斷代碼拋出的異常不會展開本地調用堆棧信息，Android也還不支持C++異常。JNI Throw和ThrowNew指令僅僅是在當前線程中放入一個異常指針。從本地代碼返回到託管代碼時，異常將會被注意到，得到適當的處理。

本地代碼能夠通過調用ExceptionCheck或者ExceptionOccurred捕獲到異常，然後使用ExceptionClear清除掉。通常，拋棄異常而不處理會導致些問題。

沒有內建的函數來處理Throwable對象自身，因此如果你想得到異常字符串，你需要找出Throwable Class，然後查找到getMessage "()Ljava/lang/String;"的方法ID，調用它，如果結果非空，使用GetStringUTFChars，得到的結果你可以傳到printf(3) 或者其它相同功能的函數輸出。

#擴展檢查

JNI的錯誤檢查很少。錯誤發生時通常會導致崩潰。Android也提供了一種模式，叫做CheckJNI，這當中JavaVM和JNIEnv函數表指針被換成了函數表，它在調用標準實現之前執行了一系列擴展檢查的。

額外的檢查包括：

- 數組：試圖分配一個長度為負的數組。
- 壞指針：傳入一個不完整jarray/jclass/jobject/jstring對象到JNI函數，或者調用JNI函數時使用空指針傳入到一個不能為空的參數中去。
- 類名：傳入了除“java/lang/String”之外的類名到JNI函數。
- 關鍵調用：在一個“關鍵的(critical)”get和它對應的release之間做出JNI調用。
- 直接的ByteBuffers：傳入不正確的參數到NewDirectByteBuffer。
- 異常：當一個異常發生時調用了JNI函數。
- JNIEnv*s：在錯誤的線程中使用一個JNIEnv*。
- jfieldIDs：使用一個空jfieldID，或者使用jfieldID設置了一個錯誤類型的值到字段（比如說，試圖將一個StringBuilder賦給String類型的域），或者使用一個靜態字段下的jfieldID設置到一個實例的字段（instance field）反之亦然，或者使用的一個類的jfieldID卻來自另一個類的實例。
- jmethodIDs：當調用Call*Method函數時時使用了類型錯誤的jmethodID：不正確的返回值，靜態/非靜態的不匹配，this的類型錯誤（對於非靜態調用）或者錯誤的類（對於靜態類調用）。
- 引用：在類型錯誤的引用上使用了DeleteGlobalRef/DeleteLocalRef。
- 釋放模式：調用release使用一個不正確的釋放模式（其它非 0，JNI_ABORT，JNI_COMMIT的值）。
- 類型安全：從你的本地代碼中返回了一個不兼容的類型（比如說，從一個聲明返回String的方法卻返回了StringBuilder）。
- UTF-8：傳入一個無效的變形UTF-8字節序列到JNI調用。

（方法和域的可訪問性仍然沒有檢查：訪問限制對於本地代碼並不適用。）

有幾種方法去啟用CheckJNI。

如果你正在使用模擬器，CheckJNI默認是打開的。

如果你有一臺root過的設備，你可以使用下面的命令序列來重啟運行時（runtime），啟用CheckJNI。

``` JAVA

adb shell stop
adb shell setprop dalvik.vm.checkjni true
adb shell start
```

隨便哪一種，當運行時（runtime）啟動時你將會在你的日誌輸出中見到如下的字符：

``` JAVA

D AndroidRuntime: CheckJNI is ON
```

如果你有一臺常規的設備，你可以使用下面的命令：

``` JAVA

adb shell setprop debug.checkjni 1
```

這將不會影響已經在運行的app，但是從那以後啟動的任何app都將打開CheckJNI(改變屬性為其它值或者只是重啟都將會再次關閉CheckJNI)。這種情況下，你將會在下一次app啟動時，在日誌輸出中看到如下字符：

``` JAVA

D Late-enabling CheckJNI
```

#本地庫

你可以使用標準的System.loadLibrary方法來從共享庫中加載本地代碼。在你的本地代碼中較好的做法是：

- 在一個靜態類初始化時調用System.loadLibrary（見之前的一個例子中，當中就使用了nativeClassInit）。參數是“未加修飾（undecorated）”的庫名稱，因此要加載“libfubar.so”，你需要傳入“fubar”。
- 提供一個本地函數：**jint JNI_OnLoad(JavaVM* vm, void* reserved)**
- 在JNI_OnLoad中，註冊所有你的本地方法。你應該聲明方法為“靜態的（static）”因此名稱不會佔據設備上符號表的空間。

JNI_OnLoad函數在C++中的寫法如下：

``` JAVA

jint JNI_OnLoad(JavaVM* vm, void* reserved)
{
    JNIEnv* env;
    if (vm->GetEnv(reinterpret_cast<void**>(&env), JNI_VERSION_1_6) != JNI_OK) {
        return -1;
    }

    // 使用env->FindClass得到jclass
    // 使用env->RegisterNatives註冊本地方法

    return JNI_VERSION_1_6;
}
```

你也可以使用共享庫的全路徑來調用System.load。對於Android app，你也許會發現從context對象中得到應用私有數據存儲的全路徑是非常有用的。

上面是推薦的方式，但不是僅有的實現方式。顯式註冊不是必須的，提供一個JNI_OnLoad函數也不是必須的。你可以使用基於特殊命名的“發現（discovery）”模式來註冊本地方法（更多細節見：[JNI spec](http://java.sun.com/javase/6/docs/technotes/guides/jni/spec/design.html#wp615)），雖然這並不可取。因為如果一個方法的簽名錯誤，在這個方法實際第一次被調用之前你是不會知道的。

關於JNI_OnLoad另一點注意的是：任何你在JNI_OnLoad中對FindClass的調用都發生在用作加載共享庫的類加載器的上下文（context）中。一般FindClass使用與“調用棧”頂部方法相關的加載器，如果當中沒有加載器（因為線程剛剛連接）則使用“系統（system）”類加載器。這就使得JNI_OnLoad成為一個查尋及緩存類引用很便利的地方。

#64位機問題

Android當前設計為運行在32位的平臺上。理論上它也能夠構建為64位的系統，但那不是現在的目標。當與本地代碼交互時，在大多數情況下這不是你需要擔心的，但是如果你打算存儲指針變量到對象的整型字段（integer field）這樣的本地結構中，這就變得非常重要了。為了支持使用64位指針的架構，**你需要使用long類型而不是int類型的字段來存儲你的本地指針**。

#不支持的特性/向後兼容性

除了下面的例外，支持所有的JNI 1.6特性：

- DefineClass沒有實現。Android不使用Java字節碼或者class文件，因此傳入二進制class數據將不會有效。

對Android以前老版本的向後兼容性，你需要注意：

- 本地函數的動態查找
在Android 2.0(Eclair)之前，在搜索方法名稱時，字符“$”不會轉換為對應的“_00024”。要使它正常工作需要使用顯式註冊方式或者將本地方法的聲明移出內部類。
- 分離線程
在Android 2.0(Eclair)之前，使用pthread_key_create析構函數來避免“退出前線程必須分離”檢查是不可行的（運行時(runtime)也使用了一個pthread key析構函數，因此這是一場看誰先被調用的競賽）。
- 全局弱引用
在Android 2.0(Eclair)之前，全局弱引用沒有被實現。如果試圖使用它們，老版本將完全不兼容。你可以使用Android平臺版本號常量來測試系統的支持性。
在Android 4.0 (Ice Cream Sandwich)之前，全局弱引用只能傳給NewLocalRef, NewGlobalRef, 以及DeleteWeakGlobalRef（強烈建議開發者在使用全局弱引用之前都為它們創建強引用hard reference，所以這不應該在所有限制當中）。
從Android 4.0 (Ice Cream Sandwich)起，全局弱引用能夠像其它任何JNI引用一樣使用了。
- 局部引用
在Android 4.0 (Ice Cream Sandwich)之前，局部引用實際上是直接指針。Ice Cream Sandwich為了更好地支持垃圾回收添加了間接指針，但這並不意味著很多JNI bug在老版本上不存在。更多細節見[JNI Local Reference Changes in ICS](http://android-developers.blogspot.com/2011/11/jni-local-reference-changes-in-ics.html)。
- 使用GetObjectRefType獲得引用類型
在Android 4.0 (Ice Cream Sandwich)之前，使用直接指針（見上面）的後果就是正確地實現GetObjectRefType是不可能的。我們可以使用依次檢測全局弱引用表，參數，局部表，全局表的方式來代替。第一次匹配到你的直接指針時，就表明你的引用類型是當前正在檢測的類型。這意味著，例如，如果你在一個全局jclass上使用GetObjectRefType，而這個全局jclass碰巧與作為靜態本地方法的隱式參數傳入的jclass一樣的，你得到的結果是JNILocalRefType而不是JNIGlobalRefType。

#FAQ: 為什麼出現了UnsatisfiedLinkError?

當使用本地代碼開發時經常會見到像下面的錯誤：

``` JAVA

java.lang.UnsatisfiedLinkError: Library foo not found
```

有時候這表示和它提示的一樣---未找到庫。但有些時候庫確實存在但不能被dlopen(3)找開，更多的失敗信息可以參見異常詳細說明。

你遇到“library not found”異常的常見原因可能有這些：

- 庫文件不存在或者不能被app訪問到。使用adb shell ls -l <path>檢查它的存在性和權限。
- 庫文件不是用NDK構建的。這就導致設備上並不存在它所依賴的函數或者庫。

另一種UnsatisfiedLinkError錯誤像下面這樣：

``` JAVA

java.lang.UnsatisfiedLinkError: myfunc
        at Foo.myfunc(Native Method)
        at Foo.main(Foo.java:10)
```

在日誌中，你會發現：

``` JAVA

W/dalvikvm(  880): No implementation found for native LFoo;.myfunc ()V
```

這意味著運行時嘗試匹配一個方法但是沒有成功，這種情況常見的原因有：

- 庫文件沒有得到加載。檢查日誌輸出中關於庫文件加載的信息。
- 由於名稱或者簽名錯誤，方法不能匹配成功。這通常是由於：
    - 對於方法的懶查尋，使用 extern "C"和對應的可見性（JNIEXPORT）來聲明C++函數沒有成功。注意Ice Cream Sandwich之前的版本，JNIEXPORT宏是不正確的，因此對新版本的GCC使用舊的jni.h頭文件將不會有效。你可以使用arm-eabi-nm查看它們出現在庫文件裡的符號。如果它們看上去比較凌亂（像_Z15Java_Foo_myfuncP7_JNIEnvP7_jclass這樣而不是Java_Foo_myfunc），或者符號類型是小寫的“t”而不是一個大寫的“T”,這時你就需要調整聲明瞭。
    - 對於顯式註冊，在進行方法簽名時可能犯了些小錯誤。確保你傳入到註冊函數的簽名能夠完全匹配上日誌文件裡提示的。記住“B”是byte，“Z”是boolean。在簽名中類名組件是以“L”開頭的，以“;”結束的，使用“/”來分隔包名/類名，使用“$”符來分隔內部類名稱（比如說，Ljava/util/Map$Entry;）。

使用javah來自動生成JNI頭文件也許能幫助你避免這些問題。

#FAQ: 為什麼FindClass不能找到我的類?

確保類名字符串有正確的格式。JNI類名稱以包名開始，然後使用左斜槓來分隔，比如java/lang/String。如果你正在查找一個數組類，你需要以對應數目的綜括號開頭，使用“L”和“;”將類名兩頭包起來，所以一個一維字符串數組應該寫成[Ljava/lang/String;。

如果類名稱看上去正確，你可能運行時遇到了類加載器的問題。FindClass想在與你代碼相關的類加載器中開始查找指定的類。檢查調用堆棧，可能看起像：

``` JAVA

Foo.myfunc(Native Method)
Foo.main(Foo.java:10)
dalvik.system.NativeStart.main(Native Method)
```

最頂層的方法是Foo.myfunc。FindClass找到與類Foo相關的ClassLoader對象然後使用它。

這通常正是你所想的。如果你創建了自己的線程那麼就會遇到麻煩（也許是調用了pthread_create然後使用AttachCurrentThread進行了連接）。現在跟蹤堆棧可能像下面這樣：

``` JAVA

dalvik.system.NativeStart.run(Native Method)
```

最頂層的方法是NativeStart.run，它不是你應用內的方法。如果你從這個線程中調用FindClass，JavaVM將會啟動“系統（system）”的而不是與你應用相關的加載器，因此試圖查找應用內定義的類都將會失敗。

下面有幾種方法可以解決這個問題：

- 在JNI_OnLoad中使用FindClass查尋一次，然後為後面的使用緩存這些類引用。任何在JNI_OnLoad當中執行的FindClass調用都使用與執行System.loadLibrary的函數相關的類加載器（這個特例，讓庫的初始化更加的方便了）。如果你的app代碼正在加載庫文件，FindClass將會使用正確的類加載器。
- 傳入類實例到一個需要它的函數，你的本地方法聲明必須帶有一個Class參數，然後傳入Foo.class。
- 在合適的地方緩存一個ClassLoader對象的引用，然後直接發起loadClass調用。這需要額外些工作。

#FAQ: 使用本地代碼怎樣共享原始數據?

也許你會遇到這樣一種情況，想從你的託管代碼或者本地代碼訪問一大塊原始數據的緩衝區。常見例子包括對bitmap或者聲音文件的處理。這裡有兩種基本實現方式。

你可以將數據存儲到byte[]。這允許你從託管代碼中快速地訪問。然而，在本地代碼端不能保證你不去拷貝一份就直接能夠訪問數據。在某些實現中，GetByteArrayElements和GetPrimitiveArrayCritical將會返回指向在維護堆中的原始數據的真實指針，但是在另外一些實現中將在本地堆空間分配一塊緩衝區然後拷貝數據過去。

還有一種選擇是將數據存儲在一塊直接字節緩衝區（direct byte buffer），可以使用java.nio.ByteBuffer.allocateDirect或者NewDirectByteBuffer JNI函數創建buffer。不像常規的byte緩衝區，它的存儲空間將不會分配在程序維護的堆空間上，總是可以從本地代碼直接訪問（使用GetDirectBufferAddress得到地址）。依賴於直接字節緩衝區訪問的實現方式，從託管代碼訪問原始數據將會非常慢。

選擇使用哪種方式取決於兩個方面：

1.大部分的數據訪問是在Java代碼還是C/C++代碼中發生？

2.如果數據最終被傳到系統API，那它必須是怎樣的形式（例如，如果數據最終被傳到一個使用byte[]作為參數的函數，在直接的ByteBuffer中處理或許是不明智的）？

如果通過上面兩種情況仍然不能明確區分的，就使用直接字節緩衝區（direct byte buffer）形式。它們的支持是直接構建到JNI中的，在未來的版本中性能可能會得到提升。

