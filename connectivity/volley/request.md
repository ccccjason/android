# 創建標準的網絡請求

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/volley/request.html>

這一課會介紹如何使用 Volley 支持的常用請求類型：

* `StringRequest`。指定一個 URL 並在響應回調中接收一個原始的字符串數據。請參考前一課的示例。
* `ImageRequest`。指定一個 URL 並在響應回調中接收一個圖片。
* `JsonObjectRequest` 與 `JsonArrayRequest`（均為 `JsonRequest` 的子類）。指定一個 URL 並在響應回調中獲取到一個 JSON 對象或者 JSON 數組。

如果我們需要的是上面演示的請求類型，那麼我們很可能不需要實現一個自定義的請求。這節課會演示如何使用那些標準的請求類型。關於如何實現自定義的請求，請看下一課：[實現自定義的請求](request-costom.html)。

## 請求一張圖片

Volley 為請求圖片提供瞭如下的類。這些類依次有著依賴關係，用來支持在不同的層級進行圖片處理：

* `ImageRequest` —— 一個封裝好的，用來處理 URL 請求圖片並且返回一張解完碼的位圖（bitmap）。它同樣提供了一些簡便的接口方法，例如指定一個大小進行重新裁剪。它的主要好處是 Volley 會確保類似 decode，resize 等耗時的操作在工作線程中執行。

