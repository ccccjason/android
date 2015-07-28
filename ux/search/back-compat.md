# 保持向下兼容

> 編寫:[Lin-H](https://github.com/Lin-H) - 原文:<http://developer.android.com/training/search/backward-compat.html>

[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)和action bar只在Android 3.0以及以上版本可用。為了支持舊版本平臺，你可以回到搜索對話框。搜索框是系統提供的UI，在調用時會覆蓋在你的應用的最頂端。

##設置最小和目標API級別

要設置搜索對話框，首先在你的manifest中聲明你要支持舊版本設備，並且目標平臺為Android 3.0或更新版本。當你這麼做之後，你的應用會自動地在Android 3.0或以上使用action bar，在舊版本的設備使用傳統的目錄系統:

```xml
<uses-sdk android:minSdkVersion="7" android:targetSdkVersion="15" />

<application>
...
```

##為舊版本設備提供搜索對話框

要在舊版本設備中調用搜索對話框，可以在任何時候，當用戶從選項目錄中選擇搜索項時，調用[onSearchRequested()](reference/android/app/Activity.html#onSearchRequested())。因為Android 3.0或以上會在action bar中顯示[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)(就像在第一節課中演示的那樣)，所以當用戶選擇目錄的搜索項時，只有Android 3.0以下版本的會調用[onOptionsItemSelected()](http://developer.android.com/reference/android/app/Activity.html#onOptionsItemSelected(android.view.MenuItem))。

```java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {
        case R.id.search:
            onSearchRequested();
            return true;
        default:
            return false;
    }
}
```

##在運行時檢查Android的構建版本

在運行時，檢查設備的版本可以保證在舊版本設備中，不使用不支持的[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)。在我們這個例子中，這一操作在[onCreateOptionsMenu()](http://developer.android.com/reference/android/app/Activity.html#onCreateOptionsMenu(android.view.Menu))方法中:

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {

    MenuInflater inflater = getMenuInflater();
    inflater.inflate(R.menu.options_menu, menu);

    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
        SearchManager searchManager =
                (SearchManager) getSystemService(Context.SEARCH_SERVICE);
        SearchView searchView =
                (SearchView) menu.findItem(R.id.search).getActionView();
        searchView.setSearchableInfo(
                searchManager.getSearchableInfo(getComponentName()));
        searchView.setIconifiedByDefault(false);
    }
    return true;
}
```
