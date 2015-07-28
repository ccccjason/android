# 創建向後兼容的UI

> 編寫:[spencer198711](https://github.com/spencer198711) - 原文:<http://developer.android.com/training/backward-compatible-ui/index.html>

這一課展示瞭如何以向後兼容的方式使用在新版本的Android上可用的UI組件和API，確保你的應用在之前的版本上依然能夠運行。

貫穿整個課程，在Android 3.0被新引入的[Action Bar Tabs](http://developer.android.com/guide/topics/ui/actionbar.html#Tabs)功能在本課程中作為指導例子，但是你可以在其他UI組件和API功能上運用這種方式。

## Sample

<http://developer.android.com/shareables/training/TabCompat.zip>

## Lessons

* [**抽象出新的APIs**](abstract.md)

	決定你的應用需要的功能和接口。學習如何為你的應用定義面向特定應用的、作為中間媒介並抽象出UI組件具體實現的java接口。


* [**代理至新的APIs**](new-impl.md)

	學習如何創建使用新的APIs的接口的具體實現


* [**使用舊的APIs實現新API的效果**](old-impl.md)

	學習如何創建使用老的APIs的自定義的接口實現


* [**使用能感知版本的組件**](using-component.md)

	學習如何在運行的時候去選擇一個具體的實現，並且開始在你的應用中使用接口。
