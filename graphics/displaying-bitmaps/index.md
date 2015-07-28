# 高效顯示Bitmap

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/displaying-bitmaps/index.html>

這一章節會介紹一些處理與加載[Bitmap](http://developer.android.com/reference/android/graphics/Bitmap.html)對象的常用方法，這些技術能夠使得程序的UI不會被阻塞，並且可以避免程序超出內存限制。如果我們不注意這些，Bitmaps會迅速的消耗掉可用內存從而導致程序崩潰，出現下面的異常:`java.lang.OutofMemoryError: bitmap size exceeds VM budget.`

在Android應用中加載Bitmaps的操作是需要特別小心處理的，有下面幾個方面的原因:

* 移動設備的系統資源有限。Android設備對於單個程序至少需要16MB的內存。[Android Compatibility Definition Document (CDD)](http://source.android.com/compatibility/downloads.html), Section 3.7. Virtual Machine Compatibility 中給出了對於不同大小與密度的屏幕的最低內存需求。 應用應該在這個最低內存限制下去優化程序的效率。當然，大多數設備的都有更高的限制需求。
* Bitmap會消耗很多內存，特別是對於類似照片等內容更加豐富的圖片。 例如，[Galaxy Nexus](http://www.android.com/devices/detail/galaxy-nexus)的照相機能夠拍攝2592x1936 pixels (5 MB)的圖片。 如果bitmap的圖像配置是使用[ARGB_8888](http://developer.android.com/reference/android/graphics/Bitmap.Config.html) (從Android 2.3開始的默認配置) ，那麼加載這張照片到內存大約需要19MB(`2592*1936*4` bytes) 的空間，從而迅速消耗掉該應用的剩餘內存空間。
* Android應用的UI通常會在一次操作中立即加載許多張bitmaps。 例如在[ListView](http://developer.android.com/reference/android/widget/ListView.html), [GridView](http://developer.android.com/reference/android/widget/GridView.html) 與 [ViewPager](http://developer.android.com/reference/android/support/v4/view/ViewPager.html) 等控件中通常會需要一次加載許多張bitmaps，而且需要預先加載一些沒有在屏幕上顯示的內容，為用戶滑動的顯示做準備。

## 參考資料

* [DEMO：DisplayingBitmaps.zip](http://developer.android.com/downloads/samples/DisplayingBitmaps.zip)
* [VEDIO：Bitmap Allocation](http://www.youtube.com/watch?v=rsQet4nBVi8)
* [VEDIO：Making App Beautiful - Part 4 - Performance Tuning](http://www.youtube.com/watch?v=pMRnGDR6Cu0)


## 章節課程

* [**高效的加載大圖(Loading Large Bitmaps Efficiently)**](load-bitmap.html)

  這節課會帶領你學習如何解析很大的Bitmaps並且避免超出程序的內存限制。


* [**非UI線程處理Bitmap(Processing Bitmaps Off the UI Thread)**](process-bitmap.html)

  處理Bitmap（裁剪，下載等操作）不能執行在主線程。這節課會帶領你學習如何使用AsyncTask在後臺線程對Bitmap進行處理，並解釋如何處理併發帶來的問題。


* [**緩存Bitmaps(Caching Bitmaps)**](cache-bitmap.html)

  這節課會帶領你學習如何使用內存與磁盤緩存來提升加載多張Bitmaps時的響應速度與流暢度。


* [**管理Bitmap的內存使用(Managing Bitmap Memory)**](manage-memory.html)

  這節課會介紹如何管理Bitmap的內存佔用，以此來提升程序的性能。


* [**在UI上顯示Bitmap(Displaying Bitmaps in Your UI)**](display-bitmap.html)

  這節課會綜合之前章節的內容，演示如何在諸如[ViewPager](http://developer.android.com/reference/android/support/v4/view/ViewPager.html)與[GridView](http://developer.android.com/reference/android/widget/GridView.html)等控件中使用後臺線程與緩存加載多張Bitmaps。
