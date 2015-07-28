# 使得View可交互

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/custom-view/make-interactive.html>

繪製UI僅僅是創建自定義View的一部分。你還需要使得你的View能夠以模擬現實世界的方式來進行反饋。對象應該總是與現實情景能夠保持一致。例如，圖片不應該突然消失又從另外一個地方出現，因為在現實世界裡面不會發生那樣的事情。正確的應該是，圖片從一個地方移動到另外一個地方。

用戶應該可以感受到UI上的微小變化，並對模仿現實世界的細微之處反應強烈。例如，當用戶fling(迅速滑動)一個對象時，應該在開始時感到摩擦帶來的阻力，在結束時感到fling帶動的動力。應該在滑動開始與結束的時候給用戶一定的反饋。

這節課會演示如何使用Android framework的功能來為自定義的View添加那些現實世界中的行為。

<!-- more -->

## 處理輸入的手勢
像許多其他UI框架一樣，Android提供一個輸入事件模型。用戶的動作會轉換成觸發一些回調函數的事件，你可以重寫這些回調方法來定製你的程序應該如何響應用戶的輸入事件。在Android中最常用的輸入事件是touch，它會觸發[onTouchEvent(android.view.MotionEvent)](http://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent))的回調。重寫這個方法來處理touch事件：

```java
@Override
public boolean onTouchEvent(MotionEvent event) {
  return super.onTouchEvent(event);
}
```

Touch事件本身並不是特別有用。如今的touch UI定義了touch事件之間的相互作用，叫做gestures。例如tapping,pulling,flinging與zooming。為了把那些touch的源事件轉換成gestures, Android提供了[GestureDetector](http://developer.android.com/reference/android/view/GestureDetector.html)。

通過傳入[GestureDetector.OnGestureListener](http://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html)的一個實例構建一個GestureDetector。如果你只是想要處理幾種gestures(手勢操作)你可以繼承[GestureDetector.SimpleOnGestureListener](http://developer.android.com/reference/android/view/GestureDetector.SimpleOnGestureListener.html)，而不用實現[GestureDetector.OnGestureListener](http://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html)接口。例如，下面的代碼創建一個繼承[GestureDetector.SimpleOnGestureListener](http://developer.android.com/reference/android/view/GestureDetector.SimpleOnGestureListener.html)的類，並重寫[onDown(MotionEvent)](http://developer.android.com/reference/android/view/GestureDetector.SimpleOnGestureListener.html#onDown(android.view.MotionEvent))。

```java
class mListener extends GestureDetector.SimpleOnGestureListener {
   @Override
   public boolean onDown(MotionEvent e) {
       return true;
   }
}
mDetector = new GestureDetector(PieChart.this.getContext(), new mListener());
```

不管你是否使用GestureDetector.SimpleOnGestureListener, 你必須總是實現onDown()方法，並返回true。這一步是必須的，因為所有的gestures都是從onDown()開始的。如果你在onDown()裡面返回false，系統會認為你想要忽略後續的gesture,那麼GestureDetector.OnGestureListener的其他回調方法就不會被執行到了。一旦你實現了GestureDetector.OnGestureListener並且創建了GestureDetector的實例, 你可以使用你的GestureDetector來中止你在onTouchEvent裡面收到的touch事件。

```java
@Override
public boolean onTouchEvent(MotionEvent event) {
   boolean result = mDetector.onTouchEvent(event);
   if (!result) {
       if (event.getAction() == MotionEvent.ACTION_UP) {
           stopScrolling();
           result = true;
       }
   }
   return result;
}
```

當你傳遞一個touch事件到onTouchEvent()時，若這個事件沒有被辨認出是何種gesture，它會返回false。你可以執行自定義的gesture-decection代碼。

## 創建基本合理的物理運動
Gestures是控制觸摸設備的一種強有力的方式，但是除非你能夠產出一個合理的觸摸反饋，否則將是違反用戶直覺的。一個很好的例子是fling手勢，用戶迅速的在屏幕上移動手指然後擡手離開屏幕。這個手勢應該使得UI迅速的按照fling的方向進行滑動，然後慢慢停下來，就像是用戶旋轉一個飛輪一樣。

但是模擬這個飛輪的感覺並不簡單，要想得到正確的飛輪模型，需要大量的物理，數學知識。幸運的是，Android有提供幫助類來模擬這些物理行為。[Scroller](http://developer.android.com/reference/android/widget/Scroller.html)是控制飛輪式的fling的基類。


要啟動一個fling，需調用`fling()`，並傳入啟動速率、x、y的最小值和最大值，對於啟動速度值，可以使用[GestureDetector](http://developer.android.com/reference/android/view/GestureDetector.html)計算得出。
```java
@Override
public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
   mScroller.fling(currentX, currentY, velocityX / SCALE, velocityY / SCALE, minX, minY, maxX, maxY);
   postInvalidate();
}
```

> **Note:** 儘管速率是通過GestureDetector來計算的，許多開發者感覺使用這個值使得fling動畫太快。通常把x與y設置為4到8倍的關係。


調用[fling()](http://developer.android.com/reference/android/widget/Scroller.html#fling(int, int, int, int, int, int, int, int))時會為fling手勢設置物理模型。然後，通過調用定期調用 [Scroller.computeScrollOffset()](http://developer.android.com/reference/android/widget/Scroller.html#computeScrollOffset())來更新Scroller。[computeScrollOffset()](http://developer.android.com/reference/android/widget/Scroller.html#computeScrollOffset())通過讀取當前時間和使用物理模型來計算x和y的位置更新Scroller對象的內部狀態。調用[getCurrX()](http://developer.android.com/reference/android/widget/Scroller.html#getCurrX())和[getCurrY()](http://developer.android.com/reference/android/widget/Scroller.html#getCurrY())來獲取這些值。

大多數view通過Scroller對象的x,y的位置直接到[scrollTo()](http://developer.android.com/reference/android/view/View.html#scrollTo(int, int))，PieChart例子稍有不同，它使用當前滾動y的位置設置圖表的旋轉角度。
```java
if (!mScroller.isFinished()) {
    mScroller.computeScrollOffset();
    setPieRotation(mScroller.getCurrY());
}
```

[Scroller](http://developer.android.com/reference/android/widget/Scroller.html) 類會為你計算滾動位置，但是他不會自動把哪些位置運用到你的view上面。你有責任確保View獲取並運用到新的座標。你有兩種方法來實現這件事情：

* 在調用fling()之後執行postInvalidate(), 這是為了確保能強制進行重畫。這個技術需要每次在onDraw裡面計算過scroll offsets(滾動偏移量)之後調用postInvalidate()。
* 使用[ValueAnimator](http://developer.android.com/reference/android/animation/ValueAnimator.html)在fling是展現動畫，並且通過調用addUpdateListener()增加對fling過程的監聽。

這個PieChart 的例子使用了第二種方法。這個方法使用起來會稍微複雜一點，但是它更有效率並且避免了不必要的重畫的view進行重繪。缺點是ValueAnimator是從API Level 11才有的。因此他不能運用到3.0的系統之前的版本上。

> ** Note: ** ValueAnimator雖然是API 11才有的，但是你還是可以在最低版本低於3.0的系統上使用它，做法是在運行時判斷當前的API Level，如果低於11則跳過。

```java
 mScroller = new Scroller(getContext(), null, true);
 mScrollAnimator = ValueAnimator.ofFloat(0,1);
 mScrollAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
     @Override
     public void onAnimationUpdate(ValueAnimator valueAnimator) {
         if (!mScroller.isFinished()) {
             mScroller.computeScrollOffset();
             setPieRotation(mScroller.getCurrY());
         } else {
             mScrollAnimator.cancel();
             onScrollFinished();
         }
     }
 });
```

## 使過渡平滑
用戶期待一個UI之間的切換是能夠平滑過渡的。UI元素需要做到漸入淡出來取代突然出現與消失。Android從3.0開始有提供[property animation framework](http://developer.android.com/guide/topics/graphics/prop-animation.html),用來使得平滑過渡變得更加容易。

使用這套動畫系統時，任何時候屬性的改變都會影響到你的視圖，所以不要直接改變屬性的值。而是使用ValueAnimator來實現改變。在下面的例子中，在PieChart 中更改選擇的部分將導致整個圖表的旋轉，以至選擇的進入選擇區內。ValueAnimator在數百毫秒內改變旋轉量，而不是突然地設置新的旋轉值。

```java
mAutoCenterAnimator = ObjectAnimator.ofInt(PieChart.this, "PieRotation", 0);
mAutoCenterAnimator.setIntValues(targetAngle);
mAutoCenterAnimator.setDuration(AUTOCENTER_ANIM_DURATION);
mAutoCenterAnimator.start();
```

如果你想改變的是view的某些基礎屬性，你可以使用[ViewPropertyAnimator](http://developer.android.com/reference/android/view/ViewPropertyAnimator.html) ,它能夠同時執行多個屬性的動畫。

```java
animate().rotation(targetAngle).setDuration(ANIM_DURATION).start();
```
