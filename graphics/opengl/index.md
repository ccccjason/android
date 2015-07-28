# 使用OpenGL ES顯示圖像

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/graphics/opengl/index.html>

Android框架提供了大量的標準工具，用來創建吸引人的，功能豐富的圖形界面。然而，如果我們希望應用在屏幕上所繪製的內容進行更多的控制，或者正在嘗試建立三維圖像，那麼我們就需要一個不同的工具了。由Android框架提供的OpenGL ES接口給予我們一組可以顯示高級動畫和圖形的工具集，它的功能僅僅受限於我們自身的想象力。同時，在許多Android設備上搭載的圖形處理單元（GPU）都能為其提供GPU加速等性能優化。

這系列課程將展示如何使用OpenGL構建應用的基礎知識，包括配置，繪製對象，移動圖形元素以及響應點擊事件。

這系列課程所涉及的樣例代碼使用的是OpenGL ES 2.0接口，這是當前Android設備所推薦的接口版本。關於更多OpenGL ES的版本信息，可以閱讀：[OpenGL開發手冊](http://developer.android.com/guide/topics/graphics/opengl.html#choosing-version)。

> **Note：**注意不要把OpenGL ES 1.x版本的接口和OpenGL ES 2.0的接口混合調用。這兩種版本的接口不是通用的。如果嘗試混用它們可能會讓你感到無奈和沮喪。

## Sample Code

[OpenGLES.zip](http://developer.android.com/shareables/training/OpenGLES.zip)

## Lessons

* [**配置OpenGL ES的環境**](environment.html)

  學習如何配置一個可以繪製OpenGL圖形的應用。


* [**定義形狀**](shapes.html)

  學習如何定義形狀，以及為何需要了解面（Faces）和卷繞（Winding）這兩個概念的原因。


* [**繪製形狀**](draw.html)

  學習如何在應用中利用OpenGL繪製形狀。


* [**運用投影與相機視角**](projection.html)

  學習如何通過投影和相機視角，獲取圖形對象的一個新的透視效果。


* [**添加移動**](motion.html)

  學習如何對一個OpenGL圖形對象添加基本的運動效果。


* [**響應觸摸事件**](touch.html)

  學習如何與OpenGL圖形進行基本的交互。
