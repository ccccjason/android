# 顯示位置地址

> 編寫:[penkzhou](https://github.com/penkzhou) - 原文:<http://developer.android.com/training/location/display-address.html>

[獲取最後可知位置](retrieve-current.html)和[獲取位置更新](receive-location-updates.html)課程描述瞭如何以一個[Location](http://developer.android.com/reference/android/location/Location.html)對象的形式獲取用戶的位置信息，這個位置信息包括了經緯度。儘管經緯度對計算地理距離和在地圖上顯示位置很有用，但是更多情況下位置的地址更有用。例如，如果我們想讓用戶知道他們在哪裡，那麼一個街道地址比地理座標（經度/緯度）更加有意義。

使用 Android 框架位置 APIs 的 [Geocoder](http://developer.android.com/reference/android/location/Geocoder.html) 類，我們可以將地址轉換成相應的地理座標。這個過程叫做*地理編碼*。或者，我們可以將地理位置轉換成相應的地址。這種地址查找功能叫做*反向地理編碼*。

這節課介紹瞭如何用 <a href="http://developer.android.com/reference/android/location/Geocoder.html#getFromLocation(double, double, int)">getFromLocation()</a> 方法將地理位置轉換成地址。這個方法返回與制定經緯度相對應的估計的街道地址。

## 獲取地理位置

設備的最後可知位置對於地址查找功能是很有用的基礎。[獲取最後可知位置](retrieve-current.html)介紹瞭如何通過調用 [fused location provider](http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html) 提供的 [getLastLocation()](http://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html#getLastLocation(com.google.android.gms.common.api.GoogleApiClient)) 方法找到設備的最後可知位置。

為了訪問 fused location provider，我們需要創建一個 Google Play services API client 的實例。關於如何連接 client，請見[連接 Google Play Services](retrieve-current.html) 。

為了讓 fused location provider 得到一個準確的街道地址，在應用的 manifest 文件添加位置權限 `ACCESS_FINE_LOCATION`，如下所示：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.google.android.gms.location.sample.locationupdates" >

  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
</manifest>
```

## 定義一個 Intent 服務來取得地址

[Geocoder](http://developer.android.com/reference/android/location/Geocoder.html) 類的 <a href="http://developer.android.com/reference/android/location/Geocoder.html#getFromLocation(double, double, int)">getFromLocation()</a> 方法接收一個經度和緯度，返回一個地址列表。這個方法是同步的，可能會花很長時間來完成它的工作，所以我們不應該在應用的主線程和 UI 線程裡調用這個方法。

[IntentService](http://developer.android.com/reference/android/app/IntentService.html) 類提供了一種結構使一個任務在後臺線程運行。使用這個類，我們可以在不影響 UI 響應速度的情況下處理一個長時間運行的操作。注意到，[AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) 類也可以執行後臺操作，但是它被設計用於短時間運行的操作。在 activity 重新創建時（例如當設備旋轉時），[AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) 不應該保存 UI 的引用。相反，當 activity 重建時，不需要取消 [IntentService](http://developer.android.com/reference/android/app/IntentService.html)。

定義一個繼承 [IntentService](http://developer.android.com/reference/android/app/IntentService.html) 的類 `FetchAddressIntentService`。這個類是地址查找服務。這個 Intent 服務在一個工作線程上異步地處理一個 intent，並在它離開這個工作時自動停止。Intent 外加的數據提供了服務需要的數據，包括一個用於轉換成地址的 [Location](http://developer.android.com/reference/android/location/Location.html) 對象和一個用於處理地址查找結果的 [ResultReceiver](http://developer.android.com/reference/android/os/ResultReceiver.html) 對象。這個服務用一個 [Geocoder](http://developer.android.com/reference/android/location/Geocoder.html) 來獲取位置的地址，並且將結果發送給 [ResultReceiver](http://developer.android.com/reference/android/os/ResultReceiver.html)。

### 在應用的 manifest 文件中定義 Intent 服務

在 manifest 文件中添加一個節點以定義 intent 服務：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.google.android.gms.location.sample.locationaddress" >
    <application
        ...
        <service
            android:name=".FetchAddressIntentService"
            android:exported="false"/>
    </application>
    ...
</manifest>
```

> **Note：**manifest 文件裡的 `<service>` 節點不需要包含一個 intent filter，這是因為我們的主 activity 通過指定 intent 用到的類的名字來創建一個隱式的 intent。

### 創建一個 Geocoder

將一個地理位置傳換成地址的過程叫做*反向地理編碼*。通過實現 `FetchAddressIntentService ` 類的 [onHandleIntent()](http://developer.android.com/reference/android/app/IntentService.html#onHandleIntent(android.content.Intent)) 來執行 intent 服務的主要工作，即反向地理編碼請求。創建一個 [Geocoder](http://developer.android.com/reference/android/location/Geocoder.html) 對象來處理反向地理編碼。

一個區域設置代表一個特定的地理上的或者語言上的區域。Locale 對象用於調整信息的呈現方式，例如數字或者日期，來適應區域設置表示的區域的約定。傳一個 [Locale](http://developer.android.com/reference/java/util/Locale.html) 對象到 [Geocoder](http://developer.android.com/reference/android/location/Geocoder.html) 對象，確保地址結果為用戶的地理區域作出了本地化。

```java
@Override
protected void onHandleIntent(Intent intent) {
    Geocoder geocoder = new Geocoder(this, Locale.getDefault());
    ...
}
```

### 獲取街道地址數據

下一步是從 geocoder 獲取街道地址，處理可能出現的錯誤，和將結果返回給請求地址的 activity。我們需要兩個分別代表成功和失敗的數字常量來報告地理編碼過程的結果。定義一個 `Constants` 類來包含這些值，如下所示：

```java
public final class Constants {
    public static final int SUCCESS_RESULT = 0;
    public static final int FAILURE_RESULT = 1;
    public static final String PACKAGE_NAME =
        "com.google.android.gms.location.sample.locationaddress";
    public static final String RECEIVER = PACKAGE_NAME + ".RECEIVER";
    public static final String RESULT_DATA_KEY = PACKAGE_NAME +
        ".RESULT_DATA_KEY";
    public static final String LOCATION_DATA_EXTRA = PACKAGE_NAME +
        ".LOCATION_DATA_EXTRA";
}
```

為了獲取與地理位置相對應的街道地址，調用 <a href="http://developer.android.com/reference/android/location/Geocoder.html#getFromLocation(double, double, int)">getFromLocation()</a>，傳入位置對象的經度和緯度，以及我們想要返回的地址的最大數量。在這種情況下，我們只需要一個地址。geocoder 返回一個地址數組。如果沒有找到匹配指定位置的地址，那麼它會返回空的列表。如果沒有可用的後臺地理編碼服務，geocoder 會返回 null。

如下面代碼介紹來檢查下述這些錯誤。如果出現錯誤，就將相應的錯誤信息傳給變量 `errorMessage`，從而將錯誤信息發送給發出請求的 activity：

* **No location data provided** - Intent 的附加數據沒有包含反向地理編碼需要用到的 [Location](http://developer.android.com/reference/android/location/Location.html) 對象。
* **Invalid latitude or longitude used** - [Location](http://developer.android.com/reference/android/location/Location.html) 對象提供的緯度和/或者經度無效。
* **No geocoder available** - 由於網絡錯誤或者 IO 異常，導致後臺地理編碼服務不可用。
* **Sorry, no address found** - geocoder 找不到指定緯度/經度對應的地址。

使用 [Address](http://developer.android.com/reference/android/location/Address.html) 類中的 [getAddressLine()](http://developer.android.com/reference/android/location/Address.html#getAddressLine(int)) 方法來獲得地址對象的個別行。然後將這些行加入一個地址 fragment 列表當中。其中，這個地址 fragment 列表準備好返回到發出地址請求的 activity。

為了將結果返回給發出地址請求的 activity，需要調用 `deliverResultToReceiver()` 方法（定義於下面的[把地址返回給請求端]()）。結果由之前提到的成功/失敗數字代碼和一個字符串組成。在反向地理編碼成功的情況下，這個字符串包含著地址。在失敗的情況下，這個字符串包含錯誤的信息。如下所示：

```java
@Override
protected void onHandleIntent(Intent intent) {
    String errorMessage = "";

    // Get the location passed to this service through an extra.
    Location location = intent.getParcelableExtra(
            Constants.LOCATION_DATA_EXTRA);

    ...

    List<Address> addresses = null;

    try {
        addresses = geocoder.getFromLocation(
                location.getLatitude(),
                location.getLongitude(),
                // In this sample, get just a single address.
                1);
    } catch (IOException ioException) {
        // Catch network or other I/O problems.
        errorMessage = getString(R.string.service_not_available);
        Log.e(TAG, errorMessage, ioException);
    } catch (IllegalArgumentException illegalArgumentException) {
        // Catch invalid latitude or longitude values.
        errorMessage = getString(R.string.invalid_lat_long_used);
        Log.e(TAG, errorMessage + ". " +
                "Latitude = " + location.getLatitude() +
                ", Longitude = " +
                location.getLongitude(), illegalArgumentException);
    }

    // Handle case where no address was found.
    if (addresses == null || addresses.size()  == 0) {
        if (errorMessage.isEmpty()) {
            errorMessage = getString(R.string.no_address_found);
            Log.e(TAG, errorMessage);
        }
        deliverResultToReceiver(Constants.FAILURE_RESULT, errorMessage);
    } else {
        Address address = addresses.get(0);
        ArrayList<String> addressFragments = new ArrayList<String>();

        // Fetch the address lines using getAddressLine,
        // join them, and send them to the thread.
        for(int i = 0; i < address.getMaxAddressLineIndex(); i++) {
            addressFragments.add(address.getAddressLine(i));
        }
        Log.i(TAG, getString(R.string.address_found));
        deliverResultToReceiver(Constants.SUCCESS_RESULT,
                TextUtils.join(System.getProperty("line.separator"),
                        addressFragments));
    }
}
```

### 把地址返回給請求端

Intent 服務最後要做的事情是將地址返回給啟動服務的 activity 裡的 [ResultReceiver](http://developer.android.com/reference/android/os/ResultReceiver.html)。這個 [ResultReceiver](http://developer.android.com/reference/android/os/ResultReceiver.html) 類允許我們發送一個帶有結果的數字代碼和一個包含結果數據的消息。這個數字代碼說明了地理編碼請求是成功還是失敗。在反向地理編碼成功的情況下，這個消息包含著地址。在失敗的情況下，這個消息包含一些描述失敗原因的文本。

我們已經可以從 geocoder 取得地址，捕獲到可能出現的錯誤，調用 `deliverResultToReceiver()` 方法。現在我們需要定義 `deliverResultToReceiver()` 方法來將結果代碼和消息包發送給結果接收端。

對於結果代碼，使用已經傳給 `deliverResultToReceiver()` 方法的 `resultCode` 參數的值。對於消息包的結構，連接 `Constants` 類的 `RESULT_DATA_KEY` 常量（定義與[獲取街道地址數據]()）和傳給 `deliverResultToReceiver()` 方法的 `message` 參數的值。如下所示：

```java
public class FetchAddressIntentService extends IntentService {
    protected ResultReceiver mReceiver;
    ...
    private void deliverResultToReceiver(int resultCode, String message) {
        Bundle bundle = new Bundle();
        bundle.putString(Constants.RESULT_DATA_KEY, message);
        mReceiver.send(resultCode, bundle);
    }
}
```

## 啟動 Intent 服務

上節課定義的 intent 服務在後臺運行，同時，該服務負責提取與指定地理位置相對應的地址。當我們啟動服務，Android 框架會實例化並啟動服務（如果該服務沒有運行），並且如果需要的話，創建一個進程。如果服務正在運行，那麼讓它保持運行狀態。因為服務繼承於 [IntentService](http://developer.android.com/reference/android/app/IntentService.html)，所以當所有 intent 都被處理完之後，該服務會自動停止。

在我們應用的主 activity 中啟動服務，並且創建一個 [Intent](http://developer.android.com/reference/android/content/Intent.html) 來把數據傳給服務。我們需要創建一個*顯式的* intent，這是因為我們只想我們的服務響應該 intent。詳細請見 [Intent Types](http://developer.android.com/guide/components/intents-filters.html#Types)。

為了創建一個顯式的 intent，需要為服務指定要用到的類名：`FetchAddressIntentService.class`。在 intent 附加數據中傳入兩個信息：

* 一個用於處理地址查找結果的 [ResultReceiver](http://developer.android.com/reference/android/os/ResultReceiver.html)。
* 一個包含想要轉換成地址的緯度和經度的 [Location](http://developer.android.com/reference/android/location/Location.html) 對象。

下面的代碼介紹瞭如何啟動 intent 服務：

```java
public class MainActivity extends ActionBarActivity implements
        ConnectionCallbacks, OnConnectionFailedListener {

    protected Location mLastLocation;
    private AddressResultReceiver mResultReceiver;
    ...

    protected void startIntentService() {
        Intent intent = new Intent(this, FetchAddressIntentService.class);
        intent.putExtra(Constants.RECEIVER, mResultReceiver);
        intent.putExtra(Constants.LOCATION_DATA_EXTRA, mLastLocation);
        startService(intent);
    }
}
```

當用戶請求查找地理地址時，調用上述的 `startIntentService()` 方法。例如，用戶可能會在我們應用的 UI 上面點擊*提取地址*按鈕。在啟動 intent 服務之前，我們需要檢查是否已經連接到 Google Play services。下面的代碼片段介紹在一個按鈕 handler 中調用 `startIntentService()` 方法。

```java
public void fetchAddressButtonHandler(View view) {
    // Only start the service to fetch the address if GoogleApiClient is
    // connected.
    if (mGoogleApiClient.isConnected() && mLastLocation != null) {
        startIntentService();
    }
    // If GoogleApiClient isn't connected, process the user's request by
    // setting mAddressRequested to true. Later, when GoogleApiClient connects,
    // launch the service to fetch the address. As far as the user is
    // concerned, pressing the Fetch Address button
    // immediately kicks off the process of getting the address.
    mAddressRequested = true;
    updateUIWidgets();
}
```

如果用戶點擊了應用 UI 上面的*提取地址*按鈕，那麼我們必須在 Google Play services 連接穩定之後啟動 intent 服務。下面的代碼片段介紹了調用 Google API Client 提供的 [onConnected()](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.ConnectionCallbacks.html#onConnected(android.os.Bundle)) 回調函數中的 `startIntentService()` 方法。

```java
public class MainActivity extends ActionBarActivity implements
        ConnectionCallbacks, OnConnectionFailedListener {
    ...
    @Override
    public void onConnected(Bundle connectionHint) {
        // Gets the best and most recent location currently available,
        // which may be null in rare cases when a location is not available.
        mLastLocation = LocationServices.FusedLocationApi.getLastLocation(
                mGoogleApiClient);

        if (mLastLocation != null) {
            // Determine whether a Geocoder is available.
            if (!Geocoder.isPresent()) {
                Toast.makeText(this, R.string.no_geocoder_available,
                        Toast.LENGTH_LONG).show();
                return;
            }

            if (mAddressRequested) {
                startIntentService();
            }
        }
    }
}
```

## 獲取地理編碼結果

Intent 服務已經處理完地理編碼請求，並用 [ResultReceiver](http://developer.android.com/reference/android/os/ResultReceiver.html) 將結果返回給發出請求的 activity。在發出請求的 activity 裡，定義一個繼承於 [ResultReceiver](http://developer.android.com/reference/android/os/ResultReceiver.html) 的 `AddressResultReceiver`，用於處理在 `FetchAddressIntentService` 中的響應。

結果包含一個數字代碼（`resultCode`）和一個包含結果數據（`resultData`）的消息。如果反向地理編碼成功的話，`resultData` 會包含地址。如果失敗，`resultData` 包含描述失敗原因的文本。關於錯誤信息更詳細的內容，請見[把地址返回給請求端]()

重寫 <a href="http://developer.android.com/reference/android/os/ResultReceiver.html#onReceiveResult(int, android.os.Bundle)">onReceiveResult()</a> 方法來處理髮送給接收端的結果，如下所示：

```java
public class MainActivity extends ActionBarActivity implements
        ConnectionCallbacks, OnConnectionFailedListener {
    ...
    class AddressResultReceiver extends ResultReceiver {
        public AddressResultReceiver(Handler handler) {
            super(handler);
        }

        @Override
        protected void onReceiveResult(int resultCode, Bundle resultData) {

            // Display the address string
            // or an error message sent from the intent service.
            mAddressOutput = resultData.getString(Constants.RESULT_DATA_KEY);
            displayAddressOutput();

            // Show a toast message if an address was found.
            if (resultCode == Constants.SUCCESS_RESULT) {
                showToast(getString(R.string.address_found));
            }

        }
    }
}
```
