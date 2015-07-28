# 處理數據層的事件

> 編寫:[wly2014](https://github.com/wly2014) - 原文: <http://developer.android.com/training/wearables/data-layer/events.html>

當做出數據層上的調用時，我們可以得到它完成後的調用狀態，也可以用監聽器監聽到調用最終實現的改變。

## 等待數據層調用的狀態

注意到，調用數據層API，有時會返回 [PendingResult](PendingResult.html)，如 <a href="http://developer.android.com/reference/com/google/android/gms/wearable/DataApi.html#putDataItem(com.google.android.gms.common.api.GoogleApiClient, com.google.android.gms.wearable.PutDataRequest)">putDataItem()</a>。[PendingResult](http://developer.android.com/reference/com/google/android/gms/common/api/PendingResult.html) 一被創建，操作就會在後臺排列等候。之後我們若無動作，這些操作最終會默默完成。然而，通常要處理操作完成後的結果，[PendingResult](http://developer.android.com/reference/com/google/android/gms/common/api/PendingResult.html) 能夠讓我們同步或異步地等待結果。

### 異步調用

若代碼運行在主UI線程上，不要讓數據層API調用阻塞UI。我們可以增加一個回調到 [PendingResult](http://developer.android.com/reference/com/google/android/gms/common/api/PendingResult.html) 對象來運行異步調用，該回調函數將在操作完成時觸發。

```java
pendingResult.setResultCallback(new ResultCallback<DataItemResult>() {
    @Override
    public void onResult(final DataItemResult result) {
        if(result.getStatus().isSuccess()) {
            Log.d(TAG, "Data item set: " + result.getDataItem().getUri());
        }
    }
});
```

### 同步調用

如果代碼是運行在後臺服務的一個獨立的處理線程上（[WearableListenerService](http://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService.html)的情況），則調用導致的阻塞沒影響。在這種情況下,我們可以用 [PendingResult](http://developer.android.com/reference/com/google/android/gms/common/api/PendingResult.html)對象調用[await()](http://developer.android.com/reference/com/google/android/gms/common/api/PendingResult.html#await())，它將阻塞至請求完成,並返回一個Result對象：

```java
DataItemResult result = pendingResult.await();
if(result.getStatus().isSuccess()) {
    Log.d(TAG, "Data item set: " + result.getDataItem().getUri());
}
```

<a name="Listen"></a>
## 監聽數據層事件

因為數據層在手持和可穿戴設備間同步併發送數據，所以通常要監聽重要事件，例如創建數據元，接收消息，或連接可穿戴設備和手機。

對於監聽數據層事件，有兩種選擇：

* 創建一個繼承自 [WearableListenerService](http://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService.html) 的 service。
* 創建一個實現 [DataApi.DataListener](http://developer.android.com/reference/com/google/android/gms/wearable/DataApi.DataListener.html) 接口的 activity。

通過這兩種選擇，為我們感興趣的事件重寫數據事件回調方法。

### 使用 WearableListenerService

通常，我們在手持設備和可穿戴設備上都創建該 service 的實例。如果我們不關心其中一個應用中的數據事件，就不需要在相應的應用中實現此 service。

例如，我們可以在一個手持設備應用程序上操作數據元對象，可穿戴設備應用監聽這些更新來更新自身的UI。而可穿戴不更新任何數據元，所以手持設備應用不監聽任何可穿戴式設備應用的數據事件。

我們可以用 [WearableListenerService](http://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService.html) 監聽如下事件：

* [onDataChanged()](http://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService.html#onDataChanged(com.google.android.gms.wearable.DataEventBuffer)) - 當數據元對象創建，更改，刪除時調用。一連接端的事件將觸發兩端的回調方法。
* [onMessageReceived()](http://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService.html#onMessageReceived(com.google.android.gms.wearable.MessageEvent)) - 消息從一連接端發出，在另一連接端觸發此回調方法。
* [onPeerConnected()](http://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService.html#onMessageReceived(com.google.android.gms.wearable.MessageEvent)) 和 [onPeerDisconnected()](http://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService.html#onPeerDisconnected(com.google.android.gms.wearable.Node)) - 當與手持或可穿戴設備連接或斷開時調用。一連接端連接狀態的改變會在兩端觸發此回調方法。

創建[WearableListenerService](http://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService.html)，我們需要：

1. 創建一個繼承自 [WearableListenerService](http://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService.html) 的類。
2. 監聽我們關心的事件，比如 [onDataChanged()](http://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService.html#onDataChanged(com.google.android.gms.wearable.DataEventBuffer))。
3. 在Android manifest中聲明一個intent filter，把我們的 [WearableListenerService](http://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService.html) 通知給系統。這樣允許系統在需要時綁定我們的 service。

下例展示如何實現一個簡單的 [WearableListenerService](http://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService.html)：

```java
public class DataLayerListenerService extends WearableListenerService {

    private static final String TAG = "DataLayerSample";
    private static final String START_ACTIVITY_PATH = "/start-activity";
    private static final String DATA_ITEM_RECEIVED_PATH = "/data-item-received";

    @Override
    public void onDataChanged(DataEventBuffer dataEvents) {
        if (Log.isLoggable(TAG, Log.DEBUG)) {
            Log.d(TAG, "onDataChanged: " + dataEvents);
        }
        final List events = FreezableUtils
                .freezeIterable(dataEvents);

        GoogleApiClient googleApiClient = new GoogleApiClient.Builder(this)
                .addApi(Wearable.API)
                .build();

        ConnectionResult connectionResult =
                googleApiClient.blockingConnect(30, TimeUnit.SECONDS);

        if (!connectionResult.isSuccess()) {
            Log.e(TAG, "Failed to connect to GoogleApiClient.");
            return;
        }

        // Loop through the events and send a message
        // to the node that created the data item.
        for (DataEvent event : events) {
            Uri uri = event.getDataItem().getUri();

            // Get the node id from the host value of the URI
            String nodeId = uri.getHost();
            // Set the data of the message to be the bytes of the URI
            byte[] payload = uri.toString().getBytes();

            // Send the RPC
            Wearable.MessageApi.sendMessage(googleApiClient, nodeId,
                    DATA_ITEM_RECEIVED_PATH, payload);
        }
    }
}
```

這是Android mainfest中相應的intent filter：

```xml
<service android:name=".DataLayerListenerService">
  <intent-filter>
      <action android:name="com.google.android.gms.wearable.BIND_LISTENER" />
  </intent-filter>
</service>
```

### 數據層回調權限

為了在數據層事件上向我們的應用傳送回調方法，Google Play services 綁定到我們的WearableListenerService，並通過IPC調用回調方法。這樣的結果是，我們的回調方法繼承了調用進程的權限。

如果我們想在一個回調中執行權限操作，安全檢查會失敗，因為回調是以調用進程的身份運行，而不是應用程序進程的身份運行。

為了解決這個問題，在進入IPC後使用 [clearCallingIdentity()](http://developer.android.com/reference/android/os/Binder.html#clearCallingIdentity()) 重置身份，當完成權限操作後，使用 [restoreCallingIdentity()](http://developer.android.com/reference/android/os/Binder.html#restoreCallingIdentity(long)) 恢復身份:

```java
long token = Binder.clearCallingIdentity();
try {
    performOperationRequiringPermissions();
} finally {
    Binder.restoreCallingIdentity(token);
}
```

### 使用一個Listener Activity

如果我們的應用只關心當用戶與應用交互時產生的數據層事件，並且不需要一個長時間運行的 service 來處理每一次數據的改變，那麼我們可以在一個 activity 中通過實現如下一個和多個接口來監聽事件：

* [DataApi.DataListener](http://developer.android.com/reference/com/google/android/gms/wearable/DataApi.DataListener.html)
* [MessageApi.MessageListener](http://developer.android.com/reference/com/google/android/gms/wearable/MessageApi.MessageListener.html)
* [NodeApi.NodeListener](http://developer.android.com/reference/com/google/android/gms/wearable/NodeApi.NodeListener.html)

創建一個 activity 監聽數據事件，需要：

1. 實現所需的接口。
2. 在 [onCreate(Bundle)](http://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle)) 中創建  [GoogleApiClient](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.html) 實例。
3. 在 [onStart()](http://developer.android.com/reference/android/app/Activity.html#onStart()) 中調用 [connect()](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.html#connect()) 將客戶端連接到 Google Play services。
4. 當連接到Google Play services後，系統調用 [onConnected()](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.ConnectionCallbacks.html#onConnected(android.os.Bundle))。這裡是我們調用 <a href="http://developer.android.com/reference/com/google/android/gms/wearable/DataApi.html#addListener(com.google.android.gms.common.api.GoogleApiClient, com.google.android.gms.wearable.DataApi.DataListener)">DataApi.addListener()</a>，<a href="http://developer.android.com/reference/com/google/android/gms/wearable/MessageApi.html#addListener(com.google.android.gms.common.api.GoogleApiClient, com.google.android.gms.wearable.MessageApi.MessageListener)">MessageApi.addListener()</a> 或 <a href="http://developer.android.com/reference/com/google/android/gms/wearable/NodeApi.html#addListener(com.google.android.gms.common.api.GoogleApiClient, com.google.android.gms.wearable.NodeApi.NodeListener)">NodeApi.addListener()</a>，以告知Google Play services 我們的 activity 要監聽數據層事件的地方。
5. 在 [onStop()](http://developer.android.com/reference/android/app/Activity.html#onStop()) 中，用 [DataApi.removeListener()](http://developer.android.com/reference/com/google/android/gms/wearable/DataApi.html#removeListener(com.google.android.gms.common.api.GoogleApiClient, com.google.android.gms.wearable.DataApi.DataListener)), [MessageApi.removeListener()](http://developer.android.com/reference/com/google/android/gms/wearable/MessageApi.html#removeListener(com.google.android.gms.common.api.GoogleApiClient, com.google.android.gms.wearable.MessageApi.MessageListener))或[NodeApi.removeListener()](http://developer.android.com/reference/com/google/android/gms/wearable/NodeApi.html#removeListener(com.google.android.gms.common.api.GoogleApiClient, com.google.android.gms.wearable.NodeApi.NodeListener)) 註銷監聽。
6. 基於我們實現的接口繼而實現 onDataChanged(), onMessageReceived(), onPeerConnected()和 onPeerDisconnected()。

這是實現DataApi.DataListener的例子 ：

```java
public class MainActivity extends Activity implements
        DataApi.DataListener, ConnectionCallbacks, OnConnectionFailedListener {

    private GoogleApiClient mGoogleApiClient;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.main);
        mGoogleApiClient = new GoogleApiClient.Builder(this)
                .addApi(Wearable.API)
                .addConnectionCallbacks(this)
                .addOnConnectionFailedListener(this)
                .build();
    }

    @Override
    protected void onStart() {
        super.onStart();
        if (!mResolvingError) {
            mGoogleApiClient.connect();
        }
    }

    @Override
    public void onConnected(Bundle connectionHint) {
        if (Log.isLoggable(TAG, Log.DEBUG)) {
            Log.d(TAG, "Connected to Google Api Service");
        }
        Wearable.DataApi.addListener(mGoogleApiClient, this);
    }

    @Override
    protected void onStop() {
        if (null != mGoogleApiClient && mGoogleApiClient.isConnected()) {
            Wearable.DataApi.removeListener(mGoogleApiClient, this);
            mGoogleApiClient.disconnect();
        }
        super.onStop();
    }

    @Override
    public void onDataChanged(DataEventBuffer dataEvents) {
        for (DataEvent event : dataEvents) {
            if (event.getType() == DataEvent.TYPE_DELETED) {
                Log.d(TAG, "DataItem deleted: " + event.getDataItem().getUri());
            } else if (event.getType() == DataEvent.TYPE_CHANGED) {
                Log.d(TAG, "DataItem changed: " + event.getDataItem().getUri());
            }
        }
    }
}
```
