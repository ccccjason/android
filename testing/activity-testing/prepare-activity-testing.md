# 建立測試環境

> 編寫:[huanglizhuo](https://github.com/huanglizhuo) - 原文:<http://developer.android.com/training/activity-testing/preparing-activity-testing.html>

在開始編寫並運行我們的測試之前，我們應該建立測試開發環境。本小節將會講解如何建立Eclipse IDE來構建和運行我們的測試，以及怎樣用Gradle構建工具在命令行下構建和運行我們的測試。

> 注意: 本小節基於的是Eclipse及ADT插件。然而，你在自己測試開發時可以自由選用IDE或命令行。

## 用Eclipse建立測試

安裝了Android Developer Tools (ADT) 插件的Eclipse將為我們創建，構建，以及運行Android程序提供一個基於圖形界面的集成開發環境。Eclipse可以自動為我們的Android應用項目創建一個對應的測試項目。

開始在Eclipse中創建測試環境:

1. 如果還沒安裝Eclipse [ADT](http://developer.android.com/sdk/installing/bundle.html)插件，請先下載安裝。
2. 導入或創建我們想要測試的Android應用項目。
3. 生成一個對應於應用程序項目測試的測試項目。為導入項目生成一個測試項目:
    a.在項目瀏覽器裡，右擊我們的應用項目，然後選擇**Android Tools > New Test Project**
    b.在新建Android測試項目面板，為我們的測試項目設置合適的參數，然後點擊**Finish**

現在應該可以在Eclipse環境中創建，構建和運行測試項目了。想要繼續學習如何在Eclipse中進行這些任務，可以閱讀[創建與執行測試用例](activity-basic-testing.html)

## 用命令行建立測試

如果正在使用Gradle version 1.6或者更高的版本作為構建工具，可以用Gradle Wrapper創建。構建和運行Android應用測試。確保在`gradle.build`文件中，`defaultConfig`部分中的[minSdkVersion](http://developer.android.com/guide/topics/manifest/uses-sdk-element.html)屬性是8或更高。可以參考包含在下載包中的示例文件gradle.build

## 用Gradle Wrapper運行測試:

1. 連接Android真機或開啟Android模擬器。
2. 在項目目錄運行如下命令:

>./gradlew build connectedCheck

進一步學習Gradle關於Android測試的內容，參看[Gradle Plugin User Guide](http://www.gradle.org/docs/current/userguide/userguide_single.html)。

進一步學習使用Gradle及其它命令行工具，參看[Testing from Other IDEs.](http://developer.android.com/tools/testing/testing_otheride.html)。

本節示例代碼[AndroidTestingFun.zip](http://developer.android.com/shareables/training/AndroidTestingFun.zip)
