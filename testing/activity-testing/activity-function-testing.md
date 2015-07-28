# 創建功能測試

> 編寫:[huanglizhuo](https://github.com/huanglizhuo) - 原文:<http://developer.android.com/training/activity-testing/activity-functional-testing.html>

功能測試包括驗證單個應用中的各個組件是否與使用者期望的那樣（與其它組件）協同工作。比如，我們可以創建一個功能測試驗證在用戶執行UI交互時[Activity](http://developer.android.com/reference/android/app/Activity.html)是否正確啟動目標[Activity](http://developer.android.com/reference/android/app/Activity.html)。

要為[Activity](http://developer.android.com/reference/android/app/Activity.html)創建功能測，我們的測試類應該對[ActivityInstrumentationTestCase2](http://developer.android.com/reference/android/test/ActivityInstrumentationTestCase2.html)進行擴展。與[ActivityUnitTestCase](http://developer.android.com/reference/android/test/ActivityUnitTestCase.html)不同，[ActivityInstrumentationTestCase2](http://developer.android.com/reference/android/test/ActivityInstrumentationTestCase2.html)中的測試可以與Android系統通信，發送鍵盤輸入及點擊事件到UI中。

要了解一個完整的測試例子可以參考示例應用中的`SenderActivityTest.java`。

## 添加測試方法驗證函數的行為

我們的函數測試目標應該包括:

* 驗證UI控制是否正確啟動了目標Activity。
* 驗證目標Activity的表現是否按照發送Activity提供的數據呈現。

我們可以這樣實現測試方法:

```java
@MediumTest
public void testSendMessageToReceiverActivity() {
    final Button sendToReceiverButton = (Button)
            mSenderActivity.findViewById(R.id.send_message_button);

    final EditText senderMessageEditText = (EditText)
            mSenderActivity.findViewById(R.id.message_input_edit_text);

    // Set up an ActivityMonitor
    ...

    // Send string input value
    ...

    // Validate that ReceiverActivity is started
    ...

    // Validate that ReceiverActivity has the correct data
    ...

    // Remove the ActivityMonitor
    ...
}
```

測試會等待匹配的Activity啟動，如果超時則會返回null。如果ReceiverActivity啟動了，那麼先前配置的[ActivityMoniter](http://developer.android.com/reference/android/app/Instrumentation.ActivityMonitor.html)就會收到一次碰撞（Hit）。我們可以使用斷言方法驗證ReceiverActivity是否的確啟動了，以及[ActivityMoniter](http://developer.android.com/reference/android/app/Instrumentation.ActivityMonitor.html)記錄的碰撞次數是否按照預想地那樣增加。

## 設立一個ActivityMonitor

為了在應用中監視單個[Activity](http://developer.android.com/reference/android/app/Activity.html)我們可以註冊一個[ActivityMoniter](http://developer.android.com/reference/android/app/Instrumentation.ActivityMonitor.html)。每當一個符合要求的Activity啟動時，系統會通知[ActivityMoniter](http://developer.android.com/reference/android/app/Instrumentation.ActivityMonitor.html)，進而更新碰撞數目。

通常來說要使用[ActivityMoniter](http://developer.android.com/reference/android/app/Instrumentation.ActivityMonitor.html)，我們可以這樣：

1. 使用<a href="http://developer.android.com/reference/android/test/InstrumentationTestCase.html#getInstrumentation()">getInstrumentation()</a>方法為測試用例實現[Instrumentation](http://developer.android.com/reference/android/app/Instrumentation.html)。
2. 使用[Instrumentation](http://developer.android.com/reference/android/app/Instrumentation.html)的一種addMonitor()方法為當前instrumentation添加一個[Instrumentation.ActivityMonitor](http://developer.android.com/reference/android/app/Instrumentation.ActivityMonitor.html)實例。匹配規則可以通過[IntentFilter](http://developer.android.com/reference/android/content/IntentFilter.html)或者類名字符串。
3. 等待開啟一個Activity。
4. 驗證監視器撞擊次數的增加。
5. 移除監視器。

下面是一個例子:

```java
// Set up an ActivityMonitor
ActivityMonitor receiverActivityMonitor =
        getInstrumentation().addMonitor(ReceiverActivity.class.getName(),
        null, false);

// Validate that ReceiverActivity is started
TouchUtils.clickView(this, sendToReceiverButton);
ReceiverActivity receiverActivity = (ReceiverActivity)
        receiverActivityMonitor.waitForActivityWithTimeout(TIMEOUT_IN_MS);
assertNotNull("ReceiverActivity is null", receiverActivity);
assertEquals("Monitor for ReceiverActivity has not been called",
        1, receiverActivityMonitor.getHits());
assertEquals("Activity is of wrong type",
        ReceiverActivity.class, receiverActivity.getClass());

// Remove the ActivityMonitor
getInstrumentation().removeMonitor(receiverActivityMonitor);
```

## 使用Instrumentation發送一個鍵盤輸入

如果[Activity](http://developer.android.com/reference/android/app/Activity.html)有一個[EditText](http://developer.android.com/reference/android/widget/EditText.html)，我們可以測試用戶是否可以給[EditText](http://developer.android.com/reference/android/widget/EditText.html)對象輸入數值。

通常在[ActivityInstrumentationTestCase2](http://developer.android.com/reference/android/test/ActivityInstrumentationTestCase2.html)中給[EditText](http://developer.android.com/reference/android/widget/EditText.html)對象發送串字符，我們可以這樣做：

1. 使用<a href="http://developer.android.com/reference/android/app/Instrumentation.html#runOnMainSync(java.lang.Runnable)">runOnMainSync()</a>方法在一個循環中同步地調用<a href="http://developer.android.com/reference/android/view/View.html#requestFocus()">requestFocus()</a>。這樣，我們的UI線程就會在獲得焦點前一直被阻塞。
2. 調用<a href="http://developer.android.com/reference/android/app/Instrumentation.html#waitForIdleSync()">waitForIdleSync()</a>方法等待主線程空閒（也就是說,沒有更多事件需要處理）。
3. 調用<a href="http://developer.android.com/reference/android/app/Instrumentation.html#sendStringSync(java.lang.String)">sendStringSync()</a>方法給[EditText](http://developer.android.com/reference/android/widget/EditText.html)對象發送一個我們輸入的字符串。

比如:

```java
// Send string input value
getInstrumentation().runOnMainSync(new Runnable() {
    @Override
    public void run() {
        senderMessageEditText.requestFocus();
    }
});
getInstrumentation().waitForIdleSync();
getInstrumentation().sendStringSync("Hello Android!");
getInstrumentation().waitForIdleSync();
```

本節例子[AndroidTestingFun.zip](http://developer.android.com/shareables/training/AndroidTestingFun.zip)
