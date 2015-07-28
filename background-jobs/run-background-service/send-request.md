# 向後臺服務發送任務請求

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/run-background-service/send-request.html>

前一篇文章演示瞭如何創建一個IntentService類。這次會演示如何通過發送一個Intent來觸發IntentService執行任務。這個Intent可以傳遞一些數據給IntentService。我們可以在Activity或者Fragment的任何時間點發送這個Intent。

## 創建任務請求併發送到IntentService

為了創建一個任務請求併發送到IntentService。需要先創建一個顯式Intent，並將請求數據添加到intent中，然後通過調用
`startService()` 方法把任務請求數據發送到IntentService。

下面的是代碼示例：

* 創建一個新的顯式Intent用來啟動IntentService。

```java
/*
 * Creates a new Intent to start the RSSPullService
 * IntentService. Passes a URI in the
 * Intent's "data" field.
 */
mServiceIntent = new Intent(getActivity(), RSSPullService.class);
mServiceIntent.setData(Uri.parse(dataUrl));
```

<!-- More -->

* 執行`startService()`

```java
// Starts the IntentService
getActivity().startService(mServiceIntent);
```

注意可以在Activity或者Fragment的任何位置發送任務請求。例如，如果你先獲取用戶輸入，您可以從響應按鈕單擊或類似手勢的回調方法裡面發送任務請求。

一旦執行了startService()，IntentService在自己本身的`onHandleIntent()`方法裡面開始執行這個任務，任務結束之後，會自動停止這個Service。

下一步是如何把工作任務的執行結果返回給發送任務的Activity或者Fragment。下節課會演示如何使用[BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html)來完成這個任務。
