# 為可穿戴設備創建自定義UI

> 編寫: [roya](https://github.com/RoyaAoki) 原文:<https://developer.android.com/training/wearables/ui/index.html>

可穿戴apps的用戶界面明顯的不同於手持設備。可穿戴設備應用應該參考Android Wear[設計規範](https://developer.android.com/design/wear/index.html)和實現推薦的[UI模式](https://developer.android.com/design/wear/patterns.html)，
這些保證在為可穿戴設備優化過的應用中保持統一的用戶體驗。

這個課程將教我們如何為[可穿戴應用](http://hukai.me/android-training-course-in-chinese/wearables/apps/creating.html)創建在所有Android可穿戴設備上看上去都不錯的自定義UI和[自定義的notifications](http://hukai.me/android-training-course-in-chinese/wearables/apps/layouts.html#CustomNotifications)。為了達到上述目的，需要實現這些UI模式：

* Card
* 倒計時和確認
* 長按退出
* 2D Picker
* 多選List

可穿戴UI庫是Android SDK的Google Repository中的一部分，其中提供的類可以幫助我們實現這些模式和創建工作在圓形和方形Android可穿戴設備的layout。

>**Note:** 我們推薦使用Android Studio做Android Wear開發,它提供工程初始配置,庫包含和方便的打包,這些在ADT中是沒有的。這系列教程假定你正在使用Android Studio。

## Lessons

[定義Layouts](https://developer.android.com/training/wearables/ui/layouts.html)

學習如何創建在圓形和方形Android Wear設備上看起來不錯的layout。
	
[創建Card](https://developer.android.com/training/wearables/ui/cards.html)
 
學習如何創建自定義layout的Card
  
[創建List](https://developer.android.com/training/wearables/ui/lists.html)

學習如何創建為可穿戴設備優化的List
  
[創建2D Picker](https://developer.android.com/training/wearables/ui/2d-picker.html)

學習如何實現2D Picker UI模式以導航各頁數據
  
[顯示確認界面](https://developer.android.com/training/wearables/ui/confirm.html)

學習如何在用戶完成操作時顯示確認動畫
  
[退出全屏的Activity](https://developer.android.com/training/wearables/ui/exit.html)

學習如何實現長按退出UI模式以退出全屏activities