<!--# Searching within TV Apps #-->
# TV應用內搜索

> 編寫:[awong1900](https://github.com/awong1900) - 原文:http://developer.android.com/training/tv/discovery/in-app-search.html

<!--Users frequently have specific content in mind when using a media app on TV. If your app contains a large catalog of content, browsing for a specific title may not be the most efficient way for users to find what they are looking for. A search interface can help your users get to the content they want faster than browsing.-->

當在TV上用媒體應用時，用戶腦中通常有期望的內容。如果我們的應用包含一個大的內容目錄，為用戶找到他們想找到的內容時，用特定的標題瀏覽可能不是最有效的方式。一個搜索界面能幫助用戶獲得他們想快速瀏覽的內容。

<!--The Leanback support library provides a set of classes to enable a standard search interface within your app that is consistent with other search functions on TV and provides features such as voice input.-->

[Leanback support library](http://developer.android.com/tools/support-library/features.html#v17-leanback)提供一套類庫去使用標準的搜索界面。在我們的應用內使用類庫，可以和TV其他搜索功能，如語音搜索，獲得一致性。

<!--This lesson discusses how to provide a search interface in your app using Leanback support library classes.-->

這節課討論如何在我們的應用中用Leanback支持類庫提供搜索界面。

<!--## Add a Search Action ##-->
## 添加搜索操作

<!--When you use the BrowseFragment class for a media browsing interface, you can enable a search interface as a standard part of the user interface. The search interface is an icon that appears in the layout when you set View.OnClickListener on the BrowseFragment object. The following sample code demonstrates this technique.-->

當我們用[BroweseFragment](http://developer.android.com/reference/android/support/v17/leanback/app/BrowseFragment.html)類做一個媒體瀏覽界面時，我們能使用搜索界面作為用戶界面的一個標準部分。當我們設置[View.OnClickListener](http://developer.android.com/reference/android/view/View.OnClickListener.html)在[BrowseFragment](http://developer.android.com/reference/android/support/v17/leanback/app/BrowseFragment.html)對象時，搜索界面作為一個圖標出現在佈局中。接下來的示例代碼展示了這個技術。

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.browse_activity);

    mBrowseFragment = (BrowseFragment)
            getFragmentManager().findFragmentById(R.id.browse_fragment);

    ...

    mBrowseFragment.setOnSearchClickedListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            Intent intent = new Intent(BrowseActivity.this, SearchActivity.class);
            startActivity(intent);
        }
    });

    mBrowseFragment.setAdapter(buildAdapter());
}
```

<!-->**Note**: You can set the color of the search icon using the setSearchAffordanceColor(int).-->

>**Note**：我們能設置搜索圖標的顏色用[setSearchAffordanceColor(int)](http://developer.android.com/reference/android/support/v17/leanback/app/BrowseFragment.html#setSearchAffordanceColor(int))。


<!--## Add a Search Input and Results-->
## 添加搜索輸入和結果展示

<!--When a user selects the search icon, the system invokes a search activity via the defined intent. Your search activity should use a linear layout containing a SearchFragment. This fragment must also implement the SearchFragment.SearchResultProvider interface in order to display the results of a search.-->

當用戶選擇搜索圖標，系統通過定義的intent關聯一個搜索activity。我們的搜索activity應該用包括[SearchFragment](http://developer.android.com/reference/android/support/v17/leanback/app/SearchFragment.html)的線性佈局。這個fragment必須實現[SearchFragment.SearchResultProvider](http://developer.android.com/reference/android/support/v17/leanback/app/SearchFragment.SearchResultProvider.html)界面去顯示搜索結果。

<!--The following code sample shows how to extend the SearchFragment class to provide a search interface and results:-->

接下來的示例代碼展示瞭如何擴展[SearchFragment](http://developer.android.com/reference/android/support/v17/leanback/app/SearchFragment.html)類去提供搜索界面和結果：

```java
public class MySearchFragment extends SearchFragment
        implements SearchFragment.SearchResultProvider {

    private static final int SEARCH_DELAY_MS = 300;
    private ArrayObjectAdapter mRowsAdapter;
    private Handler mHandler = new Handler();
    private SearchRunnable mDelayedLoad;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mRowsAdapter = new ArrayObjectAdapter(new ListRowPresenter());
        setSearchResultProvider(this);
        setOnItemClickedListener(getDefaultItemClickedListener());
        mDelayedLoad = new SearchRunnable();
    }

    @Override
    public ObjectAdapter getResultsAdapter() {
        return mRowsAdapter;
    }

    @Override
    public boolean onQueryTextChange(String newQuery) {
        mRowsAdapter.clear();
        if (!TextUtils.isEmpty(newQuery)) {
            mDelayedLoad.setSearchQuery(newQuery);
            mHandler.removeCallbacks(mDelayedLoad);
            mHandler.postDelayed(mDelayedLoad, SEARCH_DELAY_MS);
        }
        return true;
    }

    @Override
    public boolean onQueryTextSubmit(String query) {
        mRowsAdapter.clear();
        if (!TextUtils.isEmpty(query)) {
            mDelayedLoad.setSearchQuery(query);
            mHandler.removeCallbacks(mDelayedLoad);
            mHandler.postDelayed(mDelayedLoad, SEARCH_DELAY_MS);
        }
        return true;
    }
}
```

<!--The example code shown above is meant to be used with a separate SearchRunnable class that runs the search query on a separate thread. This technique keeps potentially slow-running queries from blocking the main user interface thread.-->

上面的示例代碼展示了在分開的線程用獨立的`SearchRunnable`類去運行搜索請求。這個技巧是從正在阻塞的主線程保持了潛在的慢運行請求。

----------------
[下一節: 創建TV遊戲應用 >](../games/index.html)
