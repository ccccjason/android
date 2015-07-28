# SMP(Symmetric Multi-Processor) Primer for Android

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/articles/smp.html>

從Android 3.0開始，系統針對多核CPU架構的機器做了優化支持。這份文檔介紹了針對多核系統應該如何編寫C，C++以及Java程序。這裡只是作為Android應用開發者的入門教程，並不會深入討論這個話題，並且我們會把討論範圍集中在ARM架構的CPU上。

如果你並沒有時間學習整篇文章，你可以跳過前面的理論部分，直接查看實踐部分。但是我們並不建議這樣做。

## 0)簡要介紹

**SMP** 的全稱是“**Symmetric Multi-Processor**”。 它表示的是一種雙核或者多核CPU的設計架構。在幾年前，所有的Android設備都還是單核的。

大多數的Android設備已經有了多個CPU，但是通常來說，其中一個CPU負責執行程序，其他的CPU則處理設備硬件的相關事務（例如，音頻）。這些CPU可能有著不同的架構，運行在上面的程序無法在內存中彼此進行溝通交互。

目前大多數售賣的Android設備都是SMP架構的，這使得軟件開發者處理問題更加複雜。對於多線程的程序，如果多個線程執行在不同的內核上，這會使得程序更加容易發生**race conditions**。 更糟糕的是，基於ARM架構的SMP比起x86架構來說，更加複雜，更難進行處理。那些在x86上測試通過的程序可能會在ARM上崩潰。

下面我們會介紹為何會這樣以及如何做才能夠使得你的代碼行為正常。

## 1)理論篇

這裡會快速並且簡要的介紹這個複雜的主題。其中一些部分並不完整，但是並沒有出現錯誤或者誤導。

查看文章末尾的[**進一步閱讀**]()可以瞭解這個主題的更多知識。

### 1.1)內存一致性模型(Memory consistency models)

內存一致性模型(Memory consistency models)通常也被叫做“memory models”，描述了硬件架構如何確保內存訪問的一致性。例如，如果你對地址A進行了一個賦值，然後對地址B也進行了賦值，那麼內存一致性模型就需要確保每一個CPU都需要知道剛才的操作賦值與操作順序。

這個模型通常被程序員稱為：**順序一致性(sequential consistency)**, 請從文章末尾的**進一步閱讀**查看**Adve & Gharachorloo**這篇文章。

* 所有的內存操作每次只能執行一個。
* 所有的操作，在單核CPU上，都是順序執行的。

如果你關注一段代碼在內存中的讀寫操作，在sequentially-consistent的CPU架構上，是按照期待的順序執行的。It’s possible that the CPU is actually reordering instructions and delaying reads and writes, but there is no way for code running on the device to tell that the CPU is doing anything other than execute instructions in a straightforward manner. (We’re ignoring memory-mapped device driver I/O for the moment.)

To illustrate these points it’s useful to consider small snippets of code, commonly referred to as litmus tests. These are assumed to execute in program order, that is, the order in which the instructions appear here is the order in which the CPU will execute them. We don’t want to consider instruction reordering performed by compilers just yet.

Here’s a simple example, with code running on two threads:

Thread 1	Thread 2
A = 3
B = 5	reg0 = B
reg1 = A

| Thread 1 | Thread 2 |
| -- | -- |
| A = 3 B = 5 | reg0 = B reg1 = A |

In this and all future litmus examples, memory locations are represented by capital letters (A, B, C) and CPU registers start with “reg”. All memory is initially zero. Instructions are executed from top to bottom. Here, thread 1 stores the value 3 at location A, and then the value 5 at location B. Thread 2 loads the value from location B into reg0, and then loads the value from location A into reg1. (Note that we’re writing in one order and reading in another.)

Thread 1 and thread 2 are assumed to execute on different CPU cores. You should always make this assumption when thinking about multi-threaded code.

Sequential consistency guarantees that, after both threads have finished executing, the registers will be in one of the following states:

| Registers	| States |
| -- | -- |
| reg0=5, reg1=3	| possible (thread 1 ran first) |
| reg0=0, reg1=0	| possible (thread 2 ran first) |
| reg0=0, reg1=3	| possible (concurrent execution) |
| reg0=5, reg1=0	| never |

To get into a situation where we see B=5 before we see the store to A, either the reads or the writes would have to happen out of order. On a sequentially-consistent machine, that can’t happen.

Most uni-processors, including x86 and ARM, are sequentially consistent. Most SMP systems, including x86 and ARM, are not.

#### 1.1.1)Processor consistency
#### 1.1.2)CPU cache behavior
#### 1.1.3)Observability
#### 1.1.4)ARM’s weak ordering

### 1.2)Data memory barriers

