# 建立一個Notification

> 編寫:[fastcome1985](https://github.com/fastcome1985) - 原文:<http://developer.android.com/training/notify-user/build-notification.html>

* 這節課向你說明如何創建與發佈一個Notification。

* 這節課的例子是基於[NotificationCompat.Builder](developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html)類的，[NotificationCompat.Builder](developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html)在[Support Library](developer.android.com)中。為了給許多各種不同的平臺提供最好的notification支持，你應該使用[NotificationCompat](developer.android.com/reference/android/support/v4/app/NotificationCompat.html)以及它的子類，特別是[NotificationCompat.Builder](developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html)。


## 創建Notification Buider

* 創建Notification時，可以用[NotificationCompat.Builder](developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html)對象指定Notification的UI內容與行為。一個[Builder](developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html)至少包含以下內容：

  * 一個小的icon，用[setSmallIcon()](developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setSmallIcon(int))方法設置
  * 一個標題，用[setContentTitle()](developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setContentTitle(java.lang.CharSequence))方法設置。
  * 詳細的文本，用[setContentText()](developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setContentText(java.lang.CharSequence))方法設置

例如：


```java

NotificationCompat.Builder mBuilder =
    new NotificationCompat.Builder(this)
    .setSmallIcon(R.drawable.notification_icon)
    .setContentTitle("My notification")
    .setContentText("Hello World!");

```

## 定義Notification的Action（行為）

* 儘管在Notification中Actions是可選的，但是你應該至少添加一種Action。一種Action可以讓用戶從Notification直接進入你應用內的[Activity](developer.android.com/reference/android/app/Activity.html)，在這個activity中他們可以查看引起Notification的事件或者做下一步的處理。在Notification中，action本身是由[PendingIntent](developer.android.com/reference/android/app/PendingIntent.html)定義的，PendingIntent包含了一個啟動你應用內[Activity](developer.android.com/reference/android/app/Activity.html)的[Intent](developer.android.com/reference/android/content/Intent.html)。

* 如何構建一個[PendingIntent](developer.android.com/reference/android/app/PendingIntent.html)取決於你要啟動的[activity](developer.android.com/reference/android/app/Activity.html)的類型。當從Notification中啟動一個[activity](developer.android.com/reference/android/app/Activity.html)時，你必須保存用戶的導航體驗。在下面的代碼片段中，點擊Notification啟動一個新的activity，這個activity有效地擴展了Notification的行為。在這種情形下，就沒必要人為地去創建一個返回棧（更多關於這方面的信息，請查看 [Preserving Navigation when Starting an Activity](developer.android.com/intl/zh-cn/training/notify-user/navigation.html)）


```java

Intent resultIntent = new Intent(this, ResultActivity.class);
...
// Because clicking the notification opens a new ("special") activity, there's
// no need to create an artificial back stack.
PendingIntent resultPendingIntent =
    PendingIntent.getActivity(
    this,
    0,
    resultIntent,
    PendingIntent.FLAG_UPDATE_CURRENT
);

```

## 設置Notification的點擊行為

 可以通過調用[NotificationCompat.Builder](developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html)中合適的方法，將上一步創建的[PendingIntent](developer.android.com/reference/android/app/PendingIntent.html)與一個手勢產生關聯。比方說，當點擊Notification抽屜裡的Notification文本時，啟動一個activity，可以通過調用[setContentIntent()](developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setContentIntent(android.app.PendingIntent))方法把[PendingIntent](developer.android.com/reference/android/app/PendingIntent.html)添加進去。

例如：

```java

PendingIntent resultPendingIntent;
...
mBuilder.setContentIntent(resultPendingIntent);

```


## 發佈Notification

為了發佈notification：
    * 獲取一個[NotificationManager](http://www.baidu.com/baidu?wd=NotificationManager.&tn=monline_4_dg)實例
    * 使用[notify()](developer.android.com/reference/java/lang/Object.html#notify())方法發佈Notification。當你調用[notify()](developer.android.com/reference/java/lang/Object.html#notify())方法時，指定一個notification ID。你可以在以後使用這個ID來更新你的notification。這在[Managing Notifications](developer.android.com/intl/zh-cn/training/notify-user/managing.html)中有更詳細的描述。
    * 調用[build()](developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#build())方法，會返回一個包含你的特徵的[Notification](developer.android.com/reference/android/app/Notification.html)對象。

舉個例子：

```java

NotificationCompat.Builder mBuilder;
...
// Sets an ID for the notification
int mNotificationId = 001;
// Gets an instance of the NotificationManager service
NotificationManager mNotifyMgr =
        (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
// Builds the notification and issues it.
mNotifyMgr.notify(mNotificationId, mBuilder.build());

```
