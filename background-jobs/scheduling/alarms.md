# 調度重複的鬧鐘

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/scheduling/alarms.html>

鬧鐘（基於[AlarmManager](https://developer.android.com/reference/android/app/AlarmManager.html)類）給予你一種在應用使用期之外執行與時間相關的操作的方法。你可以使用鬧鐘初始化一個長時間的操作，例如每天開啟一次後臺服務，下載當日的天氣預報。

鬧鐘具有如下特性：

* 允許你通過預設時間或者設定某個時間間隔，來觸發Intent；
* 你可以將它與BroadcastReceiver相結合，來啟動服務並執行其他操作；
* 可在應用範圍之外執行，所以你可以在你的應用沒有運行或設備處於睡眠狀態的情況下，使用它來觸發事件或行為；
* 幫助你的應用最小化資源需求，你可以使用鬧鐘調度你的任務，來替代計時器或者長時間連續運行的後臺服務。

> **Note**：對於那些需要確保在應用使用期之內發生的定時操作，可以使用鬧鐘替代使用[Handler](https://developer.android.com/reference/android/os/Handler.html)結合[Timer](https://developer.android.com/reference/java/util/Timer.html)與[Thread](https://developer.android.com/reference/java/lang/Thread.html)的方法。因為它可以讓Android系統更好地統籌系統資源。

## 權衡利弊

重複鬧鐘的機制比較簡單，沒有太多的靈活性。它對於你的應用來說或許不是一種最好的選擇，特別是當你想要觸發網絡操作的時候。設計不佳的鬧鐘會導致電量快速耗盡，而且會對服務端產生巨大的負荷。

當我們從服務端同步數據時，往往會在應用不被使用的時候時被喚醒觸發執行某些操作。此時你可能希望使用重複鬧鐘。但是如果存儲數據的服務端是由你控制的，使用[Google Cloud Messaging](https://developer.android.com/google/gcm/index.html)（GCM）結合[sync adapter](https://developer.android.com/training/sync-adapters/index.html)是一種更好解決方案。SyncAdapter提供的任務調度選項和[AlarmManager](https://developer.android.com/reference/android/app/AlarmManager.html)基本相同，但是它能提供更多的靈活性。比如：同步的觸發可能基於一條“新數據”提示消息，而消息的產生可以基於服務器或設備，用戶的操作（或者沒有操作），每天的某一時刻等等。

### 最佳實踐方法

在設計重複鬧鐘過程中，你所做出的每一個決定都有可能影響到你的應用將會如何使用系統資源。例如，我們假想一個會從服務器同步數據的應用。同步操作基於的是時鐘時間，具體來說，每一個應用的實例會在下午十一點整進行同步，巨大的服務器負荷會導致服務器響應時間變長，甚至拒絕服務。因此在我們使用鬧鐘時，請牢記下面的最佳實踐建議：

*  對任何由重複鬧鐘觸發的網絡請求添加一定的隨機性（抖動）：
	* 在鬧鐘觸發時做一些本地任務。“本地任務”指的是任何不需要訪問服務器或者從服務器獲取數據的任務；
	* 同時對於那些包含有網絡請求的鬧鐘，在調度時機上增加一些隨機性。
* 儘量讓你的鬧鐘頻率最小；
* 如果不是必要的情況，不要喚醒設備（這一點與鬧鐘的類型有關，本節課後續部分會提到）；
* 觸發鬧鐘的時間不必過度精確；
儘量使用`setInexactRepeating()`方法替代`setRepeating()`方法。當你使用`setInexactRepeating()`方法時，Android系統會集中多個應用的重複鬧鐘同步請求，並一起觸發它們。這可以減少系統將設備喚醒的總次數，以此減少電量消耗。從Android 4.4（API Level19）開始，所有的重複鬧鐘都將是非精確型的。注意雖然`setInexactRepeating()`是`setRepeating()`的改進版本，它依然可能會導致每一個應用的實例在某一時間段內同時訪問服務器，造成服務器負荷過重。因此如之前所述，對於網絡請求，我們需要為鬧鐘的觸發時機增加隨機性。
* 儘量避免讓鬧鐘基於時鐘時間。

想要在某一個精確時刻觸發重複鬧鐘是比較困難的。我們應該儘可能使用[ELAPSED_REALTIME](https://developer.android.com/reference/android/app/AlarmManager.html#ELAPSED_REALTIME)。不同的鬧鐘類型會在本節課後半部分展開。

## 設置重複鬧鐘

如上所述，對於定期執行的任務或者數據查詢而言，使用重複鬧鐘是一個不錯的選擇。它具有下列屬性：

* 鬧鐘類型（後續章節中會展開討論）；
* 觸發時間。如果觸發時間是過去的某個時間點，鬧鐘會立即被觸發；
* 鬧鐘間隔時間。例如，一天一次，每小時一次，每五秒一次，等等；
* 在鬧鐘被觸發時才被髮出的Pending Intent。如果你為同一個Pending Intent設置了另一個鬧鐘，那麼它會將第一個鬧鐘覆蓋。

### 選擇鬧鐘類型

使用重複鬧鐘要考慮的第一件事情是鬧鐘的類型。

鬧鐘類型有兩大類：`ELAPSED_REALTIME`和`REAL_TIME_CLOCK`（RTC）。`ELAPSED_REALTIME`從系統啟動之後開始計算，`REAL_TIME_CLOCK`使用的是世界統一時間（UTC）。也就是說由於`ELAPSED_REALTIME`不受地區和時區的影響，所以它適合於基於時間差的鬧鐘（例如一個每過30秒觸發一次的鬧鐘）。`REAL_TIME_CLOCK`適合於那些依賴於地區位置的鬧鐘。

兩種類型的鬧鐘都還有一個喚醒（`WAKEUP`）版本，也就是可以在設備屏幕關閉的時候喚醒CPU。這可以確保鬧鐘會在既定的時間被激活，這對於那些實時性要求比較高的應用（比如含有一些對執行時間有要求的操作）來說非常有效。如果你沒有使用喚醒版本的鬧鐘，那麼所有的重複鬧鐘會在下一次設備被喚醒時被激活。

如果你只是簡單的希望鬧鐘在一個特定的時間間隔被激活（例如每半小時一次），那麼你可以使用任意一種`ELAPSED_REALTIME`類型的鬧鐘，通常這會是一個更好的選擇。

如果你的鬧鐘是在每一天的特定時間被激活，那麼你可以選擇`REAL_TIME_CLOCK`類型的鬧鐘。不過需要注意的是，這個方法會有一些缺陷——如果地區發生了變化，應用可能無法做出正確的改變；另外，如果用戶改變了設備的時間設置，這可能會造成應用產生預期之外的行為。使用`REAL_TIME_CLOCK`類型的鬧鐘還會有精度的問題，因此我們建議你儘可能使用`ELAPSED_REALTIME`類型。

下面列出鬧鐘的具體類型：

* [ELAPSED_REALTIME](https://developer.android.com/reference/android/app/AlarmManager.html#ELAPSED_REALTIME)：從設備啟動之後開始算起，度過了某一段特定時間後，激活Pending Intent，但不會喚醒設備。其中設備睡眠的時間也會包含在內。
* [ELAPSED_REALTIME_WAKEUP](https://developer.android.com/reference/android/app/AlarmManager.html#ELAPSED_REALTIME_WAKEUP)：從設備啟動之後開始算起，度過了某一段特定時間後喚醒設備。
* [RTC](https://developer.android.com/reference/android/app/AlarmManager.html#RTC)：在某一個特定時刻激活Pending Intent，但不會喚醒設備。
* [RTC_WAKEUP](https://developer.android.com/reference/android/app/AlarmManager.html#RTC_WAKEUP)：在某一個特定時刻喚醒設備並激活Pending Intent。

### ELAPSED_REALTIME_WAKEUP案例

下面是使用[ELAPSED_REALTIME_WAKEUP](https://developer.android.com/reference/android/app/AlarmManager.html#ELAPSED_REALTIME_WAKEUP)的例子。

每隔在30分鐘後喚醒設備以激活鬧鐘：

```java
// Hopefully your alarm will have a lower frequency than this!
alarmMgr.setInexactRepeating(AlarmManager.ELAPSED_REALTIME_WAKEUP,
        AlarmManager.INTERVAL_HALF_HOUR,
        AlarmManager.INTERVAL_HALF_HOUR, alarmIntent);
```

在一分鐘後喚醒設備並激活一個一次性（無重複）鬧鐘：

```java
private AlarmManager alarmMgr;
private PendingIntent alarmIntent;
...
alarmMgr = (AlarmManager)context.getSystemService(Context.ALARM_SERVICE);
Intent intent = new Intent(context, AlarmReceiver.class);
alarmIntent = PendingIntent.getBroadcast(context, 0, intent, 0);

alarmMgr.set(AlarmManager.ELAPSED_REALTIME_WAKEUP,
        SystemClock.elapsedRealtime() +
        60 * 1000, alarmIntent);
```

### RTC案例

下面是使用[RTC_WAKEUP](https://developer.android.com/reference/android/app/AlarmManager.html#RTC_WAKEUP)的例子。

在大約下午2點喚醒設備並激活鬧鐘，並不斷重複：

```java
// Set the alarm to start at approximately 2:00 p.m.
Calendar calendar = Calendar.getInstance();
calendar.setTimeInMillis(System.currentTimeMillis());
calendar.set(Calendar.HOUR_OF_DAY, 14);

// With setInexactRepeating(), you have to use one of the AlarmManager interval
// constants--in this case, AlarmManager.INTERVAL_DAY.
alarmMgr.setInexactRepeating(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(),
        AlarmManager.INTERVAL_DAY, alarmIntent);
```

讓設備精確地在上午8點半被喚醒並激活鬧鐘，自此之後每20分鐘喚醒一次：

```java
private AlarmManager alarmMgr;
private PendingIntent alarmIntent;
...
alarmMgr = (AlarmManager)context.getSystemService(Context.ALARM_SERVICE);
Intent intent = new Intent(context, AlarmReceiver.class);
alarmIntent = PendingIntent.getBroadcast(context, 0, intent, 0);

// Set the alarm to start at 8:30 a.m.
Calendar calendar = Calendar.getInstance();
calendar.setTimeInMillis(System.currentTimeMillis());
calendar.set(Calendar.HOUR_OF_DAY, 8);
calendar.set(Calendar.MINUTE, 30);

// setRepeating() lets you specify a precise custom interval--in this case,
// 20 minutes.
alarmMgr.setRepeating(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(),
        1000 * 60 * 20, alarmIntent);
```

### 決定鬧鐘的精確度

如上所述，創建鬧鐘的第一步是要選擇鬧鐘的類型，然後你需要決定鬧鐘的精確度。對於大多數應用而言，`setInexactRepeating()`會是一個正確的選擇。當你使用該方法時，Android系統會集中多個應用的重複鬧鐘同步請求，並一起觸發它們。這樣可以減少電量的損耗。

對於另一些實時性要求較高的應用——例如，鬧鐘需要精確地在上午8點半被激活，並且自此之後每隔1小時激活一次——那麼可以使用`setRepeating()`。不過你應該儘量避免使用精確的鬧鐘。

使用`setRepeating()`時，你可以制定一個自定義的時間間隔，但在使用`setInexactRepeating()`時不支持這麼做。此時你只能選擇一些時間間隔常量，例如：[INTERVAL_FIFTEEN_MINUTES](https://developer.android.com/reference/android/app/AlarmManager.html#INTERVAL_FIFTEEN_MINUTES) ，[INTERVAL_DAY](http://developer.android.com/reference/android/app/AlarmManager.html#INTERVAL_DAY)等。完整的常量列表，可以查看[AlarmManager](https://developer.android.com/reference/android/app/AlarmManager.html)。

### 取消鬧鐘

你可能希望在應用中添加取消鬧鐘的功能。要取消鬧鐘，可以調用AlarmManager的`cancel()`方法，並把你不想激活的[PendingIntent](https://developer.android.com/reference/android/app/PendingIntent.html)傳遞進去，例如：

```java
// If the alarm has been set, cancel it.
if (alarmMgr!= null) {
    alarmMgr.cancel(alarmIntent);
}
```

###在設備啟動後啟用鬧鐘

默認情況下，所有的鬧鐘會在設備關閉時被取消。要防止鬧鐘被取消，你可以讓你的應用在用戶重啟設備後自動重啟一個重複鬧鐘。這樣可以讓[AlarmManager](https://developer.android.com/reference/android/app/AlarmManager.html)繼續執行它的工作，且不需要用戶手動重啟鬧鐘。

具體步驟如下：

1.在應用的Manifest文件中設置[RECEIVE_BOOT_CMPLETED](https://developer.android.com/reference/android/Manifest.permission.html#RECEIVE_BOOT_COMPLETED)權限，這將允許你的應用接收系統啟動完成後發出的[ACTION_BOOT_COMPLETED](https://developer.android.com/reference/android/content/Intent.html#ACTION_BOOT_COMPLETED)廣播（只有在用戶至少將你的應用啟動了一次後，這樣做才有效）：

```xml
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
```

2.實現[BoradcastReceiver](https://developer.android.com/reference/android/content/BroadcastReceiver.html)用於接收廣播：

```java
public class SampleBootReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        if (intent.getAction().equals("android.intent.action.BOOT_COMPLETED")) {
            // Set the alarm here.
        }
    }
}
```

3.在你的Manifest文件中添加一個接收器，其Intent-Filter接收[ACTION_BOOT_COMPLETED](https://developer.android.com/reference/android/content/Intent.html#ACTION_BOOT_COMPLETED)這一Action：

```xml
<receiver android:name=".SampleBootReceiver"
        android:enabled="false">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"></action>
    </intent-filter>
</receiver>
```

注意Manifest文件中，對接收器設置了`android:enabled="false"`屬性。這意味著除非應用顯式地啟用它，不然該接收器將不被調用。這可以防止接收器被不必要地調用。你可以像下面這樣啟動接收器（比如用戶設置了一個鬧鐘）：

```java
ComponentName receiver = new ComponentName(context, SampleBootReceiver.class);
PackageManager pm = context.getPackageManager();

pm.setComponentEnabledSetting(receiver,
        PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
        PackageManager.DONT_KILL_APP);
```

一旦你像上面那樣啟動了接收器，它將一直保持啟動狀態，即使用戶重啟了設備也不例外。換句話說，通過代碼設置的啟用配置將會覆蓋掉Manifest文件中的現有配置，即使重啟也不例外。接收器將保持啟動狀態，直到你的應用將其禁用。你可以像下面這樣禁用接收器（比如用戶取消了一個鬧鐘）：

```java
ComponentName receiver = new ComponentName(context, SampleBootReceiver.class);
PackageManager pm = context.getPackageManager();

pm.setComponentEnabledSetting(receiver,
        PackageManager.COMPONENT_ENABLED_STATE_DISABLED,
        PackageManager.DONT_KILL_APP);
```
