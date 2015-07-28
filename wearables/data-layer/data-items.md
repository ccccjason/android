# 同步數據單元

> 編寫:[wly2014](https://github.com/wly2014) - 原文: <http://developer.android.com/training/wearables/data-layer/data-items.html>

[DataItem](http://developer.android.com/reference/com/google/android/gms/wearable/DataItem.html)是指系統用於同步手持設備與可穿戴設備間數據的接口。一個[DataItem](http://developer.android.com/reference/com/google/android/gms/wearable/DataItem.html)通常包括以下幾點：

* **Pyload** - 一個字節數組，我們可以用來設置任何數據，讓我們的對象序列化和反序列化。Pyload的大小限制在100k之內。

* **Path** - 唯一且以前斜線開頭的字符串（如："/path/to/data"）。


通常不直接實現[DataItem](http://developer.android.com/reference/com/google/android/gms/wearable/DataItem.html)，而是：

1. 創建一個[PutdataRequest](http://developer.android.com/reference/com/google/android/gms/wearable/PutDataRequest.html)對象，指明一個字符串路徑以唯一確定該 item。
2. 調用[setData()](http://developer.android.com/reference/com/google/android/gms/wearable/PutDataRequest.html#setData(byte[]))方法設置Pyload。
3. 調用<a href="http://developer.android.com/reference/com/google/android/gms/wearable/DataApi.html#putDataItem(com.google.android.gms.common.api.GoogleApiClient, com.google.android.gms.wearable.PutDataRequest)">DataApi.putDataItem()</a>方法，請求系統創建數據元。
4. 當請求數據元的時候，系統會返回正確實現DataItem接口的對象。

然而，我們建議使用[Data Map](data-items.html#data-map)來顯示裝在一個易用的類似[Bundle](Bundle.html)接口中的數據元，而不是用setData()來處理原始字節。

## 用 Data Map 同步數據

使用[DataMap](DataMap.html)類，將數據元處理為 Android [Bundle](Bundle.html)的形式，因此會完成對象的序列化和反序列化，我們就可以以鍵值對（key-value）的形式操縱數據。

如何使用data map：

1. 創建一個 [PutDataMapRequest](http://developer.android.com/reference/com/google/android/gms/wearable/PutDataMapRequest.html)對象，設置數據元的路徑。
> **Note:** 數據元的路徑字符串是唯一確定的，這樣能夠使我們從連接任意一端訪問數據元。路徑須以前斜線開始。如果我們想在應用中使用分層數據，就要創建一個適合數據結構的路徑方案。
2. 調用[PutDataMapRequest.getDataMap()](http://developer.android.com/reference/com/google/android/gms/wearable/PutDataMapRequest.html#getDataMap())獲取一個我們可以使用的data map 對象。
3. 使用put...()方法，如：<a href="http://developer.android.com/reference/com/google/android/gms/wearable/DataMap.html#putString(java.lang.String, java.lang.String)">putString()</a>，為data map設置數據。
4. 調用[PutDataMapRequest.asPutDataRequest()](http://developer.android.com/reference/com/google/android/gms/wearable/PutDataMapRequest.html#asPutDataRequest())獲得[PutDataRequest](http://developer.android.com/reference/com/google/android/gms/wearable/PutDataRequest.html)對象。
5. 調用 <a href="http://developer.android.com/reference/com/google/android/gms/wearable/DataApi.html#putDataItem(com.google.android.gms.common.api.GoogleApiClient, com.google.android.gms.wearable.PutDataRequest)">DataApi.putDataItem()</a> 請求系統創建數據元。
> **Note:** 如果手機和可穿戴設備沒有連接，數據會緩衝並在重新建立連接時同步。

接下的例子中的`increaseCounter()`方法展示瞭如何創建一個data map，並設置數據：

```java
public class MainActivity extends Activity implements
        DataApi.DataListener,
        GoogleApiClient.ConnectionCallbacks,
        GoogleApiClient.OnConnectionFailedListener {

    private static final String COUNT_KEY = "com.example.key.count";

    private GoogleApiClient mGoogleApiClient;
    private int count = 0;

    ...

    // Create a data map and put data in it
    private void increaseCounter() {
        PutDataMapRequest putDataMapReq = PutDataMapRequest.create("/count");
        putDataMapReq.getDataMap().putInt(COUNT_KEY, count++);
        PutDataRequest putDataReq = putDataMapReq.asPutDataRequest();
        PendingResult<DataApi.DataItemResult> pendingResult =
                Wearable.DataApi.putDataItem(mGoogleApiClient, putDataReq);
    }

    ...
}
```
有關控制 [PendingResult](http://developer.android.com/reference/com/google/android/gms/common/api/PendingResult.html) 對象的更多信息，請參見 [Wait for the Status of Data Layer Calls](http://developer.android.com/training/wearables/data-layer/events.html#Wait) 。

## 監聽數據元事件

如果數據層連接的一端數據發生改變，我們很可能想要被告知在連接的另一端發生的任何改變。我們可以通過實現一個數據元事件的監聽器來完成。

當定義在上一個例子中的counter的值發生改變時，下面例子的代碼片段能夠通知我們的app。

```java
public class MainActivity extends Activity implements
        DataApi.DataListener,
        GoogleApiClient.ConnectionCallbacks,
        GoogleApiClient.OnConnectionFailedListener {

    private static final String COUNT_KEY = "com.example.key.count";

    private GoogleApiClient mGoogleApiClient;
    private int count = 0;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mGoogleApiClient = new GoogleApiClient.Builder(this)
                .addApi(Wearable.API)
                .addConnectionCallbacks(this)
                .addOnConnectionFailedListener(this)
                .build();
    }

    @Override
    protected void onResume() {
        super.onStart();
        mGoogleApiClient.connect();
    }

    @Override
    public void onConnected(Bundle bundle) {
        Wearable.DataApi.addListener(mGoogleApiClient, this);
    }

    @Override
    protected void onPause() {
        super.onPause();
        Wearable.DataApi.removeListener(mGoogleApiClient, this);
        mGoogleApiClient.disconnect();
    }

    @Override
    public void onDataChanged(DataEventBuffer dataEvents) {
        for (DataEvent event : dataEvents) {
            if (event.getType() == DataEvent.TYPE_CHANGED) {
                // DataItem 改變了
                DataItem item = event.getDataItem();
                if (item.getUri().getPath().compareTo("/count") == 0) {
                    DataMap dataMap = DataMapItem.fromDataItem(item).getDataMap();
                    updateCount(dataMap.getInt(COUNT_KEY));
                }
            } else if (event.getType() == DataEvent.TYPE_DELETED) {
                // DataItem 刪除了
            }
        }
    }

    // 我們的更新 count 的方法
    private void updateCount(int c) { ... }

    ...
}
```

這個activity是實現了[ DataItem.DataListener ](http://developer.android.com/reference/com/google/android/gms/wearable/DataApi.DataListener.html)接口。該activity在`onConnected()`方法中增加自身成為數據元事件的監聽器，並在`onPause()`方法中移除監聽器。

我們也可以用一個service實現監聽，請見 [監聽數據層事件](events.html#Listen)。


