# 隱藏導航欄

> 編寫:[K0ST](https://github.com/K0ST) - 原文:<http://developer.android.com/training/system-ui/navigation.html>

**這節課將教您**

1. 在4.0及以上版本中隱藏導航欄
2. 讓內容顯示在導航欄之後

本節課程將教您如何對導航欄進行隱藏，這個特性是Android 4.0（）版本中引入的。

即便本小節僅關注如何隱藏導航欄，但是在實際的開發中，你最好讓狀態欄與導航欄同時消失。在保證導航欄易於再次訪問的情況下，隱藏導航欄與狀態欄使內容區域佔據了整個顯示空間，因此可以提供一個更加沉浸式的用戶體驗。

![navigation-bar](navigation-bar.png)

**圖1**. 導航欄.

## 在4.0及以上版本中隱藏導航欄

你可以在Android 4.0以及以上版本，使用`SYSTEM_UI_FLAG_HIDE_NAVIGATION`標誌來隱藏導航欄。這段代碼同時隱藏了導航欄和系統欄：


```java
View decorView = getWindow().getDecorView();
// Hide both the navigation bar and the status bar.
// SYSTEM_UI_FLAG_FULLSCREEN is only available on Android 4.1 and higher, but as
// a general rule, you should design your app to hide the status bar whenever you
// hide the navigation bar.
int uiOptions = View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
              | View.SYSTEM_UI_FLAG_FULLSCREEN;
decorView.setSystemUiVisibility(uiOptions);
```

注意以下幾點
* 使用這個方法時，觸摸屏幕的任何一個區域都會使導航欄（與狀態欄）重新顯示。用戶的交互會使這個標籤`SYSTEM_UI_FLAG_HIDE_NAVIGATION`被清除。
* 一旦這個標籤被清除了，如果你想再次隱藏導航欄，你就需要重新對這個標籤進行設定。在下一節[響應UI可見性的變化](visibility.html)中，將詳細講解應用監聽系統UI變化來做出相應的調整操作。
* 在不同的地方設置UI標籤是有所區別的。如果你在Activity的onCreate()方法中隱藏系統欄，當用戶按下home鍵系統欄就會重新顯示。當用戶再重新打開activity的時候，onCreate()不會被調用，所以系統欄還會保持可見。如果你想讓在不同Activity之間切換時，系統UI保持不變，你需要在onReasume()與onWindowFocusChaned()裡設定UI標籤。
* setSystemUiVisibility()僅僅在被調用的View顯示的時候才會生效。
* 當從View導航到別的地方時，用setSystemUiVisibility()設置的標籤會被清除。


## 2)讓內容顯示在導航欄之後

在Android 4.1與更高的版本中，你可以讓應用的內容顯示在導航欄的後面，這樣當導航欄展示或隱藏的時候內容區域就不會發生布局大小的變化。可以使用`SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION`標籤來做到這個效果。同時，你也有可能需要`SYSTEM_UI_FLAG_LAYOUT_STABLE`這個標籤來幫助你的應用維持一個穩定的佈局。

當你使用這種方法的時候，就需要你來確保應用中特定區域不會被系統欄掩蓋。更詳細的信息可以瀏覽[隱藏狀態欄](hide-ui.html)一節。


