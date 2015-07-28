# 創建自定義View

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/custom-views/index.html>

Android的framework有大量的Views用來與用戶進行交互並顯示不同種類的數據。但是有時候你的程序有個特殊的需求，而Android內置的views組件並不能實現。這一章節會演示如何創建你自己的views，並使得它們是robust與reusable的。

**依賴和要求**

Android 2.1 (API level 7) 或更高

**你也可以看**

* [Custom Components](http://developer.android.com/guide/topics/ui/custom-components.html)
* [Input Events](http://developer.android.com/guide/topics/ui/ui-events.html)
* [Property Animation](http://developer.android.com/guide/topics/graphics/prop-animation.html)
* [Hardware Acceleration](http://developer.android.com/guide/topics/graphics/hardware-accel.html)
* [Accessibility](http://developer.android.com/guide/topics/ui/accessibility/index.html) developer guide

## Sample

[CustomView.zip](http://developer.android.com/shareables/training/CustomView.zip)

<!-- more -->

##Lesson

* [**創建一個View類**](create-view.md)

  創建一個像內置的view，有自定義屬性並支持[ADT](http://developer.android.com/sdk/eclipse-adt.html) layout編輯器。

* [**自定義Drawing**](custom-draw.md)

  使用Android graphics系統使你的view擁有獨特的視覺效果。

* [**使得View是可交互的**](make-interactive.md)

  用戶期望view對操作反應流暢自然。這節課會討論如何使用gesture detection, physics, 和 animation使你的用戶界面有專業的水準。

* [**優化View**](optimize-view.md)

  不管你的UI如何的漂亮，如果不能以高幀率流暢運行，用戶也不會喜歡。學習如何避免一般的性能問題，和如何使用硬件加速來使你的自定義圖像運行更流暢。

