# 適配不同的系統版本

> 編寫:[Lin-H](http://github.com/Lin-H) - 原文:<http://developer.android.com/training/basics/supporting-devices/platforms.html>

新的Android版本會為我們的app提供更棒的APIs，但我們的app仍應支持舊版本的Android，直到更多的設備升級到新版本為止。這節課程將展示如何在利用新的APIs的同時仍支持舊版本Android。

[Platform Versions](http://developer.android.com/about/dashboards/index.html)的控制面板會定時更新，通過統計訪問Google Play Store的設備數量，來顯示運行每個版本的安卓設備的分佈。一般情況下，在更新app至最新Android版本時，最好先保證新版的app可以支持90%的設備使用。

> **Tip**:為了能在幾個Android版本中都能提供最好的特性和功能，應該在我們的app中使用[Android Support Library](https://developer.android.com/tools/support-library/index.html)，它能使我們的app能在舊平臺上使用最近的幾個平臺的APIs。

## 指定最小和目標API級別

[AndroidManifest.xml](https://developer.android.com/guide/topics/manifest/manifest-intro.html)文件中描述了我們的app的細節及app支持哪些Android版本。具體來說，[`<uses-sdk>`](https://developer.android.com/guide/topics/manifest/uses-sdk-element.html)元素中的`minSdkVersion`和`targetSdkVersion` 屬性，標明在設計和測試app時，最低兼容API的級別和最高適用的API級別(這個最高的級別是需要通過我們的測試的)。例如：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android" ... >
    <uses-sdk android:minSdkVersion="4" android:targetSdkVersion="15" />
    ...
</manifest>
```

隨著新版本Android的發佈，一些風格和行為可能會改變，為了能使app能利用這些變化，而且能適配不同風格的用戶的設備，我們應該將`targetSdkVersion`的值儘量的設置與最新可用的Android版本匹配。

## 運行時檢查系統版本

Android在[Build](https://developer.android.com/reference/android/os/Build.html)常量類中提供了對每一個版本的唯一代號，在我們的app中使用這些代號可以建立條件，保證依賴於高級別的API的代碼，只會在這些API在當前系統中可用時，才會執行。

```java
private void setUpActionBar() {
    // Make sure we're running on Honeycomb or higher to use ActionBar APIs
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
        ActionBar actionBar = getActionBar();
        actionBar.setDisplayHomeAsUpEnabled(true);
    }
}
```

> **Note**:當解析XML資源時，Android會忽略當前設備不支持的XML屬性。所以我們可以安全地使用較新版本的XML屬性，而不需要擔心舊版本Android遇到這些代碼時會崩潰。例如如果我們設置`targetSdkVersion="11"`，app會在Android 3.0或更高時默認包含[ActionBar](https://developer.android.com/reference/android/app/ActionBar.html)。然後添加menu items到action bar時，我們需要在自己的menu XML資源中設置`android:showAsAction="ifRoom"`。在跨版本的XML文件中這麼做是安全的，因為舊版本的Android會簡單地忽略`showAsAction`屬性(就是這樣，你並不需要用到`res/menu-v11/`中單獨版本的文件)。

## 使用平臺風格和主題

Android提供了用戶體驗主題，為app提供基礎操作系統的外觀和體驗。這些主題可以在manifest文件中被應用於app中。通過使用內置的風格和主題，我們的app自然地隨著Android新版本的發佈，自動適配最新的外觀和體驗.

使activity看起來像對話框:

```xml
<activity android:theme="@android:style/Theme.Dialog">
```

使activity有一個透明背景:

```xml
<activity android:theme="@android:style/Theme.Translucent">
```

應用在`/res/values/styles.xml`中定義的自定義主題:

```xml
<activity android:theme="@style/CustomTheme">
```

使整個app應用一個主題(全部activities)在[<application\\>](https://developer.android.com/guide/topics/manifest/application-element.html)元素中添加`android:theme`屬性:

```xml
<application android:theme="@style/CustomTheme">
```

更多關於創建和使用主題，詳見[Styles and Themes](https://developer.android.com/guide/topics/ui/themes.html)。
