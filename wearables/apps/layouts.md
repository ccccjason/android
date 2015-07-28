# 創建自定義的佈局

> 編寫: [kesenhoo](https://github.com/kesenhoo) - 原文: <http://developer.android.com/training/wearables/apps/layouts.html>

為可穿戴設備創建佈局是和手持設備是一樣的，除了我們需要為屏幕的尺寸和glanceability進行設計。但是不要期望通過搬遷手持應用的功能與UI到可穿戴上會有一個好的用戶體驗。僅僅在有需要的時候，我們才應該創建自定義的佈局。請參考可穿戴設備的[design guidelines](http://developer.android.com/design/wear/index.html)學習如何設計一個優秀的可穿戴應用。

<a name="CustomNotification"></a>
## 創建自定義Notification

通常來說，我們應該在手持應用上創建好notification，然後讓它自動同步到可穿戴設備上。這讓我們只需要創建一次notification，然後可以在不同類型的設備(不僅僅是可穿戴設備，也包含車載設備與電視)上進行顯示，免去為不同設備進行重新設計。

如果標準的notification風格無法滿足我們的需求(例如[NotificationCompat.BigTextStyle](http://developer.android.com/reference/android/support/v4/app/NotificationCompat.BigTextStyle.html) 或者 [NotificationCompat.InboxStyle](http://developer.android.com/reference/android/support/v4/app/NotificationCompat.InboxStyle.html))，我們可以顯示一個使用自定義佈局的activity。我們只可以在可穿戴設備上創建並處理自定義的notification，同時系統不會將這些notification同步到手持設備上。

**Note:** 當在可穿戴設備上創建自定義的notification時，我們可以使用標準notification API（API Level 20），不需要使用Support Library。

為了創建自定義的notification，步驟如下：

1. 創建佈局並設置這個佈局為需要顯示的activity的content view:
```java
public void onCreate(Bundle bundle){
    ...
    setContentView(R.layout.notification_activity);
}
```
2. 為了使得activity能夠顯示在可穿戴設備上，需要在manifest文件中為activity定義必須的屬性。我們需要把activity聲明為exportable，embeddable以及擁有一個空的task affinity。我們也推薦把activity的主題設置為` Theme.DeviceDefault.Light`。例如：
```xml
<activity android:name="com.example.MyDisplayActivity"
     android:exported="true"
     android:allowEmbedded="true"
     android:taskAffinity=""
     android:theme="@android:style/Theme.DeviceDefault.Light" />
```
3. 為activity創建[PendingIntent](http://developer.android.com/reference/android/app/PendingIntent.html)，例如：：
```java
Intent notificationIntent = new Intent(this, NotificationActivity.class);
PendingIntent notificationPendingIntent = PendingIntent.getActivity(this, 0, notificationIntent,
        PendingIntent.FLAG_UPDATE_CURRENT);
```
4. 創建[Notification](http://developer.android.com/reference/android/app/Notification.html)並執行[setDisplayIntent()](http://developer.android.com/reference/android/app/Notification.WearableExtender.html#setDisplayIntent(android.app.PendingIntent))方法，參數是前面創建的PendingIntent。當用戶查看這個notification時，系統使用這個PendingIntent來啟動activity。
5. 使用[notify()](http://developer.android.com/reference/java/lang/Object.html#notify())方法觸發notification。

> **Note:** 當notification呈現在主頁時，系統會根據notification的語義，使用一個標準的模板來呈現它。這個模板可以在所有的錶盤上進行顯示。當用戶往上滑動notification時，將會看到為這個notification準備的自定義的activity。

<a name="UiLibrary"></a>
## 使用Wearable UI庫創建佈局

當我們使用Android Studio的工程嚮導創建一個Wearable應用的時候，會自動包含Wearable UI庫。你也可以通過給`build.gradle`文件添加下面的依賴聲明把庫文件添加到項目：

```xml
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.google.android.support:wearable:+'
    compile 'com.google.android.gms:play-services-wearable:+'
}
```

這個庫文件幫助我們建立為可穿戴設備設計的UI。更詳細的介紹請看[為可穿戴設備創建自定義UI](http://hukai.me/android-training-course-in-chinese/wearables/ui/index.html)。

下面是一些Wearable UI庫中主要的類：

* **BoxInsetLayout** - 一個能夠感知屏幕的形狀並把子控件居中擺放在一個圓形屏幕的FrameLayout。
* **CardFragment** - 一個能夠可拉伸，垂直可滑動卡片的fragment。
* **CircledImageView** - 一個圓形的image view。
* **ConfirmationActivity** - 一個在用戶完成一個操作之後用來顯示確認動畫的activity。
* **CrossFadeDrawable** - 一個drawable。該drawable包含兩個子drawable和提供方法來調整這兩個子drawable的融合方式。
* **DelayedConfirmationView** - 一個view。提供一個圓形倒計時器，這個計時器通常用於在一段短暫的延遲結束後自動確認某個操作。
* **DismissOverlayView** - 一個用來實現長按消失的View。
* **DotsPageIndicator** - 一個為GridViewPager提供的指示標記，用於指定當前頁面相對於所有頁面的位置。
* **GridViewPager** - 一個可以橫向與縱向滑動的局部控制器。你需要提供一個GridPagerAdapter用來生成顯示頁面的數據。
* **GridPagerAdapter** - 一個提供給GridViewPager顯示頁面的adapter。
* **FragmentGridPagerAdapter** - 一個將每個頁面表示為一個fragment的GridPagerAdapter實現。
* **WatchViewStub** - 一個可以根據屏幕的形狀生成特定佈局的類。
* **WearableListView** - 一個針對可穿戴設備優化過後的ListView。它會垂直的顯示列表內容，並在用戶停止滑動時自動顯示最靠近的Item。

### Wear UI library API reference

這個參考文獻解釋瞭如何詳細地使用每個UI組件。查看[Wear API reference documentation](http://developer.android.com/reference/android/support/wearable/view/package-summary.html)瞭解上述類的用法。

### 為用於Eclipse ADT下載Wearable UI庫

如果你正在使用Eclipse ADT，那麼下載[Wearable UI library](http://developer.android.com/shareables/training/wearable-support-lib.zip)將Wearable UI庫導入到你的工程當中。

> **Note:** 我們推薦使用[Android Studio](http://developer.android.com/sdk/index.html)來開發可穿戴應用。

