# 非UI線程處理Bitmap

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/displaying-bitmaps/process-bitmap.html>

在上一課中介紹了一系列的<a href="http://developer.android.com/reference/android/graphics/BitmapFactory.html#decodeByteArray(byte[], int, int, android.graphics.BitmapFactory.Options">BitmapFactory.decode*</a>方法，當圖片來源是網絡或者是存儲卡時（或者是任何不在內存中的形式），這些方法都不應該在UI 線程中執行。因為在上述情況下加載數據時，其執行時間是不可估計的，它依賴於許多因素（從網絡或者存儲卡讀取數據的速度，圖片的大小，CPU的速度等）。如果其中任何一個子操作阻塞了UI線程，系統都會容易出現應用無響應的錯誤。

這一節課會介紹如何使用AsyncTask在後臺線程中處理Bitmap並且演示如何處理併發（concurrency）的問題。

## 使用AsyncTask(Use a AsyncTask)

[AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) 類提供了一個在後臺線程執行一些操作的簡單方法，它還可以把後臺的執行結果呈現到UI線程中。下面是一個加載大圖的示例：

```java
class BitmapWorkerTask extends AsyncTask {
    private final WeakReference imageViewReference;
    private int data = 0;

    public BitmapWorkerTask(ImageView imageView) {
        // Use a WeakReference to ensure the ImageView can be garbage collected
        imageViewReference = new WeakReference(imageView);
    }

    // Decode image in background.
    @Override
    protected Bitmap doInBackground(Integer... params) {
        data = params[0];
        return decodeSampledBitmapFromResource(getResources(), data, 100, 100));
    }

    // Once complete, see if ImageView is still around and set bitmap.
    @Override
    protected void onPostExecute(Bitmap bitmap) {
        if (imageViewReference != null && bitmap != null) {
            final ImageView imageView = imageViewReference.get();
            if (imageView != null) {
                imageView.setImageBitmap(bitmap);
            }
        }
    }
}
```

