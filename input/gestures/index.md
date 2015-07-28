# 使用觸摸手勢

> 編寫:[Andrwyw](https://github.com/Andrwyw) - 原文:<http://developer.android.com/training/gestures/index.html>

本章節講述，如何編寫一個允許用戶通過觸摸手勢進行交互的app。Android提供了各種各樣的API，來幫助我們創建和檢測手勢。

儘管對於一些基本的操作來說，我們的app不應該依賴於觸摸手勢（因為某些情況下手勢是不用的）。但為我們的app添加基於觸摸的交互，將會大大地提高app的可用性和吸引力。

為了給用戶提供一致的、符合直覺的使用體驗，我們的app應該遵守Android觸摸手勢的慣常做法。[手勢設計指南](http://developer.android.com/design/patterns/gestures.html)介紹了在Android app中，如何使用常用的手勢。同樣，設計指南也提供了[觸摸反饋](http://developer.android.com/design/style/touch-feedback.html)的相關內容。

## Lessons

[**檢測常用的手勢**](detector.html)

  學習如何通過使用[GestureDetector](http://developer.android.com/reference/android/view/GestureDetector.html)來檢測基本的觸摸手勢，如滑動、慣性滑動以及雙擊。


[**追蹤手勢移動**](movement.html)

  學習如何追蹤手勢移動。


[**Scroll手勢動畫**](scroll.html)

  學習如何使用scrollers（[Scrollers](http://developer.android.com/reference/android/widget/Scroller.html)以及[OverScroll](http://developer.android.com/reference/android/widget/OverScroller.html)）來產生滾動動畫，以響應觸摸事件。


[**處理多觸摸手勢**](multi.html)

  學習如何檢測多點(手指)觸摸手勢。


[**拖拽與縮放**](scale.html)

  學習如何實現基於觸摸的拖拽與縮放。


[**管理ViewGroup中的觸摸事件**](viewgroup.html)

  學習如何在[ViewGroup](http://developer.android.com/reference/android/view/ViewGroup.html)中管理觸摸事件，以確保事件能被正確地分發到目標views上。
