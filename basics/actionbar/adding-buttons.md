# 添加Action按鈕

> 編寫:[Vincent 4J](http://github.com/vincent4j) - 原文:<http://developer.android.com/training/basics/actionbar/adding-buttons.html>

Action bar 允許我們為當前環境下最重要的操作添加按鈕。那些直接出現在 action bar 中的 icon 和/或文本被稱作**action buttons(操作按鈕)**。安排不下的或不足夠重要的操作被隱藏在 **action overflow** （超出空間的action，譯者注）中。

![actionbar-actions](actionbar-actions.png)

圖 1. 一個有search操作按鈕和 action overflow 的 action bar，在 action overflow 裡能展現額外的操作。

## 在 XML 中指定操作

所有的操作按鈕和 action overflow 中其他可用的條目都被定義在 [menu資源](https://developer.android.com/guide/topics/resources/menu-resource.html) 的 XML 文件中。通過在項目的 `res/menu` 目錄中新增一個 XML 文件來為 action bar 添加操作。

為想要添加到 action bar 中的每個條目添加一個 `<item>` 元素。例如：

`res/menu/main_activity_actions.xml`

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android" >
    <!-- 搜索, 應該作為動作按鈕展示-->
    <item android:id="@+id/action_search"
          android:icon="@drawable/ic_action_search"
          android:title="@string/action_search"
          android:showAsAction="ifRoom" />
    <!-- 設置, 在溢出菜單中展示 -->
    <item android:id="@+id/action_settings"
          android:title="@string/action_settings"
          android:showAsAction="never" />
</menu>
```

上述代碼聲明，當 action bar 有可用空間時，搜索操作將作為一個操作按鈕來顯示，但設置操作將一直只在 action overflow 中顯示。（默認情況下，所有的操作都顯示在 action overflow 中，但為每一個操作指明設計意圖是很好的做法。）

icon 屬性要求每張圖片提供一個 `resource ID`。在 `@drawable/` 之後的名字必須是存儲在項目目錄 `res/drawable/` 下位圖圖片的文件名。例如：`ic_action_search.png` 對應 "@drawable/ic_action_search"。同樣地，title 屬性使用通過 XML 文件定義在項目目錄 `res/values/` 中的一個 `string 資源`，詳情請參見 [創建一個簡單的 UI](../firstapp/building-ui.html) 。

> **注意**：當創建 icon 和其他 bitmap 圖片時，要為不同屏幕密度下的顯示效果提供多個優化的版本，這一點很重要。在 [支持不同屏幕](../supporting-devices/screens.html) 課程中將會更詳細地討論。

**如果為了兼容 Android 2.1 的版本使用了 Support 庫**，在 `android` 命名空間下 `showAsAction` 屬性是不可用的。Support 庫會提供替代它的屬性，我們必須聲明自己的 XML 命名空間，並且使用該命名空間作為屬性前綴。（一個自定義 XML 命名空間需要以我們的 app 名稱為基礎，但是可以取任何想要的名稱，它的作用域僅僅在我們聲明的文件之內。）例如：

`res/menu/main_activity_actions.xml`

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:yourapp="http://schemas.android.com/apk/res-auto" >
    <!-- 搜索, 應該展示為動作按鈕 -->
    <item android:id="@+id/action_search"
          android:icon="@drawable/ic_action_search"
          android:title="@string/action_search"
          yourapp:showAsAction="ifRoom"  />
    ...
</menu>
```

## 為 Action Bar 添加操作

要為 action bar 佈局菜單條目，就要在 activity 中實現 <a href="https://developer.android.com/reference/android/app/Activity.html#onCreateOptionsMenu(android.view.Menu)">onCreateOptionsMenu()</a> 回調方法來 `inflate` 菜單資源從而獲取 [Menu](https://developer.android.com/reference/android/view/Menu.html) 對象。例如：

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    // 為ActionBar擴展菜單項
    MenuInflater inflater = getMenuInflater();
    inflater.inflate(R.menu.main_activity_actions, menu);
    return super.onCreateOptionsMenu(menu);
}
```

## 為操作按鈕添加響應事件

當用戶按下某一個操作按鈕或者 action overflow 中的其他條目，系統將調用 activity 中<a href="https://developer.android.com/reference/android/app/Activity.html#onOptionsItemSelected(android.view.MenuItem)">onOptionsItemSelected()</a>的回調方法。在該方法的實現裡面調用[MenuItem](https://developer.android.com/reference/android/view/MenuItem.html)的<a href="https://developer.android.com/reference/android/view/MenuItem.html#getItemId()">getItemId()</a>來判斷哪個條目被按下 - 返回的 ID 會匹配我們聲明對應的 `<item>` 元素中 `android:id` 屬性的值。

```java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    // 處理動作按鈕的點擊事件
    switch (item.getItemId()) {
        case R.id.action_search:
            openSearch();
            return true;
        case R.id.action_settings:
            openSettings();
            return true;
        default:
            return super.onOptionsItemSelected(item);
    }
}
```

## 為下級 Activity 添加向上按鈕

在不是程序入口的其他所有屏中（activity 不位於主屏時），需要在 action bar 中為用戶提供一個導航到邏輯父屏的**up button(向上按鈕)**。

![actionbar-up.png](actionbar-up.png)

圖 2. Gmail 中的 up button。

當運行在 Android 4.1(API level 16) 或更高版本，或者使用 Support 庫中的 [ActionBarActivity](https://developer.android.com/reference/android/support/v7/app/ActionBarActivity.html) 時，實現向上導航需要在 manifest 文件中聲明父 activity ，同時在 action bar 中設置向上按鈕可用。

如何在 manifest 中聲明一個 activity 的父類，例如：

```xml
<application ... >
    ...
    <!-- 主 main/home 活動 (沒有上級活動) -->
    <activity
        android:name="com.example.myfirstapp.MainActivity" ...>
        ...
    </activity>
    <!-- 主活動的一個子活動-->
    <activity
        android:name="com.example.myfirstapp.DisplayMessageActivity"
        android:label="@string/title_activity_display_message"
        android:parentActivityName="com.example.myfirstapp.MainActivity" >
        <!--  meta-data 用於支持 support 4.0 以及以下來指明上級活動 -->
        <meta-data
            android:name="android.support.PARENT_ACTIVITY"
            android:value="com.example.myfirstapp.MainActivity" />
    </activity>
</application>
```

然後，通過調用<a href="https://developer.android.com/reference/android/app/ActionBar.html#setDisplayHomeAsUpEnabled(boolean)">setDisplayHomeAsUpEnabled()</a> 來把 app icon 設置成可用的向上按鈕：

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_displaymessage);

    getSupportActionBar().setDisplayHomeAsUpEnabled(true);
    // 如果你的minSdkVersion屬性是11活更高, 應該這麼用:
    // getActionBar().setDisplayHomeAsUpEnabled(true);
}
```

由於系統已經知道 `MainActivity` 是 `DisplayMessageActivity` 的父 activity，當用戶按下向上按鈕時，系統會導航到恰當的父 activity - 你不需要去處理向上按鈕的事件。

更多關於向上導航的信息，請見 [提供向上導航](../../ux/implement-nav/ancestral.html)。

