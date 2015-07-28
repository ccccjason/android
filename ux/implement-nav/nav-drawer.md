# 創建抽屜式導航(navigation drawer)

> 編寫:[Lin-H](https://github.com/Lin-H) - 原文: <http://developer.android.com/training/implementing-navigation/nav-drawer.html>

Navigation drawer是一個在屏幕左側邊緣顯示導航選項的面板。大部分時候是隱藏的，當用戶從屏幕左側劃屏，或在top level模式的app中點擊action bar中的app圖標時，才會顯示。

這節課敘述如何使用[Support Library](http://developer.android.com/tools/support-library/index.html)中的[DrawerLayout](http://developer.android.com/reference/android/support/v4/widget/DrawerLayout.html) API，來實現navigation drawer。

> **Navigation Drawer 設計**：在你決定在你的app中使用Navigation Drawer之前，你應該先理解在[Navigation Drawer](http://developer.android.com/design/patterns/navigation-drawer.html) design guide中定義的使用情況和設計準則。

## 創建一個Drawer Layout

要添加一個navigation drawer，在你的用戶界面layout中聲明一個用作root view(根視圖)的[DrawerLayout](http://developer.android.com/reference/android/support/v4/widget/DrawerLayout.html)對象。在[DrawerLayout](http://developer.android.com/reference/android/support/v4/widget/DrawerLayout.html)中為屏幕添加一個包含主要內容的view(當drawer隱藏時的主layout)，和其他一些包含navigation drawer內容的view。

例如，下面的layout使用了有兩個子視圖(child view)的[DrawerLayout](http://developer.android.com/reference/android/support/v4/widget/DrawerLayout.html):一個[FrameLayout](http://developer.android.com/reference/android/widget/FrameLayout.html)用來包含主要內容(在運行時被[Fragment](http://developer.android.com/reference/android/app/Fragment.html)填入)，和一個navigation drawer使用的[ListView](http://developer.android.com/reference/android/widget/ListView.html)。

```xml
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <!-- 包含主要內容的 view -->
    <FrameLayout
        android:id="@+id/content_frame"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
    <!-- navigation drawer(抽屜式導航) -->
    <ListView android:id="@+id/left_drawer"
        android:layout_width="240dp"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:choiceMode="singleChoice"
        android:divider="@android:color/transparent"
        android:dividerHeight="0dp"
        android:background="#111"/>
</android.support.v4.widget.DrawerLayout>
```

這個layout展示了一些layout的重要特點:

* 主內容view(上面的[FrameLayout](http://developer.android.com/reference/android/widget/FrameLayout.html))，在[DrawerLayout](http://developer.android.com/reference/android/support/v4/widget/DrawerLayout.html)中**必須是第一個子視圖**，因為XML的順序代表著Z軸(垂直於手機屏幕)的順序，並且drawer必須在內容的前端。

* 主內容view被設置為匹配父視圖的寬和高，因為當navigation drawer隱藏時，主內容表示整個UI部分。

* drawer視圖([ListView](http://developer.android.com/reference/android/widget/ListView.html))必須使用`android:layout_gravity`屬性**指定它的horizontal gravity**。為了支持從右邊閱讀的語言(right-to-left(RTL) language)，指定它的值為`"start"`而不是`"left"`(當layout是RTL時drawer在右邊顯示)。

* drawer視圖以`dp`為單位指定它的寬和高來匹配父視圖。drawer的寬度不能大於320dp，這樣用戶總能看到部分主內容。

## 初始化Drawer List

在你的activity中，首先要做的事就是要初始化drawer的item列表。這要根據你的app內容來處理，但是一個navigation drawer通常由一個[ListView](http://developer.android.com/reference/android/widget/ListView.html)組成，所以列表應該通過一個[Adapter](http://developer.android.com/reference/android/widget/Adapter.html)(例如[ArrayAdapter](http://developer.android.com/reference/android/widget/ArrayAdapter.html)或[SimpleCursorAdapter](http://developer.android.com/reference/android/widget/SimpleCursorAdapter.html))填入。

例如，如何使用一個字符串數組([string array](http://developer.android.com/guide/topics/resources/string-resource.html#StringArray))來初始化導航列表(navigation list):

```java
public class MainActivity extends Activity {
    private String[] mPlanetTitles;
    private DrawerLayout mDrawerLayout;
    private ListView mDrawerList;
    ...

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mPlanetTitles = getResources().getStringArray(R.array.planets_array);
        mDrawerLayout = (DrawerLayout) findViewById(R.id.drawer_layout);
        mDrawerList = (ListView) findViewById(R.id.left_drawer);

        // 為list view設置adapter
        mDrawerList.setAdapter(new ArrayAdapter<String>(this,
                R.layout.drawer_list_item, mPlanetTitles));
        // 為list設置click listener
        mDrawerList.setOnItemClickListener(new DrawerItemClickListener());

        ...
    }
}
```

這段代碼也調用了[setOnItemClickListener()](http://developer.android.com/reference/android/widget/AdapterView.html#setOnItemClickListener%28android.widget.AdapterView.OnItemClickListener%29)來接收navigation drawer列表的點擊事件。下一節會說明如何實現這個接口，並且當用戶選擇一個item時如何改變內容視圖(content view)。

## 處理導航的點擊事件

當用戶選擇drawer列表中的item，系統會調用在[setOnItemClickListener()](http://developer.android.com/reference/android/widget/AdapterView.html#setOnItemClickListener%28android.widget.AdapterView.OnItemClickListener%29)中所設置的[OnItemClickListener](http://developer.android.com/reference/android/widget/AdapterView.OnItemClickListener.html)的[onItemClick()](http://developer.android.com/reference/android/widget/AdapterView.OnItemClickListener.html#onItemClick%28android.widget.AdapterView%3C?%3E,%20android.view.View,%20int,%20long%29)。

在[onItemClick()](http://developer.android.com/reference/android/widget/AdapterView.OnItemClickListener.html#onItemClick%28android.widget.AdapterView%3C?%3E,%20android.view.View,%20int,%20long%29)方法中做什麼，取決於你如何實現你的app結構([app structure](http://developer.android.com/design/patterns/app-structure.html))。在下面的例子中，每選擇一個列表中的item，就插入一個不同的[Fragment](http://developer.android.com/reference/android/app/Fragment.html)到主內容視圖中([FrameLayout](http://developer.android.com/reference/android/widget/FrameLayout.html)元素通過`R.id.content_frame` ID辨識):

```java
private class DrawerItemClickListener implements ListView.OnItemClickListener {
    @Override
    public void onItemClick(AdapterView parent, View view, int position, long id) {
        selectItem(position);
    }
}

/** 在主內容視圖中交換fragment */
private void selectItem(int position) {
    // 創建一個新的fragment並且根據行星的位置來顯示
    Fragment fragment = new PlanetFragment();
    Bundle args = new Bundle();
    args.putInt(PlanetFragment.ARG_PLANET_NUMBER, position);
    fragment.setArguments(args);

    // 通過替換已存在的fragment來插入新的fragment
    FragmentManager fragmentManager = getFragmentManager();
    fragmentManager.beginTransaction()
                   .replace(R.id.content_frame, fragment)
                   .commit();

    // 高亮被選擇的item, 更新標題, 並關閉drawer
    mDrawerList.setItemChecked(position, true);
    setTitle(mPlanetTitles[position]);
    mDrawerLayout.closeDrawer(mDrawerList);
}

@Override
public void setTitle(CharSequence title) {
    mTitle = title;
    getActionBar().setTitle(mTitle);
}

```

## 監聽打開和關閉事件

要監聽drawer的打開和關閉事件，在你的[DrawerLayout](http://developer.android.com/reference/android/support/v4/widget/DrawerLayout.html)中調用[setDrawerListener()](http://developer.android.com/reference/android/support/v4/widget/DrawerLayout.html#setDrawerListener%28android.support.v4.widget.DrawerLayout.DrawerListener%29)，並傳入一個[DrawerLayout.DrawerListener](http://developer.android.com/reference/android/support/v4/widget/DrawerLayout.DrawerListener.html)的實現。這個接口提供drawer事件的回調例如[onDrawerOpened()](http://developer.android.com/reference/android/support/v4/widget/DrawerLayout.DrawerListener.html#onDrawerOpened%28android.view.View%29)和[onDrawerClosed()](http://developer.android.com/reference/android/support/v4/widget/DrawerLayout.DrawerListener.html#onDrawerClosed%28android.view.View%29)。

但是，如果你的activity包含有[action bar](http://developer.android.com/guide/topics/ui/actionbar.html)可以不用實現[DrawerLayout.DrawerListener](http://developer.android.com/reference/android/support/v4/widget/DrawerLayout.DrawerListener.html)，你可以繼承[ActionBarDrawerToggle](http://developer.android.com/reference/android/support/v4/app/ActionBarDrawerToggle.html)來替代。[ActionBarDrawerToggle](http://developer.android.com/reference/android/support/v4/app/ActionBarDrawerToggle.html)實現了[DrawerLayout.DrawerListener](http://developer.android.com/reference/android/support/v4/widget/DrawerLayout.DrawerListener.html)，所以你仍然可以重寫這些回調。這麼做也能使action bar圖標和 navigation drawer的交互操作變得更容易(在下節詳述)。

如[Navigation Drawer](http://developer.android.com/design/patterns/navigation-drawer.html) design guide中所述,當drawer可見時，你應該修改action bar的內容，比如改變標題和移除與主文字內容相關的action item。下面的代碼向你說明如何通過[ActionBarDrawerToggle](http://developer.android.com/reference/android/support/v4/app/ActionBarDrawerToggle.html)類的實例，重寫[DrawerLayout.DrawerListener](http://developer.android.com/reference/android/support/v4/widget/DrawerLayout.DrawerListener.html)的回調方法來實現這個目的:

```java
public class MainActivity extends Activity {
    private DrawerLayout mDrawerLayout;
    private ActionBarDrawerToggle mDrawerToggle;
    private CharSequence mDrawerTitle;
    private CharSequence mTitle;
    ...

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ...

        mTitle = mDrawerTitle = getTitle();
        mDrawerLayout = (DrawerLayout) findViewById(R.id.drawer_layout);
        mDrawerToggle = new ActionBarDrawerToggle(this, mDrawerLayout,
                R.drawable.ic_drawer, R.string.drawer_open, R.string.drawer_close) {

            /** 當drawer處於完全關閉的狀態時調用 */
            public void onDrawerClosed(View view) {
                super.onDrawerClosed(view);
                getActionBar().setTitle(mTitle);
                invalidateOptionsMenu(); // 創建對onPrepareOptionsMenu()的調用
            }

            /** 當drawer處於完全打開的狀態時調用 */
            public void onDrawerOpened(View drawerView) {
                super.onDrawerOpened(drawerView);
                getActionBar().setTitle(mDrawerTitle);
                invalidateOptionsMenu(); // 創建對onPrepareOptionsMenu()的調用
            }
        };

        // 設置drawer觸發器為DrawerListener
        mDrawerLayout.setDrawerListener(mDrawerToggle);
    }

    /* 當invalidateOptionsMenu()調用時調用 */
    @Override
    public boolean onPrepareOptionsMenu(Menu menu) {
        // 如果nav drawer是打開的, 隱藏與內容視圖相關聯的action items
        boolean drawerOpen = mDrawerLayout.isDrawerOpen(mDrawerList);
        menu.findItem(R.id.action_websearch).setVisible(!drawerOpen);
        return super.onPrepareOptionsMenu(menu);
    }
}
```

下一節會描述[ActionBarDrawerToggle](http://developer.android.com/reference/android/support/v4/app/ActionBarDrawerToggle.html)的構造參數，和處理與action bar圖標交互所需的其他步驟。

## 使用App圖標來打開和關閉

用戶可以在屏幕左側使用劃屏手勢來打開和關閉navigation drawer，但是如果你使用[action bar](http://developer.android.com/guide/topics/ui/actionbar.html),你也應該允許用戶通過點擊app圖標來打開或關閉。並且app圖標也應該使用一個特殊的圖標來指明navigation drawer的存在。你可以通過使用上一節所說的[ActionBarDrawerToggle](http://developer.android.com/reference/android/support/v4/app/ActionBarDrawerToggle.html)來實現所有的這些操作。

要使[ActionBarDrawerToggle](http://developer.android.com/reference/android/support/v4/app/ActionBarDrawerToggle.html)起作用，通過它的構造函數創建一個實例，需要用到以下參數:

* [Activity](http://developer.android.com/reference/android/app/Activity.html)用來容納drawer。

* [DrawerLayout](http://developer.android.com/reference/android/support/v4/widget/DrawerLayout.html)。

* 一個drawable資源用作drawer指示器。
  標準的navigation drawer可以在[Download the Action Bar Icon Pack](http://developer.android.com/downloads/design/Android_Design_Icons_20130926.zip)獲的

* 一個字符串資源描述"打開抽屜"操作(便於訪問)

* 一個字符串資源描述"關閉抽屜"操作(便於訪問)

那麼，不論你是否創建了用作drawer監聽器的[ActionBarDrawerToggle](http://developer.android.com/reference/android/support/v4/app/ActionBarDrawerToggle.html)的子類，你都需要在activity生命週期中的某些地方根據你的[ActionBarDrawerToggle](http://developer.android.com/reference/android/support/v4/app/ActionBarDrawerToggle.html)來調用。

```java
public class MainActivity extends Activity {
    private DrawerLayout mDrawerLayout;
    private ActionBarDrawerToggle mDrawerToggle;
    ...

    public void onCreate(Bundle savedInstanceState) {
        ...

        mDrawerLayout = (DrawerLayout) findViewById(R.id.drawer_layout);
        mDrawerToggle = new ActionBarDrawerToggle(
                this,                  /* 承載 Activity */
                mDrawerLayout,         /* DrawerLayout 對象 */
                R.drawable.ic_drawer,  /* nav drawer 圖標用來替換'Up'符號 */
                R.string.drawer_open,  /* "打開 drawer" 描述 */
                R.string.drawer_close  /* "關閉 drawer" 描述 */
                ) {

            /** 當drawer處於完全關閉的狀態時調用 */
            public void onDrawerClosed(View view) {
                super.onDrawerClosed(view);
                getActionBar().setTitle(mTitle);
            }

            /** 當drawer處於完全打開的狀態時調用 */
            public void onDrawerOpened(View drawerView) {
                super.onDrawerOpened(drawerView);
                getActionBar().setTitle(mDrawerTitle);
            }
        };

        // 設置drawer觸發器為DrawerListener
        mDrawerLayout.setDrawerListener(mDrawerToggle);

        getActionBar().setDisplayHomeAsUpEnabled(true);
        getActionBar().setHomeButtonEnabled(true);
    }

    @Override
    protected void onPostCreate(Bundle savedInstanceState) {
        super.onPostCreate(savedInstanceState);
        // 在onRestoreInstanceState發生後，同步觸發器狀態.
        mDrawerToggle.syncState();
    }

    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
        mDrawerToggle.onConfigurationChanged(newConfig);
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // 將事件傳遞給ActionBarDrawerToggle, 如果返回true，表示app 圖標點擊事件已經被處理
        if (mDrawerToggle.onOptionsItemSelected(item)) {
          return true;
        }
        // 處理你的其他action bar items...

        return super.onOptionsItemSelected(item);
    }

    ...
}
```

一個完整的navigation drawer例子,可以在原文頁面頂端的sample下載
