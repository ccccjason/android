# 訪問可穿戴數據層

> 編寫:[wly2014](https://github.com/wly2014) - 原文: <http://developer.android.com/training/wearables/data-layer/accessing.html>

調用數據層API，需創建一個 [GoogleApiClient](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.html) 實例，所有 Google Play services APIs的主要入口點。

[GoogleApiClient](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.html) 提供了一個易於創建客戶端實例的builder。最簡單的[GoogleApiClient](GoogleApiClient.html)如下：

> **Note:** 目前，此小client僅足以能啟動。但是，更多創建GoogleApiClient，實現回調方法和處理錯誤等內容，詳見 [Accessing Google Play services APIs](http://developer.android.com/google/auth/api-client.html)。

```java
GoogleApiClient mGoogleApiClient = new GoogleApiClient.Builder(this)
        .addConnectionCallbacks(new ConnectionCallbacks() {
                @Override
                public void onConnected(Bundle connectionHint) {
                    Log.d(TAG, "onConnected: " + connectionHint);
                    // Now you can use the Data Layer API
                }
                @Override
                public void onConnectionSuspended(int cause) {
                    Log.d(TAG, "onConnectionSuspended: " + cause);
                }
        })
        .addOnConnectionFailedListener(new OnConnectionFailedListener() {
                @Override
                public void onConnectionFailed(ConnectionResult result) {
                    Log.d(TAG, "onConnectionFailed: " + result);
                }
            })
        // Request access only to the Wearable API
        .addApi(Wearable.API)
        .build();
```

> **Important:** 如果我們添加多個API到一個GoogleApiClient，那麼可能會在沒有安裝[Android Wear app ](https://play.google.com/store/apps/details?id=com.google.android.wearable.app&hl=en) 的設備上遇到連接錯誤。為了連接錯誤，調用<a href="http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.Builder.html#addApiIfAvailable(com.google.android.gms.common.api.Api<? extends com.google.android.gms.common.api.Api.ApiOptions.NotRequiredOptions>, com.google.android.gms.common.api.Scope...)">addApiIfAvailable()</a>方法，並以[Wearable](http://developer.android.com/reference/com/google/android/gms/wearable/Wearable.html) API為參數傳進該方法，從而表明client應該處理缺失的API。更多的信息，請見 [Access the Wearable API](http://developer.android.com/google/auth/api-client.html#WearableApi).

在使用數據層API之前，通過調用[connect()](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.html#connect())方法進行連接，如 [Start a Connection](http://developer.android.com/google/auth/api-client.html#Starting) 中所述。當系統為我們的客戶端調用了[onConnected()](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.ConnectionCallbacks.html#onConnected(android.os.Bundle)) 方法，我們就可以使用數據層API了。

