# 實現自適應UI流（Flows）

> 編寫:[riverfeng](https://github.com/riverfeng) - 原文:<http://developer.android.com/training/multiscreen/adaptui.html>

根據當前你的應用顯示的佈局，它的UI流可能會不一樣。比如，當你的應用是雙窗格模式，點擊左邊窗格的條目（item）時，內容（content）顯示在右邊窗格中。如果是單窗格模式中，當你點擊某個item的時候，內容則顯示在一個新的activity中。

## 確定當前佈局
由於每種佈局的實現會略有差別，首先你可能要確定用戶當前可見的佈局是哪一個。比如，你可能想知道當前用戶到底是處於“單窗格”的模式還是“雙窗格”的模式。你可以通過檢查指定的視圖（view）是否存在和可見來實現：

```java
public class NewsReaderActivity extends FragmentActivity {
    boolean mIsDualPane;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main_layout);

        View articleView = findViewById(R.id.article);
        mIsDualPane = articleView != null &&
                        articleView.getVisibility() == View.VISIBLE;
    }
}
```

> 注意：使用代碼查詢id為“article”的view是否可見比直接硬編碼查詢指定的佈局更加的靈活。

另一個關於如何適配不同組件是否存在的例子，是在組件執行操作之前先檢查它是否是可用的。比如，在News Reader示例中，有一個按鈕點擊後打開一個菜單，但是這個按鈕僅僅只在Android3.0之後的版本中才能顯示（因為這個功能被ActionBar代替，在API 11+中定義）。所以，在給這個按鈕添加事件之間，你可以這樣做：
```java
Button catButton = (Button) findViewById(R.id.categorybutton);
OnClickListener listener = /* create your listener here */;
if (catButton != null) {
    catButton.setOnClickListener(listener);
}
```

## 根據當前佈局響應

一些操作會根據當前的佈局產生不同的效果。比如，在News Reader示例中，當你點擊標題（headlines）列表中的某一條headline時，如果你的UI是雙窗格模式，內容會顯示在右邊的窗格中，如果你的UI是單窗格模式，會啟動一個分開的Activity並顯示：
```java
@Override
public void onHeadlineSelected(int index) {
    mArtIndex = index;
    if (mIsDualPane) {
        /* display article on the right pane */
        mArticleFragment.displayArticle(mCurrentCat.getArticle(index));
    } else {
        /* start a separate activity */
        Intent intent = new Intent(this, ArticleActivity.class);
        intent.putExtra("catIndex", mCatIndex);
        intent.putExtra("artIndex", index);
        startActivity(intent);
    }
}
```
同樣，如果你的應用處於多窗格模式，那麼它應該在導航欄中設置帶有選項卡的action bar。而如果是單窗格模式，那麼導航欄應該設置為spinner widget。所以，你的代碼應該檢查哪個方案是最合適的：
```java
final String CATEGORIES[] = { "Top Stories", "Politics", "Economy", "Technology" };

public void onCreate(Bundle savedInstanceState) {
    ....
    if (mIsDualPane) {
        /* use tabs for navigation */
        actionBar.setNavigationMode(android.app.ActionBar.NAVIGATION_MODE_TABS);
        int i;
        for (i = 0; i < CATEGORIES.length; i++) {
            actionBar.addTab(actionBar.newTab().setText(
                CATEGORIES[i]).setTabListener(handler));
        }
        actionBar.setSelectedNavigationItem(selTab);
    }
    else {
        /* use list navigation (spinner) */
        actionBar.setNavigationMode(android.app.ActionBar.NAVIGATION_MODE_LIST);
        SpinnerAdapter adap = new ArrayAdapter(this,
                R.layout.headline_item, CATEGORIES);
        actionBar.setListNavigationCallbacks(adap, handler);
    }
}
```

## 在其他Activity中複用Fragment

在多屏幕設計時經常出現的情況是：在一些屏幕配置上設計一個窗格，而在其他屏幕配置上啟動一個獨立的Activity。例如，在News Reader中，新聞內容文字在大屏幕上市顯示在屏幕右邊的方框中，而在小屏幕中，則是由單獨的activity顯示的。

像這樣的情況，你就應該在不同的activity中使用同一個Fragment，以此來避免代碼的重複，而達到代碼複用的效果。比如，ArticleFragment在雙窗格模式下是這樣用的：
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="horizontal">
    <fragment android:id="@+id/headlines"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.HeadlinesFragment"
              android:layout_width="400dp"
              android:layout_marginRight="10dp"/>
    <fragment android:id="@+id/article"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.ArticleFragment"
              android:layout_width="fill_parent" />
</LinearLayout>
```
在小屏幕中，它又是如下方式被複用的（沒有佈局文件）：
```java
ArticleFragment frag = new ArticleFragment();
getSupportFragmentManager().beginTransaction().add(android.R.id.content, frag).commit();
```
當然，如果將這個fragment定義在XML佈局文件中，也有同樣的效果，但是在這個例子中，則沒有必要，因為這個article fragment是這個activity的唯一組件。

當你在設計fragment的時候，非常重要的一點：不要為某個特定的activity設計耦合度高的fragment。通常的做法是，通過定義抽象接口，並在接口中定義需要與該fragment進行交互的activity的抽象方法，然後與該fragment進行交互的activity實現這些抽象接口方法。

例如，在News Reader中，HeadlinesFragment就很好的詮釋了這一點：
```java
public class HeadlinesFragment extends ListFragment {
    ...
    OnHeadlineSelectedListener mHeadlineSelectedListener = null;

    /* Must be implemented by host activity */
    public interface OnHeadlineSelectedListener {
        public void onHeadlineSelected(int index);
    }
    ...

    public void setOnHeadlineSelectedListener(OnHeadlineSelectedListener listener) {
        mHeadlineSelectedListener = listener;
    }
}
```
然後，當用戶選擇了一個headline item之後，fragment將通知對應的activity指定監聽事件（而不是通過硬編碼的方式去通知）：
```java
public class HeadlinesFragment extends ListFragment {
    ...
    @Override
    public void onItemClick(AdapterView<?> parent,
                            View view, int position, long id) {
        if (null != mHeadlineSelectedListener) {
            mHeadlineSelectedListener.onHeadlineSelected(position);
        }
    }
    ...
}
```
這種技術在[支持平板與手持設備(Supporting Tablets and Handsets)](http://developer.android.com/guide/practices/tablets-and-handsets.html)有更加詳細的介紹。

## 處理屏幕配置變化

如果使用的是單獨的activity來實現你界面的不同部分，你需要注意的是，屏幕變化（如旋轉變化）的時候，你也應該根據屏幕配置的變化來保持你的UI佈局的一致性。

例如，在傳統的Android3.0或以上版本的7寸平板上，News Reader示例在豎屏的時候使用獨立的activity顯示文章內容，而在橫屏的時候，則使用兩個窗格模式（即內容顯示在右邊的方框中）。
這也就意味著，當用戶在豎屏模式下觀看文章的時候，你需要檢測屏幕是否變成了橫屏，如果改變了，則結束當前activity並返回到主activity中，這樣，content就能顯示在雙窗格模式佈局中。
```java
public class ArticleActivity extends FragmentActivity {
    int mCatIndex, mArtIndex;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mCatIndex = getIntent().getExtras().getInt("catIndex", 0);
        mArtIndex = getIntent().getExtras().getInt("artIndex", 0);

        // If should be in two-pane mode, finish to return to main activity
        if (getResources().getBoolean(R.bool.has_two_panes)) {
            finish();
            return;
        }
        ...
}
```
