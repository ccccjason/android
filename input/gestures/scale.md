# 拖拽與縮放

> 編寫:[Andrwyw](https://github.com/Andrwyw) - 原文:<http://developer.android.com/training/gestures/scale.html>

本節課程講述，使用<a href="http://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent)">onTouchEvent()</a>截獲觸摸事件後，如何使用觸摸手勢拖拽、縮放屏幕上的對象。

## 拖拽一個對象

> 如果我們的目標版本為3.0或以上，我們可以使用[View.OnDragListener](http://developer.android.com/reference/android/view/View.OnDragListener.html)監聽內置的拖放（drag-and-drop）事件，[拖拽與釋放](http://developer.android.com/guide/topics/ui/drag-drop.html)中有更多相關描述。

對於觸摸手勢來說，一個很常見的操作是在屏幕上拖拽一個對象。接下來的代碼段讓用戶可以拖拽屏幕上的圖片。需要注意以下幾點：

- 拖拽操作時，即使有額外的手指放置到屏幕上了，app也必須保持對最初的點（手指）的追蹤。比如，想象在拖拽圖片時，用戶放置了第二根手指在屏幕上，並且擡起了第一根手指。如果我們的app只是單獨地追蹤每個點，它會把第二個點當做默認的點，並且把圖片移到該點的位置。
- 為了防止這種情況發生，我們的app需要區分初始點以及隨後任意的觸摸點。要做到這一點，它需要追蹤[**處理多觸摸手勢**](multi.html)章節中提到過的 [ACTION_POINTER_DOWN](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_POINTER_DOWN) 和 [ACTION_POINTER_UP](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_POINTER_UP) 事件。每當第二根手指按下或拿起時，[ACTION_POINTER_DOWN](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_POINTER_DOWN) 和 [ACTION_POINTER_UP](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_POINTER_UP) 事件就會傳遞給`onTouchEvent()`回調函數。
- 當[ACTION_POINTER_UP](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_POINTER_UP)事件發生時，示例程序會移除對該點的索引值的引用，確保操作中的點的ID(the active pointer ID)不會引用已經不在觸摸屏上的觸摸點。這種情況下，app會選擇另一個觸摸點來作為操作中(active)的點，並保存它當前的x、y值。由於在[ACTION_MOVE](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_MOVE)事件時，這個保存的位置會被用來計算屏幕上的對象將要移動的距離，所以app會始終根據正確的觸摸點來計算移動的距離。

下面的代碼段允許用戶拖拽屏幕上的對象。它會記錄操作中的點（active pointer）的初始位置，計算觸摸點移動過的距離，再把對象移動到新的位置。如上所述，它也正確地處理了額外觸摸點的可能。

需要注意的是，代碼段中使用了<a href="http://developer.android.com/reference/android/view/MotionEvent.html#getActionMasked()">getActionMasked()</a>函數。我們應該始終使用這個函數（或者最好用<a href="http://developer.android.com/reference/android/support/v4/view/MotionEventCompat.html#getActionMasked(android.view.MotionEvent)">MotionEventCompat.getActionMasked()</a>這個兼容版本）來獲得[MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html)對應的動作(action)。不像舊的<a href="http://developer.android.com/reference/android/view/MotionEvent.html#getAction()">getAction()</a>函數，`getActionMasked()`就是設計用來處理多點觸摸的。它會返回執行過的動作的掩碼值，不包括該點的索引位。

```java
// The ‘active pointer’ is the one currently moving our object.
private int mActivePointerId = INVALID_POINTER_ID;

@Override
public boolean onTouchEvent(MotionEvent ev) {
    // Let the ScaleGestureDetector inspect all events.
    mScaleDetector.onTouchEvent(ev);

    final int action = MotionEventCompat.getActionMasked(ev);

    switch (action) {
    case MotionEvent.ACTION_DOWN: {
        final int pointerIndex = MotionEventCompat.getActionIndex(ev);
        final float x = MotionEventCompat.getX(ev, pointerIndex);
        final float y = MotionEventCompat.getY(ev, pointerIndex);

        // Remember where we started (for dragging)
        mLastTouchX = x;
        mLastTouchY = y;
        // Save the ID of this pointer (for dragging)
        mActivePointerId = MotionEventCompat.getPointerId(ev, 0);
        break;
    }

    case MotionEvent.ACTION_MOVE: {
        // Find the index of the active pointer and fetch its position
        final int pointerIndex =
                MotionEventCompat.findPointerIndex(ev, mActivePointerId);

        final float x = MotionEventCompat.getX(ev, pointerIndex);
        final float y = MotionEventCompat.getY(ev, pointerIndex);

        // Calculate the distance moved
        final float dx = x - mLastTouchX;
        final float dy = y - mLastTouchY;

        mPosX += dx;
        mPosY += dy;

        invalidate();

        // Remember this touch position for the next move event
        mLastTouchX = x;
        mLastTouchY = y;

        break;
    }

    case MotionEvent.ACTION_UP: {
        mActivePointerId = INVALID_POINTER_ID;
        break;
    }

    case MotionEvent.ACTION_CANCEL: {
        mActivePointerId = INVALID_POINTER_ID;
        break;
    }

    case MotionEvent.ACTION_POINTER_UP: {

        final int pointerIndex = MotionEventCompat.getActionIndex(ev);
        final int pointerId = MotionEventCompat.getPointerId(ev, pointerIndex);

        if (pointerId == mActivePointerId) {
            // This was our active pointer going up. Choose a new
            // active pointer and adjust accordingly.
            final int newPointerIndex = pointerIndex == 0 ? 1 : 0;
            mLastTouchX = MotionEventCompat.getX(ev, newPointerIndex);
            mLastTouchY = MotionEventCompat.getY(ev, newPointerIndex);
            mActivePointerId = MotionEventCompat.getPointerId(ev, newPointerIndex);
        }
        break;
    }
    }
    return true;
}
```

## 通過拖拽平移

前一節展示了一個，在屏幕上拖拽對象的例子。另一個常見的場景是*平移*（*panning*），平移是指用戶通過拖拽移動引起x、y軸方向發生滾動(scrolling)。上面的代碼段直接截獲了[MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html)動作來實現拖拽。這一部分的代碼段，利用了平臺對常用手勢的內置支持。它重寫了[GestureDetector.SimpleOnGestureListener](http://developer.android.com/reference/android/view/GestureDetector.SimpleOnGestureListener.html)的<a href="http://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html#onScroll(android.view.MotionEvent, android.view.MotionEvent, float, float)">onScroll()</a>函數。

更詳細地說，當用戶拖拽手指來平移內容時，`onScroll()`函數就會被調用。`onScroll()`函數只會在手指按下的情況下被調用，一旦手指離開屏幕了，要麼手勢終止，要麼快速滑動(fling)手勢開始（如果手指在離開屏幕前快速移動了一段距離）。關於滾動與快速滑動的更多討論，可以查看[滾動手勢動畫](scroll.html)章節。

這裡是`onScroll()`的相關代碼段：

```java
// The current viewport. This rectangle represents the currently visible
// chart domain and range.
private RectF mCurrentViewport =
        new RectF(AXIS_X_MIN, AXIS_Y_MIN, AXIS_X_MAX, AXIS_Y_MAX);

// The current destination rectangle (in pixel coordinates) into which the
// chart data should be drawn.
private Rect mContentRect;

private final GestureDetector.SimpleOnGestureListener mGestureListener
            = new GestureDetector.SimpleOnGestureListener() {
...

@Override
public boolean onScroll(MotionEvent e1, MotionEvent e2,
            float distanceX, float distanceY) {
    // Scrolling uses math based on the viewport (as opposed to math using pixels).

    // Pixel offset is the offset in screen pixels, while viewport offset is the
    // offset within the current viewport.
    float viewportOffsetX = distanceX * mCurrentViewport.width()
            / mContentRect.width();
    float viewportOffsetY = -distanceY * mCurrentViewport.height()
            / mContentRect.height();
    ...
    // Updates the viewport, refreshes the display.
    setViewportBottomLeft(
            mCurrentViewport.left + viewportOffsetX,
            mCurrentViewport.bottom + viewportOffsetY);
    ...
    return true;
}
```

`onScroll()`函數中滑動視窗(viewport)來響應觸摸手勢的實現：

```java
/**
 * Sets the current viewport (defined by mCurrentViewport) to the given
 * X and Y positions. Note that the Y value represents the topmost pixel position,
 * and thus the bottom of the mCurrentViewport rectangle.
 */
private void setViewportBottomLeft(float x, float y) {
    /*
     * Constrains within the scroll range. The scroll range is simply the viewport
     * extremes (AXIS_X_MAX, etc.) minus the viewport size. For example, if the
     * extremes were 0 and 10, and the viewport size was 2, the scroll range would
     * be 0 to 8.
     */

    float curWidth = mCurrentViewport.width();
    float curHeight = mCurrentViewport.height();
    x = Math.max(AXIS_X_MIN, Math.min(x, AXIS_X_MAX - curWidth));
    y = Math.max(AXIS_Y_MIN + curHeight, Math.min(y, AXIS_Y_MAX));

    mCurrentViewport.set(x, y - curHeight, x + curWidth, y);

    // Invalidates the View to update the display.
    ViewCompat.postInvalidateOnAnimation(this);
}
```

## 使用觸摸手勢進行縮放

如同[檢測常用手勢](detector.html)章節中提到的，[GestureDetector](http://developer.android.com/reference/android/view/GestureDetector.html)可以幫助我們檢測Android中的常見手勢，例如滾動，快速滾動以及長按。對於縮放，Android也提供了[ScaleGestureDetector](http://developer.android.com/reference/android/view/ScaleGestureDetector.html)類。當我們想讓view能識別額外的手勢時，我們可以同時使用[GestureDetector](http://developer.android.com/reference/android/view/GestureDetector.html)和[ScaleGestureDetector](http://developer.android.com/reference/android/view/ScaleGestureDetector.html)類。

為了報告檢測到的手勢事件，手勢檢測需要一個作為構造函數參數的listener對象。[ScaleGestureDetector](1http://developer.android.com/reference/android/view/ScaleGestureDetector.html)使用[ScaleGestureDetector.OnScaleGestureListener](http://developer.android.com/reference/android/view/ScaleGestureDetector.OnScaleGestureListener.html)。Android提供了[ScaleGestureDetector.SimpleOnScaleGestureListener](http://developer.android.com/reference/android/view/ScaleGestureDetector.SimpleOnScaleGestureListener.html)類作為幫助類，如果我們不是關注所有的手勢事件，我們可以繼承(extend)它。

### 基本的縮放示例

下面的代碼段展示了縮放功能中的基本部分。

```java
private ScaleGestureDetector mScaleDetector;
private float mScaleFactor = 1.f;

public MyCustomView(Context mContext){
    ...
    // View code goes here
    ...
    mScaleDetector = new ScaleGestureDetector(context, new ScaleListener());
}

@Override
public boolean onTouchEvent(MotionEvent ev) {
    // Let the ScaleGestureDetector inspect all events.
    mScaleDetector.onTouchEvent(ev);
    return true;
}

@Override
public void onDraw(Canvas canvas) {
    super.onDraw(canvas);

    canvas.save();
    canvas.scale(mScaleFactor, mScaleFactor);
    ...
    // onDraw() code goes here
    ...
    canvas.restore();
}

private class ScaleListener
        extends ScaleGestureDetector.SimpleOnScaleGestureListener {
    @Override
    public boolean onScale(ScaleGestureDetector detector) {
        mScaleFactor *= detector.getScaleFactor();

        // Don't let the object get too small or too large.
        mScaleFactor = Math.max(0.1f, Math.min(mScaleFactor, 5.0f));

        invalidate();
        return true;
    }
}
```

### 更加複雜的縮放示例

這是本章節提供的`InteractiveChart`示例中一個更復雜的示範。通過使用[ScaleGestureDetector](http://developer.android.com/reference/android/view/ScaleGestureDetector.html)中的"span"(<a href="http://developer.android.com/reference/android/view/ScaleGestureDetector.html#getCurrentSpanX()">getCurrentSpanX/Y</a>)和"focus"(<a href="http://developer.android.com/reference/android/view/ScaleGestureDetector.html#getFocusX()">getFocusX/Y</a>)功能，`InteractiveChart`示例同時支持滾動（平移）以及多指縮放。

```java
@Override
private RectF mCurrentViewport =
        new RectF(AXIS_X_MIN, AXIS_Y_MIN, AXIS_X_MAX, AXIS_Y_MAX);
private Rect mContentRect;
private ScaleGestureDetector mScaleGestureDetector;
...
public boolean onTouchEvent(MotionEvent event) {
    boolean retVal = mScaleGestureDetector.onTouchEvent(event);
    retVal = mGestureDetector.onTouchEvent(event) || retVal;
    return retVal || super.onTouchEvent(event);
}

/**
 * The scale listener, used for handling multi-finger scale gestures.
 */
private final ScaleGestureDetector.OnScaleGestureListener mScaleGestureListener
        = new ScaleGestureDetector.SimpleOnScaleGestureListener() {
    /**
     * This is the active focal point in terms of the viewport. Could be a local
     * variable but kept here to minimize per-frame allocations.
     */
    private PointF viewportFocus = new PointF();
    private float lastSpanX;
    private float lastSpanY;

    // Detects that new pointers are going down.
    @Override
    public boolean onScaleBegin(ScaleGestureDetector scaleGestureDetector) {
        lastSpanX = ScaleGestureDetectorCompat.
                getCurrentSpanX(scaleGestureDetector);
        lastSpanY = ScaleGestureDetectorCompat.
                getCurrentSpanY(scaleGestureDetector);
        return true;
    }

    @Override
    public boolean onScale(ScaleGestureDetector scaleGestureDetector) {

        float spanX = ScaleGestureDetectorCompat.
                getCurrentSpanX(scaleGestureDetector);
        float spanY = ScaleGestureDetectorCompat.
                getCurrentSpanY(scaleGestureDetector);

        float newWidth = lastSpanX / spanX * mCurrentViewport.width();
        float newHeight = lastSpanY / spanY * mCurrentViewport.height();

        float focusX = scaleGestureDetector.getFocusX();
        float focusY = scaleGestureDetector.getFocusY();
        // Makes sure that the chart point is within the chart region.
        // See the sample for the implementation of hitTest().
        hitTest(scaleGestureDetector.getFocusX(),
                scaleGestureDetector.getFocusY(),
                viewportFocus);

        mCurrentViewport.set(
                viewportFocus.x
                        - newWidth * (focusX - mContentRect.left)
                        / mContentRect.width(),
                viewportFocus.y
                        - newHeight * (mContentRect.bottom - focusY)
                        / mContentRect.height(),
                0,
                0);
        mCurrentViewport.right = mCurrentViewport.left + newWidth;
        mCurrentViewport.bottom = mCurrentViewport.top + newHeight;
        ...
        // Invalidates the View to update the display.
        ViewCompat.postInvalidateOnAnimation(InteractiveLineGraphView.this);

        lastSpanX = spanX;
        lastSpanY = spanY;
        return true;
    }
};
```
