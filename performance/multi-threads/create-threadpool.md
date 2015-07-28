## # # ### ## # 為多線程創建管理器

> 編寫:[AllenZheng1991](https://github.com/AllenZheng1991) - 原文:<http://developer.android.com/training/multiple-threads/create-threadpool.html>

在前面的課程中展示瞭如何在單獨的一個線程中執行一個任務。如果你的線程只想執行一次，那麼上一課的內容已經能滿足你的需要了。

如果你想在一個數據集中重複執行一個任務，而且你只需要一個執行運行一次。這時，使用一個[IntentService](http://developer.android.com/reference/android/app/IntentService.html)將能滿足你的需求。
為了在資源可用的的時候自動執行任務，或者允許不同的任務同時執行（或前後兩者），你需要提供一個管理線程的集合。
為了做這個管理線程的集合，使用一個[ThreadPoolExecutor](http://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html)實例，當一個線程在它的線程池中變得不受約束時，它會運行隊列中的一個任務。
為了能執行這個任務，你所需要做的就是把它加入到這個隊列。

一個線程池能運行多個並行的任務實例，因此你要能保證你的代碼是線程安全的，從而你需要給會被多個線程訪問的變量附上同步代碼塊(synchronized block)。
當一個線程在對一個變量進行寫操作時，通過這個方法將能阻止另一個線程對該變量進行讀取操作。
典型的，這種情況會發生在靜態變量上，但同樣它也能突然發生在任意一個只實例化一次。
為了學到更多的相關知識，你可以閱讀[進程與線程](http://developer.android.com/guide/components/processes-and-threads.html)這一API指南。

## 定義線程池類

在自己的類中實例化[ThreadPoolExecutor](http://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html)類。在這個類裡需要做以下事：

**1. 為線程池使用靜態變量**

為了有一個單一控制點用來限制CPU或涉及網絡資源的[Runnable](http://developer.android.com/reference/java/lang/Runnable.html)類型，你可能需要有一個能管理所有線程的線程池，且每個線程都會是單個實例。比如，你可以把這個作為一部分添加到你的全局變量的聲明中去：

```java
public class PhotoManager {
    ...
    static  {
        ...
        // Creates a single static instance of PhotoManager
        sInstance = new PhotoManager();
    }
    ...
```

**2. 使用私有構造方法**

讓構造方法私有從而保證這是一個單例，這意味著你不需要在同步代碼塊(synchronized block)中額外訪問這個類：

```java
public class PhotoManager {
    ...
    /**
     * Constructs the work queues and thread pools used to download
     * and decode images. Because the constructor is marked private,
     * it's unavailable to other classes, even in the same package.
     */
    private PhotoManager() {
    ...
    }
```

**3.通過調用線程池類裡的方法開啟你的任務**

在線程池類中定義一個能添加任務到線程池隊列的方法。例如：

```java
public class PhotoManager {
    ...
    // Called by the PhotoView to get a photo
    static public PhotoTask startDownload(
        PhotoView imageView,
        boolean cacheFlag) {
        ...
        // Adds a download task to the thread pool for execution
        sInstance.
                mDownloadThreadPool.
                execute(downloadTask.getHTTPDownloadRunnable());
        ...
    }
```

**4. 在構造方法中實例化一個[Handler](http://developer.android.com/reference/android/os/Handler.html)，且將它附加到你APP的UI線程。**

一個[Handler](http://developer.android.com/reference/android/os/Handler.html)允許你的APP安全地調用UI對象（例如 [View](http://developer.android.com/reference/android/view/View.html)對象）的方法。大多數UI對象只能從UI線程安全的代碼中被修改。這個方法將會在[與UI線程進行通信(Communicate with the UI Thread)](performance/multi-threads/communicate-ui.html)這一課中進行詳細的描述。例如：

```java
private PhotoManager() {
    ...
        // Defines a Handler object that's attached to the UI thread
        mHandler = new Handler(Looper.getMainLooper()) {
            /*
             * handleMessage() defines the operations to perform when
             * the Handler receives a new Message to process.
             */
            @Override
            public void handleMessage(Message inputMessage) {
                ...
            }
        ...
        }
    }
```

## 確定線程池的參數

一旦有了整體的類結構,你可以開始定義線程池了。為了初始化一個[ThreadPoolExecutor](http://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html)對象，你需要提供以下數值：

**1. 線程池的初始化大小和最大的大小**

這個是指最初分配給線程池的線程數量，以及線程池中允許的最大線程數量。在線程池中擁有的線程數量主要取決於你的設備的CPU內核數。

這個數字可以從系統環境中獲得：

```java
public class PhotoManager {
...
    /*
     * Gets the number of available cores
     * (not always the same as the maximum number of cores)
     */
    private static int NUMBER_OF_CORES =
            Runtime.getRuntime().availableProcessors();
}
```

這個數字可能並不反映設備的物理核心數量，因為一些設備根據系統負載關閉了一個或多個CPU內核，對於這樣的設備，`availableProcessors()`方法返回的是處於活動狀態的內核數量，可能少於設備的實際內核總數。

**2.線程保持活動狀態的持續時間和時間單位**

這個是指線程被關閉前保持空閒狀態的持續時間。這個持續時間通過時間單位值進行解譯，是[TimeUnit()](http://developer.android.com/reference/java/util/concurrent/TimeUnit.html)中定義的常量之一。

**3.一個任務隊列**

這個傳入的隊列由[ThreadPoolExecutor](http://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html)獲取的[Runnable](http://developer.android.com/reference/java/lang/Runnable.html)對象組成。為了執行一個線程中的代碼，一個線程池管理者從先進先出的隊列中取出一個[Runnable](http://developer.android.com/reference/java/lang/Runnable.html)對象且把它附加到一個線程。當你創建線程池時需要提供一個隊列對象，這個隊列對象類必須實現[BlockingQueue](http://developer.android.com/reference/java/util/concurrent/BlockingQueue.html)接口。為了滿足你的APP的需求，你可以選擇一個Android SDK中已經存在的隊列實現類。為了學習更多相關的知識，你可以看一下[ThreadPoolExecutor](http://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html)類的概述。下面是一個使用[LinkedBlockingQueue](http://developer.android.com/reference/java/util/concurrent/LinkedBlockingQueue.html)實現的例子：

```java
public class PhotoManager {
    ...
    private PhotoManager() {
        ...
        // A queue of Runnables
        private final BlockingQueue<Runnable> mDecodeWorkQueue;
        ...
        // Instantiates the queue of Runnables as a LinkedBlockingQueue
        mDecodeWorkQueue = new LinkedBlockingQueue<Runnable>();
        ...
    }
    ...
}
```

##創建一個線程池

為了創建一個線程池，可以通過調用<a href="http://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html#ThreadPoolExecutor(int, int, long, java.util.concurrent.TimeUnit, java.util.concurrent.BlockingQueue<java.lang.Runnable>)" target="_blank">ThreadPoolExecutor()</a>構造方法初始化一個線程池管理者對象，這樣就能創建和管理一組可約束的線程了。如果線程池的初始化大小和最大大小相同，[ThreadPoolExecutor](http://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html)在實例化的時候就會創建所有的線程對象。例如：

```java
private PhotoManager() {
        ...
        // Sets the amount of time an idle thread waits before terminating
        private static final int KEEP_ALIVE_TIME = 1;
        // Sets the Time Unit to seconds
        private static final TimeUnit KEEP_ALIVE_TIME_UNIT = TimeUnit.SECONDS;
        // Creates a thread pool manager
        mDecodeThreadPool = new ThreadPoolExecutor(
                NUMBER_OF_CORES,       // Initial pool size
                NUMBER_OF_CORES,       // Max pool size
                KEEP_ALIVE_TIME,
                KEEP_ALIVE_TIME_UNIT,
                mDecodeWorkQueue);
    }
```



