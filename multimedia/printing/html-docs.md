# 打印HTML文檔

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/printing/html-docs.html>

如果要在Android上打印比一副照片更豐富的內容，我們需要將文本和圖片組合在一個待打印的文檔中。Android框架提供了一種使用HTML語言來構建文檔並進行打印的方法，它使用的代碼數量是很小的。

[WebView](http://developer.android.com/reference/android/webkit/WebView.html)類在Android 4.4（API Level 19）中得到了更新，使得它可以打印HTML內容。該類允許我們加載一個本地HTML資源或者從網頁下載一個頁面，創建一個打印任務，並把它交給Android打印服務。

這節課將展示如何快速地構建一個包含有文本和圖片的HTML文檔，以及如何使用[WebView](http://developer.android.com/reference/android/webkit/WebView.html)打印該文檔。

## 加載一個HTML文檔

用[WebView](http://developer.android.com/reference/android/webkit/WebView.html)打印一個HTML文檔，會涉及到加載一個HTML資源，或者用一個字符串構建HTML文檔。這一節將描述如何構建一個HTML的字符串並將它加載到[WebView](http://developer.android.com/reference/android/webkit/WebView.html)中，以備打印。

該View對象一般被用來作為一個Activity佈局的一部分。然而，如果應用當前並沒有使用[WebView](http://developer.android.com/reference/android/webkit/WebView.html)，我們可以創建一個該類的實例，以進行打印。創建該自定義View的主要步驟是：
1. 在HTML資源加載完畢後，創建一個[WebViewClient](http://developer.android.com/reference/android/webkit/WebViewClient.html)用來啟動一個打印任務。
2. 加載HTML資源至[WebView](http://developer.android.com/reference/android/webkit/WebView.html)對象中。

下面的代碼展示瞭如何創建一個簡單的[WebViewClient](http://developer.android.com/reference/android/webkit/WebViewClient.html)並且加載一個動態創建的HTML文檔：

```java
private WebView mWebView;

private void doWebViewPrint() {
    // Create a WebView object specifically for printing
    WebView webView = new WebView(getActivity());
    webView.setWebViewClient(new WebViewClient() {

            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                return false;
            }

            @Override
            public void onPageFinished(WebView view, String url) {
                Log.i(TAG, "page finished loading " + url);
                createWebPrintJob(view);
                mWebView = null;
            }
    });

    // Generate an HTML document on the fly:
    String htmlDocument = "<html><body><h1>Test Content</h1><p>Testing, " +
            "testing, testing...</p></body></html>";
    webView.loadDataWithBaseURL(null, htmlDocument, "text/HTML", "UTF-8", null);

    // Keep a reference to WebView object until you pass the PrintDocumentAdapter
    // to the PrintManager
    mWebView = webView;
}
```

> **Note：**
請確保在[WebViewClient](http://developer.android.com/reference/android/webkit/WebViewClient.html#onPageFinished(android.webkit.WebView, java.lang.String))中的<a href="http://developer.android.com/reference/android/webkit/WebViewClient.html#onPageFinished(android.webkit.WebView, java.lang.String)">onPageFinished()</a>方法內調用創建打印任務的方法。如果沒有等到頁面加載完畢就進行打印，打印的輸出可能會不完整或空白，甚至可能會失敗。

> **Note：**在上面的樣例代碼中，保留了一個[WebView](http://developer.android.com/reference/android/webkit/WebView.html)對象實例的引用，這樣能夠確保它不會在打印任務創建之前就被垃圾回收器所回收。在編寫代碼時請務必這樣做，否則打印的進程可能會無法繼續執行。

如果我們希望頁面中包含圖像，將這個圖像文件放置在你的工程的“assets/”目錄中，並指定一個基URL（Base URL），並將它作為<a href="http://developer.android.com/reference/android/webkit/WebView.html#loadDataWithBaseURL(java.lang.String, java.lang.String, java.lang.String, java.lang.String, java.lang.String)">loadDataWithBaseURL()</a>方法的第一個參數，就像下面所顯示的一樣：

```java
webView.loadDataWithBaseURL("file:///android_asset/images/", htmlBody,
        "text/HTML", "UTF-8", null);
```

我們也可以加載一個需要打印的網頁，具體做法是將<a href="http://developer.android.com/reference/android/webkit/WebView.html#loadDataWithBaseURL(java.lang.String, java.lang.String, java.lang.String, java.lang.String, java.lang.String)">loadDataWithBaseURL()</a>方法替換為<a href="http://developer.android.com/reference/android/webkit/WebView.html#loadUrl(java.lang.String)">loadUrl()</a>，如下所示：

```java
// Print an existing web page (remember to request INTERNET permission!):
webView.loadUrl("http://developer.android.com/about/index.html");
```

當使用[WebView](http://developer.android.com/reference/android/webkit/WebView.html)創建打印文檔時，你要注意下面的一些限制：
* 不能為文檔添加頁眉和頁腳，包括頁號。
* HTML文檔的打印選項不包含選擇打印的頁數範圍，例如：對於一個10頁的HTMl文檔，只打印2到4頁是不可以的。
* 一個[WebView](http://developer.android.com/reference/android/webkit/WebView.html)的實例只能在同一時間處理一個打印任務。
* 若一個HTML文檔包含CSS打印屬性，比如一個landscape屬性，這是不被支持的。
* 不能通過一個HTML文檔中的JavaScript腳本來激活打印。

> **Note：**一旦在佈局中包含的[WebView](http://developer.android.com/reference/android/webkit/WebView.html)對象將文檔加載完畢後，就可以打印[WebView](http://developer.android.com/reference/android/webkit/WebView.html)對象的內容了。

如果希望創建一個更加自定義化的打印輸出並希望可以完全控制打印頁面上繪製的內容，可以學習下一節課程：[打印自定義文檔](custom-docs.html)

## 創建一個打印任務

在創建了[WebView](http://developer.android.com/reference/android/webkit/WebView.html)並加載了我們的HTML內容之後，應用就已經幾乎完成了屬於它的任務。接下來，我們需要訪問[PrintManager](http://developer.android.com/reference/android/print/PrintManager.html)，創建一個打印適配器，並在最後創建一個打印任務。下面的代碼展示瞭如何執行這些步驟：

```java
private void createWebPrintJob(WebView webView) {

    // Get a PrintManager instance
    PrintManager printManager = (PrintManager) getActivity()
            .getSystemService(Context.PRINT_SERVICE);

    // Get a print adapter instance
    PrintDocumentAdapter printAdapter = webView.createPrintDocumentAdapter();

    // Create a print job with name and adapter instance
    String jobName = getString(R.string.app_name) + " Document";
    PrintJob printJob = printManager.print(jobName, printAdapter,
            new PrintAttributes.Builder().build());

    // Save the job object for later status checking
    mPrintJobs.add(printJob);
}
```

這個例子保存了一個[PrintJob](http://developer.android.com/reference/android/print/PrintJob.html)對象的實例，以供我們的應用將來使用，當然這是不必須的。我們的應用可以使用這個對象來跟蹤打印任務執行時的進度。如果希望監控應用中的打印任務是否完成，是否失敗或者是否被用戶取消，這個方法非常有用。另外，我們不需要創建一個應用內置的通知，因為打印框架會自動的創建一個該打印任務的系統通知。
