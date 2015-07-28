# 實現自定義View的繪製

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/custom-view/custom-draw.html>

自定義view的最重要的一個部分是自定義它的外觀。根據你的程序的需求，自定義繪製可能簡單也可能很複雜。這節課會演示一些最常見的操作。

## Override onDraw()
重繪一個自定義的view的最重要的步驟是重寫onDraw()方法。onDraw()的參數是一個Canvas對象。Canvas類定義了繪製文本，線條，圖像與許多其他圖形的方法。你可以在onDraw方法裡面使用那些方法來創建你的UI。

在你調用任何繪製方法之前，你需要創建一個Paint對象。

<!-- more -->

## 創建繪圖對象
android.graphics framework把繪製定義為下面兩類:

* 繪製什麼，由Canvas處理
* 如何繪製，由Paint處理

例如Canvas提供繪製一條直線的方法，Paint提供直線顏色。Canvas提供繪製矩形的方法，Paint定義是否使用顏色填充。簡單來說：Canvas定義你在屏幕上畫的圖形，而Paint定義顏色，樣式，字體，

所以在繪製之前，你需要創建一個或者多個Paint對象。在這個PieChart 的例子，是在`init()`方法實現的，由constructor調用。

```java
private void init() {
   mTextPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
   mTextPaint.setColor(mTextColor);
   if (mTextHeight == 0) {
       mTextHeight = mTextPaint.getTextSize();
   } else {
       mTextPaint.setTextSize(mTextHeight);
   }

   mPiePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
   mPiePaint.setStyle(Paint.Style.FILL);
   mPiePaint.setTextSize(mTextHeight);

   mShadowPaint = new Paint(0);
   mShadowPaint.setColor(0xff101010);
   mShadowPaint.setMaskFilter(new BlurMaskFilter(8, BlurMaskFilter.Blur.NORMAL));

   ...
```

剛開始就創建對象是一個重要的優化技巧。Views會被頻繁的重新繪製，初始化許多繪製對象需要花費昂貴的代價。在onDraw方法裡面創建繪製對象會嚴重影響到性能並使得你的UI顯得卡頓。

## 處理佈局事件
為了正確的繪製你的view，你需要知道view的大小。複雜的自定義view通常需要根據在屏幕上的大小與形狀執行多次layout計算。而不是假設這個view在屏幕上的顯示大小。即使只有一個程序會使用你的view，仍然是需要處理屏幕大小不同，密度不同，方向不同所帶來的影響。

儘管view有許多方法是用來計算大小的，但是大多數是不需要重寫的。如果你的view不需要特別的控制它的大小，唯一需要重寫的方法是[onSizeChanged()](http://developer.android.com/reference/android/view/View.html#onSizeChanged(int, int, int, int)).

onSizeChanged()，當你的view第一次被賦予一個大小時，或者你的view大小被更改時會被執行。在onSizeChanged方法裡面計算位置，間距等其他與你的view大小值。

當你的view被設置大小時，layout manager(佈局管理器)假定這個大小包括所有的view的內邊距(padding)。當你計算你的view大小時，你必須處理內邊距的值。這段`PieChart.onSizeChanged()`中的代碼演示該怎麼做:

```java
       // Account for padding
       float xpad = (float)(getPaddingLeft() + getPaddingRight());
       float ypad = (float)(getPaddingTop() + getPaddingBottom());

       // Account for the label
       if (mShowText) xpad += mTextWidth;

       float ww = (float)w - xpad;
       float hh = (float)h - ypad;

       // Figure out how big we can make the pie.
       float diameter = Math.min(ww, hh);
```

如果你想更加精確的控制你的view的大小，需要重寫[onMeasure()](http://developer.android.com/reference/android/view/View.html#onMeasure(int, int))方法。這個方法的參數是View.MeasureSpec，它會告訴你的view的父控件的大小。那些值被包裝成int類型，你可以使用靜態方法來獲取其中的信息。

這裡是一個實現[onMeasure()](http://developer.android.com/reference/android/view/View.html#onMeasure)的例子。在這個例子中`PieChart`試著使它的區域足夠大，使pie可以像它的label一樣大:

```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
   // Try for a width based on our minimum
   int minw = getPaddingLeft() + getPaddingRight() + getSuggestedMinimumWidth();
   int w = resolveSizeAndState(minw, widthMeasureSpec, 1);

   // Whatever the width ends up being, ask for a height that would let the pie
   // get as big as it can
   int minh = MeasureSpec.getSize(w) - (int)mTextWidth + getPaddingBottom() + getPaddingTop();
   int h = resolveSizeAndState(MeasureSpec.getSize(w) - (int)mTextWidth, heightMeasureSpec, 0);

   setMeasuredDimension(w, h);
}
```

上面的代碼有三個重要的事情需要注意:

* 計算的過程有把view的padding考慮進去。這個在後面會提到，這部分是view所控制的。
* 幫助方法resolveSizeAndState()是用來創建最終的寬高值的。這個方法會通過比較view的需求大小與spec值，返回一個合適的View.MeasureSpec值，並傳遞到onMeasure方法中。
* onMeasure()沒有返回值。它通過調用setMeasuredDimension()來獲取結果。調用這個方法是強制執行的，如果你遺漏了這個方法，會出現運行時異常。

## 繪圖!
每個view的onDraw都是不同的，但是有下面一些常見的操作：

* 繪製文字使用drawText()。指定字體通過調用setTypeface(), 通過setColor()來設置文字顏色.
* 繪製基本圖形使用drawRect(), drawOval(), drawArc(). 通過setStyle()來指定形狀是否需要filled, outlined.
* 繪製一些複雜的圖形，使用Path類. 通過給Path對象添加直線與曲線, 然後使用drawPath()來繪製圖形. 和基本圖形一樣，paths也可以通過setStyle來設置是outlined, filled, both.
* 通過創建LinearGradient對象來定義漸變。調用setShader()來使用LinearGradient。
* 通過使用drawBitmap來繪製圖片.

```java
protected void onDraw(Canvas canvas) {
   super.onDraw(canvas);

   // Draw the shadow
   canvas.drawOval(
           mShadowBounds,
           mShadowPaint
   );

   // Draw the label text
   canvas.drawText(mData.get(mCurrentItem).mLabel, mTextX, mTextY, mTextPaint);

   // Draw the pie slices
   for (int i = 0; i < mData.size(); ++i) {
       Item it = mData.get(i);
       mPiePaint.setShader(it.mShader);
       canvas.drawArc(mBounds,
               360 - it.mEndAngle,
               it.mEndAngle - it.mStartAngle,
               true, mPiePaint);
   }

   // Draw the pointer
   canvas.drawLine(mTextX, mPointerY, mPointerX, mPointerY, mTextPaint);
   canvas.drawCircle(mPointerX, mPointerY, mPointerSize, mTextPaint);
}
```
