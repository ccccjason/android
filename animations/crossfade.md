# View間漸變

> 編寫:[XizhiXu](https://github.com/XizhiXu) - 原文:<http://developer.android.com/training/animation/crossfade.html>

漸變動畫（也叫消失）通常指漸漸的淡出某個UI組件，同時同步地淡入另一個。當App想切換內容或View的情況下，這種動畫很有用。漸變簡短不易察覺，同時又提供從一個界面到下一個之間流暢的轉換。如果在需要轉換的時候沒有使用任何動畫效果，這會使得轉換看上去感到生硬而倉促。

下面是一個利用進度指示漸變到一些文本內容的例子。

<div style="
  background: transparent url(device_galaxynexus_blank_land_span8.png) no-repeat
scroll top left; padding: 26px 68px 38px 72px; overflow: hidden;">

<video style="width: 320px; height: 180px;" controls="" autoplay="">
    <source src="anim_crossfade.mp4" type="video/mp4">
    <source src="anim_crossfade.webm" type="video/webm">
    <source src="anim_crossfade.ogv" type="video/ogg">
</video>

</div>


如果你想跳過這部分介紹直接查看樣例，[下載](http://developer.android.com/shareables/training/Animations.zip)並運行樣例App然後選擇漸變例子。查看下列文件中的代碼實現：

* `src/CrossfadeActivity.java`
* `layout/activity_crossfade.xml`
* `menu/activity_crossfade.xml`

## 創建View

創建兩個我們想相互漸變的View。下面的例子創建了一個進度提示圈和可滑動文本View。

```xml
<FrameLayout xmlns:android="/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ScrollView xmlns:android="/apk/res/android"
        android:id="@+id/content"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <TextView style="?android:textAppearanceMedium"
            android:lineSpacingMultiplier="1.2"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/lorem_ipsum"
            android:padding="16dp" />

    </ScrollView>

    <ProgressBar android:id="@+id/loading_spinner"
        style="?android:progressBarStyleLarge"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center" />

</FrameLayout>
```
## 設置動畫

為設置動畫，我們需要按照如下步驟來做：

1. 為我們想漸變的View 創建成員變量。在之後動畫應用途中修改View的時候我們會需要這些引用。

2. 對於被淡入的View，設置它的visibility為[`GONE`](http://developer.android.com/reference/android/view/View.html#GONE)。這樣防止view再佔據佈局的空間，而且也能在佈局計算中將其忽略，加速處理過程。

3. 將[`config_shortAnimTime`](http://developer.android.com/reference/android/R.integer.html#config_shortAnimTime)系統屬性暫存到一個成員變量裡。這個屬性為動畫定義了一個標準的“短”持續時間。對於細微或者快速發生的動畫，這是個很理想的持續時段。也可以根據實際需求使用[`config_longAnimTime`](http://developer.android.com/reference/android/R.integer.html#config_longAnimTime)或[`config_mediumAnimTime`](http://developer.android.com/reference/android/R.integer.html#config_mediumAnimTime)。

下面的例子使用了前文提到的佈局文件：

```java
public class CrossfadeActivity extends Activity {

    private View mContentView;
    private View mLoadingView;
    private int mShortAnimationDuration;

    ...

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_crossfade);

        mContentView = findViewById(R.id.content);
        mLoadingView = findViewById(R.id.loading_spinner);

        // Initially hide the content view.
        mContentView.setVisibility(View.GONE);

        // Retrieve and cache the system's default "short" animation time.
        mShortAnimationDuration = getResources().getInteger(
                android.R.integer.config_shortAnimTime);
    }
```

## 漸變View

進行了上述配置之後，接下來就讓我們實現漸變動畫吧：

1. 對於正在淡入的View，設置它的alpha值為0並且設置visibility為 [`VISIBLE`](http://developer.android.com/reference/android/view/View.html#VISIBLE)（記住他起初被設置成了 [`GONE`](http://developer.android.com/reference/android/view/View.html#GONE)）。這樣View就變成可見的了，但是此時它是透明的。

2. 對於正在淡入的View，把alpha值從0動態改變到1。同時，對於淡出的View，把alpha值從1動態變到0。

3. 使用[`Animator.AnimatorListener`](http://developer.android.com/reference/android/animation/Animator.AnimatorListener.html)中的 <a href="http://developer.android.com/reference/android/animation/Animator.AnimatorListener.html#onAnimationEnd(android.animation.Animator)">`onAnimationEnd()`</a>，設置淡出View的visibility為[`GONE`](http://developer.android.com/reference/android/view/View.html#GONE)。即使alpha值為0，也要把View的visibility設置成[`GONE`](http://developer.android.com/reference/android/view/View.html#GONE)來防止 view 佔據佈局空間，還能把它從佈局計算中忽略，加速處理過程。

詳見下面的例子：

```java
private View mContentView;
private View mLoadingView;
private int mShortAnimationDuration;

...

private void crossfade() {

    // Set the content view to 0% opacity but visible, so that it is visible
    // (but fully transparent) during the animation.
    mContentView.setAlpha(0f);
    mContentView.setVisibility(View.VISIBLE);

    // Animate the content view to 100% opacity, and clear any animation
    // listener set on the view.
    mContentView.animate()
            .alpha(1f)
            .setDuration(mShortAnimationDuration)
            .setListener(null);

    // Animate the loading view to 0% opacity. After the animation ends,
    // set its visibility to GONE as an optimization step (it won't
    // participate in layout passes, etc.)
    mLoadingView.animate()
            .alpha(0f)
            .setDuration(mShortAnimationDuration)
            .setListener(new AnimatorListenerAdapter() {
                @Override
                public void onAnimationEnd(Animator animation) {
                    mLoadingView.setVisibility(View.GONE);
                }
            });
}
```
