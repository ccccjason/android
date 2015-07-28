# 提供向上的導航

> 編寫:[Lin-H](https://github.com/Lin-H) - 原文:<http://developer.android.com/training/implementing-navigation/ancestral.html>

所有不是從主屏幕("home"屏幕)進入app的，都應該給用戶提供一種方法，通過點擊[action bar](http://developer.android.com/guide/topics/ui/actionbar.html)中的Up按鈕。可以回到app的結構層次中邏輯父屏幕。本課程向你說明如何正確地實現這一操作。

>**Up Navigation 設計**

>[Designing Effective Navigation](http://developer.android.com/training/design-navigation/ancestral-temporal.html)和the [Navigation](http://developer.android.com/training/design-navigation/ancestral-temporal.html) design guide中描述了向上導航的概念和設計準則。

![Figure 1. action bar中的Up按鈕.](implementing-navigation-up.png)

**Figure 1**. action bar中的Up按鈕.

## 指定父Activity

要實現向上導航，第一步就是為每一個activity聲明合適的父activity。這麼做可以使系統簡化導航模式，例如向上導航，因為系統可以從manifest文件中判斷它的邏輯父(logical parent)activity。

從Android 4.1 (API level 16)開始，你可以通過指定[`<activity>`](http://developer.android.com/guide/topics/manifest/activity-element.html)元素中的[android:parentActivityName](http://developer.android.com/guide/topics/manifest/activity-element.html#parent)屬性來聲明每一個activity的邏輯父activity。

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
        <!-- 父activity的meta-data，用來支持4.0以下版本 -->
        <meta-data
            android:name="android.support.PARENT_ACTIVITY"
            android:value="com.example.myfirstapp.MainActivity" />
    </activity>
</application>
```

在父activity這樣聲明後，你可以使用[NavUtils](http://developer.android.com/reference/android/support/v4/app/NavUtils.html) API進行向上導航操作，就像下一面這節。

## 添加向上操作(Up Action)

要使用action bar的app圖標來完成向上導航，需要調用[setDisplayHomeAsUpEnabled()](http://developer.android.com/reference/android/app/ActionBar.html#setDisplayHomeAsUpEnabled%28boolean%29):

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    ...
    getActionBar().setDisplayHomeAsUpEnabled(true);
}
```

這樣，在app旁添加了一個左向符號，並用作操作按鈕。當用戶點擊它時，你的activity會接收一個對[onOptionsItemSelected()](http://developer.android.com/reference/android/app/Activity.html#onOptionsItemSelected%28android.view.MenuItem%29)的調用。操作的ID是`android.R.id.home`。

## 向上導航至父activity

要在用戶點擊app圖標時向上導航，你可以使用[NavUtils](http://developer.android.com/reference/android/support/v4/app/NavUtils.html)類中的靜態方法[navigateUpFromSameTask()](http://developer.android.com/reference/android/support/v4/app/NavUtils.html#navigateUpFromSameTask%28android.app.Activity%29)。當你調用這一方法時，系統會結束當前的activity並啟動(或恢復)相應的父activity。如果目標activity在任務的後退棧中(back stack)，則目標activity會像[FLAG_ACTIVITY_CLEAR_TOP](http://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_CLEAR_TOP)定義的那樣，提到棧頂。提到棧頂的方式取決於父activity是否處理了對<a href="http://developer.android.com/reference/android/app/Activity.html#onNewIntent(android.content.Intent)">onNewIntent()</a>的調用。

例如:

```java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {
    // 對action bar的Up/Home按鈕做出反應
    case android.R.id.home:
        NavUtils.navigateUpFromSameTask(this);
        return true;
    }
    return super.onOptionsItemSelected(item);
}
```

但是，**只能是當你的app擁有當前任務(current task)**(用戶從你的app中發起這一任務)時[navigateUpFromSameTask()](http://developer.android.com/reference/android/support/v4/app/NavUtils.html#navigateUpFromSameTask%28android.app.Activity%29)才有用。如果你的activity是從別的app的任務中啟動的話，向上導航操作就應該創建一個屬於你的app的新任務，並需要你創建一個新的後退棧。

### 用新的後退棧來向上導航

如果你的activity提供了任何允許被別的app啟動的[intent filters](http://developer.android.com/guide/components/intents-filters.html#ifs)，那麼你應該實現[onOptionsItemSelected()](http://developer.android.com/reference/android/app/Activity.html#onOptionsItemSelected%28android.view.MenuItem%29)回調，在用戶從別的app任務進入你的activity後，點擊Up按鈕，在向上導航之前你的app用相應的後退棧開啟一個新的任務。

在這麼做之前，你可以先調用[shouldUpRecreateTask()](http://developer.android.com/reference/android/support/v4/app/NavUtils.html#shouldUpRecreateTask%28android.app.Activity,%20android.content.Intent%29)來檢查當前的activity實例是否在另一個不同的app任務中。如果返回true，就使用[TaskStackBuilder](http://developer.android.com/reference/android/support/v4/app/TaskStackBuilder.html)創建一個新任務。或者，你可以向上面那樣使用[navigateUpFromSameTask()](http://developer.android.com/reference/android/support/v4/app/NavUtils.html#navigateUpFromSameTask%28android.app.Activity%29)方法。

例如:

```java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {
    // 對action bar的Up/Home按鈕做出反應
    case android.R.id.home:
        Intent upIntent = NavUtils.getParentActivityIntent(this);
        if (NavUtils.shouldUpRecreateTask(this, upIntent)) {
            // 這個activity不是這個app任務的一部分, 所以當向上導航時創建
            // 用合成後退棧(synthesized back stack)創建一個新任務。
            TaskStackBuilder.create(this)
                    // 添加這個activity的所有父activity到後退棧中
                    .addNextIntentWithParentStack(upIntent)
                    // 向上導航到最近的一個父activity
                    .startActivities();
        } else {
            // 這個activity是這個app任務的一部分, 所以
            // 向上導航至邏輯父activity.
            NavUtils.navigateUpTo(this, upIntent);
        }
        return true;
    }
    return super.onOptionsItemSelected(item);
}
```

>**Note**:為了能使[addNextIntentWithParentStack()](http://developer.android.com/reference/android/support/v4/app/TaskStackBuilder.html#addNextIntentWithParentStack%28android.content.Intent%29)方法起作用，你必須像上面說的那樣，在你的manifest文件中使用[android:parentActivityName](http://developer.android.com/guide/topics/manifest/activity-element.html#parent)(和相應的[`<meta-data>`](http://developer.android.com/guide/topics/manifest/meta-data-element.html)元素)屬性聲明所有的activity的邏輯父activity。
