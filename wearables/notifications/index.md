# 為Notification賦加可穿戴特性

> 編寫:[wangyachen](https://github.com/wangyacheng) - 原文: <http://developer.android.com/training/wearables/notifications/index.html>

當一部Android手持設備（手機或平板）與Android可穿戴設備連接後，手持設備能夠自動與可穿戴設備共享Notification。在可穿戴設備上，每個Notification都是以一張新卡片的形式出現在[context stream](http://developer.android.com/design/wear/index.html)中。

與此同時，為了給予用戶以最佳的體驗，開發者應當為自己創建的Notification增加一些具備可穿戴特性的功能。下面的課程將指導我們如何實現同時支持手持設備和可穿戴設備的Notification。

![](notification_phone@2x.png)

**Figure 1.** 同時展示在手持設備和可穿戴設備的Notification

## Lessons

[創建Notification](creating.html)

學習如何應用Android support library創建具備可穿戴特性的Notification。

[在Notification中接收語音輸入](voice-input.html)

學習在可穿戴式設備上的Notification添加一個action以接收來自用戶的語音輸入，並且將錄入的消息傳遞給手持設備應用。

[為Notification添加頁面](pages.html)

學習如何為Notification創建附加的頁面，使得用戶在向左滑動時能看到更多的信息。

[將Notification放成一疊](stacks.html)

學習如何將我們應用中所有相似的notification放在一個堆疊中，使得在不將多個卡片添加到卡片流的情況下，允許用戶能夠獨立地查看每一個Notification。
