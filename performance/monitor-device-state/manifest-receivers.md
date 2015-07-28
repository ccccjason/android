# 按需操控BroadcastReceiver

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/monitoring-device-state/manifest-receivers.html>

監測設備狀態變化最簡單的方法，是為你所要監聽的每一個狀態創建一個[BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html)，並在Manifest文件中註冊它們。之後就可以在每一個BroadcastReceiver中，根據當前設備的狀態調整一些計劃任務。

上述方法的副作用是：一旦你的接收器收到了廣播，應用就會喚醒設備。喚醒的頻率可能會遠高於需要的頻率

更好的方法是在程序運行時開啟或者關閉BroadcastReceiver。這樣的話，你就可以讓這些接收器僅在需要的時候被激活。

## 切換是否開啟接收器以提高效率

我們可以使用[PackageManager](http://developer.android.com/reference/android/content/pm/PackageManager.html)來切換任何一個在Mainfest裡面定義好的組件的開啟狀態。通過下面的方法可以開啟或者關閉任何一個BroadcastReceiver：

```java
ComponentName receiver = new ComponentName(context, myReceiver.class);

PackageManager pm = context.getPackageManager();

pm.setComponentEnabledSetting(receiver,
        PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
        PackageManager.DONT_KILL_APP)
```

使用這種技術，如果我們確定網絡連接已經斷開，那麼可以在這個時候關閉除了監聽網絡狀態變化的接收器之外的其它所有接收器。

相反的，一旦重新建立網絡連接，我們可以停止監聽網絡連接的改變，而僅僅在執行需要聯網的操作之前判斷當前網絡是否可以用。

同樣地，你可以使用上面的技術來暫緩一個需要更高帶寬的下載任務。這僅需要啟用一個監聽網絡連接變化的BroadcastReceiver，並在連接到Wi-Fi時，初始化下載任務。