#### 1.2.1)Store/store and load/load
#### 1.2.2)Load/store and store/load
#### 1.2.3)Barrier instructions
#### 1.2.4)Address dependencies and causal consistency
#### 1.2.5)Memory barrier summary

### 1.3)Atomic operations

#### 1.3.1)Atomic essentials
#### 1.3.2)Atomic + barrier pairing
#### 1.3.3)Acquire and release


## 2)實踐篇

調試內存一致性(memory consistency)的問題非常困難。如果內存柵欄(memory barrier)導致一些代碼讀取到陳舊的數據，你將無法通過調試器檢查內存dumps文件來找出原因。By the time you can issue a debugger query, the CPU cores will have all observed the full set of accesses, and the contents of memory and the CPU registers will appear to be in an “impossible” state.

### 2.1)What not to do in C
#### 2.1.1)C/C++ and “volatile”
#### 2.1.2)Examples

### 2.2)在Java中不應該做的事

我們沒有討論過Java語言的一些相關特性，因此我們首先來簡要的看下那些特性。

#### 2.2.1)Java中的"synchronized"與"volatile"關鍵字

**“synchronized”**關鍵字提供了Java一種內置的鎖機制。每一個對象都有一個相對應的“monitor”，這個監聽器可以提供互斥的訪問。

“synchronized”代碼段的實現機制與自旋鎖(spin lock)有著相同的基礎結構: 他們都是從獲取到CAS開始，以釋放CAS結束。這意味著編譯器(compilers)與代碼優化器(code optimizers)可以輕鬆的遷移代碼到“synchronized”代碼段中。一個實踐結果是：你**不能**判定synchronized代碼段是執行在這段代碼下面一部分的前面，還是這段代碼上面一部分的後面。更進一步，如果一個方法有兩個synchronized代碼段並且鎖住的是同一個對象，那麼在這兩個操作的中間代碼都無法被其他的線程所檢測到，編譯器可能會執行“鎖粗化lock coarsening”並且把這兩者綁定到同一個代碼塊上。

另外一個相關的關鍵字是**“volatile”**。在Java 1.4以及之前的文檔中是這樣定義的：volatile聲明和對應的C語言中的一樣可不靠。從Java 1.5開始，提供了更有力的保障，甚至和synchronization一樣具備強同步的機制。

volatile的訪問效果可以用下面這個例子來說明。如果線程1給volatile字段做了賦值操作，線程2緊接著讀取那個字段的值，那麼線程2是被確保能夠查看到之前線程1的任何寫操作。更通常的情況是，**任何**線程對那個字段的寫操作對於線程2來說都是可見的。實際上，寫volatile就像是釋放件監聽器，讀volatile就像是獲取監聽器。

非volatile的訪問有可能因為照顧volatile的訪問而需要做順序的調整。例如編譯器可能會往上移動一個非volatile加載操作，但是不會往下移動。Volatile之間的訪問不會因為彼此而做出順序的調整。虛擬機會注意處理如何的內存柵欄(memory barriers)。

當加載與保存大多數的基礎數據類型，他們都是原子的atomic, 對於long以及double類型的數據則不具備原子型，除非他們被聲明為volatile。即使是在單核處理器上，併發多線程更新非volatile字段值也還是不確定的。

#### 2.2.2)Examples

