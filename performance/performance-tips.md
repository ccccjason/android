# 代碼性能優化建議

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/articles/perf-tips.html>

這篇文章主要介紹一些小細節的優化技巧，雖然這些小技巧不能較大幅度的提升應用性能，但是恰當的運用這些小技巧併發生累積效應的時候，對於整個App的性能提升還是有不小作用的。通常來說，選擇合適的算法與數據結構會是你首要考慮的因素，在這篇文章中不會涉及這方面的知識點。你應該使用這篇文章中的小技巧作為平時寫代碼的習慣，這樣能夠提升代碼的效率。

通常來說，高效的代碼需要滿足下面兩個原則：

* 不要做冗餘的工作
* 儘量避免執行過多的內存分配操作

在優化App時其中一個難點就是讓App能在各種型號的設備上運行。不同版本的虛擬機在不同的處理器上會有不同的運行速度。你甚至不能簡單的認為“設備X的速度是設備Y的F倍”，然後還用這種倍數關係去推測其他設備。另外，在模擬器上的運行速度和在實際設備上的速度沒有半點關係。同樣，設備有沒有JIT也對運行速度有重大影響：在有JIT情況下的最優化代碼不一定在沒有JIT的情況下也是最優的。

為了確保App在各設備上都能良好運行，就要確保你的代碼在不同檔次的設備上都儘可能的優化。

## 避免創建不必要的對象

創建對象從來不是免費的。**Generational GC**可以使臨時對象的分配變得廉價一些，但是執行分配內存總是比不執行分配操作更昂貴。

隨著你在App中分配更多的對象，你可能需要強制gc，而gc操作會給用戶體驗帶來一點點卡頓。雖然從Android 2.3開始，引入了併發gc，它可以幫助你顯著提升gc的效率，減輕卡頓，但畢竟不必要的內存分配操作還是應該儘量避免。

因此請儘量避免創建不必要的對象，有下面一些例子來說明這個問題：

* 如果你需要返回一個String對象，並且你知道它最終會需要連接到一個`StringBuffer`，請修改你的函數實現方式，避免直接進行連接操作，應該採用創建一個臨時對象來做字符串的拼接這個操作。
* 當從已經存在的數據集中抽取出String的時候，嘗試返回原數據的substring對象，而不是創建一個重複的對象。使用substring的方式，你將會得到一個新的String對象，但是這個string對象是和原string共享內部`char[]`空間的。

一個稍微激進點的做法是把所有多維的數據分解成一維的數組:

