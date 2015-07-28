# 使用 WiFi 建立 P2P 連接

> 編寫:[naizhengtan](https://github.com/naizhengtan) - 原文:<http://developer.android.com/training/connect-devices-wirelessly/wifi-direct.html>

Wi-Fi 點對點（P2P）API 允許應用程序在無需連接到網絡和熱點的情況下連接到附近的設備。（Android Wi-Fi P2P 使用 [Wi-Fi Direct™](http://www.wi-fi.org/discover-and-learn/wi-fi-direct) 驗證程序進行編譯）。Wi-Fi P2P 技術使得應用程序可以快速發現附近的設備並與之交互。相比於藍牙技術，Wi-Fi P2P 的優勢是具有較大的連接範圍。

本節主要內容是使用 Wi-Fi P2P 技術發現並連接到附近的設備。

## 配置應用權限

使用 Wi-Fi P2P 技術，需要添加 [CHANGE_WIFI_STATE](http://developer.android.com/reference/android/Manifest.permission.html#CHANGE_WIFI_STATE)，[ACCESS_WIFI_STATE](http://developer.android.com/reference/android/Manifest.permission.html#ACCESS_WIFI_STATE) 以及 [INTERNET](http://developer.android.com/reference/android/Manifest.permission.html#INTERNET) 三種權限到應用的 manifest 文件。Wi-Fi P2P 技術雖然不需要訪問互聯網，但是它會使用標準的 Java socket（需要 [INTERNET](http://developer.android.com/reference/android/Manifest.permission.html#INTERNET) 權限）。下面是使用 Wi-Fi P2P 技術需要申請的權限。

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

## 設置廣播接收器（BroadCast Receiver）和 P2P 管理器

使用 Wi-Fi P2P 的時候，需要偵聽當某個事件出現時發出的broadcast intent。在應用中，實例化一個 [IntentFilter](http://developer.android.com/reference/android/content/IntentFilter.html)，並將其設置為偵聽下列事件：

[WIFI_P2P_STATE_CHANGED_ACTION](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#WIFI_P2P_STATE_CHANGED_ACTION)

　　指示　Wi-Fi P2P　是否開啟

[WIFI_P2P_PEERS_CHANGED_ACTION](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#WIFI_P2P_PEERS_CHANGED_ACTION)

　　代表對等節點（peer）列表發生了變化

[WIFI_P2P_CONNECTION_CHANGED_ACTION](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#WIFI_P2P_CONNECTION_CHANGED_ACTION)

　　表明Wi-Fi P2P的連接狀態發生了改變

[WIFI_P2P_THIS_DEVICE_CHANGED_ACTION](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#WIFI_P2P_THIS_DEVICE_CHANGED_ACTION)

　　指示設備的詳細配置發生了變化

```java
private final IntentFilter intentFilter = new IntentFilter();
...
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.main);

    //  Indicates a change in the Wi-Fi P2P status.
    intentFilter.addAction(WifiP2pManager.WIFI_P2P_STATE_CHANGED_ACTION);

    // Indicates a change in the list of available peers.
    intentFilter.addAction(WifiP2pManager.WIFI_P2P_PEERS_CHANGED_ACTION);

    // Indicates the state of Wi-Fi P2P connectivity has changed.
    intentFilter.addAction(WifiP2pManager.WIFI_P2P_CONNECTION_CHANGED_ACTION);

    // Indicates this device's details have changed.
    intentFilter.addAction(WifiP2pManager.WIFI_P2P_THIS_DEVICE_CHANGED_ACTION);

    ...
}
```

在　<a href="http://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle)">onCreate()</a>　方法的最後，需要獲得　[WifiPpManager](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html)　的實例，並調用它的　<a href="http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#initialize(android.content.Context, android.os.Looper, android.net.wifi.p2p.WifiP2pManager.ChannelListener)">initialize()</a> 方法。該方法將返回 [WifiP2pManager.Channel](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.Channel.html) 對象。
我們的應用將在後面使用該對象連接 Wi-Fi P2P 框架。

```java
@Override

Channel mChannel;

public void onCreate(Bundle savedInstanceState) {
    ....
    mManager = (WifiP2pManager) getSystemService(Context.WIFI_P2P_SERVICE);
    mChannel = mManager.initialize(this, getMainLooper(), null);
}
```

接下來，創建一個新的 [BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html) 類偵聽系統中 Wi-Fi P2P 狀態的變化。在 <a href="http://developer.android.com/reference/android/content/BroadcastReceiver.html#onReceive(android.content.Context, android.content.Intent)">onReceive()</a> 方法中，加入對上述四種不同 P2P 狀態變化的處理。

```java
    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        if (WifiP2pManager.WIFI_P2P_STATE_CHANGED_ACTION.equals(action)) {
            // Determine if Wifi P2P mode is enabled or not, alert
            // the Activity.
            int state = intent.getIntExtra(WifiP2pManager.EXTRA_WIFI_STATE, -1);
            if (state == WifiP2pManager.WIFI_P2P_STATE_ENABLED) {
                activity.setIsWifiP2pEnabled(true);
            } else {
                activity.setIsWifiP2pEnabled(false);
            }
        } else if (WifiP2pManager.WIFI_P2P_PEERS_CHANGED_ACTION.equals(action)) {

            // The peer list has changed!  We should probably do something about
            // that.

        } else if (WifiP2pManager.WIFI_P2P_CONNECTION_CHANGED_ACTION.equals(action)) {

            // Connection state changed!  We should probably do something about
            // that.

        } else if (WifiP2pManager.WIFI_P2P_THIS_DEVICE_CHANGED_ACTION.equals(action)) {
            DeviceListFragment fragment = (DeviceListFragment) activity.getFragmentManager()
                    .findFragmentById(R.id.frag_list);
            fragment.updateThisDevice((WifiP2pDevice) intent.getParcelableExtra(
                    WifiP2pManager.EXTRA_WIFI_P2P_DEVICE));

        }
    }
```

最後，在主 activity 開啟時，加入註冊 intent filter 和 broadcast receiver 的代碼，並在 activity 暫停或關閉時，註銷它們。上述做法最好放在 onResume() 和 onPause() 方法中。

```java
    /** register the BroadcastReceiver with the intent values to be matched */
    @Override
    public void onResume() {
        super.onResume();
        receiver = new WiFiDirectBroadcastReceiver(mManager, mChannel, this);
        registerReceiver(receiver, intentFilter);
    }

    @Override
    public void onPause() {
        super.onPause();
        unregisterReceiver(receiver);
    }
```

## 初始化對等節點發現（Peer Discovery）

調用 <a href="http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#discoverPeers(android.net.wifi.p2p.WifiP2pManager.Channel, android.net.wifi.p2p.WifiP2pManager.ActionListener)">discoverPeers()</a> 開始搜尋附近帶有 Wi-Fi P2P 的設備。該方法需要以下參數：

- 上節中調用 WifiP2pManager 的 initialize() 函數獲得的 [WifiP2pManager.Channel](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.Channel.html) 對象
- 一個對 [WifiP2pManager.ActionListener](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.ActionListener.html) 接口的實現，包括了當系統成功和失敗發現所調用的方法。

```java
mManager.discoverPeers(mChannel, new WifiP2pManager.ActionListener() {

        @Override
        public void onSuccess() {
            // Code for when the discovery initiation is successful goes here.
            // No services have actually been discovered yet, so this method
            // can often be left blank.  Code for peer discovery goes in the
            // onReceive method, detailed below.
        }

        @Override
        public void onFailure(int reasonCode) {
            // Code for when the discovery initiation fails goes here.
            // Alert the user that something went wrong.
        }
});
```

需要注意的是，這僅僅表示對Peer發現（Peer Discovery）完成初始化。<a href="http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#discoverPeers(android.net.wifi.p2p.WifiP2pManager.Channel, android.net.wifi.p2p.WifiP2pManager.ActionListener)">discoverPeers()</a> 方法開啟了發現過程並且立即返回。系統會通過調用 WifiP2pManager.ActionListener 中的方法通知應用對等節點發現過程初始化是否正確。同時，對等節點發現過程本身仍然繼續運行，直到一條連接或者一個 P2P 小組建立。

## 獲取對等節點列表

在完成對等節點發現過程的初始化後，我們需要進一步獲取附近的對等節點列表。第一步是實現 [WifiP2pManager.PeerListListener](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.PeerListListener.html) 接口。該接口提供了 Wi-Fi P2P 框架發現的對等節點信息。下列代碼實現了相應功能：

```java
 private List peers = new ArrayList();
    ...

    private PeerListListener peerListListener = new PeerListListener() {
        @Override
        public void onPeersAvailable(WifiP2pDeviceList peerList) {

            // Out with the old, in with the new.
            peers.clear();
            peers.addAll(peerList.getDeviceList());

            // If an AdapterView is backed by this data, notify it
            // of the change.  For instance, if you have a ListView of available
            // peers, trigger an update.
            ((WiFiPeerListAdapter) getListAdapter()).notifyDataSetChanged();
            if (peers.size() == 0) {
                Log.d(WiFiDirectActivity.TAG, "No devices found");
                return;
            }
        }
    }
```

接下來，完善 Broadcast Receiver 的 onReceiver() 方法。
當收到 [WIFI_P2P_PEERS_CHANGED_ACTION](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#WIFI_P2P_PEERS_CHANGED_ACTION) 事件時，
調用 <a href="http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#requestPeers(android.net.wifi.p2p.WifiP2pManager.Channel, android.net.wifi.p2p.WifiP2pManager.PeerListListener)">requestPeer()</a> 方法獲取對等節點列表。我們需要將 WifiP2pManager.PeerListListener 傳遞給 receiver。一種方法是在 broadcast receiver 的構造函數中，將對象作為參數傳入。

```java
public void onReceive(Context context, Intent intent) {
    ...
    else if (WifiP2pManager.WIFI_P2P_PEERS_CHANGED_ACTION.equals(action)) {

        // Request available peers from the wifi p2p manager. This is an
        // asynchronous call and the calling activity is notified with a
        // callback on PeerListListener.onPeersAvailable()
        if (mManager != null) {
            mManager.requestPeers(mChannel, peerListListener);
        }
        Log.d(WiFiDirectActivity.TAG, "P2P peers changed");
    }...
}
```

現在，一個帶有 [WIFI_P2P_PEERS_CHANGED_ACTION](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#WIFI_P2P_PEERS_CHANGED_ACTION) action 的 intent 將觸發應用對 Peer 列表的更新。

## 連接一個對等節點

為了連接到一個對等節點，我們需要創建一個新的 [WifiP2pConfig](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pConfig.html) 對象，並將要連接的設備信息從表示我們想要連接設備的 [WifiP2pDevice](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pDevice.html) 拷貝到其中。然後調用 <a href="http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#connect(android.net.wifi.p2p.WifiP2pManager.Channel, android.net.wifi.p2p.WifiP2pConfig, android.net.wifi.p2p.WifiP2pManager.ActionListener)">connect()</a> 方法。

```java
    @Override
    public void connect() {
        // Picking the first device found on the network.
        WifiP2pDevice device = peers.get(0);

        WifiP2pConfig config = new WifiP2pConfig();
        config.deviceAddress = device.deviceAddress;
        config.wps.setup = WpsInfo.PBC;

        mManager.connect(mChannel, config, new ActionListener() {

            @Override
            public void onSuccess() {
                // WiFiDirectBroadcastReceiver will notify us. Ignore for now.
            }

            @Override
            public void onFailure(int reason) {
                Toast.makeText(WiFiDirectActivity.this, "Connect failed. Retry.",
                        Toast.LENGTH_SHORT).show();
            }
        });
    }
```


在本段代碼中的 [WifiP2pManager.ActionListener](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.ActionListener.html) 實現僅能通知我們初始化的成功或失敗。想要監聽連接狀態的變化，需要實現 [WifiP2pManager.ConnectionInfoListener](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.ConnectionInfoListener.html) 接口。接口中的 <a href="http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.ConnectionInfoListener.html#onConnectionInfoAvailable(android.net.wifi.p2p.WifiP2pInfo)">onConnectionInfoAvailable()</a> 回調函數會在連接狀態發生改變時通知應用程序。當有多個設備同時試圖連接到一臺設備時（例如多人遊戲或者聊天群），這一臺設備將被指定為“群主”（group owner）。

```java
    @Override
    public void onConnectionInfoAvailable(final WifiP2pInfo info) {

        // InetAddress from WifiP2pInfo struct.
        InetAddress groupOwnerAddress = info.groupOwnerAddress.getHostAddress());

        // After the group negotiation, we can determine the group owner.
        if (info.groupFormed && info.isGroupOwner) {
            // Do whatever tasks are specific to the group owner.
            // One common case is creating a server thread and accepting
            // incoming connections.
        } else if (info.groupFormed) {
            // The other device acts as the client. In this case,
            // you'll want to create a client thread that connects to the group
            // owner.
        }
    }
```

此時，回頭繼續完善 broadcast receiver 的 `onReceive()` 方法，並修改對 [WIFI_P2P_CONNECTION_CHANGED_ACTION]() intent 的監聽部分的代碼。當接收到該 intent 時，調用 [requestConnectionInfo()](http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#WIFI_P2P_CONNECTION_CHANGED_ACTION) 方法。此方法為異步，所以結果將會被我們提供的 <a href="http://developer.android.com/reference/android/net/wifi/p2p/WifiP2pManager.html#requestConnectionInfo(android.net.wifi.p2p.WifiP2pManager.Channel, android.net.wifi.p2p.WifiP2pManager.ConnectionInfoListener)">WifiP2pManager.ConnectionInfoListener</a> 所獲取。

```java
        ...
        } else if (WifiP2pManager.WIFI_P2P_CONNECTION_CHANGED_ACTION.equals(action)) {

            if (mManager == null) {
                return;
            }

            NetworkInfo networkInfo = (NetworkInfo) intent
                    .getParcelableExtra(WifiP2pManager.EXTRA_NETWORK_INFO);

            if (networkInfo.isConnected()) {

                // We are connected with the other device, request connection
                // info to find group owner IP

                mManager.requestConnectionInfo(mChannel, connectionListener);
            }
            ...
```
