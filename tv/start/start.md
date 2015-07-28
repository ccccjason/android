<!-- Get Started with TV Apps -->
# 創建TV應用的第一步

> 編寫:[awong1900](https://github.com/awong1900) - 原文:<http://developer.android.com/training/tv/start/start.html>

<!-- TV apps use the same structure as those for phones and tablets. This similarity means you can modify your existing apps to also run on TV devices or create new apps based on what you already know about building apps for Android. -->

TV應用使用與手機和平板同樣的架構。這種相似性意味著我們可以修改現有的應用到TV設備或者用以前安卓應用的經驗開發TV應用。

<!-- Important: There are specific requirements your app must meet to qualify as an Android TV app on Google Play. For more information, see the requirements listed in TV App Quality. -->

>**Important**: 想把Android TV應用放在Google Play中應滿足一些特定要求。更多信息, 參考[TV App Quality](http://developer.android.com/distribute/essentials/quality/tv.html)中的要求列表。

<!-- This lesson describes how to prepare your development environment for building TV apps, and the minimum required changes to enable an app to run on TV devices. -->

本課程介紹如何準備TV應用開發環境,和使應用能夠運行在TV設備上的最低要求。


<!-- ## Determine Media Format Support -->
## 查明支持的媒體格式
<!--
See the following documentation for information about the codecs, protocols, and formats supported by Android TV.

- Supported Media Formats
- DRM
- android.drm
- ExoPlayer
- android.media.MediaPlayer
-->

查看以下文檔信息，包括代碼，協議和Android TV支持的格式。

- [支持的媒體格式](http://developer.android.com/guide/appendix/media-formats.html)
- [DRM](https://source.android.com/devices/drm.html)
- [android.drm](http://developer.android.com/reference/android/drm/package-summary.html)
- [ExoPlayer](http://developer.android.com/guide/topics/media/exoplayer.html)
- [android.media.MediaPlay](http://developer.android.com/reference/android/media/MediaPlayer.html)


<!-- ## Determine Media Format Support -->
## 查明支持的媒體格式
<!--
See the following documentation for information about the codecs, protocols, and formats supported by Android TV.

- Supported Media Formats
- DRM
- android.drm
- ExoPlayer
- android.media.MediaPlayer
-->

查看一下文檔關於代碼，協議和Android TV支持的格式。

- [支持的媒體格式](http://developer.android.com/guide/appendix/media-formats.html)
- [DRM](https://source.android.com/devices/drm.html)
- [android.drm](http://developer.android.com/reference/android/drm/package-summary.html)
- [ExoPlayer](http://developer.android.com/guide/topics/media/exoplayer.html)
- [android.media.MediaPlay](http://developer.android.com/reference/android/media/MediaPlayer.html)

<!--Set up a TV Project -->
## 創建TV項目

<!--This section discusses how to modify an existing app to run on TV devices, or create a new one. These are the main components you must use to create an app that runs on TV devices: -->

本節討論如何修改已有的應用或者新建一個應用使之能夠運行在電視設備上。在TV設備上運行的應用必須使用這些主要組件:

<!--
* Activity for TV (Required) - In your application manifest, declare an activity that is intended to run on TV devices.
* TV Support Libraries (Optional) - There are several Support Libraries available for TV devices that provide widgets for building user interfaces. -->

* **Activity for TV** (必須) - 在您的application manifest中, 聲明一個可在TV設備上運行的activity。
* **TV Support Libraries** (可選) - 這些支持庫[Support Libraries](http://developer.android.com/training/tv/start/start.html#tv-libraries) 可以提供搭建TV用戶界面的控件。

<!-- Prerequisites -->
### 前提條件

<!-- Before you begin building apps for TV, you must: -->
在創建TV應用前, 必須做以下事情:

<!--
* Update your SDK tools to version 24.0.0 or higher
	The updated SDK tools enable you to build and test apps for TV.
* Update your SDK with Android 5.0 (API 21) or higher
	The updated platform version provides new APIs for TV apps.
* Create or update your app project
	In order to access new APIs for TV devices, you must create a project or modify an existing project that targets Android 5.0 (API level 21) or higher.
-->

- [更新SDK tools到版本24.0.0或更高](http://developer.android.com/sdk/installing/adding-packages.html#GetTools)
	更新的SDK工具能確保編譯和測試TV應用

- [更新SDK為Android 5.0 (API 21)或更高](http://developer.android.com/sdk/installing/adding-packages.html#GetTools)
	更新的平臺版本為TV應用提供更新的API

- [創建或更新應用工程](http://developer.android.com/sdk/installing/create-project.html)
	為了支持TV新API, 我們必須創建一個新工程或者修改原工程的目標平臺為Android 5.0 (API版本21)或者更高。


<!-- Declare a TV Activity -->
### 聲明一個TV Activity

<!-- An application intended to run on TV devices must declare a launcher activity for TV in its manifest using a [CATEGORY_LEANBACK_LAUNCHER](http://developer.android.com/reference/android/content/Intent.html#CATEGORY_LEANBACK_LAUNCHER) intent filter. This filter identifies your app as being enabled for TV, and is required for your app to be considered a TV app in Google Play. Declaring this intent also identifies which activity in your app to launch when a user selects its icon on the TV home screen. -->

一個應用想要運行在TV設備中，必須在它的manifest中定義一個啟動activity，用intent filter包含[CATEGORY_LEANBACK_LAUNCHER](http://developer.android.com/reference/android/content/Intent.html#CATEGORY_LEANBACK_LAUNCHER)。這個filter表明你的應用是在TV上可用，並且為Google Play上發佈TV應用所必須。定義這個intent也意味著點擊主屏幕的應用圖標時，就是打開的這個activity。

<!-- The following code snippet shows how to include this intent filter in your manifest: -->
接下來的代碼片段顯示如何在manifest中包含這個intent filter：

```java
<application
  android:banner="@drawable/banner" >
  ...
  <activity
    android:name="com.example.android.MainActivity"
    android:label="@string/app_name" >

    <intent-filter>
      <action android:name="android.intent.action.MAIN" />
      <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
  </activity>

  <activity
    android:name="com.example.android.TvActivity"
    android:label="@string/app_name"
    android:theme="@style/Theme.Leanback">

    <intent-filter>
      <action android:name="android.intent.action.MAIN" />
      <category android:name="android.intent.category.LEANBACK_LAUNCHER" />
    </intent-filter>

  </activity>
</application>
```

<!-- The second activity manifest entry in this example specifies that activity as the one to launch on a TV device. -->
例子中第二個activity manifest定義的activity是TV設備中的一個啟動入口。

<!-- > **Caution**: If you do not include the [CATEGORY_LEANBACK_LAUNCHER](http://developer.android.com/reference/android/content/Intent.html#CATEGORY_LEANBACK_LAUNCHER) intent filter in your app, it is not visible to users running the Google Play store on TV devices. Also, if your app does not have this filter when you load it onto a TV device using developer tools, the app does not appear in the TV user interface. -->

> **Caution**：如果在你的應用中不包含[CATEGORY_LEANBACK_LAUNCHER](http://developer.android.com/reference/android/content/Intent.html#CATEGORY_LEANBACK_LAUNCHER) intent filter，它不會出現在TV設備的Google Play商店中。並且，即使你把不包含此filter的應用用開發工具裝載到TV設備中，應用仍然不會出現在TV用戶界面上。


<!-- If you are modifying an existing app for use on TV, your app should not use the same activity layout for TV that it does for phones and tablets. The user interface of your TV app (or TV portion of your existing app) should provide a simpler interface that can be easily navigated using a remote control from a couch. For guidelines on designing an app for TV, see the [TV Design](http://developer.android.com/design/tv/index.html) guide. For more information on the minimum implementation requirements for interface layouts on TV, see [Building TV Layouts](http://developer.android.com/training/tv/start/layouts.html). -->

如果你正在為TV設備修改現有的應用，就不應該與手機和平板用同樣的activity佈局。TV的用戶界面（或者現有應用的TV部分）應該提供一個更簡單的界面，更容易坐在沙發上用遙控器操作。TV應用設計指南，參考[TV Design](http://developer.android.com/design/tv/index.html)指導。查看TV界面佈局的最低要求，參考：[Building TV Layouts](http://developer.android.com/training/tv/start/layouts.html)。


<!-- ### Declare Leanback support -->
### 聲明Leanback支持

<!-- Declare that your app uses the Leanback user interface required by Android TV. If you are developing an app that runs on mobile (phones, wearables, tablets, etc.) as well as Android TV, set the required attribute value to `false`. If you set the `required` attribute value to `true`, your app will run only on devices that use the Leanback UI. -->

Android TV需要你的應用使用Leanback用戶界面。如果你正在開發一個運行在移動設備（手機，可穿戴，平板等等）也包括TV的應用，設置`required`屬性為`false`。因為如果設置為`true`，你的應用將僅能運行在用Leanback UI的設備上。

```java
<manifest>
    <uses-feature android:name="android.software.leanback"
        android:required="false" />
    ...
</manifest>
```

<!-- ### Declare touchscreen not required -->
### 聲明不需要觸屏

<!-- Applications that are intended to run on TV devices do not rely on touch screens for input. In order to make this clear, the manifest of your TV app must declare that a the android.hardware.touchscreen feature is not required. This setting identifies your app as being able to work on a TV device, and is required for your app to be considered a TV app in Google Play. The following code example shows how to include this manifest declaration: -->

運行在TV設備上的應用不依靠觸屏去輸入。為了清楚表明這一點，TV應用的manifest必須聲明`android.hardware.touchscreen`為不需要。這個設置表明應用能夠工作在TV設備上，並且也是Google Play認定你的應用為TV應用的要求。接下來的示例代碼展示這個manifest聲明：

```java
<manifest>
    <uses-feature android:name="android.hardware.touchscreen"
              android:required="false" />
    ...
</manifest>
```

<!-- >**Caution**: You must declare that a touch screen is not required in your app manifest, as shown this example code, or your app cannot appear in the Google Play store on TV devices. -->

>**Caution**：必須在manifest中聲明觸屏是不需要的，否則應用不會出現在TV設備的Google Play商店中。

<!-- ### Provide a home screen banner -->
### 提供一個主屏幕橫幅

<!-- An application must provide a home screen banner for each localization if it includes a Leanback launcher intent filter. The banner is the app launch point that appears on the home screen in the apps and games rows. Desribe the banner in the manifest as follows: -->

如果應用包含一個Leanback的intent filter，它必須提供每個語言的主屏幕橫幅。橫幅是出現在應用和遊戲欄的主屏的啟動點。在manifest中這樣描述橫幅：

```java
<application
    ...
    android:banner="@drawable/banner" >

    ...
</application>
```

<!-- Use the [android:banner] attribute with the [application] tag to supply a default banner for all application activities, or with the [activity] tag to supply a banner for a specific activity. -->

在[`application`](http://developer.android.com/guide/topics/manifest/application.html)中添加[`android:banner`](http://developer.android.com/guide/topics/manifest/application-element.html#banner)屬性為所有的應用activity提供默認的橫幅，或者在特定activity的[`activity`](http://developer.android.com/guide/topics/manifest/activity-element.html)中添加橫幅。

<!-- See [Banners](http://developer.android.com/design/tv/patterns.html#banner) in the UI Patterns for TV design guide. -->
在UI模式和TV設計指導中查看[Banners](http://developer.android.com/design/tv/patterns.html#banner)。


<!-- ## Add TV Support Libraries -->
## 添加TV支持庫

<!-- The Android SDK includes support libraries that are intended for use with TV apps. These libraries provide APIs and user interface widgets for use on TV devices. The libraries are located in the <sdk>/extras/android/support/ directory. Here is a list of the libraries and their general purpose: -->

Android SDK包含用於TV應用的支持庫。這些庫為TV設備提供API和用戶界面控件。這些庫位於`<sdk>/extras/android/support/`目錄。以下是這些庫的列表和它們的作用介紹：

<!--
* [v17 leanback library](http://developer.android.com/tools/support-library/features.html#v17-leanback) - Provides user interface widgets for TV apps, particularly for apps that do media playback.
* [v7 recyclerview library](http://developer.android.com/tools/support-library/features.html#v7-recyclerview) - Provides classes for managing display of long lists in a memory efficient manner. Several classes in the v17 leanback library depend on the classes in this library.
* [v7 cardview library](http://developer.android.com/tools/support-library/features.html#v7-cardview) - Provides user interface widgets for displaying information cards, such as media item pictures and descriptions.
-->

* [v17 leanback library](http://developer.android.com/tools/support-library/features.html#v17-leanback) - 提供TV應用的用戶界面控件，特別是用於媒體播放應用的控件。
* [v7 recyclerview library](http://developer.android.com/tools/support-library/features.html#v7-recyclerview) - 提供了內存高效方式的長列表的管理顯示類。有一些v17 leanback庫的類依賴於本庫的類。
* [v7 cardview library](http://developer.android.com/tools/support-library/features.html#v7-cardview) - 提供顯示信息卡的用戶界面控件，如媒體圖片和描述。


<!-- >**Note**: You are not required to use these support libraries for your TV app. However, we strongly recommend using them, particularly for apps that provide a media catalog browsing interface. -->

>**Note**：TV應用中可以不用這些庫。但是，我們強烈推薦使用它們，特別是為應用提供媒體目錄瀏覽界面時。


<!-- If you decide to use the v17 leanback library for your app, you should note that it is dependent on the [v4 support library](http://developer.android.com/tools/support-library/features.html#v4). This means that apps that use the leanback support library should include all of these support libraries: -->

如果我們決定用`v17 leanback library`，我們應該注意它依賴於[v4 support library](http://developer.android.com/tools/support-library/features.html#v4)。這意味著要用leanback支持庫必須包含以下所有的支持庫：

* v4 support library
* v7 recyclerview support library
* v17 leanback support library


<!-- The v17 leanback library contains resources, which require you to take specific steps to include it in app projects. For instructions on importing a support library with resources, see [Support Library Setup]. -->

`v17 leanback library`包含資源文件，需要你在應用中採取特定的步驟去包含它。插入帶資源文件的支持庫的說明，查看[Support Library Setup](http://developer.android.com/tools/support-library/setup.html#libs-with-res)。


<!-- ## Build TV Apps -->
## 創建TV應用

<!-- After you have completed the steps described above, it's time to start building apps for the big screen! Check out these additional topics to help you build your app for TV: -->

在完成上面的步驟之後，到了給大屏幕創建應用的時候了！檢查一下這些額外的專題可以幫助我們創建TV應用：

<!--
* [Building TV Playback Apps](http://developer.android.com/training/tv/playback/index.html) - TVs are built to entertain, so Android provides a set of user interface tools and widgets for building TV apps that play videos and music, and let users browse for the content they want.
* [Helping Users Find Your Content on TV](http://developer.android.com/training/tv/discovery/index.html) - With all the content choices at users' fingertips, helping them find content they enjoy is almost as important as providing that content. This training discusses how to surface your content on TV devices.
* [Games for TV](http://developer.android.com/training/tv/discovery/index.html) - TV devices are a great platform for games. See this topic for information on building great game experiences for TV.
-->

* [創建TV播放應用](http://developer.android.com/training/tv/playback/index.html) - TV就是用來娛樂的，因此安卓提供了一套用戶界面工具和控件，用來創建視頻和音樂的TV應用，並且讓用戶瀏覽想看到的內容。
* [幫助用戶找到TV內容](http://developer.android.com/training/tv/discovery/index.html) - 因為所有的內容選擇都用手指操作遙控器，所以幫助用戶找到想要的內容幾乎和提供內容同樣重要。這個主題討論如何在TV設備中處理內容。
* [TV遊戲](http://developer.android.com/training/tv/games/index.html) - TV設備是非常好的遊戲平臺。參考這個主題去創造更好的TV遊戲體驗。

<!-- ## Run TV Apps -->
## 運行TV應用

<!-- Running your app is an important part of the development process. The AVD Manager in the Android SDK provides the device definitions that allow you to create virtual TV devices for running and testing your applications. -->

運行應用是在開發過程中的一個重要的部分。在安卓SDK中的AVD管理器提供了創建虛擬TV設備的功能，可以讓應用在虛擬設備中運行和測試。

<!-- To create an virtual TV device: -->
創建一個虛擬TV設備

<!--
1. Start the AVD Manager. For more information, see the AVD Manager help.
2. In the AVD Manager dialog, click the Device Definitions tab.
3. Select one of the Android TV device definitions and click Create AVD.
4. Select the emulator options and click OK to create the AVD.
-->

1. 打開AVD管理器。更多信息，參考[AVD管理器](http://developer.android.com/tools/help/avd-manager.html)幫助。
2. 在AVD管理器窗口，點擊**Device Definitions**標籤。
3. 選擇一個Android TV設備描述，並且點擊**Create AVD**。
4. 選擇模擬器選項並且點擊**OK**創建AVD。

<!-- >**Note**: For best performance of the TV emulator device, enable the Use Host GPU option and, where supported, use virtual device acceleration. For more information on hardware acceleration of the emulator, see [Using the Emulator](http://developer.android.com/tools/devices/emulator.html#acceleration). -->

>**Note**：獲得TV模擬器設備的最佳性能，打開**Use Host GPU option**，支持虛擬設備加速。更多模擬器硬件加速信息，參考[Using the Emulator](http://developer.android.com/tools/devices/emulator.html#acceleration)。

<!-- To test your application on the virtual TV device: -->
在虛擬設備中測試應用

<!--
1. Compile your TV application in your development environment.
2. Run the application from your development environment and choose the TV virtual device as the target.
-->

1. 在開發環境中編譯TV應用。
2. 從開發環境中運行應用並選擇目標為TV虛擬設備。


<!-- For more information about using emulators see, [Using the Emulator](http://developer.android.com/tools/devices/emulator.html). For more information on deploying apps from Android Studio to virtual devices, see [Debugging with Android Studio](http://developer.android.com/sdk/installing/studio-debug.html). For more information about deploying apps to emulators from Eclipse with ADT, see [Building and Running from Eclipse with ADT ](http://developer.android.com/tools/building/building-eclipse.html) -->

更多模擬器信息：[Using the Emulator](http://developer.android.com/tools/devices/emulator.html)。 用Android Studio部署應用到模擬器，查看[Debugging with Android Studio](http://developer.android.com/sdk/installing/studio-debug.html)。用帶ADT插件的Eclipse部署應用到模擬器，查看[Building and Running from Eclipse with ADT ](http://developer.android.com/tools/building/building-eclipse.html)。

-------------------------
[下一節: 處理TV硬件 >](hardware.html)
