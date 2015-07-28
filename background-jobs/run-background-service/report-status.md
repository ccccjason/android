# 報告任務執行狀態

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/run-background-service/report-status.html>

這章節會演示如何回傳IntentService中執行的任務狀態與結果給發送方。 例如，回傳任務的執行狀態給Activity並進行更新UI。推薦的方式是使用[LocalBroadcastManager](http://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html)，這個組件可以限制broadcast intent只在自己的app中進行傳遞。

## 利用IntentService 發送任務狀態

為了在IntentService中向其他組件發送任務狀態，首先創建一個Intent並在data字段中包含需要傳遞的信息。作為一個可選項，還可以給這個Intent添加一個action與data URI。

下一步，通過執行`LocalBroadcastManager.sendBroadcast()` 來發送Intent。Intent被髮送到任何有註冊接受它的組件中。為了獲取到LocalBroadcastManager的實例，可以執行getInstance()。代碼示例如下：

```java
public final class Constants {
    ...
    // Defines a custom Intent action
    public static final String BROADCAST_ACTION =
        "com.example.android.threadsample.BROADCAST";
    ...
    // Defines the key for the status "extra" in an Intent
    public static final String EXTENDED_DATA_STATUS =
        "com.example.android.threadsample.STATUS";
    ...
}
public class RSSPullService extends IntentService {
...
    /*
     * Creates a new Intent containing a Uri object
     * BROADCAST_ACTION is a custom Intent action
     */
    Intent localIntent =
            new Intent(Constants.BROADCAST_ACTION)
            // Puts the status into the Intent
            .putExtra(Constants.EXTENDED_DATA_STATUS, status);
    // Broadcasts the Intent to receivers in this app.
    LocalBroadcastManager.getInstance(this).sendBroadcast(localIntent);
...
}
```

<!-- More -->

下一步是在發送任務的組件中接收發送出來的broadcast數據。

## 接收來自IntentService的狀態廣播

為了接受廣播的數據對象，需要使用BroadcastReceiver的子類並實現`BroadcastReceiver.onReceive()` 的方法，這裡可以接收LocalBroadcastManager發出的廣播數據。

```java
// Broadcast receiver for receiving status updates from the IntentService
private class ResponseReceiver extends BroadcastReceiver
{
    // Prevents instantiation
    private DownloadStateReceiver() {
    }
    // Called when the BroadcastReceiver gets an Intent it's registered to receive
    @
    public void onReceive(Context context, Intent intent) {
...
        /*
         * Handle Intents here.
         */
...
    }
}
```

一旦定義了BroadcastReceiver，也應該定義actions，categories與data用過濾廣播。為了實現這些，需要使用[IntentFilter](http://developer.android.com/reference/android/content/IntentFilter.html)。如下所示：

```java
// Class that displays photos
public class DisplayActivity extends FragmentActivity {
    ...
    public void onCreate(Bundle stateBundle) {
        ...
        super.onCreate(stateBundle);
        ...
        // The filter's action is BROADCAST_ACTION
        IntentFilter mStatusIntentFilter = new IntentFilter(
                Constants.BROADCAST_ACTION);

        // Adds a data filter for the HTTP scheme
        mStatusIntentFilter.addDataScheme("http");
        ...
```

為了給系統註冊這個BroadcastReceiver和IntentFilter，需要通過LocalBroadcastManager執行registerReceiver()的方法。如下所示：

```java
// Instantiates a new DownloadStateReceiver
DownloadStateReceiver mDownloadStateReceiver =
        new DownloadStateReceiver();
// Registers the DownloadStateReceiver and its intent filters
LocalBroadcastManager.getInstance(this).registerReceiver(
        mDownloadStateReceiver,
        mStatusIntentFilter);
...
```

一個BroadcastReceiver可以處理多種類型的廣播數據。每個廣播數據都有自己的ACTION。這個功能使得不用定義多個不同的BroadcastReceiver來分別處理不同的ACTION數據。為BroadcastReceiver定義另外一個IntentFilter，只需要創建一個新的IntentFilter並重復執行registerReceiver()即可。例如:

```java
/*
 * Instantiates a new action filter.
 * No data filter is needed.
 */
statusIntentFilter = new IntentFilter(Constants.ACTION_ZOOM_IMAGE);
...
// Registers the receiver with the new filter
LocalBroadcastManager.getInstance(getActivity()).registerReceiver(
        mDownloadStateReceiver,
        mIntentFilter);
```

發送一個廣播Intent並不會啟動或重啟一個Activity。即使是你的app在後臺運行，Activity的BroadcastReceiver也可以接收、處理Intent對象。但是這不會迫使你的app進入前臺。當你的app不可見時，如果想通知用戶一個發生在後臺的事件，建議使用[Notification](http://developer.android.com/reference/android/app/Notification.html)。**永遠**不要為了響應一個廣播Intent而去啟動Activity。
