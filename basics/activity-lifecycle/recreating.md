# 重新創建Activity

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文: <http://developer.android.com/training/basics/activity-lifecycle/recreating.html>

有幾個場景中，Activity是由於正常的程序行為而被Destory的。例如當用戶點擊返回按鈕或者是Activity通過調用<a href="http://developer.android.com/reference/android/app/Activity.html#finish()">finish()</a>來發出停止信號。系統也有可能會在Activity處於stop狀態且長時間不被使用，或者是在前臺activity需要更多系統資源的時關閉後臺進程，以圖獲取更多的內存。

當Activity是因為用戶點擊Back按鈕或者是activity通過調用finish()結束自己時，系統就丟失了對Activity實例的引用，因為這一行為意味著不再需要這個activity了。然而，如果因為系統資源緊張而導致Activity的Destory， 系統會在用戶回到這個Activity時有這個Activity存在過的記錄，系統會使用那些保存的記錄數據（描述了當Activity被Destory時的狀態）來重新創建一個新的Activity實例。那些被系統用來恢復之前狀態而保存的數據被叫做 "instance state" ，它是一些存放在[Bundle](http://developer.android.com/reference/android/os/Bundle.html)對象中的key-value pairs。(*請注意這裡的描述，這對理解onSaveInstanceState執行的時刻很重要*)

> **Caution:** 你的Activity會在每次旋轉屏幕時被destroyed與recreated。當屏幕改變方向時，系統會Destory與Recreate前臺的activity，因為屏幕配置被改變，你的Activity可能需要加載另一些替代的資源(例如layout).

<!-- more -->

默認情況下, 系統使用 Bundle 實例來保存每一個View(視圖)對象中的信息(例如輸入EditText 中的文本內容)。因此，如果Activity被destroyed與recreated, 則layout的狀態信息會自動恢復到之前的狀態。然而，activity也許存在更多你想要恢復的狀態信息，例如記錄用戶Progress的成員變量(member variables)。

> **Note:** 為了使Android系統能夠恢復Activity中的View的狀態，**每個View都必須有一個唯一ID**，由[android:id](http://developer.android.com/reference/android/view/View.html#attr_android:id)定義。

為了可以保存額外更多的數據到saved instance state。在Activity的生命週期裡面存在一個額外的回調函數，你必須重寫這個函數。該回調函數並沒有在前面課程的圖片示例中顯示。這個方法是<a href="http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle)">onSaveInstanceState()</a> ，當用戶離開Activity時，系統會調用它。當系統調用這個函數時，系統會在Activity被異常Destory時傳遞 Bundle 對象，這樣我們就可以增加額外的信息到Bundle中並保存到系統中。若系統在Activity被Destory之後想重新創建這個Activity實例時，之前的Bundle對象會(系統)被傳遞到你我們activity的<a href="http://developer.android.com/reference/android/app/Activity.html#onRestoreInstanceState(android.os.Bundle)">onRestoreInstanceState()</a>方法與 onCreate() 方法中。

![basic-lifecycle-savestate](basic-lifecycle-savestate.png)

**Figure 2.** 當系統開始停止Activity時，只有在Activity實例會需要重新創建的情況下才會調用到<a href="http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle)">onSaveInstanceState()</a> (1) ，在這個方法裡面可以指定額外的狀態數據到Bunde中。如果這個Activity被destroyed然後這個實例又需要被重新創建時，系統會傳遞在 (1) 中的狀態數據到 onCreate()  (2) 與 <a href="http://developer.android.com/reference/android/app/Activity.html#onRestoreInstanceState(android.os.Bundle)">onRestoreInstanceState()</a>(3).

*(通常來說，跳轉到其他的activity或者是點擊Home都會導致當前的activity執行onSaveInstanceState，因為這種情況下的activity都是有可能會被destory並且是需要保存狀態以便後續恢復使用的，而從跳轉的activity點擊back回到前一個activity，那麼跳轉前的activity是執行退棧的操作，所以這種情況下是不會執行onSaveInstanceState的，因為這個activity不可能存在需要重建的操作)*



## 保存Activity狀態

當我們的activity開始Stop，系統會調用 onSaveInstanceState() ，Activity可以用鍵值對的集合來保存狀態信息。這個方法會默認保存Activity視圖的狀態信息，如在 EditText 組件中的文本或 ListView 的滑動位置。

為了給Activity保存額外的狀態信息，你必須實現onSaveInstanceState() 並增加key-value pairs到 Bundle 對象中，例如：

```java
static final String STATE_SCORE = "playerScore";
static final String STATE_LEVEL = "playerLevel";
...

@Override
public void onSaveInstanceState(Bundle savedInstanceState) {
    // Save the user's current game state
    savedInstanceState.putInt(STATE_SCORE, mCurrentScore);
    savedInstanceState.putInt(STATE_LEVEL, mCurrentLevel);

    // Always call the superclass so it can save the view hierarchy state
    super.onSaveInstanceState(savedInstanceState);
}
```

> **Caution**: 必須要調用 onSaveInstanceState() 方法的父類實現，這樣默認的父類實現才能保存視圖狀態的信息。

## 恢復Activity狀態

當Activity從Destory中重建，我們可以從系統傳遞的Activity的Bundle中恢復保存的狀態。 onCreate() 與 onRestoreInstanceState() 回調方法都接收到了同樣的Bundle，裡面包含了同樣的實例狀態信息。

由於 onCreate() 方法會在第一次創建新的Activity實例與重新創建之前被Destory的實例時都被調用，我們必須在嘗試讀取 Bundle 對象前檢測它是否為null。如果它為null，系統則是創建一個新的Activity實例，而不是恢復之前被Destory的Activity。

下面是一個示例：演示在onCreate方法裡面恢復一些數據：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState); // Always call the superclass first

    // Check whether we're recreating a previously destroyed instance
    if (savedInstanceState != null) {
        // Restore value of members from saved state
        mCurrentScore = savedInstanceState.getInt(STATE_SCORE);
        mCurrentLevel = savedInstanceState.getInt(STATE_LEVEL);
    } else {
        // Probably initialize members with default values for a new instance
    }
    ...
}
```

我們也可以選擇實現 onRestoreInstanceState()  ，而不是在onCreate方法裡面恢復數據。 **onRestoreInstanceState()方法會在 onStart() 方法之後執行. 系統僅僅會在存在需要恢復的狀態信息時才會調用 onRestoreInstanceState() ，因此不需要檢查 Bundle 是否為null。**

```java
public void onRestoreInstanceState(Bundle savedInstanceState) {
    // Always call the superclass so it can restore the view hierarchy
    super.onRestoreInstanceState(savedInstanceState);

    // Restore state members from saved instance
    mCurrentScore = savedInstanceState.getInt(STATE_SCORE);
    mCurrentLevel = savedInstanceState.getInt(STATE_LEVEL);
}
```

> **Caution**: 與上面保存一樣，總是需要調用onRestoreInstanceState()方法的父類實現，這樣默認的父類實現才能保存視圖狀態的信息。更多關於運行時狀態改變引起的recreate我們的activity。請參考[Handling Runtime Changes](http://developer.android.com/guide/topics/resources/runtime-changes.html).
