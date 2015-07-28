# 創建Android項目

> 編寫:[yuanfentiank789](https://github.com/yuanfentiank789) - 原文:<http://developer.android.com/training/basics/firstapp/creating-project.html>

一個Android項目包含了所有構成Android應用的源代碼文件。

本小節介紹如何使用Android Studio或者是SDK Tools中的命令行來創建一個新的項目。

> **Note**：在此之前，我們應該已經安裝了Android SDK，如果使用Android Studio開發，應該確保已經安裝了[Android Studio](http://developer.android.com/sdk/installing/studio.html)。否則，請先閱讀 [Installing the Android SDK](http://developer.android.com/sdk/installing/index.html)按照嚮導完成安裝步驟。

## 使用Android Studio創建項目

1\. 使用Android Studio創建Android項目，啟動Android Studio。

* 如果我們還沒有用Android Studio打開過項目，會看到歡迎頁，點擊New Project。
* 如果已經用Android Studio打開過項目，點擊菜單中的File，選擇New Project來創建一個新的項目。

2\.  參照圖1在彈出的窗口（**Configure your new project**）中填入內容，點擊**Next**。按照如圖所示的值進行填寫會使得後續的操作步驟不不容易差錯。

* **Application Name**此處填寫想呈現給用戶的應用名稱，此處我們使用“My First App”。
* **Company domain** 包名限定符，Android Studio會將這個限定符應用於每個新建的Android項目。
* **Package Name**是應用的包命名空間（同Java的包的概念），該包名在同一Android系統上所有已安裝的應用中具有唯一性，我們可以獨立地編輯該包名。
* **Project location**操作系統存放項目的目錄。

![studio-setup-1](studio-setup-1.png)
**圖1** Configure your new project

3\. 在**Select the form factors your app will run on**窗口勾選**Phone and Tablet**。

4\. **Minimum SDK**, 選擇 **API 8: Android 2.2 (Froyo)**. Minimum Required SDK表示我們的應用支持的最低Android版本，為了支持儘可能多的設備，我們應該設置為能支持你應用核心功能的最低API版本。如果某些非核心功能僅在較高版本的API支持，你可以只在支持這些功能的版本上開啟它們(參考[兼容不同的系統版本](../supporting-devices/platforms.html)),此處採用默認值即可。

5\. 不要勾選其他選項 (TV, Wear, and Glass) ，點擊 **Next**.

6\. 在**Add an activity to *<template\>*** 窗口選擇**Blank Activity**，點擊 **Next**.

7\. 在**Choose options for your new file** 窗口修改**Activity Name** 為*MyActivity*，修改 **Layout Name** 為*activity\_my*，**Title** 修改為*MyActivity*，**Menu Resource Name** 修改為*menu_my*。

8\. 點擊**Finish**完成創建。

剛創建的Android項目是一個基礎的Hello World項目，包含一些默認文件，我們花一點時間看看最重要的部分：

`app/src/main/res/layout/activity_my.xml`

這是剛才用Android Studio創建項目時新建的Activity對應的xml佈局文件，按照創建新項目的流程，Android Studio會同時展示這個文件的文本視圖和圖形化預覽視圖，該文件包含一些默認設置和一個顯示內容為“Hello world!”的TextView元素。

`app/src/main/java/com.mycompany.myfirstapp/MyActivity.java`

用Android Studio創建新項目完成後，可在Android Studio看到該文件對應的選項卡，選中該選項卡，可以看到剛創建的Activity類的定義。編譯並運行該項目後，Activity啟動並加載佈局文件activity_my.xml，顯示一條文本："Hello world!"

`app/src/main/AndroidManifest.xml`

[manifest](http://developer.android.com/guide/topics/manifest/manifest-intro.html)文件描述了項目的基本特徵並列出了組成應用的各個組件，接下來的學習會更深入瞭解這個文件並添加更多組件到該文件中。

`app/build.gradle`

Android Studio使用Gradle 編譯運行Android工程. 工程的每個模塊以及整個工程都有一個build.gradle文件。通常你只需要關注模塊的build.gradle文件，該文件存放編譯依賴設置，包括defaultConfig設置：

* compiledSdkVersion
是我們的應用將要編譯的目標Android版本，此處默認為你的SDK已安裝的最新Android版本(目前應該是4.1或更高版本，如果你沒有安裝一個可用Android版本，就要先用[SDK Manager](http://developer.android.com/sdk/installing/adding-packages.html)來完成安裝)，我們仍然可以使用較老的版本編譯項目，但把該值設為最新版本，可以使用Android的最新特性，同時可以在最新的設備上優化應用來提高用戶體驗。
* applicationId 創建新項目時指定的包名。
* minSdkVersion 創建項目時指定的最低SDK版本，是新建應用支持的最低SDK版本。
* targetSdkVersion 表示你測試過你的應用支持的最高Android版本(同樣用API level表示).當Android發佈最新版本後，我們應該在最新版本的Android測試自己的應用同時更新target sdk到Android最新版本，以便充分利用Android新版本的特性。更多知識，請閱讀[Supporting Different Platform Versions](http://developer.android.com/training/basics/supporting-devices/platforms.html)。


更多關於Gradle的知識請閱讀[Building Your Project with Gradle](http://developer.android.com/sdk/installing/studio-build.html)

注意/res目錄下也包含了[resources](http://developer.android.com/guide/topics/resources/overview.html)資源：

`drawable<density>/`

存放各種densities圖像的文件夾，mdpi，hdpi等，這裡能夠找到應用運行時的圖標文件ic_launcher.png

`layout/`

存放用戶界面文件，如前邊提到的activity_my.xml，描述了MyActivity對應的用戶界面。

`menu/`

存放應用裡定義菜單項的文件。

`values/`

存放其他xml資源文件，如string，color定義。string.xml定義了運行應用時顯示的文本"Hello world!"

要運行這個APP，繼續[下個小節](running-app.html)的學習。

## 使用命令行創建項目

如果沒有使用Android Studio開發Android項目，我們可以在命令行使用SDK提供的tools來創建一個Android項目。

1\. 打開命令行切換到SDK根目錄下；

2\. 執行:

```java
tools/android list targets
```

會在屏幕上打印出我們所有的Android SDK中下載好的可用Android  platforms，找想要創建項目的目標platform，記錄該platform對應的Id，推薦使用最新的platform。我們仍可以使自己的應用支持較老版本的platform，但設置為最新版本允許我們為最新的Android設備優化我們的應用。
如果沒有看到任何可用的platform，我們需要使用Android SDK Manager完成下載安裝，參見 [Adding Platforms and Packages](http://developer.android.com/sdk/installing/adding-packages.html)。

3\. 執行：

```java
android create project --target <target-id> --name MyFirstApp \
--path <path-to-workspace>/MyFirstApp --activity MyActivity \
--package com.example.myfirstapp
```

替換`<target-id>`為上一步記錄好的Id，替換`<path-to-workspace>`為我們想要保存項目的路徑。

> **Tip**:把`platform-tools/`和 `tools/`添加到環境變量`PATH`，開發更方便。

到此為止，我們的Android項目已經是一個基本的“Hello World”程序，包含了一些默認的文件。要運行它，繼續[下個小節](running-app.html)的學習。
