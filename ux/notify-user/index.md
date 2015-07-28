# 通知提示用戶

> 編寫:[fastcome1985](https://github.com/fastcome1985) - 原文:<http://developer.android.com/training/notify-user/index.html>

* Notification是一種在你APP常規UI外展示、用來指示某個事件發生的用戶交互元素。用戶可以在使用其它apps時查看notification，並在方便的時候做出迴應。

*  [Notification設計指導](developer.android.com/design/patterns/notifications.html)向你展示如何設計實用的notifications以及何時使用它們。這節課將會教你實現大多數常用的notification設計。

* 完整的Demo示例：[NotifyUser.zip](developer.android.com/shareables/training/NotifyUser.zip)

## Lessons

* [建立一個Notification](build-notification.md)

  學習如何創建一個notification [Builder](developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html)，設置需要的特徵，以及發佈notification。


* [當Activity啟動時保留導航](nav.md)

  學習如何為一個從notification啟動的[Activity](http://developer.android.com/intl/zh-cn/reference/android/app/Activity.html)執行適當的導航。


* [更新notifications](update-notification.md)

  學習如何更新與移除notifications


* [使用BigView風格](expand-notification.md)

  學習用擴展的notification來創建一個BigView，並且維持老版本的兼容性。


* [顯示notification進度](progess-notification.md)

  學習在notification中顯示某個操作的進度，既可以用於那些你可以估算已經完成多少（確定進度，determinate）的操作，也可以用於那些你無法知道完成了多少（不確定進度，indefinite ）的操作
