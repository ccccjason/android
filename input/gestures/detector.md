# 檢測常用的手勢

> 編寫:[Andrwyw](https://github.com/Andrwyw) - 原文:<http://developer.android.com/training/gestures/detector.html>

當用戶把用一根或多根手指放在觸摸屏上，並且應用把這樣的觸摸方式解釋為特定的手勢時，“觸摸手勢”就發生了。相應地，檢測手勢也就有以下兩個階段：

1. 收集觸摸事件的相關數據。
2. 分析這些數據，看它們是否符合app所支持的手勢的標準。

### Support Library 中的類

本節課程的示例程序使用了[GestureDetectorCompat](http://developer.android.com/reference/android/support/v4/view/GestureDetectorCompat.html)和[MotionEventCompat](http://developer.android.com/reference/android/support/v4/view/MotionEventCompat.html)類。這些類都是在 [Support Library](http://developer.android.com/tools/support-library/index.html) 中定義的。如果有可能的情況話，我們應該使用 Support Library 中的類，來為運行著Android1.6及以上版本系統的設備提供兼容性功能。需要注意的一點是，[MotionEventCompat](http://developer.android.com/reference/android/support/v4/view/MotionEventCompat.html)並不是[MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html)的替代品，而是提供了一些靜態工具類函數。我們可以把[MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html)對象作為參數傳遞給這些工具類函數，來獲得與觸摸事件相關的動作(action)。

## 收集數據

當用戶把用一根或多根手指放在觸摸屏上時，會觸發 View 上用於接收觸摸事件的 <a href="http://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent)">onTouchEvent()</a> 回調函數。對於一系列連續的、最終會被識別為一種手勢的觸摸事件（位置、壓力、大小、添加另一根手指等等），<a href="http://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent)">onTouchEvent()</a>會被調用若干次。

當用戶第一次觸摸屏幕時，手勢就開始了。其後系統會持續地追蹤用戶手指的位置，在用戶手指全都離開屏幕時，手勢結束。在整個交互期間，被分發給 <a href="http://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent)">onTouchEvent()</a> 函數的 [MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html) 對象，提供了每次交互的詳細信息。我們的app可以使用 [MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html) 提供的這些數據，來判斷某種特定的手勢是否發生了。

### 為Activity或View捕獲觸摸事件

為了捕獲Activity或View中的觸摸事件，我們可以重寫<a href="http://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent)">onTouchEvent()</a>回調函數。

接下來的代碼段使用了<a href="http://developer.android.com/reference/android/support/v4/view/MotionEventCompat.html#getActionMasked(android.view.MotionEvent)">getActionMasked()</a>函數，來從 `event` 參數中抽取出用戶執行的動作。它提供了一些原始的觸摸數據，我們可以使用這些數據，來判斷某個特定手勢是否發生了。

```java
public class MainActivity extends Activity {
...
// This example shows an Activity, but you would use the same approach if
// you were subclassing a View.
@Override
public boolean onTouchEvent(MotionEvent event){

        int action = MotionEventCompat.getActionMasked(event);

        switch(action) {
                case (MotionEvent.ACTION_DOWN) :
                Log.d(DEBUG_TAG,"Action was DOWN");
                return true;
        case (MotionEvent.ACTION_MOVE) :
                Log.d(DEBUG_TAG,"Action was MOVE");
                return true;
        case (MotionEvent.ACTION_UP) :
                Log.d(DEBUG_TAG,"Action was UP");
                return true;
        case (MotionEvent.ACTION_CANCEL) :
                Log.d(DEBUG_TAG,"Action was CANCEL");
                return true;
        case (MotionEvent.ACTION_OUTSIDE) :
                Log.d(DEBUG_TAG,"Movement occurred outside bounds " +
                        "of current screen element");
                return true;
        default :
                return super.onTouchEvent(event);
        }
}
```

然後，我們可以對這些事件做些自己的處理，以判斷某個手勢是否出現了。這種是針對自定義手勢，我們所需要進行的處理。然而，如果我們的app僅僅需要一些常見的手勢，如雙擊，長按，快速滑動（fling）等，那麼我們可以使用[GestureDetector](http://developer.android.com/reference/android/view/GestureDetector.html)類來完成。 [GestureDetector](http://developer.android.com/reference/android/view/GestureDetector.html)可以讓我們簡單地檢測常見手勢，並且無需自行處理單個觸摸事件。相關內容將會在下面的[檢測手勢](#detect)中討論。

### 捕獲單個view的觸摸事件

作為<a href="http://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent)">onTouchEvent()</a>的一種替換方式，我們也可以使用 <a href="http://developer.android.com/reference/android/view/View.html#setOnTouchListener(android.view.View.OnTouchListener)">setOnTouchListener()</a> 函數，來把 [View.OnTouchListener](http://developer.android.com/reference/android/view/View.OnTouchListener.html) 關聯到任意的[View](http://developer.android.com/reference/android/view/View.html)上。這樣可以在不繼承已有的 [View](http://developer.android.com/reference/android/view/View.html) 的情況下，也能監聽觸摸事件。比如:

```java
View myView = findViewById(R.id.my_view);
myView.setOnTouchListener(new OnTouchListener() {
public boolean onTouch(View v, MotionEvent event) {
        // ... Respond to touch events
        return true;
    }
});
```

創建listener對象時，注意 [ACTION_DOWN](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_DOWN) 事件返回 `false` 的情況。如果返回 `false`，會讓listener對象接收不到後續的[ACTION_MOVE](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_UP)、[ACTION_UP](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_UP)等系列事件。這是因為[ACTION_DOWN](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_DOWN)事件是所有觸摸事件的開端。

如果我們正在寫一個自定義View，我們也可以像上面描述的那樣重寫<a href="http://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent)">onTouchEvent()</a>函數。

<a name="detect"> </a>
## 檢測手勢

Android提供了[GestureDetector](http://developer.android.com/reference/android/view/GestureDetector.html)類來檢測常用的手勢。它所支持的手勢包括<a href="http://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html#onDown(android.view.MotionEvent)">onDown()</a>、<a href="http://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html#onLongPress(android.view.MotionEvent)">onLongPress()</a>、<a href="http://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html#onFling(android.view.MotionEvent,android.view.MotionEvent,float,float)">onFling()</a> 等。我們可以把[GestureDetector](http://developer.android.com/reference/android/view/GestureDetector.html)和上面描述的onTouchEvent()函數結合在一起使用。

### 檢測所有支持的手勢

當我們實例化一個[GestureDetectorCompat](http://developer.android.com/reference/android/support/v4/view/GestureDetectorCompat.html)對象時，需要一個實現了[GestureDetector.OnGestureListener](http://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html)接口的類作為參數。當某個特定的觸摸事件發生時，[GestureDetector.OnGestureListener](http://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html)就會通知用戶。為了讓我們的[GestureDetector](http://developer.android.com/reference/android/view/GestureDetector.html)對象能到接收到觸摸事件，我們需要重寫 View 或 Activity 的 onTouchEvent() 函數，並且把所有捕獲到的事件傳遞給 detector 實例。

接下來的代碼段中，`on<TouchEvent>` 型的函數的返回值是 `true`，意味著我們已經處理完這個觸摸事件了。如果返回 `false`，則會把事件沿view棧傳遞，直到觸摸事件被成功地處理了。

運行下面的代碼段，來了解當我們與觸摸屏交互時，動作（action）是如何觸發的，以及每個觸摸事件[MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html)中的內容。我們也會意識到，一個簡單的交互會產生多少的數據。

```java
public class MainActivity extends Activity implements
        GestureDetector.OnGestureListener,
        GestureDetector.OnDoubleTapListener{

    private static final String DEBUG_TAG = "Gestures";
    private GestureDetectorCompat mDetector;

    // Called when the activity is first created.
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // Instantiate the gesture detector with the
        // application context and an implementation of
        // GestureDetector.OnGestureListener
        mDetector = new GestureDetectorCompat(this,this);
        // Set the gesture detector as the double tap
        // listener.
        mDetector.setOnDoubleTapListener(this);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event){
        this.mDetector.onTouchEvent(event);
        // Be sure to call the superclass implementation
        return super.onTouchEvent(event);
    }

    @Override
    public boolean onDown(MotionEvent event) {
        Log.d(DEBUG_TAG,"onDown: " + event.toString());
        return true;
    }

    @Override
    public boolean onFling(MotionEvent event1, MotionEvent event2,
            float velocityX, float velocityY) {
        Log.d(DEBUG_TAG, "onFling: " + event1.toString()+event2.toString());
        return true;
    }

    @Override
    public void onLongPress(MotionEvent event) {
        Log.d(DEBUG_TAG, "onLongPress: " + event.toString());
    }

    @Override
    public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX,
            float distanceY) {
        Log.d(DEBUG_TAG, "onScroll: " + e1.toString()+e2.toString());
        return true;
    }

    @Override
    public void onShowPress(MotionEvent event) {
        Log.d(DEBUG_TAG, "onShowPress: " + event.toString());
    }

    @Override
    public boolean onSingleTapUp(MotionEvent event) {
        Log.d(DEBUG_TAG, "onSingleTapUp: " + event.toString());
        return true;
    }

    @Override
    public boolean onDoubleTap(MotionEvent event) {
        Log.d(DEBUG_TAG, "onDoubleTap: " + event.toString());
        return true;
    }

    @Override
    public boolean onDoubleTapEvent(MotionEvent event) {
        Log.d(DEBUG_TAG, "onDoubleTapEvent: " + event.toString());
        return true;
    }

    @Override
    public boolean onSingleTapConfirmed(MotionEvent event) {
        Log.d(DEBUG_TAG, "onSingleTapConfirmed: " + event.toString());
        return true;
    }
}
```

### 檢測部分支持的手勢

如果我們只想處理幾種手勢，那麼可以選擇繼承 [GestureDetector.SimpleOnGestureListener](http://developer.android.com/reference/android/view/GestureDetector.SimpleOnGestureListener.html) 類，而不是實現 [GestureDetector.OnGestureListener](http://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html) 接口。

GestureDetector.SimpleOnGestureListener 類實現了所有的 `on<TouchEvent>` 型函數，其中，這些函數都返回 `false`。因此，我們可以僅僅重寫我們需要的函數。比如，下面的代碼段中，創建了一個繼承自 [GestureDetector.SimpleOnGestureListener](http://developer.android.com/reference/android/view/GestureDetector.SimpleOnGestureListener.html) 的類，並重寫了 onFling() 和 onDown() 函數。

無論我們是否使用[GestureDetector.OnGestureListener](http://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html)類，最好都實現 onDown() 函數並且返回 `true`。這是因為所有的手勢都是由 onDown() 消息開始的。如果讓 onDown() 函數返回 `false`，就像[GestureDetector.SimpleOnGestureListener](http://developer.android.com/reference/android/view/GestureDetector.SimpleOnGestureListener.html)類中默認實現的那樣，系統會假定我們想忽略剩餘的手勢，[GestureDetector.OnGestureListener](http://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html)中的其他函數也就永遠不會被調用。這可能會導致我們的app出現意想不到的問題。僅僅當我們真的想忽略全部手勢時，我們才應該讓 onDown() 函數返回 `false`。

```java
public class MainActivity extends Activity {

    private GestureDetectorCompat mDetector;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mDetector = new GestureDetectorCompat(this, new MyGestureListener());
    }

    @Override
    public boolean onTouchEvent(MotionEvent event){
        this.mDetector.onTouchEvent(event);
        return super.onTouchEvent(event);
    }

    class MyGestureListener extends GestureDetector.SimpleOnGestureListener {
        private static final String DEBUG_TAG = "Gestures";

        @Override
        public boolean onDown(MotionEvent event) {
            Log.d(DEBUG_TAG,"onDown: " + event.toString());
            return true;
        }

        @Override
        public boolean onFling(MotionEvent event1, MotionEvent event2,
                float velocityX, float velocityY) {
            Log.d(DEBUG_TAG, "onFling: " + event1.toString()+event2.toString());
            return true;
        }
    }
}
```
