# 維護兼容性

> 編寫: [allenlsy](https://github.com/allenlsy) - 原文: <https://developer.android.com/training/material/compatibility.html>

有些Material Design特性，比如主題和自定義Activits切換效果等，只在Android 5.0 (API level 21) 以上中可用。不過，你仍然可以使用這些特性實現Material Design，並保持對舊版本Android 系統的兼容。

## 定義備選Style

你可以配置你的應用，在支持Material Design的設備上使用Material主題，在舊版本Android上使用舊的主題：

1. 在`res/values/styles/xml`中定義一個主題繼承自舊主題（比如Holo）
2. 在`res/values-v21/styles.xml`中定義一個同名的主題，繼承自Material 主題
3. 在`AndroidManifest.xml`中，將這個主題設置為應用的主題

> **Note:** 如果你的應用設置了一個主題，但是沒有提供備選Style，你可能無法在低於Android 5.0版本的系統中運行應用。

## 提供備選layout

如果你根據Material Design設計的應用的Layout中沒有使用任何Android 5.0 (API level 21)中新的XML屬性，他們在舊版本Android中就能正常工作。否則，你要提供備選Layout。你可以在備選Layout中定義你的應用在舊版本系統中的界面。

在`res/layout-v21/`中定義Android 5.0 (API level 21) 以上系統的Layout，在`res/layout`中定義早前版本Android的Layout。比如，`res/layout/my_activity.xml`是對於`res/layout-v21/my_activity.xml`的一個備選Layout。

為了避免代碼重複，在`res/values`中定義style，然後在`res/values-v21`中修改新API需要的style。使用style的繼承，在`res/values/`中定義父style，在`res/values-v21/`中繼承。

## 使用 Support Library

[v7 support libraries r21](https://developer.android.com/tools/support-library/features.html#v7) 及更高版本包含了以下Material Design 特性：

* 當你應用一個 `Theme.AppCompat` 主題時， 會得到為一些系統控件準備的 Material Design style
* `Theme.AppCompat`主題包含調色板主體屬性
* `RecyclerView` 組件用於顯示數據集
* `CardView` 組件用於創建卡片
* `Palette` 類用於從圖片提取主色調

### 系統組件

`Theme.AppCompat` 主題中提供了這些組件的 Material Design style：

* EditText
* Spinner
* CheckBox
* RadioButton
* SwitchCompat
* CheckedTextView

### 調色板

要獲取Material Design style，並用v7 support library自定義調色板，就要應用以下中的一個Theme.AppCompat主題：

```xml
<!-- extend one of the Theme.AppCompat themes -->
<style name="Theme.MyTheme" parent="Theme.AppCompat.Light">
    <!-- customize the color palette -->
    <item name="colorPrimary">@color/material_blue_500</item>
    <item name="colorPrimaryDark">@color/material_blue_700</item>
    <item name="colorAccent">@color/material_green_A200</item>
</style>
```

### 列表和卡片

`RecyclerView`和`CardView`組件可通過v7 support libraries支持舊版本Android，但有以下限制：

* CardView需要編程實現陰影和其他的padding
* CardView不能將附著與原件有重合部分的子視圖

### 依賴

要在Android 5.0之前的版本使用這些特性，需要在項目的Gradle依賴中加入Android v7 support library:

```
dependencies {
    compile 'com.android.support:appcompat-v7:21.0.+'
    compile 'com.android.support:cardview-v7:21.0.+'
    compile 'com.android.support:recyclerview-v7:21.0.+'
}
```

## 檢查系統版本

以下特性只在Android 5.0 (API level 21) 及以上版本中可用：

* Activity 切換動畫
* 觸摸反饋
* Reveal 動畫（填充動畫效果，譯者注）
* 基於路徑的動畫
* 矢量drawable
* Drawable染色

要保持向下兼容，請在使用這些特性是，使用以下代碼在運行時檢查系統版本：

```java
// Check if we're running on Android 5.0 or higher
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
    // Call some material design APIs here
} else {
    // Implement this feature without material design
}
```

> **Note:**要聲明應用支持哪些Android 版本，在manifest文件中使用`android:minSdkVersion`和`android:targetSdkVersion`屬性。要在Android 5.0中使用Material Design特性，設置`android:targetSdkVersion`屬性為21。更多信息，參見[`<uses-sdk>` API指南](https://developer.android.com/guide/topics/manifest/uses-sdk-element.html)。