* `ImageLoader` —— 一個用來處理加載與緩存從網絡上獲取到的圖片的幫助類。`ImageLoader` 是大量 `ImageRequest` 的協調器。例如，在 [`ListView`](http://developer.android.com/reference/android/widget/ListView.html) 中需要顯示大量縮略圖的時候。`ImageLoader` 為通常的 Volley cache 提供了更加前瞻的內存緩存，這個緩存對於防止圖片抖動非常有用。這還使得在不阻塞或者延遲主線程的前提下實現緩存命中（這對於使用磁盤 I/O 是無法實現的）。`ImageLoader` 還能夠實現響應聯合（response coalescing），避免幾乎每一個響應回調裡面都設置 bitmap 到 view 上面。響應聯合使得能夠同時提交多個響應，這提升了性能。

* `NetworkImageView` —— 在 `ImageLoader` 的基礎上建立，並且在通過網絡 URL 取回的圖片的情況下，有效地替換 `ImageView`。如果 view 從層次結構中分離，`NetworkImageView` 也可以管理取消掛起請求。

### 使用 ImageRequest

下面是一個使用 `ImageRequest` 的示例。它會獲取 URL 上指定的圖片並顯示到 app 上。注意到，裡面演示的 `RequestQueue` 是通過上一課提到的單例類實現的：

```java
ImageView mImageView;
String url = "http://i.imgur.com/7spzG.png";
mImageView = (ImageView) findViewById(R.id.myImage);
...

// Retrieves an image specified by the URL, displays it in the UI.
ImageRequest request = new ImageRequest(url,
    new Response.Listener() {
        @Override
        public void onResponse(Bitmap bitmap) {
            mImageView.setImageBitmap(bitmap);
        }
    }, 0, 0, null,
    new Response.ErrorListener() {
        public void onErrorResponse(VolleyError error) {
            mImageView.setImageResource(R.drawable.image_load_error);
        }
    });
// Access the RequestQueue through your singleton class.
MySingleton.getInstance(this).addToRequestQueue(request);
```

### 使用 ImageLoader 和 NetworkImageView

我們可以使用 `ImageLoader` 與 `NetworkImageView` 來有效地管理類似 ListView 等顯示多張圖片的情況。在 layout XML 文件中，我們以與使用 [ImageView](http://developer.android.com/reference/android/widget/ImageView.html) 差不多的方法使用 `NetworkImageView`，例如:

```xml
<com.android.volley.toolbox.NetworkImageView
        android:id="@+id/networkImageView"
        android:layout_width="150dp"
        android:layout_height="170dp"
        android:layout_centerHorizontal="true" />
```

我們可以使用 `ImageLoader` 自身來顯示一張圖片，例如：

```java
ImageLoader mImageLoader;
ImageView mImageView;
// The URL for the image that is being loaded.
private static final String IMAGE_URL =
    "http://developer.android.com/images/training/system-ui.png";
...
mImageView = (ImageView) findViewById(R.id.regularImageView);

// Get the ImageLoader through your singleton class.
mImageLoader = MySingleton.getInstance(this).getImageLoader();
mImageLoader.get(IMAGE_URL, ImageLoader.getImageListener(mImageView,
         R.drawable.def_image, R.drawable.err_image));
```

然而，如果我們要做的是為 `ImageView` 進行圖片設置，那麼我們可以使用 `NetworkImageView` 來實現，例如：

```java
ImageLoader mImageLoader;
NetworkImageView mNetworkImageView;
private static final String IMAGE_URL =
    "http://developer.android.com/images/training/system-ui.png";
...

// Get the NetworkImageView that will display the image.
mNetworkImageView = (NetworkImageView) findViewById(R.id.networkImageView);

// Get the ImageLoader through your singleton class.
mImageLoader = MySingleton.getInstance(this).getImageLoader();

// Set the URL of the image that should be loaded into this view, and
// specify the ImageLoader that will be used to make the request.
mNetworkImageView.setImageUrl(IMAGE_URL, mImageLoader);
```

上面的代碼是通過通過前一節課講到的單例類來訪問 `RequestQueue` 與 `ImageLoader`。這種方法保證了我們的 app 創建這些類的單例會持續存在於 app 的生命週期。這對於 `ImageLoader`（一個用來處理加載與緩存圖片的幫助類）很重要的原因是：內存緩存的主要功能是允許非抖動旋轉。使用單例模式可以使得 bitmap 的緩存比 activity 存在的時間長。如果我們在 activity 中創建 `ImageLoader`，這個 `ImageLoader` 有可能會在每次旋轉設備的時候都被重新創建。這可能會導致抖動。

#### 舉一個 LRU cache 的例子

Volley 工具箱中提供了一種通過 `DiskBasedCache` 類實現的標準緩存。這個類能夠緩存文件到磁盤的指定目錄。但是為了使用 `ImageLoader`，我們應該提供一個自定義的內存 LRC bitmap 緩存，這個緩存實現了 `ImageLoader.ImageCache` 接口。我們可能想把緩存設置成一個單例。關於更多的有關內容，請參考[建立請求隊列](request.html).

下面是一個內存 `LruBitmapCache` 類的實現示例。它繼承 [LruCache](http://developer.android.com/reference/android/support/v4/util/LruCache.html) 類並實現了 `ImageLoader.ImageCache` 接口：

```java
import android.graphics.Bitmap;
import android.support.v4.util.LruCache;
import android.util.DisplayMetrics;
import com.android.volley.toolbox.ImageLoader.ImageCache;

public class LruBitmapCache extends LruCache<String, Bitmap>
        implements ImageCache {

    public LruBitmapCache(int maxSize) {
        super(maxSize);
    }

    public LruBitmapCache(Context ctx) {
        this(getCacheSize(ctx));
    }

    @Override
    protected int sizeOf(String key, Bitmap value) {
        return value.getRowBytes() * value.getHeight();
    }

    @Override
    public Bitmap getBitmap(String url) {
        return get(url);
    }

    @Override
    public void putBitmap(String url, Bitmap bitmap) {
        put(url, bitmap);
    }

    // Returns a cache size equal to approximately three screens worth of images.
    public static int getCacheSize(Context ctx) {
        final DisplayMetrics displayMetrics = ctx.getResources().
                getDisplayMetrics();
        final int screenWidth = displayMetrics.widthPixels;
        final int screenHeight = displayMetrics.heightPixels;
        // 4 bytes per pixel
        final int screenBytes = screenWidth * screenHeight * 4;

        return screenBytes * 3;
    }
}
```

下面是如何實例化一個 `ImageLoader` 來使用這個 cache:

```java
RequestQueue mRequestQueue; // assume this exists.
ImageLoader mImageLoader = new ImageLoader(mRequestQueue, new LruBitmapCache(LruBitmapCache.getCacheSize()));
```

## 請求 JSON

Volley 提供了以下的類用來執行 JSON 請求：

* `JsonArrayRequest` —— 一個為了獲取給定 URL 的 [JSONArray](http://developer.android.com/reference/org/json/JSONArray.html) 響應正文的請求。
* `JsonObjectRequest` —— 一個為了獲取給定 URL 的 [JSONObject](http://developer.android.com/reference/org/json/JSONObject.html) 響應正文的請求。允許傳進一個可選的 [JSONObject](http://developer.android.com/reference/org/json/JSONObject.html) 作為請求正文的一部分。

這兩個類都是基於一個公共基類 `JsonRequest`。我們遵循我們在其它請求類型使用的同樣的基本模式來使用這些類。如下演示瞭如果獲取一個 JSON feed 並顯示到 UI 上：

```java
TextView mTxtDisplay;
ImageView mImageView;
mTxtDisplay = (TextView) findViewById(R.id.txtDisplay);
String url = "http://my-json-feed";

JsonObjectRequest jsObjRequest = new JsonObjectRequest
        (Request.Method.GET, url, null, new Response.Listener() {

    @Override
    public void onResponse(JSONObject response) {
        mTxtDisplay.setText("Response: " + response.toString());
    }
}, new Response.ErrorListener() {

    @Override
    public void onErrorResponse(VolleyError error) {
        // TODO Auto-generated method stub

    }
});

// Access the RequestQueue through your singleton class.
MySingleton.getInstance(this).addToRequestQueue(jsObjRequest);
```

關於基於 [Gson](http://code.google.com/p/google-gson/) 實現一個自定義的 JSON 請求對象，請參考下一節課：[實現一個自定義的請求](request-custom.html)。
