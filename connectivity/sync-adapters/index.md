# 使用Sync Adapter傳輸數據

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/sync-adapters/index.html>

如果我們的應用允許 Android 設備和網絡服務器之間進行數據同步，那麼它無疑將變得更加實用，更加吸引用戶的注意。例如，將數據傳輸到服務器可以實現數據的備份，另一方面，從服務器獲取數據可以讓用戶隨時隨地都能使用我們的應用。有時候，用戶可能會覺得在線編輯他們的數據並將其發送到設備上，會是一件很方便的事情；或者他們有時會希望將收集到的數據上傳到一個統一的存儲區域中。

儘管我們可以設計一套自己的系統來實現應用中的數據傳輸，但我們也可以考慮一下使用 Android 的同步適配器框架（Android's Sync Adapter Framework）。該框架可以用來幫助管理數據，自動傳輸數據，以及協調不同應用間的同步問題。當使用這個框架時，我們可以利用它的一些特性，而這些特性可能是我們自己設計的傳輸方案中所沒有的：

**插件架構（Plug-in Architecture）：**

允許我們以可調用組件的形式，將傳輸代碼添加到系統中。

**自動執行（Automated Execution）：**

允許我們基於不同的準則自動地執行數據傳輸，比如：當數據變更時，或者每隔固定一段時間，亦或者每天，來自動執行一次數據傳輸。另外，系統會自動把當前無法執行的傳輸添加到一個隊列中，並且在合適的時候運行它們。

**自動網絡監測（Automated Network Checking）：**

系統只在有網絡連接的時候才會運行數據傳輸。

**提升電池使用效率：**

允許我們將所有的數據傳輸任務統一地進行一次性批量傳輸，這樣的話多個數據傳輸任務會在同一段時間內運行。我們應用的數據傳輸任務也會和其它應用的傳輸任務相結合，並一起傳輸。這樣做可以減少系統連接網絡的次數，進而減少電量的使用。

**賬戶管理和授權：**

如果我們的應用需要用戶登錄授權，那麼我們可以將賬戶管理和授權的功能集成到數據傳輸組件中。

本系列課程將展示如何創建一個 Sync Adapter，如何創建一個綁定了 Sync Adapter 的服務（[Service](http://developer.android.com/reference/android/app/Service.html)），如何提供其它組件來幫助我們將 Sync Adapter 集成到框架中，以及如何通過不同的方法來運行 Sync Adapter。

> **Note：**Sync Adapter 是異步執行的，它可以定期且有效地傳輸數據，但在實時性上一般難以滿足要求。如果我們想要實時地傳輸數據，那麼應該在 [AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) 或 [IntentService](http://developer.android.com/reference/android/app/IntentService.html) 中完成這一任務。

## Sample Code

[BasicSyncAdapter.zip](http://developer.android.com/shareables/training/BasicSyncAdapter.zip)

## Lessons

[創建 Stub 授權器](create-authenticator.html)

  學習如何在我們的應用中添加一個 Sync Adapter 框架需要的賬戶處理組件。這節課將展示如何簡單地創建一個 Stub Authenticator 組件。

[創建 Stub Content Provider](create-stub-provider.html)

  學習如何在我們的應用中添加一個 Sync Adapter 框架需要的 Content Provider 組件。在這節課中，假設我們的應用實際上不需要使用 Content Provider，所以它將教我們如何添加一個 Stub 組件。如果我們的應用已經有了一個 Content Provider 組件，那麼可以跳過這節課。

[創建 Sync Adapter](create-sync-adapter.html)

  學習如何將我們的數據傳輸代碼封裝到組件當中，並讓其可以被 Sync Adapter 框架自動執行。

[執行 Sync Adapter](running-sync-adapter.html)

  學習如何使用 Sync Adapter 框架激活並調度數據傳輸。
