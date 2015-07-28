# 適配不同的屏幕

> 編寫:[Lin-H](http://github.com/Lin-H) - 原文:<http://developer.android.com/training/basics/supporting-devices/screens.html>

Android用尺寸和分辨率這兩種常規屬性對不同的設備屏幕加以分類。我們應該想到自己的app會被安裝在各種屏幕尺寸和分辨率的設備中。這樣，app中就應該包含一些可選資源，針對不同的屏幕尺寸和分辨率，來優化其外觀。

- 有4種普遍尺寸：小(small)，普通(normal)，大(large)，超大(xlarge)
- 4種普遍分辨率：低精度(ldpi), 中精度(mdpi), 高精度(hdpi), 超高精度(xhdpi)

聲明針對不同屏幕所用的layout和bitmap，必須把這些可選資源放置在獨立的目錄中，這與適配不同語言時的做法類似。

同樣要注意屏幕的方向(橫向或縱向)也是一種需要考慮的屏幕尺寸變化，因此許多app會修改layout，來針對不同的屏幕方向優化用戶體驗。

## 創建不同的layout

為了針對不同的屏幕去優化用戶體驗，我們需要為每一種將要支持的屏幕尺寸創建唯一的XML文件。每一種layout需要保存在相應的資源目錄中，目錄以`-<screen_size>`為後綴命名。例如，對大尺寸屏幕(large screens)，一個唯一的layout文件應該保存在`res/layout-large/`中。

> **Note**:為了匹配合適的屏幕尺寸Android會自動地測量我們的layout文件。所以不需要因不同的屏幕尺寸去擔心UI元素的大小，而應該專注於layout結構對用戶體驗的影響。(比如關鍵視圖相對於同級視圖的尺寸或位置)

例如，這個工程包含一個默認layout和一個適配大屏幕的layout：

```
MyProject/
    res/
        layout/
            main.xml
        layout-large/
            main.xml
```

layout文件的名字必須完全一樣，為了對相應的屏幕尺寸提供最優的UI，文件的內容不同。

如平常一樣在app中簡單引用：

```java
@Override
 protected void onCreate(Bundle savedInstanceState) {
     super.onCreate(savedInstanceState);
     setContentView(R.layout.main);
}
```

系統會根據app所運行的設備屏幕尺寸，在與之對應的layout目錄中加載layout。更多關於Android如何選擇恰當資源的信息，詳見[Providing Resources](https://developer.android.com/guide/topics/resources/providing-resources.html#BestMatch)。

另一個例子，這一個工程中有為適配橫向屏幕的layout:

```
MyProject/
    res/
        layout/
            main.xml
        layout-land/
            main.xml
```

默認的，`layout/main.xml`文件用作豎屏的layout。

如果想給橫屏提供一個特殊的layout，也適配於大屏幕，那麼則需要使用`large`和`land`修飾符。

```
 MyProject/
    res/
        layout/              # default (portrait)
            main.xml
        layout-land/         # landscape
            main.xml
        layout-large/        # large (portrait)
            main.xml
        layout-large-land/   # large landscape
            main.xml
```

> **Note**:Android 3.2及以上版本支持定義屏幕尺寸的高級方法，它允許我們根據屏幕最小長度和寬度，為各種屏幕尺寸指定與密度無關的layout資源。這節課程不會涉及這一新技術，更多信息詳見[Designing for Multiple Screens](../../ui/multiscreen/index.html)。

## 創建不同的bitmap

我們應該為4種普遍分辨率:低，中，高，超高精度，都提供相適配的bitmap資源。這能使我們的app在所有屏幕分辨率中都能有良好的畫質和效果。

要生成這些圖像，應該從原始的矢量圖像資源著手，然後根據下列尺寸比例，生成各種密度下的圖像。

- xhdpi: 2.0
- hdpi:  1.5
- mdpi:  1.0 (基準)
- ldpi:  0.75

這意味著，如果針對xhdpi的設備生成了一張200x200的圖像，那麼應該為hdpi生成150x150,為mdpi生成100x100, 和為ldpi生成75x75的圖片資源。

然後，將這些文件放入相應的drawable資源目錄中:

```
MyProject/
    res/
        drawable-xhdpi/
            awesomeimage.png
        drawable-hdpi/
            awesomeimage.png
        drawable-mdpi/
            awesomeimage.png
        drawable-ldpi/
            awesomeimage.png
```

任何時候，當引用`@drawable/awesomeimage`時系統會根據屏幕的分辨率選擇恰當的bitmap。

> **Note**:低密度(ldpi)資源是非必要的，當提供了hdpi的圖像，系統會把hdpi的圖像按比例縮小一半，去適配ldpi的屏幕。

更多關於為app創建圖標assets的信息和指導，詳見[Iconography design](https://developer.android.com/design/style/iconography.html)。
