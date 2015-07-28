# 使得你的App內容可被Google搜索

> 編寫:[Lin-H](https://github.com/Lin-H) - 原文:<http://developer.android.com/training/app-indexing/index.html>

隨著移動app變得越來越普遍，用戶不僅僅從網站上查找相關信息，也在他們安裝的app上查找。你可以使Google能夠抓取你的app內容，當內容與你自己的網頁一致時，Google搜索的結果會將你的app作為結果展示給用戶。

通過為你的activity提供intent filter，可以使Google搜索展示你的app中特定的內容。Google搜索應用索引(Google Search app indexing)通過在用戶搜索結果的網頁鏈接旁附上相關的app內容鏈接，補充了這一功能。使用移動設備的用戶可以在他們的搜索結果中點擊鏈接來打開你的app，使他們能夠直接瀏覽你的app中的內容，而不需要打開網頁。

要啟用Google搜索應用索引，你需要把有關app與網頁之間聯繫的信息提供給Google。這個過程包括下面幾個步驟:

1. 通過在你的app manifest中添加intent filter來開啟鏈接到你的app中指定內容的深度鏈接。

2. 在你的網站中的相關頁面或Sitemap文件中為這些鏈接添加註解。

3. 選擇允許谷歌爬蟲(Googlebot)在Google Play store中通過APK抓取，建立app內容索引。在早期採用者計劃(early adopter program)中作為參與者加入時，會自動選擇允許。

這節課程，會向你展示如何啟用深度鏈接和建立應用內容索引，使用戶可以從移動設備搜索結果直接打開此內容。

##Lessons

* [為App內容開啟深度鏈接](deep-linking.md)

  演示如何添加intent filter來啟用鏈接app內容的深度鏈接


* [為索引指定App內容](enable-app-indexing.md)

  演示如何給網站的metadata添加註解，使Google的算法能為app內容建立索引
