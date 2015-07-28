# 使用 Volley 傳輸網絡數據

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/volley/index.html>

`Volley` 是一個 HTTP 庫，它能夠幫助 Android app 更方便地執行網絡操作，最重要的是，它更快速高效。我們可以通過開源的 [AOSP](https://android.googlesource.com/platform/frameworks/volley) 倉庫獲取到 Volley 。

**YOU SHOULD ALSO SEE**

使用 Volley 來編寫一個 app，請參考[2013 Google I/O schedule app](https://github.com/google/iosched)。另外需要特別關注下面2個部分：

* [ImageLoader](https://github.com/google/iosched/blob/master/android/src/main/java/com/google/android/apps/iosched/util/ImageLoader.java)
* [BitmapCache](https://github.com/google/iosched/blob/master/android/src/main/java/com/google/android/apps/iosched/util/BitmapCache.java)

[**VIDEO - Volley: Easy,Fast Networking for Android**](https://developers.google.com/events/io/sessions/325304728)
***
Volley 有如下的優點：

* 自動調度網絡請求。
* 高併發網絡連接。
* 通過標準的 HTTP [cache coherence](https://en.wikipedia.org/wiki/Cache_coherence)（高速緩存一致性）緩存磁盤和內存透明的響應。
* 支持指定請求的優先級。
* 撤銷請求 API。我們可以取消單個請求，或者指定取消請求隊列中的一個區域。
* 框架容易被定製，例如，定製重試或者回退功能。
* 強大的指令（Strong ordering）可以使得異步加載網絡數據並正確地顯示到 UI 的操作更加簡單。
* 包含了調試與追蹤工具。

Volley 擅長執行用來顯示 UI 的 RPC 類型操作，例如獲取搜索結果的數據。它輕鬆的整合了任何協議，並輸出操作結果的數據，可以是原始的字符串，也可以是圖片，或者是 JSON。通過提供內置的我們可能使用到的功能，Volley 可以使得我們免去重複編寫樣板代碼，使我們可以把關注點放在 app 的功能邏輯上。

Volley 不適合用來下載大的數據文件。因為 Volley 會保持在解析的過程中所有的響應。對於下載大量的數據操作，請考慮使用 [DownloadManager](http://developer.android.com/reference/android/app/DownloadManager.html)。

Volley 框架的核心代碼是託管在 AOSP 倉庫的 `frameworks/volley` 中，相關的工具放在 `toolbox` 下。把 Volley 添加到項目中最簡便的方法是 Clone 倉庫，然後把它設置為一個 library project：

1. 通過下面的命令來Clone倉庫：

    ```
    git clone https://android.googlesource.com/platform/frameworks/volley
    ```

2. 以一個 Android library project 的方式導入下載的源代碼到你的項目中。(如果你使用 Eclipse，請參考 <a href="http://developer.android.com/tools/projects/projects-eclipse.html)">Managing Projects from Eclipse with ADT</a>，或者編譯成一個 `.jar` 文件。

## Lessons

[**發送一個簡單的網絡請求(Sending a Simple Request)**](simple.html)

  學習如何通過 Volley 默認的行為發送一個簡單的請求，以及如何取消一個請求。

[**建立一個請求隊列(Setting Up a RequestQueue)**](request-queue.html)

  學習如何建立一個請求隊列（`RequestQueue`），以及如何實現一個單例模式來創建一個請求隊列，使 `RequestQueue` 能夠持續保持在我們 app 的生命週期中。

[**生成一個標準的請求(Making a Standard Request)**](request.html)

  學習如何使用 Volley 的 out-of-the-box（可直接使用、無需配置）請求類型（原始字符串、圖片和 JSON）來發送一個請求。

[**實現自定義的請求(Implementing a Custom Request)**](request-custom.html)

  學習如何實現一個自定義的請求。