下面是一個錯誤實現的單步計數器(monotonic counter)的示例: ([Java theory and practice: Managing volatility](smp.html#more)).

```java
class Counter {
    private int mValue;

    public int get() {
        return mValue;
    }
    public void incr() {
        mValue++;
    }
}
```

假設get()與incr()方法是被多線程調用的。然後我們想確保當get()方法被調用時，每一個線程都能夠看到當前的數量。最引人注目的問題是mValue++實際上包含了下面三個操作。

1. reg = mValue
2. reg = reg + 1
3. mValue = reg

如果兩個線程同時在執行`incr()`方法，其中的一個更新操作會丟失。為了確保正確的執行`++`的操作，我們需要把`incr()`方法聲明為“synchronized”。這樣修改之後，這段代碼才能夠在單核多線程的環境中正確的執行。

然而，在SMP的系統下還是會執行失敗。不同的線程通過`get()`方法獲取到得值可能是不一樣的。因為我們是使用通常的加載方式來讀取這個值的。我們可以通過聲明`get()`方法為synchronized的方式來修正這個錯誤。通過這些修改，這樣的代碼才是正確的了。

不幸的是，我們有介紹過有可能發生的鎖競爭(lock contention)，這有可能會傷害到程序的性能。除了聲明`get()`方法為synchronized之外，我們可以聲明`mValue`為**“volatile”**. (請注意`incr()`必須使用synchronize) 現在我們知道volatile的mValue的寫操作對於後續的讀操作都是可見的。`incr()`將會稍稍有點變慢，但是`get()`方法將會變得更加快速。因此讀操作多於寫操作時，這會是一個比較好的方案。(請參考AtomicInteger.)

下面是另外一個示例，和之前的C示例有點類似：

```java
class MyGoodies {
    public int x, y;
}
class MyClass {
    static MyGoodies sGoodies;

    void initGoodies() {    // runs in thread 1
        MyGoodies goods = new MyGoodies();
        goods.x = 5;
        goods.y = 10;
        sGoodies = goods;
    }

    void useGoodies() {    // runs in thread 2
        if (sGoodies != null) {
            int i = sGoodies.x;    // could be 5 or 0
            ....
        }
    }
}
```

這段代碼同樣存在著問題，`sGoodies = goods`的賦值操作有可能在`goods`成員變量賦值之前被察覺到。如果你使用`volatile`聲明`sGoodies`變量，你可以認為load操作為`atomic_acquire_load()`，並且把store操作認為是`atomic_release_store()`。

(請注意僅僅是`sGoodies`的引用本身為`volatile`，訪問它的內部字段並不是這樣的。賦值語句`z = sGoodies.x`會執行一個volatile load  MyClass.sGoodies的操作，其後會伴隨一個non-volatile的load操作：：`sGoodies.x`。如果你設置了一個本地引用`MyGoodies localGoods = sGoodies, z = localGoods.x`，這將不會執行任何volatile loads.)

另外一個在Java程序中更加常用的範式就是臭名昭著的**“double-checked locking”**:

```java
class MyClass {
    private Helper helper = null;

    public Helper getHelper() {
        if (helper == null) {
            synchronized (this) {
                if (helper == null) {
                    helper = new Helper();
                }
            }
        }
        return helper;
    }
}
```

上面的寫法是為了獲得一個MyClass的單例。我們只需要創建一次這個實例，通過`getHelper()`這個方法。為了避免兩個線程會同時創建這個實例。我們需要對創建的操作加synchronize機制。然而，我們不想要為了每次執行這段代碼的時候都為“synchronized”付出額外的代價，因此我們僅僅在helper對象為空的時候加鎖。

在單核系統上，這是不能正常工作的。JIT編譯器會破壞這件事情。請查看[4)Appendix](#appendix)的“‘Double Checked Locking is Broken’ Declaration”獲取更多的信息, 或者是Josh Bloch’s Effective Java書中的Item 71 (“Use lazy initialization judiciously”)。

在SMP系統上執行這段代碼，引入了一個額外的方式會導致失敗。把上面那段代碼換成C的語言實現如下：

```c
if (helper == null) {
    // acquire monitor using spinlock
    while (atomic_acquire_cas(&this.lock, 0, 1) != success)
        ;
    if (helper == null) {
        newHelper = malloc(sizeof(Helper));
        newHelper->x = 5;
        newHelper->y = 10;
        helper = newHelper;
    }
    atomic_release_store(&this.lock, 0);
}
```

此時問題就更加明顯了: `helper`的store操作發生在memory barrier之前，這意味著其他的線程能夠在store x/y之前觀察到非空的值。

你應該嘗試確保store helper執行在`atomic_release_store()`方法之後。通過重新排序代碼進行加鎖，但是這是無效的，因為往上移動的代碼，編譯器可以把它移動回原來的位置：在`atomic_release_store()`前面。
(*這裡沒有讀懂，下次再回讀*)

有2個方法可以解決這個問題：

* 刪除外層的檢查。這確保了我們不會在synchronized代碼段之外做任何的檢查。
* 聲明helper為volatile。僅僅這樣一個小小的修改，在前面示例中的代碼就能夠在Java 1.5及其以後的版本中正常工作。

下面的示例演示了使用volatile的2各重要問題：

```java
class MyClass {
    int data1, data2;
    volatile int vol1, vol2;

    void setValues() {    // runs in thread 1
        data1 = 1;
        vol1 = 2;
        data2 = 3;
    }

    void useValues1() {    // runs in thread 2
        if (vol1 == 2) {
            int l1 = data1;    // okay
            int l2 = data2;    // wrong
        }
    }
    void useValues2() {    // runs in thread 2
        int dummy = vol2;
        int l1 = data1;    // wrong
        int l2 = data2;    // wrong
    }
```

請注意`useValues1()`，如果thread 2還沒有察覺到`vol1`的更新操作，那麼它也無法知道`data1`或者`data2`被設置的操作。一旦它觀察到了`vol1`的更新操作，那麼它也能夠知道data1的更新操作。然而，對於`data2`則無法做任何猜測，因為store操作是在volatile store之後發生的。

`useValues2()`使用了第2個volatile字段：vol2，這會強制VM生成一個memory barrier。這通常不會發生。為了建立一個恰當的“happens-before”關係，2個線程都需要使用同一個volatile字段。在thread 1中你需要知道vol2是在data1/data2之後被設置的。(The fact that this doesn’t work is probably obvious from looking at the code; the caution here is against trying to cleverly “cause” a memory barrier instead of creating an ordered series of accesses.)


### 2.3)What to do
#### 2.3.1)General advice
在C/C++中，使用`pthread`操作，例如mutexes與semaphores。他們會使用合適的memory barriers，在所有的Android平臺上提供正確有效的行為。請確保正確這些技術，例如在沒有獲得對應的mutex的情況下賦值操作需要很謹慎。

避免直接使用atomic方法。如果locking與unlocking之間沒有競爭，locking與unlocking一個pthread mutex 分別需要一個單獨的atomic操作。如果你需要一個lock-free的設計，你必須在開始寫代碼之前瞭解整篇文檔的要點。（或者是尋找一個已經為SMP ARM設計好的庫文件）。

Be extremely circumspect with "volatile” in C/C++. It often indicates a concurrency problem waiting to happen.

In Java, the best answer is usually to use an appropriate utility class from the java.util.concurrent package. The code is well written and well tested on SMP.

Perhaps the safest thing you can do is make your class immutable. Objects from classes like String and Integer hold data that cannot be changed once the class is created, avoiding all synchronization issues. The book Effective Java, 2nd Ed. has specific instructions in “Item 15: Minimize Mutability”. Note in particular the importance of declaring fields “final" (Bloch).

If neither of these options is viable, the Java “synchronized” statement should be used to guard any field that can be accessed by more than one thread. If mutexes won’t work for your situation, you should declare shared fields “volatile”, but you must take great care to understand the interactions between threads. The volatile declaration won’t save you from common concurrent programming mistakes, but it will help you avoid the mysterious failures associated with optimizing compilers and SMP mishaps.

The Java Memory Model guarantees that assignments to final fields are visible to all threads once the constructor has finished — this is what ensures proper synchronization of fields in immutable classes. This guarantee does not hold if a partially-constructed object is allowed to become visible to other threads. It is necessary to follow safe construction practices.(Safe Construction Techniques in Java).

#### 2.3.2)Synchronization primitive guarantees
The pthread library and VM make a couple of useful guarantees: all accesses previously performed by a thread that creates a new thread are observable by that new thread as soon as it starts, and all accesses performed by a thread that is exiting are observable when a join() on that thread returns. This means you don’t need any additional synchronization when preparing data for a new thread or examining the results of a joined thread.

Whether or not these guarantees apply to interactions with pooled threads depends on the thread pool implementation.

In C/C++, the pthread library guarantees that any accesses made by a thread before it unlocks a mutex will be observable by another thread after it locks that same mutex. It also guarantees that any accesses made before calling signal() or broadcast() on a condition variable will be observable by the woken thread.

Java language threads and monitors make similar guarantees for the comparable operations.

#### 2.3.3)Upcoming changes to C/C++
The C and C++ language standards are evolving to include a sophisticated collection of atomic operations. A full matrix of calls for common data types is defined, with selectable memory barrier semantics (choose from relaxed, consume, acquire, release, acq_rel, seq_cst).

See the Further Reading section for pointers to the specifications.

## 3)Closing Notes
While this document does more than merely scratch the surface, it doesn’t manage more than a shallow gouge. This is a very broad and deep topic. Some areas for further exploration:

* Learn the definitions of **happens-before**, **synchronizes-with**, and other essential concepts from the Java Memory Model. (It’s hard to understand what “volatile” really means without getting into this.)
* Explore what compilers are and aren’t allowed to do when reordering code. (The JSR-133 spec has some great examples of legal transformations that lead to unexpected results.)
* Find out how to write immutable classes in Java and C++. (There’s more to it than just “don’t change anything after construction”.)
* Internalize the recommendations in the Concurrency section of **Effective Java, 2nd Edition**. (For example, you should avoid calling methods that are meant to be overridden while inside a synchronized block.)
* Understand what sorts of barriers you can use on x86 and ARM. (And other CPUs for that matter, for example Itanium’s acquire/release instruction modifiers.)
* Read through the **java.util.concurrent** and **java.util.concurrent.atomic** APIs to see what's available.
* Consider using concurrency annotations like `@ThreadSafe` and `@GuardedBy` (from net.jcip.annotations).

The Further Reading section in the appendix has links to documents and web sites that will better illuminate these topics.

<a name="appendix"></a>
## 4)Appendix
### 4.1)SMP failure example
### 4.2)Implementing synchronization stores

<a name="more"></a>
### 4.3)Further reading
