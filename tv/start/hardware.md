<!-- # Handling TV Hardware -->
# 處理TV硬件

> 編寫:[awong1900](https://github.com/awong1900) - 原文:<http://developer.android.com/training/tv/start/hardware.html>

<!-- TV hardware is substantially different from other Android devices. TVs do not include some of the hardware features found on other Android devices, such as touch screens, cameras, and GPS receivers. TVs are also completely dependent on secondary hardware devices. In order for users to interact with TV apps, they must use a remote control or game pad. When you build an app for TV, you must carefully consider the hardware limitations and requirements of operating on TV hardware. -->

TV硬件和其他Android設備有實質性的不同。TV不包含一些其他Android設備具備的硬件特性，如觸摸屏，攝像頭，和GPS。TV操作也完全依賴於其他輔助硬件設備。為了讓用戶與TV應用交互，他們必須使用遙控器或者遊戲手柄。當我們創建TV應用時，必須小心的考慮到TV硬件的限制和操作要求。


<!-- This lesson discusses how to check if your app is running on a TV, how to handle unsupported hardware features, and discusses the requirements for handling controllers for TV devices. -->

本節課程討論如何檢查應用是不是運行在TV上，怎樣去處理不支持的硬件特性，和討論處理TV設備控制器的要求。



<!-- ## Check for a TV Device ## -->
## TV設備的檢測

<!-- If you are building an app that operates both on TV devices and other devices, you may need to check what kind of device your app is running on and adjust the operation of your app. For instance, if you have an app that can be started through an Intent, your application should check the device properties to determine if it should start a TV-oriented activity or a phone activity. -->

如果我們創建的應用同時支持TV設備和其他設備，我們可能需要檢測應用當前運行在哪種設備上，並調整應用的執行。例如，如果有一個應用通過[Intent](http://developer.android.com/reference/android/content/Intent.html)啟動，應用應該檢查設備特性然後決定是應該啟動TV方面的activity還是手機的activity。


<!-- The recommended way to determine if your app is running on a TV device is to use the UiModeManager.getCurrentModeType() method to check if the device is running in television mode. The following example code shows you how to check if your app is running on a TV device: -->

檢查應用是否運行在TV設備上，推薦的方式是用[UiModeManager.getCurrentModeType()](http://developer.android.com/reference/android/app/UiModeManager.html#getCurrentModeType())方法檢測設備是否運行在TV模式。下面的示例代碼展示瞭如何檢查應用是否運行在TV設備上：

```java
public static final String TAG = "DeviceTypeRuntimeCheck";

UiModeManager uiModeManager = (UiModeManager) getSystemService(UI_MODE_SERVICE);
if (uiModeManager.getCurrentModeType() == Configuration.UI_MODE_TYPE_TELEVISION) {
    Log.d(TAG, "Running on a TV Device")
} else {
    Log.d(TAG, "Running on a non-TV Device")
}
```


<!-- ## Handle Unsupported Hardware Features ## -->
## 處理不支持的硬件特性

<!-- Depending on the design and functionality of your app, you may be able to work around certain hardware features being unavailable. This section discusses what hardware features are typically not available for TV, how to detect missing hardware features, and suggests alternatives to using these features. -->

基於應用的設計和功能，我們可能需要在某些硬件特性不可用的情況下工作。這節討論哪些硬件特性對於TV是典型不可用的，如何去檢測缺少的硬件特性，並且去用這些特性的推薦替代方法。


<!-- ### Unsupported TV hardware features ### -->
### 不支持的TV硬件特性

<!-- TVs have a different purpose from other devices, and so they do not have hardware features that other Android-powered devices often have. For this reason, the Android system does not support the following features for a TV device: -->

TV和其他設備有不同的目的，因此它們沒有一些其他Android設備通常有的硬件特性。由於這個原因，TV設備的Android系統不支持以下特性：

硬件      				|	 Android特性描述
:-----------------------|:-------------------------------------
觸屏						|	`android.hardware.touchscreen`
觸屏模擬器				|	`android.hardware.faketouch`
電話						|	`android.hardware.telephony`
攝像頭					|	`android.hardware.camera`
藍牙						|	`android.hardware.bluetooth`
近場通訊（NFC）			|	`android.hardware.nfc`
GPS						|	`android.hardware.location.gps`
麥克風 **[1]**			|	`android.hardware.microphone`
傳感器					|	`android.hardware.sensor`

<!-- >**[1]** Some TV controllers have a microphone, which is not the same as the microphone hardware feature described here. The controller microphone is fully supported. -->

>**[1]** 一些TV控制器有麥克風，但不是這裡描述的麥克風硬件特性。控制器麥克風是完全被支持的。

<!-- See the Features Reference for a complete list of features, subfeatures, and their descriptors. -->

查看[Features Reference](http://developer.android.com/guide/topics/manifest/uses-feature-element.html#features-reference)獲得完全的特性和子特性列表，和它們的描述。


<!-- ### Declaring hardware requirements for TV ### -->
### 聲明TV硬件需求

<!-- Android apps can declare hardware feature requirements in the app manifest to ensure that they do not get installed on devices that do not provide those features. If you are extending an existing app for use on TV, closely review your app's manifest for any hardware requirement declarations that might prevent it from being installed on a TV device. -->

Android應用能通過在manifest中定義硬件特性需求來確保應用不能被安裝在不提供這些特性的設備上。如果我們正在擴展應用到TV上，仔細地審查我們的manifest的硬件特性需求，它有可能阻止應用安裝到TV設備上。


<!-- If your app uses hardware features (such as a touchscreen or camera) that are not available on TV, but can operate without the use of those features, modify your app's manifest to indicate that these features are not required by your app. The following manifest code snippet demonstrates how to declare that your app does not require hardware features which are unavailable on TV devices, even though your app may use these features on non-TV devices: -->

即使我們的應用使用了TV上不存在的硬件特性（如觸屏或者攝像頭），應用也可以在沒有那些特性的情況下工作，需要修改應用的manifest來表明這些特性不是必須的。接下來的manifest代碼片段示範瞭如何聲明在TV設備中不可用的硬件特性，儘管我們的應用在非TV設備上可能會用上這些特性。

```xml
<uses-feature android:name="android.hardware.touchscreen"
        android:required="false"></uses>
<uses-feature android:name="android.hardware.faketouch"
        android:required="false"></uses>
<uses-feature android:name="android.hardware.telephony"
        android:required="false"></uses>
<uses-feature android:name="android.hardware.camera"
        android:required="false"></uses>
<uses-feature android:name="android.hardware.bluetooth"
        android:required="false"></uses>
<uses-feature android:name="android.hardware.nfc"
        android:required="false"></uses>
<uses-feature android:name="android.hardware.gps"
        android:required="false"></uses>
<uses-feature android:name="android.hardware.microphone"
        android:required="false"></uses>
<uses-feature android:name="android.hardware.sensor"
        android:required="false"></uses>
```

<!-- >Note: Some features have subfeatures like `android.hardware.camera.front`, as described in the Feature Reference. Be sure to mark as `required="false"` any subfeatures also used in your app. -->

>**Note**：一些特性有子特性，如`android.hardware.camera.front`，參考：[Feature Reference](http://developer.android.com/guide/topics/manifest/uses-feature-element.html#features-reference)。確保應用中任何子特性也標記為`required="false"`。


<!-- All apps intended for use on TV devices must declare that the touch screen feature is not required as described in Get Started with TV Apps. If your app normally uses one or more of the features listed above, change the android:required attribute setting to false for those features in your manifest. -->

所有想用在TV設備上的應用必須聲明觸屏特性不被需要，在[創建TV應用的第一步](http://developer.android.com/training/tv/start/start.html#no-touchscreen)有描述。如果我們的應用使用了一個或更多的上面列表上的特性，改變manifest特性的`android:required`屬性為`false`。


<!-- >**Caution**: Declaring a hardware feature as required by setting its value to `true` prevents your app from being installed on TV devices or appearing in the Android TV home screen launcher. -->

>**Caution**：表明一個硬件特性是必須的，設置它的值為`true`可以阻止應用在TV設備上安裝或者出現在AndroidTV的主屏幕啟動列表上。


<!-- Once you decide to make hardware features optional for your app, you must check for the availability of those features at runtime and then adjust your app's behavior. The next section discusses how to check for hardware features and suggests some approaches for changing the behavior of your app. -->

一旦我們決定了應用的硬件特性選項，那就必須檢查在運行時這些特性的可用性，然後調整應用的行為。下一節討論如何檢查硬件特性和改變應用行為的建議處理。


<!-- For more information on filtering and declaring features in the manifest, see the uses-feature guide. -->

更多關於filter和在manifest裡聲明特性，參考：[uses-feature](http://developer.android.com/guide/topics/manifest/uses-feature-element.html)。



<!-- ### Declaring permissions that imply hardware features ### -->
### 聲明權限會隱含硬件特性 ###

<!-- Some uses-permission manifest declarations imply hardware features. This behavior means that requesting some permissions in your app manifest can exclude your app from being installed and used on TV devices. The following commonly requested permissions create an implicit hardware feature requirement: -->

一些[uses-permission](http://developer.android.com/guide/topics/manifest/uses-permission-element.html) manifest聲明隱含了硬件特性。這些行為意味著在應用中請求一些權限能導致應用不能安裝和使用在TV設備上。下面普通的權限請求包含了一個隱式的硬件特性需求：

權限                       |	隱式的硬件需求
:-------------------------|:--------------------------------
[RECORD_AUDIO]()          |	`android.hardware.microphone`
[CAMERA]()                |	`android.hardware.camera` *and* `android.hardware.camera.autofocus`
[ACCESS_COARSE_LOCATION]()|	`android.hardware.location` *and* `android.hardware.location.network`
[ACCESS_FINE_LOCATION]()  |	`android.hardware.location` *and* `android.hardware.location.gps`

<!-- For a complete list of permission requests that imply a hardware feature requirement, see the uses-feature guide. If your app requests one of the features listed above, include a uses-feature declaration in your manifest for the implied hardware feature that indicates it is not required (android:required="false"). -->

包含隱式硬件特性需求的完整權限需求列表，參考：[uses-feature](http://developer.android.com/guide/topics/manifest/uses-feature-element.html#permissions-features)。如果我們的應用請求了上面列表上的特性的任何一個，在manifest中設置它的隱式硬件特性為不需要（`android:required="false"`）。


<!-- ### Checking for hardware features ### -->
### 檢查硬件特性

<!-- The Android framework can tell you if hardware features are not available on the device where your app is running. Use the hasSystemFeature(String) method to check for specific features at runtime. This method takes a single string argument that specifies the feature you want to check. -->

在應用運行時，Android framework能告訴硬件特性是否可用。用[hasSystemFeature(String)](http://developer.android.com/reference/android/content/pm/PackageManager.html#hasSystemFeature(java.lang.String))方法在運行時檢查特定的特性。這個方法只需要一個字符串參數，即想檢查的特性名字。


<!-- The following code example demonstrates how to detect the availability of hardware features at runtime: -->

接下來的示例代碼展示瞭如何在運行時檢測硬件特性的可用性：

```java
// Check if the telephony hardware feature is available.
if (getPackageManager().hasSystemFeature("android.hardware.telephony")) {
    Log.d("HardwareFeatureTest", "Device can make phone calls");
}

// Check if android.hardware.touchscreen feature is available.
if (getPackageManager().hasSystemFeature("android.hardware.touchscreen")) {
    Log.d("HardwareFeatureTest", "Device has a touch screen.");
}
```

<!-- ### Touch screen ### -->
### 觸屏

<!-- Since most TVs do not have touch screens, Android does not support touch screen interaction for TV devices. Furthermore, using a touch screen is not consistent with a viewing environment where the user is seated 10 feet away from the display. Make sure that your UI elements and text do not require or imply the use of a touchscreen. -->

因為大部分的TV沒有觸摸屏，在TV設備上，Android不支持觸屏交互。此外，用觸屏交互和坐在離顯示器3米外觀看是相互矛盾的。

<!-- On TV devices, you should design your app to work with this interaction model by supporting navigation using a directional pad (D-pad) on a TV remote control. For more information on properly supporting navigation using TV-friendly controls, see Creating TV Navigation. -->

在TV設備中，我們應該設計出支持遙控器方向鍵（D-pad）遠程操作的交互模式。更多關於正確地支持TV友好的控制器操作的信息，參考[Creating TV Navigation](http://developer.android.com/training/tv/start/navigation.html)。


<!-- ### Camera ### -->
### 攝像頭

<!-- Although a TV typically does not have a camera, you can still provide a photography-related app on a TV. For example, if you have an app that takes, views, and edits photos, you can disable its picture-taking functionality for TVs and still allow users to view and even edit photos. If you decide to enable your camera-related app to work on a TV, add the following feature declaration your app manifest: -->

儘管TV通常沒有攝像頭，但是我們仍然可以提供拍照相關的TV應用，如果應用有拍照，查看和編輯圖片功能，在TV上可以關閉拍照功能但仍可以允許用戶查看甚至編輯圖片。如果我們決定在TV上使用攝像相關的應用，在manifest裡添加接下來的特性聲明：

```xml
<uses-feature android:name="android.hardware.camera" android:required="false" ></uses>
```

<!-- If you enable your app to run without a camera, add code to your app that detects if the camera feature is available and makes adjustments to the operation of your app. The following code example demonstrates how to detect the presence of a camera: -->

如果在缺少攝像頭情況下運行應用，在我們應用中添加代碼去檢測是否攝像頭特性可用，並且調整應用的操作。接下來的示例代碼展示瞭如何檢測一個攝像頭的存在：

```java
// Check if the camera hardware feature is available.
if (getPackageManager().hasSystemFeature("android.hardware.camera")) {
    Log.d("Camera test", "Camera available!");
} else {
    Log.d("Camera test", "No camera available. View and edit features only.");
}
```

<!-- ### GPS ### -->
### GPS

<!-- TVs are stationary, indoor devices, and do not have built-in global positioning system (GPS) receivers. If your app uses location information, you can still allow users to search for a location, or use a static location provider such as a zip code configured during the TV device setup. -->

TV是固定的室內設備，並且沒有內置的全球定位系統（GPS）接收器。如果我們應用使用定位信息，我們仍可以允許用戶搜索位置，或者用固定位置提供商代替，如在TV設置中設置郵政編碼。

```java
// Request a static location from the location manager
LocationManager locationManager = (LocationManager) this.getSystemService(
        Context.LOCATION_SERVICE);
Location location = locationManager.getLastKnownLocation("static");

// Attempt to get postal or zip code from the static location object
Geocoder geocoder = new Geocoder(this);
Address address = null;
try {
  address = geocoder.getFromLocation(location.getLatitude(),
          location.getLongitude(), 1).get(0);
  Log.d("Zip code", address.getPostalCode());

} catch (IOException e) {
  Log.e(TAG, "Geocoder error", e);
}
```

<!-- ## Handling Controllers ## -->
## 處理控制器

<!-- TV devices require a secondary hardware device for interacting with apps, in the form of a basic remote controller or game controller. This means that your app must support D-pad input. It also means that your app may need to handle controllers going offline and input from more than one type of controller. -->

TV設備需要輔助硬件設備與應用交互，如一個基本形式的遙控器或者遊戲手柄。這意味著我們應用必須支持D-pad（十字方向鍵）輸入。它也意味著我們應用可能需要處理手柄掉線和更多類型的手柄輸入。


<!-- ### D-pad minimum controls ### -->
### D-pad最低控制要求

<!-- The default controller for a TV device is a D-pad. In general, your app should be operable from a remote controller that only has up, down, left, right, select, Back, and Home buttons. If your app is a game that typically requires a game controller with additional controls, your app should attempt to allow gameplay with these D-pad controls. In this case, your app should also warn the user that a controller is required and allow them to exit your game gracefully using the D-pad controller. For more information about handling navigation with D-pad controller for TV devices, see Creating TV Navigation. -->

默認的TV設備控制器是D-pad。通常，我們可以用遙控器的上，下，左，右，選擇，返回，和Home鍵操作應用。如果應用是一個遊戲而需要遊戲手柄額外的控制，它也應該嘗試允許用D-pad操作。這種情況下，應用也應該警告用戶需要手柄，並且允許他們用D-pad優雅的退出遊戲。更多關於在TV設備如理D-pad的操作，參考[Creating TV Navigation](http://developer.android.com/training/tv/start/navigation.html)。


<!-- ### Handle controller disconnects ### -->
### 處理手柄掉線

<!-- Controllers for TV are frequently Bluetooth devices which may attempt to save power by periodically going into sleep mode and disconnecting from the TV device. This means that an app might be interrupted or restarted if it is not configured to handle these reconnect events. These events can happen in any of the following circumstances: -->

TV的手柄通常是藍牙設備，它為了省電而定期的休眠並且與TV設備斷開連接。這意味著如果不處理這些重連事件，應用可能被中斷或者重新開始。這些事件可以發生在下面任何情景中：

<!--
- While watching a video which is several minutes long, a D-Pad or game controller goes into sleep mode, disconnects from the TV device and then reconnects later on.
- During gameplay, a new player joins the game using a game controller that is not currently connected.
- During gameplay, a player leaves the game and disconnects a game controller.
-->

- 當在看幾分鐘的視頻，D-Pad或者遊戲手柄進入了睡眠模式，從TV設備上斷開連接並且隨後重新連接。
- 在玩遊戲時，新玩家用不是當前連接的遊戲手柄加入遊戲。
- 在玩遊戲時，一個玩家離開遊戲並且斷開遊戲手柄。

<!-- Any TV app activity that is subject to disconnect and reconnect events must be configured to handle reconnection events in the app manifest. The following code sample demonstrates how to enable an activity to handle configuration changes, including a keyboard or navigation device connecting, disconnecting, or reconnecting: -->

任何TV應用activity相關於斷開和重連事件。這些事件必須在應用的manifest配置去處理。接下來的示例代碼展示瞭如何確保一個activity去處理配置改變，包括鍵盤或者操作設備連接，斷開連接，或者重新連接：

```java
<activity
  android:name="com.example.android.TvActivity"
  android:label="@string/app_name"
  android:configChanges="keyboard|keyboardHidden|navigation"
  android:theme="@style/Theme.Leanback">

  <intent-filter>
    <action android:name="android.intent.action.MAIN" ></action>
    <category android:name="android.intent.category.LEANBACK_LAUNCHER" ></category>
  </intent-filter>
  ...
</activity>
```

<!-- This configuration change allows the app to continue running through a reconnection event, rather than being restarted by the Android framework, which is not a good user experience. -->

這個配置改變屬性允許應用通過重連事件繼續運行，比較而言Android framework強制重啟應用會導致一個不好的用戶體驗。

<!-- ### Handle D-pad input variations ### -->
### 處理D-pad變種輸入

<!-- TV device users may have more than one type of controller that they use with their TV. For example, a user might have both a basic D-pad controller and a game controller. The key codes provided by a game controller when it is being used for D-pad functions may vary from the key codes sent by a physical D-pad. -->

TV設備用戶可能有超過一種類型的控制器來操作TV。例如，一個用戶可能有基本D-pad控制器和一個遊戲控制器。遊戲控制器用於D-pad功能的按鍵代碼可能和物理十字鍵提供的不相同。

<!-- Your app should handle the variations of D-pad input from a game controller, so the user does not have to physically switch controllers to operate your app. For more information on handling these input variations, see Handling Controller Actions. -->

我們的應用應該處理遊戲控制器D-pad的變種輸入，這樣用戶不需要通過手動切換控制器去操作應用。更多信息關於處理這些變種輸入，參考[Handling Controller Actions](http://developer.android.com/training/tv/start/hardware.html)。

-------------
[下一節: 創建TV佈局 >](layouts.html)
