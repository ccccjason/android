# 實現自定義的網絡請求

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/volley/request-custom.html>

這節課會介紹如何實現自定義的請求類型，這些自定義的類型不屬於 Volley 內置支持包裡面。

## 編寫一個自定義請求

大多數的請求類型都已經包含在 Volley 的工具箱裡面。如果我們的請求返回數值是一個 string，image 或者 JSON，那麼是不需要自己去實現請求類的。

對於那些需要自定義的請求類型，我們需要執行以下操作：

* 繼承 `Request<T>` 類，`<T>` 表示解析過的響應請求預期的數據類型。因此如果我們需要解析的響應類型是一個 String，可以通過繼承 `Request<String>` 來創建自定義的請求。請參考 Volley 工具類中的 `StringRequest` 與 `ImageRequest` 來學習如何繼承 `Request<T>`。
* 實現抽象方法 `parseNetworkResponse()` 與 ` deliverResponse()`，下面會詳細介紹。

### parseNetworkResponse

一個 `Response` 封裝了用於發送的給定類型（例如，string、image、JSON等）解析過的響應。下面會演示如何實現 `parseNetworkResponse()`：

```java
@Override
protected Response<T> parseNetworkResponse(
        NetworkResponse response) {
    try {
        String json = new String(response.data,
        HttpHeaderParser.parseCharset(response.headers));
    return Response.success(gson.fromJson(json, clazz),
    HttpHeaderParser.parseCacheHeaders(response));
    }
    // handle errors
...
}
```

請注意：

* `parseNetworkResponse()` 的參數是類型是 `NetworkResponse`，這種參數以 byte[]、HTTP status code 以及 response headers 的形式包含響應負載。
* 我們實現的方法必須返回一個 `Response<T>`，它包含了我們指定類型的響應對象與緩存 metadata 或者是一個錯誤。

如果我們的協議沒有標準的緩存機制，那麼我們可以自己建立一個 `Cache.Entry`, 但是大多數請求都可以用下面的方式來處理:

```java
return Response.success(myDecodedObject,
        HttpHeaderParser.parseCacheHeaders(response));
```

Volley 在工作線程中執行 `parseNetworkResponse()` 方法。這確保了耗時的解析操作，例如 decode 一張 JPEG 圖片成 bitmap，不會阻塞 UI 線程。

### deliverResponse

Volley 會把 `parseNetworkResponse()` 方法返回的數據帶到主線程的回調中。如下所示：

```java
protected void deliverResponse(T response) {
        listener.onResponse(response);
```

### Example: GsonRequest

[Gson](http://code.google.com/p/google-gson/) 是一個使用映射支持 JSON 與 Java 對象之間相互轉換的庫文件。我們可以定義與 JSON keys 相對應名稱的 Java 對象。把對象傳遞給 Gson，然後 Gson 會幫我們為對象填充字段值。下面是一個完整的示例：演示了使用 Gson 解析 Volley 數據：

```java
public class GsonRequest<T> extends Request<T> {
    private final Gson gson = new Gson();
    private final Class<T> clazz;
    private final Map<String, String> headers;
    private final Listener<T> listener;

    /**
     * Make a GET request and return a parsed object from JSON.
     *
     * @param url URL of the request to make
     * @param clazz Relevant class object, for Gson's reflection
     * @param headers Map of request headers
     */
    public GsonRequest(String url, Class<T> clazz, Map<String, String> headers,
            Listener<T> listener, ErrorListener errorListener) {
        super(Method.GET, url, errorListener);
        this.clazz = clazz;
        this.headers = headers;
        this.listener = listener;
    }

    @Override
    public Map<String, String> getHeaders() throws AuthFailureError {
        return headers != null ? headers : super.getHeaders();
    }

    @Override
    protected void deliverResponse(T response) {
        listener.onResponse(response);
    }

    @Override
    protected Response<T> parseNetworkResponse(NetworkResponse response) {
        try {
            String json = new String(
                    response.data,
                    HttpHeaderParser.parseCharset(response.headers));
            return Response.success(
                    gson.fromJson(json, clazz),
                    HttpHeaderParser.parseCacheHeaders(response));
        } catch (UnsupportedEncodingException e) {
            return Response.error(new ParseError(e));
        } catch (JsonSyntaxException e) {
            return Response.error(new ParseError(e));
        }
    }
}
```

如果你願意使用的話，Volley 提供了現成的 `JsonArrayRequest` 與 ` JsonArrayObject`類。參考上一課[創建標準的網絡請求](request.html)。
