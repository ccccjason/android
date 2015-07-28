# 判斷並監測設備的底座狀態與類型

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/monitoring-device-state/connectivity-monitoring.html>

Android設備可以放置在許多不同的底座中，包括車載底座，家庭底座還有數字信號底座以及模擬信號底座等。由於許多底座會向設備充電，因此底座狀態通常與充電狀態密切相關。

你的應用類型決定了底座類型會對更新頻率產生怎樣的影響。對於一個體育類應用，可以讓設備在筆記本底座狀態下增加更新的頻率，或者當設備在車載底座狀態下停止更新。相反的，如果你的後臺服務用來更新交通數據，你也可以選擇在車載底座模式下最大化更新的頻率。

底座狀態也是以Sticky Intent方式來廣播的，這樣可以通過查詢Intent裡面的數據來判斷目前設備是否放置在底座中，以及底座的類型。

## 判斷當前底座狀態

底座狀態的具體信息會以Extra數據的形式，包含在具有[ACTION_DOCK_EVENT](http://developer.android.com/reference/android/content/Intent.html#ACTION_DOCK_EVENT)這一Action的某個Sticky廣播中 ，因此，你不需要為其註冊一個[BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html)。如下所示，僅需要將`null`作為參數傳遞給<a href="http://developer.android.com/reference/android/content/Context.html#registerReceiver(android.content.BroadcastReceiver, android.content.IntentFilter)">registerReceiver()</a>方法就可以了：

```java
IntentFilter ifilter = new IntentFilter(Intent.ACTION_DOCK_EVENT);
Intent dockStatus = context.registerReceiver(null, ifilter);
```

你可以從`EXTRA_DOCK_STATE`這一Extra數據中，提取出當前的底座狀態：

```java
int dockState = battery.getIntExtra(EXTRA_DOCK_STATE, -1);
boolean isDocked = dockState != Intent.EXTRA_DOCK_STATE_UNDOCKED;
```

## 判斷當前底座類型

如果設備被放置在了底座中，那麼它可以有下面四種底座類型：

* Car
* Desk
* Low-End (Analog) Desk
* High-End (Digital) Desk

注意最後兩種底座類型僅在API Level 11及以後版本的Android系統中才被支持。如果你只在乎底座的類型而不管它是數字的還是模擬的，那麼可以僅監測三種類型：

```java
boolean isCar = dockState == EXTRA_DOCK_STATE_CAR;
boolean isDesk = dockState == EXTRA_DOCK_STATE_DESK ||
                 dockState == EXTRA_DOCK_STATE_LE_DESK ||
                 dockState == EXTRA_DOCK_STATE_HE_DESK;
```

## 監測底座狀態或者類型的改變

當設備被放置在或者拔出底座時，系統會發出一個具有[ACTION_DOCK_EVENT](http://developer.android.com/reference/android/content/Intent.html#ACTION_DOCK_EVENT)這一Action的廣播。為了監聽底座狀態的變化，我們只需要在應用的Manifest文件中註冊一個BroadcastReceiver，如下所示：

```xml
<action android:name="android.intent.action.ACTION_DOCK_EVENT"/>
```

之於該BroadcastReceiver的具體實現，可以參考前面提到的那些方法，以此來提取出當前的底座類型和狀態。
