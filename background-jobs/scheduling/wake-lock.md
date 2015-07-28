# 保持設備喚醒

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/scheduling/wakelock.html>

為了避免電量過度消耗，Android設備會在被閒置之後迅速進入睡眠狀態。然而有時候應用會需要喚醒屏幕或者是喚醒CPU並且保持它們的喚醒狀態，直至一些任務被完成。

想要做到這一點，所採取的方法依賴於應用的具體需求。但是通常來說，我們應該使用最輕量級的方法，減小其對系統資源的影響。在接下來的部分中，我們將會描述在設備默認的睡眠行為與應用的需求不相符合的情況下，我們應該如何進行對應的處理。

## 保持屏幕常亮

某些應用需要保持屏幕常亮，比如遊戲與視頻應用。最好的方式是在你的Activity中（且僅在Activity中，而不是在Service或其他應用組件裡）使用[FLAG_KEEP_SCREEN_ON](https://developer.android.com/reference/android/view/WindowManager.LayoutParams.html#FLAG_KEEP_SCREEN_ON)屬性，例如：

```java
public class MainActivity extends Activity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
  }
```

該方法的優點與喚醒鎖（Wake Locks）不同（喚醒鎖的內容在本章節後半部分），它不需要任何特殊的權限，系統會正確地
管理應用之間的切換，且不必關心釋放資源的問題。

另外一種方法是在應用的XML佈局文件裡，使用[android:keepScreenOn](https://developer.android.com/reference/android/R.attr.html#keepScreenOn)屬性:

```java
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:keepScreenOn="true">
    ...
</RelativeLayout>
```

使用`android:keepScreenOn="true"`與使用[FLAG_KEEP_SCRRE_ON](https://developer.android.com/reference/android/view/WindowManager.LayoutParams.html#FLAG_KEEP_SCREEN_ON)等效。你可以選擇最適合你的應用的方法。在Activity中通過代碼設置常亮標識的優點在於：你可以通過代碼動態清除這個標示，從而使屏幕可以關閉。

> **Notes**：除非你不再希望正在運行的應用長時間點亮屏幕（例如：在一定時間無操作發生後，你想要將屏幕關閉），否則你是不需要清除[FLAG_KEEP_SCRRE_ON](https://developer.android.com/reference/android/view/WindowManager.LayoutParams.html#FLAG_KEEP_SCREEN_ON) 標識的。WindowManager會在應用進入後臺或者返回前臺時，正確管理屏幕的點亮或者關閉。但是如果你想要顯式地清除這一標識，從而使得屏幕能夠關閉，可以使用`getWindow().clearFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)`方法。

## 保持CPU運行

如果你需要在設備睡眠之前，保持CPU運行來完成一些工作，你可以使用[PowerManager](https://developer.android.com/reference/android/os/PowerManager.html)系統服務中的喚醒鎖功能。喚醒鎖允許應用控制設備的電源狀態。

創建和保持喚醒鎖會對設備的電源壽命產生巨大影響。因此你應該僅在你確實需要時使用喚醒鎖，且使用的時間應該越短越好。如果想要在Activity中使用喚醒鎖就顯得沒有必要了。如上所述，可以在Activity中使用[FLAG_KEEP_SCRRE_ON](https://developer.android.com/reference/android/view/WindowManager.LayoutParams.html#FLAG_KEEP_SCREEN_ON)讓屏幕保持常亮。

使用喚醒鎖的一種合理情況可能是：一個後臺服務需要在屏幕關閉時利用喚醒鎖保持CPU運行。再次強調，應該儘可能規避使用該方法，因為它會影響到電池壽命。

> **不必使用喚醒鎖的情況**：
> 1. 如果你的應用正在執行一個HTTP長連接的下載任務，可以考慮使用[DownloadManager](http://developer.android.com/reference/android/app/DownloadManager.html)。
> 2. 如果你的應用正在從一個外部服務器同步數據，可以考慮創建一個[SyncAdapter](http://developer.android.com/training/sync-adapters/index.html)
> 3. 如果你的應用需要依賴於某些後臺服務，可以考慮使用[RepeatingAlarm](http://developer.android.com/training/scheduling/alarms.html)或者[Google Cloud Messaging](http://developer.android.com/google/gcm/index.html)，以此每隔特定的時間，將這些服務激活。

為了使用喚醒鎖，首先需要在應用的Manifest清單文件中增加[WAKE_LOCK](https://developer.android.com/reference/android/Manifest.permission.html#WAKE_LOCK)權限：

```xml
<uses-permission android:name="android.permission.WAKE_LOCK" />
```

如果你的應用包含一個BroadcastReceiver並使用Service來完成一些工作，你可以通過[WakefulBroadcastReceiver](https://developer.android.com/reference/android/support/v4/content/WakefulBroadcastReceiver.html)管理你喚醒鎖。後續章節中將會提到，這是一種推薦的方法。如果你的應用不滿足上述情況，可以使用下面的方法直接設置喚醒鎖：

```java
PowerManager powerManager = (PowerManager) getSystemService(POWER_SERVICE);
Wakelock wakeLock = powerManager.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK,
        "MyWakelockTag");
wakeLock.acquire();
```

可以調用`wakelock.release()`來釋放喚醒鎖。當應用使用完畢時，應該釋放該喚醒鎖，以避免電量過度消耗。

### 使用WakefulBroadcastReceiver

你可以將BroadcastReceiver和Service結合使用，以此來管理後臺任務的生命週期。[WakefulBroadcastReceiver](https://developer.android.com/reference/android/support/v4/content/WakefulBroadcastReceiver.html)是一種特殊的BroadcastReceiver，它專注於創建和管理應用的[PARTIAL_WAKE_LOCK](https://developer.android.com/reference/android/os/PowerManager.html#PARTIAL_WAKE_LOCK)。WakefulBroadcastReceiver會將任務交付給[Service](https://developer.android.com/reference/android/app/Service.html)（一般會是一個[IntentService](https://developer.android.com/reference/android/app/IntentService.html)），同時確保設備在此過程中不會進入睡眠狀態。如果在該過程當中沒有保持住喚醒鎖，那麼還沒等任務完成，設備就有可能進入睡眠狀態了。其結果就是：應用可能會在未來的某一個時間節點才把任務完成，這顯然不是你所期望的。

要使用WakefulBroadcastReceiver，首先在Manifest文件添加一個標籤：

```xml
<receiver android:name=".MyWakefulReceiver"></receiver>
```

下面的代碼通過`startWakefulService()`啟動`MyIntentService`。該方法和`startService()`類似，除了WakeflBroadcastReceiver會在Service啟動後將喚醒鎖保持住。傳遞給`startWakefulService()`的Intent會攜帶有一個Extra數據，用來標識喚醒鎖。

```java
public class MyWakefulReceiver extends WakefulBroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {

        // Start the service, keeping the device awake while the service is
        // launching. This is the Intent to deliver to the service.
        Intent service = new Intent(context, MyIntentService.class);
        startWakefulService(context, service);
    }
}
```

當Service結束之後，它會調用`MyWakefulReceiver.completeWakefulIntent()`來釋放喚醒鎖。`completeWakefulIntent()`方法中的Intent參數是和WakefulBroadcastReceiver傳遞進來的Intent參數一致的：

```java
public class MyIntentService extends IntentService {
    public static final int NOTIFICATION_ID = 1;
    private NotificationManager mNotificationManager;
    NotificationCompat.Builder builder;
    public MyIntentService() {
        super("MyIntentService");
    }
    @Override
    protected void onHandleIntent(Intent intent) {
        Bundle extras = intent.getExtras();
        // Do the work that requires your app to keep the CPU running.
        // ...
        // Release the wake lock provided by the WakefulBroadcastReceiver.
        MyWakefulReceiver.completeWakefulIntent(intent);
    }
}
```
