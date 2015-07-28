# 創建和監視地理圍欄

> 編寫:[penkzhou](https://github.com/penkzhou) - 原文:<http://developer.android.com/training/location/geofencing.html>

地理圍欄將用戶當前位置感知和附件地點特徵感知相結合。為了標示一個感興趣的位置，我們需要指定這個位置的經緯度。為了調整位置的鄰近度，需要添加一個半徑。經緯度和半徑定義一個地理圍欄，即在感興趣的位置創建一個圓形區域或者圍欄。

我們可以有多個活動的地理圍欄（限制是一個設備用戶100個）。對於每個地理圍欄，我們可以讓 Location Services 發出進入和離開事件，或者我們可以在觸發一個事件之前，指定在某個地理圍欄區域等待一段時間或者停留。通過指定一個以毫秒為單位的截止時間，我們可以限制任何一個地理圍欄的持續時間。當地理圍欄失效後，Location Services 會自動刪除這個地理圍欄。

![geofence](geofence.png)

這節課介紹如何添加和刪除地理圍欄，和用 [IntentService](http://developer.android.com/reference/android/app/IntentService.html) 監聽地理位置變化。

## 設置地理圍欄監視

請求地理圍欄監視的第一步就是設置必要的權限。在使用地理圍欄時，我們必須設置 <a href="http://developer.android.com/reference/android/Manifest.permission.html#ACCESS_FINE_LOCATION">ACCESS\_FINE\_LOCATION</a> 權限。在應用的 manifest 文件中添加如下子節點即可：

```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
```

如果想要用 [IntentService](http://developer.android.com/reference/android/app/IntentService.html) 監聽地理位置變化，那麼還需要添加一個節點來指定服務名字。這個節點必須是 [<application>](http://developer.android.com/guide/topics/manifest/application-element.html) 的子節點：

```xml
<application
   android:allowBackup="true">
   ...
   <service android:name=".GeofenceTransitionsIntentService"/>
<application/>
```

為了訪問位置 API，我們需要創建一個 Google Play services API client 的實例。想要學習如何連接 client，請見[連接Google Play Services](retrieve-current.html)。

### 創建和添加地理圍欄

我們的應用需要用位置 API 的 builder 類來創建地理圍欄，用 convenience 類來添加地理圍欄。另外，我們可以定義一個 [PendingIntent](http://developer.android.com/reference/android/app/PendingIntent.html)（將在這節課介紹）來處理當地理位置發生遷移時，Location Services 發出的 intent。

### 創建地理圍欄對象

首先，用 [Geofence.Builder](http://developer.android.com/reference/com/google/android/gms/location/Geofence.Builder.%20%20%20%20html) 創建一個地理圍欄，設置想要的半徑，持續時間，和地理圍欄遷移的類型。例如，填充一個叫做 `mGeofenceList` 的 list 對象：

```java
mGeofenceList.add(new Geofence.Builder()
    // Set the request ID of the geofence. This is a string to identify this
    // geofence.
    .setRequestId(entry.getKey())

    .setCircularRegion(
            entry.getValue().latitude,
            entry.getValue().longitude,
            Constants.GEOFENCE_RADIUS_IN_METERS
    )
    .setExpirationDuration(Constants.GEOFENCE_EXPIRATION_IN_MILLISECONDS)
    .setTransitionTypes(Geofence.GEOFENCE_TRANSITION_ENTER |
            Geofence.GEOFENCE_TRANSITION_EXIT)
    .build());
```

這個例子從一個固定的文件中獲取數據。在實際情況下，應用可能會根據用戶的位置動態地創建地理圍欄。

### 指定地理圍欄和初始化觸發器

下面的代碼用到 [GeofencingRequest](http://developer.android.com/reference/com/google/android/gms/location/GeofencingRequest.html) 類。該類嵌套了 [GeofencingRequestBuilder](http://developer.android.com/reference/com/google/android/gms/location/GeofencingRequest.Builder.html) 類來需要監視的地理圍欄和設置如何觸發地理圍欄事件：

```java
private GeofencingRequest getGeofencingRequest() {
    GeofencingRequest.Builder builder = new GeofencingRequest.Builder();
    builder.setInitialTrigger(GeofencingRequest.INITIAL_TRIGGER_ENTER);
    builder.addGeofences(mGeofenceList);
    return builder.build();
}
```

這個例子介紹了兩個地理圍欄觸發器。當設備進入一個地理圍欄時， <a href="http://developer.android.com/reference/com/google/android/gms/location/Geofence.html#GEOFENCE_TRANSITION_ENTER">GEOFENCE\_TRANSITION\_ENTER</a> 轉移會觸發。當設備離開一個地理圍欄時， <a href="http://developer.android.com/reference/com/google/android/gms/location/Geofence.html#GEOFENCE_TRANSITION_EXIT">GEOFENCE\_TRANSITION\_EXIT</a> 轉移會觸發。如果設備已經在地理圍欄裡面，那麼指定 <a href="http://developer.android.com/reference/com/google/android/gms/location/GeofencingRequest.html#INITIAL_TRIGGER_ENTER">INITIAL\_TRIGGER\_ENTER</a> 來通知位置服務觸發 <a href="http://developer.android.com/reference/com/google/android/gms/location/Geofence.html#GEOFENCE_TRANSITION_ENTER">GEOFENCE\_TRANSITION\_ENTER</a>。

在很多情況下，使用 <a href="http://developer.android.com/reference/com/google/android/gms/location/GeofencingRequest.html#INITIAL_TRIGGER_DWELL">INITIAL\_TRIGGER\_DWELL</a> 可能會更好。僅僅當由於到達地理圍欄中已定義好的持續時間，而導致用戶停止時，<a href="http://developer.android.com/reference/com/google/android/gms/location/GeofencingRequest.html#INITIAL_TRIGGER_DWELL">INITIAL\_TRIGGER\_DWELL</a> 才會觸發事件。這個方法可以減少當設備短暫地進入和離開地理圍欄時，由大量的通知造成的“垃圾警告信息”。另一種獲取最好的地理圍欄結果的策略是設置最小半徑為100米。這有助於估計典型的 Wifi 網絡的位置精確度，也有利於降低設備的功耗。

### 為地理圍欄轉移定義Intent

從 Location Services 發送來的Intent能夠觸發各種應用內的動作，但是不能用它來打開一個 Activity 或者 Fragment，這是因為應用內的組件只能在響應用戶動作時才可見。大多數情況下，處理這一類 Intent 最好使用 [IntentService](http://developer.android.com/reference/android/app/IntentService.html)。一個 [IntentService](http://developer.android.com/reference/android/app/IntentService.html) 可以推送一個通知，可以進行長時間的後臺作業，可以將 intent 發送給其他的 services ，還可以發送一個廣播 intent。下面的代碼展示瞭如何定義一個 [PendingIntent](http://developer.android.com/reference/android/app/PendingIntent.html) 來啟動一個 [IntentService](http://developer.android.com/reference/android/app/IntentService.html):

```java
public class MainActivity extends FragmentActivity {
    ...
    private PendingIntent getGeofencePendingIntent() {
        // Reuse the PendingIntent if we already have it.
        if (mGeofencePendingIntent != null) {
            return mGeofencePendingIntent;
        }
        Intent intent = new Intent(this, GeofenceTransitionsIntentService.class);
        // We use FLAG_UPDATE_CURRENT so that we get the same pending intent back when
        // calling addGeofences() and removeGeofences().
        return PendingIntent.getService(this, 0, intent, PendingIntent.
                FLAG_UPDATE_CURRENT);
    }
```

### 添加地理圍欄

使用 <a href="http://developer.android.com/reference/com/google/android/gms/location/GeofencingApi.html#addGeofences(com.google.android.gms.common.api.GoogleApiClient, com.google.android.gms.location.GeofencingRequest, android.app.PendingIntent)">GeoencingApi.addGeofences()</a> 方法來添加地理圍欄。為該方法提供 Google API client，[GeofencingRequest](http://developer.android.com/reference/com/google/android/gms/location/GeofencingRequest) 對象和 [PendingIntent](http://developer.android.com/reference/android/app/PendingIntent.html)。下面的代碼，在 [onResult()](http://developer.android.com/reference/com/google/android/gms/common/api/ResultCallback.html#onResult(R)) 中處理結果，假設主 activity 實現 [ResultCallback](http://developer.android.com/reference/com/google/android/gms/common/api/ResultCallback.html)。

```java
public class MainActivity extends FragmentActivity {
    ...
    LocationServices.GeofencingApi.addGeofences(
                mGoogleApiClient,
                getGeofencingRequest(),
                getGeofencePendingIntent()
        ).setResultCallback(this);
```

## 處理地理圍欄轉移

當 Location Services 探測到用戶進入或者離開一個地理圍欄，它會發送一個包含在 [PendingIntent](http://developer.android.com/reference/android/app/PendingIntent.html) 的 Intent，這個 [PendingIntent](http://developer.android.com/reference/android/app/PendingIntent.html) 就是在添加地理圍欄時被我們包括在請求當中。這個 Intent 被一個類似 `GeofenceTransitionsIntentService` 的 service 接收，這個 service 從 intent 得到地理圍欄事件，決定地理圍欄轉移的類型，和決定觸發哪個已定義的地理圍欄。然後它會發出一個通知。

下面的代碼介紹瞭如何定義一個 IntentService。這個 IntentService 在地理圍欄轉移出現時，會推送一個通知。當用戶點擊這個通知，那麼應用的主 activity 會出現：

```java
public class GeofenceTransitionsIntentService extends IntentService {
   ...
    protected void onHandleIntent(Intent intent) {
        GeofencingEvent geofencingEvent = GeofencingEvent.fromIntent(intent);
        if (geofencingEvent.hasError()) {
            String errorMessage = GeofenceErrorMessages.getErrorString(this,
                    geofencingEvent.getErrorCode());
            Log.e(TAG, errorMessage);
            return;
        }

        // Get the transition type.
        int geofenceTransition = geofencingEvent.getGeofenceTransition();

        // Test that the reported transition was of interest.
        if (geofenceTransition == Geofence.GEOFENCE_TRANSITION_ENTER ||
                geofenceTransition == Geofence.GEOFENCE_TRANSITION_EXIT) {

            // Get the geofences that were triggered. A single event can trigger
            // multiple geofences.
            List triggeringGeofences = geofencingEvent.getTriggeringGeofences();

            // Get the transition details as a String.
            String geofenceTransitionDetails = getGeofenceTransitionDetails(
                    this,
                    geofenceTransition,
                    triggeringGeofences
            );

            // Send notification and log the transition details.
            sendNotification(geofenceTransitionDetails);
            Log.i(TAG, geofenceTransitionDetails);
        } else {
            // Log the error.
            Log.e(TAG, getString(R.string.geofence_transition_invalid_type,
                    geofenceTransition));
        }
    }
```

在通過 PendingIntent 檢測轉移事件之後，這個 IntentService 獲取地理圍欄轉移類型和測試一個事件是不是應用用來觸發通知的 —— 要麼是 GEOFENCE_TRANSITION_ENTER，要麼是 GEOFENCE_TRANSITION_EXIT。然後，這個 service 會發出一個通知並且記錄轉移的詳細信息。

## 停止地理圍欄監視

當不再需要監視地理圍欄或者想要節省設備的電池電量和 CPU 週期時，需要停止地理圍欄監視。我們可以在用於添加和刪除地理圍欄的主 activity 裡停止地理圍欄監視；刪除地理圍欄會導致它馬上停止。API 要麼通過 request IDs，要麼通過刪除與指定 PendingIntent 相關的地理圍欄來刪除地理圍欄。

下面的代碼通過 PendingIntent 刪除地理圍欄，當設備進入或者離開之前已經添加的地理圍欄時，停止所有通知：

```java
LocationServices.GeofencingApi.removeGeofences(
            mGoogleApiClient,
            // This is the same pending intent that was used in addGeofences().
            getGeofencePendingIntent()
    ).setResultCallback(this); // Result processed in onResult().
}
```

你可以將地理圍欄同其他位置感知的特性結合起來，比如週期性的位置更新。像要了解更多的信息，請看本章的其它課程。
