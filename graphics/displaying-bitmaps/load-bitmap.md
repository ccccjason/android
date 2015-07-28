# 高效加載大圖

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/displaying-bitmaps/load-bitmap.html>

圖片有不同的形狀與大小。在大多數情況下它們的實際大小都比需要呈現的尺寸大很多。例如，系統的圖庫應用會顯示那些我們使用相機拍攝的照片，但是那些圖片的分辨率通常都比設備屏幕的分辨率要高很多。

考慮到應用是在有限的內存下工作的，理想情況是我們只需要在內存中加載一個低分辨率的照片即可。為了更便於顯示，這個低分辨率的照片應該是與其對應的UI控件大小相匹配的。加載一個超過屏幕分辨率的高分辨率照片不僅沒有任何顯而易見的好處，還會佔用寶貴的內存資源，另外在快速滑動圖片時容易產生額外的效率問題。

這一課會介紹如何通過加載一個縮小版本的圖片，從而避免超出程序的內存限制。

## 讀取位圖的尺寸與類型(Read Bitmap Dimensions and Type)

[BitmapFactory](http://developer.android.com/reference/android/graphics/BitmapFactory.html)提供了一些解碼（decode）的方法（<a href="http://developer.android.com/reference/android/graphics/BitmapFactory.html#decodeByteArray(byte[], int, int, android.graphics.BitmapFactory.Options)">decodeByteArray()</a>, <a href="http://developer.android.com/reference/android/graphics/BitmapFactory.html#decodeFile(java.lang.String, android.graphics.BitmapFactory.Options)">decodeFile()</a>, <a href="http://developer.android.com/reference/android/graphics/BitmapFactory.html#decodeResource(android.content.res.Resources, int, android.graphics.BitmapFactory.Options)">decodeResource()</a>等），用來從不同的資源中創建一個Bitmap。 我們應該根據圖片的數據源來選擇合適的解碼方法。 這些方法在構造位圖的時候會嘗試分配內存，因此會容易導致`OutOfMemory`的異常。每一種解碼方法都可以通過[BitmapFactory.Options](http://developer.android.com/reference/android/graphics/BitmapFactory.Options.html)設置一些附加的標記，以此來指定解碼選項。設置 [inJustDecodeBounds](http://developer.android.com/reference/android/graphics/BitmapFactory.Options.html#inJustDecodeBounds) 屬性為`true`可以在解碼的時候避免內存的分配，它會返回一個`null`的Bitmap，但是可以獲取到 outWidth, outHeight 與 outMimeType。該技術可以允許你在構造Bitmap之前優先讀圖片的尺寸與類型。

```java
BitmapFactory.Options options = new BitmapFactory.Options();
options.inJustDecodeBounds = true;
BitmapFactory.decodeResource(getResources(), R.id.myimage, options);
int imageHeight = options.outHeight;
int imageWidth = options.outWidth;
String imageType = options.outMimeType;
```

為了避免`java.lang.OutOfMemory` 的異常，我們需要在真正解析圖片之前檢查它的尺寸（除非你能確定這個數據源提供了準確無誤的圖片且不會導致佔用過多的內存）。

## 加載一個按比例縮小的版本到內存中(Load a Scaled Down Version into Memory)

通過上面的步驟我們已經獲取到了圖片的尺寸，這些數據可以用來幫助我們決定應該加載整個圖片到內存中還是加載一個縮小的版本。有下面一些因素需要考慮：

* 評估加載完整圖片所需要耗費的內存。
* 程序在加載這張圖片時可能涉及到的其他內存需求。
* 呈現這張圖片的控件的尺寸大小。
* 屏幕大小與當前設備的屏幕密度。

例如，如果把一個大小為1024x768像素的圖片顯示到大小為128x96像素的ImageView上嗎，就沒有必要把整張原圖都加載到內存中。

為了告訴解碼器去加載一個縮小版本的圖片到內存中，需要在[BitmapFactory.Options](http://developer.android.com/reference/android/graphics/BitmapFactory.Options.html) 中設置 inSampleSize 的值。例如, 一個分辨率為2048x1536的圖片，如果設置 inSampleSize 為4，那麼會產出一個大約512x384大小的Bitmap。加載這張縮小的圖片僅僅使用大概0.75MB的內存，如果是加載完整尺寸的圖片，那麼大概需要花費12MB（前提都是Bitmap的配置是 ARGB_8888）。下面有一段根據目標圖片大小來計算Sample圖片大小的代碼示例：

```java
public static int calculateInSampleSize(
            BitmapFactory.Options options, int reqWidth, int reqHeight) {
    // Raw height and width of image
    final int height = options.outHeight;
    final int width = options.outWidth;
    int inSampleSize = 1;

    if (height > reqHeight || width > reqWidth) {

        final int halfHeight = height / 2;
        final int halfWidth = width / 2;

        // Calculate the largest inSampleSize value that is a power of 2 and keeps both
        // height and width larger than the requested height and width.
        while ((halfHeight / inSampleSize) > reqHeight
                && (halfWidth / inSampleSize) > reqWidth) {
            inSampleSize *= 2;
        }
    }

    return inSampleSize;
}
```

> **Note:** 設置[inSampleSize](http://developer.android.com/reference/android/graphics/BitmapFactory.Options.html#inSampleSize)為2的冪是因為解碼器最終還是會對非2的冪的數進行向下處理，獲取到最靠近2的冪的數。詳情參考[inSampleSize](http://developer.android.com/reference/android/graphics/BitmapFactory.Options.html#inSampleSize)的文檔。

為了使用該方法，首先需要設置 [inJustDecodeBounds](http://developer.android.com/reference/android/graphics/BitmapFactory.Options.html#inJustDecodeBounds) 為 `true`, 把options的值傳遞過來，然後設置 [inSampleSize](http://developer.android.com/reference/android/graphics/BitmapFactory.Options.html#inSampleSize) 的值並設置 inJustDecodeBounds 為 `false`，之後重新調用相關的解碼方法。

```java
public static Bitmap decodeSampledBitmapFromResource(Resources res, int resId,
        int reqWidth, int reqHeight) {

    // First decode with inJustDecodeBounds=true to check dimensions
    final BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    BitmapFactory.decodeResource(res, resId, options);

    // Calculate inSampleSize
    options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);

    // Decode bitmap with inSampleSize set
    options.inJustDecodeBounds = false;
    return BitmapFactory.decodeResource(res, resId, options);
}
```

使用上面這個方法可以簡單地加載一張任意大小的圖片。如下面的代碼樣例顯示了一個**接近** 100x100像素的縮略圖：

```java
mImageView.setImageBitmap(
    decodeSampledBitmapFromResource(getResources(), R.id.myimage, 100, 100));
```

我們可以通過替換合適的<a href="http://developer.android.com/reference/android/graphics/BitmapFactory.html#decodeByteArray(byte[], int, int, android.graphics.BitmapFactory.Options)">BitmapFactory.decode*</a> 方法來實現一個類似的方法，從其他的數據源解析Bitmap。
