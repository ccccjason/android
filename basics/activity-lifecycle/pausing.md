# 暫停與恢復Activity

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/basics/activity-lifecycle/pausing.html>

在正常使用app時，前端的activity有時會被其他可見的組件阻塞(obstructed)，從而導致當前的activity進入Pause狀態。例如，當打開一個半透明的activity時(例如以對話框的形式)，之前的activity會被暫停。 只要之前的activity仍然被部分可見，這個activity就會一直處於Paused狀態。

然而，一旦之前的activity被完全阻塞並不可見時，則其會進入Stop狀態(將在下一小節討論)。

activity一旦進入paused狀態，系統就會調用activity中的<a href="http://developer.android.com/reference/android/app/Activity.html#onPause()">onPause()</a>方法, 該方法中可以停止不應該在暫停過程中執行的操作，如暫停視頻播放；或者保存那些有可能需要長期保存的信息。如果用戶從暫停狀態回到當前activity，系統應該恢復那些數據並執行<a href="http://developer.android.com/reference/android/app/Activity.html#onResume()">onResume()</a>方法。

> **Note:** 當我們的activity收到調用onPause()的信號時，那可能意味者activity將被暫停一段時間，並且用戶很可能回到我們的activity。然而，那也是用戶要離開我們的activtiy的第一個信號。

![basic-lifecycle-paused](basic-lifecycle-paused.png)

**Figure 1.** 當一個半透明的activity阻塞activity時，系統會調用onPause()方法並且這個activity會停留在Paused 狀態(1). 如果用戶在這個activity還是在Paused 狀態時回到這個activity，系統則會調用它的onResume() (2).

## 暫停Activity

當系統調用activity中的onPause()，從技術上講，意味著activity仍然處於部分可見的狀態.但更多時候意味著用戶正在離開這個activity，並馬上會進入Stopped state. 通常應該在onPause()回調方法裡面做以下事情:
	
* 停止動畫或者是其他正在運行的操作，那些都會導致CPU的浪費.
* 提交在用戶離開時期待保存的內容(例如郵件草稿).
* 釋放系統資源，例如broadcast receivers, sensors (比如GPS), 或者是其他任何會影響到電量的資源。

例如, 如果程序使用[Camera](http://developer.android.com/reference/android/hardware/Camera.html),onPause()會是一個比較好的地方去做那些釋放資源的操作。

```java
@Override
public void onPause() {
    super.onPause();  // Always call the superclass method first

    // Release the Camera because we don't need it when paused
    // and other activities might need to use it.
    if (mCamera != null) {
        mCamera.release()
        mCamera = null;
    }
}
```

通常，**不應該**使用onPause()來保存用戶改變的數據 (例如填入表格中的個人信息) 到永久存儲(File或者DB)上。僅僅當確認用戶期待那些改變能夠被自動保存的時候(例如正在撰寫郵件草稿)，才把那些數據存到永久存儲 。但是，我們應該避免在onPause()時執行CPU-intensive 的工作，例如寫數據到DB，因為它會導致切換到下一個activity變得緩慢(應該把那些heavy-load的工作放到<a href="http://developer.android.com/reference/android/app/Activity.html#onStop()">onStop()</a>去做)。

如果activity實際上是要被Stop，那麼我們應該為了切換的順暢而減少在OnPause()方法裡面的工作量。

> **Note:**當activity處於暫停狀態，[Activity](http://developer.android.com/reference/android/app/Activity.html)實例是駐留在內存中的，並且在activity 恢復的時候重新調用。我們不需要在恢復到Resumed狀態的一系列回調方法中重新初始化組件。

## 恢復activity

當用戶從Paused狀態恢復activity時，系統會調用onResume()方法。

請注意，系統每次調用這個方法時，activity都處於前臺，包括第一次創建的時候。所以，應該實現onResume()來初始化那些在onPause方法裡面釋放掉的組件，並執行那些activity每次進入Resumed state都需要的初始化動作 (例如開始動畫與初始化那些只有在獲取用戶焦點時才需要的組件)

下面的onResume()的例子是與上面的onPause()例子相對應的。

```java
@Override
public void onResume() {
    super.onResume();  // Always call the superclass method first

    // Get the Camera instance as the activity achieves full user focus
    if (mCamera == null) {
        initializeCamera(); // Local method to handle camera init
    }
}
```
