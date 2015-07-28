# 與UI線程通信

> 編寫:[AllenZheng1991](https://github.com/AllenZheng1991) - 原文:<http://developer.android.com/training/multiple-threads/communicate-ui.html>

在前面的課程中你學習瞭如何在一個被[ThreadPoolExecutor](http://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html)管理的線程中開啟一個任務。最後這一節課將會向你展示如何從執行的任務中發送數據給運行在UI線程中的對象。這個功能允許你的任務可以做後臺工作，然後把得到的結果數據轉移給UI元素使用，例如位圖數據。

任何一個APP都有自己特定的一個線程用來運行UI對象，比如[View](http://developer.android.com/reference/android/view/View.html)對象，這個線程我們稱之為UI線程。只有運行在UI線程中的對象能訪問運行在其它線程中的對象。因為你的任務執行的線程來自一個線程池而不是執行在UI線程，所以他們不能訪問UI對象。為了把數據從一個後臺線程轉移到UI線程，需要使用一個運行在UI線程裡的[Handler](http://developer.android.com/reference/android/os/Handler.html)。

##在UI線程中定義一個Handler

[Handler](http://developer.android.com/reference/android/os/Handler.html)屬於Android系統的線程管理框架的一部分。一個[Handler](http://developer.android.com/reference/android/os/Handler.html)對象用於接收消息和執行處理消息的代碼。一般情況下，如果你為一個新線程創建了一個[Handler](http://developer.android.com/reference/android/os/Handler.html)，你還需要創建一個[Handler](http://developer.android.com/reference/android/os/Handler.html)，讓它與一個已經存在的線程關聯，用於這兩個線程之間的通信。如果你把一個[Handler](http://developer.android.com/reference/android/os/Handler.html)關聯到UI線程，處理消息的代碼就會在UI線程中執行。

你可以在一個用於創建你的線程池的類的構造方法中實例化一個[Handler](http://developer.android.com/reference/android/os/Handler.html)對象，並把它定義為全局變量，然後通過使用[Handler (Looper) ](http://developer.android.com/reference/android/os/Handler.html#Handler)這一構造方法實例化它，用於關聯到UI線程。<a href="http://developer.android.com/reference/android/os/Handler.html#Handler(android.os.Looper)" target="_blank">Handler(Looper)</a>這一構造方法需要傳入了一個[Looper](http://developer.android.com/reference/android/os/Looper.html)對象，它是Android系統的線程管理框架中的另一部分。當你在一個特定的[Looper](http://developer.android.com/reference/android/os/Looper.html)實例的基礎上去實例化一個[Handler](http://developer.android.com/reference/android/os/Handler.html)時，這個[Handler](http://developer.android.com/reference/android/os/Handler.html)與[Looper](http://developer.android.com/reference/android/os/Looper.html)運行在同一個線程裡。例如：

```java
private PhotoManager() {
...
    // Defines a Handler object that's attached to the UI thread
    mHandler = new Handler(Looper.getMainLooper()) {
    ...
```

在這個[Handler](http://developer.android.com/reference/android/os/Handler.html)裡需要重寫<a href="http://developer.android.com/reference/android/os/Handler.html#handleMessage(android.os.Message)" target="_blank">handleMessage()</a>方法。當這個[Handler](http://developer.android.com/reference/android/os/Handler.html)接收到由另外一個線程管理的[Handler](http://developer.android.com/reference/android/os/Handler.html)發送過來的新消息時，Android系統會自動調用這個方法，而所有線程對應的[Handler](http://developer.android.com/reference/android/os/Handler.html)都會收到相同信息。例如：

```java
        /*
         * handleMessage() defines the operations to perform when
         * the Handler receives a new Message to process.
         */
        @Override
        public void handleMessage(Message inputMessage) {
            // Gets the image task from the incoming Message object.
            PhotoTask photoTask = (PhotoTask) inputMessage.obj;
            ...
        }
    ...
    }
}
```

下一部分將向你展示如何用[Handler](http://developer.android.com/reference/android/os/Handler.html)轉移數據。

##把數據從一個任務中轉移到UI線程

為了從一個運行在後臺線程的任務對象中轉移數據到UI線程中的一個對象，首先需要存儲任務對象中的數據和UI對象的引用；接下來傳遞任務對象和狀態碼給實例化[Handler](http://developer.android.com/reference/android/os/Handler.html)的那個對象。在這個對象裡，發送一個包含任務對象和狀態的[Message](http://developer.android.com/reference/android/os/Message.html)給[Handler](http://developer.android.com/reference/android/os/Handler.html)也運行在UI線程中，所以它可以把數據轉移到UI線程。

###在任務對象中存儲數據

比如這裡有一個[Runnable](http://developer.android.com/reference/java/lang/Runnable.html)，它運行在一個編碼了一個[Bitmap](http://developer.android.com/reference/android/graphics/Bitmap.html)且存儲這個[Bitmap](http://developer.android.com/reference/android/graphics/Bitmap.html)到父類*PhotoTask*對象裡的後臺線程。這個[Runnable](http://developer.android.com/reference/java/lang/Runnable.html)同樣也存儲了狀態碼*DECODE_STATE_COMPLETED*。

```java
// A class that decodes photo files into Bitmaps
class PhotoDecodeRunnable implements Runnable {
    ...
    PhotoDecodeRunnable(PhotoTask downloadTask) {
        mPhotoTask = downloadTask;
    }
    ...
    // Gets the downloaded byte array
    byte[] imageBuffer = mPhotoTask.getByteBuffer();
    ...
    // Runs the code for this task
    public void run() {
        ...
        // Tries to decode the image buffer
        returnBitmap = BitmapFactory.decodeByteArray(
                imageBuffer,
                0,
                imageBuffer.length,
                bitmapOptions
        );
        ...
        // Sets the ImageView Bitmap
        mPhotoTask.setImage(returnBitmap);
        // Reports a status of "completed"
        mPhotoTask.handleDecodeState(DECODE_STATE_COMPLETED);
        ...
    }
    ...
}
...
```

*PhotoTask*類還包含一個用於給[ImageView](http://developer.android.com/reference/android/widget/ImageView.html)顯示[Bitmap](http://developer.android.com/reference/android/graphics/Bitmap.html)的handler。雖然[Bitmap](http://developer.android.com/reference/android/graphics/Bitmap.html)和[ImageView](http://developer.android.com/reference/android/widget/ImageView.html)ImageView</a>的引用在同一個對象中，但你不能把這個[Bitmap](http://developer.android.com/reference/android/graphics/Bitmap.html)分配給[ImageView](http://developer.android.com/reference/android/widget/ImageView.html)去顯示，因為它們並沒有運行在UI線程中。

這時，下一步應該發送這個狀態給`PhotoTask`對象。

###發送狀態取決於對象層次

*PhotoTask*是下一個層次更高的對象，它包含將要展示數據的編碼數據和[View](http://developer.android.com/reference/android/view/View.html)對象的引用。它會收到一個來自*PhotoDecodeRunnable*的狀態碼，並把這個狀態碼單獨傳遞到一個包含線程池和[Handler](http://developer.android.com/reference/android/os/Handler.html)實例的對象：

```java
public class PhotoTask {
    ...
    // Gets a handle to the object that creates the thread pools
    sPhotoManager = PhotoManager.getInstance();
    ...
    public void handleDecodeState(int state) {
        int outState;
        // Converts the decode state to the overall state.
        switch(state) {
            case PhotoDecodeRunnable.DECODE_STATE_COMPLETED:
                outState = PhotoManager.TASK_COMPLETE;
                break;
            ...
        }
        ...
        // Calls the generalized state method
        handleState(outState);
    }
    ...
    // Passes the state to PhotoManager
    void handleState(int state) {
        /*
         * Passes a handle to this task and the
         * current state to the class that created
         * the thread pools
         */
        sPhotoManager.handleState(this, state);
    }
    ...
}
```

###轉移數據到UI
從*PhotoTask*對象那裡，*PhotoManager*對象收到了一個狀態碼和一個*PhotoTask*對象的handler。因為狀態碼是*TASK_COMPLETE*，所以創建一個[Message](http://developer.android.com/reference/android/os/Message.html)應該包含狀態和任務對象，然後把它發送給[Handler](http://developer.android.com/reference/android/os/Handler.html)：

```java
public class PhotoManager {
    ...
    // Handle status messages from tasks
    public void handleState(PhotoTask photoTask, int state) {
        switch (state) {
            ...
            // The task finished downloading and decoding the image
            case TASK_COMPLETE:
                /*
                 * Creates a message for the Handler
                 * with the state and the task object
                 */
                Message completeMessage =
                        mHandler.obtainMessage(state, photoTask);
                completeMessage.sendToTarget();
                break;
            ...
        }
        ...
    }
```

最終，<a href="http://developer.android.com/reference/android/os/Handler.html#handleMessage(android.os.Message)" target="_blank">Handler.handleMessage()</a>會檢查每個傳入進來的[Message](http://developer.android.com/reference/android/os/Message.html)，如果狀態碼是*TASK_COMPLETE*，這時任務就完成了，而傳入的[Message](http://developer.android.com/reference/android/os/Message.html)裡的*PhotoTask*對象裡同時包含一個[Bitmap](http://developer.android.com/reference/android/graphics/Bitmap.html)和一個[ImageView](http://developer.android.com/reference/android/widget/ImageView.html)。因為<a href="http://developer.android.com/reference/android/os/Handler.html#handleMessage(android.os.Message)" target="_blank">Handler.handleMessage()</a>運行在UI線程裡，所以它能安全地轉移[Bitmap](http://developer.android.com/reference/android/graphics/Bitmap.html)數據給[ImageView](http://developer.android.com/reference/android/widget/ImageView.html)：

```java
private PhotoManager() {
        ...
            mHandler = new Handler(Looper.getMainLooper()) {
                @Override
                public void handleMessage(Message inputMessage) {
                    // Gets the task from the incoming Message object.
                    PhotoTask photoTask = (PhotoTask) inputMessage.obj;
                    // Gets the ImageView for this task
                    PhotoView localView = photoTask.getPhotoView();
                    ...
                    switch (inputMessage.what) {
                        ...
                        // The decoding is done
                        case TASK_COMPLETE:
                            /*
                             * Moves the Bitmap from the task
                             * to the View
                             */
                            localView.setImageBitmap(photoTask.getImage());
                            break;
                        ...
                        default:
                            /*
                             * Pass along other messages from the UI
                             */
                            super.handleMessage(inputMessage);
                    }
                    ...
                }
                ...
            }
            ...
    }
...
}
```

