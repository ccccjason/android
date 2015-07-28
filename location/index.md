# Android位置信息

> 編寫:[penkzhou](https://github.com/penkzhou) - 原文:<http://developer.android.com/training/location/index.html>

位置感知是移動應用一個獨特的功能。用戶去到哪裡都會帶著他們的移動設備，而將位置感知功能添加到我們的應用裡，可以讓用戶有更加真實的情境體驗。位置服務API集成在Google Play服務裡面，這便於我們將自動位置跟蹤、地理圍欄和用戶活動識別等位置感知功能添加到我們的應用當中。

我們喜歡用[Google Play services location APIs](http://developer.android.com/reference/com/google/android/gms/location/package-summary.html)勝過Android framework location APIs ([android.location](http://developer.android.com/reference/android/location/package-summary.html)) 來給我們的應用添加位置感知功能。如果你現在正在使用Android framework location APIs，我們強烈建議你儘可能切換到Google Play services location APIs。

這個課程介紹如何使用Google Play services location APIs來獲取當前位置、週期性地更新位置以及查找地址。創建並監視地理圍欄以及探測用戶的活動。這個課程包括示例應用和代碼片段，你可以利用這些資源作為添加位置感知到你的應用的基礎。

> **Note：**因為這個課程基於Google Play services client library，所以在使用這些示例應用和代碼段之前確保你安裝了最新版本的Google Play services client library。要想學習如何安裝最新版的client library，請參考[安裝Google Play services嚮導](http://developer.android.com/google/play-services/setup.html)。

## Lessons

* [**獲取最後可知位置**](retrieve-current.html)

    學習如何獲取Android設備的最後可知位置。通常Android設備的最後可知位置相當於用戶的當前位置。


* [**接收位置更新**](receive-location-updates.html)

    學習如何請求和接收週期性的位置更新。


* [**顯示位置地址**](display-address.html)

    學習如何將一個位置的經緯度轉化成一個地址（反向地理編碼）。


* [**創建和監視地理圍欄**](geofencing.html)

    學習如何將一個或多個地理區域定義成一個興趣位置集合，稱為地理圍欄。學習如何探測用戶靠近或者進入地理圍欄事件。
