# 管理Bitmap的內存使用

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/displaying-bitmaps/manage-memory.html>

這節課將作為緩存Bitmaps課程的進一步延伸。為了優化垃圾回收機制與Bitmap的重用，我們還有一些特定的事情可以做。 同時根據Android的不同版本，推薦的策略會有所差異。[DisplayingBitmaps](http://developer.android.com/downloads/samples/DisplayingBitmaps.zip)的示例程序會演示如何設計我們的程序，使得它能夠在不同的Android平臺上高效地運行.

為了給這節課奠定基礎，我們首先要知道Android管理Bitmap內存使用的演變進程:

* 在Android 2.2 (API level 8)以及之前，當垃圾回收發生時，應用的線程是會被暫停的，這會導致一個延遲滯後，並降低系統效率。 **從Android 2.3開始，添加了併發垃圾回收的機制， 這意味著在一個Bitmap不再被引用之後，它所佔用的內存會被立即回收。**
* 在Android 2.3.3 (API level 10)以及之前, 一個Bitmap的像素級數據（pixel data）是存放在Native內存空間中的。 這些數據與Bitmap本身是隔離的，Bitmap本身被存放在Dalvik堆中。我們無法預測在Native內存中的像素級數據何時會被釋放，這意味著程序容易超過它的內存限制並且崩潰。 **自Android 3.0 (API Level 11)開始， 像素級數據則是與Bitmap本身一起存放在Dalvik堆中。**

下面會介紹如何在不同的Android版本上優化Bitmap內存使用。

<!-- more -->

## 管理Android 2.3.3及以下版本的內存使用

在Android 2.3.3 (API level 10) 以及更低版本上，推薦使用<a href="http://developer.android.com/reference/android/graphics/Bitmap.html#recycle()">recycle()</a>方法。 如果在應用中顯示了大量的Bitmap數據，我們很可能會遇到[OutOfMemoryError](http://developer.android.com/reference/java/lang/OutOfMemoryError.html)的錯誤。 <a href="http://developer.android.com/reference/android/graphics/Bitmap.html#recycle()">recycle()</a>方法可以使得程序更快的釋放內存。

> **Caution：**只有當我們確定這個Bitmap不再需要用到的時候才應該使用recycle()。在執行recycle()方法之後，如果嘗試繪製這個Bitmap， 我們將得到`"Canvas: trying to use a recycled bitmap"`的錯誤提示。

下面的代碼片段演示了使用`recycle()`的例子。它使用了引用計數的方法（`mDisplayRefCount` 與 `mCacheRefCount`）來追蹤一個Bitmap目前是否有被顯示或者是在緩存中。並且在下面列舉的條件滿足時，回收Bitmap：

* `mDisplayRefCount` 與 `mCacheRefCount` 的引用計數均為 0；
* bitmap不為`null`, 並且它還沒有被回收。

```java
private int mCacheRefCount = 0;
private int mDisplayRefCount = 0;
...
// Notify the drawable that the displayed state has changed.
// Keep a count to determine when the drawable is no longer displayed.
public void setIsDisplayed(boolean isDisplayed) {
    synchronized (this) {
        if (isDisplayed) {
            mDisplayRefCount++;
            mHasBeenDisplayed = true;
        } else {
            mDisplayRefCount--;
        }
    }
    // Check to see if recycle() can be called.
    checkState();
}

// Notify the drawable that the cache state has changed.
// Keep a count to determine when the drawable is no longer being cached.
public void setIsCached(boolean isCached) {
    synchronized (this) {
        if (isCached) {
            mCacheRefCount++;
        } else {
            mCacheRefCount--;
        }
    }
    // Check to see if recycle() can be called.
    checkState();
}

private synchronized void checkState() {
    // If the drawable cache and display ref counts = 0, and this drawable
    // has been displayed, then recycle.
    if (mCacheRefCount <= 0 && mDisplayRefCount <= 0 && mHasBeenDisplayed
            && hasValidBitmap()) {
        getBitmap().recycle();
    }
}

private synchronized boolean hasValidBitmap() {
    Bitmap bitmap = getBitmap();
    return bitmap != null && !bitmap.isRecycled();
}
```

## 管理Android 3.0及其以上版本的內存

從Android 3.0 (API Level 11)開始，引進了[BitmapFactory.Options.inBitmap](http://developer.android.com/reference/android/graphics/BitmapFactory.Options.html#inBitmap)字段。 如果使用了這個設置字段，decode方法會在加載Bitmap數據的時候去重用已經存在的Bitmap。這意味著Bitmap的內存是被重新利用的，這樣可以提升性能，並且減少了內存的分配與回收。然而，使用[inBitmap](http://developer.android.com/reference/android/graphics/BitmapFactory.Options.html#inBitmap)有一些限制，特別是在Android 4.4 (API level 19)之前，只有同等大小的位圖才可以被重用。詳情請查看[inBitmap文檔](http://developer.android.com/reference/android/graphics/BitmapFactory.Options.html#inBitmap)。

### 保存Bitmap供以後使用

下面演示瞭如何將一個已經存在的Bitmap存放起來以便後續使用。當一個應用運行在Android 3.0或者更高的平臺上並且Bitmap從LruCache中移除時，Bitmap的一個軟引用會被存放在[Hashset](http://developer.android.com/reference/java/util/HashSet.html)中，這樣便於之後可能被[inBitmap](http://developer.android.com/reference/android/graphics/BitmapFactory.Options.html#inBitmap)重用：

```java
Set<SoftReference<Bitmap>> mReusableBitmaps;
private LruCache<String, BitmapDrawable> mMemoryCache;

// If you're running on Honeycomb or newer, create a
// synchronized HashSet of references to reusable bitmaps.
if (Utils.hasHoneycomb()) {
    mReusableBitmaps =
            Collections.synchronizedSet(new HashSet<SoftReference<Bitmap>>());
}

mMemoryCache = new LruCache<String, BitmapDrawable>(mCacheParams.memCacheSize) {

    // Notify the removed entry that is no longer being cached.
    @Override
    protected void entryRemoved(boolean evicted, String key,
            BitmapDrawable oldValue, BitmapDrawable newValue) {
        if (RecyclingBitmapDrawable.class.isInstance(oldValue)) {
            // The removed entry is a recycling drawable, so notify it
            // that it has been removed from the memory cache.
            ((RecyclingBitmapDrawable) oldValue).setIsCached(false);
        } else {
            // The removed entry is a standard BitmapDrawable.
            if (Utils.hasHoneycomb()) {
                // We're running on Honeycomb or later, so add the bitmap
                // to a SoftReference set for possible use with inBitmap later.
                mReusableBitmaps.add
                        (new SoftReference<Bitmap>(oldValue.getBitmap()));
            }
        }
    }
....
}
```

### 使用已經存在的Bitmap

在運行的程序中，decode方法會檢查看是否存在可重用的Bitmap。 例如:

```java
public static Bitmap decodeSampledBitmapFromFile(String filename,
        int reqWidth, int reqHeight, ImageCache cache) {

    final BitmapFactory.Options options = new BitmapFactory.Options();
    ...
    BitmapFactory.decodeFile(filename, options);
    ...

    // If we're running on Honeycomb or newer, try to use inBitmap.
    if (Utils.hasHoneycomb()) {
        addInBitmapOptions(options, cache);
    }
    ...
    return BitmapFactory.decodeFile(filename, options);
}
```

下面的代碼是上述代碼片段中，`addInBitmapOptions()`方法的具體實現。 它會為inBitmap查找一個已經存在的Bitmap，並將它設置為inBitmap的值。 注意這個方法只有在找到合適且可重用的Bitmap時才會賦值給inBitmap（我們需要在賦值之前進行檢查）：

```java
private static void addInBitmapOptions(BitmapFactory.Options options,
        ImageCache cache) {
    // inBitmap only works with mutable bitmaps, so force the decoder to
    // return mutable bitmaps.
    options.inMutable = true;

    if (cache != null) {
        // Try to find a bitmap to use for inBitmap.
        Bitmap inBitmap = cache.getBitmapFromReusableSet(options);

        if (inBitmap != null) {
            // If a suitable bitmap has been found, set it as the value of
            // inBitmap.
            options.inBitmap = inBitmap;
        }
    }
}

// This method iterates through the reusable bitmaps, looking for one
// to use for inBitmap:
protected Bitmap getBitmapFromReusableSet(BitmapFactory.Options options) {
        Bitmap bitmap = null;

    if (mReusableBitmaps != null && !mReusableBitmaps.isEmpty()) {
        synchronized (mReusableBitmaps) {
            final Iterator<SoftReference<Bitmap>> iterator
                    = mReusableBitmaps.iterator();
            Bitmap item;

            while (iterator.hasNext()) {
                item = iterator.next().get();

                if (null != item && item.isMutable()) {
                    // Check to see it the item can be used for inBitmap.
                    if (canUseForInBitmap(item, options)) {
                        bitmap = item;

                        // Remove from reusable set so it can't be used again.
                        iterator.remove();
                        break;
                    }
                } else {
                    // Remove from the set if the reference has been cleared.
                    iterator.remove();
                }
            }
        }
    }
    return bitmap;
}
```

最後，下面這個方法判斷候選Bitmap是否滿足inBitmap的大小條件:

```java
static boolean canUseForInBitmap(
        Bitmap candidate, BitmapFactory.Options targetOptions) {

    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
        // From Android 4.4 (KitKat) onward we can re-use if the byte size of
        // the new bitmap is smaller than the reusable bitmap candidate
        // allocation byte count.
        int width = targetOptions.outWidth / targetOptions.inSampleSize;
        int height = targetOptions.outHeight / targetOptions.inSampleSize;
        int byteCount = width * height * getBytesPerPixel(candidate.getConfig());
        return byteCount <= candidate.getAllocationByteCount();
    }

    // On earlier versions, the dimensions must match exactly and the inSampleSize must be 1
    return candidate.getWidth() == targetOptions.outWidth
            && candidate.getHeight() == targetOptions.outHeight
            && targetOptions.inSampleSize == 1;
}

/**
 * A helper function to return the byte usage per pixel of a bitmap based on its configuration.
 */
static int getBytesPerPixel(Config config) {
    if (config == Config.ARGB_8888) {
        return 4;
    } else if (config == Config.RGB_565) {
        return 2;
    } else if (config == Config.ARGB_4444) {
        return 2;
    } else if (config == Config.ALPHA_8) {
        return 1;
    }
    return 1;
}
```
