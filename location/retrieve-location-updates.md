# 獲取位置更新

> 編寫:[penkzhou](https://github.com/penkzhou) - 原文:<http://developer.android.com/training/location/receive-location-updates.html>

如果我們的應用可以週期性地跟蹤位置，那麼應用可以給用戶提供更多相關信息。例如，如果我們的應用在用戶行走或者駕車時幫助找到他們的路，或者如果我們的應用跟蹤用戶的位置，那麼它需要定期獲取設備的位置。除了地理位置之外（經度和緯度），我們可能還想為用戶提供更多的信息，例如方位（行駛的水平方向）、海拔或者設備的速度。這些信息可以在 [Location](http://developer.android.com/reference/android/location/Location.html) 對象中獲得，我們的應用可以從 [fused location provider](http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html) 中得到這個對象。

當我們用 [getLastLocation()](http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html#getLastLocation(com.google.android.gms.common.api.GoogleApiClient)) 獲取設備的位置時，如上一節課[獲取最後可知位置](retrieve-current.html)介紹的一樣，一個更加直接的方法是從 fused location provider 中請求週期性的更新。作為迴應，API根據現有的位置供應源，如Wifi和GPS（Global Positioning System），用最佳位置週期地更新我們的應用。這些providers、我們請求的權限和我們在位置請求中設置的選項決定了位置的精確度。

這節課介紹如何用fused location provider的<a href="http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html#requestLocationUpdates(com.google.android.gms.common.api.GoogleApiClient, com.google.android.gms.location.LocationRequest, com.google.android.gms.location.LocationListener)">requestLocationUpdates()</a>方法來請求定期更新設備的位置。

## 連接Location Services

應用的 Location Services 由 Google Play services 和 fused location provider 提供。為了用這些服務，用 Google API Client 連接到我們的應用，然後請求位置更新。用 [GoogleApiClient](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.html) 進行連接的詳細步驟請見[獲取最後可知位置](retrieve-current.html)，包括了請求當前位置。

設備的最後可知位置提供有關起點的基準信息，在開始定期更新位置信息前，保證應用擁有一個可知的位置。[獲取最後可知位置](retrieve-current.html)介紹瞭如何通過調用 [getLastLocation()](http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html#getLastLocation(com.google.android.gms.common.api.GoogleApiClient)) 獲取最後可知位置。接下來的內容假設我們的應用已經取得最後可知位置，並已將最後可知位置作為一個 [Location](http://developer.android.com/reference/android/location/Location.html) 對象保存在全局變量 `mCurrentLocation`中。

使用位置服務的應用必須請求位置權限。在這節課中我們需要很好的定位檢測，使得我們的應用可以從可用的位置供應源得到儘可能精確的位置數據。在我們應用的manifest文件中，用`uses-permission`節點請求位置權限，如下所示：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.google.android.gms.location.sample.locationupdates" >

  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
</manifest>
```

## 設置位置請求

創建一個 [LocationRequest](http://developer.android.com/reference/com/google/android/gms/location/LocationRequest.html) 以保存請求 fused location provider 的參數。這些參數決定了請求精確度的水平。對於位置請求中所有可用的選項，請見 [LocationRequest](http://developer.android.com/reference/com/google/android/gms/location/LocationRequest.html) 類的參考文檔。這節課設置更新間隔、最快更新間隔和優先級。如下所述：

**更新間隔**

[setInterval()](http://developer.android.com/reference/com/google/android/gms/location/LocationRequest.html#setInterval(long)) - 這個方法設置應用接收位置更新的速率（每毫秒）。注意如果另一個應用正在接收一個更快的或者更慢的更新速率，又或者根本沒有更新（例如，設備還沒有連接），那麼我們應用的位置更新速率可能會比`setInterval()`設置的速率更快。

**最快更新間隔**

[setFastestInterval()](http://developer.android.com/reference/com/google/android/gms/location/LocationRequest.html#setFastestInterval(long)) - 這個方法設置應用可以處理位置更新的**最快**速率（每毫秒）。因為其它應用會影響到已發送出去的位置更新的速率，所以我們需要設置這個最快速率。Google Play services location APIs 發送任何應用用 [setInterval()](http://developer.android.com/reference/com/google/android/gms/location/LocationRequest.html#setInterval(long)) 請求的最快的更新速率。如果這個速率比我們的應用可以處理的速率還要快，那麼我們可能會遇到UI閃爍或者數據溢出等問題。為了避免這個問題，調用 [setFastestInterval()](http://developer.android.com/reference/com/google/android/gms/location/LocationRequest.html#setFastestInterval(long)) 限制更新速率的上限。

**優先級**

[setPriority()](http://developer.android.com/reference/com/google/android/gms/location/LocationRequest.html#setPriority(int)) - 這個方法設置請求的優先級，為 Google Play services 位置服務提供了關於使用哪個位置源的強烈的暗示。支持下面幾個值：

* <a href="http://developer.android.com/reference/com/google/android/gms/location/LocationRequest.html#PRIORITY_BALANCED_POWER_ACCURACY">PRIORITY\_BALANCED\_POWER\_ACCURACY</a> - 這個設置請求一個城市街區範圍的位置精確度（精確度約為100米）。這被認為是一個粗略的精確度，也可能是耗電較小的設置。對於這個設置，位置服務可能使用 WiFi 和基站進行定位。注意，無論如何，位置供應源的選擇依賴於很多其它的因素。

* <a href="http://developer.android.com/reference/com/google/android/gms/location/LocationRequest.html#PRIORITY_HIGH_ACCURACY">PRIORITY\_HIGH\_ACCURACY</a> - 這個設置請求最高精度的位置信息。對於這個設置，位置服務更可能使用 GPS(Global Positioning System) 來定位。

* <a href="http://developer.android.com/reference/com/google/android/gms/location/LocationRequest.html#PRIORITY_LOW_POWER"> PRIORITY\_LOW\_POWER</a> - 這個設置請求一個城市範圍的精確度（精確度約為10公里）。這被認為是一個粗略的精確度，也可能是耗電較小的設置。

* <a href="http://developer.android.com/reference/com/google/android/gms/location/LocationRequest.html#PRIORITY_NO_POWER">PRIORITY\_NO\_POWER</a> - 如果需要對功率消耗的影響微乎其微，但又想在可用的時候接收位置更新，那麼使用這個設置。對於這個設置，我們的應用不會觸發任何位置更新，但是會接收由其它應用觸發的位置。

下面的示例介紹創建位置請求和設置相關的參數：

```java
protected void createLocationRequest() {
    LocationRequest mLocationRequest = new LocationRequest();
    mLocationRequest.setInterval(10000);
    mLocationRequest.setFastestInterval(5000);
    mLocationRequest.setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY);
}
```

<a href="http://developer.android.com/reference/com/google/android/gms/location/LocationRequest.html#PRIORITY_HIGH_ACCURACY">PRIORITY\_HIGH\_ACCURACY</a> 的優先級聯合在我們應用的 manifest 文件中定義的 <a href="http://developer.android.com/reference/android/Manifest.permission.html#ACCESS_FINE_LOCATION">ACCESS\_FINE\_LOCATION</a> 權限和一個5000毫秒（5秒）的更新間隔。該優先級使 fused location provider 返回精確到幾英尺之內的位置更新。這個方法適用於需要實時顯示位置的地圖應用。

> **性能提示：**如果我們的應用在接收一個位置更新後接入網絡或者執行持續時間長的工作，那麼將最快更新間隔調整到一個更慢的值。這個調整防止我們的應用接收不可用的更新。一旦持續時間長的工作完成，將最快更新間隔改回一個快的值。

## 請求位置更新

我們已經設置了包含應用位置更新要求的位置請求，我們可以調用 <a href="http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html#requestLocationUpdates(com.google.android.gms.common.api.GoogleApiClient, com.google.android.gms.location.LocationRequest, com.google.android.gms.location.LocationListener)">requestLocationUpdates()</a> 來啟動週期性的更新。在 Google API Client 提供的 [onConnected()](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.ConnectionCallbacks.html#onConnected(android.os.Bundle)) 回調函數（當 client 準備好之後會調用這個回調函數）中啟動週期性更新。 

根據請求的形式，fused location provider 要麼調用 [LocationListener.onLocationChanged()](http://developer.android.com/reference/com/google/android/gms/location/LocationListener.html) 回調函數並傳遞一個 [Location](http://developer.android.com/reference/android/location/Location.html) 對象，要麼發出一個將位置信息包含在擴展數據的 [PendingIntent](http://developer.android.com/reference/android/app/PendingIntent.html)。更新的精確度和頻率受已請求的位置權限和在位置請求對象中設置的選項等因素影響。

這節課介紹如何使用 [LocationListener](http://developer.android.com/reference/com/google/android/gms/location/LocationListener.html) 回調函數獲取位置更新。調用 <a href="http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html#requestLocationUpdates(com.google.android.gms.common.api.GoogleApiClient, com.google.android.gms.location.LocationRequest, com.google.android.gms.location.LocationListener)">requestLocationUpdates()</a> ,並傳入 [GoogleApiClient](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.html) 的實例、[LocationRequest](http://developer.android.com/reference/com/google/android/gms/location/LocationRequest.html) 對象和一個 [LocationListener](http://developer.android.com/reference/com/google/android/gms/location/LocationListener.html)。定義一個 `startLocationUpdates()` 方法，該方法在 [onConnected()](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.ConnectionCallbacks.html#onConnected(android.os.Bundle)) 回調函數被調用，如下面的示例代碼所示：

```java
@Override
public void onConnected(Bundle connectionHint) {
    ...
    if (mRequestingLocationUpdates) {
        startLocationUpdates();
    }
}

protected void startLocationUpdates() {
    LocationServices.FusedLocationApi.requestLocationUpdates(
            mGoogleApiClient, mLocationRequest, this);
}
```

注意到上述的代碼片段提到一個布爾標誌位，`mRequestingLocationUpdates`，該標誌位用於判斷用戶將位置更新打開還是關閉。關於這個標誌位更詳細的介紹，請見下面的[保存 Activity 的狀態]()的內容。

## 定義位置更新回調函數

fused location provider 調用 [LocationListener.onLocationChanged()](http://developer.android.com/reference/com/google/android/gms/location/LocationListener.html#onLocationChanged(android.location.Location)) 回調函數。這個回調函數傳入的參數是一個含有位置經緯度的 [Location](http://developer.android.com/reference/android/location/Location.html) 對象。下面的代碼介紹瞭如何實現 [LocationListener](http://developer.android.com/reference/com/google/android/gms/location/LocationListener.html) 接口和定義方法，然後獲取位置更新的時間戳並在應用用戶界面上顯示經度、緯度和時間戳：

```java
public class MainActivity extends ActionBarActivity implements
        ConnectionCallbacks, OnConnectionFailedListener, LocationListener {
    ...
    @Override
    public void onLocationChanged(Location location) {
        mCurrentLocation = location;
        mLastUpdateTime = DateFormat.getTimeInstance().format(new Date());
        updateUI();
    }

    private void updateUI() {
        mLatitudeTextView.setText(String.valueOf(mCurrentLocation.getLatitude()));
        mLongitudeTextView.setText(String.valueOf(mCurrentLocation.getLongitude()));
        mLastUpdateTimeTextView.setText(mLastUpdateTime);
    }
}
```

## 停止位置更新

我們需要考慮當 activity 不在焦點上時我們是否需要停止位置更新，例如，當用戶切換到另一個應用或者同一個應用的不同 activity 的情況。假如應用即使在後臺運行時也不需要收集用戶數據，將會有利於降低功耗。這節課會介紹如何在 activity 的 [onPause()](http://developer.android.com/reference/android/app/Activity.html#onPause()) 方法裡停止位置更新。

為了停止位置更新，調用 <a href="http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html#removeLocationUpdates(com.google.android.gms.common.api.GoogleApiClient, com.google.android.gms.location.LocationListener)">removeLocationUpdates()</a>，並傳入 [GoogleApiClient](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.html) 對象的實例和一個 [LocationListener](http://developer.android.com/reference/com/google/android/gms/location/LocationListener.html)，如下面的示例代碼所示：

```java
@Override
protected void onPause() {
    super.onPause();
    stopLocationUpdates();
}

protected void stopLocationUpdates() {
    LocationServices.FusedLocationApi.removeLocationUpdates(
            mGoogleApiClient, this);
}
```

使用一個布爾值，`mRequestingLocationUpdates`，來判斷當前位置更新是否打開。在 activity 的 [onResume()](http://developer.android.com/reference/android/app/Activity.html#onResume()) 方法裡，檢查當前的位置更新是否起作用。如果位置更新不起作用，那麼激活它：

```java
@Override
public void onResume() {
    super.onResume();
    if (mGoogleApiClient.isConnected() && !mRequestingLocationUpdates) {
        startLocationUpdates();
    }
}
```

## 保存 Activity 的狀態

一個設備配置的變動，如旋轉屏幕或者改變語言，可以導致當前的 activity 崩潰。我們的應用必須保存任何在重新創建 activity 時需要用到的信息。一種方法是通過一個保存在 [Bundle](http://developer.android.com/reference/android/os/Bundle.html) 對象的實例狀態來解決這個問題。

下面的示例代碼介紹瞭如何用 activity 的 [onSaveInstanceState()](http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle)) 回調函數來保存實例狀態：

```java
public void onSaveInstanceState(Bundle savedInstanceState) {
    savedInstanceState.putBoolean(REQUESTING_LOCATION_UPDATES_KEY,
            mRequestingLocationUpdates);
    savedInstanceState.putParcelable(LOCATION_KEY, mCurrentLocation);
    savedInstanceState.putString(LAST_UPDATED_TIME_STRING_KEY, mLastUpdateTime);
    super.onSaveInstanceState(savedInstanceState);
}
```

定義一個 `updateValuesFromBundle()` 方法來恢復保存在 activity 的上一個實例的值（如果這些值可用的話）。在 [onCreate()](http://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle)) 中調用這個方法。如下所示：

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    ...
    updateValuesFromBundle(savedInstanceState);
}

private void updateValuesFromBundle(Bundle savedInstanceState) {
    if (savedInstanceState != null) {
        // Update the value of mRequestingLocationUpdates from the Bundle, and
        // make sure that the Start Updates and Stop Updates buttons are
        // correctly enabled or disabled.
        if (savedInstanceState.keySet().contains(REQUESTING_LOCATION_UPDATES_KEY)) {
            mRequestingLocationUpdates = savedInstanceState.getBoolean(
                    REQUESTING_LOCATION_UPDATES_KEY);
            setButtonsEnabledState();
        }

        // Update the value of mCurrentLocation from the Bundle and update the
        // UI to show the correct latitude and longitude.
        if (savedInstanceState.keySet().contains(LOCATION_KEY)) {
            // Since LOCATION_KEY was found in the Bundle, we can be sure that
            // mCurrentLocationis not null.
            mCurrentLocation = savedInstanceState.getParcelable(LOCATION_KEY);
        }

        // Update the value of mLastUpdateTime from the Bundle and update the UI.
        if (savedInstanceState.keySet().contains(LAST_UPDATED_TIME_STRING_KEY)) {
            mLastUpdateTime = savedInstanceState.getString(
                    LAST_UPDATED_TIME_STRING_KEY);
        }
        updateUI();
    }
}
```

更多關於保存實例狀態的內容，請看 [Android Activity](http://developer.android.com/reference/android/app/Activity.html#ConfigurationChanges) 類的參考文檔。

> **Note：**為了可以更加持久地存儲，我們可以將用戶的偏好設定保存在應用的 [SharedPreferences](http://developer.android.com/reference/android/content/SharedPreferences.html) 中。在 activity 的 [onPause()](http://developer.android.com/reference/android/app/Activity.html#onPause()) 方法中設置偏好設定，在 [onResume()](http://developer.android.com/reference/android/app/Activity.html#onResume()) 中獲取這些設定。更多關於偏好設定的內容，請見[保存到 Rreference](http://hukai.me/android-training-course-in-chinese/basics/data-storage/shared-preference.html)。

下一節課，[顯示位置地址](display-address.html)，介紹如何顯示指定位置的街道地址。