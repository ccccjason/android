# 優化電池壽命

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/monitoring-device-state/index.html>

顯然，手持設備的電量使用情況需要引起很大的重視。通過這一系列的課程，你將學會如何根據設備的狀態來改變App的某些行為與功能。

通過在失去網絡連接時關閉後臺更新服務，在剩餘電量較低時減少更新數據的頻率等操作，你可以在不影響用戶體驗的前提下，確保App對電池壽命的影響減到最小。

# 課程

## [檢測電量與充電狀態](battery-monitor.html)

學習如何通過判斷與檢測當前電池電量以及充電狀態的變化，改變應用程序的更新頻率。

## [判斷並監測設備的底座狀態與類型](docking-monitor.html)

設備使用習慣的區別也會影響到刷新頻率的優化措施，這節課中將學習如何判斷與監測底座狀態及其種類來改變應用程序的行為。

## [判斷並檢測網絡連接狀態](connectivity-monitor.html)

在沒有連接到互聯網的情況下，你是無法在線更新應用的。這一節課將學習如何根據網絡的連接狀態，改變後臺更新的頻率，以及如何在高帶寬傳輸任務開始前，判斷網絡連接類型(Wi-Fi/數據連接)。

## [按需操縱BroadcastReceiver](manifest-receivers.html)

在Manifest清單文件中聲明的BroadcastReceiver可以在運行時切換其開啟狀態，這樣一來，我們就可以根據當前設備的狀態，禁用那些沒有必要開啟的BroadcastReceiver。在這一節課將學習如何通過切換這些BroadcastReceiver的開啟狀態，以及如何根據設備的狀態延遲某一操作的執行時機，來提高應用的效率。
