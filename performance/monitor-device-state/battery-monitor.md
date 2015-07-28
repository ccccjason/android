# 監測電池的電量與充電狀態

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/monitoring-device-state/battery-monitoring.html>

當你想通過改變後臺更新操作的頻率來減少對電池壽命的影響時，那麼首先需要檢查當前電量與充電狀態。

執行應用更新對電池壽命的影響是與電量和充電狀態密切相關的。當使用交流電對設備充電時，更新操作的影響可以忽略不計，所以在大多數情況下，如果使用壁式充電器對設備進行充電，我們可以將刷新頻率設置到最大。相反的，如果設備沒有在充電狀態，那麼我們就需要儘量減少設備的更新操作來延長電池的續航能力。

同樣的，如果我們監測到電量即將耗盡時，那麼應該儘可能降低甚至停止更新操作。

## 判斷當前充電狀態

首先來看一下應該如何確定當前的充電狀態。[BatteryManager](http://developer.android.com/reference/android/os/BatteryManager.html)會廣播一個帶有電池與充電詳情的[Sticky Intent](http://developer.android.com/guide/topics/fundamentals/services.html)

因為廣播的是一個sticky Intent，所以不需要註冊[BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html)。僅僅只需要調用一個以`null`作為Receiver參數的`registerReceiver()`方法就可以了。如下面的代碼片段中展示的那樣，它返回了保存當前電池信息的Intent。你也可以在這裡傳入一個實際的[BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html)對象，但這並不是必須的。

```java
IntentFilter ifilter = new IntentFilter(Intent.ACTION_BATTERY_CHANGED);
Intent batteryStatus = context.registerReceiver(null, ifilter);
```

我們可以提取出當前的充電狀態，以及設備處於充電時，是通過USB還是交流充電器充電的。

```java
// Are we charging / charged?
int status = batteryStatus.getIntExtra(BatteryManager.EXTRA_STATUS, -1);
boolean isCharging = status == BatteryManager.BATTERY_STATUS_CHARGING ||
                     status == BatteryManager.BATTERY_STATUS_FULL;

// How are we charging?
int chargePlug = batteryStatus.getIntExtra(BatteryManager.EXTRA_PLUGGED, -1);
boolean usbCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_USB;
boolean acCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_AC;
```

通常，我們可以在設備使用交流充電時最大化後臺更新頻率，在使用USB充電時降低更新頻率，在非充電狀態時，將更新頻率進一步降低。

## 監測充電狀態的改變

充電狀態隨時可能改變，所以我們應該檢查充電狀態的改變來調整更新頻率。

[BatteryManager](http://developer.android.com/reference/android/os/BatteryManager.html)會在設備連接或者斷開充電器的時候廣播一個Action。即使應用沒有運行，我們也應該接收這些事件的廣播，主要原因是因為這些事件會影響到應用啟動（從而進行更新）的頻率，因此我們應該在Manifest文件裡面註冊一個[BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html)來監聽含有[ACTION_POWER_CONNECTED](http://developer.android.com/reference/android/content/Intent.html#ACTION_POWER_CONNECTED) 與 [ACTION_POWER_DISCONNECTED](http://developer.android.com/reference/android/content/Intent.html#ACTION_POWER_DISCONNECTED)的Intent。

```xml
<receiver android:name=".PowerConnectionReceiver">
  <intent-filter>
    <action android:name="android.intent.action.ACTION_POWER_CONNECTED"/>
    <action android:name="android.intent.action.ACTION_POWER_DISCONNECTED"/>
  </intent-filter>
</receiver>
```

我們可以在該[BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html)的實現中，提取出當前的充電狀態，如下所示：

```java
public class PowerConnectionReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        int status = intent.getIntExtra(BatteryManager.EXTRA_STATUS, -1);
        boolean isCharging = status == BatteryManager.BATTERY_STATUS_CHARGING ||
                            status == BatteryManager.BATTERY_STATUS_FULL;

        int chargePlug = intent.getIntExtra(BatteryManager.EXTRA_PLUGGED, -1);
        boolean usbCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_USB;
        boolean acCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_AC;
    }
}
```

## 判斷當前電池電量

在一些情況下，獲取到當前電池電量也很有幫助。我們可以在獲知電量少於某個級別的時候減少後臺的更新頻率。
我們可以通過電池狀態Intent獲取到電池電量與容量等信息，如下所示：

```java
int level = batteryStatus.getIntExtra(BatteryManager.EXTRA_LEVEL, -1);
int scale = batteryStatus.getIntExtra(BatteryManager.EXTRA_SCALE, -1);

float batteryPct = level / (float)scale;
```

## 檢測電量的有效改變

我們不能不停地監測電池狀態，實際上這也是不必要的。通常來說，不間斷地監測電量信息對電池的影響會遠大於應用本身對電池的影響。所以我們應該僅監測電量的一些顯著性變化，特別是當設備進入或者離開低電量狀態時。

在下面的Manifest文件片段中，BroadcastReceiver僅僅監聽`ACTION_BATTERY_LOW`與`ACTION_BATTERY_OKAY`，這樣它就只會在設備電量進入低電量或者離開低電量的時候被觸發。

```xml
<receiver android:name=".BatteryLevelReceiver">
<intent-filter>
  <action android:name="android.intent.action.ACTION_BATTERY_LOW"/>
  <action android:name="android.intent.action.ACTION_BATTERY_OKAY"/>
  </intent-filter>
</receiver>
```

通常我們都需要在進入低電量的情況下，關閉所有後臺更新來維持設備的續航，因為這個時候做任何更新等操作都極有可能是無用的，因為也許在你還沒來得及處理更新的數據時，設備就因電量耗盡而自動關機了。

在很多時候，用戶往往會將設備放入某種底座中充電（譯註：比如車載的底座式充電器），在下一節課程當中，我們將會學習如何確定當前的底座狀態，以及如何監聽設備底座的變化。