為[ImageView](http://developer.android.com/reference/android/widget/ImageView.html)使用[WeakReference](http://developer.android.com/reference/java/lang/ref/WeakReference.html)確保了AsyncTask所引用的資源可以被垃圾回收器回收。由於當任務結束時不能確保[ImageView](http://developer.android.com/reference/android/widget/ImageView.html)仍然存在，因此我們必須在`onPostExecute()`裡面對引用進行檢查。該ImageView在有些情況下可能已經不存在了，例如，在任務結束之前用戶使用了回退操作，或者是配置發生了改變（如旋轉屏幕等）。

開始異步加載位圖，只需要創建一個新的任務並執行它即可:

```java
public void loadBitmap(int resId, ImageView imageView) {
    BitmapWorkerTask task = new BitmapWorkerTask(imageView);
    task.execute(resId);
}
```

## 處理併發問題(Handle Concurrency)

通常類似ListView與GridView等視圖控件在使用上面演示的AsyncTask 方法時，會同時帶來併發的問題。首先為了更高的效率，ListView與GridView的子Item視圖會在用戶滑動屏幕時被循環使用。如果每一個子視圖都觸發一個AsyncTask，那麼就無法確保關聯的視圖在結束任務時，分配的視圖已經進入循環隊列中，給另外一個子視圖進行重用。而且， 無法確保所有的異步任務的完成順序和他們本身的啟動順序保持一致。

[Multithreading for Performance](http://android-developers.blogspot.com/2010/07/multithreading-for-performance.html) 這篇博文更進一步的討論瞭如何處理併發問題，並且提供了一種解決方法：ImageView保存最近使用的AsyncTask的引用，這個引用可以在任務完成的時候再次讀取檢查。使用這種方式, 就可以對前面提到的AsyncTask進行擴展。

創建一個專用的[Drawable](http://developer.android.com/reference/android/graphics/drawable/Drawable.html)的子類來儲存任務的引用。在這種情況下，我們使用了一個[BitmapDrawable](http://developer.android.com/reference/android/graphics/drawable/BitmapDrawable.html)，在任務執行的過程中，一個佔位圖片會顯示在[ImageView](http://developer.android.com/reference/android/widget/ImageView.html)中:

```java
static class AsyncDrawable extends BitmapDrawable {
    private final WeakReference bitmapWorkerTaskReference;

    public AsyncDrawable(Resources res, Bitmap bitmap,
            BitmapWorkerTask bitmapWorkerTask) {
        super(res, bitmap);
        bitmapWorkerTaskReference =
            new WeakReference(bitmapWorkerTask);
    }

    public BitmapWorkerTask getBitmapWorkerTask() {
        return bitmapWorkerTaskReference.get();
    }
}
```

在執行[BitmapWorkerTask](http://developer.android.com/training/displaying-bitmaps/process-bitmap.html#BitmapWorkerTask) 之前，你需要創建一個AsyncDrawable並且將它綁定到目標控件ImageView中：

```java
public void loadBitmap(int resId, ImageView imageView) {
    if (cancelPotentialWork(resId, imageView)) {
        final BitmapWorkerTask task = new BitmapWorkerTask(imageView);
        final AsyncDrawable asyncDrawable =
                new AsyncDrawable(getResources(), mPlaceHolderBitmap, task);
        imageView.setImageDrawable(asyncDrawable);
        task.execute(resId);
    }
}
```

在上面的代碼示例中，`cancelPotentialWork` 方法檢查是否有另一個正在執行的任務與該ImageView關聯了起來，如果的確是這樣，它通過執行`cancel()`方法來取消另一個任務。在少數情況下, 新創建的任務數據可能會與已經存在的任務相吻合，這樣的話就不需要進行下一步動作了。下面是 `cancelPotentialWork`方法的實現 。

```java
public static boolean cancelPotentialWork(int data, ImageView imageView) {
    final BitmapWorkerTask bitmapWorkerTask = getBitmapWorkerTask(imageView);

    if (bitmapWorkerTask != null) {
        final int bitmapData = bitmapWorkerTask.data;
        if (bitmapData == 0 || bitmapData != data) {
            // Cancel previous task
            bitmapWorkerTask.cancel(true);
        } else {
            // The same work is already in progress
            return false;
        }
    }
    // No task associated with the ImageView, or an existing task was cancelled
    return true;
}
```

在上面的代碼中有一個輔助方法：`getBitmapWorkerTask()`，它被用作檢索AsyncTask是否已經被分配到指定的ImageView:

```java
private static BitmapWorkerTask getBitmapWorkerTask(ImageView imageView) {
   if (imageView != null) {
       final Drawable drawable = imageView.getDrawable();
       if (drawable instanceof AsyncDrawable) {
           final AsyncDrawable asyncDrawable = (AsyncDrawable) drawable;
           return asyncDrawable.getBitmapWorkerTask();
       }
    }
    return null;
}
```

最後一步是在BitmapWorkerTask的`onPostExecute()` 方法裡面做更新操作:

```java
class BitmapWorkerTask extends AsyncTask {
    ...

    @Override
    protected void onPostExecute(Bitmap bitmap) {
        if (isCancelled()) {
            bitmap = null;
        }

        if (imageViewReference != null && bitmap != null) {
            final ImageView imageView = imageViewReference.get();
            final BitmapWorkerTask bitmapWorkerTask =
                    getBitmapWorkerTask(imageView);
            if (this == bitmapWorkerTask && imageView != null) {
                imageView.setImageBitmap(bitmap);
            }
        }
    }
}
```

這個方法不僅僅適用於ListView與GridView控件，在那些需要循環利用子視圖的控件中同樣適用：只需要在設置圖片到ImageView的地方調用 `loadBitmap`方法。例如，在GridView 中實現這個方法可以在 getView()中調用。
