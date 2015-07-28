# 管理Activity的生命週期

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/basics/activity-lifecycle/index.html>

當用戶進入，退出，回到我們的App時，程序中的[Activity](http://developer.android.com/reference/android/app/Activity.html) 實例會在生命週期中的不同狀態間進行切換。例如，activity第一次啟動的時候，它來到系統的前臺，開始接受用戶的焦點。在此期間，Android系統調用了一系列的生命週期中的方法。如果用戶執行了啟動另一個activity或者切換到另一個app(此時雖然當前activity不可見，但其實例與數據仍然存在)的操作, 系統又會調用一些生命週期中的方法。

在生命週期的回調方法中，可以聲明當用戶離開或者重新進入這個Activity所需要執行的操作。例如, 如果我們建立了一個streaming video player, 在用戶切換到另外一個app的時候，應該暫停video 並終止網絡連接。當用戶返回時，我們可以重新建立網絡連接並允許用戶從同樣的位置恢復播放。

本章會介紹一些[Activity](http://developer.android.com/reference/android/app/Activity.html)生命週期中重要的回調方法，如何使用那些方法，使得程序符合用戶的期望且在activity不需要的時候不會導致系統資源的浪費。

**完整的Demo示例**：[ActivityLifecycle.zip](http://developer.android.com/shareables/training/ActivityLifecycle.zip)

<!-- more -->

## Lessons

* [**啟動與銷燬Activity**](starting.html)

  學習關於activity生命週期的基礎知識，用戶如何啟動應用及執行基本activity的創建。


* [**暫停與恢復Activity**](pausing.html)

  學習activity暫停發生時，我們應該做哪些事情。


* [**停止與重啟Activity**](stopping.html)

  學習用戶離開activity與返回activity時會發生的事情。


* [**重新創建Activity**](recreating.html)

  學習當我們的activity被銷燬時發生了什麼事情，以及在有必要時如何重建我們的activity。
