# 管理ViewGroup中的觸摸事件

> 編寫:[Andrwyw](https://github.com/Andrwyw) - 原文:<http://developer.android.com/training/gestures/viewgroup.html>

因為很多時候是用ViewGroup的子類來做不同觸摸事件的目標，而不是ViewGroup本身，所以處理[ViewGroup](http://developer.android.com/reference/android/view/ViewGroup.html)中的觸摸事件需要特別注意。
為了確保每個view能正確地接收到它們想要的觸摸事件，可以重寫<a href="http://developer.android.com/reference/android/view/ViewGroup.html#onInterceptTouchEvent(android.view.MotionEvent)">onInterceptTouchEvent()</a>函數。

## 在ViewGroup中截獲觸摸事件

每當在[ViewGroup](http://developer.android.com/reference/android/view/ViewGroup.html)（包括它的子View）的表面上檢測到一個觸摸事件，<a href="http://developer.android.com/reference/android/view/ViewGroup.html#onInterceptTouchEvent(android.view.MotionEvent)">onInterceptTouchEvent()</a>都會被調用。如果`onInterceptTouchEvent()`返回`true`，[MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html)就被截獲了，這表示它不會被傳遞給其子View，而是傳遞給該父view自身的<a href="http://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent)">onTouchEvent()</a>方法。

`onInterceptTouchEvent()`方法讓父view能夠在它的子view之前處理觸摸事件。如果我們讓`onInterceptTouchEvent()`返回`true`，則之前處理觸摸事件的子view會收到[ACTION_CANCEL](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_CANCEL)事件，並且該點之後的事件會被髮送給該父view自身的`onTouchEvent()`函數，進行常規處理。`onInterceptTouchEvent()`也可以返回`false`，這樣事件沿view層級分發到目標前，父view可以簡單地觀察該事件。這裡的目標是指，通過`onTouchEvent()`處理消息事件的view。

接下來的代碼段中，`MyViewGroup`繼承自[ViewGroup](http://developer.android.com/reference/android/view/ViewGroup.html)。`MyViewGroup`有多個子view。如果我們在某個子View上水平地拖動手指，該子view不會接收到觸摸事件，而是應該由`MyViewGroup`處理這些觸摸事件來滾動它的內容。然而，如果我們點擊子view中的button，或垂直地滾動子view，則父view不會截獲這些觸摸事件，因為子view本身就是預定目標。在這些情況下，`onInterceptTouchEvent()`應該返回`false`，`MyViewGroup`的`onTouchEvent()`也不會被調用。

```java
public class MyViewGroup extends ViewGroup {

    private int mTouchSlop;

    ...

    ViewConfiguration vc = ViewConfiguration.get(view.getContext());
    mTouchSlop = vc.getScaledTouchSlop();

    ...

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        /*
         * This method JUST determines whether we want to intercept the motion.
         * If we return true, onTouchEvent will be called and we do the actual
         * scrolling there.
         */


        final int action = MotionEventCompat.getActionMasked(ev);

        // Always handle the case of the touch gesture being complete.
        if (action == MotionEvent.ACTION_CANCEL || action == MotionEvent.ACTION_UP) {
            // Release the scroll.
            mIsScrolling = false;
            return false; // Do not intercept touch event, let the child handle it
        }

        switch (action) {
            case MotionEvent.ACTION_MOVE: {
                if (mIsScrolling) {
                    // We're currently scrolling, so yes, intercept the
                    // touch event!
                    return true;
                }

                // If the user has dragged her finger horizontally more than
                // the touch slop, start the scroll

                // left as an exercise for the reader
                final int xDiff = calculateDistanceX(ev);

                // Touch slop should be calculated using ViewConfiguration
                // constants.
                if (xDiff > mTouchSlop) {
                    // Start scrolling!
                    mIsScrolling = true;
                    return true;
                }
                break;
            }
            ...
        }

        // In general, we don't want to intercept touch events. They should be
        // handled by the child view.
        return false;
    }

    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        // Here we actually handle the touch event (e.g. if the action is ACTION_MOVE,
        // scroll this container).
        // This method will only be called if the touch event was intercepted in
        // onInterceptTouchEvent
        ...
    }
}
```

注意[ViewGroup](http://developer.android.com/reference/android/view/ViewGroup.html)也提供了<a href="http://developer.android.com/reference/android/view/ViewGroup.html#requestDisallowInterceptTouchEvent(boolean)">requestDisallowInterceptTouchEvent()</a>方法。當子view不想該父view和祖先view通過`onInterceptTouchEvent()`截獲它的觸摸事件時，可調用[ViewGroup](http://developer.android.com/reference/android/view/ViewGroup.html)的該方法。

## 使用ViewConfiguration的常量

上面的代碼段中使用了當前的[ViewConfiguration](http://developer.android.com/reference/android/view/ViewConfiguration.html)來初始化`mTouchSlop`變量。我們可以使用[ViewConfiguration](http://developer.android.com/reference/android/view/ViewConfiguration.html)類來獲取Android系統常用的一些距離、速度、時間值。

“Touch slop”是指在被識別為移動的手勢前，用戶觸摸可移動的那一段像素距離。Touch slop通常用來預防用戶在做一些其他觸摸操作時，出現意外地滑動，例如觸摸屏幕上的組件。

另外兩個常用的[ViewConfiguration](http://developer.android.com/reference/android/view/ViewConfiguration.html)函數是<a href="http://developer.android.com/reference/android/view/ViewConfiguration.html#getScaledMinimumFlingVelocity()">getScaledMinimumFlingVelocity()</a>和<a href="http://developer.android.com/reference/android/view/ViewConfiguration.html#getScaledMaximumFlingVelocity()">getScaledMaximumFlingVelocity()</a>。這兩個函數會返回初始化一個快速滑動(fling)的最小、最大速度（分別地），以像素每秒為測量單位。如：

```java
ViewConfiguration vc = ViewConfiguration.get(view.getContext());
private int mSlop = vc.getScaledTouchSlop();
private int mMinFlingVelocity = vc.getScaledMinimumFlingVelocity();
private int mMaxFlingVelocity = vc.getScaledMaximumFlingVelocity();

...

case MotionEvent.ACTION_MOVE: {
    ...
    float deltaX = motionEvent.getRawX() - mDownX;
    if (Math.abs(deltaX) > mSlop) {
        // A swipe occurred, do something
    }

...

case MotionEvent.ACTION_UP: {
    ...
    } if (mMinFlingVelocity <= velocityX && velocityX <= mMaxFlingVelocity
            && velocityY < velocityX) {
        // The criteria have been satisfied, do something
    }
}
```

## 擴展子view的可觸摸區域

Android提供了[TouchDelegate](http://developer.android.com/reference/android/view/TouchDelegate.html)類，讓父view擴展超出子view自身邊界的可觸摸區域。這在當子view很小，但需要一個更大的觸摸區域時非常有用。如果需要，我們也可以使用這種方式來實現對子view的觸摸區域的收縮。

在下面的例子中，[ImageButton](http://developer.android.com/reference/android/widget/ImageButton.html)對象是所謂的"delegate view"（是指觸摸區域將被父view擴展的那個子view）。這是佈局文件：

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
     android:id="@+id/parent_layout"
     android:layout_width="match_parent"
     android:layout_height="match_parent"
     tools:context=".MainActivity" >

     <ImageButton android:id="@+id/button"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:background="@null"
          android:src="@drawable/icon" />
</RelativeLayout>
```

下面的代碼段做了這樣幾件事：

- 獲得父view對象併發送一個[Runnable](http://developer.android.com/reference/java/lang/Runnable.html)到UI線程。這會確保父view在調用<a href="http://developer.android.com/reference/android/view/View.html#getHitRect(android.graphics.Rect)">getHitRect()</a>函數前會佈局它的子view。`getHitRect()`函數會獲得子view在父view座標系中的點擊矩形（觸摸區域）。
- 找到[ImageButton](http://developer.android.com/reference/android/widget/ImageButton.html)子view，然後調用`getHitRect()`來獲得它的觸摸區域的邊界。
- 擴展[ImageButton](http://developer.android.com/reference/android/widget/ImageButton.html)的點擊矩形的邊界。
- 實例化一個[TouchDelegate](http://developer.android.com/reference/android/view/TouchDelegate.html)對象，並把擴展過的點擊矩形和[ImageButton](http://developer.android.com/reference/android/widget/ImageButton.html)子view作為參數傳遞給它。
- 設置父view的[TouchDelegate](http://developer.android.com/reference/android/view/TouchDelegate.html)，這樣在touch delegate邊界內的點擊就會傳遞到該子view上。

在[ImageButton](http://developer.android.com/reference/android/widget/ImageButton.html)子view的touch delegate範圍內，父view會接收到所有的觸摸事件。如果觸摸事件發生在子view自身的點擊矩形中，父view會把觸摸事件交給子view處理。

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // Get the parent view
        View parentView = findViewById(R.id.parent_layout);

        parentView.post(new Runnable() {
            // Post in the parent's message queue to make sure the parent
            // lays out its children before you call getHitRect()
            @Override
            public void run() {
                // The bounds for the delegate view (an ImageButton
                // in this example)
                Rect delegateArea = new Rect();
                ImageButton myButton = (ImageButton) findViewById(R.id.button);
                myButton.setEnabled(true);
                myButton.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View view) {
                        Toast.makeText(MainActivity.this,
                                "Touch occurred within ImageButton touch region.",
                                Toast.LENGTH_SHORT).show();
                    }
                });

                // The hit rectangle for the ImageButton
                myButton.getHitRect(delegateArea);

                // Extend the touch area of the ImageButton beyond its bounds
                // on the right and bottom.
                delegateArea.right += 100;
                delegateArea.bottom += 100;

                // Instantiate a TouchDelegate.
                // "delegateArea" is the bounds in local coordinates of
                // the containing view to be mapped to the delegate view.
                // "myButton" is the child view that should receive motion
                // events.
                TouchDelegate touchDelegate = new TouchDelegate(delegateArea,
                        myButton);

                // Sets the TouchDelegate on the parent view, such that touches
                // within the touch delegate bounds are routed to the child.
                if (View.class.isInstance(myButton.getParent())) {
                    ((View) myButton.getParent()).setTouchDelegate(touchDelegate);
                }
            }
        });
    }
}
```
