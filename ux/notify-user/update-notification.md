# 更新Notification

> 編寫:[fastcome1985](https://github.com/fastcome1985) - 原文:<http://developer.android.com/training/notify-user/managing.html>

當你需要對同一事件發佈多次Notification時，你應該避免每次都生成一個全新的Notification。相反，你應該考慮去更新先前的Notification，或者改變它的值，或者增加一些值，或者兩者同時進行。

下面的章節描述瞭如何更新Notifications，以及如何移除它們。



## 改變一個Notification

想要設置一個可以被更新的Notification，需要在發佈它的時候調用[NotificationManager.notify(ID, notification)](developer.android.com/reference/android/app/NotificationManager.html#notify(int,%20android.app.Notification))方法為它指定一個notification ID。更新一個已經發布的Notification，需要更新或者創建一個[NotificationCompat.Builder](developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html)對象，並從這個對象創建一個[Notification](developer.android.com/reference/android/app/Notification.html)對象，然後用與先前一樣的ID去發佈這個[Notification](developer.android.com/reference/android/app/Notification.html)。

下面的代碼片段演示了更新一個notification來反映事件發生的次數，它把notification堆積起來，顯示一個總數。


```java

mNotificationManager =
        (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
// Sets an ID for the notification, so it can be updated
int notifyID = 1;
mNotifyBuilder = new NotificationCompat.Builder(this)
    .setContentTitle("New Message")
    .setContentText("You've received new messages.")
    .setSmallIcon(R.drawable.ic_notify_status)
numMessages = 0;
// Start of a loop that processes data and then notifies the user
...
    mNotifyBuilder.setContentText(currentText)
        .setNumber(++numMessages);
    // Because the ID remains unchanged, the existing notification is
    // updated.
    mNotificationManager.notify(
            notifyID,
            mNotifyBuilder.build());
...

```

## 移除Notification

Notifications 將持續可見，除非下面任何一種情況發生。


    * 用戶清除Notification單獨地或者使用“清除所有”（如果Notification能被清除）。
    * 你在創建notification時調用了 setAutoCancel(developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setAutoCancel(boolean))方法，以及用戶點擊了這個notification，
    * 你為一個指定的 notification ID調用了[cancel()](developer.android.com/reference/android/app/NotificationManager.html#cancel(int))方法。這個方法也會刪除正在進行的notifications。
    * 你調用了[cancelAll()](developer.android.com/reference/android/app/NotificationManager.html#cancelAll())方法，它將會移除你先前發佈的所有Notification。
