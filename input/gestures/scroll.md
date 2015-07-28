# 滾動手勢動畫

> 編寫:[Andrwyw](https://github.com/Andrwyw) - 原文:<http://developer.android.com/training/gestures/scroll.html>

在Android中，通常使用[ScrollView](http://developer.android.com/reference/android/widget/ScrollView.html)類來實現滾動（scroll）。任何可能超過父類邊界的佈局，都應該嵌套在[ScrollView](http://developer.android.com/reference/android/widget/ScrollView.html)中，來提供一個由系統框架管理的可滾動的view。僅在某些特殊情形下，我們才要實現一個自定義scroller。本節課程就描述了這樣一個情形：使用 *scrollers* 顯示滾動效果，以響應觸摸手勢。

為了收集數據來產生滾動動畫，以響應一個觸摸事件，我們可以使用scrollers（[Scroller](http://developer.android.com/reference/android/widget/Scroller.html)或者[OverScroller](http://developer.android.com/reference/android/widget/OverScroller.html)）。這兩個類很相似，但[OverScroller](http://developer.android.com/reference/android/widget/OverScroller.html)有一些函數，能在平移或快速滑動手勢後，向用戶指出已經達到內容的邊緣。`InteractiveChart` 例子使用了[EdgeEffect](http://developer.android.com/reference/android/widget/EdgeEffect.html)類（實際上是[EdgeEffectCompat](http://developer.android.com/reference/android/support/v4/widget/EdgeEffectCompat.html)類），在用戶到達內容的邊緣時顯示“發光”效果。

>**Note:** 比起Scroller類，我們更推薦使用[OverScroller](http://developer.android.com/reference/android/widget/OverScroller.html)類來產生滾動動畫。[OverScroller](http://developer.android.com/reference/android/widget/OverScroller.html)類為老設備提供了很好的向後兼容性。
>另外需要注意的是，僅當我們要自己實現滾動時，才需要使用scrollers。如果我們把佈局嵌套在[ScrollView](http://developer.android.com/reference/android/widget/ScrollView.html)和[HorizontalScrollView](http://developer.android.com/reference/android/widget/HorizontalScrollView.html)中，它們會幫我們把這些做好。

通過使用平臺標準的滾動物理因素（摩擦、速度等），scroller被用來隨著時間的推移產生滾動動畫。實際上，scroller本身不會繪製任何東西。Scrollers只是隨著時間的推移，追蹤滾動的偏移量，但它們不會自動地把這些位置應用到view上。我們應該按一定頻率，獲取並應用這些新的座標值，來讓滾動動畫更加順滑。

## 理解滾動術語

在Android中，“Scrolling”這個詞根據不同情景有著不同的含義。

**滾動**（Scrolling）是指移動視窗（viewport）（指你正在看的內容所在的‘窗口’）的一般過程。當在x軸和y軸方向同時滾動時，就叫做*平移*（*panning*）。示例程序提供的 `InteractiveChart` 類，展示了兩種不同類型的滾動，拖拽與快速滑動。

- **拖拽**（dragging）是滾動的一種類型，當用戶在觸摸屏上拖動手指時發生。簡單的拖拽一般可以通過重寫 [GestureDetector.OnGestureListener](http://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html) 的 <a href="http://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html#onScroll(android.view.MotionEvent,android.view.MotionEvent,float,float)">onScroll()</a> 來實現。關於拖拽的更多討論，可以查看[**拖拽與縮放**](scale.html)章節。
- **快速滑動**（fling）這種類型的滾動，在用戶快速拖拽後，擡起手指時發生。當用戶擡起手指後，我們通常想繼續保持滾動（移動視窗），但會一直減速直到視窗停止移動。通過重寫[GestureDetector.OnGestureListener](http://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html)的<a href="http://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html#onFling(android.view.MotionEvent,android.view.MotionEvent,float,float)">onFling()</a>函數，使用scroller對象，可實現快速滑動。這種用法也就是本節課程的主題。

scroller對象通常會與快速滑動手勢結合起來使用。但在任何我們想讓UI展示滾動動畫，以響應觸摸事件的場景，都可以用scroller對象來實現。比如，我們可以重寫<a href="http://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent)">onTouchEvent()</a>函數，直接處理觸摸事件，並且產生一個滾動效果或“頁面對齊”動畫(snapping to page)，來響應這些觸摸事件。

## 實現基於觸摸的滾動

本節講述如何使用scroller。下面的代碼段來自 `InteractiveChart` 示例。它使用[GestureDetector](http://developer.android.com/reference/android/view/GestureDetector.html)，並且重寫了[GestureDetector.SimpleOnGestureListener](http://developer.android.com/reference/android/view/GestureDetector.SimpleOnGestureListener.html)的 `onFling()` 函數。它使用[OverScroller](http://developer.android.com/reference/android/widget/OverScroller.html)追蹤快速滑動（fling）手勢。快速滑動手勢後，如果用戶到達內容邊緣，應用會顯示一種發光效果。

> **Note:** `InteractiveChart`示例程序展示了一個可縮放、平移、滑動的表格。在接下來的代碼段中，`mContentRect`表示view中的一塊矩形座標區域，該區域將被用來繪製表格。在任意給定的時間點，表格中某一部分會被繪製在這個區域內。`mCurrentViewport`表示當前在屏幕上可見的那一部分表格。因為像素偏移量通常當作整型處理，所以`mContentRect`是[Rect](http://developer.android.com/reference/android/graphics/Rect.html)類型的。因為圖表的區域範圍是數值型/浮點型值，所以`mCurrentViewport`是[RectF](http://developer.android.com/reference/android/graphics/RectF.html)類型。

代碼段的第一部分展示了`onFling()`函數的實現：

```java
// The current viewport. This rectangle represents the currently visible 
// chart domain and range. The viewport is the part of the app that the
// user manipulates via touch gestures.
private RectF mCurrentViewport =
        new RectF(AXIS_X_MIN, AXIS_Y_MIN, AXIS_X_MAX, AXIS_Y_MAX);

// The current destination rectangle (in pixel coordinates) into which the
// chart data should be drawn.
private Rect mContentRect;

private OverScroller mScroller;
private RectF mScrollerStartViewport;
...
private final GestureDetector.SimpleOnGestureListener mGestureListener
        = new GestureDetector.SimpleOnGestureListener() {
    @Override
    public boolean onDown(MotionEvent e) {
        // Initiates the decay phase of any active edge effects.
        releaseEdgeEffects();
        mScrollerStartViewport.set(mCurrentViewport);
        // Aborts any active scroll animations and invalidates.
        mScroller.forceFinished(true);
        ViewCompat.postInvalidateOnAnimation(InteractiveLineGraphView.this);
        return true;
    }
    ...
    @Override
    public boolean onFling(MotionEvent e1, MotionEvent e2,
            float velocityX, float velocityY) {
        fling((int) -velocityX, (int) -velocityY);
        return true;
    }
};

private void fling(int velocityX, int velocityY) {
    // Initiates the decay phase of any active edge effects.
    releaseEdgeEffects();
    // Flings use math in pixels (as opposed to math based on the viewport).
    Point surfaceSize = computeScrollSurfaceSize();
    mScrollerStartViewport.set(mCurrentViewport);
    int startX = (int) (surfaceSize.x * (mScrollerStartViewport.left -
            AXIS_X_MIN) / (
            AXIS_X_MAX - AXIS_X_MIN));
    int startY = (int) (surfaceSize.y * (AXIS_Y_MAX -
            mScrollerStartViewport.bottom) / (
            AXIS_Y_MAX - AXIS_Y_MIN));
    // Before flinging, aborts the current animation.
    mScroller.forceFinished(true);
    // Begins the animation
    mScroller.fling(
            // Current scroll position
            startX,
            startY,
            velocityX,
            velocityY,
            /*
             * Minimum and maximum scroll positions. The minimum scroll
             * position is generally zero and the maximum scroll position
             * is generally the content size less the screen size. So if the
             * content width is 1000 pixels and the screen width is 200
             * pixels, the maximum scroll offset should be 800 pixels.
             */
            0, surfaceSize.x - mContentRect.width(),
            0, surfaceSize.y - mContentRect.height(),
            // The edges of the content. This comes into play when using
            // the EdgeEffect class to draw "glow" overlays.
            mContentRect.width() / 2,
            mContentRect.height() / 2);
    // Invalidates to trigger computeScroll()
    ViewCompat.postInvalidateOnAnimation(this);
}
```

當`onFling()`函數調用<a href="http://developer.android.com/reference/android/support/v4/view/ViewCompat.html#postInvalidateOnAnimation(android.view.View)">postInvalidateOnAnimation()</a>時，它會觸發<a href="http://developer.android.com/reference/android/view/View.html#computeScroll()">computeScroll()</a>來更新x、y的值。通常一個子view用scroller對象來產生滾動動畫時會這樣做，就像本例一樣。

大多數views直接通過<a href="http://developer.android.com/reference/android/view/View.html#scrollTo(int,int)">scrollTo()</a>函數傳遞scroller對象的x、y座標值。接下來的`computeScroll()`函數的實現中採用了一種不同的方式。它調用<a href="http://developer.android.com/reference/android/widget/OverScroller.html#computeScrollOffset()">computeScrollOffset()</a>函數來獲得當前位置的x、y值。當滿足邊緣顯示發光效果的條件時（圖表已被放大顯示，x或y值超過邊界，並且app當前沒有顯示overscroll），這段代碼會設置overscroll發光效果，並調用`postInvalidateOnAnimation()`函數來讓view失效重繪：

```java
// Edge effect / overscroll tracking objects.
private EdgeEffectCompat mEdgeEffectTop;
private EdgeEffectCompat mEdgeEffectBottom;
private EdgeEffectCompat mEdgeEffectLeft;
private EdgeEffectCompat mEdgeEffectRight;

private boolean mEdgeEffectTopActive;
private boolean mEdgeEffectBottomActive;
private boolean mEdgeEffectLeftActive;
private boolean mEdgeEffectRightActive;

@Override
public void computeScroll() {
    super.computeScroll();

    boolean needsInvalidate = false;

    // The scroller isn't finished, meaning a fling or programmatic pan
    // operation is currently active.
    if (mScroller.computeScrollOffset()) {
        Point surfaceSize = computeScrollSurfaceSize();
        int currX = mScroller.getCurrX();
        int currY = mScroller.getCurrY();

        boolean canScrollX = (mCurrentViewport.left > AXIS_X_MIN
                || mCurrentViewport.right < AXIS_X_MAX);
        boolean canScrollY = (mCurrentViewport.top > AXIS_Y_MIN
                || mCurrentViewport.bottom < AXIS_Y_MAX);

        /*
         * If you are zoomed in and currX or currY is
         * outside of bounds and you're not already
         * showing overscroll, then render the overscroll
         * glow edge effect.
         */
        if (canScrollX
                && currX < 0
                && mEdgeEffectLeft.isFinished()
                && !mEdgeEffectLeftActive) {
            mEdgeEffectLeft.onAbsorb((int)
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectLeftActive = true;
            needsInvalidate = true;
        } else if (canScrollX
                && currX > (surfaceSize.x - mContentRect.width())
                && mEdgeEffectRight.isFinished()
                && !mEdgeEffectRightActive) {
            mEdgeEffectRight.onAbsorb((int)
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectRightActive = true;
            needsInvalidate = true;
        }

        if (canScrollY
                && currY < 0
                && mEdgeEffectTop.isFinished()
                && !mEdgeEffectTopActive) {
            mEdgeEffectTop.onAbsorb((int)
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectTopActive = true;
            needsInvalidate = true;
        } else if (canScrollY
                && currY > (surfaceSize.y - mContentRect.height())
                && mEdgeEffectBottom.isFinished()
                && !mEdgeEffectBottomActive) {
            mEdgeEffectBottom.onAbsorb((int)
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectBottomActive = true;
            needsInvalidate = true;
        }
        ...
    }
```

這是縮放部分的代碼：

```java
// Custom object that is functionally similar to Scroller
Zoomer mZoomer;
private PointF mZoomFocalPoint = new PointF();
...

// If a zoom is in progress (either programmatically or via double
// touch), performs the zoom.
if (mZoomer.computeZoom()) {
    float newWidth = (1f - mZoomer.getCurrZoom()) *
            mScrollerStartViewport.width();
    float newHeight = (1f - mZoomer.getCurrZoom()) *
            mScrollerStartViewport.height();
    float pointWithinViewportX = (mZoomFocalPoint.x -
            mScrollerStartViewport.left)
            / mScrollerStartViewport.width();
    float pointWithinViewportY = (mZoomFocalPoint.y -
            mScrollerStartViewport.top)
            / mScrollerStartViewport.height();
    mCurrentViewport.set(
            mZoomFocalPoint.x - newWidth * pointWithinViewportX,
            mZoomFocalPoint.y - newHeight * pointWithinViewportY,
            mZoomFocalPoint.x + newWidth * (1 - pointWithinViewportX),
            mZoomFocalPoint.y + newHeight * (1 - pointWithinViewportY));
    constrainViewport();
    needsInvalidate = true;
}
if (needsInvalidate) {
    ViewCompat.postInvalidateOnAnimation(this);
}

```

這是上面代碼段中調用過的`computeScrollSurfaceSize()`函數。它會以像素為單位計算當前可滾動的尺寸。舉例來說，如果整個圖表區域都是可見的，它的值就簡單地等於`mContentRect`的大小。如果圖表在兩個方向上都放大到200%，此函數返回的尺寸在水平、垂直方向上都會大兩倍。

```java
private Point computeScrollSurfaceSize() {
    return new Point(
            (int) (mContentRect.width() * (AXIS_X_MAX - AXIS_X_MIN)
                    / mCurrentViewport.width()),
            (int) (mContentRect.height() * (AXIS_Y_MAX - AXIS_Y_MIN)
                    / mCurrentViewport.height()));
}
```

關於scroller用法的另一個示例，可查看[ViewPager](http://developer.android.com/reference/android/support/v4/view/ViewPager.html)類的[源代碼](http://github.com/android/platform_frameworks_support/blob/master/v4/java/android/support/v4/view/ViewPager.java)。它用滾動來響應快速滑動（fling），並且使用滾動來實現“頁面對齊”(snapping to page)動畫。
