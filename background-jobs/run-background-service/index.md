# 在IntentService中執行後臺任務

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/run-background-service/index.html>

除非我們特別為某個操作指定特定的線程，否則大部分在前臺UI界面上的操作任務都執行在一個叫做UI Thread的特殊線程中。這可能存在某些隱患，因為部分在UI界面上的耗時操作可能會影響界面的響應性能。UI界面的性能問題會容易惹惱用戶，甚至可能導致系統ANR錯誤。為了避免這樣的問題，Android Framework提供了幾個類，用來幫助你把那些耗時操作移動到後臺線程中執行。那些類中最常用的就是[IntentService](http://developer.android.com/reference/android/app/IntentService.html).

這一章節會講到如何實現一個IntentService，向它發送任務並反饋任務的結果給其他模塊。

## Demos
[**ThreadSample.zip**](http://developer.android.com/shareables/training/ThreadSample.zip)

## Lessons

* [創建IntentService](create-service.html)

  學習如何創建一個IntentService。


* [發送任務請求給IntentService](send-request.html)

  學習如何發送工作任務給IntentService。


* [報告後臺任務的執行狀態](report-status.html)

  學習如何使用Intent與LocalBroadcastManager在Activit與IntentService之間進行交互。
