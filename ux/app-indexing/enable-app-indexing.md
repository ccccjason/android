# 為索引指定App內容

> 編寫:[Lin-H](https://github.com/Lin-H) - 原文: <http://developer.android.com/training/app-indexing/enabling-app-indexing.html>

Google的網頁爬蟲機器([Googlebot](https://support.google.com/webmasters/answer/182072?hl=en))會抓取頁面，併為Google搜索引擎建立索引，也能為你的Android app內容建立索引。通過選擇加入這一功能，你可以允許Googlebot通過抓取在Google Play Store中的APK內容，為你的app內容建立索引。要指出哪些app內容你想被Google索引，只需要添加鏈接元素到現有的[Sitemap](https://support.google.com/webmasters/answer/156184?hl=en)文件，或添加到你的網站中每個頁面的`<head>`元素中，以相同的方式為你的頁面添加。

你所共享給Google搜索的深度鏈接必須按照下面的URI格式:

```
android-app://<package_name>/<scheme>/<host_path>
```

構成URI的各部分是:

* **package_name** 代表在[Google Play Developer Console](https://play.google.com/apps/publish)中所列出來的你的APK的包名。

* **scheme** 匹配你的intent filter的URI方案。

* **host_path** 找出你的應用中所指定的內容。

下面的幾節敘述如何添加一個深度鏈接URI到你的Sitemap或網頁中。

##添加深度鏈接(Deep link)到你的Sitemap

要在你的[Sitemap](https://support.google.com/webmasters/answer/156184?hl=en)中為Google搜索app索引(Google Search app indexing)添加深度鏈接的註解，使用`<xhtml:link>`標籤，並指定用作替代URI的深度鏈接。

例如，下面一段XML代碼向你展示如何使用`<loc>`標籤指定一個鏈接到你的頁面的鏈接，以及如何使用`<xhtml:link>`標籤指定鏈接到你的Android app的深度鏈接。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<urlset
    xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
    xmlns:xhtml="http://www.w3.org/1999/xhtml">
    <url>
        <loc>example://gizmos</loc>
            <xhtml:link
                rel="alternate"
                href="android-app://com.example.android/example/gizmos" />
    </url>
    ...
</urlset>
```

##添加深度鏈接到你的網頁中

除了在你的Sitemap文件中，為Google搜索app索引指定深度鏈接外，你還可以在你的HTML標記網頁中給深度鏈接添加註解。你可以在`<head>`標籤內這麼做，為每一個頁面添加一個`<link>`標籤，並指定用作替代URI的深度鏈接。

例如，下面的一段HTML代碼向你展示如何在頁面中指定一個URL為`example://gizmos`的相應的深度鏈接。

```html
<html>
<head>
    <link rel="alternate"
          href="android-app://com.example.android/example/gizmos" />
    ...
</head>
<body> ... </body>
```

##允許Google通過你的app抓取URL請求

一般來說，你可以通過使用[robots.txt](https://developers.google.com/webmasters/control-crawl-index/docs/robots_txt)文件，來控制Googlebot如何抓取你網站上的公開訪問的URL。當Googlebot為你的app內容建立索引後，你的app可以把HTTP請求當做一般操作。但是，這些請求會被視為從Googlebot發出，發送到你的服務器上。因此，你必須正確配置你的服務器上的`robots.txt`文件來允許這些請求。

例如，下面的`robots.txt`指示向你展示，如何允許你網站上的特定目錄(如 `/api/` )能被你的app訪問，並限制Googlebot訪問你的網站上的其他目錄。

```
User-Agent: Googlebot
Allow: /api/
Disallow: /
```

學習更多關於如何修改`robots.txt`，來控制頁面抓取，詳見[Controlling Crawling and Indexing Getting Started](https://developers.google.com/webmasters/control-crawl-index/docs/getting_started)。
