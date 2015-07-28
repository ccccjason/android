# 兼容不同的屏幕密度

> 編寫:[riverfeng](https://github.com/riverfeng) - 原文:<http://developer.android.com/training/multiscreen/screendensities.html>

這節課將教你如何通過提供不同的資源和使用獨立分辨率（dp）來支持不同的屏幕密度。

## 使用密度獨立像素（dp）

設計佈局時，要避免使用絕對像素（absolutepixels）定義距離和尺寸。使用像素單位來定義佈局大小是有問題的。因為，不同的屏幕有不同的像素密度，所以，同樣單位的像素在不同的設備上會有不同的物理尺寸。因此，在指定單位的時候，通常使用dp或者sp。一個dp代表一個密度獨立像素，也就相當於在160 dpi的一個像素的物理尺寸，sp也是一個基本的單位，不過它主要是用在文本尺寸上（它也是一種尺寸規格獨立的像素），所以，你在定義文本尺寸的時候應該使用這種規格單位（不要使用在布尺寸上）。

例如，當你是定義兩個view之間的空間時，應該使用dp而不是px：
```xml
<Button android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/clickme"
    android:layout_marginTop="20dp" />
```
當指定文本尺寸時，始終應該使用sp：
```xml
<TextView android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:textSize="20sp" />
```

## 提供可供選擇的圖片

因為Android能運行在很多不同屏幕密度的設備上，所以，你應該針對不同的設備密度提供不同的bitmap資源：小屏幕（low），medium（中），high（高）以及超高（extra-high）密度。這將能幫助你在所有的屏幕密度中得到非常好的圖形質量和性能。

為了提供更好的用戶體驗，你應該使用以下幾種規格來縮放圖片大小，為不同的屏幕密度提供相應的位圖資源：
```xml
xhdpi:2.0
hdpi:1.5
mdpi:1.0(標準線)
ldpi:0.75
```

這也就意味著如果在xhdpi設備上你需要一個200x200的圖片，那麼你則需要一張150x150的圖片用於hdpi，100x100的用於mdpi以及75x75的用戶ldpi設備。

然後將這些圖片資源放到res/對應的目錄下面，系統會自動根據當前設備屏幕密度自動去選擇合適的資源進行加載：
```xml
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
這樣放置圖片資源後，不論你什麼時候使用@drawable/awesomeimage，系統都會給予屏幕的dp來選擇合適的圖片。

如果你想知道更多關於如何為你的應用程序創建icon資源，你可以看看Icon設計指南[Icon Design Guidelines](http://developer.android.com/guide/practices/ui_guidelines/icon_design.html).
