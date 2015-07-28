# 打印照片

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/printing/photos.html>

拍攝並分享照片是移動設備最流行的用法之一。如果我們的應用拍攝了照片，並期望可以展示他們，或者允許用戶共享照片，那麼我們就應該考慮讓應用可以打印出這些照片來。[Android Support Library](http://developer.android.com/tools/support-library/index.html)提供了一個方便的函數，通過這一函數，僅僅使用很少量的代碼和一些簡單的打印佈局配置集，就能夠進行照片打印。

這堂課將展示如何使用v4 support library中的[PrintHelper](http://developer.android.com/reference/android/support/v4/print/PrintHelper.html)類打印一幅圖片。

## 打印一幅圖片

Android Support Library中的[PrintHelper](http://developer.android.com/reference/android/support/v4/print/PrintHelper.html)類提供了一種打印圖片的簡單方法。該類有一個單一的佈局選項：<a href="http://developer.android.com/reference/android/support/v4/print/PrintHelper.html#setScaleMode(int)">setScaleMode()</a>，它允許我們使用下面的兩個選項之一：
* [SCALE_MODE_FIT](http://developer.android.com/reference/android/support/v4/print/PrintHelper.html#SCALE_MODE_FIT)：該選項會調整圖像的大小，這樣整個圖像就會在打印有效區域內全部顯示出來（等比例縮放至長和寬都包含在紙張頁面內）。
* [SCALE_MODE_FILL](http://developer.android.com/reference/android/support/v4/print/PrintHelper.html#SCALE_MODE_FILL)：該選項同樣會等比例地調整圖像的大小使圖像充滿整個打印有效區域，即讓圖像充滿整個紙張頁面。這就意味著如果選擇這個選項，那麼圖片的一部分（頂部和底部，或者左側和右側）將無法打印出來。如果不設置圖像的打印佈局選項，該模式將是默認的圖像拉伸方式。

這兩個<a href="http://developer.android.com/reference/android/support/v4/print/PrintHelper.html#setScaleMode(int)">setScaleMode()</a>的圖像佈局選項都會保持圖像原有的長寬比。下面的代碼展示瞭如何創建一個[PrintHelper](http://developer.android.com/reference/android/support/v4/print/PrintHelper.html)類的實例，設置佈局選項，並開始打印進程：

```java
private void doPhotoPrint() {
    PrintHelper photoPrinter = new PrintHelper(getActivity());
    photoPrinter.setScaleMode(PrintHelper.SCALE_MODE_FIT);
    Bitmap bitmap = BitmapFactory.decodeResource(getResources(),
            R.drawable.droids);
    photoPrinter.printBitmap("droids.jpg - test print", bitmap);
}
```

該方法可以作為一個菜單項的Action來被調用。注意對於那些不一定被設備支持的菜單項（比如有些設備可能無法支持打印），應該放置在“更多菜單（overflow menu）”中。要獲取有關這方面的更多知識，可以閱讀：[Action Bar](http://developer.android.com/design/patterns/actionbar.html)。

在<a href="http://developer.android.com/reference/android/support/v4/print/PrintHelper.html#printBitmap(java.lang.String, android.graphics.Bitmap)">printBitmap()</a>被調用之後，我們的應用就不再需要進行其他的操作了。之後Android打印界面就會出現，允許用戶選擇一個打印機和它的打印選項。用戶可以打印圖像或者取消這一次操作。如果用戶選擇了打印圖像，那麼一個打印任務將會被創建，同時在系統的通知欄中會顯示一個打印提醒通知。

如果希望在打印輸出中包含更多的內容，而不僅僅是一張圖片，那麼就必須構造一個打印文檔。這方面知識將會在後面的兩節課程中展開。
