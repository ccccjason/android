# 雲同步

> 編寫:[kesenhoo](https://github.com/kesenhoo)，[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/cloudsync/index.html>

通過為網絡連接提供強大的 API，Android Framework 可以幫助我們建立豐富的、具有云功能的 App。這些 App 可以同步數據到遠程服務器端，確保我們所有的設備都能保持數據同步，並且重要的數據都能夠備份在雲端。

本章節會介紹幾種不同的策略來實現具有云功能的 App。包括：使用我們自己的後端網絡應用進行數據雲同步，以及使用雲對數據進行備份。這樣的話，當用戶將我們的 app 安裝到一臺新的設備上時，他們之前的使用數據就可以得到恢復了。

## Lessons

* [**使用備份API**](backupapi.html)

  學習如何將 Backup API 集成到應用中。通過 Backup API 可以將用戶數據（比如配置信息、筆記、高分記錄等）無縫地在多臺設備上進行同步更新。


* [**使用Google Cloud Messaging（已廢棄）**](gcm.html)

  學習如何高效的發送多播消息，如何正確地響應接收到的Google Cloud Messaging (GCM) 消息，以及如何使用GCM消息與服務器進行高效同步。
