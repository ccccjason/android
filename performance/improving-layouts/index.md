# 提升Layout的性能

> 編寫: [allenlsy](https://github.com/allenlsy) - 原文: <http://developer.android.com/training/improving-layouts/index.html>

Layout 是 Android 應用中直接影響用戶體驗的關鍵部分。如果實現的不好，你的 Layout 會導致程序非常佔用內存並且 UI 運行緩慢。Android SDK 帶有幫助你找到 Layout 性能問題的工具。結合本課內容使用它，你將學會使用最小的內存空間實現流暢的 UI。

## Lessons

#### [優化Layout的層級](optimizing-layout.html)

就像一個複雜的網頁會減慢載入速度，你的Layout結構如果太複雜，也會造成性能問題。本節教你如何使用SDK自帶工具來查看Layout並找到性能瓶頸。

#### [使用`<include/>`標籤重用Layout](reuse-layouts.html)

如果你的程序的 UI 在不同地方重複使用某個 Layout，那本節將教你如何創建高效的，可重用的Layout部件，並把它們“包含”到其他 UI Layout 中。

#### [按需載入視圖](loading-ondemand.html)

除了簡單的把一個 Layout 包含到另一個 Layout 中，你可能還想在程序開始之後，僅當你的 Layout 對用戶可見時才開始載入。本節告訴你如何使用分步載入 Layout 來提高 Layout 的首次加載性能。

#### [優化ListView的滑動性能](smooth-scrolling.html)

如果你有一個每個列表項 (item) 都包含很多數據或者複雜數據的 ListView ，那麼列表滾動的性能很有可能會存在問題。本節會介紹給你一些如何優化滾動流暢度的技巧。
