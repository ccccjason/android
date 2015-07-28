# 管理設備的喚醒狀態

> 編寫:[jdneo](https://github.com/jdneo)，[lttowq](https://github.com/lttowq) - 原文:<http://developer.android.com/training/scheduling/index.html>

當一個Android設備閒置時，首先它的屏幕將會變暗，然後關閉屏幕，最後關閉CPU。
這樣可以防止設備的電量被迅速消耗殆盡。但是，有時候也會存在一些特例：

* 例如遊戲或視頻應用需要保持屏幕常亮；
* 其它應用也許不需要屏幕常亮，但或許會需要CPU保持運行，直到某個關鍵操作結束。

這節課描述如何在必要的時候保持設備喚醒，同時又不會過多消耗它的電量。

## Demos
[**Scheduler.zip**](http://developer.android.com/shareables/training/Scheduler.zip)

## Lessons

### [保持設備喚醒](wake-lock.html)

學習如何在必要的時候保持屏幕和CPU喚醒，同時減少對電池壽命的影響。

### [調度重複鬧鐘](alarms.html)

對於那些發生在應用生命週期之外的操作，學習如何使用重複鬧鐘對它們進行調度，即使該應用沒有運行或者設備處於睡眠狀態。

