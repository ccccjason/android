# 執行網絡操作

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/basics/network-ops/index.html>

這一章會介紹一些基本的網絡操作，涉及到網絡連接、監視網絡連接（包括網絡改變）和讓用戶控制 app 的網絡用途。還會介紹如何解析與使用 XML 數據。

這節課包括一個示例應用，展示如何執行常見的網絡操作。我們可以下載下面的的範例，並把它作為可重用代碼在自己的應用中使用。

[NetworkUsage.zip](http://developer.android.com/shareables/training/NetworkUsage.zip)

通過學習這章節的課程，我們將會學習到一些有關於如何創建一個使用最少的網絡流量下載並解析數據的高效 app 的基礎知識。

你還可以參考下面文章進階學習:

* [Optimizing Battery Life](performance/monitoring-device-state/index.html)
* [Transferring Data Without Draining the Battery](connectivity/efficient-downloads/index.html)
* [Web Apps Overview](http://developer.android.com/guide/webapps/index.html)
* [Transmitting Network Data Using Volley](connectivity/volley/index.md)

> **Node:** 查看[使用 Volley 傳輸網絡數據](connectivity/volley/index.md)課程獲取 Volley 的相關信息，它是一個能幫助 Android apps 更方便快捷地執行網絡操作的 HTTP 庫。Volly 可以在開源 [AOSP](https://android.googlesource.com/platform/frameworks/volley) 庫中找到。Volly 可能會幫助我們簡化網絡操作，提高我們 app 的網絡操作性能。

## Lessons

[連接到網絡 - Connecting to the Network](connecting.html)

  學習如何連接到網絡，選擇一個 HTTP client，以及在 UI 線程外執行網絡操作。


[管理網絡的使用情況 - Managing Network Usage](managing.html)

  學習如何檢查設備的網絡連接情況，創建偏好界面來控制網絡使用，以及響應連接變化。


[解析 XML 數據 - Parsing XML Data](xml.html)

  學習如何解析和使用 XML 數據。
