# 使用 WiFi P2P 服務發現

> 編寫:[naizhengtan](https://github.com/naizhengtan) - 原文:<http://developer.android.com/training/connect-devices-wirelessly/nsd-wifi-direct.html>

在本章第一節“[使用網絡服務發現](nsd.html)”中介紹瞭如何在局域網中發現已連接到網絡的服務。然而，即使在不接入網絡的情況下，Wi-Fi P2P 服務發現也可以使我們的應用直接發現附近的設備。我們也可以向外公佈自己設備上的服務。這些能力可以在沒有局域網或者網絡熱點的情況下，在應用間進行通信。

雖然本節所述的 API 與第一節 NSD（Network Service Discovery）的 API 相似，但是具體的實現代碼卻截然不同。本節將講述如何通 過Wi-Fi P2P 技術發現其它設備中可用的服務。本節假設讀者已經對 Wi-Fi P2P 的 API 有一定了解。

## 配置 Manifest

使用 Wi-Fi P2P 技術，需要添加 [CHANGE_WIFI_STATE](http://developer.android.com/reference/android/Manifest.permission.html#CHANGE_WIFI_STATE)、[ACCESS_WIFI_STATE](http://developer.android.com/reference/android/Manifest.permission.html#ACCESS_WIFI_STATE) 以及 [INTERNET](http://developer.android.com/reference/android/Manifest.permission.html#INTERNET) 三種權限到應用的 manifest 文件。雖然 Wi-Fi P2P 技術不需要訪問互聯網，但是它會使用 Java 中的標準 socket，而使用 socket 需要具有 INTERNET 權限，這也是 Wi-Fi P2P 技術需要申請該權限的原因。

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.android.nsdchat"
    ...

    <uses-permission
        android:required="true"
        android:name="android.permission.ACCESS_WIFI_STATE"/>
    <uses-permission
        android:required="true"
        android:name="android.permission.CHANGE_WIFI_STATE"/>
    <uses-permission
        android:required="true"
        android:name="android.permission.INTERNET"/>
    ...
```

## 添加本地服務

如果我們想提供一個本地服務，就需要在服務發現框架中註冊該服務。當本地服務被成功註冊，系統將自動回覆所有來自附近的服務發現請求。

三步創建本地服務：

1. 新建 [WifiP2pServiceInfo](http://developer.android.com/reference/android/net/wifi/p2p/nsd/WifiP2pServiceInfo.html) 對象
2. 加入相應服務的詳細信息
3. 調用 <a href="http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#addLocalService(android.net.wifi.p2p.WifiP2pManager.Channel, android.net.wifi.p2p.nsd.WifiP2pServiceInfo, android.net.wifi.p2p.WifiP2pManager.ActionListener)">addLocalService()</a> 為服務發現註冊本地服務

```java
private void startRegistration() {
        //  Create a string map containing information about your service.
        Map record = new HashMap();
        record.put("listenport", String.valueOf(SERVER_PORT));
        record.put("buddyname", "John Doe" + (int) (Math.random() * 1000));
        record.put("available", "visible");

        // Service information.  Pass it an instance name, service type
        // _protocol._transportlayer , and the map containing
        // information other devices will want once they connect to this one.
        WifiP2pDnsSdServiceInfo serviceInfo =
                WifiP2pDnsSdServiceInfo.newInstance("_test", "_presence._tcp", record);

        // Add the local service, sending the service info, network channel,
        // and listener that will be used to indicate success or failure of
        // the request.
        mManager.addLocalService(channel, serviceInfo, new ActionListener() {
            @Override
            public void onSuccess() {
                // Command successful! Code isn't necessarily needed here,
                // Unless you want to update the UI or add logging statements.
            }

            @Override
            public void onFailure(int arg0) {
                // Command failed.  Check for P2P_UNSUPPORTED, ERROR, or BUSY
            }
        });
    }
```

## 發現附近的服務

Android 使用回調函數通知應用程序附近可用的服務，因此首先要做的是設置這些回調函數。新建一個 [WifiP2pManager.DnsSdTxtRecordListener](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.DnsSdTxtRecordListener.html) 實例監聽實時收到的記錄（record）。這些記錄可以是來自其他設備的廣播。當收到記錄時，將其中的設備地址和其他相關信息拷貝到當前方法之外的外部數據結構中，供之後使用。下面的例子假設這條記錄包含一個帶有用戶身份的“buddyname”域（field）。

```java
final HashMap<String, String> buddies = new HashMap<String, String>();
...
private void discoverService() {
    DnsSdTxtRecordListener txtListener = new DnsSdTxtRecordListener() {
        @Override
        /* Callback includes:
         * fullDomain: full domain name: e.g "printer._ipp._tcp.local."
         * record: TXT record dta as a map of key/value pairs.
         * device: The device running the advertised service.
         */

        public void onDnsSdTxtRecordAvailable(
                String fullDomain, Map record, WifiP2pDevice device) {
                Log.d(TAG, "DnsSdTxtRecord available -" + record.toString());
                buddies.put(device.deviceAddress, record.get("buddyname"));
            }
        };
    ...
}
```

接下來創建 [WifiP2pManager.DnsSdServiceResponseListener](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.DnsSdServiceResponseListener.html) 對象，用以獲取服務的信息。這個對象將接收服務的實際描述以及連接信息。上一段代碼構建了一個包含設備地址和“buddyname”鍵值對的 [Map](http://developer.android.com/reference/java/util/Map.html) 對象。[WifiP2pManager.DnsSdServiceResponseListener](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.DnsSdServiceResponseListener.html) 對象使用這些配對信息將 DNS 記錄和對應的服務信息對應起來。當上述兩個 listener 構建完成後，調用 <a href="http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#setDnsSdResponseListeners(android.net.wifi.p2p.WifiP2pManager.Channel, android.net.wifi.p2p.WifiP2pManager.DnsSdServiceResponseListener, android.net.wifi.p2p.WifiP2pManager.DnsSdTxtRecordListener)">setDnsSdResponseListeners()</a> 將他們加入到 [WifiP2pManager](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html)。

```java
private void discoverService() {
...

    DnsSdServiceResponseListener servListener = new DnsSdServiceResponseListener() {
        @Override
        public void onDnsSdServiceAvailable(String instanceName, String registrationType,
                WifiP2pDevice resourceType) {

                // Update the device name with the human-friendly version from
                // the DnsTxtRecord, assuming one arrived.
                resourceType.deviceName = buddies
                        .containsKey(resourceType.deviceAddress) ? buddies
                        .get(resourceType.deviceAddress) : resourceType.deviceName;

                // Add to the custom adapter defined specifically for showing
                // wifi devices.
                WiFiDirectServicesList fragment = (WiFiDirectServicesList) getFragmentManager()
                        .findFragmentById(R.id.frag_peerlist);
                WiFiDevicesAdapter adapter = ((WiFiDevicesAdapter) fragment
                        .getListAdapter());

                adapter.add(resourceType);
                adapter.notifyDataSetChanged();
                Log.d(TAG, "onBonjourServiceAvailable " + instanceName);
        }
    };

    mManager.setDnsSdResponseListeners(channel, servListener, txtListener);
    ...
}
```

現在調用 <a href="http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#addServiceRequest(android.net.wifi.p2p.WifiP2pManager.Channel, android.net.wifi.p2p.nsd.WifiP2pServiceRequest, android.net.wifi.p2p.WifiP2pManager.ActionListener)">addServiceRequest()</a> 創建服務請求。這個方法也需要一個 Listener 報告請求成功與失敗。

```java
        serviceRequest = WifiP2pDnsSdServiceRequest.newInstance();
        mManager.addServiceRequest(channel,
                serviceRequest,
                new ActionListener() {
                    @Override
                    public void onSuccess() {
                        // Success!
                    }

                    @Override
                    public void onFailure(int code) {
                        // Command failed.  Check for P2P_UNSUPPORTED, ERROR, or BUSY
                    }
                });
```

最後調用 <a href="http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#discoverServices(android.net.wifi.p2p.WifiP2pManager.Channel, android.net.wifi.p2p.WifiP2pManager.ActionListener)">discoverServices()</a>。

```java
        mManager.discoverServices(channel, new ActionListener() {

            @Override
            public void onSuccess() {
                // Success!
            }

            @Override
            public void onFailure(int code) {
                // Command failed.  Check for P2P_UNSUPPORTED, ERROR, or BUSY
                if (code == WifiP2pManager.P2P_UNSUPPORTED) {
                    Log.d(TAG, "P2P isn't supported on this device.");
                else if(...)
                    ...
            }
        });
```

如果所有部分都配置正確，我們應該就能看到正確的結果了！如果遇到了問題，可以查看 [WifiP2pManager.ActionListener](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.ActionListener.html) 中的回調函數。它們能夠指示操作是否成功。我們可以將 debug 的代碼放置在 <a href="http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.ActionListener.html#onFailure(int)">onFailure()</a> 中來診斷問題。其中的一些錯誤碼（Error Code）也許能為我們帶來不小啟發。下面是一些常見的錯誤：

[P2P_UNSUPPORTED](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#P2P_UNSUPPORTED)

　　當前的設備不支持 Wi-Fi P2P

[BUSY](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#BUSY)

　　系統忙，無法處理當前請求

[ERROR](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#ERROR)

　　內部錯誤導致操作失敗
