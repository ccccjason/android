# Android Wear 上的位置檢測

> 編寫:[heray1990](https://github.com/heray1990) - 原文: <http://developer.android.com/training/articles/wear-location-detection.html>

可穿戴設備上的位置感知讓我們可以創建為用戶提供更好地瞭解地理位置、移動和周圍事物的應用。由於可穿戴設備小型和方便的特點，我們可以構建低摩擦應用來記錄和響應位置數據。

一些可穿戴設備帶有 GPS 感應器，它們可以在不需要其它設備的幫助下檢索位置數據。無論如何，當我們在可穿戴應用上請求獲取位置數據，我們不需要擔心位置數據從哪裡發出；系統會用最節能的方法來檢索位置更新。我們的應用應該可以處理位置數據的丟失，以防沒有內置 GPS 感應器的可穿戴設備與配套設備斷開連接。

這篇文章介紹如何檢查設備上的位置感應器、檢索位置數據和監視數據連接。

> **Note:** 這篇文章假設我們知道如何使用 Google Play services API 來檢索位置數據。更多相關的內容，請見 [Android 位置信息](http://hukai.me/android-training-course-in-chinese/location/index.html)。

## 連接 Google Play Services

可穿戴設備上的位置數據可以通過 Google Play services location APIs 來獲取。我們可以使用 [FusedLocationProviderApi](http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html) 和它伴隨的類來獲取這個數據。為了訪問位置服務，可以創建 [GoogleApiClient](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.html) 實例，這個實例是任何 Google Play services APIs 的主要入口。

> **Caution:** 不要使用 Android 框架已有的 [Location](http://developer.android.com/reference/android/location/package-summary.html) APIs。檢索位置更新最好的方法是通過這篇文章介紹的 Google Play services API 獲取。

為了連接 Google Play services，配置應用來創建 [GoogleApiClient](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.html) 實例：

1. 創建一個 activity 來指定 [ConnectionCallbacks](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.ConnectionCallbacks.html)、[OnConnectionFailedListener](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.OnConnectionFailedListener.html) 和 [LocationListener](http://developer.android.com/reference/com/google/android/gms/location/LocationListener.html) 接口的實現。
2. 在 activity 的 [onCreate()](http://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle)) 方法中，創建 [GoogleApiClient](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.html) 實例和添加位置服務。
3. 為了優雅地管理連接的生命週期，在 [onResume()](http://developer.android.com/reference/android/app/Activity.html#onResume()) 方法裡調用 [connect()](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.html#connect()) 和在 [onPause()](http://developer.android.com/reference/android/app/Activity.html#onPause()) 方法裡調用 [disconnect()](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.html#disconnect())。

下面的代碼示例介紹了一個 activity 的實現來實現 [LocationListener](http://developer.android.com/reference/com/google/android/gms/location/LocationListener.html) 接口：

```java
public class WearableMainActivity extends Activity implements
    GoogleApiClient.ConnectionCallbacks,
    GoogleApiClient.OnConnectionFailedListener,
    LocationListener {

    private GoogleApiClient mGoogleApiClient;
    ...

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        ...
        mGoogleApiClient = new GoogleApiClient.Builder(this)
                .addApi(LocationServices.API)
                .addApi(Wearable.API)  // used for data layer API
                .addConnectionCallbacks(this)
                .addOnConnectionFailedListener(this)
                .build();
    }

    @Override
    protected void onResume() {
        super.onResume();
        mGoogleApiClient.connect();
        ...
    }

    @Override
    protected void onPause() {
        super.onPause();
        ...
        mGoogleApiClient.disconnect();
    }
}
```

更多關於連接 Google Play services 的內容，請見 [Accessing Google APIs](http://developer.android.com/google/auth/api-client.html)。

## 請求位置更新

應用連接到 Google Play services API 之後，它已經準備好開始接收位置更新了。當系統為我們的客戶端調用 [onConnected()](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.ConnectionCallbacks.html#onConnected(android.os.Bundle)) 回調函數時，我們可以按照下面的步驟構建位置更新請求：

1. 創建一個 [LocationRequest](http://developer.android.com/reference/com/google/android/gms/location/LocationRequest.html) 對象並且用像 [setPriority()](http://developer.android.com/reference/com/google/android/gms/location/LocationRequest.html#setPriority(int)) 這樣的方法設置選項。
2. 使用 <a href="http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html#requestLocationUpdates(com.google.android.gms.common.api.GoogleApiClient, com.google.android.gms.location.LocationRequest, com.google.android.gms.location.LocationListener)">requestLocationUpdates()</a> 請求位置更新。
3. 在 [onPause()]() 方法裡使用 <a href="http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html#removeLocationUpdates(com.google.android.gms.common.api.GoogleApiClient, com.google.android.gms.location.LocationListener)">removeLocationUpdates()</a> 刪除位置更新。

下面的例子介紹瞭如何接收和刪除位置更新：

```java
@Override
public void onConnected(Bundle bundle) {
    LocationRequest locationRequest = LocationRequest.create()
            .setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY)
            .setInterval(UPDATE_INTERVAL_MS)
            .setFastestInterval(FASTEST_INTERVAL_MS);

    LocationServices.FusedLocationApi
            .requestLocationUpdates(mGoogleApiClient, locationRequest, this)
            .setResultCallback(new ResultCallback() {

                @Override
                public void onResult(Status status) {
                    if (status.getStatus().isSuccess()) {
                        if (Log.isLoggable(TAG, Log.DEBUG)) {
                            Log.d(TAG, "Successfully requested location updates");
                        }
                    } else {
                        Log.e(TAG,
                                "Failed in requesting location updates, "
                                        + "status code: "
                                        + status.getStatusCode()
                                        + ", message: "
                                        + status.getStatusMessage());
                    }
                }
            });
}

@Override
protected void onPause() {
    super.onPause();
    if (mGoogleApiClient.isConnected()) {
        LocationServices.FusedLocationApi
             .removeLocationUpdates(mGoogleApiClient, this);
    }
    mGoogleApiClient.disconnect();
}

@Override
public void onConnectionSuspended(int i) {
    if (Log.isLoggable(TAG, Log.DEBUG)) {
        Log.d(TAG, "connection to location client suspended");
    }
}
```

至此，我們已經打開了位置更新，系統調用 [onLocationChanged()](http://developer.android.com/reference/android/location/LocationListener.html#onLocationChanged(android.location.Location)) 方法，同時按照 [setInterval()](http://developer.android.com/reference/com/google/android/gms/location/LocationRequest.html#setInterval(long)) 指定的時間間隔更新位置。

## 檢測設備上的 GPS

不是所有的可穿戴設備都有 GPS 感應器。如果用戶出去外面並且將他們的手機放在家裡，那麼我們的可穿戴應用無法通過一個綁定連接來接收位置數據。如果可穿戴設備沒有 GPS 感應器，那麼我們應該檢測到這種情況並且警告用戶位置功能不可用。

使用 [hasSystemFeature()](http://developer.android.com/reference/android/content/pm/PackageManager.html#hasSystemFeature(java.lang.String)) 方法確定 Android Wear 設備是否有內置的 GPS 感應器。下面的代碼用於當我們啟動一個 activity 時，檢測設備是否有內置的 GPS 感應器：

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    setContentView(R.layout.main_activity);
    if (!hasGps()) {
        Log.d(TAG, "This hardware doesn't have GPS.");
        // Fall back to functionality that does not use location or
        // warn the user that location function is not available.
    }

    ...
}

private boolean hasGps() {
    return getPackageManager().hasSystemFeature(PackageManager.FEATURE_LOCATION_GPS);
}
```

## 處理斷開事件

可穿戴設備在回答綁定連接位置數據時可能會突然斷開連接。如果我們的可穿戴應用期待持續的數據，那麼我們必須處理數據中斷或者不可用的斷線問題。在一個不帶有 GPS 感應器的可穿戴設備上，當設備與綁定數據連接斷開時，位置數據會丟失。

以防基於綁定位置數據連接的應用和可穿戴設備沒有 GPS 感應器，我們應該檢測連接的斷線，警告用戶和優雅地降低應用的功能。

為了檢測數據連接的斷線：

1. 繼承 [WearableListenerService](http://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService.html) 來監聽重要的數據層事件。
2. 在 Android manifest 文件中聲明一個 intent filter 來把[WearableListenerService](http://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService.html) 通知給系統。這個 filter 允許系統按需綁定我們的服務。
```xml
<service android:name=".NodeListenerService">
    <intent-filter>
        <action android:name="com.google.android.gms.wearable.BIND_LISTENER" />
    </intent-filter>
</service>
```
3. 實現 [onPeerDisconnected()](http://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService.html#onPeerDisconnected(com.google.android.gms.wearable.Node)) 方法並處理設備是否有內置 GPS 的情況。

```java
public class NodeListenerService extends WearableListenerService {

    private static final String TAG = "NodeListenerService";

    @Override
    public void onPeerDisconnected(Node peer) {
        Log.d(TAG, "You have been disconnected.");
        if(!hasGPS()) {
            // Notify user to bring tethered handset
            // Fall back to functionality that does not use location
        }
    }
    ...
}
```

更多相關的信息，請見 [監聽數據層事件](http://hukai.me/android-training-course-in-chinese/wearables/data-layer/events.html#Listen) 指南。

## 處理找不到位置的情況

當 GPS 信號丟失了，我們仍然可以使用 [getLastLocation()](http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html#getLastLocation(com.google.android.gms.common.api.GoogleApiClient)) 檢索最後可知位置。這個方法在我們無法修復 GPS 連接或者設備沒有內置 GPS 並且斷開與手機連接的情況下很有用。

下面的代碼使用 [getLastLocation()](http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html#getLastLocation(com.google.android.gms.common.api.GoogleApiClient)) 檢索最後可知位置：

```java
Location location = LocationServices.FusedLocationApi
                .getLastLocation(mGoogleApiClient);
```

## 同步數據

如果可穿戴應用使用內置 GPS 記錄數據，那麼我們可能想要與手持應用同步位置數據。對於 [LocationListener](http://developer.android.com/reference/android/location/LocationListener.html)，我們可以實現 [onLocationChanged()](http://developer.android.com/reference/android/location/LocationListener.html#onLocationChanged(android.location.Location)) 方法來檢測和記錄它改變的位置。

下面的可穿戴應用代碼檢測位置變化和使用數據層 API 來保存用於手機應用日後檢索的數據：

```java
@Override
public void onLocationChanged(Location location) {
    ...
    addLocationEntry(location.getLatitude(), location.getLongitude());

}

private void addLocationEntry(double latitude, double longitude) {
    if (!mSaveGpsLocation || !mGoogleApiClient.isConnected()) {
        return;
    }

    mCalendar.setTimeInMillis(System.currentTimeMillis());

    // Set the path of the data map
    String path = Constants.PATH + "/" + mCalendar.getTimeInMillis();
    PutDataMapRequest putDataMapRequest = PutDataMapRequest.create(path);

    // Set the location values in the data map
    putDataMapRequest.getDataMap()
            .putDouble(Constants.KEY_LATITUDE, latitude);
    putDataMapRequest.getDataMap()
            .putDouble(Constants.KEY_LONGITUDE, longitude);
    putDataMapRequest.getDataMap()
            .putLong(Constants.KEY_TIME, mCalendar.getTimeInMillis());

    // Prepare the data map for the request
    PutDataRequest request = putDataMapRequest.asPutDataRequest();

    // Request the system to create the data item
    Wearable.DataApi.putDataItem(mGoogleApiClient, request)
            .setResultCallback(new ResultCallback() {
                @Override
                public void onResult(DataApi.DataItemResult dataItemResult) {
                    if (!dataItemResult.getStatus().isSuccess()) {
                        Log.e(TAG, "Failed to set the data, "
                                + "status: " + dataItemResult.getStatus()
                                .getStatusCode());
                    }
                }
            });
}
```

更多關於如何使用數據層 API 的內容，請見 [發送與同步數據](http://hukai.me/android-training-course-in-chinese/wearables/data-layer/index.html) 指南。