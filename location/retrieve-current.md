# 獲取最後可知位置

> 編寫:[penkzhou](https://github.com/penkzhou) - 原文:<http://developer.android.com/training/location/retrieve-current.html>

使用Google Play services location APIs，我們的應用可以請求獲得用戶設備的最後可知位置。大多數情況下，我們會對用戶的當前位置比較感興趣。而通常用戶的當前位置相當於設備的最後可知位置。

特別地，使用[fused location provider](http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html)來獲取設備的最後可知位置。fused location provider是Google Play services location APIs中的一個。它處理基本定位技術並提供一個簡單的API，使得我們可以指定高水平的需求，如高精度或者低功耗。同時它優化了設備的耗電情況。

這節課介紹如何通過使用fused location provider的[getLastLocation()](http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html#getLastLocation(com.google.android.gms.common.api.GoogleApiClient))方法為設備的位置構造一個單一請求。

## 安裝Google Play Services
為了訪問fused location provider，我們的應用開發工程必須包括Google Play services。通過[SDK Manager](http://developer.android.com/tools/help/sdk-manager.html)下載和安裝Google Play services組件，添加相關的庫到我們的工程。更詳細的介紹，請看[Setting Up Google Play Services](http://developer.android.com/google/play-services/setup.html)。

## 確定應用的權限

使用位置服務的應用必須請求用戶位置權限。Android擁有兩種位置權限：[ACCESS_COARSE_LOCATION](http://developer.android.com/reference/android/Manifest.permission.html#ACCESS_COARSE_LOCATION) 和 [ACCESS_FINE_LOCATION](http://developer.android.com/reference/android/Manifest.permission.html#ACCESS_FINE_LOCATION)。我們選擇的權限決定API返回的位置信息的精度。如果我們選擇了[ACCESS_COARSE_LOCATION](http://developer.android.com/reference/android/Manifest.permission.html#ACCESS_COARSE_LOCATION)，API返回的位置信息的精確度大體相當於一個城市街區。

這節課只要求粗略的定位。在我們應用的manifest文件中，用`uses-permission`節點請求這個權限，如下所示：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.google.android.gms.location.sample.basiclocationsample" >
  
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
</manifest>
```

## 連接Google Play Services

為了連接到API，我們需要創建一個Google Play services API客戶端實例。關於使用這個客戶端的更詳細的介紹，請看[Accessing Google APIs](http://developer.android.com/google/auth/api-client.html#Starting)。

在我們的activity的[onCreate()](http://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle))方法中，用[GoogleApiClient.Builder](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.Builder.html)創建一個Google API Client實例。使用這個builder添加[LocationServices](http://developer.android.com/reference/com/google/android/gms/location/LocationServices.html) API。

實例應用定義了一個`buildGoogleApiClient()`方法，這個方法在activity的onCreate()方法中被調用。`buildGoogleApiClient()`方法包括下面的代碼。

```java
protected synchronized void buildGoogleApiClient() {
    mGoogleApiClient = new GoogleApiClient.Builder(this)
        .addConnectionCallbacks(this)
        .addOnConnectionFailedListener(this)
        .addApi(LocationServices.API)
        .build();
}
```

## 獲取最後可知位置

一旦我們將Google Play services和location services API連接完成後，我們就可以獲取用戶設備的最後可知位置。當我們的應用連接到這些服務之後，我們可以用fused location provider的[getLastLocation()](http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html#getLastLocation(com.google.android.gms.common.api.GoogleApiClient))方法來獲取設備的位置。調用這個方法返回的定位精確度是由我們在應用的manifest文件裡添加的權限決定的，如本文的[確定應用的權限]()部分描述的內容一樣。

為了請求最後可知位置，調用[getLastLocation()](http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html#getLastLocation(com.google.android.gms.common.api.GoogleApiClient))方法，並將我們創建的[GoogleApiClient](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.html)對象的實例傳給該方法。在Google API Client提供的[onConnected()](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.ConnectionCallbacks.html#onConnected(android.os.Bundle))回調函數裡調用[getLastLocation()](http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html#getLastLocation(com.google.android.gms.common.api.GoogleApiClient))方法，這個回調函數在client準備好的時候被調用。下面的示例代碼說明了請求和一個對響應簡單的處理：

```java
public class MainActivity extends ActionBarActivity implements
        ConnectionCallbacks, OnConnectionFailedListener {
    ...
    @Override
    public void onConnected(Bundle connectionHint) {
        mLastLocation = LocationServices.FusedLocationApi.getLastLocation(
                mGoogleApiClient);
        if (mLastLocation != null) {
            mLatitudeText.setText(String.valueOf(mLastLocation.getLatitude()));
            mLongitudeText.setText(String.valueOf(mLastLocation.getLongitude()));
        }
    }
}
```

[getLastLocation()](http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html#getLastLocation(com.google.android.gms.common.api.GoogleApiClient))方法返回一個[Location](http://developer.android.com/reference/android/location/Location.html)對象。通過[Location](http://developer.android.com/reference/android/location/Location.html)對象，我們可以取得地理位置的經度和緯度座標。在少數情況下，當位置不可用時，這個Location對象會返回null。

下一課，[獲取位置更新](receive-location-updates.html)，教你如何週期性地獲取位置信息更新。

