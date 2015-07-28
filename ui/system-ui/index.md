# 管理系統UI

> 編寫:[K0ST](https://github.com/K0ST) - 原文:<http://developer.android.com/training/system-ui/index.html>

[System Bar](https://developer.android.com/design/get-started/ui-overview.html#system-bars)是用來展示通知、表現設備狀態和完成設備導航的屏幕區域。通常上來說，系統欄(System bar)包括狀態欄和導航欄(Figure 1)，他們一般都是與程序同時顯示在屏幕上的。而照片、視頻等這類沉浸式的應用可以臨時弱化系統欄圖標來創造一個更加專注的體驗環境，甚至可以完全隱藏系統Bar。

![](system-ui.png)

Figure 1. System bars，包含[1]狀態欄，和[2]導航欄。

如果你對[Android Design Guide](http://developer.android.com/design/index.html)很熟悉，你應該已經知道遵照標準的Android UI Guideline與遵循模式來設計App的重要性。在你修改系統欄之前，你應該仔細的考慮一下用戶的需求與預期，因為它們是操作設備和觀察設備狀態的的常規途徑。

這節課描述瞭如何在不同版本的Android上隱藏或淡化系統欄，來營造一個沉浸式的用戶體驗，同時做到快速的訪問與操作系統欄。

## Sample

**ImmersiveMode** - <http://developer.android.com/samples/ImmersiveMode/index.html
>

## Lessons

* [**淡化系統欄**](dim.md)

  學習如何淡化和隱藏狀態欄與導航欄。


* [**隱藏狀態欄**](hide-ui.md)

  學習如何在不同版本的Android上隱藏狀態欄。


* [**隱藏導航欄**](hide-nav.md)

  學習如何隱藏導航欄。


* [**全屏沉浸式應用**](immersive.md)

  學習如何在你的App中創建沉浸模式。


* [**響應UI可見性的變化**](visibility.md)

  學習如何註冊一個監聽器來監聽系統UI可見性的變化，以便於相應的調整App的UI。
