# 緩存Bitmap

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/displaying-bitmaps/cache-bitmap.html>

將單個Bitmap加載到UI是簡單直接的，但是如果我們需要一次性加載大量的圖片，事情則會變得複雜起來。在大多數情況下（例如在使用ListView，GridView或ViewPager時），屏幕上的圖片和因滑動將要顯示的圖片的數量通常是沒有限制的。

通過循環利用子視圖可以緩解內存的使用，垃圾回收器也會釋放那些不再需要使用的Bitmap。這些機制都非常好，但是為了保證一個流暢的用戶體驗，我們希望避免在每次屏幕滑動回來時，都要重複處理那些圖片。內存與磁盤緩存通常可以起到輔助作用，允許控件可以快速地重新加載那些處理過的圖片。

這一課會介紹在加載多張Bitmap時使用內存緩存與磁盤緩存來提高響應速度與UI流暢度。

## 使用內存緩存(Use a Memory Cache)

內存緩存以花費寶貴的程序內存為前提來快速訪問位圖。[LruCache](http://developer.android.com/reference/android/util/LruCache.html)類（在API Level 4的Support Library中也可以找到）特別適合用來緩存Bitmaps，它使用一個強引用（strong referenced）的[LinkedHashMap](http://developer.android.com/reference/java/util/LinkedHashMap.html)保存最近引用的對象，並且在緩存超出設置大小的時候剔除（evict）最近最少使用到的對象。

> **Note:** 在過去，一種比較流行的內存緩存實現方法是使用軟引用（SoftReference）或弱引用（WeakReference）對Bitmap進行緩存，然而我們並不推薦這樣的做法。從Android 2.3 (API Level 9)開始，垃圾回收機制變得更加頻繁，這使得釋放軟（弱）引用的頻率也隨之增高，導致使用引用的效率降低很多。而且在Android 3.0 (API Level 11)之前，備份的Bitmap會存放在Native Memory中，它不是以可預知的方式被釋放的，這樣可能導致程序超出它的內存限制而崩潰。

為了給LruCache選擇一個合適的大小，需要考慮到下面一些因素：

* 應用剩下了多少可用的內存?
* 多少張圖片會同時呈現到屏幕上？有多少圖片需要準備好以便馬上顯示到屏幕？
* 設備的屏幕大小與密度是多少？一個具有特別高密度屏幕（xhdpi）的設備，像Galaxy Nexus會比Nexus S（hdpi）需要一個更大的緩存空間來緩存同樣數量的圖片。
* Bitmap的尺寸與配置是多少，會花費多少內存？
* 圖片被訪問的頻率如何？是其中一些比另外的訪問更加頻繁嗎？如果是，那麼我們可能希望在內存中保存那些最常訪問的圖片，或者根據訪問頻率給Bitmap分組，為不同的Bitmap組設置多個LruCache對象。
* 是否可以在緩存圖片的質量與數量之間尋找平衡點？某些時候保存大量低質量的Bitmap會非常有用，加載更高質量圖片的任務可以交給另外一個後臺線程。

通常沒有指定的大小或者公式能夠適用於所有的情形，我們需要分析實際的使用情況後，提出一個合適的解決方案。緩存太小會導致額外的花銷卻沒有明顯的好處，緩存太大同樣會導致java.lang.OutOfMemory的異常，並且使得你的程序只留下小部分的內存用來工作（緩存佔用太多內存，導致其他操作會因為內存不夠而拋出異常）。

下面是一個為Bitmap建立LruCache的示例：

```java
private LruCache<String, Bitmap> mMemoryCache;

@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    // Get max available VM memory, exceeding this amount will throw an
    // OutOfMemory exception. Stored in kilobytes as LruCache takes an
    // int in its constructor.
    final int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);

    // Use 1/8th of the available memory for this memory cache.
    final int cacheSize = maxMemory / 8;

    mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
        @Override
        protected int sizeOf(String key, Bitmap bitmap) {
            // The cache size will be measured in kilobytes rather than
            // number of items.
            return bitmap.getByteCount() / 1024;
        }
    };
    ...
}

public void addBitmapToMemoryCache(String key, Bitmap bitmap) {
    if (getBitmapFromMemCache(key) == null) {
        mMemoryCache.put(key, bitmap);
    }
}

public Bitmap getBitmapFromMemCache(String key) {
    return mMemoryCache.get(key);
}
```

> **Note:**在上面的例子中, 有1/8的內存空間被用作緩存。 這意味著在常見的設備上（hdpi），最少大概有4MB的緩存空間（32/8）。如果一個填滿圖片的GridView控件放置在800x480像素的手機屏幕上，大概會花費1.5MB的緩存空間（800x480x4 bytes），因此緩存的容量大概可以緩存2.5頁的圖片內容。

當加載Bitmap顯示到ImageView 之前，會先從LruCache 中檢查是否存在這個Bitmap。如果確實存在，它會立即被用來顯示到ImageView上，如果沒有找到，會觸發一個後臺線程去處理顯示該Bitmap任務。

```java
public void loadBitmap(int resId, ImageView imageView) {
    final String imageKey = String.valueOf(resId);

    final Bitmap bitmap = getBitmapFromMemCache(imageKey);
    if (bitmap != null) {
        mImageView.setImageBitmap(bitmap);
    } else {
        mImageView.setImageResource(R.drawable.image_placeholder);
        BitmapWorkerTask task = new BitmapWorkerTask(mImageView);
        task.execute(resId);
    }
}
```

上面的程序中 `BitmapWorkerTask` 需要把解析好的Bitmap添加到內存緩存中：

```java
class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    ...
    // Decode image in background.
    @Override
    protected Bitmap doInBackground(Integer... params) {
        final Bitmap bitmap = decodeSampledBitmapFromResource(
                getResources(), params[0], 100, 100));
        addBitmapToMemoryCache(String.valueOf(params[0]), bitmap);
        return bitmap;
    }
    ...
}
```

## 使用磁盤緩存(Use a Disk Cache)

內存緩存能夠提高訪問最近用過的Bitmap的速度，但是我們無法保證最近訪問過的Bitmap都能夠保存在緩存中。像類似GridView等需要大量數據填充的控件很容易就會用盡整個內存緩存。另外，我們的應用可能會被類似打電話等行為而暫停並退到後臺，因為後臺應用可能會被殺死，那麼內存緩存就會被銷燬，裡面的Bitmap也就不存在了。一旦用戶恢復應用的狀態，那麼應用就需要重新處理那些圖片。

磁盤緩存可以用來保存那些已經處理過的Bitmap，它還可以減少那些不再內存緩存中的Bitmap的加載次數。當然從磁盤讀取圖片會比從內存要慢，而且由於磁盤讀取操作時間是不可預期的，讀取操作需要在後臺線程中處理。

> **Note:**如果圖片會被更頻繁的訪問，使用[ContentProvider](http://developer.android.com/reference/android/content/ContentProvider.html)或許會更加合適，比如在圖庫應用中。

這一節的範例代碼中使用了一個從[Android源碼](https://android.googlesource.com/platform/libcore/+/jb-mr2-release/luni/src/main/java/libcore/io/DiskLruCache.java)中剝離出來的`DiskLruCache`。改進過的範例代碼在已有內存緩存的基礎上增加磁盤緩存的功能。

```java
private DiskLruCache mDiskLruCache;
private final Object mDiskCacheLock = new Object();
private boolean mDiskCacheStarting = true;
private static final int DISK_CACHE_SIZE = 1024 * 1024 * 10; // 10MB
private static final String DISK_CACHE_SUBDIR = "thumbnails";

@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    // Initialize memory cache
    ...
    // Initialize disk cache on background thread
    File cacheDir = getDiskCacheDir(this, DISK_CACHE_SUBDIR);
    new InitDiskCacheTask().execute(cacheDir);
    ...
}

class InitDiskCacheTask extends AsyncTask<File, Void, Void> {
    @Override
    protected Void doInBackground(File... params) {
        synchronized (mDiskCacheLock) {
            File cacheDir = params[0];
            mDiskLruCache = DiskLruCache.open(cacheDir, DISK_CACHE_SIZE);
            mDiskCacheStarting = false; // Finished initialization
            mDiskCacheLock.notifyAll(); // Wake any waiting threads
        }
        return null;
    }
}

class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    ...
    // Decode image in background.
    @Override
    protected Bitmap doInBackground(Integer... params) {
        final String imageKey = String.valueOf(params[0]);

        // Check disk cache in background thread
        Bitmap bitmap = getBitmapFromDiskCache(imageKey);

        if (bitmap == null) { // Not found in disk cache
            // Process as normal
            final Bitmap bitmap = decodeSampledBitmapFromResource(
                    getResources(), params[0], 100, 100));
        }

        // Add final bitmap to caches
        addBitmapToCache(imageKey, bitmap);

        return bitmap;
    }
    ...
}

public void addBitmapToCache(String key, Bitmap bitmap) {
    // Add to memory cache as before
    if (getBitmapFromMemCache(key) == null) {
        mMemoryCache.put(key, bitmap);
    }

    // Also add to disk cache
    synchronized (mDiskCacheLock) {
        if (mDiskLruCache != null && mDiskLruCache.get(key) == null) {
            mDiskLruCache.put(key, bitmap);
        }
    }
}

public Bitmap getBitmapFromDiskCache(String key) {
    synchronized (mDiskCacheLock) {
        // Wait while disk cache is started from background thread
        while (mDiskCacheStarting) {
            try {
                mDiskCacheLock.wait();
            } catch (InterruptedException e) {}
        }
        if (mDiskLruCache != null) {
            return mDiskLruCache.get(key);
        }
    }
    return null;
}

// Creates a unique subdirectory of the designated app cache directory. Tries to use external
// but if not mounted, falls back on internal storage.
public static File getDiskCacheDir(Context context, String uniqueName) {
    // Check if media is mounted or storage is built-in, if so, try and use external cache dir
    // otherwise use internal cache dir
    final String cachePath =
            Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState()) ||
                    !isExternalStorageRemovable() ? getExternalCacheDir(context).getPath() :
                            context.getCacheDir().getPath();

    return new File(cachePath + File.separator + uniqueName);
}
```

> **Note**:因為初始化磁盤緩存涉及到I/O操作，所以它不應該在主線程中進行。但是這也意味著在初始化完成之前緩存可以被訪問。為了解決這個問題，在上面的實現中，有一個鎖對象（lock object）來確保在磁盤緩存完成初始化之前，應用無法對它進行讀取。

內存緩存的檢查是可以在UI線程中進行的，磁盤緩存的檢查需要在後臺線程中處理。磁盤操作永遠都不應該在UI線程中發生。當圖片處理完成後，Bitmap需要添加到內存緩存與磁盤緩存中，方便之後的使用。

## 處理配置改變(Handle Configuration Changes)

如果運行時設備配置信息發生改變，例如屏幕方向的改變會導致Android中當前顯示的Activity先被銷燬然後重啟。（關於這一方面的更多信息，請參考[Handling Runtime Changes](http://developer.android.com/guide/topics/resources/runtime-changes.html)）。我們需要在配置改變時避免重新處理所有的圖片，這樣才能提供給用戶一個良好的平滑過度的體驗。

幸運的是，在前面介紹使用內存緩存的部分，我們已經知道了如何建立內存緩存。這個緩存可以通過調用[setRetainInstance(true)](http://developer.android.com/reference/android/app/Fragment.html#setRetainInstance(boolean))保留一個[Fragment](http://developer.android.com/reference/android/app/Fragment.html)實例的方法把緩存傳遞給新的Activity。在這個Activity被重新創建之後，這個保留的Fragment會被重新附著上。這樣你就可以訪問緩存對象了，從緩存中獲取到圖片信息並快速的重新顯示到ImageView上。

下面是配置改變時使用Fragment來保留LruCache的代碼示例：

```java
private LruCache<String, Bitmap> mMemoryCache;

@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    RetainFragment retainFragment =
            RetainFragment.findOrCreateRetainFragment(getFragmentManager());
    mMemoryCache = retainFragment.mRetainedCache;
    if (mMemoryCache == null) {
        mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
            ... // Initialize cache here as usual
        }
        retainFragment.mRetainedCache = mMemoryCache;
    }
    ...
}

class RetainFragment extends Fragment {
    private static final String TAG = "RetainFragment";
    public LruCache<String, Bitmap> mRetainedCache;

    public RetainFragment() {}

    public static RetainFragment findOrCreateRetainFragment(FragmentManager fm) {
        RetainFragment fragment = (RetainFragment) fm.findFragmentByTag(TAG);
        if (fragment == null) {
            fragment = new RetainFragment();
            fm.beginTransaction().add(fragment, TAG).commit();
        }
        return fragment;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setRetainInstance(true);
    }
}
```

為了測試上面的效果，可以嘗試在保留Fragment與沒有這樣做的情況下旋轉屏幕。我們會發現當保留緩存時，從內存緩存中重新繪製幾乎沒有延遲的現象。 內存緩存中沒有的圖片可能存儲在磁盤緩存中。如果兩個緩存中都沒有，則圖像會像平時正常流程一樣被處理。
