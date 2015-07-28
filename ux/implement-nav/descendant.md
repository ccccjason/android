# 實現向下的導航

> 編寫:[Lin-H](https://github.com/Lin-H) - 原文:<http://developer.android.com/training/implementing-navigation/descendant.html>

Descendant Navigation是用來向下導航至應用的信息層次。在[Designing Effective Navigation](http://developer.android.com/training/design-navigation/descendant-lateral.html)和[Android Design: Application Structure](http://developer.android.com/design/patterns/app-structure.html)中說明。

Descendant navigation通常使用[Intent](http://developer.android.com/reference/android/content/Intent.html)和[startActivity()](http://developer.android.com/reference/android/content/Context.html#startActivity%28android.content.Intent%29)實現，或使用[FragmentTransaction](http://developer.android.com/reference/android/app/FragmentTransaction.html)對象添加fragment到一個activity中。這節課程涵蓋了在實現Descendant navigation時遇到的其他有趣的情況。

## 在手機和平板(Tablet)上實現Master/Detail Flow

在master/detail導航流程(navigation flow)中，master screen(主屏幕)包含一個集合中item的列表，detail screen(詳細屏幕)顯示集合中特定item的詳細信息。實現從master screen到detail screen的導航是Descendant Navigation的一種形式。

手機觸摸屏非常適合一次顯示一種屏幕(master screen或detail screen)；這一想法在[Planning for Multiple Touchscreen Sizes](http://developer.android.com/training/design-navigation/multiple-sizes.html)中進一步說明。在這種情況下，一般使用[Intent](http://developer.android.com/reference/android/content/Intent.html)啟動detail screen來實現activityDescendant navigation。另一方面，平板的顯示，特別是用橫屏來瀏覽時，最適合一次顯示多個內容窗格，master內容在左邊，detail在右邊。在這裡一般就使用[FragmentTransaction](http://developer.android.com/reference/android/app/FragmentTransaction.html)實現descendant navigation。[FragmentTransaction](http://developer.android.com/reference/android/app/FragmentTransaction.html)用來添加、刪除或用新內容替換detail窗格(pane)。

實現這一模式的基礎內容在Designing for Multiple Screens的[Implementing Adaptive UI Flows](http://developer.android.com/training/multiscreen/adaptui.html)課程中說明。課程中說明了如何在手機上使用兩個activity，在平板上使用一個activity來實現master/detail flow。

## 導航至外部Activities

有很多情況，是從別的應用下降(descend)至你的應用信息層次(application's information hierarchy)再到activity。例如，當正在瀏覽手機通訊錄中聯繫信息的details screen，子屏幕詳細顯示由社交網絡聯繫提供的最近文章，子屏幕可就可以屬於一個社交網絡應用。

當啟動另一個應用的activity來允許用戶說話，發郵件或選擇一個照片附件，如果用戶是從啟動器(設備的home屏幕)重啟你的應用，你一般不會希望用戶返回到別的activity。如果點擊你的應用圖標又回到“發郵件”的屏幕，這會使用戶感到很迷惑。

為防止這種情況的發生，只需要添加[FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET](http://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET)標記到用來啟動外部activity的intent中，就像:

```java
Intent externalActivityIntent = new Intent(Intent.ACTION_PICK);
externalActivityIntent.setType("image/*");
externalActivityIntent.addFlags(
        Intent.FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET);
startActivity(externalActivityIntent);
```
