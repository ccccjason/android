# 多線程操作

> 編寫:[AllenZheng1991](https://github.com/AllenZheng1991) - 原文:<http://developer.android.com/training/multiple-threads/index.html>

把一個相對耗時且數據操作複雜的任務分割成多個小的操作，然後分別運行在多個線程上，這能夠提高完成任務的速度和效率。在多核CPU的設備上，系統可以並行運行多個線程，而不需要讓每個子操作等待CPU的時間片切換。例如，如果要解碼大量的圖片文件並以縮略圖的形式把圖片顯示在屏幕上，當你把每個解碼操作單獨用一個線程去執行時，會發現速度快了很多。

這個章節會向你展示如何在一個Android應用中創建和使用多線程，以及如何使用線程池對象（thread pool object）。你還將瞭解到如何使得代碼運行在指定的線程中，以及如何讓你創建的線程和UI線程進行通信。

## Sample Code

點擊下載：[**ThreadSample**](http://developer.android.com/shareables/training/ThreadSample.zip)

## 課程

###[在一個線程中執行一段特定的代碼](define-runnable.html)

學習如何通過實現[Runnable](http://developer.android.com/reference/java/lang/Runnable.html)接口定義一個線程類，讓你寫的代碼能在單獨的一個線程中執行。

###[為多線程創建線程池](create-threadpool.html)

學習如何創建一個能管理線程池和任務隊列的對象，需要使用一個叫[ThreadPoolExecutor](http://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html)的類。

###[在線程池中的一個線程裡執行代碼](run-code.html)

學習如何讓線程池裡的一個線程執行一個任務。

###[與UI線程通信](communicate-ui.html)

學習如何讓線程池裡的一個普通線程與UI線程進行通信。
