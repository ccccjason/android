# 使用Drawables

> 編寫: [allenlsy](https://github.com/allenlsy) - 原文: <https://developer.android.com/training/material/drawables.html>

## 使用Drawable

以下這些drawable的功能，能幫助你在應用中實現Material Design：

* Drawable染色
* 提取主色調
* 矢量Drawable

本課教你如何在應用中使用這些特性：

## 給 Drawable 資源染色

使用 Android 5.0 (API level 21)以上版本，你可以使用alpha mask（透明度圖層，譯者注）給位圖和nine patches圖片染色。你可以用顏色Resource或者主題屬性來獲取顏色（比如，`?android:attr/colorPrimary`）。通常，你只需要創建一次這些顏色asset，便可以在主題中自動匹配這些顏色。

你可以用`setTint()`方法將一種染色方式應用到`BitmapDrawable`或者`NinePatchDrawable`對象。你也在layout中使用`android:tint`和`android:initMode`屬性設置染色的顏色和模式。

## 從圖片中提取主色調

Android Support Library v21及更高版本帶有`Palatte`類，可以讓你從圖片中提取主色調。這個類可以提取以下顏色：

* Vibrant: 亮色
* Vibrant dark: 深亮色
* Vibrant light: 淺亮色
* Muted: 暗色
* Muted dark: 深暗色
* Muted light: 淺暗色

提取這些顏色時，在你載入圖片的後臺線程中傳入一個Bitmap對象給`Palette.generate()`靜態方法。如果你不能使用那個線程，可以調用`Palatte.generateAsync()`方法，並提供一個listener。

你可以用Palette類的一個getter方法從圖片獲取主色調，比如`Palette.getVibrantColor()`。

要使用`Palette`類，在你的應用模塊的Gradle依賴中添加以下代碼：

```
dependencies {
    ...
    compile 'com.android.support:palette-v7:21.0.+'
}
```

更多信息，請參見[Palette](http://developer.android.com/reference/android/support/v7/graphics/Palette.html)類的API文檔。

## 創建矢量Drawable

在Android 5.0 (API level 21)以上版本中，你可以定義矢量drawable，用於無損的拉伸圖片。相對於一張普通圖片需要為每個不同屏幕密度的設備提供一個圖片來說，一個矢量圖片只需要一個asset文件。要創建矢量圖片，你可以在`<vector>` XML元素中定義形狀。

以下代碼定義了一個心形：

```xml
<!-- res/drawable/heart.xml -->
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    <!-- intrinsic size of the drawable -->
    android:height="256dp"
    android:width="256dp"
    <!-- size of the virtual canvas -->
    android:viewportWidth="32"
    android:viewportHeight="32">

  <!-- draw a path -->
  <path android:fillColor="#8fff"
      android:pathData="M20.5,9.5
                        c-1.955,0,-3.83,1.268,-4.5,3
                        c-0.67,-1.732,-2.547,-3,-4.5,-3
                        C8.957,9.5,7,11.432,7,14
                        c0,3.53,3.793,6.257,9,11.5
                        c5.207,-5.242,9,-7.97,9,-11.5
                        C25,11.432,23.043,9.5,20.5,9.5z" />
</vector>
```

矢量圖片在Android中用[VectorDrawable](http://developer.android.com/reference/android/graphics/drawable/VectorDrawable.html)對象來表示。更多關於`pathData`語法的信息，請看[SVG Path](http://www.w3.org/TR/SVG11/paths.html#PathData)的文檔。更多關於矢量drawable動畫的信息，請參見[矢量drawable動畫](https://developer.android.com/training/material/animations.html#AnimVector)。
