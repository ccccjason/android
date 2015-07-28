# 構建錶盤服務

> 編寫:[heray1990](https://github.com/heray1990) - 原文: <http://developer.android.com/training/wearables/watch-faces/service.html>

Android Wear 的錶盤在可穿戴應用中實現為[服務（services）](http://developer.android.com/guide/components/services.html)和包。當用戶安裝一個包含錶盤的可穿戴應用的手持式應用時，這些錶盤在手持式設備的 [Android Wear 配套應用](https://play.google.com/store/apps/details?id=com.google.android.wearable.app&hl=en)和可穿戴錶盤選擇器中可用。當用戶選擇一個可用的錶盤時，可穿戴設備會顯示錶盤並且按需要調用它的服務毀掉方法。

這節課介紹如何配置包含錶盤的 Android 工程和如何實現錶盤服務。

## 創建並配置工程

在 Android Studio 中為錶盤創建一個 Android 工程，需要：

1. 打開 Android Studio。
2. 創建一個新的工程：
	* 如果沒有打開過任何工程，那麼在 **Welcome** 界面中點擊 **New Project**。
	* 如果已經打開過工程，那麼在 **File** 菜單中選擇 **New Project**。
3. 填寫應用名字，然後點擊 **Next**。
4. 選擇 **Phone and Tablet** 尺寸係數。
5. 在 **Minimum SDK** 下拉菜單選擇 API 18。
6. 選擇 **Wear** 尺寸係數。
7. 在 **Minimum SDK** 下拉菜單選擇 API 21，然後點擊 **Next**。
8. 選擇 **Add No Activity** 然後在接下來的兩個界面點擊 **Next** 。
9. 點擊 **Finish**。
10. 在 IDE 窗口點擊 **View > Tool Windows > Project**。

至此，Android Studio 創建了一個含有 `wear` 和 `mobile` 模塊的工程。更多關於創建工程的內容，請見 [Creating a Project](http://developer.android.com/sdk/installing/create-project.html)。

### 依賴

Wearable Support 庫提供了必要的類，我們可以繼承這些類來創建錶盤的實現。需要用Google Play services client 庫（`play-services` 和 `play-services-wearable`）在配套設備和含有[可穿戴數據層 API](http://hukai.me/android-training-course-in-chinese/wearables/data-layer/index.html) 的可穿戴應用之間同步數據項。

當我們按照上述的方法創建工程時，Android Studio 會自動添加需要的條目到 `build.gradle` 文件。

### Wearable Support 庫 API 參考資源

該參考文檔提供了用於實現錶盤的詳細信息。詳見 [API 參考文檔](http://developer.android.com/reference/android/support/wearable/watchface/package-summary.html)。

### 在 Eclipse ADT 中下載 Wearable Support 庫

如果你使用 Eclipse ADT，那麼請下載 [Wearable Support 庫](http://developer.android.com/shareables/training/wearable-support-lib.zip) 並且將該庫作為依賴包含在你的工程當中。

### 聲明權限

錶盤需要 `PROVIDE_BACKGROUND` 和 `WAKE_LOCK` 權限。在可穿戴和手持式應用的 manifest 文件中 `manifest` 節點下添加如下權限：

```xml
<manifest ...>
    <uses-permission
        android:name="com.google.android.permission.PROVIDE_BACKGROUND" />
    <uses-permission
        android:name="android.permission.WAKE_LOCK" />
    ...
</manifest>
```

> **Caution:** 手持式應用必須包括所有在可穿戴應用中聲明的權限。

## 實現服務和回調方法

Android Wear 的錶盤實現為[服務(services)](http://developer.android.com/guide/components/services.html)。當錶盤處於活動狀態時，系統會在時間改變或者出現重要的時間（如切換到環境模式或者接收到一個新的通知）的時候調用服務的方法。服務實現接著根據更新的時間和其它相關的數據將錶盤繪製到屏幕上。

實現一個錶盤，我們需要繼承 `CanvasWatchFaceService` 和 `CanvasWatchFaceService.Engine` 類，然後重寫 `CanvasWatchFaceService.Engine` 類的回調方法。這些類都包含在 Wearable Support 庫裡。

下面的代碼片段略述了我們需要實現的主要方法：

```java
public class AnalogWatchFaceService extends CanvasWatchFaceService {

    @Override
    public Engine onCreateEngine() {
        /* provide your watch face implementation */
        return new Engine();
    }

    /* implement service callback methods */
    private class Engine extends CanvasWatchFaceService.Engine {

        @Override
        public void onCreate(SurfaceHolder holder) {
            super.onCreate(holder);
            /* initialize your watch face */
        }

        @Override
        public void onPropertiesChanged(Bundle properties) {
            super.onPropertiesChanged(properties);
            /* get device features (burn-in, low-bit ambient) */
        }

        @Override
        public void onTimeTick() {
            super.onTimeTick();
            /* the time changed */
        }

        @Override
        public void onAmbientModeChanged(boolean inAmbientMode) {
            super.onAmbientModeChanged(inAmbientMode);
            /* the wearable switched between modes */
        }

        @Override
        public void onDraw(Canvas canvas, Rect bounds) {
            /* draw your watch face */
        }

        @Override
        public void onVisibilityChanged(boolean visible) {
            super.onVisibilityChanged(visible);
            /* the watch face became visible or invisible */
        }
    }
}
```

> **Note:** Android SDK 裡的 *WatchFace* 示例示範瞭如何通過繼承 `CanvasWatchFaceService ` 類來實現模擬和數字表盤。這個示例位於 `android-sdk/samples/android-21/wearable/WatchFace` 目錄。

`CanvasWatchFaceService` 類提供一個類似 [View.invalidate()](http://developer.android.com/reference/android/view/View.html#invalidate()) 方法的銷燬機制。當我們想要系統重新繪製錶盤時，我們可以在實現中調用 `invalidate()` 方法。在主 UI 線程中，我們可以只用 `invalidate()` 方法。然後調用 `postInvalidate()` 方法從其它的的線程中銷燬畫布。

更多關於實現 `CanvasWatchFaceService.Engine` 類的方法，請見[繪製錶盤](drawing.html)。

## 註冊表盤服務

實現完錶盤服務之後，我們需要在可穿戴應用的 manifest 文件中註冊該實現。當用戶安裝此應用時，系統會使用關於服務的信息，使得可穿戴設備上 Android Wear 配套應用和錶盤選擇器裡的錶盤可用。

下面的代碼片段介紹瞭如何在 [application](http://developer.android.com/guide/topics/manifest/application-element.html) 節點下注冊一個錶盤實現：

```xml
<service
    android:name=".AnalogWatchFaceService"
    android:label="@string/analog_name"
    android:allowEmbedded="true"
    android:taskAffinity=""
    android:permission="android.permission.BIND_WALLPAPER" >
    <meta-data
        android:name="android.service.wallpaper"
        android:resource="@xml/watch_face" />
    <meta-data
        android:name="com.google.android.wearable.watchface.preview"
        android:resource="@drawable/preview_analog" />
    <meta-data
        android:name="com.google.android.wearable.watchface.preview_circular"
        android:resource="@drawable/preview_analog_circular" />
    <intent-filter>
        <action android:name="android.service.wallpaper.WallpaperService" />
        <category
            android:name=
            "com.google.android.wearable.watchface.category.WATCH_FACE" />
    </intent-filter>
</service>
```

當向用戶展示所有安裝在可穿戴設備的錶盤時，設備上的Android Wear 配套應用和錶盤選擇器使用 `com.google.android.wearable.watchface.preview` 元數據項定義的預覽圖。為了取得這個 drawable，可以運行 Android Wear 設備或者模擬器上的錶盤並截圖。在 hdpi 屏幕的 Android Wear 設備上，預覽圖像一般是 320x320 像素。

圓形設備上看起來非常不同的錶盤可以提供圓形和方形的預覽圖。使用 `com.google.android.wearable.watchface.preview` 元數據項指定一個圓形的預覽圖。如果一個錶盤包含兩種預覽圖，可穿戴應用上的配套應用和錶盤選擇器會根據手錶的形狀選擇適合的預覽圖。如果沒有包含圓形的預覽圖，那麼方形和圓形的設備都會用方形的預覽圖。對於圓形的設備，方形的預覽圖會被一個圓形剪裁掉。

`android.service.wallpaper` 元數據項指定包含 `wallpaper` 節點的 `watch_face.xml` 資源文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<wallpaper xmlns:android="http://schemas.android.com/apk/res/android" />
```

我們的可穿戴應用可以包含多個錶盤。我們必須為每個錶盤實現添加一個服務節點到可穿戴應用的 manifest 文件中。