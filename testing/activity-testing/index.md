# 測試你的Activity

> 編寫:[huanglizhuo](https://github.com/huanglizhuo) - 原文:<http://developer.android.com/training/activity-testing/index.html>

我們應該把編寫和運行測試作為Android應用開發週期的一部分。完備的測試可以幫助我們在開發過程中儘早發現漏洞，並讓我們對自己的代碼更有信心。

測試用例定義了一系列對象和方法從而獨立進行多個測試。測試用例可以編寫成測試組並按計劃的運行，由測試框架組織成一個可以重複運行的測試Runner（運行器，譯者注）。

這節內容將會講解如何基於最流行的JUnit框架來自定義測試框架。我們可以編寫測試用例來測試我們應用程序的特定行為，並在不同的Android設備上檢測一致性。測試用例還可以用來描述應用組件的預期行為，並作為內部代碼文檔。

## 課程

* [**建立測試環境**](prepare-activity-testing.html)

學習如何創建測試項目

* [**創建與執行測試用例**](activity-basic-testing.html)

學習如何寫測試用例來檢驗Activity中的特性，並使用Android框架提供的Instrumentation運行用例。

* [**測試UI組件**](activity-ui-testing.html)

學習如何編寫UI測試用例

* [**創建單元測試**](activity-unit-testing.html)

學習如何隔離開Activity執行單元測試

* [**創建功能測試**](activity-function-testing.html)

學習如何執行功能測試來檢驗各Activity之間的交互
