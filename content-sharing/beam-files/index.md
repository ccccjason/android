# 使用NFC分享文件

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/beam-files/index.html>

Android允許我們通過Android Beam文件傳輸功能在設備之間傳送大文件。該功能具有簡單的API，它使得用戶僅需要通過一些簡單的觸控操作就能啟動文件傳輸過程。Android Beam會自動地將文件從一臺設備拷貝至另一臺設備中，並在完成時告知用戶。

Android Beam文件傳輸API可以用來處理規模較大的數據，而在Android4.0（API Level 14）引入的Android Beam NDEF傳輸API則用來處理規模較小的數據，如URI或者消息數據等。另外，Android Beam僅僅只是Android NFC框架提供的眾多特性之一，它允許我們從NFC標籤中讀取NDEF消息。更多有關Android Beam的知識，請參考：[Beaming NDEF Messages to Other Devices](http://developer.android.com/guide/topics/connectivity/nfc/nfc.html#p2p)。更多有關NFC框架的知識，請參考：[Near Field Communication](http://developer.android.com/guide/topics/connectivity/nfc/index.html)。

## Lessons

* [**發送文件給其他設備**](sending-files.html)

  學習如何配置應用程序，使其可以發送文件給其他設備。


* [**接收其他設備的文件**](receive-files.html)

  學習如何配置應用程序，使其可以接收其他設備發送的文件。
