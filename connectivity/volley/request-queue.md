# 建立請求隊列（RequestQueue）

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/volley/requestqueue.html>

前一節課演示瞭如何使用 `Volley.newRequestQueue` 這一簡便的方法來建立一個`RequestQueue`，這是利用了 Volley 默認行為的優勢。這節課會介紹如何顯式地建立一個 `RequestQueue`，以便滿足我們自定義的需求。

這節課同樣會介紹一種推薦的實現方式：創建一個單例的 `RequestQueue`，這使得 `RequestQueue` 能夠持續保持在我們 app 的生命週期中。

## 建立網絡和緩存

一個 `RequestQueue` 需要兩部分來支持它的工作：一部分是網絡操作，用來傳輸請求，另外一個是用來處理緩存操作的 Cache。在 Volley 的工具箱中包含了標準的實現方式：`DiskBasedCache` 提供了每個文件與對應響應數據一一映射的緩存實現。 `BasicNetwork` 提供了一個基於 [AndroidHttpClient](http://developer.android.com/reference/android/net/http/AndroidHttpClient.html) 或者 [HttpURLConnection](http://developer.android.com/reference/java/net/HttpURLConnection.html) 的網絡傳輸。

`BasicNetwork` 是 Volley 默認的網絡操作實現方式。一個 `BasicNetwork` 必須使用我們的 app 用於連接網絡的 HTTP Client 進行初始化。這個 Client 通常是[AndroidHttpClient](http://developer.android.com/reference/android/net/http/AndroidHttpClient.html) 或者 [HttpURLConnection](http://developer.android.com/reference/java/net/HttpURLConnection.html)：

* 對於 app target API level 低於 API 9（Gingerbread）的使用 AndroidHttpClient。在 Gingerbread 之前，HttpURLConnection 是不可靠的。對於這個的細節，請參考 [Android's HTTP Clients](http://android-developers.blogspot.com/2011/09/androids-http-clients.html)。
* 對於 API Level 9 以及以上的，使用 HttpURLConnection。

我們可以通過檢查系統版本選擇合適的 HTTP Client，從而創建一個能夠運行在所有 Android 版本上的應用。例如：

```java
HttpStack stack;
...
// If the device is running a version >= Gingerbread...
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.GINGERBREAD) {
    // ...use HttpURLConnection for stack.
} else {
    // ...use AndroidHttpClient for stack.
}
Network network = new BasicNetwork(stack);
```

下面的代碼片段演示瞭如何一步步建立一個 `RequestQueue`:

```java
RequestQueue mRequestQueue;

// Instantiate the cache
Cache cache = new DiskBasedCache(getCacheDir(), 1024 * 1024); // 1MB cap

// Set up the network to use HttpURLConnection as the HTTP client.
Network network = new BasicNetwork(new HurlStack());

// Instantiate the RequestQueue with the cache and network.
mRequestQueue = new RequestQueue(cache, network);

// Start the queue
mRequestQueue.start();

String url ="http://www.myurl.com";

// Formulate the request and handle the response.
StringRequest stringRequest = new StringRequest(Request.Method.GET, url,
        new Response.Listener<String>() {
    @Override
    public void onResponse(String response) {
        // Do something with the response
    }
},
    new Response.ErrorListener() {
        @Override
        public void onErrorResponse(VolleyError error) {
            // Handle error
    }
});

// Add the request to the RequestQueue.
mRequestQueue.add(stringRequest);
...
```

如果我們僅僅是想做一個單次的請求並且不想要線程池一直保留，我們可以通過使用在前面一課：[發送一個簡單的請求（Sending a Simple Request）](simple.html)文章中提到的 `Volley.newRequestQueue()` 方法，在任何需要的時刻創建 `RequestQueue`，然後在我們的響應回調裡面執行 `stop()` 方法來停止操作。但是更通常的做法是創建一個 `RequestQueue` 並設置為一個單例。下面部分將演示這種做法。

## 使用單例模式

如果我們的應用需要持續地使用網絡，更加高效的方式應該是建立一個 `RequestQueue` 的單例，這樣它能夠持續保持在整個 app 的生命週期中。我們可以通過多種方式來實現這個單例。推薦的方式是實現一個單例類，裡面封裝了 `RequestQueue` 對象與其它的 Volley 功能。另外一個方法是繼承 [`Application`](http://developer.android.com/reference/android/app/Application.html) 類，並在 `Application.OnCreate()` 方法裡面建立 `RequestQueue`。但是我們並不推薦這個方法，因為一個 static 的單例能夠以一種更加模塊化的方式提供同樣的功能。

一個關鍵的概念是 `RequestQueue` 必須使用 Application context 來實例化，而不是 Activity context。這確保了 `RequestQueue` 在我們 app 的生命週期中一直存活，而不會因為 activity 的重新創建而被重新創建(例如，當用戶旋轉設備時)。

下面是一個單例類，提供了 `RequestQueue` 與 `ImageLoader` 功能：

```java
public class MySingleton {
    private static MySingleton mInstance;
    private RequestQueue mRequestQueue;
    private ImageLoader mImageLoader;
    private static Context mCtx;

    private MySingleton(Context context) {
        mCtx = context;
        mRequestQueue = getRequestQueue();

        mImageLoader = new ImageLoader(mRequestQueue,
                new ImageLoader.ImageCache() {
            private final LruCache<String, Bitmap>
                    cache = new LruCache<String, Bitmap>(20);

            @Override
            public Bitmap getBitmap(String url) {
                return cache.get(url);
            }

            @Override
            public void putBitmap(String url, Bitmap bitmap) {
                cache.put(url, bitmap);
            }
        });
    }

    public static synchronized MySingleton getInstance(Context context) {
        if (mInstance == null) {
            mInstance = new MySingleton(context);
        }
        return mInstance;
    }

    public RequestQueue getRequestQueue() {
        if (mRequestQueue == null) {
            // getApplicationContext() is key, it keeps you from leaking the
            // Activity or BroadcastReceiver if someone passes one in.
            mRequestQueue = Volley.newRequestQueue(mCtx.getApplicationContext());
        }
        return mRequestQueue;
    }

    public <T> void addToRequestQueue(Request<T> req) {
        getRequestQueue().add(req);
    }

    public ImageLoader getImageLoader() {
        return mImageLoader;
    }
}
```

下面演示了利用單例類來執行 `RequestQueue` 的操作：

```java
// Get a RequestQueue
RequestQueue queue = MySingleton.getInstance(this.getApplicationContext()).
    getRequestQueue();
...

// Add a request (in this example, called stringRequest) to your RequestQueue.
MySingleton.getInstance(this).addToRequestQueue(stringRequest);
```








