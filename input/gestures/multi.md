# 處理多點觸控手勢

> 編寫:[Andrwyw](https://github.com/Andrwyw) - 原文:<http://developer.android.com/training/gestures/multi.html>

多點觸控手勢是指在同一時間有多點（手指）觸碰屏幕。本節課程講述，如何檢測涉及多點的觸摸手勢。

## 追蹤多點

當多個手指同時觸摸屏幕時，系統會產生如下的觸摸事件：

- [ACTION_DOWN](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_DOWN) - 針對觸摸屏幕的第一個點。此事件是手勢的開端。第一觸摸點的數據在[MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html)中的索引總是0。
- <a href="http://developer.android.com/reference/android/support/v4/view/MotionEventCompat.html#ACTION_POINTER_DOWN">ACTION\_POINTER\_DOWN</a> - 針對第一點後，出現在屏幕上額外的點。這個點的數據在MotionEvent中的索引，可以通過<a href="(http://developer.android.com/reference/android/support/v4/view/MotionEventCompat.html#getActionIndex(android.view.MotionEvent)">getActionIndex()</a>獲得。
- [ACTION_MOVE](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_MOVE) - 在按下手勢期間發生變化。
- <a href="http://developer.android.com/reference/android/support/v4/view/MotionEventCompat.html#ACTION_POINTER_UP">ACTION\_POINTER\_UP</a> - 當非主要點（non-primary pointer）離開屏幕時，發送此事件。
- [ACTION_UP](http://developer.android.com/reference/android/view/MotionEvent.html#ACTION_UP) - 當最後一點離開屏幕時發送此事件。

我們可以通過各個點的索引以及id，單獨地追蹤[MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html)中的每個點。

- **Index**：[MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html)把各個點的信息都存儲在一個數組中。點的索引值就是它在數組中的位置。大多數用來與點交互的[MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html)函數都是以索引值而不是點的ID作為參數的。
- **ID**：每個點也都有一個ID映射，該ID映射在整個手勢期間一直存在，以便我們單獨地追蹤每個點。

每個獨立的點在移動事件中出現的次序是不固定的。因此，從一個事件到另一個事件，點的索引值是可以改變的，但點的ID在它的生命週期內是保證不會改變的。使用<a href="http://developer.android.com/reference/android/view/MotionEvent.html#getPointerId(int)">getPointerId()</a>可以獲得一個點的ID，在手勢隨後的移動事件中，就可以用該ID來追蹤這個點。對於隨後一系列的事件，可以使用<a href="http://developer.android.com/reference/android/view/MotionEvent.html#findPointerIndex(int)">findPointerIndex()</a>函數，來獲得對應給定ID的點在移動事件中的索引值。如下：

```java
private int mActivePointerId;

public boolean onTouchEvent(MotionEvent event) {
    ....
    // Get the pointer ID
    mActivePointerId = event.getPointerId(0);

    // ... Many touch events later...

    // Use the pointer ID to find the index of the active pointer
    // and fetch its position
    int pointerIndex = event.findPointerIndex(mActivePointerId);
    // Get the pointer's current position
    float x = event.getX(pointerIndex);
    float y = event.getY(pointerIndex);
}
```

## 獲取MotionEvent的動作

我們應該總是使用<a href="http://developer.android.com/reference/android/view/MotionEvent.html#getActionMasked()">getActionMasked()</a>函數（或者用<a href="http://developer.android.com/reference/android/support/v4/view/MotionEventCompat.html#getActionMasked(android.view.MotionEvent)">MotionEventCompat.getActionMasked()</a>這個兼容版本更好）來獲取[MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html)的動作(action)。與舊的<a href="http://developer.android.com/reference/android/view/MotionEvent.html#getAction()">getAction()</a>函數不同的是，`getActionMasked()`是設計用來處理多點觸摸的。它會返回執行過的動作的掩碼值，不包括點的索引位。然後，我們可以使用`getActionIndex()`來獲得與該動作關聯的點的索引值。這在接下來的代碼段中可以看到。

> **Note:** 這個樣例使用的是[MotionEventCompat](http://developer.android.com/reference/android/support/v4/view/MotionEventCompat.html)類。該類在[**Support Library**](http://developer.android.com/tools/support-library/index.html)中。我們應該使用[MotionEventCompat](http://developer.android.com/reference/android/support/v4/view/MotionEventCompat.html)類，來提供對更多平臺的支持。需要注意的一點是，[MotionEventCompat](http://developer.android.com/reference/android/support/v4/view/MotionEventCompat.html)並不是[MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html)類的替代品。準確來說，它提供了一些靜態工具類函數，我們可以把[MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html)對象作為參數傳遞給這些函數，來得到與事件相關的動作。

```java
int action = MotionEventCompat.getActionMasked(event);
// Get the index of the pointer associated with the action.
int index = MotionEventCompat.getActionIndex(event);
int xPos = -1;
int yPos = -1;

Log.d(DEBUG_TAG,"The action is " + actionToString(action));

if (event.getPointerCount() > 1) {
    Log.d(DEBUG_TAG,"Multitouch event");
    // The coordinates of the current screen contact, relative to
    // the responding View or Activity.
    xPos = (int)MotionEventCompat.getX(event, index);
    yPos = (int)MotionEventCompat.getY(event, index);

} else {
    // Single touch event
    Log.d(DEBUG_TAG,"Single touch event");
    xPos = (int)MotionEventCompat.getX(event, index);
    yPos = (int)MotionEventCompat.getY(event, index);
}
...

// Given an action int, returns a string description
public static String actionToString(int action) {
    switch (action) {

        case MotionEvent.ACTION_DOWN: return "Down";
        case MotionEvent.ACTION_MOVE: return "Move";
        case MotionEvent.ACTION_POINTER_DOWN: return "Pointer Down";
        case MotionEvent.ACTION_UP: return "Up";
        case MotionEvent.ACTION_POINTER_UP: return "Pointer Up";
        case MotionEvent.ACTION_OUTSIDE: return "Outside";
        case MotionEvent.ACTION_CANCEL: return "Cancel";
    }
    return "";
}
```

關於多點觸摸的更多內容以及示例，可以查看[拖拽與縮放](scale.html)章節。
