# 實現高效的導航

> 編寫:[Lin-H](https://github.com/Lin-H) - 原文:<http://developer.android.com/training/implementing-navigation/index.html>

這節課將會演示如何實現在[Designing Effective Navigation](http://developer.android.com/training/design-navigation/index.html)中所詳述的關鍵導航設計模式。

在閱讀這節課程內容之後，你會對如何使用tabs, swipe views, 和navigation drawer實現導航模式有一個深刻的理解。也會明白如何提供合適的向前向後導航(Up and Back navigation)。

> **Note**:本節課中的幾個元素需要使用[Support Library](http://developer.android.com/tools/support-library/index.html) API。如果你之前沒有使用過Support Library，可以按照[Support Library Setup](http://developer.android.com/tools/support-library/setup.html)文檔說明來使用。

## Sample Code

[EffectiveNavigation.zip](http://developer.android.com/shareables/training/EffectiveNavigation.zip)

## Lessons

* [使用Tabs創建Swipe View](lateral.md)

  學習如何在action bar中實現tab，並提供橫向分頁(swipe views)在tab之間導航切換。


* [創建抽屜導航(Navigation Drawer)](nav-drawer.md)

  學習如何建立隱藏於屏幕邊上的界面，通過劃屏(swipe)或點擊action bar中的app圖標來顯示這個界面。


* [提供向上導航](ancestral.md)

  學習如何使用action bar中的app圖標實現向上導航


* [提供適當的向後導航](temporal.md)

  學習如何正確處理特殊情況下的向後按鈕(Back button)，包括在通知或app widget中的深度鏈接，如何將activity插入後退棧(back stack)中。


* [實現Descendant Navigation](descendant.md)

  學習更精細地導航進入你的應用信息層。

