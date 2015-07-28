# 創建詳情頁

編寫:[huanglizhuo](https://github.com/huanglizhuo) - 原文:<http://developer.android.com/training/tv/playback/details.html>

待認領進行編寫，有意向的小夥伴，可以直接修改對應的markdown文件，進行提交！

[ v17 leanback support library ](http://developer.android.com/tools/support-library/features.html#v17-leanback)庫提供的媒體瀏覽接口包含顯示附加媒體信息的類,比如描述和預覽,以及對項目的操作,比如購買或播放。


這節課討論如何為媒體項目的詳細信息創建 presenter 類，以及用戶選擇一個媒體項目時如何擴展[ DetailsFragment](http://developer.android.com/reference/android/support/v17/leanback/app/DetailsFragment.html)類來實現顯示媒體詳細信息視圖。

> **小貼士:** 這裡的實現例子用的是包含[ DetailsFragment](http://developer.android.com/reference/android/support/v17/leanback/app/DetailsFragment.html)的附加activity。但也可以在同一個 activity 中用 fragment 轉換將[ BrowseFragment](http://developer.android.com/reference/android/support/v17/leanback/app/BrowseFragment.html)替換為[ DetailsFragment](http://developer.android.com/reference/android/support/v17/leanback/app/DetailsFragment.html).更多關於fragment的信息請參考[Building a Dynamic UI with Fragments](http://developer.android.com/training/basics/fragments/fragment-ui.html#Replace)

##創建詳細Presenter

在leanback庫提供的媒體瀏覽框架中,可以用presenter對象控制屏幕顯示數據,包括媒體詳細信息。[AbstractDetailsDescriptionPresenter ](http://developer.android.com/reference/android/support/v17/leanback/widget/AbstractDetailsDescriptionPresenter.html)類提供的框架幾乎是媒體項目詳細信息的完全繼承。我們只需要實現[onBindDescription()]()方法,像下面這樣把數據信息和視圖綁定起來。

```xml
public class DetailsDescriptionPresenter
        extends AbstractDetailsDescriptionPresenter {

    @Override
    protected void onBindDescription(ViewHolder viewHolder, Object itemData) {
        MyMediaItemDetails details = (MyMediaItemDetails) itemData;
        // In a production app, the itemData object contains the information
        // needed to display details for the media item:
        // viewHolder.getTitle().setText(details.getShortTitle());

        // Here we provide static data for testing purposes:
        viewHolder.getTitle().setText(itemData.toString());
        viewHolder.getSubtitle().setText("2014   Drama   TV-14");
        viewHolder.getBody().setText("Lorem ipsum dolor sit amet, consectetur "
                + "adipisicing elit, sed do eiusmod tempor incididunt ut labore "
                + " et dolore magna aliqua. Ut enim ad minim veniam, quis "
                + "nostrud exercitation ullamco laboris nisi ut aliquip ex ea "
                + "commodo consequat.");
    }
}
```

##擴展詳細fragment

當使用[ DetailsFragment](http://developer.android.com/reference/android/support/v17/leanback/app/DetailsFragment.html)類顯示我們的媒體項目詳細信息時,擴展該類並提供像預覽圖片,操作等附加內容。我們也可以提供一系列的相關媒體信息。

下面的例子演示了怎樣用presenter類為媒體項目添加預覽圖片和操作。這個例子也演示了添加相關媒體行。

```xml
public class MediaItemDetailsFragment extends DetailsFragment {
    private static final String TAG = "MediaItemDetailsFragment";
    private ArrayObjectAdapter mRowsAdapter;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        Log.i(TAG, "onCreate");
        super.onCreate(savedInstanceState);

        buildDetails();
    }

    private void buildDetails() {
        ClassPresenterSelector selector = new ClassPresenterSelector();
        // Attach your media item details presenter to the row presenter:
        DetailsOverviewRowPresenter rowPresenter =
            new DetailsOverviewRowPresenter(new DetailsDescriptionPresenter());

        selector.addClassPresenter(DetailsOverviewRow.class, rowPresenter);
        selector.addClassPresenter(ListRow.class,
                new ListRowPresenter());
        mRowsAdapter = new ArrayObjectAdapter(selector);

        Resources res = getActivity().getResources();
        DetailsOverviewRow detailsOverview = new DetailsOverviewRow(
                "Media Item Details");

        // Add images and action buttons to the details view
        detailsOverview.setImageDrawable(res.getDrawable(R.drawable.jelly_beans));
        detailsOverview.addAction(new Action(1, "Buy $9.99"));
        detailsOverview.addAction(new Action(2, "Rent $2.99"));
        mRowsAdapter.add(detailsOverview);

        // Add a Related items row
        ArrayObjectAdapter listRowAdapter = new ArrayObjectAdapter(
                new StringPresenter());
        listRowAdapter.add("Media Item 1");
        listRowAdapter.add("Media Item 2");
        listRowAdapter.add("Media Item 3");
        HeaderItem header = new HeaderItem(0, "Related Items", null);
        mRowsAdapter.add(new ListRow(header, listRowAdapter));

        setAdapter(mRowsAdapter);
    }
}
```

##創建詳細信息activity

像[ DetailsFragment](http://developer.android.com/reference/android/support/v17/leanback/app/DetailsFragment.html)這樣的 fragment 為了使用或顯示必須包含activity。為我們的詳細信息與瀏覽分開創建activity並通過傳遞Intent打開。這節演示瞭如何創建一個包含媒體詳細信息的activity。

創建詳細信息前先為[ DetailsFragment](http://developer.android.com/reference/android/support/v17/leanback/app/DetailsFragment.html)創建一個佈局文件:

```xml
<!-- file: res/layout/details.xml -->

<fragment xmlns:android="http://schemas.android.com/apk/res/android"
    android:name="com.example.android.mediabrowser.MediaItemDetailsFragment"
    android:id="@+id/details_fragment"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
/>
```

接下來用上面的佈局文件創建一個activity:

```xml
public class DetailsActivity extends Activity
{
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.details);
    }
}
```

最後在manifest文件中申明activity。記得添加Leanback主題以確保用戶界面中有媒體瀏覽activity。

```xml
<application>
  ...

  <activity android:name=".DetailsActivity"
    android:exported="true"
    android:theme="@style/Theme.Leanback"/>

</application>
```

##為點擊項目添加Listener

實現[ DetailsFragment](http://developer.android.com/reference/android/support/v17/leanback/app/DetailsFragment.html)後,在用戶點擊媒體條目時將我們的媒體瀏覽view切換詳細信息view。為了確保動作的實現,在[BrowserFragment]()中添加[OnItemViewClickedListener]通過Intent開啟詳細信息activity。


下面的例子演示了實現怎樣在媒體瀏覽view中實現一個 listener開啟詳細信息view。

```xml
public class BrowseMediaActivity extends Activity {
    ...

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...

        // create the media item rows
        buildRowsAdapter();

        // add a listener for selected items
        mBrowseFragment.OnItemViewClickedListener(
            new OnItemViewClickedListener() {
                @Override
                public void onItemClicked(Object item, Row row) {
                    System.out.println("Media Item clicked: " + item.toString());
                    Intent intent = new Intent(BrowseMediaActivity.this,
                            DetailsActivity.class);
                    // pass the item information
                    intent.getExtras().putLong("id", item.getId());
                    startActivity(intent);
                }
            });
    }
}
```

-----------
[下一節：顯示正在播放卡片 >](now-playing.html)