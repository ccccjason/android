# 優化自定義View

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/custom-views/optimizing-view.html>

前面的課程學習到了如何創建設計良好的View，並且能夠使之在手勢與狀態切換時得到正確的反饋。下面要介紹的是如何使得view能夠執行更快。為了避免UI顯得卡頓，你必須確保動畫能夠保持在60fps。

<!-- more -->

## Do Less, Less Frequently

為了加速你的view，對於頻繁調用的方法，需要儘量減少不必要的代碼。先從onDraw開始，需要特別注意不應該在這裡做內存分配的事情，因為它會導致GC，從而導致卡頓。在初始化或者動畫間隙期間做分配內存的動作。不要在動畫正在執行的時候做內存分配的事情。

你還需要儘可能的減少onDraw被調用的次數，大多數時候導致onDraw都是因為調用了invalidate().因此請儘量減少調用invaildate()的次數。如果可能的話，儘量調用含有4個參數的invalidate()方法而不是沒有參數的invalidate()。沒有參數的invalidate會強制重繪整個view。

另外一個非常耗時的操作是請求layout。任何時候執行requestLayout()，會使得Android UI系統去遍歷整個View的層級來計算出每一個view的大小。如果找到有衝突的值，它會需要重新計算好幾次。另外需要儘量保持View的層級是扁平化的，這樣對提高效率很有幫助。

如果你有一個複雜的UI，你應該考慮寫一個自定義的ViewGroup來執行他的layout操作。與內置的view不同，自定義的view可以使得程序僅僅測量這一部分，這避免了遍歷整個view的層級結構來計算大小。這個PieChart 例子展示瞭如何繼承ViewGroup作為自定義view的一部分。PieChart 有子views，但是它從來不測量它們。而是根據他自身的layout法則，直接設置它們的大小。

## 使用硬件加速

從Android 3.0開始，Android的2D圖像系統可以通過GPU (Graphics Processing Unit))來加速。GPU硬件加速可以提高許多程序的性能。但是這並不是說它適合所有的程序。Android framework讓你能過隨意控制你的程序的各個部分是否啟用硬件加速。

參考 Android Developers Guide 中的[Hardware Acceleration](http://developer.android.com/guide/topics/graphics/hardware-accel.html) 來學習如何在application, activity, 或 window 層啟用加速。注意除了 Android Guide 的指導之外，你必須要設置你的應用的target API為11，或更高，通過在你的AndroidManifest.xml 文件中增加 < uses-sdk android:targetSdkVersion="11"/> 。

一旦你開啟了硬件加速，性能的提示並不一定可以明顯察覺到。移動設備的GPU在某些例如scaling,rotating與translating的操作中表現良好。但是對其他一些任務，比如畫直線或曲線，則表現不佳。為了充分發揮GPU加速，你應該最大化GPU擅長的操作的數量，最小化GPU不擅長操作的數量。

在下面的例子中，繪製pie是相對來說比較費時的。解決方案是把pie放到一個子view中，並設置View使用LAYER_TYPE_HARDWARE來進行加速。

```java
private class PieView extends View {

       public PieView(Context context) {
           super(context);
           if (!isInEditMode()) {
               setLayerType(View.LAYER_TYPE_HARDWARE, null);
           }
       }
       
       @Override
       protected void onDraw(Canvas canvas) {
           super.onDraw(canvas);

           for (Item it : mData) {
               mPiePaint.setShader(it.mShader);
               canvas.drawArc(mBounds,
                       360 - it.mEndAngle,
                       it.mEndAngle - it.mStartAngle,
                       true, mPiePaint);
           }
       }

       @Override
       protected void onSizeChanged(int w, int h, int oldw, int oldh) {
           mBounds = new RectF(0, 0, w, h);
       }

       RectF mBounds;
   }
```

通過這樣的修改以後，PieChart.PieView.onDraw()只會在第一次現實的時候被調用。之後，pie chart會被緩存為一張圖片，並通過GPU來進行重畫不同的角度。GPU特別擅長這類的事情，並且表現效果突出。

緩存圖片到hardware layer會消耗video memory，而video memory又是有限的。基於這樣的考慮，僅僅在用戶觸發scrolling的時候使用`LAYER_TYPE_HARDWARE`，在其他時候，使用`LAYER_TYPE_NONE`。
