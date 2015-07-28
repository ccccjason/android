# 創建可穿戴的應用

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/wearables/apps/index.html>

可穿戴應用直接運行在穿戴設備上，應用可以直接訪問例如傳感器與GPU這樣的硬件。這些應用和一般的Android應用的基礎部分是一致的，只是在設計與可用性還有一些特殊功能上有比較大差異。手持設備與可穿戴設備上的應用主要有下面的一些差異：

* 系統會強制執行超時機制。如果我們顯示了一個Activity，用戶並沒有進行操作，設備會進入睡眠狀態。當設備喚醒時，穿戴設備會顯示主界面而不是剛才的activity。如果我們想要持續的顯示一些東西，請使用notification來替代。
* 相比起手持設備的應用，可穿戴應用的界面相對更小，功能也相對更少。他僅僅包含了那些對於可穿戴有意義的功能，這些功能通常是手持設備的一個子集。通常來說，我們應該儘可能的把運行操作搬到手持設備上，然後發送操作結果到可穿戴設備。
* 用戶不會直接將應用下載到可穿戴設備上進行安裝。相反，我們將可穿戴設備應用打包到手持設備應用裡。當用戶安裝手持設備的應用時，系統會自動安裝可穿戴應用。然而，為了開發便利，我們還是可以直接安裝應用到可穿戴設備。
* 可穿戴應用可以使用大多數的標準Android APIs，除了下面的以外：
    * [android.webkit](http://developer.android.com/reference/android/webkit/package-summary.html)
    * [android.print](http://developer.android.com/reference/android/print/package-summary.html)
    * [android.app.backup](http://developer.android.com/reference/android/app/backup/package-summary.html)
    * [android.appwidget](http://developer.android.com/reference/android/appwidget/package-summary.html)
    * [android.hardware.usb](http://developer.android.com/reference/android/hardware/usb/package-summary.html)

  在使用某個API之前，我們可以通過執行[hasSystemFeature()](http://developer.android.com/reference/android/content/pm/PackageManager.html#hasSystemFeature(java.lang.String)) 來判斷可穿戴應用是否支持某個功能。

> **Note:** 我們推薦使用Android Studio來開發Android Wear的應用，因為它提供了建立工程，添加庫依賴，打包程序等在ADT上沒有的功能。下面的培訓課程的前提是假設你已經在使用Android Studio了。

## Lessons
* [創建並運行可穿戴應用(Creating and Running a Wearable App)](creating.html)

  學習如何創建一個包含了可穿戴與手持應用的Android Studio工程。學習如何在設備或者模擬器上運行應用。


* [創建自定義的佈局(Creating Custom Layouts)](layouts.html)

  學習如何為notification與activity創建並顯示一個自定義的佈局


* [添加語音功能(Adding Voice Capabilities)](voice.html)

  學習如何使用語音指令啟動一個activity，學習如何啟動系統語音識別應用來獲取用戶的語音輸入。


* [打包可穿戴應用(Packaging Wearable Apps)](packaging.html)

  學習如何把可穿戴應用打包到手持應用上。這使得系統能夠在安裝Google Play商店上的手持應用時自動安裝可穿戴應用。


* [通過藍牙進行調試(Debugging over Bluetooth)](bt-debugging.html)

  學習如何通過藍牙而不是USB來調試可穿戴應用。

