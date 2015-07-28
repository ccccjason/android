# 與其他應用的交互

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/basics/intents/index.html>

* 一個Android app通常都會有多個activities。 每個activity的界面都扮演者用戶接口的角色，允許用戶執行一些特定任務（例如查看地圖或者是開始拍照等）。為了讓用戶能夠從一個activity跳到另一個activity，我們的app必須使用Intent來定義自己的意圖。當使用startActivity()的方法，且參數是intent時，系統會使用這個 Intent 來定義並啟動合適的app組件。使用intents甚至還可以讓app啟動另一個app裡面的activity。
* 一個 Intent 可以顯式的指明需要啟動的模塊（用一個指定的Activity實例），也可以隱式的指明自己可以處理哪種類型的動作（比如拍一張照等）。
* 本章節將演示如何使用Intent 與其他app執行一些基本的交互。比如啟動另外一個app，從其他app接受數據，以及使得我們的app能夠響應從其他app中發出的intent等。

## Lessons
* [**Intent的發送(Sending the User to Another App )**](sending.html)

  演示如何創建一個隱式Intent喚起能夠接收這個動作的App。


* [**接收Activity返回的結果(Getting a Result from an Activity)**](result.html)

  演示如何啟動另外一個Activity並接收返回值。


* [**Intent過濾(Allowing Other Apps to Start Your Activity)**](filters.html)

  演示如何通過定義隱式的Intent的過濾器來使我們的應用能夠被其他應用喚起。
