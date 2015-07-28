# 判斷並監測網絡連接狀態

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/monitoring-device-state/connectivity-monitoring.html>

重複鬧鐘和後臺服務最常見的功能之一，是用來從網絡上獲取應用更新，存儲數據或者執行大文件的下載。但是如果沒有獲得網絡連接，或者連接的速度太慢以至於下載無法完成，那麼就沒有必要喚醒設備並執行那些更新等操作了。

我們可以使用[ConnectivityManager](http://developer.android.com/reference/android/net/ConnectivityManager.html)來檢查設備是否連接到網絡，以及網絡的類型（譯註：通過網絡的連接狀況改變，相應的改變app的行為，減少無謂的操作，從而延長設備的續航能力）。

## 判斷當前是否有網絡連接
如果沒有網絡連接，那麼就沒有必要做那些需要聯網的事情。下面的代碼片段展示瞭如何通過[ConnectivityManager](http://developer.android.com/reference/android/net/ConnectivityManager.html)檢查當前活動的網絡類型，並確定它是否可以連接到互聯網：

```java
ConnectivityManager cm =
        (ConnectivityManager)context.getSystemService(Context.CONNECTIVITY_SERVICE);
 
NetworkInfo activeNetwork = cm.getActiveNetworkInfo();
boolean isConnected = activeNetwork != null &&
                      activeNetwork.isConnectedOrConnecting();
```

## 判斷連接網絡的類型

我們還可以獲取到當前的網絡連接類型。

設備通常可以有移動網絡，WiMax，Wi-Fi與以太網連接等類型。通過查詢當前活動的網絡類型，可以根據網絡的帶寬對更新頻率進行調整：

```java
boolean isWiFi = activeNetwork.getType() == ConnectivityManager.TYPE_WIFI;
```

移動網絡的使用費會比Wi-Fi更高，所以多數情況下，如果設備正在使用移動網絡，我們應該減少應用的更新頻率；同樣地，還應該臨時地掛起一些文件下載任務直到有Wi-Fi連接時再繼續下載。

如果已經關閉了更新操作，那麼需要監聽網絡連接的變化，這樣就可以在建立了互聯網訪問之後，重新恢復它們。

## 監聽網絡連接的變化

當網絡連接發生改變時，[ConnectivityManager](http://developer.android.com/reference/android/net/ConnectivityManager.html)會廣播[CONNECTIVITY_ACTION](http://developer.android.com/reference/android/net/ConnectivityManager.html#CONNECTIVITY_ACTION)（`android.net.conn.CONNECTIVITY_CHANGE`）的Action消息。
我們可以在Manifest文件裡面註冊一個BroadcastReceiver，來監聽這些變化，並適當地恢復（或掛起）你的後臺更新:

```xml
<action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
```

設備的網絡變化可能會比較頻繁，因此每當你在移動網絡與Wi-Fi之間切換的時候，這一廣播就會被觸發。因此，我們可以僅在之前的更新或者下載任務被掛起的時候去監聽這一廣播（用來恢復那些任務）。通常我們可以在開始更新前檢查一下網絡連接，如果當前沒有連接到互聯網，那麼就將更新任務掛起，直到連接恢復。

上述方法會涉及到Broadcast Receiver開啟狀態的切換，這一內容會在下一節課中展開。
