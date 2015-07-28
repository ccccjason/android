# 建立ActionBar

> 編寫:[Vincent 4J](http://github.com/vincent4j) - 原文:<http://developer.android.com/training/basics/actionbar/setting-up.html>

Action bar 最基本的形式，就是為 Activity 顯示標題，並且在標題左邊顯示一個 app icon。即使在這樣簡單的形式下，action bar對於所有的 activity 來說是十分有用的。它告知用戶他們當前所處的位置，併為你的 app 維護了持續的同一標識。

![actionbar-basic](actionbar-basic.png)

圖 1. 一個有 app icon 和 Activity 標題的 action bar

設置一個基本的 action bar，需要 app 使用一個 activity 主題，該主題必須是 action bar 可用的。如何聲明這樣的主題取決於我們 app 支持的 Android 最低版本。本課程根據我們 app 支持的 Android 最低版本分為兩部分。

## 僅支持 Android 3.0 及以上版本

從 Android 3.0(API lever 11) 開始，所有使用 [Theme.Holo](http://developer.android.com/reference/android/R.style.html#Theme_Holo) 主題（或者它的子類）的 Activity 都包含了 action bar，當 [targetSdkVersion](http://developer.android.com/guide/topics/manifest/uses-sdk-element.html#target) 或 [minSdkVersion](http://developer.android.com/guide/topics/manifest/uses-sdk-element.html#min) 屬性被設置成 “11” 或更大時，它是默認主題。

所以，要為 activity 添加 action bar，只需簡單地設置屬性為 `11` 或者更大。例如：

```xml
<manifest ... >
    <uses-sdk android:minSdkVersion="11" ... />
    ...
</manifest>
```

> **注意**: 如果創建了一個自定義主題，需確保這個主題使用一個 Theme.Holo的主題作為父類。詳情見 [Action bar 的風格化](styling.html)

到此，我們的 app 使用了 `Theme.Holo` 主題，並且所有的 activity 都顯示 action bar。

## 支持 Android 2.1 及以上版本

當 app 運行在 Andriod 3.0 以下版本（不低於 Android 2.1）時，如果要添加 action bar，需要加載 Android Support 庫。

開始之前，通過閱讀[Support Library Setup](http://developer.android.com/tools/support-library/setup.html)文檔來建立**v7 appcompat** library（下載完library包之後，按照[Adding libraries with resources](http://developer.android.com/tools/support-library/setup.html#libs-with-res)的指引進行操作）。

在 Support Library集成到你的 app 工程中之後：

1、更新 activity，以便於它繼承於 [ActionBarActivity](http://developer.android.com/reference/android/support/v7/app/ActionBarActivity.html)。例如：

```java
public class MainActivity extends ActionBarActivity { ... }
```

2、在 mainfest 文件中，更新 [`<application>`](http://developer.android.com/guide/topics/manifest/application-element.html) 標籤或者單一的 [`<activity>`](http://developer.android.com/guide/topics/manifest/application-element.html) 標籤來使用一個 [Theme.AppCompat](http://developer.android.com/reference/android/support/v7/appcompat/R.style.html#Theme_AppCompat) 主題。例如：

```xml
<activity android:theme="@style/Theme.AppCompat.Light" ... >
```

> **注意**: 如果創建一個自定義主題，需確保其使用一個 `Theme.AppCompat` 主題作為父類。詳情見 [Action bar 風格化](styling.html)

現在，當 app 運行在 Android 2.1(API level 7) 或者以上時，activity 將包含 action bar。

切記，在 manifest 中正確地設置 app 支持的 API level：

```xml
<manifest ... >
    <uses-sdk android:minSdkVersion="7"  android:targetSdkVersion="18" />
    ...
</manifest>
```