* 一組int數據要比一組Integer對象要好很多。可以得知，兩組一維數組要比一個二維數組更加的有效率。同樣的，這個道理可以推廣至其他原始數據類型。
* 如果你需要實現一個數組用來存放(Foo,Bar)的對象，記住使用Foo[]與Bar[]要比(Foo,Bar)好很多。(例外的是，為了某些好的API的設計，可以適當做一些妥協。但是在自己的代碼內部，你應該多多使用分解後的容易）。

通常來說，需要避免創建更多的臨時對象。更少的對象意味者更少的gc動作，gc會對用戶體驗有比較直接的影響。

## 選擇Static而不是Virtual

如果你不需要訪問一個對象的值，請保證這個方法是static類型的，這樣方法調用將快15%-20%。這是一個好的習慣，因為你可以從方法聲明中得知調用無法改變這個對象的狀態。

## 常量聲明為Static Final

考慮下面這種聲明的方式

```java
static int intVal = 42;
static String strVal = "Hello, world!";
```

編譯器會使用一個初始化類的函數<clinit>，然後當類第一次被使用的時候執行。這個函數將42存入`intVal`，還從class文件的常量表中提取了`strVal`的引用。當之後使用`intVal`或`strVal`的時候，他們會直接被查詢到。

我們可以用`final`關鍵字來優化：

```java
static final int intVal = 42;
static final String strVal = "Hello, world!";
```

這時再也不需要上面的<clinit>方法了，因為final聲明的常量進入了靜態dex文件的域初始化部分。調用`intVal`的代碼會直接使用42，調用`strVal`的代碼也會使用一個相對廉價的“字符串常量”指令，而不是查表。

> **Notes：**這個優化方法只對原始類型和String類型有效，而不是任意引用類型。不過，在必要時使用`static final`是個很好的習慣。

## 避免內部的Getters/Setters

像C++等native language，通常使用getters(i = getCount())而不是直接訪問變量(i = mCount)。這是編寫C++的一種優秀習慣，而且通常也被其他面向對象的語言所採用，例如C#與Java，因為編譯器通常會做inline訪問，而且你需要限制或者調試變量，你可以在任何時候在getter/setter裡面添加代碼。

然而，在Android上，這不是一個好的寫法。虛函數的調用比起直接訪問變量要耗費更多。在面向對象編程中，將getter和setting暴露給公用接口是合理的，但在類內部應該僅僅使用域直接訪問。

在沒有JIT(Just In Time Compiler)時，直接訪問變量的速度是調用getter的3倍。有JIT時，直接訪問變量的速度是通過getter訪問的7倍。

請注意，如果你使用[ProGuard](http://developer.android.com/tools/help/proguard.html)，你可以獲得同樣的效果，因為ProGuard可以為你inline accessors.

## 使用增強的For循環

增強的For循環（也被稱為 for-each 循環）可以被用在實現了 Iterable 接口的 collections 以及數組上。使用collection的時候，Iterator會被分配，用於for-each調用`hasNext()`和`next()`方法。使用ArrayList時，手寫的計數式for循環會快3倍（不管有沒有JIT），但是對於其他collection，增強的for-each循環寫法會和迭代器寫法的效率一樣。

請比較下面三種循環的方法：

```java
static class Foo {
    int mSplat;
}

Foo[] mArray = ...

public void zero() {
    int sum = 0;
    for (int i = 0; i < mArray.length; ++i) {
        sum += mArray[i].mSplat;
    }
}

public void one() {
    int sum = 0;
    Foo[] localArray = mArray;
    int len = localArray.length;

    for (int i = 0; i < len; ++i) {
        sum += localArray[i].mSplat;
    }
}

public void two() {
    int sum = 0;
    for (Foo a : mArray) {
        sum += a.mSplat;
    }
}
```

* zero()是最慢的，因為JIT沒有辦法對它進行優化。
* one()稍微快些。
* two() 在沒有做JIT時是最快的，可是如果經過JIT之後，與方法one()是差不多一樣快的。它使用了增強的循環方法for-each。

所以請儘量使用for-each的方法，但是對於ArrayList，請使用方法one()。

> **Tips：**你還可以參考 Josh Bloch 的 《Effective Java》這本書的第46條

## 使用包級訪問而不是內部類的私有訪問

參考下面一段代碼

```java
public class Foo {
    private class Inner {
        void stuff() {
            Foo.this.doStuff(Foo.this.mValue);
        }
    }

    private int mValue;

    public void run() {
        Inner in = new Inner();
        mValue = 27;
        in.stuff();
    }

    private void doStuff(int value) {
        System.out.println("Value is " + value);
    }
}
```

這裡重要的是，我們定義了一個私有的內部類（`Foo$Inner`），它直接訪問了外部類中的私有方法以及私有成員對象。這是合法的，這段代碼也會如同預期一樣打印出"Value is 27"。

問題是，VM因為`Foo`和`Foo$Inner`是不同的類，會認為在`Foo$Inner`中直接訪問`Foo`類的私有成員是不合法的。即使Java語言允許內部類訪問外部類的私有成員。為了去除這種差異，編譯器會產生一些仿造函數：

```java
/*package*/ static int Foo.access$100(Foo foo) {
    return foo.mValue;
}
/*package*/ static void Foo.access$200(Foo foo, int value) {
    foo.doStuff(value);
}
```

每當內部類需要訪問外部類中的mValue成員或需要調用doStuff()函數時，它都會調用這些靜態方法。這意味著，上面的代碼可以歸結為，通過accessor函數來訪問成員變量。早些時候我們說過，通過accessor會比直接訪問域要慢。所以，這是一個特定語言用法造成性能降低的例子。

如果你正在性能熱區（hotspot:高頻率、重複執行的代碼段）使用像這樣的代碼，你可以把內部類需要訪問的域和方法聲明為包級訪問，而不是私有訪問權限。不幸的是，這意味著在相同包中的其他類也可以直接訪問這些域，所以在公開的API中你不能這樣做。

## 避免使用float類型

Android系統中float類型的數據存取速度是int類型的一半，儘量優先採用int類型。

就速度而言，現代硬件上，float 和 double 的速度是一樣的。空間而言，double 是兩倍float的大小。在空間不是問題的情況下，你應該使用 double 。

同樣，對於整型，有些處理器實現了硬件幾倍的乘法，但是沒有除法。這時，整型的除法和取餘是在軟件內部實現的，這在你使用哈希表或大量計算操作時要考慮到。

## 使用庫函數

除了那些常見的讓你多使用自帶庫函數的理由以外，記得系統函數有時可以替代第三方庫，並且還有彙編級別的優化，他們通常比帶有JIT的Java編譯出來的代碼更高效。典型的例子是：Android API 中的 `String.indexOf()`，Dalvik出於內聯性能考慮將其替換。同樣 `System.arraycopy()`函數也被替換，這樣的性能在Nexus One測試，比手寫的for循環並使用JIT還快9倍。

> **Tips：**參見 Josh Bloch 的 《Effective Java》這本書的第47條

## 謹慎使用native函數

結合Android NDK使用native代碼開發，並不總是比Java直接開發的效率更好的。Java轉native代碼是有代價的，而且JIT不能在這種情況下做優化。如果你在native代碼中分配資源（比如native堆上的內存，文件描述符等等），這會對收集這些資源造成巨大的困難。你同時也需要為各種架構重新編譯代碼（而不是依賴JIT）。你甚至對已同樣架構的設備都需要編譯多個版本：為G1的ARM架構編譯的版本不能完全使用Nexus One上ARM架構的優勢，反之亦然。

Native 代碼是在你已經有本地代碼，想把它移植到Android平臺時有優勢，而不是為了優化已有的Android Java代碼使用。

如果你要使用JNI,請學習[JNI Tips](http://developer.android.com/guide/practices/jni.html)

> **Tips：**參見 Josh Bloch 的 《Effective Java》這本書的第54條

## 關於性能的誤區

在沒有JIT的設備上，使用一種確切的數據類型確實要比抽象的數據類型速度要更有效率（例如，調用`HashMap map`要比調用`Map map`效率更高）。有誤傳效率要高一倍，實際上只是6%左右。而且，在JIT之後，他們直接並沒有大多差異。

在沒有JIT的設備上，讀取緩存域比直接讀取實際數據大概快20%。有JIT時，域讀取和本地讀取基本無差。所以優化並不值得除非你覺得能讓你的代碼更易讀（這對 final, static, static final 域同樣適用）。

## 關於測量

在優化之前，你應該確定你遇到了性能問題。你應該確保你能夠準確測量出現在的性能，否則你也不會知道優化是否真的有效。

本章節中所有的技巧都需要Benchmark（基準測試）的支持。Benchmark可以在 [code.google.com "dalvik" project](http://code.google.com/p/dalvik/source/browse/#svn/trunk/benchmarks) 中找到

Benchmark是基於Java版本的 [Caliper](http://code.google.com/p/caliper/) microbenchmarking框架開發的。Microbenchmarking很難做準確，所以Caliper幫你完成這部分工作，甚至還幫你測了你沒想到需要測量的部分（因為，VM幫你管理了代碼優化，你很難知道這部分優化有多大效果）。我們強烈推薦使用Caliper來做你的基準微測工作。

我們也可以用[Traceview](http://developer.android.com/tools/debugging/debugging-tracing.html) 來測量，但是測量的數據是沒有經過JIT優化的，所以實際的效果應該是要比測量的數據稍微好些。

關於如何測量與調試，還可以參考下面兩篇文章：

* [Profiling with Traceview and dmtracedump](http://developer.android.com/tools/debugging/debugging-tracing.html)
* [Analysing Display and Performance with Systrace](http://developer.android.com/tools/debugging/systrace.html)
