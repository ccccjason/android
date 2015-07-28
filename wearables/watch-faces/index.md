# 創建錶盤

> 編寫:[heray1990](https://github.com/heray1990) - 原文: <http://developer.android.com/training/wearables/watch-faces/index.html>

Android Wear 的錶盤是一個動態的數字畫布，它用顏色、動畫和相關的上下文信息來表示時間。[Android Wear companion app](https://play.google.com/store/apps/details?id=com.google.android.wearable.app) 提供了不同風格和形狀的錶盤。當用戶選擇可穿戴設備應用或者配套應用上可用的錶盤，可穿戴設備會提供錶盤的預覽並讓用戶設置選項。

Android Wear 允許我們為 Wear 設備創建自定義的錶盤。當用戶安裝一個包含錶盤的可穿戴應用的手持式應用時，它們可以在手持式設備上的 Android Wear 配套應用和在可穿戴設備上的錶盤選擇器中使用。

這個課程教我們實現自定義錶盤並將它們打包進一個可穿戴應用。這節課還覆蓋設計方面的考慮和實現提示，從而確保我們的設計整合到系統 UI 並且節能。

> **Note:** 我們推薦使用 Android Studio 做 Android Wear 開發，它提供工程初始配置，庫包含和方便的打包流程，這些在ADT中是沒有的。這系列教程假定你正在使用Android Studio。

## Lesson

[設計錶盤](designing.html)

學習如何設計一個可以工作在 Android Wear 設備上的錶盤。

[構建錶盤服務](service.html)

學習如何在錶盤的生命週期期間響應重要的時間。

[繪製錶盤](drawing.html)

學習如何在一個 Wear 設備的屏幕上繪製錶盤。

[在錶盤上顯示信息](information.html)

學習如何將上下文信息集成到錶盤中。

[提供配置 Activity](configuration.html)

學習如何創建帶有可配置參數的錶盤。

[定位常見的問題](issues.html)

學習如何在開發錶盤的時候修改常見的問題。

[優化性能和電池使用時間](performance.html)

學習如何提高動畫的幀速率和節能。