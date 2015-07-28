# 為App內容開啟深度鏈接

> 編寫:[Lin-H](https://github.com/Lin-H) - 原文:<http://developer.android.com/training/app-indexing/deep-linking.html>

為使Google能夠抓取你的app內容，並允許用戶從搜索結果進入你的app，你必須給你的app manifest中相關的activity添加intent filter。這些intent filter能使深度鏈接與你的任何activity相連。例如，用戶可以在購物app中，點擊一條深度鏈接來瀏覽一個介紹了自己所搜索的產品的頁面。

##為你的深度鏈接添加Intent filter

要創建一條與你的app內容相連的深度鏈接，添加一個包含了以下這些元素和屬性值的intent filter到你的manifest中:

[`<action>`](http://developer.android.com/guide/topics/manifest/action-element.html)

指定[ACTION_VIEW](http://developer.android.com/reference/android/content/Intent.html#ACTION_VIEW)的操作，使得Google搜索可以觸及intent filter。

[`<data>`](http://developer.android.com/guide/topics/manifest/data-element.html)

添加一個或多個[`<data>`](http://developer.android.com/guide/topics/manifest/data-element.html)標籤，每一個標籤代表一種activity對URI格式的解析，[`<data>`](http://developer.android.com/guide/topics/manifest/data-element.html)必須至少包含[android:scheme](http://developer.android.com/guide/topics/manifest/data-element.html#scheme)屬性。

你可以添加額外的屬性來改善activity所接受的URI類型。例如，你或許有幾個activity可以接受相似的URI，它們僅僅是路徑名不同。在這種情況下，使用[android:path](http://developer.android.com/guide/topics/manifest/data-element.html#path)屬性或它的變形(`pathPattern`或`pathPrefix`)，使系統能辨別對不同的URI路徑應該啟動哪個activity。

[`<category>`](http://developer.android.com/guide/topics/manifest/category-element.html)

包括[BROWSABLE](http://developer.android.com/reference/android/content/Intent.html#CATEGORY_BROWSABLE) category。[BROWSABLE](http://developer.android.com/reference/android/content/Intent.html#CATEGORY_BROWSABLE) category對於使intent filter能被瀏覽器訪問是必要的。沒有這個category，在瀏覽器中點擊鏈接無法解析到你的app。[DEFAULT](http://developer.android.com/reference/android/content/Intent.html#CATEGORY_DEFAULT) category是可選的，但建議添加。沒有這個category，activity只能夠使用app組件名稱以顯示(explicit)intent啟動。

下面的一段XML代碼向你展示，你應該如何在manifest中為深度鏈接指定一個intent filter。URI “example://gizmos” 和 “http://www.example.com/gizmos” 都能夠解析到這個activity。

```xml
<activity
    android:name="com.example.android.GizmosActivity"
    android:label="@string/title_gizmos" >
    <intent-filter android:label="@string/filter_title_viewgizmos">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <!-- 接受以"example://gizmos”開頭的 URIs  -->
        <data android:scheme="example"
              android:host="gizmos" />
        <!-- 接受以"http://www.example.com/gizmos”開頭的 URIs  -->
        <data android:scheme="http"
              android:host="www.example.com"
              android:pathPrefix="gizmos" />
    </intent-filter>
</activity>
```

當你把包含有指定activity內容的URI的intent filter添加到你的app manifest後，Android就可以在你的app運行時，為app與匹配URI的[Intent](http://developer.android.com/reference/android/content/Intent.html)建立路徑。

> **Note:** 對一個URI pattern，intent filter可以只包含一個單一的`data`元素，創建不同的intent filter來匹配額外的URI pattern。

學習更多關於定義intent filter，見[Allow Other Apps to Start Your Activity](http://developer.android.com/training/basics/intents/filters.html)

##從傳入的intent讀取數據

一旦系統通過一個intent filter啟動你的activity，你可以使用由[Intent](http://developer.android.com/reference/android/content/Intent.html)提供的數據來決定需要處理什麼。調用[getData()](http://developer.android.com/reference/android/content/Intent.html#getData())和[getAction()](http://developer.android.com/reference/android/content/Intent.html#getAction())方法來取出傳入[Intent](http://developer.android.com/reference/android/content/Intent.html)中的數據與操作。你可以在activity生命週期的任何時候調用這些方法，但一般情況下你應該在前期回調如[onCreate()](http://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle))或[onStart()](http://developer.android.com/reference/android/app/Activity.html#onStart())中調用。

這個是一段代碼，展示如何從[Intent](http://developer.android.com/reference/android/content/Intent.html)中取出數據:

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.main);

    Intent intent = getIntent();
    String action = intent.getAction();
    Uri data = intent.getData();
}
```

遵守下面這些慣例來提高用戶體驗:

* 深度鏈接應直接為用戶打開內容，不需要任何提示，插播式廣告頁和登錄頁面。要確保用戶能看到app的內容，即使之前從沒打開過這個應用。當用戶從啟動器打開app時，可以在操作結束後給出提示。這個準則也同樣適用於網站的[first click free](https://support.google.com/webmasters/answer/74536?hl=en)體驗。

* 遵循[Navigation with Back and Up](http://developer.android.com/design/patterns/navigation.html)中的設計指導，來使你的app能夠滿足用戶通過深度鏈接進入app後，向後導航的需求。

##測試你的深度鏈接

你可以使用[Android Debug Bridge](http://developer.android.com/tools/help/adb.html)和activity管理(am)工具來測試你指定的intent filter URI，能否正確解析到正確的app activity。你可以在設備或者模擬器上運行adb命令。

測試intent filter URI的一般adb語法是:

```
$ adb shell am start
        -W -a android.intent.action.VIEW
        -d <URI> <PACKAGE>
```

例如，下面的命令試圖瀏覽與指定URI相關的目標app activity。

```
$ adb shell am start
        -W -a android.intent.action.VIEW
        -d "example://gizmos" com.example.android
```
