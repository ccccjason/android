# 使用模擬位置進行測試

> 編寫:[penkzhou](https://github.com/penkzhou) - 原文:<http://developer.android.com/training/location/location-testing.html>

當你在測試一個使用Location Services基於地理位置的應用時，你是不需要把你的設備從一個地方移動到另一個地方來產生位置數據的。你可以將Location Services設置成模擬模式。在這個模式裡面，你可以發送模擬位置給Location Services，然後Location Services再將這些數據發送給位置client。在模擬模式裡面，Location Services也可以使用模擬位置對象來觸發地理圍欄。

使用模擬位置有以下幾個優點：

* 模擬位置可以讓你創建特定的模擬數據，而不需要你移動你的設備到特定的地方來獲取接近的數據。

* 因為模擬位置來源於Location Services，它們可以測試你處理地理位置代碼的每一個部分。而且，因為你可以從你的正式版應用之外發送模擬數據，那麼你就不必在發佈你的應用之前禁用或者刪掉測試代碼。

* 因為你不必通過移動設備來產生測試位置，那你就可以使用模擬器來測試應用了。


使用模擬位置最好的方式就是從一個單獨的模擬位置提供應用發送模擬位置數據。這一課就包括了一個位置提供應用，你可以下載下來測試你的軟件。你也可以更改這個應用來滿足你自己的需求。為應用提供測試數據的一些想法也列在[管理測試數據](location-testing.html#TestData)這一塊裡面。

這個課程接下來的部分就是教你如何開啟模擬模式以及如何使用一個位置client來發送模擬數據給Location Services。

> **Note:** 模擬位置對Location Services的活動識別算法沒有影響想要了解更多關於活動識別，請參看課程 [識別用戶的當下活動](activity-recognition.html)。

## 1)開啟模擬模式

一個應用要想在模擬模式下面給Location Services發送模擬位置 ，那麼它必須要設置 [```ACCESS_MOCK_LOCATION```](http://developer.android.com/reference/android/Manifest.permission.html#ACCESS_MOCK_LOCATION)權限。而且，你必須在測試設備上開啟模擬位置選項。要了解如何開啟設備的模擬位置選項，請參看開啟設備的開發者模式。

為了在Location Services裡面開啟模擬模式，你需要先連接一個位置client到Location Services，就像之前的課程 [接收當前位置信息](retrieve-current.html)一樣。接著，調用[LocationClient.setMockMode(true)](http://developer.android.com/reference/com/google/android/gms/location/LocationClient.html#setMockMode(boolean))方法。一旦你調用了這個方法，Location Services就會關掉它內部的位置提供器，然後只轉發你發給它的模擬位置。下面的代碼教你如何調用[LocationClient.setMockMode(true)](http://developer.android.com/reference/com/google/android/gms/location/LocationClient.html#setMockMode(boolean))方法：

```java
// Define a LocationClient object
    public LocationClient mLocationClient;
    ...
    // Connect to Location Services
    mLocationClient.connect();
    ...
    // When the location client is connected, set mock mode
    mLocationClinet.setMockMode(true);
```

一旦這個位置client連接上了Location Services，你必須保持這個連接知道你結束髮送模擬位置為止。一旦你調用[LocationClient.disconnect()](http://developer.android.com/reference/com/google/android/gms/location/LocationClient.html#disconnect())這個方法，Location Services便會開始啟用它的內部位置提供器。在位置client連接的時候調用[LocationClient.setMockMode(false)](http://developer.android.com/reference/com/google/android/gms/location/LocationClient.html#setMockMode(boolean))方法就可以關掉模擬模式了。

## 2)發送模擬位置

一旦你設置好了模擬模式，你就可以創建模擬位置對象了，然後就可以將它們發送給Location Services。接著，Location Services 又會把這些模擬位置發送給連接的位置clients。Location Services 還可以使用模擬位置來控制地理圍欄的觸發。

要創建一個新的模擬位置，你要用你的測試數據創建一個新的位置對象。你還需要將提供者的值設為```flp```，接著Location Services把這些信息放到位置對象裡面，然後發送出去。下面的代碼展示瞭如何創建一個新的模擬位置：

```java
   private static final String PROVIDER = "flp";
    private static final double LAT = 37.377166;
    private static final double LNG = -122.086966;
    private static final float ACCURACY = 3.0f;
    ...
    /*
     * From input arguments, create a single Location with provider set to
     * "flp"
     */
    public Location createLocation(double lat, double lng, float accuracy) {
        // Create a new Location
        Location newLocation = new Location(PROVIDER);
        newLocation.setLatitude(lat);
        newLocation.setLongitude(lng);
        newLocation.setAccuracy(accuracy);
        return newLocation;
    }
    ...
    // Example of creating a new Location from test data
    Location testLocation = createLocation(LAT, LNG, ACCURACY);
```

在模擬模式裡面，你需要使用[LocationClient.setMockLocation()](http://developer.android.com/reference/com/google/android/gms/location/LocationClient.html#setMockLocation(android.location.Location))方法來發送模擬位置給Location Services。 例如：

```java
 mLocationClient.setMockLocation(testLocation);
```

Location Services 將這個模擬位置設為當前位置，接著這個位置會在下一個位置更新來的時候被送出去。如何這個新的模擬位置進入了一個地理圍欄，Location Services 會觸發這個地理圍欄的。

## 3)運行模擬位置提供應用

這一部分包含了這個模擬位置提供應用的總體概覽，然後給你一些使用這個應用測試你自己的代碼的一些指導。

### 3.1)總體概覽

這個模擬位置提供應用從後臺運行的已經啟動的一個服務發送模擬位置對象給Location Services。通過使用一個已經啟動服務，這個應用可以即使在主界面因為系統配置改變被銷燬的前提下保持運行狀態。通過使用 一個後臺線程，這個服務可以執行長時的測試而不會阻塞UI主線程。

這個應用啟動的界面可以讓你控制發送的模擬數據類型。你有以下可選項：

Pause before test
* 這個參數可以設置應用在開始發送測試數據給Location Services之前要等待的秒數。這個間隔可以允許你在測試開始之前從模擬位置提供應用跳轉至當前測試應用。

Send interval
* 這個參數可以設置模擬位置發送週期。你可以參考下面的[測試小技巧](location-testing.html#TestingTips)來了解更多發送週期的設置。

Run once
* 從正常模式轉換至模擬模式，運行完測試數據之後，又轉換回正常模式，接著便終結服務。

Run continuously
* 從正常模式轉換至模擬模式，然後無期限的運行測試數據。 後臺線程和啟動的服務會一直運行下去，即便主界面被銷燬。

Stop test
* 如果處於測試中，那麼這個測試會被終止，否則會發回一個警告信息。啟動的服務會從模擬模式轉回正常模式，然後自己停止自己。這個操作也會停掉後臺線程。


在這些選項之外，這個應用還提供了兩種狀態顯示：

App status
* 顯示這個應用相關的生命週期信息。

Connection status
* 顯示這個連接的位置client相關的狀態信息。
會

在這個啟動的服務運行的時候，它還會發送測試狀態的通知。這些通知可以讓你看到即便應用不在前臺的時候也能知道它的狀態更新。當你點擊這些通知的時候，主界面會回到前臺來。

### 3.2)使用模擬位置提供應用來測試

測試來自模擬位置提供應用的測試模擬位置數據：

1. 在已經安裝好了Google Play Services的設備上安裝模擬位置提供應用。Location Services是Google Play services的一部分。
2. 在設備上，開啟模擬位置選項。要了解如何操作，請參看如何開啟設備開發者模式。
3. 從桌面啟動應用，然後選擇你要設置的選項。
4. 除非你刪掉這個pause interval這個特徵，要不然應用會暫停幾秒鐘，然後開始發生模擬位置數據給Location Services。
5. 運行你要測試的應用。在模擬位置提供應用運行的時候，你測試的應用接收的時模擬位置而不是真實地位置。
6. 你可以在模擬應用測試到一半的時候點擊停止測試將模式從模擬轉換至真實位置。這個操作會強制啟動的服務去停掉模擬模式，然後自己停掉自己。當服務自己停掉自己之後，後臺線程也會被銷燬。

## 4)測試小貼士

下面的教程包含了創建模擬位置數據以及使用模擬位置數據的一些小貼士。

### 4.1)選擇一個發送週期

每一個位置提供者在為有Location Services發送的位置服務時都有自己的更新頻率。例如，GPS最快的頻率也是一秒鐘一次更新，WiFi的更新頻率最快是5秒鐘一次。這些週期時間是真實位置裡面的處理週期，但是你在使用模擬位置的時候你需要設置好這些。例如，你的頻率不能超過一秒一次。如果你在室內測試，這說明你很依賴WiFi，那麼你應該將頻率設為5秒一次。

### 4.2)模擬速度

為了模擬一個真實設備的速度，縮短或者加長兩個連續位置之間的距離。例如，通過每秒改變設備位置88英尺來模擬汽車駕駛，因為這樣算出來的時速是60英里。作為比較，通過每秒改變設備位置1.5英尺來模擬跑步，因為換算成時速就是3英里。

### 4.3)計算位置數據

通過搜索，你可以找到很多計算指定距離的位置經緯度和兩點之間的距離的小程序。事實上，Location類提供了兩個計算位置距離的方法：

[distanceBetween()](http://developer.android.com/reference/android/location/Location.html#distanceBetween(double, double, double, double, float[]))
* 計算兩個已知經緯度的地點之間的距離的靜態方法。

[distanceTo()](http://developer.android.com/reference/android/location/Location.html#distanceTo(android.location.Location))
* 給定一個地點，返回到另一個地點的距離。

### 4.4)地理圍欄測試

當你的測試一個使用地理圍欄探測的應用時，使用反應不同運動模式的測試數據，這些模式包括步行，騎行，駕駛，火車。對於慢的運動模式，可以對位置做較小的更改；相反，對於快的運動模式，可以對位置做較大的更改。

### 4.5)管理測試數據

這一課裡面的模擬位置提供應用以常量的形式包含了測試經緯度，數據精度。你可能想以其他形式來組織數據：

XML
* 將位置數據保存到XML文件裡面。這樣將代碼和數據分離開，你可以很容易更改數據了。

Server download
* 將位置數據保存到服務器上面，然後使用應用下載下來。因為數據和應用已經完全分隔開來，你可以無需重建應用就可以更改數據了。你還可以直接在服務器上面更改數據然後影響你的模擬位置。。

Recorded data
* 除了生成測試數據，寫一個工具應用來記錄你的設備在移動的時候產生的地理位置信息。使用這些記錄作為你的測試數據，或者使用這些數據來引導你開發測試數據。例如，記錄你在步行的時候設備產生的位置信息，然後用它來創建模擬位置，因為這種數據隨著時間有比較合適的改變。
