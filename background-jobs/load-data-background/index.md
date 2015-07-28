# 使用CursorLoader在後臺加載數據

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/load-data-background/index.html>

從[ContentProvider](http://developer.android.com/reference/android/content/ContentProvider.html)查詢你需要顯示的數據是比較耗時的。如果你在Activity中直接執行查詢的操作，那麼有可能導致Activity出現ANR的錯誤。即使沒有發生ANR，用戶也容易感知到一個令人煩惱的UI卡頓。為了避免那些問題，你應該在另外一個線程中執行查詢的操作，等待查詢操作完成，然後再顯示查詢結果。

通過[CursorLoader](http://developer.android.com/reference/android/support/v4/content/CursorLoader.html)對象，你可以用一種簡單的方式實現異步查詢，查詢結束時它會和Activity進行重新連接。
CursorLoader不僅僅能夠實現在後臺查詢數據，還能夠在查詢數據發生變化時自動執行重新查詢的操作。

這節課會介紹如何使用CursorLoader來執行一個後臺查詢數據的操作。在這節課中的演示代碼使用的是[v4 Support Library](http://developer.android.com/tools/support-library/features.html#v4)中的類。

## Demos

** [ThreadSample](http://developer.android.com/shareables/training/ThreadSample.zip) **

## Lessons

* [使用CursorLoader執行查詢任務](setup-loader.html)

  學習如何使用CursorLoader在後臺執行查詢操作。


* [處理CursorLoader查詢的結果](handle-result.html)

  學習如何處理從CursorLoader查詢到的數據，以及在loader框架重置CursorLoader時如何解除當前Cursor的引用。
