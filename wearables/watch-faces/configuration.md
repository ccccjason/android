# 提供配置 Activity

> 編寫:[heray1990](https://github.com/heray1990) - 原文: <http://developer.android.com/training/wearables/watch-faces/configuration.html>

當用戶安裝一個包含錶盤的可穿戴應用的手持式應用時，它們可以在手持式設備上的 Android Wear 配套應用和在可穿戴設備上的錶盤選擇器中使用。用戶可以在配套應用上或者在可穿戴設備的錶盤選擇器上選擇使用哪個錶盤。

一些錶盤提供配置參數，讓用戶客製化錶盤的外觀和行為。例如，一些錶盤讓用戶選擇自定義的背景顏色，另一些錶盤提供兩個不同時區的時間，使得用戶可以選擇感興趣的時區。

提供配置參數的錶盤讓用戶通過可穿戴應用的一個 activity、手持應用的一個 activity或者兩者的 activity 來客製化錶盤。用戶可以啟動可穿戴設備上的可穿戴配置 activity，他們也可以啟動 Android Wear 配套應用的配套配置 activity。

Android SDK 中 *WatchFace* 示例的數字表盤介紹瞭如何實現手持式和可穿戴配置 activity 和如何應配置變化而更新錶盤。這個示例位於 `android-sdk/samples/android-21/wearable/WatchFace` 目錄。

<a name="Intent"></a>
## 指定配置 activity 的 Intent

如果錶盤包括配置的 activity，那麼添加下面的元數據項到可穿戴應用 manifest 文件的服務聲明部分：

```xml
<service
    android:name=".DigitalWatchFaceService" ... />
    <!-- companion configuration activity -->
    <meta-data
        android:name=
           "com.google.android.wearable.watchface.companionConfigurationAction"
        android:value=
           "com.example.android.wearable.watchface.CONFIG_DIGITAL" />
    <!-- wearable configuration activity -->
    <meta-data
        android:name=
           "com.google.android.wearable.watchface.wearableConfigurationAction"
        android:value=
           "com.example.android.wearable.watchface.CONFIG_DIGITAL" />
    ...
</service>
```

在應用的包名之前定義這些元數據項的值。配置 activity 為這個 intent 註冊 intent filters，然後系統在用戶想配置錶盤時啟動這個 intent。

如果錶盤只包括一個配套或者可穿戴配置 activity，那麼我們只需要包括上述例子響應的元數據項。

## 創建可穿戴配置 activity

可穿戴配置 activity 提供了有限組錶盤客製化選擇，這是因為複雜的菜單在小屏幕上很難導航。我們的可穿戴配置 activity 應該提供二元選擇和很少的選項來客製化錶盤主要的方面。

為了創建一個可穿戴配置 activity，添加一個新的 activity 到可穿戴應用並且在可穿戴應用的 manifest 文件中聲明下面的 intent filter：

```xml
<activity
    android:name=".DigitalWatchFaceWearableConfigActivity"
    android:label="@string/digital_config_name">
    <intent-filter>
        <action android:name=
            "com.example.android.wearable.watchface.CONFIG_DIGITAL" />
        <category android:name=
        "com.google.android.wearable.watchface.category.WEARABLE_CONFIGURATION" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</activity>
```

這個 intent filter 的 action 的名字必須與之前在[指定配置 activity 的 Intent](#Intent)定義的 intent 名字一樣。

在我們的配置 activity 中，構建一個簡單的 UI 為用戶提供選擇來客製化錶盤。當用戶做出選擇時，使用[可穿戴數據層 API](http://hukai.me/android-training-course-in-chinese/wearables/data-layer/index.html)傳達配置的變化給錶盤 activity。

更多詳細內容，請見 *WatchFace* 示例中的 `DigitalWatchFaceWearableConfigActivity` 和 `DigitalWatchFaceUtil` 類。

## 創建配套配置 activity
配套配置 activity 讓用戶可以訪問全套錶盤客製化選擇，這是因為在手持式設備更大的屏幕上，用戶更加容易與複雜的菜單互動。例如，手持設備上的一個配置 activity 向用戶顯示覆雜的顏色選擇器，讓用戶從該選擇器中選擇錶盤的背景顏色。

為了創建配套配置 activity，添加一個新的 activity 到手持應用並且在手持應用的 manifest 文件中聲明下面的 intent filter：

```xml
<activity
    android:name=".DigitalWatchFaceCompanionConfigActivity"
    android:label="@string/app_name">
    <intent-filter>
        <action android:name=
            "com.example.android.wearable.watchface.CONFIG_DIGITAL" />
        <category android:name=
        "com.google.android.wearable.watchface.category.COMPANION_CONFIGURATION" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</activity>
```

在我們的配置 activity 中，構建一個 UI 為用戶提供選項來客製化錶盤所有的可配置組件。當用戶做出選擇時，使用[可穿戴數據層 API](http://hukai.me/android-training-course-in-chinese/wearables/data-layer/index.html)傳達配置的變化給錶盤 activity。

更多詳細內容，請見 *WatchFace* 示例中的 `DigitalWatchFaceCompanionConfigActivity` 類。

## 在可穿戴應用中創建一個監聽器服務

為了接收配置 activity 中已更新的配置參數，需要在可穿戴應用創建一個服務來實現[可穿戴數據層 API](http://hukai.me/android-training-course-in-chinese/wearables/data-layer/index.html) 的 `WearableListenerService` 接口。我們的錶盤實現可以在配置參數改變時重新繪製錶盤。

更多詳細內容，請見 *WatchFace* 示例的 `DigitalWatchFaceConfigListenerService` 和 `DigitalWatchFaceService` 類。