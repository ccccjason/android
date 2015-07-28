# 為 Notification 添加頁面

> 編寫:[wangyachen](https://github.com/wangyacheng) - 原文:<http://developer.android.com/training/wearables/notifications/pages.html>

當開發者想要在不需要用戶在他們的手機上打開app的情況下，還可以允許表達更多的信息，那麼開發者可以在可穿戴設備上的Notification中添加一個或多個的頁面。添加的頁面會馬上出現在主 Notification 卡片的右邊。

![](09_pages.png)
![](08_pages.png)

為了創建一個擁有多個頁面的 Notification，開發者需要：

1. 通過[NotificationCompat.Builder](http://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html)創建主Notification（首頁），以開發者想要的方式使其出現在手持設備上。
2. 通過[NotificationCompat.Builder](http://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html)為可穿戴設備添加更多的頁面。
3. 通過[addPage()](http://developer.android.com/reference/android/support/v4/app/NotificationCompat.WearableExtender.html#addPage(android.app.Notification))方法將這些頁面應用到主 Notification 中，或者通過[addPages()](http://developer.android.com/reference/android/support/v4/app/NotificationCompat.WearableExtender.html#addPage(android.app.Notification))將多個頁面添加到一個[Collection](http://developer.android.com/reference/java/util/Collection.html)。

舉個例子，以下代碼為Notification添加了第二個頁面：

```java
// Create builder for the main notification
NotificationCompat.Builder notificationBuilder =
        new NotificationCompat.Builder(this)
        .setSmallIcon(R.drawable.new_message)
        .setContentTitle("Page 1")
        .setContentText("Short message")
        .setContentIntent(viewPendingIntent);

// Create a big text style for the second page
BigTextStyle secondPageStyle = new NotificationCompat.BigTextStyle();
secondPageStyle.setBigContentTitle("Page 2")
               .bigText("A lot of text...");

// Create second page notification
Notification secondPageNotification =
        new NotificationCompat.Builder(this)
        .setStyle(secondPageStyle)
        .build();

// Add second page with wearable extender and extend the main notification
Notification twoPageNotification =
        new WearableExtender()
                .addPage(secondPageNotification)
                .extend(notificationBuilder)
                .build();

// Issue the notification
notificationManager =
        NotificationManagerCompat.from(this);
notificationManager.notify(notificationId, twoPageNotification);
```

下一課：[以Stack的方式顯示Notifications](stacks.html)
