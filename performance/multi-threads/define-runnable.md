# 在一個線程中執行一段特定的代碼

> 編寫:[AllenZheng1991](https://github.com/AllenZheng1991) - 原文:<http://developer.android.com/training/multiple-threads/define-runnable.html>

這一課向你展示瞭如何通過實現 [Runnable](http://developer.android.com/reference/java/lang/Runnable.html)接口得到一個能在重寫的[`Runnable.run()`](http://developer.android.com/reference/java/lang/Runnable.html)方法中執行一段代碼的單獨的線程。另外你可以傳遞一個[Runnable](http://developer.android.com/reference/java/lang/Runnable.html)對象到另一個對象，然後這個對象可以把它附加到一個線程，並執行它。一個或多個執行特定操作的[Runnable](http://developer.android.com/reference/java/lang/Runnable.html)對象有時也被稱為一個任務。

[Thread](http://developer.android.com/reference/java/lang/Runnable.html)和[Runnable](http://developer.android.com/reference/java/lang/Runnable.html)只是兩個基本的線程類，通過他們能發揮的作用有限，但是他們是強大的Android線程類的基礎類，例如Android中的[HandlerThread](http://developer.android.com/reference/android/os/HandlerThread.html), [AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html)和[IntentService](http://developer.android.com/reference/android/app/IntentService.html)都是以它們為基礎。[Thread](http://developer.android.com/reference/java/lang/Runnable.html)和[Runnable](http://developer.android.com/reference/java/lang/Runnable.html)同時也是[ThreadPoolExecutor](http://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html)類的基礎。[ThreadPoolExecutor](http://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html)類能自動管理線程和任務隊列，甚至可以並行執行多個線程。

##定義一個實現Runnable接口的類

直接了當的方法是通過實現[Runnable](http://developer.android.com/reference/java/lang/Runnable.html)接口去定義一個線程類。例如：

```java
public class PhotoDecodeRunnable implements Runnable {
    ...
    @Override
    public void run() {
        /*
         * 把你想要在線程中執行的代碼寫在這裡
         */
        ...
    }
    ...
}
```

##實現run()方法

在一個類裡，[`Runnable.run()`](http://developer.android.com/reference/java/lang/Runnable.html)
包含執行了的代碼。通常在[Runnable](http://developer.android.com/reference/java/lang/Runnable.html)
中執行任何操作都是可以的，但需要記住的是，因為[Runnable](http://developer.android.com/reference/java/lang/Runnable.html)
不會在UI線程中運行，所以它不能直接更新UI對象，例如[View](http://developer.android.com/reference/android/view/View.html)
對象。為了與UI對象進行通信，你必須使用另一項技術，在[與UI線程進行通信](performance/multi-threads/communicate-ui.html)
這一課中我們會對其進行描述。

在[Runnable.run()](http://developer.android.com/reference/java/lang/Runnable.html#run())方法的開始的地方通過調用參數為[THREAD_PRIORITY_BACKGROUND](http://developer.android.com/reference/android/os/Process.html#THREAD_PRIORITY_BACKGROUND")
的<a href="http://developer.android.com/reference/android/os/Process.html#setThreadPriority(int)" target="_blank">Process.setThreadPriority()</a>方法來設置線程使用的是後臺運行優先級。
這個方法減少了通過<a href="http://developer.android.com/reference/java/lang/Runnable.html" target="_blank">Runnable</a>創建的線程和和UI線程之間的資源競爭。

**你還應該通過在Runnable</a>
自身中調用<a href="http://developer.android.com/reference/java/lang/Thread.html#currentThread()">Thread.currentThread()</a>來存儲一個引用到Runnable對象的線程。**

下面這段代碼展示瞭如何創建run()方法：

```java
class PhotoDecodeRunnable implements Runnable {
...
    /*
     * 定義要在這個任務中執行的代碼
     */
    @Override
    public void run() {
        // 把當前的線程變成後臺執行的線程
        android.os.Process.setThreadPriority(android.os.Process.THREAD_PRIORITY_BACKGROUND);
        ...
        /*
         * 在PhotoTask實例中存儲當前線程，以至於這個實例能中斷這個線程
         */
        mPhotoTask.setImageDecodeThread(Thread.currentThread());
        ...
    }
...
}
```

