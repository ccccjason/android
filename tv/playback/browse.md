# 創建目錄瀏覽器

> 編寫:[huanglizhuo](https://github.com/huanglizhuo) - 原文:<http://developer.android.com/training/tv/playback/browse.html>

在TV上運行的 多媒體應用得允許用戶瀏覽,選擇和播放它所提供的內容。目錄瀏覽器的用戶體驗要簡單和直觀，以及賞心悅目，引人入勝。

這節課討論如何使用的[V17 Leanback](http://developer.android.com/tools/support-library/features.html#v17-leanback)庫提供的類來實現用戶界面，用於從您的應用程序的媒體目錄瀏覽音樂或視頻。

##創建一個目錄佈局

leanback 類庫中的[BrowseFragment](http://developer.android.com/reference/android/support/v17/leanback/app/BrowseFragment.html)允許您用最少的代碼創建一個用於按行瀏覽的主佈局 ,下面的例子將演示如何創建包含[BrowseFragment](http://developer.android.com/reference/android/support/v17/leanback/app/BrowseFragment.html)的佈局

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  android:orientation="vertical"
  >

  <fragment
      android:name="android.support.v17.leanback.app.BrowseFragment"
      android:id="@+id/browse_fragment"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      />
</LinearLayout>
```

為了使 activity 工作,需要在佈局中取回[BrowseFragment](http://developer.android.com/reference/android/support/v17/leanback/app/BrowseFragment.html)的元素。使用這個類中的方法設置顯示參數,如圖標,標題,以及該類別是否可用。下面的代碼簡單的演示了怎樣設置[BrowseFragment](http://developer.android.com/reference/android/support/v17/leanback/app/BrowseFragment.html)佈局參數:

```xml
public class BrowseMediaActivity extends Activity {

    public static final String TAG ="BrowseActivity";

    protected BrowseFragment mBrowseFragment;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.browse_fragment);

        final FragmentManager fragmentManager = getFragmentManager();
        mBrowseFragment = (BrowseFragment) fragmentManager.findFragmentById(
                R.id.browse_fragment);

        // Set display parameters for the BrowseFragment
        mBrowseFragment.setHeadersState(BrowseFragment.HEADERS_ENABLED);
        mBrowseFragment.setTitle(getString(R.string.app_name));
        mBrowseFragment.setBadgeDrawable(getResources().getDrawable(
                R.drawable.ic_launcher));
        mBrowseFragment.setBrowseParams(params);

    }
}
```

##顯示媒體列表

[BrowseFragment](http://developer.android.com/reference/android/support/v17/leanback/app/BrowseFragment.html)允許您定義和使用 adapter 和presenter 定義顯示可瀏覽媒體內容類別和媒體項目。Adapters 允許我們連接本地或網絡數據資源。Presenters操控的媒體項目的數據，並提供佈局信息在屏幕上顯示的項目。

下面的示例代碼演示了一個為顯示字符串數據的Presenters的實現

```xml
public class StringPresenter extends Presenter {
    private static final String TAG = "StringPresenter";

    public ViewHolder onCreateViewHolder(ViewGroup parent) {
        TextView textView = new TextView(parent.getContext());
        textView.setFocusable(true);
        textView.setFocusableInTouchMode(true);
        textView.setBackground(
                parent.getContext().getResources().getDrawable(R.drawable.text_bg));
        return new ViewHolder(textView);
    }

    public void onBindViewHolder(ViewHolder viewHolder, Object item) {
        ((TextView) viewHolder.view).setText(item.toString());
    }

    public void onUnbindViewHolder(ViewHolder viewHolder) {
        // no op
    }
}
```

當我們已經為我們的媒體項目構建了一個Presenter類，我們可以為[BrowseFragment](http://developer.android.com/reference/android/support/v17/leanback/app/BrowseFragment.html)建立並添加一個適配器並在屏幕上顯示這些媒體項目。下面的示例代碼演示瞭如何用StringPresenter類構造一個類別和項目適配器：

```xml
private ArrayObjectAdapter mRowsAdapter;
private static final int NUM_ROWS = 4;

@Override
protected void onCreate(Bundle savedInstanceState) {
    ...

    buildRowsAdapter();
}

private void buildRowsAdapter() {
    mRowsAdapter = new ArrayObjectAdapter(new ListRowPresenter());

    for (int i = 0; i < NUM_ROWS; ++i) {
        ArrayObjectAdapter listRowAdapter = new ArrayObjectAdapter(
                new StringPresenter());
        listRowAdapter.add("Media Item 1");
        listRowAdapter.add("Media Item 2");
        listRowAdapter.add("Media Item 3");
        HeaderItem header = new HeaderItem(i, "Category " + i, null);
        mRowsAdapter.add(new ListRow(header, listRowAdapter));
    }

    mBrowseFragment.setAdapter(mRowsAdapter);
}
```

這個例子顯示了靜態實現適配器。典型的媒體瀏覽器使用網絡數據庫或網絡服務。使用從網絡取回的數據做的媒體瀏覽器,參看例子[Android TV](https://github.com/googlesamples/androidtv-leanback)

##更新背景

為了給媒體瀏覽應用增加視覺趣味，我們可以在用戶瀏覽的內容時更新背景圖片。這種技術可以讓我們的應用程序的互動感倍增。

Leanback庫提供了[BackgroundManager](http://developer.android.com/reference/android/support/v17/leanback/app/BackgroundManager.html)類為我們的TV應用的activity更換背景。下面的例子演示瞭如何創建一個簡單的方法更換背景:

```xml
protected void updateBackground(Drawable drawable) {
    BackgroundManager.getInstance(this).setDrawable(drawable);
}
```

許多現有的媒體瀏覽應用在用戶瀏覽媒體列表自動更新的背景。為了做到這一點，我們可以設置一個選擇監聽器，根據用戶的當前選擇自動更新背景。下面的例子演示瞭如何建立一個[OnItemViewSelectedListener](http://developer.android.com/reference/android/support/v17/leanback/widget/OnItemViewSelectedListener.html)監聽選擇事件並更新背景：

```xml
protected void clearBackground() {
    BackgroundManager.getInstance(this).setDrawable(mDefaultBackground);
}

protected OnItemViewSelectedListener getDefaultItemViewSelectedListener() {
    return new OnItemViewSelectedListener() {
        @Override
        public void onItemSelected(Object item, Row row) {
            if (item instanceof Movie ) {
                URI uri = ((Movie)item).getBackdropURI();
                updateBackground(uri);
            } else {
                clearBackground();
            }
        }
    };
}
```

> **注意**:以上的示例是為了簡單。當我們在自己的應用程序創建這個功能，我們應該考慮運行在一個單獨的線程在後臺更新操作獲得更好的性能。此外，如果我們正計劃在用戶觸發項目滾動時更新背景，考慮增加一個時延，直到用戶停止操作時再更新背景圖像。這樣可以避免過多的背景圖片的更新。

------------
[下一節：提供一個卡片View >](card.html)