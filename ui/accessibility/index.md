# 實現輔助功能

> 編寫:[K0ST](https://github.com/K0ST) - 原文:<http://developer.android.com/training/accessibility/index.html>

當我們需要儘可能擴大我們用戶的基數的時候，就要開始注意我們軟件的可達性了(*Accessibility 易接近，可親性*)。在界面中展示提示對大多數用戶而言是可行的，比如說當按鈕被按下時視覺上的變化，但是對於那些視力上有些缺陷的用戶而言效果就不是那麼理想了。

本章將給您演示如何最大化利用Android框架中的Accessibility特性。包括如何利用焦點導航(*focus navigation*)與內容描述(*content description*)對你的應用的可達性進行優化。也包括了創建Accessibility Service， 使用戶與應用（不僅僅是你自己的應用）之間的交互更加容易。

## Lessons

* [**開發Accessibility應用**](accessible-app.md)

  學習如何讓你的程序更易用，具有可達性。 允許使用鍵盤或者十字鍵(*directional pad*)來進行導航，利用Accessibility Service特性設置標籤或執行事件來打造更舒適的用戶體驗。


* [**編寫 Accessibility Services**](accessible-service.md)

  編寫一個Accessibility Service來監聽可達性事件，利用這些不同類型的事件和內容描述來幫助用戶與應用的交互。本例將會實現利用一個TTS引擎來向用戶發出語音提示的功能。
