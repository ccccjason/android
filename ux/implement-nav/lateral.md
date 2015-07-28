# 使用Tabs創建Swipe視圖

> 編寫:[Lin-H](https://github.com/Lin-H) - 原文:<http://developer.android.com/training/implementing-navigation/lateral.html>

Swipe View提供在同級屏幕中的橫向導航，例如通過橫向劃屏手勢切換的tab(一種稱作橫向分頁的模式)。這節課會教你如何使用swipe view創建一個tab layout實現在tab之間切換，或顯示一個標題條替代tab。

>**Swipe View 設計**

> 在實現這些功能之前，你要先明白在[Designing Effective Navigation](http://developer.android.com/training/design-navigation/descendant-lateral.html), [Swipe Views](http://developer.android.com/design/patterns/swipe-views.html) design guide中的概念和建議

## 實現Swipe View

你可以使用[Support Library](http://developer.android.com/tools/support-library/index.html)中的[ViewPager](http://developer.android.com/reference/android/support/v4/view/ViewPager.html)控件在你的app中創建swipe view。[ViewPager](http://developer.android.com/reference/android/support/v4/view/ViewPager.html)是一個子視圖在layout上相互獨立的佈局控件(layout widget)。

使用[ViewPager](http://developer.android.com/reference/android/support/v4/view/ViewPager.html)來設置你的layout，要添加一個`<ViewPager>`元素到你的XML layout中。例如，在你的swipe view中如果每一個頁面都會佔用整個layout，那麼你的layout應該是這樣:

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.view.ViewPager
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/pager"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

要插入每一個頁面的子視圖，你需要把這個layout與[PagerAdapter](http://developer.android.com/reference/android/support/v4/view/PagerAdapter.html)掛鉤。有兩種adapter(適配器)你可以用:

[FragmentPagerAdapter](http://developer.android.com/reference/android/support/v4/app/FragmentPagerAdapter.html)

在同級屏幕(sibling screen)只有少量的幾個固定頁面時，使用這個最好。

[FragmentStatePagerAdapter](http://developer.android.com/reference/android/support/v4/app/FragmentStatePagerAdapter.html)

當根據對象集的數量來劃分頁面，即一開始頁面的數量未確定時，使用這個最好。當用戶切換到其他頁面時，fragment會被銷燬來降低內存消耗。

例如，這裡的代碼是當你使用[FragmentStatePagerAdapter](http://developer.android.com/reference/android/support/v4/app/FragmentStatePagerAdapter.html)來在[Fragment](http://developer.android.com/reference/android/app/Fragment.html)對象集合中進行橫屏切換:

```java
public class CollectionDemoActivity extends FragmentActivity {
    // 當被請求時, 這個adapter會返回一個DemoObjectFragment,
    // 代表在對象集中的一個對象.
    DemoCollectionPagerAdapter mDemoCollectionPagerAdapter;
    ViewPager mViewPager;

    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_collection_demo);

        // ViewPager和他的adapter使用了support library
        // fragments,所以要用getSupportFragmentManager.
        mDemoCollectionPagerAdapter =
                new DemoCollectionPagerAdapter(
                        getSupportFragmentManager());
        mViewPager = (ViewPager) findViewById(R.id.pager);
        mViewPager.setAdapter(mDemoCollectionPagerAdapter);
    }
}

// 因為這是一個對象集所以使用FragmentStatePagerAdapter,
// 而不是FragmentPagerAdapter.
public class DemoCollectionPagerAdapter extends FragmentStatePagerAdapter {
    public DemoCollectionPagerAdapter(FragmentManager fm) {
        super(fm);
    }

    @Override
    public Fragment getItem(int i) {
        Fragment fragment = new DemoObjectFragment();
        Bundle args = new Bundle();
        // 我們的對象只是一個整數 :-P
        args.putInt(DemoObjectFragment.ARG_OBJECT, i + 1);
        fragment.setArguments(args);
        return fragment;
    }

    @Override
    public int getCount() {
        return 100;
    }

    @Override
    public CharSequence getPageTitle(int position) {
        return "OBJECT " + (position + 1);
    }
}

// 這個類的實例是一個代表了數據集中一個對象的fragment
public static class DemoObjectFragment extends Fragment {
    public static final String ARG_OBJECT = "object";

    @Override
    public View onCreateView(LayoutInflater inflater,
            ViewGroup container, Bundle savedInstanceState) {
        // 最後兩個參數保證LayoutParam能被正確填充
        View rootView = inflater.inflate(
                R.layout.fragment_collection_object, container, false);
        Bundle args = getArguments();
        ((TextView) rootView.findViewById(android.R.id.text1)).setText(
                Integer.toString(args.getInt(ARG_OBJECT)));
        return rootView;
    }
}
```

這個例子只顯示了創建swipe view的必要代碼。下面一節向你說明如何通過添加tab使導航更方便在頁面間切換。

## 添加Tab到Action Bar

Action bar [tab](http://developer.android.com/design/building-blocks/tabs.html)能給用戶提供更熟悉的界面來在app的同級屏幕中切換和分辨。

使用[ActionBar](http://developer.android.com/reference/android/app/ActionBar.html)來創建tab，你需要啟用[NAVIGATION_MODE_TABS](http://developer.android.com/reference/android/app/ActionBar.html#NAVIGATION_MODE_TABS)，然後創建幾個[ActionBar.Tab](http://developer.android.com/reference/android/app/ActionBar.Tab.html)的實例，並對每個實例實現[ActionBar.TabListener](http://developer.android.com/reference/android/app/ActionBar.TabListener.html)接口。例如在你的activity的[onCreate()](http://developer.android.com/reference/android/app/Activity.html#onCreate%28android.os.Bundle%29)方法中，你可以使用與下面相似的代碼:

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    final ActionBar actionBar = getActionBar();
    ...

    // 指定在action bar中顯示tab.
    actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_TABS);

    // 創建一個tab listener，在用戶切換tab時調用.
    ActionBar.TabListener tabListener = new ActionBar.TabListener() {
        public void onTabSelected(ActionBar.Tab tab, FragmentTransaction ft) {
            // 顯示指定的tab
        }

        public void onTabUnselected(ActionBar.Tab tab, FragmentTransaction ft) {
            // 隱藏指定的tab
        }

        public void onTabReselected(ActionBar.Tab tab, FragmentTransaction ft) {
            // 可以忽略這個事件
        }
    };

    // 添加3個tab, 並指定tab的文字和TabListener
    for (int i = 0; i < 3; i++) {
        actionBar.addTab(
                actionBar.newTab()
                        .setText("Tab " + (i + 1))
                        .setTabListener(tabListener));
    }
}
```

根據你如何創建你的內容來處理[ActionBar.TabListener](http://developer.android.com/reference/android/app/ActionBar.TabListener.html)回調改變tab。但是如果你是像上面那樣，通過[ViewPager](http://developer.android.com/reference/android/support/v4/view/ViewPager.html)對每個tab使用fragment，下面這節就會說明當用戶選擇一個tab時如何切換頁面，當用戶劃屏切換頁面時如何更新相應頁面的tab。

## 使用Swipe View切換Tab

當用戶選擇tab時，在[ViewPager](http://developer.android.com/reference/android/support/v4/view/ViewPager.html)中切換頁面，需要實現[ActionBar.TabListener](http://developer.android.com/reference/android/app/ActionBar.TabListener.html)來調用在[ViewPager](http://developer.android.com/reference/android/support/v4/view/ViewPager.html)中的[setCurrentItem()]()來選擇相應的頁面:

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    ...

    // Create a tab listener that is called when the user changes tabs.
    ActionBar.TabListener tabListener = new ActionBar.TabListener() {
        public void onTabSelected(ActionBar.Tab tab, FragmentTransaction ft) {
            // 當tab被選中時, 切換到ViewPager中相應的頁面.
            mViewPager.setCurrentItem(tab.getPosition());
        }
        ...
    };
}
```

同樣的，當用戶通過觸屏手勢(touch gesture)切換頁面時，你也應該選擇相應的tab。你可以通過實現[ViewPager.OnPageChangeListener](http://developer.android.com/reference/android/support/v4/view/ViewPager.OnPageChangeListener.html)接口來設置這個操作，當頁面變化時當前的tab也相應變化。例如:

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    ...

    mViewPager = (ViewPager) findViewById(R.id.pager);
    mViewPager.setOnPageChangeListener(
            new ViewPager.SimpleOnPageChangeListener() {
                @Override
                public void onPageSelected(int position) {
                    // 當劃屏切換頁面時，選擇相應的tab.
                    getActionBar().setSelectedNavigationItem(position);
                }
            });
    ...
}
```

## 使用標題欄替代Tab

如果你不想使用action bar tab，而想使用[scrollable tabs](http://developer.android.com/design/building-blocks/tabs.html#scrollable)來提供一個更簡短的可視化配置，你可以在swipe view中使用[PagerTitleStrip](http://developer.android.com/reference/android/support/v4/view/PagerTitleStrip.html)。

下面是一個內容為[ViewPager](http://developer.android.com/reference/android/support/v4/view/ViewPager.html)，有一個[PagerTitleStrip](http://developer.android.com/reference/android/support/v4/view/PagerTitleStrip.html)頂端對齊的activity的layout XML文件示例。單個頁面(adapter提供)佔據[ViewPager](http://developer.android.com/reference/android/support/v4/view/ViewPager.html)中的剩餘空間。

```xml
<android.support.v4.view.ViewPager
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/pager"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.v4.view.PagerTitleStrip
        android:id="@+id/pager_title_strip"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_gravity="top"
        android:background="#33b5e5"
        android:textColor="#fff"
        android:paddingTop="4dp"
        android:paddingBottom="4dp" />

</android.support.v4.view.ViewPager>
```
