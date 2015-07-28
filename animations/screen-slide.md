# 使用ViewPager實現屏幕滑動

> 編寫:[XizhiXu](https://github.com/XizhiXu) - 原文:<http://developer.android.com/training/animation/screen-slide.html>

屏幕划動是在兩個完整界面間的轉換，它在一些UI中很常見，比如設置嚮導和幻燈放映。這節課將告訴你怎樣通過[support library](http://developer.android.com/tools/support-library/index.html)提供的[`ViewPager`](http://developer.android.com/reference/android/support/v4/view/ViewPager.html)實現屏幕滑動。[`ViewPager`](http://developer.android.com/reference/android/support/v4/view/ViewPager.html)能自動實現屏幕滑動動畫。下面展示了從一個內容界面到一下界面的屏幕滑動轉換是什麼樣子的。

<div style="
  background: transparent url(device_galaxynexus_blank_land_span8.png) no-repeat
scroll top left; padding: 26px 68px 38px 72px; overflow: hidden;">

<video style="width: 320px; height: 180px;" controls="" autoplay="">
    <source src="anim_screenslide.mp4" type="video/mp4">
    <source src="anim_screenslide.webm" type="video/webm">
    <source src="anim_screenslide.ogv" type="video/ogg">
</video>

</div>

如果你想直接查看整個例子，[下載](http://developer.android.com/shareables/training/Animations.zip)並運行App樣例然後選擇屏幕滑動例子。查看下列文件中的代碼實現：

* `src/ScreenSlidePageFragment.java`
* `src/ScreenSlideActivity.java`
* `layout/activity_screen_slide.xml`
* `layout/fragment_screen_slide_page.xml`

##創建View

創建Fragment所使用的佈局文件。下面的例子包含一個顯示文本的TextView：

```xml
<!-- fragment_screen_slide_page.xml -->
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/content"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <TextView style="?android:textAppearanceMedium"
        android:padding="16dp"
        android:lineSpacingMultiplier="1.2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/lorem_ipsum" />
</ScrollView>
```

與此同時我們還定義了一個字符串作為該Fragment的內容。

## 創建Fragment

創建一個 [`Fragment`](http://developer.android.com/reference/android/support/v4/app/Fragment.html) 子類，它從<a href="http://developer.android.com/reference/android/app/Fragment.html#onCreateView(android.view.LayoutInflater, android.view.ViewGroup, android.os.Bundle)"> `onCreateView()` </a>方法中返回之前創建的佈局。無論何時如果我們需要為用戶展示一個新的頁面，可以在它的父Activity中創建該Fragment的實例：

```java
import android.support.v4.app.Fragment;
...
public class ScreenSlidePageFragment extends Fragment {

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
            Bundle savedInstanceState) {
        ViewGroup rootView = (ViewGroup) inflater.inflate(
                R.layout.fragment_screen_slide_page, container, false);

        return rootView;
    }
}
```

## 添加ViewPager

[`ViewPager`](http://developer.android.com/reference/android/support/v4/view/ViewPager.html) 有內建的滑動手勢用來在頁面間轉換，並且它默認使用滑屏動畫，所以我們不用自己為其創建。[`ViewPager`](http://developer.android.com/reference/android/support/v4/view/ViewPager.html)使用[`PagerAdapter`](http://developer.android.com/reference/android/support/v4/view/PagerAdapter.html)來補充新頁面，所以[`PagerAdapter`](http://developer.android.com/reference/android/support/v4/view/PagerAdapter.html)會用到你之前新建的Fragment類。

開始之前，創建一個包含[`ViewPager`](http://developer.android.com/reference/android/support/v4/view/ViewPager.html)的佈局：

```xml
<!-- activity_screen_slide.xml -->
<android.support.v4.view.ViewPager
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/pager"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

創建一個Activity來做下面這些事情：

* 把ContentView設置成這個包含[`ViewPager`](http://developer.android.com/reference/android/support/v4/view/ViewPager.html)的佈局。

* 創建一個繼承自[`FragmentStatePagerAdapter `](http://developer.android.com/reference/android/support/v13/app/FragmentStatePagerAdapter.html)抽象類的類，然後實現<a href="http://developer.android.com/reference/android/support/v4/app/FragmentStatePagerAdapter.html#getItem(int)">`getItem()`</a>方法來把`ScreenSlidePageFragment`實例作為新頁面補充進來。PagerAdapter還需要實現<a href="http://developer.android.com/reference/android/support/v4/view/PagerAdapter.html#getCount()">`getCount()`</a>方法，它返回 Adapter將要創建頁面的總數（例如5個）。

* 把[`PagerAdapter`](http://developer.android.com/reference/android/support/v4/view/PagerAdapter.html)和[`ViewPager`](http://developer.android.com/reference/android/support/v4/view/ViewPager.html)關聯起來。

* 處理Back按鈕，按下變為在虛擬的Fragment棧中回退。如果用戶已經在第一個頁面了，則在Activity的回退棧（back stack）中回退。

```java
import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentManager;
...
public class ScreenSlidePagerActivity extends FragmentActivity {
    /**
     * The number of pages (wizard steps) to show in this demo.
     */
    private static final int NUM_PAGES = 5;

    /**
     * The pager widget, which handles animation and allows swiping horizontally to access previous
     * and next wizard steps.
     */
    private ViewPager mPager;

    /**
     * The pager adapter, which provides the pages to the view pager widget.
     */
    private PagerAdapter mPagerAdapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_screen_slide);

        // Instantiate a ViewPager and a PagerAdapter.
        mPager = (ViewPager) findViewById(R.id.pager);
        mPagerAdapter = new ScreenSlidePagerAdapter(getSupportFragmentManager());
        mPager.setAdapter(mPagerAdapter);
    }

    @Override
    public void onBackPressed() {
        if (mPager.getCurrentItem() == 0) {
            // If the user is currently looking at the first step, allow the system to handle the
            // Back button. This calls finish() on this activity and pops the back stack.
            super.onBackPressed();
        } else {
            // Otherwise, select the previous step.
            mPager.setCurrentItem(mPager.getCurrentItem() - 1);
        }
    }

    /**
     * A simple pager adapter that represents 5 ScreenSlidePageFragment objects, in
     * sequence.
     */
    private class ScreenSlidePagerAdapter extends FragmentStatePagerAdapter {
        public ScreenSlidePagerAdapter(FragmentManager fm) {
            super(fm);
        }

        @Override
        public Fragment getItem(int position) {
            return new ScreenSlidePageFragment();
        }

        @Override
        public int getCount() {
            return NUM_PAGES;
        }
    }
}
```

## 用PageTransformer自定義動畫

要展示不同於默認滑屏效果的動畫，我們需要實現[`ViewPager.PageTransformer`](http://developer.android.com/reference/android/support/v4/view/ViewPager.PageTransformer.html)接口，然後把它補充到ViewPager裡就行了。這個接口只暴露了一個方法，<a href="http://developer.android.com/reference/android/support/v4/view/ViewPager.PageTransformer.html#transformPage(android.view.View, float)">`transformPage()`</a>。每次界面切換，這個方法都會為每個可見頁面（通常只有一個頁面可見）和剛消失的相鄰頁面調用一次。例如，第三頁可見而且用戶向第四頁拖動，<a href="http://developer.android.com/reference/android/support/v4/view/ViewPager.PageTransformer.html#transformPage(android.view.View, float)">`transformPage()`</a>在操作的各個階段為第二，三，四頁分別調用。

在<a href="http://developer.android.com/reference/android/support/v4/view/ViewPager.PageTransformer.html#transformPage(android.view.View, float)">`transformPage()`</a>的實現中，基於當前屏幕顯示的頁面的`position`（`position` 由<a href="http://developer.android.com/reference/android/support/v4/view/ViewPager.PageTransformer.html#transformPage(android.view.View, float)">`transformPage()`</a>方法的參數給出）決定哪些頁面需要被動畫轉換，這樣我們就能創建自己的動畫。

`position`參數表示特定頁面相對於屏幕中的頁面的位置。它的值在用戶滑動頁面過程中動態變化。當某一頁面填充屏幕，它的值為0。當頁面剛向屏幕右側方向被拖走，它的值為1。如果用戶在頁面1和頁面2間滑動到一半，那麼頁面1的position為-0.5並且頁面2的position為 0.5。根據屏幕上頁面的position，我們可以通過<a href="http://developer.android.com/reference/android/view/View.html#setAlpha(float)">`setAlpha()`</a>，<a href="http://developer.android.com/reference/android/view/View.html#setTranslationX(float)">`setTranslationX()`</a>或<a href="http://developer.android.com/reference/android/view/View.html#setScaleY(float)">`setScaleY()`</a>這些方法設定頁面屬性來自定義滑動動畫。

當我們實現了[`PageTransformer`](http://developer.android.com/reference/android/support/v4/view/ViewPager.PageTransformer.html)後，用我們的實現調用<a href="http://developer.android.com/reference/android/support/v4/view/ViewPager.html#setPageTransformer(boolean, android.support.v4.view.ViewPager.PageTransformer)">`setPageTransformer()`</a>來應用這些自定義動畫。例如，如果我們有一個叫做`ZoomOutPageTransformer`的[`PageTransformer`](http://developer.android.com/reference/android/support/v4/view/ViewPager.PageTransformer.html)，可以這樣設置自定義動畫：

```java
ViewPager mPager = (ViewPager) findViewById(R.id.pager);
...
mPager.setPageTransformer(true, new ZoomOutPageTransformer());
```

詳情查看[Zoom-out Page Transformer](#Zoom-out Page Transformer)和[Depth Page Transformer](#Depth Page Transformer)部分的 [`PageTransformer`](http://developer.android.com/reference/android/support/v4/view/ViewPager.PageTransformer.html)視頻和例子。

### Zoom-out Page Transformer

當在相鄰界面滑動時，這個Page Transformer使頁面收縮並褪色。當頁面越靠近中心，它將漸漸還原到正常大小並且圖像漸入。

<div style="
  background: transparent url(device_galaxynexus_blank_land_span8.png) no-repeat
scroll top left; padding: 26px 68px 38px 72px; overflow: hidden;">

<video style="width: 320px; height: 180px;" controls="" autoplay="">
    <source src="anim_page_transformer_zoomout.mp4" type="video/mp4">
    <source src="anim_page_transformer_zoomout.webm" type="video/webm">
    <source src="anim_page_transformer_zoomout.ogv" type="video/ogg">
</video>

</div>

```java
public class ZoomOutPageTransformer implements ViewPager.PageTransformer {
    private static final float MIN_SCALE = 0.85f;
    private static final float MIN_ALPHA = 0.5f;

    public void transformPage(View view, float position) {
        int pageWidth = view.getWidth();
        int pageHeight = view.getHeight();

        if (position < -1) { // [-Infinity,-1)
            // This page is way off-screen to the left.
            view.setAlpha(0);

        } else if (position <= 1) { // [-1,1]
            // Modify the default slide transition to shrink the page as well
            float scaleFactor = Math.max(MIN_SCALE, 1 - Math.abs(position));
            float vertMargin = pageHeight * (1 - scaleFactor) / 2;
            float horzMargin = pageWidth * (1 - scaleFactor) / 2;
            if (position < 0) {
                view.setTranslationX(horzMargin - vertMargin / 2);
            } else {
                view.setTranslationX(-horzMargin + vertMargin / 2);
            }

            // Scale the page down (between MIN_SCALE and 1)
            view.setScaleX(scaleFactor);
            view.setScaleY(scaleFactor);

            // Fade the page relative to its size.
            view.setAlpha(MIN_ALPHA +
                    (scaleFactor - MIN_SCALE) /
                    (1 - MIN_SCALE) * (1 - MIN_ALPHA));

        } else { // (1,+Infinity]
            // This page is way off-screen to the right.
            view.setAlpha(0);
        }
    }
}
```

### Depth Page Transformer

這個Page Transformer使用默認動畫的屏幕左滑動畫。但是為右滑使用一種“潛藏”效果的動畫。潛藏動畫將page淡出，並且線性縮小它。

<div style="
  background: transparent url(device_galaxynexus_blank_land_span8.png) no-repeat
scroll top left; padding: 26px 68px 38px 72px; overflow: hidden;">

<video style="width: 320px; height: 180px;" controls="" autoplay="">
    <source src="anim_page_transformer_depth.mp4" type="video/mp4">
    <source src="anim_page_transformer_depth.webm" type="video/webm">
    <source src="anim_page_transformer_depth.ogv" type="video/ogg">
</video>

</div>

> **注意：**在潛藏過程中，默認動畫（屏幕滑動）是仍舊發生的，所以你必須用負的X平移來抵消它。例如：

```java
view.setTranslationX(-1 * view.getWidth() * position);
```

下面的例子展示瞭如何抵消默認滑屏動畫：

```java
public class DepthPageTransformer implements ViewPager.PageTransformer {
    private static final float MIN_SCALE = 0.75f;

    public void transformPage(View view, float position) {
        int pageWidth = view.getWidth();

        if (position < -1) { // [-Infinity,-1)
            // This page is way off-screen to the left.
            view.setAlpha(0);

        } else if (position <= 0) { // [-1,0]
            // Use the default slide transition when moving to the left page
            view.setAlpha(1);
            view.setTranslationX(0);
            view.setScaleX(1);
            view.setScaleY(1);

        } else if (position <= 1) { // (0,1]
            // Fade the page out.
            view.setAlpha(1 - position);

            // Counteract the default slide transition
            view.setTranslationX(pageWidth * -position);

            // Scale the page down (between MIN_SCALE and 1)
            float scaleFactor = MIN_SCALE
                    + (1 - MIN_SCALE) * (1 - Math.abs(position));
            view.setScaleX(scaleFactor);
            view.setScaleY(scaleFactor);

        } else { // (1,+Infinity]
            // This page is way off-screen to the right.
            view.setAlpha(0);
        }
    }
}
```
