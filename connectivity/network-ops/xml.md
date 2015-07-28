# 解析 XML 數據

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/basics/network-ops/xml.html>

Extensible Markup Language（XML）是一組將文檔編碼成機器可讀形式的規則，也是一種在網絡上共享數據的普遍格式。頻繁更新內容的網站，比如新聞網站或者博客，經常會提供 XML 提要（XML feed）來使得外部程序可以跟上內容的變化。下載與解析 XML 數據是網絡連接相關 app 的一個常見功能。 這一課會介紹如何解析 XML 文檔並使用它們的數據。

**示例**：[NetworkUsage.zip](http://developer.android.com/shareables/training/NetworkUsage.zip)

## 選擇一個 Parser

我們推薦 [XmlPullParser](http://developer.android.com/reference/org/xmlpull/v1/XmlPullParser.html)，它是 Android 上一個高效且可維護的解析 XML 的方法。 Android 上有這個接口的兩種實現方式：

* [KXmlParser](http://kxml.sourceforge.net/)，通過 <a href="http://developer.android.com/reference/org/xmlpull/v1/XmlPullParserFactory.html#newPullParser()">XmlPullParserFactory.newPullParser()</a> 得到。
* `ExpatPullParser`，通過 <a href="http://developer.android.com/reference/android/util/Xml.html#newPullParser()">Xml.newPullParser()</a> 得到。

兩個選擇都是比較好的。下面的示例中是通過 `Xml.newPullParser()` 得到 `ExpatPullParser`。

<a name="analyze"></a>
## 分析 Feed

解析一個 feed 的第一步是決定我們需要獲取的字段。這樣解析器便去抽取出那些需要的字段而忽視其他的字段。

下面的XML片段是章節概覽示例 app 中解析的 Feed 的片段。[StackOverflow.com](http://stackoverflow.com/) 上每一個帖子在 feed 中以包含幾個嵌套的子標籤的 `entry` 標籤的形式出現。

```xml
<?xml version="1.0" encoding="utf-8"?>
<feed xmlns="http://www.w3.org/2005/Atom" xmlns:creativeCommons="http://backend.userland.com/creativeCommonsRssModule" ...">
<title type="text">newest questions tagged android - Stack Overflow</title>
...
    <entry>
    ...
    </entry>
    <entry>
        <id>http://stackoverflow.com/q/9439999</id>
        <re:rank scheme="http://stackoverflow.com">0</re:rank>
        <title type="text">Where is my data file?</title>
        <category scheme="http://stackoverflow.com/feeds/tag?tagnames=android&sort=newest/tags" term="android"/>
        <category scheme="http://stackoverflow.com/feeds/tag?tagnames=android&sort=newest/tags" term="file"/>
        <author>
            <name>cliff2310</name>
            <uri>http://stackoverflow.com/users/1128925</uri>
        </author>
        <link rel="alternate" href="http://stackoverflow.com/questions/9439999/where-is-my-data-file" />
        <published>2012-02-25T00:30:54Z</published>
        <updated>2012-02-25T00:30:54Z</updated>
        <summary type="html">
            <p>I have an Application that requires a data file...</p>

        </summary>
    </entry>
    <entry>
    ...
    </entry>
...
</feed>
```

示例 app 從 `entry` 標籤與它的子標籤 `title`，`link` 和 `summary` 中提取數據.

## 實例化 Parser

下一步就是實例化一個 parser 並開始解析的操作。在下面的片段中，一個 parser 被初始化來處理名稱空間，並且將 [InputStream](http://developer.android.com/reference/java/io/InputStream.html) 作為輸入。它通過調用 <a href="http://developer.android.com/reference/org/xmlpull/v1/XmlPullParser.html#nextTag()">nextTag()</a> 開始解析，並調用 `readFeed()` 方法，`readFeed()` 方法會提取並處理 app 需要的數據：

```java
public class StackOverflowXmlParser {
    // We don't use namespaces
    private static final String ns = null;

    public List parse(InputStream in) throws XmlPullParserException, IOException {
        try {
            XmlPullParser parser = Xml.newPullParser();
            parser.setFeature(XmlPullParser.FEATURE_PROCESS_NAMESPACES, false);
            parser.setInput(in, null);
            parser.nextTag();
            return readFeed(parser);
        } finally {
            in.close();
        }
    }
 ...
}
```

## 讀取Feed

`readFeed()` 方法實際的工作是處理 feed 的內容。它尋找一個 "entry" 的標籤作為遞歸處理整個 feed 的起點。`readFeed()` 方法會跳過不是 `entry` 的標籤。當整個 feed 都被遞歸處理後，`readFeed()` 會返回一個從 feed 中提取的包含了 `entry` 標籤內容（包括裡面的數據成員）的 [List](http://developer.android.com/reference/java/util/List.html)。然後這個 [List](http://developer.android.com/reference/java/util/List.html) 成為 parser 的返回值。

```java
private List readFeed(XmlPullParser parser) throws XmlPullParserException, IOException {
    List entries = new ArrayList();

    parser.require(XmlPullParser.START_TAG, ns, "feed");
    while (parser.next() != XmlPullParser.END_TAG) {
        if (parser.getEventType() != XmlPullParser.START_TAG) {
            continue;
        }
        String name = parser.getName();
        // Starts by looking for the entry tag
        if (name.equals("entry")) {
            entries.add(readEntry(parser));
        } else {
            skip(parser);
        }
    }
    return entries;
}
```

## 解析 XML

解析 XML feed 的步驟如下：

1. 正如在上面[分析 Feed](#analyze) 所說的，判斷出應用中想要的標籤。這個例子抽取了 `entry` 標籤與它的內部標籤 `title`，`link` 和 `summary` 中的數據。
2. 創建下面的方法:
* 為每一個我們想要獲取的標籤創建一個 "read" 方法。例如 `readEntry()`，`readTitle()` 等等。解析器從輸入流中讀取標籤。當讀取到 `entry`，`title`，`link` 或者 `summary` 標籤時，它會為那些標籤調用相應的方法。否則，跳過這個標籤。

* 為每一個不同的標籤創建提取數據的方法，和使 parser 繼續解析下一個標籤的方法。例如：

	* 對於 `title` 和 `summary` 標籤，解析器調用 `readText()`。這個方法通過調用 `parser.getText()` 來獲取數據。

	* 對於 `link` 標籤，解析器先判斷這個 link 是否是我們想要的類型。然後再使用 `parser.getAttributeValue()` 來獲取 link 標籤的值。

	* 對於 `entry` 標籤，解析器調用 `readEntry()`。這個方法解析 entry 的內部標籤並返回一個帶有 `title`，`link` 和 `summary` 數據成員的 `Entry` 對象。

* 一個遞歸的輔助方法：`skip()`。關於這部分的討論，請看下面一部分內容：[跳過不關心的標籤](#skip)。

下面的代碼演示瞭如何解析 entries，titles，links 與 summaries。

```java
public static class Entry {
    public final String title;
    public final String link;
    public final String summary;

    private Entry(String title, String summary, String link) {
        this.title = title;
        this.summary = summary;
        this.link = link;
    }
}

// Parses the contents of an entry. If it encounters a title, summary, or link tag, hands them off
// to their respective "read" methods for processing. Otherwise, skips the tag.
private Entry readEntry(XmlPullParser parser) throws XmlPullParserException, IOException {
    parser.require(XmlPullParser.START_TAG, ns, "entry");
    String title = null;
    String summary = null;
    String link = null;
    while (parser.next() != XmlPullParser.END_TAG) {
        if (parser.getEventType() != XmlPullParser.START_TAG) {
            continue;
        }
        String name = parser.getName();
        if (name.equals("title")) {
            title = readTitle(parser);
        } else if (name.equals("summary")) {
            summary = readSummary(parser);
        } else if (name.equals("link")) {
            link = readLink(parser);
        } else {
            skip(parser);
        }
    }
    return new Entry(title, summary, link);
}

// Processes title tags in the feed.
private String readTitle(XmlPullParser parser) throws IOException, XmlPullParserException {
    parser.require(XmlPullParser.START_TAG, ns, "title");
    String title = readText(parser);
    parser.require(XmlPullParser.END_TAG, ns, "title");
    return title;
}

// Processes link tags in the feed.
private String readLink(XmlPullParser parser) throws IOException, XmlPullParserException {
    String link = "";
    parser.require(XmlPullParser.START_TAG, ns, "link");
    String tag = parser.getName();
    String relType = parser.getAttributeValue(null, "rel");
    if (tag.equals("link")) {
        if (relType.equals("alternate")){
            link = parser.getAttributeValue(null, "href");
            parser.nextTag();
        }
    }
    parser.require(XmlPullParser.END_TAG, ns, "link");
    return link;
}

// Processes summary tags in the feed.
private String readSummary(XmlPullParser parser) throws IOException, XmlPullParserException {
    parser.require(XmlPullParser.START_TAG, ns, "summary");
    String summary = readText(parser);
    parser.require(XmlPullParser.END_TAG, ns, "summary");
    return summary;
}

// For the tags title and summary, extracts their text values.
private String readText(XmlPullParser parser) throws IOException, XmlPullParserException {
    String result = "";
    if (parser.next() == XmlPullParser.TEXT) {
        result = parser.getText();
        parser.nextTag();
    }
    return result;
}
  ...
}
```

<a name="skip"></a>
## 跳過不關心的標籤

上面描述的 XML 解析步驟中有一步就是跳過不關心的標籤，下面演示解析器的 `skip()` 方法:

```java
private void skip(XmlPullParser parser) throws XmlPullParserException, IOException {
    if (parser.getEventType() != XmlPullParser.START_TAG) {
        throw new IllegalStateException();
    }
    int depth = 1;
    while (depth != 0) {
        switch (parser.next()) {
        case XmlPullParser.END_TAG:
            depth--;
            break;
        case XmlPullParser.START_TAG:
            depth++;
            break;
        }
    }
}
```

下面解釋這個方法如何工作:

* 如果當前事件不是一個 `START_TAG`，拋出異常。
* 它消耗掉 `START_TAG` 以及接下來的所有內容，包括與開始標籤配對的 `END_TAG`。
* 為了保證方法在遇到正確的 `END_TAG` 時停止，而不是在最開始的 `START_TAG` 後面的第一個標籤，方法隨時記錄嵌套深度。

因此如果目前的標籤有子標籤, 那麼直到解析器已經處理了所有位於 `START_TAG` 與對應的 `END_TAG` 之間的事件之前，`depth` 的值不會為 0。例如，看解析器如何跳過 `<author>` 標籤，它有2個子標籤，`<name>` 與 `<uri>`：

* 第一次循環, 在 `<author>` 之後 parser 遇到的第一個標籤是 `<name>` 標籤的 `START_TAG`。`depth` 值變為2。
* 第二次循環, parser 遇到的下一個標籤是 `END_TAG` `</name>`。depth 值變為1。
* 第三次循環, parser 遇到的下一個標籤是 `START_TAG` `<uri>`。depth 值變為2。
* 第四次循環, parser 遇到的下一個標籤是 `END_TAG` `</uri>`。depth 值變為1。
* 第五次同時也是最後一次循環, parser 遇到的下一個標籤是 `END_TAG` `</author>`。 depth 值變為0。表明成功跳過了 `<author>` 標籤。

## 使用 XML 數據

示例程序是在 [AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) 中獲取與解析 XML 數據的。這會在主 UI 線程之外進行處理。當處理完畢後，app 會更新 main activity（`NetworkActivity`）的 UI。

在下面示例代碼中，`loadPage()` 方法做了下面的事情：

* 初始化一個帶有 URL 地址的字符串變量，用來訂閱 XML feed。
* 如果用戶設置與網絡連接都允許，會調用 `new DownloadXmlTask().execute(url)`。這會初始化一個新的 `DownloadXmlTask` 對象（[AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) 的子類）並且開始執行它的 <a href="http://developer.android.com/reference/android/os/AsyncTask.html#execute(Params...)">execute()</a> 方法，這個方法會下載並解析 feed，並返回展示在 UI 上的字符串。

```java
public class NetworkActivity extends Activity {
    public static final String WIFI = "Wi-Fi";
    public static final String ANY = "Any";
    private static final String URL = "http://stackoverflow.com/feeds/tag?tagnames=android&sort=newest";

    // Whether there is a Wi-Fi connection.
    private static boolean wifiConnected = false;
    // Whether there is a mobile connection.
    private static boolean mobileConnected = false;
    // Whether the display should be refreshed.
    public static boolean refreshDisplay = true;
    public static String sPref = null;

    ...

    // Uses AsyncTask to download the XML feed from stackoverflow.com.
    public void loadPage() {

        if((sPref.equals(ANY)) && (wifiConnected || mobileConnected)) {
            new DownloadXmlTask().execute(URL);
        }
        else if ((sPref.equals(WIFI)) && (wifiConnected)) {
            new DownloadXmlTask().execute(URL);
        } else {
            // show error
        }
    }
```

下面展示的是 [AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) 的子類，`DownloadXmlTask`，實現了 [AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) 的如下方法：

* <a href="http://developer.android.com/reference/android/os/AsyncTask.html#doInBackground(Params...)">doInBackground()</a> 執行 `loadXmlFromNetwork()` 方法。它以 feed 的 URL 作為參數。`loadXmlFromNetwork()` 獲取並處理 feed。當它完成時，返回一個結果字符串。
* <a href="http://developer.android.com/reference/android/os/AsyncTask.html#onPostExecute(Result)">onPostExecute()</a> 接收返回的字符串並將其展示在UI上。

```java
// Implementation of AsyncTask used to download XML feed from stackoverflow.com.
private class DownloadXmlTask extends AsyncTask<String, Void, String> {
    @Override
    protected String doInBackground(String... urls) {
        try {
            return loadXmlFromNetwork(urls[0]);
        } catch (IOException e) {
            return getResources().getString(R.string.connection_error);
        } catch (XmlPullParserException e) {
            return getResources().getString(R.string.xml_error);
        }
    }

    @Override
    protected void onPostExecute(String result) {
        setContentView(R.layout.main);
        // Displays the HTML string in the UI via a WebView
        WebView myWebView = (WebView) findViewById(R.id.webview);
        myWebView.loadData(result, "text/html", null);
    }
}
```

下面是 `DownloadXmlTask` 中調用的 `loadXmlFromNetwork()` 方法做的事情：

1. 實例化一個 `StackOverflowXmlParser`。它同樣創建一個 `Entry` 對象（`entries`）的 List，和 `title`，`url`，`summary`，來保存從 XML feed 中提取的值。
2. 調用 `downloadUrl()`，它會獲取 feed, 並將其作為 [InputStream](http://developer.android.com/reference/java/io/InputStream.html) 返回。
3. 使用 `StackOverflowXmlParser` 解析 [InputStream](http://developer.android.com/reference/java/io/InputStream.html)。`StackOverflowXmlParser` 用從 feed 中獲取的數據填充 `entries` 的 List。
4. 處理 `entries` 的 List，並將 feed 數據與 HTML 標記結合起來。
5. 返回一個 HTML 字符串，[AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) 的 <a href="http://developer.android.com/reference/android/os/AsyncTask.html#onPostExecute(Result)">onPostExecute()</a> 方法會將其展示在 main activity 的 UI 上。

```java
// Uploads XML from stackoverflow.com, parses it, and combines it with
// HTML markup. Returns HTML string.【這裡可以看出應該是Download】
private String loadXmlFromNetwork(String urlString) throws XmlPullParserException, IOException {
    InputStream stream = null;
    // Instantiate the parser
    StackOverflowXmlParser stackOverflowXmlParser = new StackOverflowXmlParser();
    List<Entry> entries = null;
    String title = null;
    String url = null;
    String summary = null;
    Calendar rightNow = Calendar.getInstance();
    DateFormat formatter = new SimpleDateFormat("MMM dd h:mmaa");

    // Checks whether the user set the preference to include summary text
    SharedPreferences sharedPrefs = PreferenceManager.getDefaultSharedPreferences(this);
    boolean pref = sharedPrefs.getBoolean("summaryPref", false);

    StringBuilder htmlString = new StringBuilder();
    htmlString.append("<h3>" + getResources().getString(R.string.page_title) + "</h3>");
    htmlString.append("<em>" + getResources().getString(R.string.updated) + " " +
            formatter.format(rightNow.getTime()) + "</em>");

    try {
        stream = downloadUrl(urlString);
        entries = stackOverflowXmlParser.parse(stream);
    // Makes sure that the InputStream is closed after the app is
    // finished using it.
    } finally {
        if (stream != null) {
            stream.close();
        }
     }

    // StackOverflowXmlParser returns a List (called "entries") of Entry objects.
    // Each Entry object represents a single post in the XML feed.
    // This section processes the entries list to combine each entry with HTML markup.
    // Each entry is displayed in the UI as a link that optionally includes
    // a text summary.
    for (Entry entry : entries) {
        htmlString.append("<p><a href='");
        htmlString.append(entry.link);
        htmlString.append("'>" + entry.title + "</a></p>");
        // If the user set the preference to include summary text,
        // adds it to the display.
        if (pref) {
            htmlString.append(entry.summary);
        }
    }
    return htmlString.toString();
}

// Given a string representation of a URL, sets up a connection and gets
// an input stream.
【關於Timeout具體應該設置多少，可以借鑑這裡的數據，當然前提是一般情況下】
// Given a string representation of a URL, sets up a connection and gets
// an input stream.
private InputStream downloadUrl(String urlString) throws IOException {
    URL url = new URL(urlString);
    HttpURLConnection conn = (HttpURLConnection) url.openConnection();
    conn.setReadTimeout(10000 /* milliseconds */);
    conn.setConnectTimeout(15000 /* milliseconds */);
    conn.setRequestMethod("GET");
    conn.setDoInput(true);
    // Starts the query
    conn.connect();
    return conn.getInputStream();
}
```

