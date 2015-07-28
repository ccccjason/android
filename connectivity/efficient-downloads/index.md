# 傳輸數據時避免消耗大量電量

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/efficient-downloads/index.html>

在這一章，我們將學習最小化下載，網絡連接，尤其是無線電連接對電量的影響。

下面幾節課會演示如何使用像緩存（caching）、輪詢（polling）和預取（prefetching）這樣的技術來調度與執行下載操作。我們還會學習無線電波的 power-use 屬性配置是如何影響我們對於在何時，用什麼，以何種方式來傳輸數據的選擇。當然這些選擇是為了最小化對電量的影響。

**我們同樣需要閱讀**
[優化電池使用時間](performance/monitoring-device-state/index.html)

## Lesson

[**優化下載以高效地訪問網絡**](efficient-network-access.html)

  這節課介紹了無線電波狀態機（wireless radio state machine），解釋了 app 的連接模型（connectivity model）如何與它交互，以及如何最小化數據連接和使用預取（prefetching）和捆綁（bundling）來最小化數據傳輸對電池消耗的影響。


[**最小化定期更新造成的影響**](regular-update.html)

  這節課我們將瞭解如何調整刷新頻率以最大程度減輕底層無線電波狀態機的後臺更新所造成的影響。


[**重複的下載是冗餘的**](redundant-redundant.html)

  減少下載的最根本途徑是隻下載我們需要的內容。這節課介紹了消除冗餘下載的一些最佳實踐。


* [**根據網絡連接類型來調整下載模式**](connectivity-patterns.html)

  不同連接類型對電池電量的影響並不相同。不僅僅是 Wi-Fi 比無線電波更省電，不同的無線電波技術對電量也有不同的影響。
