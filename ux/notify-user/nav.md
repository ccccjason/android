# 啟動Activity時保留導航

> 編寫:[fastcome1985](https://github.com/fastcome1985) - 原文:<http://developer.android.com/training/notify-user/navigation.html>

部分設計一個notification的目的是為了保持用戶的導航體驗。為了詳細討論這個課題，請看 [Notifications](developer.android.com/guide/topics/ui/notifiers/notifications.html#NotificationResponse) API引導，分為下列兩種主要情況：

    * 常規的activity
    你啟動的是你application工作流中的一部分[Activity](developer.android.com/reference/android/app/Activity.html)。
    * 特定的activity
    用戶只能從notification中啟動，才能看到這個[Activity](http://developer.android.com/intl/zh-cn/reference/android/app/Activity.html)，在某種意義上，這個[Activity](http://developer.android.com/intl/zh-cn/reference/android/app/Activity.html)是notification的擴展，額外展示了一些notification本身難以展示的信息。


## 設置一個常規的Activity PendingIntent

設置一個直接啟動的入口Activity的PendingIntent，遵循以下步驟：



1  在manifest中定義你application的[Activity](developer.android.com/reference/android/app/Activity.html)層次，最終的manifest文件應該像這個：


```java
<activity
    android:name=".MainActivity"
    android:label="@string/app_name" >
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
<activity
    android:name=".ResultActivity"
    android:parentActivityName=".MainActivity">
    <meta-data
        android:name="android.support.PARENT_ACTIVITY"
        android:value=".MainActivity"/>
</activity>

```

2 在基於啟動[Activity](developer.android.com/reference/android/app/Activity.html)的[Intent](developer.android.com/reference/android/content/Intent.html)中創建一個返回棧，比如：


```java
int id = 1;
...
Intent resultIntent = new Intent(this, ResultActivity.class);
TaskStackBuilder stackBuilder = TaskStackBuilder.create(this);
// Adds the back stack
stackBuilder.addParentStack(ResultActivity.class);
// Adds the Intent to the top of the stack
stackBuilder.addNextIntent(resultIntent);
// Gets a PendingIntent containing the entire back stack
PendingIntent resultPendingIntent =
        stackBuilder.getPendingIntent(0, PendingIntent.FLAG_UPDATE_CURRENT);
...
NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
builder.setContentIntent(resultPendingIntent);
NotificationManager mNotificationManager =
    (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
mNotificationManager.notify(id, builder.build());

```

## 設置一個特定的Activity PendingIntent

一個特定的[Activity](developer.android.com/reference/android/app/Activity.html)不需要一個返回棧，所以你不需要在manifest中定義[Activity](developer.android.com/reference/android/app/Activity.html)的層次，以及你不需要調用 [addParentStack()](developer.android.com/reference/android/support/v4/app/TaskStackBuilder.html#addParentStack(android.app.Activity))方法去構建一個返回棧。作為代替，你需要用manifest設置[Activity](developer.android.com/reference/android/app/Activity.html)任務選項，以及調用 [getActivity()](developer.android.com/reference/android/app/PendingIntent.html#getActivity(android.content.Context,%20int,%20android.content.Intent,%20int))創建[PendingIntent](developer.android.com/reference/android/app/PendingIntent.html)

1. manifest中，在[Activity](developer.android.com/reference/android/app/Activity.html)的 [<activity>](developer.android.com/guide/topics/manifest/activity-element.html) 標籤中增加下列屬性：
  [android:name="activityclass"](developer.android.com/guide/topics/manifest/activity-element.html#nm)
    activity的完整的類名。
  [android:taskAffinity=""](developer.android.com/guide/topics/manifest/activity-element.html#aff)
  結合你在代碼裡設置的[FLAG_ACTIVITY_NEW_TASK](developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_NEW_TASK)標識， 確保這個[Activity](developer.android.com/reference/android/app/Activity.html)不會進入application的默認任務。任何與 application的默認任務有密切關係的任務都不會受到影響。
  [android:excludeFromRecents="true"](developer.android.com/guide/topics/manifest/activity-element.html#exclude)
  將新任務從最近列表中排除，目的是為了防止用戶不小心返回到它。

2. 建立以及發佈notification：
  a.創建一個啟動[Activity](developer.android.com/reference/android/app/Activity.html)的[Intent](developer.android.com/reference/android/content/Intent.html).
  b.通過調用[setFlags()](developer.android.com/reference/android/content/Intent.html#setFlags(int))方法並設置標識[FLAG_ACTIVITY_NEW_TASK](developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_NEW_TASK) 與 [FLAG_ACTIVITY_CLEAR_TASK](developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_CLEAR_TASK)，來設置[Activity](developer.android.com/reference/android/app/Activity.html)在一個新的，空的任務中啟動。
  c.在[Intent](developer.android.com/reference/android/content/Intent.html)中設置其他你需要的選項。
  d.通過調用 [getActivity()](http://developer.android.com/intl/zh-cn/reference/android/app/PendingIntent.html#getActivity%28android.content.Context,%20int,%20android.content.Intent,%20int%29)方法從[Intent](developer.android.com/reference/android/content/Intent.html)中創建一個 [PendingIntent](developer.android.com/reference/android/app/PendingIntent.html)，你可以把這個[PendingIntent](developer.android.com/reference/android/app/PendingIntent.html) 當做 [setContentIntent()](http://developer.android.com/intl/zh-cn/reference/android/support/v4/app/NotificationCompat.Builder.html#setContentIntent%28android.app.PendingIntent%29)的參數來使用。
下面的代碼片段演示了這個過程：

```java
// Instantiate a Builder object.
NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
// Creates an Intent for the Activity
Intent notifyIntent =
        new Intent(new ComponentName(this, ResultActivity.class));
// Sets the Activity to start in a new, empty task
notifyIntent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK |
        Intent.FLAG_ACTIVITY_CLEAR_TASK);
// Creates the PendingIntent
PendingIntent notifyIntent =
        PendingIntent.getActivity(
        this,
        0,
        notifyIntent,
        PendingIntent.FLAG_UPDATE_CURRENT
);

// Puts the PendingIntent into the notification builder
builder.setContentIntent(notifyIntent);
// Notifications are issued by sending them to the
// NotificationManager system service.
NotificationManager mNotificationManager =
    (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
// Builds an anonymous Notification object from the builder, and
// passes it to the NotificationManager
mNotificationManager.notify(id, builder.build());

```
