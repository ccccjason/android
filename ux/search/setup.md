# 建立搜索界面

> 編寫:[Lin-H](https://github.com/Lin-H) - 原文:<http://developer.android.com/training/search/setup.html>

從Android 3.0開始，在action bar中使用[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)作為item，是在你的app中提供搜索的一種更好方法。像其他所有在action bar中的item一樣，你可以定義[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)在有足夠空間的時候總是顯示，或設置為一個摺疊操作(collapsible action),一開始[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)作為一個圖標顯示，當用戶點擊圖標時再顯示搜索框佔據整個action bar。

>**Note**:在本課程的後面，你會學習對那些不支持[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)的設備，如何使你的app向下兼容至Android 2.1(API level 7)版本。

##添加Search View到action bar中

為了在action bar中添加[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)，在你的工程目錄`res/menu/`中創建一個名為`options_menu.xml`的文件，再把下列代碼添加到文件中。這段代碼定義瞭如何創建search item，比如使用的圖標和item的標題。`collapseActionView`屬性允許你的[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)佔據整個action bar，在不使用的時候摺疊成普通的action bar item。由於在手持設備中action bar的空間有限，建議使用`collapsibleActionView`屬性來提供更好的用戶體驗。

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/search"
          android:title="@string/search_title"
          android:icon="@drawable/ic_search"
          android:showAsAction="collapseActionView|ifRoom"
          android:actionViewClass="android.widget.SearchView" />
</menu>
```

>**Note**:如果你的menu items已經有一個XML文件，你可以只把`<item>`元素添加入文件。

要在action bar中顯示[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)，在你的activity中[onCreateOptionsMenu()](http://developer.android.com/reference/android/app/Activity.html#onCreateOptionsMenu(android.view.Menu))方法內填充XML菜單資源(`res/menu/options_menu.xml`):

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    MenuInflater inflater = getMenuInflater();
    inflater.inflate(R.menu.options_menu, menu);

    return true;
}
```

如果你立即運行你的app，[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)就會顯示在你app的action bar中，但還無法使用。你現在需要定義[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)如何運行。

##創建一個檢索配置

[檢索配置(searchable configuration)](http://developer.android.com/guide/topics/search/searchable-config.html)在 `res/xml/searchable.xml`文件中定義了[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)如何運行。檢索配置中至少要包含一個`android:label`屬性，與Android manifest中的`<application>`或`<activity>` `android:label`屬性值相同。但我們還是建議添加`android:hint`屬性來告訴用戶應該在搜索框中輸入什麼內容:

```xml
<?xml version="1.0" encoding="utf-8"?>

<searchable xmlns:android="http://schemas.android.com/apk/res/android"
        android:label="@string/app_name"
        android:hint="@string/search_hint" />
```

在你的應用的manifest文件中，聲明一個指向`res/xml/searchable.xml`文件的[`<meta-data>`](http://developer.android.com/guide/topics/manifest/meta-data-element.html)元素，來告訴你的應用在哪裡能找到檢索配置。在你想要顯示[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)的`<activity>`中聲明`<meta-data>`元素:

```xml
<activity ... >
    ...
    <meta-data android:name="android.app.searchable"
            android:resource="@xml/searchable" />

</activity>
```

在你之前創建的[onCreateOptionsMenu()](http://developer.android.com/reference/android/app/Activity.html#onCreateOptionsMenu(android.view.Menu))方法中，調用[setSearchableInfo(SearchableInfo)](http://developer.android.com/reference/android/widget/SearchView.html#setSearchableInfo(android.app.SearchableInfo))把[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)和檢索配置關聯在一起:

```xml
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    MenuInflater inflater = getMenuInflater();
    inflater.inflate(R.menu.options_menu, menu);

    // 關聯檢索配置和SearchView
    SearchManager searchManager =
           (SearchManager) getSystemService(Context.SEARCH_SERVICE);
    SearchView searchView =
            (SearchView) menu.findItem(R.id.search).getActionView();
    searchView.setSearchableInfo(
            searchManager.getSearchableInfo(getComponentName()));

    return true;
}
```

調用[getSearchableInfo()](http://developer.android.com/reference/android/app/SearchManager.html#getSearchableInfo(android.content.ComponentName))返回一個[SearchableInfo](http://developer.android.com/reference/android/app/SearchableInfo.html)由檢索配置XML文件創建的對象。檢索配置與[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)正確關聯後，當用戶提交一個搜索請求時，[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)會以[ACTION_SEARCH](http://developer.android.com/reference/android/content/Intent.html#ACTION_SEARCH) intent啟動一個activity。所以你現在需要一個能過濾這個intent和處理搜索請求的activity。

##創建一個檢索activity

當用戶提交一個搜索請求時，[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)會嘗試以[ACTION_SEARCH](http://developer.android.com/reference/android/content/Intent.html#ACTION_SEARCH)啟動一個activity。檢索activity會過濾[ACTION_SEARCH](http://developer.android.com/reference/android/content/Intent.html#ACTION_SEARCH) intent並在某種數據集中根據請求進行搜索。要創建一個檢索activity，在你選擇的activity中聲明對[ACTION_SEARCH](http://developer.android.com/reference/android/content/Intent.html#ACTION_SEARCH) intent過濾:

```xml
<activity android:name=".SearchResultsActivity" ... >
    ...
    <intent-filter>
        <action android:name="android.intent.action.SEARCH" />
    </intent-filter>
    ...
</activity>
```

在你的檢索activity中，通過在[onCreate()](http://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle))方法中檢查[ACTION_SEARCH](http://developer.android.com/reference/android/content/Intent.html#ACTION_SEARCH) intent來處理它。

>**Note**:如果你的檢索activity在single top mode下啟動(`android:launchMode="singleTop"`)，也要在[onNewIntent()](http://developer.android.com/reference/android/app/Activity.html#onNewIntent(android.content.Intent))方法中處理[ACTION_SEARCH](http://developer.android.com/reference/android/content/Intent.html#ACTION_SEARCH) intent。在single top mode下你的activity只有一個會被創建，而隨後啟動的activity將不會在棧中創建新的activity。這種啟動模式很有用，因為用戶可以在當前activity中進行搜索，而不用在每次搜索時都創建一個activity實例。

```java
public class SearchResultsActivity extends Activity {

    @Override
    public void onCreate(Bundle savedInstanceState) {
        ...
        handleIntent(getIntent());
    }

    @Override
    protected void onNewIntent(Intent intent) {
        ...
        handleIntent(intent);
    }

    private void handleIntent(Intent intent) {

        if (Intent.ACTION_SEARCH.equals(intent.getAction())) {
            String query = intent.getStringExtra(SearchManager.QUERY);
            //通過某種方法，根據請求檢索你的數據
        }
    }
    ...
}
```

如果你現在運行你的app，[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)就能接收用戶的搜索請求，以[ACTION_SEARCH](http://developer.android.com/reference/android/content/Intent.html#ACTION_SEARCH) intent啟動你的檢索activity。現在就由你來解決如何依據請求來儲存和搜索數據。
