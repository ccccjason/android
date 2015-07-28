# 使用網絡服務發現

> 編寫:[naizhengtan](https://github.com/naizhengtan) - 原文:<http://developer.android.com/training/connect-devices-wirelessly/nsd.html>

添加網絡服務發現（Network Service Discovery）到我們的 app 中，可以使我們的用戶辨識在局域網內支持我們的 app 所請求的服務的設備。這種技術在點對點應用中能夠提供大量幫助，例如文件共享、聯機遊戲等。Android 的網絡服務發現（NSD）API 大大降低實現上述功能的難度。

本講將簡要介紹如何創建 NSD 應用，使其能夠在本地網絡內廣播自己的名稱和連接信息，並且掃描其它正在做同樣事情的應用信息。最後，將介紹如何連接運行著同樣應用的另一臺設備。

## 註冊 NSD 服務

> **Note:** 這一步驟是選做的。如果我們並不關心在本地網絡上廣播 app 服務，那麼我們可以跳過這一步，直接嘗試[發現網絡中的服務](#discover)。

在局域網內註冊自己服務的第一步是創建 [NsdServiceInfo](http://developer.android.com/reference/android/net/nsd/NsdServiceInfo.html) 對象。此對象包含的信息能夠幫助網絡中的其他設備決定是否要連接到我們所提供的服務。

```java
public void registerService(int port) {
    // Create the NsdServiceInfo object, and populate it.
    NsdServiceInfo serviceInfo  = new NsdServiceInfo();

    // The name is subject to change based on conflicts
    // with other services advertised on the same network.
    serviceInfo.setServiceName("NsdChat");
    serviceInfo.setServiceType("_http._tcp.");
    serviceInfo.setPort(port);
    ....
}
```

這段代碼將服務命名為“NsdChat”。該名稱將對所有局域網絡中使用 NSD 查找本地服務的設備可見。需要注意的是，在網絡內該名稱必須是獨一無二的。Android 系統會自動處理衝突的服務名稱。如果同時有兩個名為“NsdChat”的應用，其中一個會被自動轉換為類似“NsdChat(1)”這樣的名稱。

第二個參數設置了服務類型，即指定應用使用的協議和傳輸層。語法是“\_< protocol >.\_< transportlayer >”。在上面的代碼中，服務使用了TCP協議上的HTTP協議。想要提供打印服務（例如，一臺網絡打印機）的應用應該將服務的類型設置為“_ipp._tcp”。

> **Note:** 互聯網編號分配機構（International Assigned Numbers Authority，簡稱 IANA）提供用於服務發現協議（例如 NSD 和 Bonjour）的官方服務種類列表。我們可以下載該列表瞭解相應的服務名稱和端口號碼。如果我們想起用新的服務種類，應該向 IANA 官方提交申請。

當為我們的服務設置端口號時，應該儘量避免將其硬編碼在代碼中，以防止與其他應用產生衝突。例如，如果我們的應用僅僅使用端口1337，就可能與其他使用1337端口的應用發生衝突。解決方法是，不要硬編碼，使用下一個可用的端口。不必擔心其他應用無法知曉服務的端口號，因為該信息將包含在服務的廣播包中。接收到廣播後，其他應用將從廣播包中得知服務端口號，並通過端口連接到我們的服務上。

如果使用的是 socket，那麼我們可以將端口設置為 0，來初始化 socket 到任意可用的端口。

```java
public void initializeServerSocket() {
    // Initialize a server socket on the next available port.
    mServerSocket = new ServerSocket(0);

    // Store the chosen port.
    mLocalPort =  mServerSocket.getLocalPort();
    ...
}
```

現在，我們已經成功的創建了 [NsdServiceInfo](http://developer.android.com/reference/android/net/nsd/NsdServiceInfo.html) 對象，接下來要做的是實現 [RegistrationListener](http://developer.android.com/reference/android/net/nsd/NsdManager.RegistrationListener.html) 接口。該接口包含了註冊在 Android 系統中的回調函數，作用是通知應用程序服務註冊和註銷的成功或者失敗。

```java
public void initializeRegistrationListener() {
    mRegistrationListener = new NsdManager.RegistrationListener() {

        @Override
        public void onServiceRegistered(NsdServiceInfo NsdServiceInfo) {
            // Save the service name.  Android may have changed it in order to
            // resolve a conflict, so update the name you initially requested
            // with the name Android actually used.
            mServiceName = NsdServiceInfo.getServiceName();
        }

        @Override
        public void onRegistrationFailed(NsdServiceInfo serviceInfo, int errorCode) {
            // Registration failed!  Put debugging code here to determine why.
        }

        @Override
        public void onServiceUnregistered(NsdServiceInfo arg0) {
            // Service has been unregistered.  This only happens when you call
            // NsdManager.unregisterService() and pass in this listener.
        }

        @Override
        public void onUnregistrationFailed(NsdServiceInfo serviceInfo, int errorCode) {
            // Unregistration failed.  Put debugging code here to determine why.
        }
    };
}
```

萬事俱備只欠東風，調用 <a href="http://developer.android.com/reference/android/net/nsd/NsdManager.html#registerService(android.net.nsd.NsdServiceInfo, int, android.net.nsd.NsdManager.RegistrationListener">registerService()</a> 方法，真正註冊服務。

因為該方法是異步的，所以在服務註冊之後的操作都需要在 <a href="http://developer.android.com/reference/android/net/nsd/NsdManager.RegistrationListener.html#onServiceRegistered(android.net.nsd.NsdServiceInfo)">onServiceRegistered()</a> 方法中進行。

```java
public void registerService(int port) {
    NsdServiceInfo serviceInfo  = new NsdServiceInfo();
    serviceInfo.setServiceName("NsdChat");
    serviceInfo.setServiceType("_http._tcp.");
    serviceInfo.setPort(port);

    mNsdManager = Context.getSystemService(Context.NSD_SERVICE);

    mNsdManager.registerService(
            serviceInfo, NsdManager.PROTOCOL_DNS_SD, mRegistrationListener);
}
```

<a name="discover"></a>
## 發現網絡中的服務

網絡充斥著我們的生活，從網絡打印機到網絡攝像頭，再到聯網井字棋。網絡服務發現是能讓我們的應用融入這一切功能的關鍵。我們的應用需要偵聽網絡內服務的廣播，發現可用的服務，過濾無效的信息。

與註冊網絡服務類似，服務發現需要兩步驟：用相應的回調函數設置發現監聽器（Discover Listener），以及調用 <a href="http://developer.android.com/reference/android/net/nsd/NsdManager.html#discoverServices(java.lang.String, int, android.net.nsd.NsdManager.DiscoveryListener)">discoverServices()</a> 這個異步API。

首先，實例化一個實現 [NsdManager.DiscoveryListener](http://developer.android.com/reference/android/net/nsd/NsdManager.DiscoveryListener.html) 接口的匿名類。下列代碼是一個簡單的範例：

```java
public void initializeDiscoveryListener() {

    // Instantiate a new DiscoveryListener
    mDiscoveryListener = new NsdManager.DiscoveryListener() {

        //  Called as soon as service discovery begins.
        @Override
        public void onDiscoveryStarted(String regType) {
            Log.d(TAG, "Service discovery started");
        }

        @Override
        public void onServiceFound(NsdServiceInfo service) {
            // A service was found!  Do something with it.
            Log.d(TAG, "Service discovery success" + service);
            if (!service.getServiceType().equals(SERVICE_TYPE)) {
                // Service type is the string containing the protocol and
                // transport layer for this service.
                Log.d(TAG, "Unknown Service Type: " + service.getServiceType());
            } else if (service.getServiceName().equals(mServiceName)) {
                // The name of the service tells the user what they'd be
                // connecting to. It could be "Bob's Chat App".
                Log.d(TAG, "Same machine: " + mServiceName);
            } else if (service.getServiceName().contains("NsdChat")){
                mNsdManager.resolveService(service, mResolveListener);
            }
        }

        @Override
        public void onServiceLost(NsdServiceInfo service) {
            // When the network service is no longer available.
            // Internal bookkeeping code goes here.
            Log.e(TAG, "service lost" + service);
        }

        @Override
        public void onDiscoveryStopped(String serviceType) {
            Log.i(TAG, "Discovery stopped: " + serviceType);
        }

        @Override
        public void onStartDiscoveryFailed(String serviceType, int errorCode) {
            Log.e(TAG, "Discovery failed: Error code:" + errorCode);
            mNsdManager.stopServiceDiscovery(this);
        }

        @Override
        public void onStopDiscoveryFailed(String serviceType, int errorCode) {
            Log.e(TAG, "Discovery failed: Error code:" + errorCode);
            mNsdManager.stopServiceDiscovery(this);
        }
    };
}
```

NSD API 通過使用該接口中的方法通知用戶程序發現何時開始、何時失敗以及何時找到可用服務和何時服務丟失（丟失意味著“不再可用”）。在上述代碼中，當發現了可用的服務時，程序做了幾次檢查。

1. 比較找到服務的名稱與本地服務的名稱，判斷設備是否獲得自己的（合法的）廣播。
2. 檢查服務的類型，確認這個類型我們的應用是否可以接入。
3. 檢查服務的名稱，確認是否接入了正確的應用。

我們並不需要每次都檢查服務名稱，僅當我們想要接入特定的應用時需要檢查。例如，應用只想與運行在其他設備上的相同應用通信。然而，如果應用僅僅想接入到一臺網絡打印機，那麼看到服務類型是“_ipp._tcp”的服務就足夠了。

當配置好監聽器後，調用 <a href="http://developer.android.com/reference/android/net/nsd/NsdManager.html#discoverServices(java.lang.String, int, android.net.nsd.NsdManager.DiscoveryListener)">discoverService()</a> 函數，其參數包括試圖發現的服務種類、發現使用的協議、以及上一步創建的監聽器。

```java
mNsdManager.discoverServices(
        SERVICE_TYPE, NsdManager.PROTOCOL_DNS_SD, mDiscoveryListener);
```

## 連接到網絡上的服務

當我們的應用發現了網上可接入的服務，首先需要調用 <a href="http://developer.android.com/reference/android/net/nsd/NsdManager.html#resolveService(android.net.nsd.NsdServiceInfo, android.net.nsd.NsdManager.ResolveListener)">resolveService()</a> 方法，以確定服務的連接信息。實現 [NsdManager.ResolveListener](http://developer.android.com/reference/android/net/nsd/NsdManager.ResolveListener.html) 對象並將其傳入 `resolveService()` 方法，並使用這個 `NsdManager.ResolveListener` 對象獲得包含連接信息的 [NsdSerServiceInfo](http://developer.android.com/reference/android/net/nsd/NsdServiceInfo.html)。

```java
public void initializeResolveListener() {
    mResolveListener = new NsdManager.ResolveListener() {

        @Override
        public void onResolveFailed(NsdServiceInfo serviceInfo, int errorCode) {
            // Called when the resolve fails.  Use the error code to debug.
            Log.e(TAG, "Resolve failed" + errorCode);
        }

        @Override
        public void onServiceResolved(NsdServiceInfo serviceInfo) {
            Log.e(TAG, "Resolve Succeeded. " + serviceInfo);

            if (serviceInfo.getServiceName().equals(mServiceName)) {
                Log.d(TAG, "Same IP.");
                return;
            }
            mService = serviceInfo;
            int port = mService.getPort();
            InetAddress host = mService.getHost();
        }
    };
}
```

當服務解析完成後，我們將獲得服務的詳細資料，包括其 IP 地址和端口號。此時，我們就可以創建自己網絡連接與服務進行通訊。

## 當程序退出時註銷服務

在應用的生命週期中正確的開啟和關閉 NSD 服務是十分關鍵的。在程序退出時註銷服務可以防止其他程序因為不知道服務退出而反覆嘗試連接的行為。另外，服務發現是一種開銷很大的操作，應該隨著父 Activity 的暫停而停止，當用戶返回該界面時再開啟。因此，開發者應該重寫 Activity 的生命週期函數，並添加按照需要開啟和停止服務廣播和發現的代碼。

```java
//In your application's Activity

    @Override
    protected void onPause() {
        if (mNsdHelper != null) {
            mNsdHelper.tearDown();
        }
        super.onPause();
    }

    @Override
    protected void onResume() {
        super.onResume();
        if (mNsdHelper != null) {
            mNsdHelper.registerService(mConnection.getLocalPort());
            mNsdHelper.discoverServices();
        }
    }

    @Override
    protected void onDestroy() {
        mNsdHelper.tearDown();
        mConnection.tearDown();
        super.onDestroy();
    }

    // NsdHelper's tearDown method
        public void tearDown() {
        mNsdManager.unregisterService(mRegistrationListener);
        mNsdManager.stopServiceDiscovery(mDiscoveryListener);
    }

```
