# 根據網絡連接類型來調整下載模式

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/efficient-downloads/connectivity_patterns.html>

所有的網絡類型（Wi-Fi、3G、2G等）對電量的消耗並不是一樣的。不僅是 Wi-Fi 電波比無線電波的耗電量要少很多，而且不同的無線電波（3G、2G、LTE……）使用的電量也不同。

## 使用 Wi-Fi

在大多數情況下，Wi-Fi 電波會在使用相對較低電量的情況下提供一個相對較大的帶寬。因此，我們需要爭取儘量使用 Wi-Fi 來傳遞數據。

我們可以使用 Broadcast Receiver 來監聽網絡連接狀態的變化。當切換為 Wi-Fi 時，我們可以進行大量的數據傳遞操作，例如下載，執行定時的更新操作，甚至是在這個時候暫時加大更新頻率。這些內容都可以在前面的課程中找到。

<!-- More -->

## 使用更大的帶寬來更不頻繁地下載更多數據

當通過無線電進行連接的時候，更大的帶寬通常伴隨著更多的電量消耗。這意味著 LTE（一種4G網絡制式）會比 3G 制式更耗電，當然比起 2G 更甚。

從 Lesson 1 我們知道了無線電狀態機是怎麼回事，通常來說相對更寬的帶寬網絡制式會有更長的狀態切換時間（也就是從 full power 過渡到 standby 有更長一段時間的延遲）。

同時，更高的帶寬意味著可以更大量的進行預取，下載更多的數據。也許這個說法不是很直觀，因為過渡的時間比較長，而過渡時間的長短我們無法控制，也就是過渡時間的電量消耗差不多是固定了。既然這樣，我們在每次傳輸會話中為了減少更新的頻率而把無線電激活的時間拉長，這樣顯的更有效率。也就是儘量一次性把事情做完，而不是斷斷續續的請求。

例如：如果 LTE 無線電的帶寬與電量消耗都是 3G 無線電的2倍，我們應該在每次會話的時候都下載4倍於 3G 的數據量，或者是差不多 10Mb（前面文章有說明 3G 一般每次下載 2Mb）。當然，下載到這麼多數據的時候，我們需要好好考慮預取本地存儲的效率並且需要經常刷新預取的緩存。

我們可以使用 connectivity manager 來判斷當前激活的無線電波，並且根據不同結果來修改預取操作。

```java
ConnectivityManager cm =
 (ConnectivityManager)getSystemService(Context.CONNECTIVITY_SERVICE);

TelephonyManager tm =
  (TelephonyManager)getSystemService(Context.TELEPHONY_SERVICE);

NetworkInfo activeNetwork = cm.getActiveNetworkInfo();

int PrefetchCacheSize = DEFAULT_PREFETCH_CACHE;

switch (activeNetwork.getType()) {
  case (ConnectivityManager.TYPE_WIFI):
    PrefetchCacheSize = MAX_PREFETCH_CACHE; break;
  case (ConnectivityManager.TYPE_MOBILE): {
    switch (tm.getNetworkType()) {
      case (TelephonyManager.NETWORK_TYPE_LTE |
            TelephonyManager.NETWORK_TYPE_HSPAP):
        PrefetchCacheSize *= 4;
        break;
      case (TelephonyManager.NETWORK_TYPE_EDGE |
            TelephonyManager.NETWORK_TYPE_GPRS):
        PrefetchCacheSize /= 2;
        break;
      default: break;
    }
    break;
  }
  default: break;
}
```

**Ps：想要最大化效率與最小化電量的消耗，需要考慮的東西太多了，通常來說，會根據 app 的功能需求來選擇有所側重，那麼前提就是需要了解到底哪些對效率的影響比較大,這有利於我們做出最優選擇。**
