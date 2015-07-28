# 提供向後的導航

> 編寫:[Lin-H](https://github.com/Lin-H) - 原文:<http://developer.android.com/training/implementing-navigation/temporal.html>

向後導航(Back navigation)是用戶根據屏幕歷史記錄返回之前所查看的界面。所有Android設備都可以為這種導航提供後退按鈕，所以**你的app不需要在UI中添加後退按鈕**。

在幾乎所有情況下，當用戶在應用中進行導航時，系統會保存activity的後退棧。這樣當用戶點擊後退按鈕時，系統可以正確地向後導航。但是，有少數幾種情況需要手動指定app的後退操作，來提供更好的用戶體驗。

>**Back Navigation 設計**

>在繼續閱讀篇文章之前，你應該先在[Navigation](http://developer.android.com/design/patterns/navigation.html) design guide中對後退導航的概念和設計準則有個瞭解。

手動指定後退操作需要的導航模式:

* 當用戶從[notification](http://developer.android.com/guide/topics/ui/notifiers/notifications.html)(通知)，[app widget](http://developer.android.com/guide/topics/appwidgets/index.html)，[navigation drawer](http://developer.android.com/training/implementing-navigation/nav-drawer.html)直接進入深層次activity。

* 用戶在[fragment](http://developer.android.com/guide/components/fragments.html)之間切換的某些情況。

* 當用戶在[WebView](http://developer.android.com/reference/android/webkit/WebView.html)中對網頁進行導航。

下面說明如何在這幾種情況下實現恰當的向後導航。

## 為深度鏈接合併新的後退棧

一般而言，當用戶從一個activity導航到下一個時，系統會遞增地創建後退棧。但是當用戶從一個在自己的任務中啟動activity的深度鏈接進入app，你就有必要去同步新的後退棧，因為新的activity是運行在一個沒有任何後退棧的任務中。

例如，當用戶從通知進入你的app中的深層activity時，你應該添加別的activity到你的任務的後退棧中，這樣當點擊後退(Back)時向上導航，而不是退出app。這個模式在[Navigation](http://developer.android.com/design/patterns/navigation.html#into-your-app) design guide中有更詳細的介紹。

### 在manifest中指定父activity

從Android 4.1 (API level 16)開始，你可以通過指定[`<activity>`](http://developer.android.com/guide/topics/manifest/activity-element.html)元素中的[android:parentActivityName](http://developer.android.com/guide/topics/manifest/activity-element.html#parent)屬性來聲明每一個activity的邏輯父activity。這樣系統可以使導航模式變得更容易，因為系統可以根據這些信息判斷邏輯Back Up navigation的路徑。

如果你的app需要支持Android 4.0以下版本，在你的app中包含[Support Library](http://developer.android.com/tools/support-library/index.html)並添加[`<meta-data>`](http://developer.android.com/guide/topics/manifest/meta-data-element.html)元素到[`<activity>`](http://developer.android.com/guide/topics/manifest/activity-element.html)中。然後指定父activity的值為`android.support.PARENT_ACTIVITY`，並匹配[android:parentActivityName](http://developer.android.com/guide/topics/manifest/activity-element.html#parent)的值。

例如:

```xml
<application ... >
    ...
    <!-- main/home activity (沒有父activity) -->
    <activity
        android:name="com.example.myfirstapp.MainActivity" ...>
        ...
    </activity>
    <!-- 主activity的一個子activity -->
    <activity
        android:name="com.example.myfirstapp.DisplayMessageActivity"
        android:label="@string/title_activity_display_message"
        android:parentActivityName="com.example.myfirstapp.MainActivity" >
        <!-- 4.1 以下的版本需要使用meta-data元素 -->
        <meta-data
            android:name="android.support.PARENT_ACTIVITY"
            android:value="com.example.myfirstapp.MainActivity" />
    </activity>
</application>
```

當父activity用這種方式聲明，你就可以使用[NavUtils](http://developer.android.com/reference/android/support/v4/app/NavUtils.html) API，通過確定每個activity相應的父activity來同步新的後退棧。

### 在啟動activity時創建後退棧

在發生用戶進入app的事件時，開始添加activity到後退棧中。就是說，使用[TaskStackBuilder](http://developer.android.com/reference/android/support/v4/app/TaskStackBuilder.html) API定義每個被放到新後退棧的activity，不使用[startActivity()](http://developer.android.com/reference/android/content/Context.html#startActivity%28android.content.Intent%29)。然後調用[startActivities()](http://developer.android.com/reference/android/support/v4/app/TaskStackBuilder.html#startActivities%28%29)來啟動目標activity，或調用[getPendingIntent()](http://developer.android.com/reference/android/support/v4/app/TaskStackBuilder.html#getPendingIntent%28int,%20int%29)來創建相應的[PendingIntent](http://developer.android.com/reference/android/app/PendingIntent.html)。

例如，當用戶從通知進入你的app中的深層activity時，你可以使用這段代碼來創建一個啟動activity並把新後退棧插入目標任務的[PendingIntent](http://developer.android.com/reference/android/app/PendingIntent.html)。

```java
// 當用戶選擇通知時，啟動activity的intent
Intent detailsIntent = new Intent(this, DetailsActivity.class);

// 使用TaskStackBuilder創建後退棧，並獲取PendingIntent
PendingIntent pendingIntent =
        TaskStackBuilder.create(this)
                        // 添加所有DetailsActivity的父activity到棧中,
                        // 然後再添加DetailsActivity自己
                        .addNextIntentWithParentStack(upIntent)
                        .getPendingIntent(0, PendingIntent.FLAG_UPDATE_CURRENT);

NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
builder.setContentIntent(pendingIntent);
...
```

產生的[PendingIntent](http://developer.android.com/reference/android/app/PendingIntent.html)不僅指定了啟動哪個activity(被`detailsIntent`所定義)還指定了要插入任務(所有被`detailsIntent`定義的`DetailsActivity`)的後退棧。所以當`DetailsActivity`啟動時，點擊Back向後導航至每一個`DetailsActivity`類的父activity。

>**Note**:為了使[addNextIntentWithParentStack()](http://developer.android.com/reference/android/support/v4/app/TaskStackBuilder.html#addNextIntentWithParentStack%28android.content.Intent%29)方法起作用，像上面所說那樣，你必須在你的manifest文件中使用[android:parentActivityName](http://developer.android.com/guide/topics/manifest/activity-element.html#parent)(和相應的元素[`<meta-data>`](http://developer.android.com/guide/topics/manifest/meta-data-element.html))屬性聲明每個activity的邏輯父activity。

## 為Fragment實現向後導航

當在app中使用fragment時，個別的[FragmentTransaction](http://developer.android.com/reference/android/app/FragmentTransaction.html)對象可以代表要加入後退棧中變化的內容。例如，如果你要在手機上通過交換fragment實現一個[master/detail flow](http://developer.android.com/training/implementing-navigation/descendant.html#master-detail)(主/詳細流程)，你就要保證點擊Back按鈕可以從detail screen返回到master screen。要這麼做，你可以在提交事務(transaction)之前調用[addToBackStack()](http://developer.android.com/reference/android/app/FragmentTransaction.html#addToBackStack%28java.lang.String%29):

```java
// 使用framework FragmentManager
// 或support package FragmentManager (getSupportFragmentManager).
getSupportFragmentManager().beginTransaction()
                           .add(detailFragment, "detail")
                           // 提交這一事務到後退棧中
                           .addToBackStack()
                           .commit();
```

當後退棧中有[FragmentTransaction](http://developer.android.com/reference/android/app/FragmentTransaction.html)對象並且用戶點擊Back按鈕時,[FragmentManager](http://developer.android.com/reference/android/app/FragmentManager.html)會從後退棧中彈出最近的事務，然後執行反向操作(例如如果事務添加了一個fragment，那麼就刪除一個fragment)。

>**Note**:當事務用作水平導航(例如切換tab)或者修改內容外觀(例如在調整filter時)時，**不要將這個事務添加到後退棧中**。更多關於向後導航的恰當時間的信息，詳見[Navigation](http://developer.android.com/design/patterns/navigation.html) design guide。

如果你的應用更新了別的UI元素來反應當前的fragment狀態，例如action bar，記得當你提交事務時更新UI。除了在提交事務的時候，在後退棧發生變化時也要更新你的UI。你可以設置一個[FragmentManager.OnBackStackChangedListener](http://developer.android.com/reference/android/app/FragmentManager.OnBackStackChangedListener.html)來監聽[FragmentTransaction](http://developer.android.com/reference/android/app/FragmentTransaction.html)什麼時候復原:

```java
getSupportFragmentManager().addOnBackStackChangedListener(
        new FragmentManager.OnBackStackChangedListener() {
            public void onBackStackChanged() {
                // 在這裡更新你的UI
            }
        });
```

## 為WebView實現向後導航

如果你的應用的一部分包含在[WebView](http://developer.android.com/reference/android/webkit/WebView.html)中，可以通過瀏覽器歷史使用Back。要這麼做，如果[WebView](http://developer.android.com/reference/android/webkit/WebView.html)有歷史記錄，你可以重寫onBackPressed()並代理給[WebView](http://developer.android.com/reference/android/webkit/WebView.html):

```java
@Override
public void onBackPressed() {
    if (mWebView.canGoBack()) {
        mWebView.goBack();
        return;
    }

    // 否則遵從系統的默認操作.
    super.onBackPressed();
}
```

要注意當使用這一機制時，高動態化的頁面會產生大量歷史。會生成大量歷史的頁面，例如經常改變文件散列(document hash)的頁面,當要退出你的activity時，這會使你的用戶感到繁瑣。

更多關於使用[WebView](http://developer.android.com/reference/android/webkit/WebView.html)的信息，詳見[Building Web Apps in WebView](http://developer.android.com/guide/webapps/webview.html)。
