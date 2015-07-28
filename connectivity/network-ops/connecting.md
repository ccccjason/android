# 連接到網絡

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/basics/network-ops/connecting.html>

這一課會演示如何實現一個簡單的連接到網絡的程序。它提供了一些我們在創建即使最簡單的網絡連接程序時也應該遵循的最佳示例。

請注意，想要執行本課的網絡操作首先需要在程序的 manifest 文件中添加以下權限：

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

## 選擇一個 HTTP Client

大多數連接網絡的 Android app 會使用 HTTP 來發送與接收數據。Android 提供了兩種 HTTP clients：[HttpURLConnection](http://developer.android.com/reference/java/net/HttpURLConnection.html) 與 Apache [HttpClient](http://developer.android.com/reference/org/apache/http/client/HttpClient.html)。二者均支持 HTTPS、流媒體上傳和下載、可配置的超時、IPv6 與連接池（connection pooling）。**對於 Android 2.3 Gingerbread 或更高的版本，推薦使用 HttpURLConnection**。關於這部分的更多詳情，請參考 [Android's HTTP Clients](http://android-developers.blogspot.com/2011/09/androids-http-clients.html)。

## 檢查網絡連接

在我們的 app 嘗試連接網絡之前，應通過函數 <a href="http://developer.android.com/reference/android/net/ConnectivityManager.html#getActiveNetworkInfo()">getActiveNetworkInfo()</a> 和 <a href="http://developer.android.com/reference/android/net/NetworkInfo.html#isConnected()">isConnected()</a> 檢測當前網絡是否可用。請注意，設備可能不在網絡覆蓋範圍內，或者用戶可能關閉 Wi-Fi 與移動網絡連接。關於這部分的更多詳情，請參考[管理網絡的使用情況](managing.html)

```java
public void myClickHandler(View view) {
    ...
    ConnectivityManager connMgr = (ConnectivityManager)
        getSystemService(Context.CONNECTIVITY_SERVICE);
    NetworkInfo networkInfo = connMgr.getActiveNetworkInfo();
    if (networkInfo != null && networkInfo.isConnected()) {
        // fetch data
    } else {
        // display error
    }
    ...
}
```

## 在一個單獨的線程中執行網絡操作

網絡操作會遇到不可預期的延遲。為了避免造成不好的用戶體驗，總是在 UI 線程之外單獨的線程中執行網絡操作。[AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) 類提供了一種簡單的方式來處理這個問題。這部分的詳情，請參考 [Multithreading For Performance](http://android-developers.blogspot.com/2010/07/multithreading-for-performance.html)。

在下面的代碼示例中，`myClickHandler()` 方法會執行 `new DownloadWebpageTask().execute(stringUrl)`。`DownloadWebpageTask` 是 `AsyncTask` 的子類，它實現了下面兩個方法:

* [doInBackground()](http://developer.android.com/reference/android/os/AsyncTask.html) 執行 `downloadUrl()` 方法。它以網頁的 URL 作為參數，方法 `downloadUrl()` 獲取並處理網頁返回的數據。執行完畢後，返回一個結果字符串。
* [onPostExecute()](http://developer.android.com/reference/android/os/AsyncTask.html) 接收結果字符串並把它顯示到 UI 上。

```java
public class HttpExampleActivity extends Activity {
    private static final String DEBUG_TAG = "HttpExample";
    private EditText urlText;
    private TextView textView;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        urlText = (EditText) findViewById(R.id.myUrl);
        textView = (TextView) findViewById(R.id.myText);
    }

    // When user clicks button, calls AsyncTask.
    // Before attempting to fetch the URL, makes sure that there is a network connection.
    public void myClickHandler(View view) {
        // Gets the URL from the UI's text field.
        String stringUrl = urlText.getText().toString();
        ConnectivityManager connMgr = (ConnectivityManager)
            getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo networkInfo = connMgr.getActiveNetworkInfo();
        if (networkInfo != null && networkInfo.isConnected()) {
            new DownloadWebpageText().execute(stringUrl);
        } else {
            textView.setText("No network connection available.");
        }
    }

     // Uses AsyncTask to create a task away from the main UI thread. This task takes a
     // URL string and uses it to create an HttpUrlConnection. Once the connection
     // has been established, the AsyncTask downloads the contents of the webpage as
     // an InputStream. Finally, the InputStream is converted into a string, which is
     // displayed in the UI by the AsyncTask's onPostExecute method.
     private class DownloadWebpageText extends AsyncTask {
        @Override
        protected String doInBackground(String... urls) {

            // params comes from the execute() call: params[0] is the url.
            try {
                return downloadUrl(urls[0]);
            } catch (IOException e) {
                return "Unable to retrieve web page. URL may be invalid.";
            }
        }
        // onPostExecute displays the results of the AsyncTask.
        @Override
        protected void onPostExecute(String result) {
            textView.setText(result);
       }
    }
    ...
}
```

上面這段代碼的事件順序如下:

1. 當用戶點擊按鈕時調用 `myClickHandler()`，app 將指定的 URL 傳給 [AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) 的子類 `DownloadWebpageTask`。
2. [AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) 的 <a href="http://developer.android.com/reference/android/os/AsyncTask.html#doInBackground(Params...)">doInBackground()</a> 方法調用 `downloadUrl()` 方法。
3. `downloadUrl()` 方法以一個 URL 字符串作為參數，並用它創建一個 [URL](http://developer.android.com/reference/java/net/URL.html) 對象。
4. 這個 [URL](http://developer.android.com/reference/java/net/URL.html) 對象被用來創建一個 [HttpURLConnection](http://developer.android.com/reference/java/net/HttpURLConnection.html)。
5. 一旦建立連接，[HttpURLConnection](http://developer.android.com/reference/java/net/HttpURLConnection.html) 對象將獲取網頁的內容並得到一個 [InputStream](http://developer.android.com/reference/java/io/InputStream.html)。
6. [InputStream](http://developer.android.com/reference/java/io/InputStream.html) 被傳給 `readIt()` 方法，該方法將流轉換成字符串。
7. 最後，[AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) 的 <a href="http://developer.android.com/reference/android/os/AsyncTask.html#onPostExecute(Result)">onPostExecute()</a> 方法將字符串展示在 main activity 的 UI 上。

## 連接並下載數據

在執行網絡交互的線程裡面，我們可以使用 [HttpURLConnection](http://developer.android.com/reference/java/net/HttpURLConnection.html) 來執行一個 GET 類型的操作並下載數據。在調用 `connect()` 之後，我們可以通過調用 `getInputStream()` 來得到一個包含數據的 [InputStream](http://developer.android.com/reference/java/io/InputStream.html) 對象。

在下面的代碼示例中，<a href="http://developer.android.com/reference/android/os/AsyncTask.html#doInBackground(Params...)">doInBackground()</a> 方法會調用 `downloadUrl()`。這個 `downloadUrl()` 方法使用給予的 URL，通過 [HttpURLConnection](http://developer.android.com/reference/java/net/HttpURLConnection.html) 連接到網絡。一旦建立連接後，app 就會使用 `getInputStream()` 來獲取包含數據的 [InputStream](http://developer.android.com/reference/java/io/InputStream.html)。

```java
// Given a URL, establishes an HttpUrlConnection and retrieves
// the web page content as a InputStream, which it returns as
// a string.
private String downloadUrl(String myurl) throws IOException {
    InputStream is = null;
    // Only display the first 500 characters of the retrieved
    // web page content.
    int len = 500;

    try {
        URL url = new URL(myurl);
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
        conn.setReadTimeout(10000 /* milliseconds */);
        conn.setConnectTimeout(15000 /* milliseconds */);
        conn.setRequestMethod("GET");
        conn.setDoInput(true);
        // Starts the query
        conn.connect();
        int response = conn.getResponseCode();
        Log.d(DEBUG_TAG, "The response is: " + response);
        is = conn.getInputStream();

        // Convert the InputStream into a string
        String contentAsString = readIt(is, len);
        return contentAsString;

    // Makes sure that the InputStream is closed after the app is
    // finished using it.
    } finally {
        if (is != null) {
            is.close();
        }
    }
}
```

請注意，`getResponseCode()` 會返回連接的狀態碼（status code）。這是一種獲知額外網絡連接信息的有效方式。其中，狀態碼是 200 則意味著連接成功。

## 將輸入流（InputStream）轉換為字符串

[InputStream](http://developer.android.com/reference/java/io/InputStream.html) 是一種可讀的 byte 數據源。如果我們獲得了一個 [InputStream](http://developer.android.com/reference/java/io/InputStream.html)，通常會進行解碼（decode）或者轉換為目標數據類型。例如，如果我們是在下載圖片數據，那麼可能需要像下面這樣解碼並展示它：

```java
InputStream is = null;
...
Bitmap bitmap = BitmapFactory.decodeStream(is);
ImageView imageView = (ImageView) findViewById(R.id.image_view);
imageView.setImageBitmap(bitmap);
```

在上面演示的示例中，[InputStream](http://developer.android.com/reference/java/io/InputStream.html) 包含的是網頁的文本內容。下面會演示如何把 [InputStream](http://developer.android.com/reference/java/io/InputStream.html) 轉換為字符串，以便顯示在 UI 上。

```java
// Reads an InputStream and converts it to a String.
public String readIt(InputStream stream, int len) throws IOException, UnsupportedEncodingException {
    Reader reader = null;
    reader = new InputStreamReader(stream, "UTF-8");
    char[] buffer = new char[len];
    reader.read(buffer);
    return new String(buffer);
}
```
