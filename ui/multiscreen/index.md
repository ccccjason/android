# 為多屏幕設計

> 編寫:[riverfeng](https://github.com/riverfeng) - 原文:<http://developer.android.com/training/multiscreen/index.html>

從小屏手機到大屏電視，android擁有數百種不同屏幕尺寸的設備。因此，設計兼容不同屏幕尺寸的應用程序滿足不同的用戶體驗就變得非常重要。

但是，只是單純的兼容不同的設備類型是遠遠不夠的。每個不同的屏幕尺寸都給用戶體驗帶來不同的可能性和挑戰。所以，為了充分的滿足和打動用戶，你的應用不僅要支持多屏幕，更要針對每個屏幕配置優化你的用戶體驗。

這個課程就將教你如何針對不同屏幕配置來優化你的UI。

本課程提供了一個簡單的示例[NewsReader](http://developer.android.com/shareables/training/NewsReader.zip)。這個示例中每節課的代碼展示瞭如何更好的優化多屏幕適配，你也可以將這個示例中的代碼運用到你自己的項目中。

> Note：這節課中相關的例子為了兼容android 3.0以下的版本使用了support library中的Fragment相關APIs。在使用該示例前，請先確定support library已經添加到你的應用中。

## Lessons

* [支持不同屏幕尺寸](screen-sizes.md)

  這節課程將引導你如何設計適配多種不同尺寸的佈局（通過使用靈活的尺寸規格guige（dimensions），相對佈局（RelativeLayout），屏幕尺寸和方向限定（qualifiers），別名過濾器（alias filter）和點9圖片）。

* [支持不同的屏幕密度](screen-desities.md)

  這節課程將演示如何支持不同像素密度的屏幕（使用密度獨立像素（dip）以及為不同的密度提供合適的位圖（bitmap））。


* [實現自適應UI流（Flows）](adapt-ui.md)

  這節課將演示如何以UI流（flow）的方式來適配一些屏幕大小/密度組合（動態佈局運行時檢測，響應當前佈局，處理屏幕配置變化）。
