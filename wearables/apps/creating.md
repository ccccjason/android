# 創建並運行可穿戴應用

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/wearables/apps/creating.html>

可穿戴應用可以直接運行在可穿戴的設備上。擁有訪問類似傳感器的硬件權限，還有操作activity，services等權限。

當我們想要將可穿戴設備應用發佈到Google Play商店時，我們需要有該應用的配套手持設備應用。因為可穿戴設備不支持Google Play商店，所以當用戶下載配套手持設備應用的時候，會自動安裝可穿戴應用到可穿戴設備上。手持設備應用還可以用來處理一些繁重的任務、網絡指令或者其它工作，和發送操作結果給可穿戴設備。

這節課會介紹如何安裝一個設備或者模擬器，和如何創建一個包含了手持應用與可穿戴應用的工程。

## 升級 SDK

在開始建立可穿戴設備應用前，必須：

* **將SDK工具升級到23.0.0或者更高的版本**

　　升級後的SDK工具使我們可以建立和測試可穿戴應用。

* **將SDK升級到 Android 4.4W.2(API 20) 或者更高**

　　升級後的平臺版本為可穿戴應用提供了新的 API。

想要了解如何升級SDK，請查看[Get the latest SDK tools](http://developer.android.com/sdk/installing/adding-packages.html#GetTools)。

## 搭建Android Wear模擬器或者真機設備。

我們推薦在真機上進行開發，這樣可以更好地評估用戶體驗。然而，模擬器可以使我們在不同類型的設備屏幕上進行模擬，這對測試來說非常有用。

### 搭建Android Wear虛擬設備

建立Android Wear虛擬設備需要下面幾個步驟：

1. 點擊**Tools > Android > AVD Manager**。
2. 點擊**Create Virtual Device...**。
	1. 點擊Category列表的**Wear**選項。
	2. 選擇Android Wear Square或者Android Wear Round。
	3. 點擊**Next**按鈕。
	4. 選擇一個release name（例如，KitKat Wear）。
	5. 點擊**Next**按鈕。
	6. （可選）改變虛擬設備的首選項。
	7. 點擊**Finish**按鈕。
3. 啟動模擬器:
	1. 選擇我們剛才創建的虛擬設備。
	2. 點擊**Play**按鈕。
	3. 等待模擬器初始化直到顯示Android Wear的主界面。
4. 匹配手持和模擬器:
	1. 在我們的手持設備上，從Google Play安裝Android Wear應用。
	2. 通過USB將手持設備連接到電腦。
	3. 切換AVD的通信端口到已連接的手持設備(每次連接上手持設備時都要執行這個步驟)：
    ```git
    adb -d forward tcp:5601 tcp:5601
    ```
	4. 啟動手持設備上的Android Wear應用，並連接到模擬器。
	5. 點擊Android Wear應用右上角的菜單，選擇**Demo Cards**。
	6. 我們選擇的卡片會以Notification的形式呈現在模擬器的主頁上。

### 搭建Android Wear真機

建立Android Wear真機，需要下面幾個步驟：

1. 在手持設備的Google Play上安裝Android Wear應用。
2. 按照應用的命令指示與我們的可穿戴設備進行配對。如果你有做建立notification的操作，這個步驟剛好可以測試這一功能。
3. 保持Android Wear應用在手機上的打開狀態。
4. 打開Android Wear設備的adb調試開關。
	1. 選擇**Settings > About**。
	2. 點擊**Build number** 7次。
	3. 右滑返回到Setting菜單。
	4. 進入屏幕底部的**Developer options**。
	5. 點擊**ADB Debugging**來打開adb。
5. 通過USB連接可穿戴設備到電腦上，這樣我們能夠直接安裝應用到可穿戴設備上。此時，在可穿戴設備與Android Wear應用上會顯示一個消息，提示是否允許進行調試。
6. 在Android Wear應用上，選擇**Always allow from this computer**並且點擊**OK**。

Android Studio上的**Android** Tool窗口可以顯示可穿戴設備的日誌。當你執行`adb devices`命令的時候，可穿戴設備應該會出現在該窗口中。

## 創建工程

在開始開發之前，需要創建一個包含可穿戴應用與手持應用這兩個模塊的工程。在Android Studio中，點擊**File** > **New Project**，然後按照[創建工程](http://developer.android.com/sdk/installing/create-project.html)的指引進行操作。在我們按照安裝嚮導操作的過程中，輸入下面的信息：

1. 在**Configure your Project**窗口裡，輸入應用的名稱與一個包名。
2. 在**Form Factors**窗口中:
    * 勾選**Phone and Tablet**並在**Minimum SDK**下拉菜單中選擇**API 9: Android 2.3 (Gingerbread)**。
    * 勾選**Wear**並在**Minimum SDK**下拉菜單中選擇**API 20: Android 4.4 (KitKat Wear)**。
3. 在第一個**Add an Activity**窗口，為手機應用添加一個空白的activity。
4. 在第二個**Add an Activity**窗口，為可穿戴應用添加一個空白的activity。

當安裝嚮導完成後，Andorid Studio創建了一個包含**mobile**與**wear**兩個模塊的工程。現在，我們有一個工程可以在手持設備和可穿戴設備應用中創建activity，service，layout等。在手持應用裡面，需要承擔大部分繁重的任務，例如網絡請求，密集計算任務或者是需要大量用戶交互的任務。待這些任務完成之後，通常會把任務結果通過notification發送給可穿戴設備上，或者是通過同步機制發送數據給可穿戴設備。

> **Note:** 可穿戴模塊包含了一個"Hello World"的activity，它是使用`WatchViewStub`類。該類根據設備屏幕是圓的還是方的來填充一個佈局。`WatchViewStub`類是[wearable support library](http://hukai.me/android-training-course-in-chinese/wearables/apps/layouts.html#UiLibrary)中的一個UI組件。

## 安裝可穿戴應用

在開發過程中，我們可以像安裝手持應用一樣直接將應用安裝到可穿戴設備上。可以使用`adb install`命令，也可以使用Android Studio上面的**Play**按鈕。

當需要把應用發佈給用戶的時候，需要把可穿戴應用打包到手持應用中。當用戶從Google Play安裝手持應用時，連接上的可穿戴設備會自動收到可穿戴應用。

> **Note:** 如果我們給應用簽名是debug key，是無法完成自動安裝可穿戴應用的（只有release key才可以）。請參考[打包可穿戴應用](packaging.html)獲取更多信息，學習如何正確的打包。

為了安裝"Hello World"應用到可穿戴設備，在Android Studiod的**Run/Debug configuration**的下拉菜單中選中**wear**，點擊**Play**按鈕即可。在可穿戴設備上會顯示activity並打印"Hello world!"

## include正確的libraries

項目安裝嚮導會自動把合適的模塊依賴添加到對應的`build.gradle`文件中。然而，這些依賴並不是必須的，請閱讀下面描述判斷是否需要這些依賴。

**Notifications**

[The Android v4 support library](http://developer.android.com/tools/support-library/features.html#v4) (或者v13)包含一些API，這些API可以將手持設備應用已經存在的notification擴展到可穿戴應用上。

對於只顯示在可穿戴設備上的notification(這意味著，他們是由直接執行在可穿戴設備上的app進行處理的)，我們可以在Wear模塊僅僅使用標準APIs (API Level 20) 並且把Mobile模塊的support library依賴移除。

**Wearable Data Layer**

可穿戴與手持設備之間進行同步與發送數據需要使用Wearable Data Layer APIs, 這需要用到最新版本的[Google Play Services](http://developer.android.com/google/play-services/setup.html)。如果我們不需要這些APIs，可以從這兩個模塊中把這部分的依賴移除。

**Wearable UI support library**

這是一個非官方正式的library，它包含了[為可穿戴設備設計的UI組件](http://hukai.me/android-training-course-in-chinese/wearables/apps/layouts.html#UiLibrary)。我們鼓勵你在你的應用中使用他們，因為這些組件是最佳實踐的例證。但是他們可能隨時發生變化。然而，如果library有更新，你的應用並不會發送崩潰，因為那些代碼已經編譯到你的應用中了。為了獲取更新包中新的功能，你只需要更新鏈接到新的版本並相應的更新你的應用就好了。這個library只是在你需要創建可穿戴應用時才會使用到。

在下一節課，我們將會學習如何創建為可穿戴設備設計的佈局，同時學習如何使用各種語音action。