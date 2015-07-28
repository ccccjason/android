# 啟動與停止線程池中的線程

> 編寫:[AllenZheng1991](https://github.com/AllenZheng1991) - 原文:<http://developer.android.com/training/multiple-threads/run-code.html>

在前面的課程中向你展示瞭如何去定義一個可以管理線程池且能在他們中執行任務代碼的類。在這一課中我們將向你展示如何在線程池中執行任務代碼。為了達到這個目的，你需要把任務添加到線程池的工作隊列中去，當一個線程變成可運行狀態時，ThreadPoolExecutor從工作隊列中取出一個任務，然後在該線程中執行。

這節課同時也向你展示瞭如何去停止一個正在執行的任務，這個任務可能在剛開始執行時是你想要的，但後來發現它所做的工作並不是你所需要的。你可以取消線程正在執行的任務，而不是浪費處理器的運行時間。例如你正在從網絡上下載圖片且對下載的圖片進行了緩存，當檢測到正在下載的圖片在緩存中已經存在時，你可能希望停止這個下載任務。當然，這取決於你編寫APP的方式，因為可能壓在你啟動下載任務之前無法獲知是否需要啟動這個任務。

## 啟動線程池中的線程執行任務

為了在一個特定的線程池的線程裡開啟一個任務，可以通過調用ThreadPoolExecutor.execute()，它需要提供一個Runnable類型的參數，這個調用會把該任務添加到這個線程池中的工作隊列。當一個空閒的線程進入可執行狀態時，線程管理者從工作隊列中取出等待時間最長的那個任務，並且在線程中執行它。

```java
public class PhotoManager {
    public void handleState(PhotoTask photoTask, int state) {
        switch (state) {
            // The task finished downloading the image
            case DOWNLOAD_COMPLETE:
            // Decodes the image
                mDecodeThreadPool.execute(
                        photoTask.getPhotoDecodeRunnable());
            ...
        }
        ...
    }
    ...
}
```

當ThreadPoolExecutor在一個線程中開啟一個Runnable後，它會自動調用Runnable的run()方法。

## 中斷正在執行的代碼

為了停止執行一個任務，你必須中斷執行這個任務的線程。在準備做這件事之前，當你創建一個任務時，你需要存儲處理該任務的線程。例如：

```java
class PhotoDecodeRunnable implements Runnable {
    // Defines the code to run for this task
    public void run() {
        /*
         * Stores the current Thread in the
         * object that contains PhotoDecodeRunnable
         */
        mPhotoTask.setImageDecodeThread(Thread.currentThread());
        ...
    }
    ...
}
```

想要中斷一個線程，你可以調用[Thread.interrupt()](http://developer.android.com/reference/java/lang/Thread.html#interrupt())。需要注意的是這些線程對象都被系統控制，系統可以在你的APP進程之外修改
他們。因為這個原因，在你要中斷一個線程時，你需要把這段代碼放在一個同步代碼塊中對這個線程的訪問加鎖來解決這個問題。例如：

```java
public class PhotoManager {
    public static void cancelAll() {
        /*
         * Creates an array of Runnables that's the same size as the
         * thread pool work queue
         */
        Runnable[] runnableArray = new Runnable[mDecodeWorkQueue.size()];
        // Populates the array with the Runnables in the queue
        mDecodeWorkQueue.toArray(runnableArray);
        // Stores the array length in order to iterate over the array
        int len = runnableArray.length;
        /*
         * Iterates over the array of Runnables and interrupts each one's Thread.
         */
        synchronized (sInstance) {
            // Iterates over the array of tasks
            for (int runnableIndex = 0; runnableIndex < len; runnableIndex++) {
                // Gets the current thread
                Thread thread = runnableArray[taskArrayIndex].mThread;
                // if the Thread exists, post an interrupt to it
                if (null != thread) {
                    thread.interrupt();
                }
            }
        }
    }
    ...
}
```

在大多數情況下，通過調用Thread.interrupt()能立即中斷這個線程，然而他只能停止那些處於等待狀態的線程，卻不能中斷那些佔據CPU或者耗時的連接網絡的任務。為了避免拖慢系統速度或造成系統死鎖，在嘗試執行耗時操作之前，你應該測試當前是否存在處於掛起狀態的中斷請求：

```java
/*
 * Before continuing, checks to see that the Thread hasn't
 * been interrupted
 */
if (Thread.interrupted()) {
    return;
}
...
// Decodes a byte array into a Bitmap (CPU-intensive)
BitmapFactory.decodeByteArray(
        imageBuffer, 0, imageBuffer.length, bitmapOptions);
...
```



