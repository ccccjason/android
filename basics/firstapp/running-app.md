# 執行Android程序

> 編寫:[yuanfentiank789](https://github.com/yuanfentiank789) - 原文:<http://developer.android.com/training/basics/firstapp/running-app.html>

通過[上一節課](creating-project.html)創建了一個Android的Hello World項目，項目默認包含一系列源文件，它讓我們可以立即運行應用程序。

如何運行Android應用取決於兩件事情：是否有一個Android設備和是否正在使用Android Studio開發程序。本節課將會教使用Android Studio和命令行兩種方式在真實的android設備或者android模擬器上安裝並且運行應用。

## 在真實設備上運行

如果有一個真實的Android設備，以下的步驟可以使我們在自己的設備上安裝和運行應用程序：

### 手機設置

1. 把設備用USB線連接到計算機上。如果是在windows系統上進行開發的，你可能還需要安裝你設備對應的USB驅動，詳見[OEM USB Drivers](http://developer.android.com/tools/extras/oem-usb.html) 文檔。
2. 開啟設備上的**USB調試**選項。
    * 在大部分運行Andriod3.2或更老版本系統的設備上，這個選項位於“**設置**>**應用程序**>**開發選項**”裡。
    * 在Andriod 4.0或更新版本中，這個選項在“**設置**>**開發人員選項**”裡。

> **Note:** 從Android4.2開始，**開發人員選項**在默認情況下是隱藏的，想讓它可見，可以去**設置>關於手機（或者關於設備)**點擊**版本號**七次。再返回就能找到**開發人員選項**了。

### 從Android Studio運行程序

1. 選擇項目的一個文件，點擊工具欄裡的**Run**![as-run](as-run.png)按鈕。

2. **Choose Device**窗口出現時，選擇**Choose a running device**單選框，點擊**OK**。

Android Studio 會把應用程序安裝到我們的設備中並啟動應用程序。

### 從命令行安裝運行應用程序

打開命令行並切換當前目錄到Andriod項目的根目錄，在debug模式下使用Gradle編譯項目，使用gradle腳本執行assembleDebug編譯項目，執行後會在build/目錄下生成MyFirstApp-debug.apk。

Windows操作系統下，執行：

```
gradlew.bat assembleDebug
```

Mac OS或Linux系統下：

```
$ chmod +x gradlew
$ ./gradlew assembleDebug
```

編譯完成後在app/build/outputs/apk/目錄生成apk。

> **Note:** chmod命令是給gradlew增加執行權限，只需要執行一次。

確保 Android SDK裡的 `platform-tools/` 路徑已經添加到環境變量`PATH`中，執行：

```
adb install bin/MyFirstApp-debug.apk
```

在我們的Android設備中找到 MyFirstActivity，點擊打開。

## 在模擬器上運行

無論是使用 Android Studio 還是命令行，在模擬器中運行程序首先要創建一個 [Android Virtual Device](http://developer.android.com/tools/devices/index.html) (AVD)。AVD 是對 Android 模擬器的配置，可以讓我們模擬不同的設備。

###創建一個 AVD:
1\. 啟動 Android Virtual Device Manager（AVD Manager）的兩種方式：
    * 用Android Studio, **Tools > Android > AVD Manager**,或者點擊工具欄裡面Android Virtual Device Manager![image](avd-manager-studio.png)；
    * 在命令行窗口中，把當前目錄切換到`<sdk>/tools/` 後執行：
```
android avd
```
![avds-config](studio-avdmgr-firstscreen.png)

2\. 在AVD Manager 面板中，點擊**Create Virtual Device**.

3\. 在Select Hardware窗口，選擇一個設備，比如 Nexus 6，點擊**Next**。

4\. 選擇列出的合適系統鏡像.

5\. 校驗模擬器配置，點擊**Finish**。

更多AVD的知識請閱讀[Managing AVDs with AVD Manager](http://developer.android.com/tools/devices/managing-avds.html).

### 從Android Studio運行程序：

1\. 在Android Studio選擇要運行的項目，從工具欄選擇**Run**![image](as-run.png)；

2\. **Choose Device**窗口出現時，選擇**Launch emulator**單選框；

3\. 從** Android virtual device**下拉菜單選擇創建好的模擬器，點擊**OK**；

模擬器啟動需要幾分鐘的時間，啟動完成後，解鎖即可看到程序已經運行到模擬器屏幕上了。

### 從命令行安裝運行應用程序

1\. 用命令行編譯應用，生成位於app/build/outputs/apk/的apk；

2\. 確認platform-tools/ 已添加到PATH環境變量；

3\. 執行如下命令：

```
adb install app/build/outputs/MyFirstApp-debug.apk
```
4\. 在模擬器上找到MyFirstApp，並運行。

以上就是創建並在設備上運行一個應用的全部過程！想要開始開發，點擊[next lesson](building-ui.html)。
