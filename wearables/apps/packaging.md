# 打包可穿戴應用

> 編寫: [kesenhoo](https://github.com/kesenhoo) - 原文: <http://developer.android.com/training/wearables/apps/packaging.html>

當發佈應用給用戶之前，我們必須把可穿戴應用打包到手持應用內。因為用戶不能直接在可穿戴設備上瀏覽並安裝應用。如果打包正確，當用戶下載手持應用時，系統會自動下發可穿戴應用到配對好的可穿戴設備上。

> **Note:** 如果開發時簽名用的是debug key，這個功能是無法正常工作的。在開發時，需要使用`adb install`命令或者Android Studio來安裝可穿戴應用。

## 使用Android Studio打包

在Android Studio中打包可穿戴應用有下面幾個步驟：

1. 在手持設備應用的manifest文件中包括所有在可穿戴設備應用manifest文件中聲明的權限。例如，如果我們在可穿戴應用中指定了[VIBRATE](http://developer.android.com/reference/android/Manifest.permission.html#VIBRATE)權限，那麼我們必須將該權限添加到手持設備應用中。
2. 確保可穿戴應用和手持應用都有相同的包名和版本號。
3. 在手持應用的`buidl.gradle`文件中聲明一個Gradle依賴用於指向可穿戴應用：
```xml
dependencies {
   compile 'com.google.android.gms:play-services:5.0.+@aar'
   compile 'com.android.support:support-v4:20.0.+''
   wearApp project(':wearable')
}
```
4. 點擊**Build > Generate Signed APK...**，按照屏幕上的指示來制定我們的release key併為我們的app進行簽名。Android Studio將簽名好的內置了可穿戴應用的手持應用自動導出到工程的根目錄。或者，我們可以使用[Gradle wrapper](http://developer.android.com/sdk/installing/studio-build.html#gradleWrapper)在命令行下為在可穿戴應用與手持應用簽名。為了能夠正常自動推送可穿戴應用，這兩個應用都必須簽名。將我們的key文件位置和憑證保存到環境變量中，然後如下運行Gradle wrapper：
```xml
./gradlew assembleRelease \
  -Pandroid.injected.signing.store.file=$KEYFILE \
  -Pandroid.injected.signing.store.password=$STORE_PASSWORD \
  -Pandroid.injected.signing.key.alias=$KEY_ALIAS \
  -Pandroid.injected.signing.key.password=$KEY_PASSWORD
```

### 分別為可穿戴應用與手持應用進行簽名

如果我們的構建過程需要將可穿戴應用的簽名與手持應用的分開，那麼我們可以像下面一樣在手持應用的`build.gradle`文件中聲明Gradle規則。從而嵌入預先簽名的可穿戴應用：

```xml
dependencies {
  ...
  wearApp files('/path/to/wearable_app.apk')
}
```

我們可以以任何我們想要的方式為手持應用進行簽名（可以是Android Studio **Build > Generate Signed APK...**的方式，也可以是Gradle `signingConfig`規則的方式）。

## 手動打包

如果我們使用的是其它IDE或者其它方法來構建應用，我們仍然可以手動地把可穿戴應用打包到手持應用中。

1. 在手機應用的manifest文件中包括所有在可穿戴設備應用manifest文件中聲明的權限。例如，如果我們在可穿戴應用中指定了[VIBRATE](http://developer.android.com/reference/android/Manifest.permission.html#VIBRATE)權限，那麼我們必須將該權限添加到手機應用中。
2. 確保可穿戴應用和手持應用的APK都有相同的包名和版本號。
3. 把簽好名的可穿戴應用放到手持應用工程的`res/raw`目錄下。我們假設這個APK名為`wearable_app.apk`。
4. 創建`res/xml/wearable_app_desc.xml`文件，裡面包含可穿戴設備的版本信息與路徑。例如:
```xml
<wearableApp package="wearable.app.package.name">
  <versionCode>1</versionCode>
  <versionName>1.0</versionName>
  <rawPathResId>wearable_app</rawPathResId>
</wearableApp>
```
`package`, `versionCode`與`versionName`需要和可穿戴應用的AndroidManifest.xml裡面的信息一致。`rawPathResId`是一個靜態變量表示APK的名稱。例如，對於`wearable_app.apk`，這個靜態變量名為`wearable_app`。
5. 添加`meta-data`標籤到我們的手持應用的`<application>`標籤下，指明引用`wearable_app_desc.xml`文件
```xml
<meta-data android:name="com.google.android.wearable.beta.app"
                 android:resource="@xml/wearable_app_desc"/>
```
6. 構建並簽名手持應用。

## 關閉資源壓縮

許多構建工具會自動壓縮放在`res/raw`目錄下的文件。因為可穿戴APK已經被壓縮過了，所以這些工具再次壓縮可穿戴APK會導致可穿戴應用安裝程序無法讀取可穿戴應用。

這樣的話，安裝失敗。在手持應用上，`PackageUpdateService`會輸出如下的錯誤日誌："this file cannot be opened as a file descriptor; it is probably compressed."

Android Studio 默認不會壓縮APK，但是如果我們使用其它構建方式，需要確保不要重複壓縮可穿戴應用。
