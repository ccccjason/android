<!-- # Building Layouts for TV # -->
# 創建TV佈局

> 編寫:[awong1900](https://github.com/awong1900) - 原文:<http://developer.android.com/training/tv/start/layouts.html>

<!-- A TV screen is typically viewed from about 10 feet away, and while it is much larger than most other Android device displays, this type of screen does not provide the same level of precise detail and color as a smaller device. These factors require you to create app layouts with TV devices in mind in order to create a useful and enjoyable user experience. -->

TV通常在3米外觀看，並且它比大部分Android設備大的多。這類屏不能達到類似小設備的精細細節和顏色的水平。這些因素需要我們在頭腦中考慮，並設計出對於TV設備更為有用且好用的應用佈局。

<!-- This lesson describes the minimum requirements and implementation details for building effective layouts in TV apps. -->

這節課程描述了創建有效的TV應用佈局的基本要求和實現細節。

<!-- ## Use Layout Themes for TV ## -->
## 用TV佈局主題

<!-- Android Themes can provide a basis for layouts in your TV apps. You should use a theme to modify the display of your app activities that are meant to run on a TV device. This section explains which themes you should use. -->

Android主題能給我們的TV應用佈局提供基礎框架。對於打算在TV設備上運行的應用activity，我們應該用一款主題改變它的顯示。這節課程教我們應該用哪個主題。

<!-- ### Leanback theme ### -->
### Leanback主題

<!-- A support library for TV user interfaces called the v17 leanback library provides a standard theme for TV activities, called Theme.Leanback. This theme establishes a consistent visual style for TV apps. Use of this theme is recommended for most TV apps. This theme is strongly recommended for any TV app that uses v17 leanback classes. The following code sample shows how to apply this theme to a given activity within an app: -->

支持TV用戶界面的庫叫做[v17 leanback libarary](http://developer.android.com/tools/support-library/features.html#v17-leanback)，它提供了一個標準的TV activity主題，叫做`Theme.Leanback`。這一主題為TV應用程序建立了一致的視覺風格。強烈推薦在任何用了v17 leanback類的TV應用中使用這個主題。接下來的代碼展示如何在應用中對給定的activity使用這個主題：

```xml
<activity
  android:name="com.example.android.TvActivity"
  android:label="@string/app_name"
  android:theme="@style/Theme.Leanback">
```

<!-- ### NoTitleBar theme ### -->
### NoTitleBar主題

<!-- The title bar is a standard user interface element for Android apps on phones and tablets, but it is not appropriate for TV apps. If you are not using v17 leanback classes, you should apply this theme to your TV activities to suppress the display of a title bar. The following code example from a TV app manifest demonstrates how to apply this theme to remove the display of a title bar: -->

在手機和平板的Android應用中，標題欄是標準的用戶界面元素。但是在TV應用中是不適合的。如果沒有用v17 leanback類，我們應該在TV activity使用這個主題來隱去標題欄的顯示。接下來的TV應用manifest代碼示範瞭如何應用這個主題來刪除標題欄。

```xml
<application>
  ...

  <activity
    android:name="com.example.android.TvActivity"
    android:label="@string/app_name"
    android:theme="@android:style/Theme.NoTitleBar">
    ...

  </activity>
</application>
```

<!-- ## Build Basic TV Layouts ## -->
## 創建基本的TV佈局

<!-- Layouts for TV devices should follow some basic guidelines to ensure they are usable and effective on large screens. Follow these tips to build landscape layouts optimized for TV screens: -->

TV設備的佈局應該遵循一些基本的指引確保它們在大屏幕下是可用的和有效率的。遵循這些技巧去創建最優化的TV橫屏佈局。

<!--
- Build layouts with a landscape orientation. TV screens always display in landscape mode.
- Put on-screen navigation controls on the left or right side of the screen and save the vertical space for content.
- Create UIs that are divided into sections, using Fragments, and use view groups like GridView instead of ListView to make better use of the horizontal screen space.
- Use view groups such as RelativeLayout or LinearLayout to arrange views. This approach allows the system to adjust the position of the views to the size, alignment, aspect ratio, and pixel density of a TV screen.
- Add sufficient margins between layout controls to avoid a cluttered UI.
-->

- 創建橫屏佈局。TV屏幕總是顯示在橫屏模式。
- 把導航控件放置在屏幕的左邊或者右邊，並且保持內容在垂直區間。
- 創建分離的UI，用[Fragment](http://developer.android.com/guide/components/fragments.html)，並且用框架如[GridView](http://developer.android.com/reference/android/widget/GridView.html)代替[ListView](http://developer.android.com/reference/android/widget/ListView.html)獲得屏幕水平方向上更好的使用。
- 用框架如[RelativeLayout](http://developer.android.com/reference/android/widget/RelativeLayout.html)或者[LinearLayout](http://developer.android.com/reference/android/widget/LinearLayout.html)來排列視圖。基於對齊方式，縱橫比，和電視屏幕的像素密度，這個方法允許系統調整視圖大小的位置。
- 在佈局控件之間添加足夠的邊際，以避免成為一個雜亂的UI。

<!-- ### Overscan ### -->
### Overscan

<!-- Layouts for TV have some unique requirements due to the evolution of TV standards and the desire to always present a full screen picture to viewers. For this reason, TV devices may clip the outside edge of an app layout in order to ensure that the entire display is filled. This behavior is generally referred to as overscan. -->

由於TV標準的演進，TV的佈局有一個獨特的需求是總是希望給觀眾顯示全屏圖像。因為這個原因，TV設備可能剪掉應用佈局的外邊緣去確保整個顯示器被填滿。這種行為通常簡稱為overscan。

<!--  Avoid screen elements being clipped due to overscan and by incorporating a 10% margin on all sides of your layout. This translates into a 48dp margin on the left and right edges and a 27dp margin on the top and bottom of your base layouts for activities. The following example layout demonstrates how to set these margins in the root layout for a TV app: -->

避免屏幕元素由於overscan被剪掉，可以在佈局所有的邊緣增加總共10%的邊際。這換算為在activity的基礎佈局上左右邊緣留48dp的邊際和在上下留27dp的邊際。接下來的佈局例子展示瞭如何在TV應用根佈局上設置這些邊際。

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
  android:id="@+id/base_layout"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  android:orientation="vertical"
  android:layout_marginTop="27dp"
  android:layout_marginLeft="48dp"
  android:layout_marginRight="48dp"
  android:layout_marginBottom="27dp" >
</LinearLayout>
```

<!-- >**Caution**: Do not apply overscan margins to your layout if you are using the v17 leanback classes, such as BrowseFragment or related widgets, as those layouts already incorporate overscan-safe margins. -->

>**Caution**：如果我們正在使用v17 leanback類，不要在佈局中留overscan邊際，諸如[BrowseFragment](http://developer.android.com/reference/android/support/v17/leanback/app/BrowseFragment.html)或者相關控件，因為那些佈局已經包含了overscan安全邊際。

<!-- ## Build Useable Text and Controls ## -->
## 創建方便使用的文本和控件

<!-- The text and controls in a TV app layout should be easily visible and navigable from a distance. Follow these tips to make your user interface elements easier to see from a distance: -->

在TV應用佈局中的文本和控件應該在一定距離外是容易查看和導航的。接下來的技巧是確保我們的用戶界面元素在一定距離外更容易查看。

<!--
- Break text into small chunks that users can quickly scan.
- Use light text on a dark background. This style is easier to read on a TV.
- Avoid lightweight fonts or fonts that have both very narrow and very broad strokes. Use simple sans-serif fonts and anti-aliasing to increase readability.
- Use Android's standard font sizes:
-->

- 分解文本為小塊，用戶可以快速瀏覽。
- 在暗背景下用亮色文字。這種風格在TV中更容易閱讀。
- 避免輕字體或者字體既窄且有非常寬闊的筆觸效果。用簡單的sans-serif字體並且去掉鋸齒效果以增加可讀性。
- 用Android標準的字體大小。

    ```xml
    <TextView
          android:id="@+id/atext"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:gravity="center_vertical"
          android:singleLine="true"
          android:textAppearance="?android:attr/textAppearanceMedium"/>
    ```

<!--
- Ensure that all your view widgets are large enough to be clearly visible to someone sitting 10 feet away from the screen (this distance is greater for very large screens). The best way to do this is to use layout-relative sizing rather than absolute sizing, and density-independent pixel (dip) units instead of absolute pixel units. For example, to set the width of a widget, use wrap_content instead of a pixel measurement, and to set the margin for a widget, use dip values instead of px values.
For more information about density-independent pixels and building layouts to handle larger screen sizes, see Supporting Multiple Screens.
-->

- 確保所有的控件是足夠大，使人們站在屏幕3米外（更大的屏幕這個距離會更大）可以看清楚。做這個最好的方式是用佈局相對大小而不是絕對大小，並且用密度無關像素（dip）單位代替像素單位。例如，設置控件的寬度，用`wrap_content`代替特定像素值，並且設置控件的邊際，用dip代替px值。
更多關於密度無關像素和創建大尺寸屏幕的佈局，查看[Support Mutiple Screens](http://developer.android.com/guide/practices/screens_support.html)。

<!-- ## Manage Layout Resources for TV ## -->
## 管理TV佈局資源

<!-- The common high-definition TV display resolutions are 720p, 1080i, and 1080p. Your TV layout should target a screen size of 1920 x 1080 pixels, and then allow the Android system to downscale your layout elements to 720p if necessary. In general, downscaling (removing pixels) does not degrade your layout presentation quality. However, upscaling can cause display artifacts that degrade the quality of your layout and have a negative impact on the user experience of your app. -->

通常的高清晰度TV分辨率是720p，1080i和1080p。假定我們的TV佈局對象是一個1920 x 1080像素的屏幕，然後要允許Android系統必要情況下縮減佈局元素到720p。通常，降低分辨率（刪除像素）不會降低佈局的外觀質量。但是增加分辨率會降低佈局顯示的質量，並且會對用戶體驗造成負面影響。

<!-- To get the best scaling results for images, provide them as 9-patch image elements if possible. If you provide low quality or small images in your layouts, they will appear pixelated, fuzzy, or grainy, which is not a good experience for the user. Use high-quality images instead. -->

為了獲得最好的圖像縮放效果，儘可能提供[9-patch](http://developer.android.com/tools/help/draw9patch.html)圖片元素。如果在我們的佈局中使用低質量或者小的圖片，它們將出現馬賽克，模糊或者顆粒，這不是一個好的用戶體驗。用高質量圖片代替它。

<!-- For more information on optimizing layouts and resources for large screens see Designing for multiple screens. -->
更多關於優化佈局和大屏幕的資源文件問題，參考[Designing for multiple screens](http://developer.android.com/training/multiscreen/index.html)。

<!-- ## Avoid Layout Anti-Patterns ## -->
## 避免反模式佈局

<!--  There are a few approaches to building layouts that you should avoid because they do not work well on TV devices and lead to bad user experiences. Here are some user interface approaches you should specifically not use when developing a layout for TV. -->

有幾種創建佈局的方法我們應該避免使用，因為它們不能在TV設備上很好的工作並且導致不好的用戶體驗。當開發TV佈局時，以下一些用戶界面是我們應該明確不能使用的。

<!--
- **Re-using phone or tablet layouts** - Do not reuse layouts from a phone or tablet app without modification. Layouts built for other Android device form factors are not well suited for TV devices and should be simplified for operation on a TV.
- **ActionBar** - While this user interface convention is recommended for use on phones and tablets, it is not appropriate for a TV interface. In particular, using an action bar options menu (or any pull-down menu for that matter) is strongly discouraged, due to the difficulty in navigating such a menu with a remote control.
- **ViewPager** - Sliding between screens can work great on a phone or tablet, but don't try this on a TV!
For more information on designing layouts that are appropriate to TV, see the TV Design guide.
-->

- **重用手機和平板佈局** - 不要重用沒有修改的手機或者平板應用的佈局。為其他Android設備開發的佈局不適合TV設備，並且TV上佈局應該被簡化。
- **狀態欄** - 儘管這種用戶界面習慣是推薦使用在手機和平板上，但是他不適合TV界面。通常，狀態欄選項菜單（或者任何下拉菜單）堅決不要使用，因為用遙控器操作這樣的菜單是困難的。
- **ViewPager** - 在屏幕之間滑動能很好在手機或平板上工作，但是不要在TV上嘗試！
更多信息關於設計適合TV的佈局，參考[TV Design](http://developer.android.com/design/tv/index.html)指導。


<!-- ## Handle Large Bitmaps ## -->
## 處理大圖片

<!-- TV devices, like any other Android device, have a limited amount of memory. If you build your app layout with very high-resolution images or use many high-resolution images in the operation of your app, it can quickly run into memory limits and cause out of memory errors. To avoid these types of problems, follow these tips: -->

TV設備，像任何其他Android設備，內存有一定限制。如果我們創建的應用中用了很高分辨率的圖片或者用了很多高分辨率圖片，它可能很快達到內存限制，並且導致內存溢出錯誤。避免這些類型的問題，遵循以下方法：

<!--
- Load images only when they are displayed on the screen. For example, when displaying multiple images in a GridView or Gallery, only load an image when getView() is called on the view's Adapter.
- Call recycle() on Bitmap views that are no longer needed.
- Use WeakReference for storing references to Bitmap objects in an in-memory Collection.
- If you fetch images from the network, use AsyncTask to fetch and store them on the device for faster access. Never do network transactions on the application's main user interface thread.
- Scale down large images to a more appropriate size as you download them; otherwise, downloading the image itself may cause an out of memory exception.
For more information on getting the best performance when working with images, see Displaying Bitmaps Efficiently.
-->

- 僅當圖片顯示在屏幕時才加載。例如，當在[GridView](http://developer.android.com/reference/android/widget/GridView.html)或者[Gallery](http://developer.android.com/reference/android/widget/Gallery.html)中顯示多個圖片時，僅當[getView()](http://developer.android.com/reference/android/widget/Adapter.html#getView(int, android.view.View, android.view.ViewGroup))在視圖的[Adapter](http://developer.android.com/reference/android/widget/Adapter.html)中被調用時才加載圖片。
- 在[Bitmap](http://developer.android.com/reference/android/graphics/Bitmap.html)視圖中調用[recycle()](http://developer.android.com/reference/android/graphics/Bitmap.html#recycle())不再需要。
- 對存儲在內存中[集合](http://developer.android.com/reference/java/util/Collection.html)中的位圖對象使用[弱引用](http://developer.android.com/reference/java/lang/ref/WeakReference.html)。
- 如果我們從網絡上獲取圖片，用[AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html)去操作並且存儲它們在設備上以方便更快的存取。絕對不要在應用的主線程操作網絡傳輸。
- 當下載大圖片時，降低圖片到合適的尺寸，否則，下載圖片本身可能導致內存溢出問題。
更多信息關於獲得最好的圖片操作性能，參考 [Displaying Bitmaps Efficiently](http://developer.android.com/training/displaying-bitmaps/index.html)。

<!-- ## Provide Effective Advertising ## -->
## 提供有效的廣告

<!-- Advertising on Android TV must always be full-screen. Ads must not appear alongside or over content. The user must be able to dismiss an advertisement with the D-pad controller. Video ads must be dismissible within 30 seconds of their start time. -->

Android TV的廣告必須總是全屏。廣告不可以出現在內容的旁邊或者覆蓋內容。用戶應當能用D-pad控制器關閉廣告。視頻廣告在開始時間後的30秒內應當能被關閉。

<!-- Android TV does not provide a web browser. Your ads must not attempt to launch a web browser or redirect to the Google Play Store. -->

Android TV不提供網頁瀏覽器。我們的廣告不應該嘗試去啟動網頁瀏覽器或者重定向到Google Play商店。

<!-- >Note: You can use the WebView class for logins to services like Google+ and Facebook. -->

>**Note**：[WebView](http://developer.android.com/reference/android/webkit/WebView.html)類用於登入服務器，如Google+和Facebook。

---------------------------------------
[下一節: 創建TV導航 >](navigation.html)
