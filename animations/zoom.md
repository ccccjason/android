# 縮放View

> 編寫:[XizhiXu](https://github.com/XizhiXu) - 原文:<http://developer.android.com/training/animation/zoom.html>

這節課展示怎樣實現點擊縮放動畫，這對相冊很有用，他能為相片從縮略圖轉換成原圖並填充屏幕提供動畫。

下面展示了觸摸縮放動畫的效果，它將縮略圖擴大並填充屏幕。

<div style="
  background: transparent url(device_galaxynexus_blank_land_span8.png) no-repeat
scroll top left; padding: 26px 68px 38px 72px; overflow: hidden;">

<video style="width: 320px; height: 180px;" controls="" autoplay="">
    <source src="anim_zoom.mp4" type="video/mp4">
    <source src="anim_zoom.webm" type="video/webm">
    <source src="anim_zoom.ogv" type="video/ogg">
</video>

</div>

如果你想直接查看整個例子，[下載](http://developer.android.com/shareables/training/Animations.zip)並運行App樣例然後選擇縮放的例子。查看下列文件中的代碼實現：

* `src/TouchHighlightImageButton.java`（簡單的helper類，當Image Button被按下它顯示藍色高亮）
* `src/ZoomActivity.java`
* `layout/activity_zoom.xml`

## 創建View

為想要縮放的內容創建一大一小兩個版本佈局文件。下面的例子為可點擊的縮略圖新建了一個[`ImageButton`](http://developer.android.com/reference/android/widget/ImageButton.html)和一個[`ImageView`](http://developer.android.com/reference/android/widget/ImageView.html)來展示原圖：

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">

        <ImageButton
            android:id="@+id/thumb_button_1"
            android:layout_width="100dp"
            android:layout_height="75dp"
            android:layout_marginRight="1dp"
            android:src="@drawable/thumb1"
            android:scaleType="centerCrop"
            android:contentDescription="@string/description_image_1" />

    </LinearLayout>

    <!-- This initially-hidden ImageView will hold the expanded/zoomed version of
         the images above. Without transformations applied, it takes up the entire
         screen. To achieve the "zoom" animation, this view's bounds are animated
         from the bounds of the thumbnail button above, to its final laid-out
         bounds.
         -->

    <ImageView
        android:id="@+id/expanded_image"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:visibility="invisible"
        android:contentDescription="@string/description_zoom_touch_close" />

</FrameLayout>
```

## 設置縮放動畫

一旦實現了佈局，我們需要設置觸發縮放動畫的事件handler。下面的例子為[`ImageButton`](http://developer.android.com/reference/android/widget/ImageButton.html)添加了一個[`View.OnClickListener`](http://developer.android.com/reference/android/view/View.OnClickListener.html)，當用戶點擊按鈕時它執行放大動畫。

```java
public class ZoomActivity extends FragmentActivity {
    // Hold a reference to the current animator,
    // so that it can be canceled mid-way.
    private Animator mCurrentAnimator;

    // The system "short" animation time duration, in milliseconds. This
    // duration is ideal for subtle animations or animations that occur
    // very frequently.
    private int mShortAnimationDuration;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_zoom);

        // Hook up clicks on the thumbnail views.

        final View thumb1View = findViewById(R.id.thumb_button_1);
        thumb1View.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                zoomImageFromThumb(thumb1View, R.drawable.image1);
            }
        });

        // Retrieve and cache the system's default "short" animation time.
        mShortAnimationDuration = getResources().getInteger(
                android.R.integer.config_shortAnimTime);
    }
    ...
}
```

## 縮放View

我們現在需要適時應用放大動畫了。通常來說，我們需要按邊界來從小號View放大到大號View。下面的方法展示瞭如何實現縮放動畫：

1. 把高清圖像資源設置到已經被隱藏的“放大版”的[`ImageView`](http://developer.android.com/reference/android/widget/ImageView.html)中。為表簡單，下面的例子在UI線程中加載了一張大圖。但是我們需要在一個單獨的線程中來加載以免阻塞UI線程，然後再回到UI線程中設置。理想狀況下，圖片不要大過屏幕尺寸。

2. 計算[`ImageView`](http://developer.android.com/reference/android/widget/ImageView.html)開始和結束時的邊界。

3. 從起始邊到結束邊同步地動態改變四個位置和大小屬性[`X`](http://developer.android.com/reference/android/view/View.html#X)，[`Y`](http://developer.android.com/reference/android/view/View.html#Y)（[`SCALE_X`](http://developer.android.com/reference/android/view/View.html#SCALE_X) 和 [`SCALE_Y`](http://developer.android.com/reference/android/view/View.html#SCALE_Y)）。這四個動畫被加入到了[`AnimatorSet`](http://developer.android.com/reference/android/animation/AnimatorSet.html)，所以我們可以讓它們一起開始。

4. 縮小則運行相同的動畫，但是是在用戶點擊屏幕放大時的逆向效果。我們可以在[`ImageView`](http://developer.android.com/reference/android/widget/ImageView.html)中添加一個[`View.OnClickListener`](http://developer.android.com/reference/android/view/View.OnClickListener.html)來實現它。當點擊時，[`ImageView`](http://developer.android.com/reference/android/widget/ImageView.html)縮回到原來縮略圖的大小，然後設置它的visibility為[`GONE`](http://developer.android.com/reference/android/view/View.html#GONE)來隱藏。

```java
private void zoomImageFromThumb(final View thumbView, int imageResId) {
    // If there's an animation in progress, cancel it
    // immediately and proceed with this one.
    if (mCurrentAnimator != null) {
        mCurrentAnimator.cancel();
    }

    // Load the high-resolution "zoomed-in" image.
    final ImageView expandedImageView = (ImageView) findViewById(
            R.id.expanded_image);
    expandedImageView.setImageResource(imageResId);

    // Calculate the starting and ending bounds for the zoomed-in image.
    // This step involves lots of math. Yay, math.
    final Rect startBounds = new Rect();
    final Rect finalBounds = new Rect();
    final Point globalOffset = new Point();

    // The start bounds are the global visible rectangle of the thumbnail,
    // and the final bounds are the global visible rectangle of the container
    // view. Also set the container view's offset as the origin for the
    // bounds, since that's the origin for the positioning animation
    // properties (X, Y).
    thumbView.getGlobalVisibleRect(startBounds);
    findViewById(R.id.container)
            .getGlobalVisibleRect(finalBounds, globalOffset);
    startBounds.offset(-globalOffset.x, -globalOffset.y);
    finalBounds.offset(-globalOffset.x, -globalOffset.y);

    // Adjust the start bounds to be the same aspect ratio as the final
    // bounds using the "center crop" technique. This prevents undesirable
    // stretching during the animation. Also calculate the start scaling
    // factor (the end scaling factor is always 1.0).
    float startScale;
    if ((float) finalBounds.width() / finalBounds.height()
            > (float) startBounds.width() / startBounds.height()) {
        // Extend start bounds horizontally
        startScale = (float) startBounds.height() / finalBounds.height();
        float startWidth = startScale * finalBounds.width();
        float deltaWidth = (startWidth - startBounds.width()) / 2;
        startBounds.left -= deltaWidth;
        startBounds.right += deltaWidth;
    } else {
        // Extend start bounds vertically
        startScale = (float) startBounds.width() / finalBounds.width();
        float startHeight = startScale * finalBounds.height();
        float deltaHeight = (startHeight - startBounds.height()) / 2;
        startBounds.top -= deltaHeight;
        startBounds.bottom += deltaHeight;
    }

    // Hide the thumbnail and show the zoomed-in view. When the animation
    // begins, it will position the zoomed-in view in the place of the
    // thumbnail.
    thumbView.setAlpha(0f);
    expandedImageView.setVisibility(View.VISIBLE);

    // Set the pivot point for SCALE_X and SCALE_Y transformations
    // to the top-left corner of the zoomed-in view (the default
    // is the center of the view).
    expandedImageView.setPivotX(0f);
    expandedImageView.setPivotY(0f);

    // Construct and run the parallel animation of the four translation and
    // scale properties (X, Y, SCALE_X, and SCALE_Y).
    AnimatorSet set = new AnimatorSet();
    set
            .play(ObjectAnimator.ofFloat(expandedImageView, View.X,
                    startBounds.left, finalBounds.left))
            .with(ObjectAnimator.ofFloat(expandedImageView, View.Y,
                    startBounds.top, finalBounds.top))
            .with(ObjectAnimator.ofFloat(expandedImageView, View.SCALE_X,
            startScale, 1f)).with(ObjectAnimator.ofFloat(expandedImageView,
                    View.SCALE_Y, startScale, 1f));
    set.setDuration(mShortAnimationDuration);
    set.setInterpolator(new DecelerateInterpolator());
    set.addListener(new AnimatorListenerAdapter() {
        @Override
        public void onAnimationEnd(Animator animation) {
            mCurrentAnimator = null;
        }

        @Override
        public void onAnimationCancel(Animator animation) {
            mCurrentAnimator = null;
        }
    });
    set.start();
    mCurrentAnimator = set;

    // Upon clicking the zoomed-in image, it should zoom back down
    // to the original bounds and show the thumbnail instead of
    // the expanded image.
    final float startScaleFinal = startScale;
    expandedImageView.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            if (mCurrentAnimator != null) {
                mCurrentAnimator.cancel();
            }

            // Animate the four positioning/sizing properties in parallel,
            // back to their original values.
            AnimatorSet set = new AnimatorSet();
            set.play(ObjectAnimator
                        .ofFloat(expandedImageView, View.X, startBounds.left))
                        .with(ObjectAnimator
                                .ofFloat(expandedImageView,
                                        View.Y,startBounds.top))
                        .with(ObjectAnimator
                                .ofFloat(expandedImageView,
                                        View.SCALE_X, startScaleFinal))
                        .with(ObjectAnimator
                                .ofFloat(expandedImageView,
                                        View.SCALE_Y, startScaleFinal));
            set.setDuration(mShortAnimationDuration);
            set.setInterpolator(new DecelerateInterpolator());
            set.addListener(new AnimatorListenerAdapter() {
                @Override
                public void onAnimationEnd(Animator animation) {
                    thumbView.setAlpha(1f);
                    expandedImageView.setVisibility(View.GONE);
                    mCurrentAnimator = null;
                }

                @Override
                public void onAnimationCancel(Animator animation) {
                    thumbView.setAlpha(1f);
                    expandedImageView.setVisibility(View.GONE);
                    mCurrentAnimator = null;
                }
            });
            set.start();
            mCurrentAnimator = set;
        }
    });
}
```
