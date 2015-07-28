# 重複的下載是冗餘的

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/efficient-downloads/redundant_redundant.html>

減少下載的最基本方法是僅僅下載那些我們需要的。從數據的角度看，我們可以通過傳遞類似上次更新時間這樣的參數來制定查詢數據的條件。

同樣，在下載圖片的時候，server 那邊最好能夠減少圖片的大小，而不是讓我們下載完整大小的圖片。

## 緩存文件到本地

另一個重要的技術是避免下載重複的數據。可以使用緩存機制來處理這個問題。緩存靜態的資源，包括按需下載例如完整的圖片（只要合理和興）。這些緩存的資源需要分開存放，使得我們可以定期地清理這些緩存，從而控制緩存數據的大小。

為了保證 app 不會因為緩存而導致顯示的是舊數據，請在緩存中獲取數據的同時檢測其是否過期，當數據過期的時候，會提示進行刷新。

<!-- More -->

```java
long currentTime = System.currentTimeMillis();

HttpURLConnection conn = (HttpURLConnection) url.openConnection();

long expires = conn.getHeaderFieldDate("Expires", currentTime);
long lastModified = conn.getHeaderFieldDate("Last-Modified", currentTime);

setDataExpirationDate(expires);

if (lastModified < lastUpdateTime) {
  // Skip update
} else {
  // Parse update
}
```

使用這種方法，可以有效保證緩存裡面一直是最新的數據。

我們可以緩存非敏感數據到非受管的外部緩存目錄（目錄會是sdcard下面的`Android/data/data/com.xxx.xxx/cache`）：

```java
Context.getExternalCacheDir();
```

或者，我們可以使用受管/安全的應用緩存。請注意，當系統的可用存儲空間較小時，存放在內存中的數據有可能會被清除（類似:`system/data/data/com.xxx.xxx./cache`）。

```java
Context.getCache();
```

緩存在上面兩個地方的文件都會在 app 卸載的時候被清除。

**Ps：請注意這點:發現很多應用總是隨便在 sdcard 下面創建一個目錄用來存放緩存，可是這些緩存又不會隨著程序的卸載而被刪除，這其實是不符合規範，程序都被卸載了，為何還要留那麼多垃圾文件，而且這些文件有可能會洩漏一些隱私信息。除非你的程序是音樂下載，拍照程序等等，這些確定程序生成的文件是會被用戶需要留下的，不然都應該使用上面的那種方式來獲取 Cache 目錄。**

## 使用 HttpURLConnection 響應緩存

在 `Android 4.0` 裡面為 `HttpURLConnection` 增加了一個響應緩存（這是一個很好的減少 http 請求次數的機制，Android 官方推薦使用 HttpURLConnection 而不是 Apache 的 DefaultHttpClient，就是因為前者不僅僅有針對 android 做 http 請求的優化，還在4.0上增加了 Reponse Cache，這進一步提高了效率)。我們可以使用反射機制開啟 HTTP response cache，看下面的例子：

```java
private void enableHttpResponseCache() {
  try {
    long httpCacheSize = 10 * 1024 * 1024; // 10 MiB
    File httpCacheDir = new File(getCacheDir(), "http");
    Class.forName("android.net.http.HttpResponseCache")
         .getMethod("install", File.class, long.class)
         .invoke(null, httpCacheDir, httpCacheSize);
  } catch (Exception httpResponseCacheNotAvailable) {
    Log.d(TAG, "HTTP response cache is unavailable.");
  }
}
```

上面的示例代碼在 Android 4.0 以上的設備上會開啟 response cache，同時不會影響到之前的程序。

在cache被開啟之後，所有cache中的HTTP請求都可以直接在本地存儲中進行響應，並不需要開啟一個新的網絡連接。被cache起來的response可以被server所確保沒有過期，這樣就減少了下載所需的帶寬。

沒有被cached的response會為了方便下次請求而被存儲在response cache中。