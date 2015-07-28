# 追蹤手勢移動

> 編寫:[Andrwyw](https://github.com/Andrwyw) - 原文：<http://developer.android.com/training/gestures/movement.html>

本節課程講述如何追蹤手勢移動。

每當當前的觸摸位置、壓力、大小發生變化時，[ACTION_MOVE](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_MOVE)事件都會觸發<a href="http://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent)">onTouchEvent()</a>函數。正如[**檢測常用的手勢**](/detector.html)中描述的那樣，觸摸事件全部都記錄在onTouchEvent()函數的[MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html)參數中。

因為基於手指的觸摸的交互方式並不總是非常精確，所以檢測觸摸事件更多的是基於手勢移動，而非簡單地基於觸摸。為了幫助app區分基於移動的手勢（如滑動）和非移動手勢（如簡單地點擊），Android引入了“touch slop”的概念。Touch slop是指，在被識別為基於移動的手勢前，用戶觸摸可移動的那一段像素距離。關於這一主題的更多討論，可以在[管理ViewGroup中的觸摸事件](viewgroup.html)中查看。

根據應用的需求，有多種追蹤手勢移動的方式可以選擇。比如：

* 追蹤手指的起始和終止位置（比如，把屏幕上的對象從A點移動到B點）
* 根據x、y軸座標，追蹤手指移動的方向。
* 追蹤歷史狀態。我們可以通過調用[MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html)的<a href="http://developer.android.com/reference/android/view/MotionEvent.html#getHistorySize()">getHistorySize()</a>方法，來獲得一個手勢的歷史尺寸。我們可以通過移動事件的`getHistorical<Value>`系列函數，來獲得事件之前的位置、尺寸、時間以及按壓力(pressures)。當我們需要繪製用戶手指痕跡時，歷史狀態非常有用，比如觸摸繪圖。查看[MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html)來了解更多細節。
* 追蹤手指在觸摸屏上滑過的速度。

## 追蹤速度

我們可以簡單地用基於距離，或(和)基於手指移動方向的移動手勢。但是速度經常也是追蹤手勢特性的一個決定性因素，甚至是判斷一個手勢是否發生的依據。為了讓計算速度更容易，Android提供了[VelocityTracker](http://developer.android.com/reference/android/view/VelocityTracker.html)類以及[**Support Library**](http://developer.android.com/tools/support-library/index.html)中的[VelocityTrackerCompat](http://developer.android.com/reference/android/support/v4/view/VelocityTrackerCompat.html)類。[VelocityTracker](http://developer.android.com/reference/android/view/VelocityTracker.html)類可以幫助我們追蹤觸摸事件中的速度因素。如果速度是手勢的一個判斷標準，比如快速滑動(fling)，那麼這些類是很有用的。

下面是一個簡單的例子，說明了[VelocityTracker](http://developer.android.com/reference/android/view/VelocityTracker.html)中API函數的用處。

```java
public class MainActivity extends Activity {
    private static final String DEBUG_TAG = "Velocity";
        ...
    private VelocityTracker mVelocityTracker = null;
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int index = event.getActionIndex();
        int action = event.getActionMasked();
        int pointerId = event.getPointerId(index);

        switch(action) {
            case MotionEvent.ACTION_DOWN:
                if(mVelocityTracker == null) {
                    // Retrieve a new VelocityTracker object to watch the velocity of a motion.
                    mVelocityTracker = VelocityTracker.obtain();
                }
                else {
                    // Reset the velocity tracker back to its initial state.
                    mVelocityTracker.clear();
                }
                // Add a user's movement to the tracker.
                mVelocityTracker.addMovement(event);
                break;
            case MotionEvent.ACTION_MOVE:
                mVelocityTracker.addMovement(event);
                // When you want to determine the velocity, call
                // computeCurrentVelocity(). Then call getXVelocity()
                // and getYVelocity() to retrieve the velocity for each pointer ID.
                mVelocityTracker.computeCurrentVelocity(1000);
                // Log velocity of pixels per second
                // Best practice to use VelocityTrackerCompat where possible.
                Log.d("", "X velocity: " +
                        VelocityTrackerCompat.getXVelocity(mVelocityTracker,
                        pointerId));
                Log.d("", "Y velocity: " +
                        VelocityTrackerCompat.getYVelocity(mVelocityTracker,
                        pointerId));
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                // Return a VelocityTracker object back to be re-used by others.
                mVelocityTracker.recycle();
                break;
        }
        return true;
    }
}
```

> **Note:** 需要注意的是，我們應該在[ACTION_MOVE](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_MOVE)事件，而不是在[ACTION_UP](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_UP)事件後計算速度。在[ACTION_UP](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_UP)事件之後，計算x、y方向上的速度都會是0。
