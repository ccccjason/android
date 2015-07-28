# 最小化定期更新造成的影響

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/efficient-downloads/regular_updates.html>

最佳的定期更新頻率是不確定的，通常由設備狀態，網絡連接狀態，用戶行為與用戶顯式定義的偏好而決定。

[Optimizing Battery Life](http://developer.android.com/training/monitoring-device-state/index.html)這一章有討論如何根據主設備狀態來修改更新頻率，從而達到編寫一個低電量消耗的程序。可執行的操作包括當斷開網絡連接的時候去關閉後臺服務，在電量比較低的時候減少更新的頻率等。

這一課會介紹更新頻率是多少才會使得更新操作對無線電狀態機的影響最小。(C2DM與指數退避算法的使用)

## 使用 Google Cloud Messaging 來輪詢

<!-- More -->

每次 app 去向 server 詢問檢查是否有更新操作的時候，都會激活無線電，這樣造成了不必要的能量消耗（在3G情況下，會差不多消耗20秒的能量）。

[Google Cloud Messaging for Android (GCM)](http://developer.android.com/google/gcm/index.html) 是一個用來從 server 到特定 app 傳輸數據的輕量級的機制。使用 GCM，server 會在某個 app 需要獲取新數據的時候通知 app 有這個消息。

比起輪詢方式（app 為了即時拿到最新的數據需要定時去ping server），GCM 這種由事件驅動的模式會在僅僅有數據更新的時候通知 app 去創建網絡連接來獲取數據（很顯然這樣減少了 app 的大量操作，當然也減少了很多電量消耗）。

GCM 需要通過使用持續的 TCP/IP 連接來實現操作。當我們可以實現自己的推送服務，最好使用 GCM（這個地方應該不是傳統意義上的固定IP，可以理解為某個會話情況下）
。很明顯，使用 GCM 既減少了網絡連接次數，也優化了帶寬，還減少了對電量的消耗。

**PS：大陸的 Google 框架通常被移除掉，這導致 GCM 實際上根本沒有辦法在大陸的 App 上使用。**

## 使用不嚴格的重複通知和指數避退算法來優化輪詢

如果需要使用輪詢機制，在不影響用戶體驗的前提下，設置默認的更新頻率當然是越低越好（減少耗電量）。

一個簡單的方法是給用戶顯式修改更新頻率的選項，允許用戶自己來處理如何平衡數據及時性與電量的消耗。

當設置安排好更新操作後，可以使用不確定重複提醒的方式來允許系統把當前這個操作進行定向移動（比如推遲一會）。

```java
int alarmType = AlarmManager.ELAPSED_REALTIME;
long interval = AlarmManager.INTERVAL_HOUR;
long start = System.currentTimeMillis() + interval;

alarmManager.setInexactRepeating(alarmType, start, interval, pi);
```

如果幾個提醒都安排在某個點同時被觸發，那麼就可以使得多個操作在同一個無線電狀態下操作完。

如果可以，請設置提醒的類型為 `ELAPSED_REALTIME` 或者 `RTC` 而不是 `_WAKEUP`。通過一直等待知道手機在提醒通知觸發之前不再處於 standby 模式，進一步地減少電量的消耗。

我們還可以通過根據最近 app 被使用的頻率來有選擇性地減少更新的頻率，從而降低這些定期通知的影響。

另一個方法是在 app 在上一次更新操作之後還未被使用的情況下，使用指數退避算法 `exponential back-off algorithm` 來減少更新頻率。斷言一個最小的更新頻率和任何時候使用 app 都去重置頻率通常都是有用的方法。例如：

```java
SharedPreferences sp =
  context.getSharedPreferences(PREFS, Context.MODE_WORLD_READABLE);

boolean appUsed = sp.getBoolean(PREFS_APPUSED, false);
long updateInterval = sp.getLong(PREFS_INTERVAL, DEFAULT_REFRESH_INTERVAL);

if (!appUsed)
  if ((updateInterval *= 2) > MAX_REFRESH_INTERVAL)
    updateInterval = MAX_REFRESH_INTERVAL;

Editor spEdit = sp.edit();
spEdit.putBoolean(PREFS_APPUSED, false);
spEdit.putLong(PREFS_INTERVAL, updateInterval);
spEdit.apply();

rescheduleUpdates(updateInterval);
executeUpdateOrPrefetch();
```

初始化一個網絡連接的花費不會因為是否成功下載了數據而改變。對於那些成功完成是很重要的時間敏感的傳輸，我們可以使用指數退避算法來減少重複嘗試的次數，這樣能夠避免浪費電量。例如：

```java
private void retryIn(long interval) {
  boolean success = attemptTransfer();

  if (!success) {
    retryIn(interval*2 < MAX_RETRY_INTERVAL ?
            interval*2 : MAX_RETRY_INTERVAL);
  }
}
```

另外，對於可以容忍失敗連接的傳輸（例如定期更新），我們可以簡單地忽略失敗的連接和傳輸嘗試。

***

**筆者結語:這一課講到GCM與指數退避算法等，其實這些細節很值得我們注意，如果能在實際項目中加以應用，很明顯程序的質量上升了一個檔次！**
