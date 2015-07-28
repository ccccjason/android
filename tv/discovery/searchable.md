<!--# Making TV Apps Searchable #-->
# 使TV應用是可被搜索的

> 編寫:[awong1900](https://github.com/awong1900) - 原文:http://developer.android.com/training/tv/discovery/searchable.html

<!--Android TV uses the Android search interface to retrieve content data from installed apps and deliver search results to the user. Your app's content data can be included with these results, to give the user instant access to the content in your app.-->

Android TV使用Android[搜索接口](http://developer.android.com/guide/topics/search/index.html)從安裝的應用中檢索內容數據並且釋放搜索結果給用戶。我們的應用內容數據能被包含在這些結果中，去給用戶即時訪問應用程序中的內容。

<!--Your app must provide Android TV with the data fields from which it generates suggested search results as the user enters characters in the search dialog. To do that, your app must implement a Content Provider that serves up the suggestions along with a searchable.xml configuration file that describes the content provider and other vital information for Android TV. You also need an activity that handles the intent that fires when the user selects a suggested search result. All of this is described in more detail in Adding Custom Suggestions. Here are described the main points for Android TV apps.-->

我們的應用必須提供Android TV數據字段，它是用戶在搜索框中輸入字符生成的建議搜索結果。去做這個，我們的應用必須實現[Content Provider](http://developer.android.com/guide/topics/providers/content-providers.html)，在[searchable.xml](http://developer.android.com/guide/topics/search/searchable-config.html)配置文件描述content provider和其他必要的Android TV信息。我們也需要一個activity在用戶選擇一個建議的搜索結果時處理intent的觸發。所有的這些被描述在[Adding Custom Suggestions](http://developer.android.com/guide/topics/search/adding-custom-suggestions.html)。本文描述Android TV應用搜索的關鍵點。

<!--This lesson builds on your knowledge of using search in Android to show you how to make your app searchable in Android TV. Be sure you are familiar with the concepts explained in the Search API guide before following this lesson. See also the training Adding Search Functionality.-->

這節課展示Android中搜索的知識，展示如何使我們的應用在Android TV裡是可被搜索的。確信我們熟悉[Search API guide](http://developer.android.com/guide/topics/search/index.html)的解釋。在下面的這節課程之前，查看[Adding Search Functionality](http://developer.android.com/training/search/index.html)訓練課程。

<!--This discussion describes some code from the Android Leanback sample app, available on GitHub.-->
這個討論描述的一些代碼，從[Android Leanback示例代碼](https://github.com/googlesamples/androidtv-Leanback)摘出。代碼可以在Github上找到。

<!--## Identify Columns ##-->
## 識別列

<!--The SearchManager describes the data fields it expects by representing them as columns of an SQLite database. Regardless of your data's format, you must map your data fields to these columns, usually in the class that accessess your content data. For information about building a class that maps your existing data to the required fields, see Building a suggestion table.-->

[SearchManager](http://developer.android.com/reference/android/app/SearchManager.html)描述了數據字段，它被代表為SQLite數據庫的列。不管我們的數據格式，我們必須把我們的數據字段填到那些列，通常用存取我們的內容數據的類。更多信息，查看[Building a suggestion table()](http://developer.android.com/guide/topics/search/adding-custom-suggestions.html#SuggestionTable)。

<!--The SearchManager class includes several columns for Android TV. Some of the more important columns are described below.-->
SearchManager類為AndroidTV包含了幾個列。下面是重要的一些列：

值								    |	描述
:-----------------------------------|:--------------------------------
`SUGGEST_COLUMN_TEXT_1`				|內容名字 **(必須)**
`SUGGEST_COLUMN_TEXT_2`				|內容的文本描述
`SUGGEST_COLUMN_RESULT_CARD_IMAGE`|圖片/封面
`SUGGEST_COLUMN_CONTENT_TYPE`		|媒體的MIME類型 **(必須)**
`SUGGEST_COLUMN_VIDEO_WIDTH`		|媒體的分辨率寬度     
`SUGGEST_COLUMN_VIDEO_HEIGHT`		|媒體的分辨率高度 
`SUGGEST_COLUMN_PRODUCTION_YEAR`	|內容的產品年份 **(必須)**
`SUGGEST_COLUMN_DURATION`			|媒體的時間長度

<!--The search framework requires the following columns:-->
搜索framework需要以下的列：

- [SUGGEST_COLUMN_TEXT_1](http://developer.android.com/reference/android/app/SearchManager.html#SUGGEST_COLUMN_TEXT_1)
- [SUGGEST_COLUMN_CONTENT_TYPE](http://developer.android.com/reference/android/app/SearchManager.html#SUGGEST_COLUMN_CONTENT_TYPE)
- [SUGGEST_COLUMN_PRODUCTION_YEAR](http://developer.android.com/reference/android/app/SearchManager.html#SUGGEST_COLUMN_PRODUCTION_YEAR)

<!--When the values of these columns for your content match the values for the same content from other providers found by Google servers, the system provides a deep link to your app in the details view for the content, along with links to the apps of other providers. This is discussed more in Display Content in the Details Screen, below.-->

當這些內容的列的值匹配Google服務的providers提供的的值時，系統提供一個[深鏈接](http://developer.android.com/training/app-indexing/deep-linking.html)到我們的應用，用於詳情查看，以及指向應用的其他Providers的鏈接。更多討論在[在詳情頁顯示內容](http://developer.android.com/training/tv/discovery/searchable.html#details)。

<!--Your application's database class might define the columns as follows:-->
我們的應用的數據庫類可能定義以下的列：

```java
public class VideoDatabase {
  //The columns we'll include in the video database table
  public static final String KEY_NAME = SearchManager.SUGGEST_COLUMN_TEXT_1;
  public static final String KEY_DESCRIPTION = SearchManager.SUGGEST_COLUMN_TEXT_2;
  public static final String KEY_ICON = SearchManager.SUGGEST_COLUMN_RESULT_CARD_IMAGE;
  public static final String KEY_DATA_TYPE = SearchManager.SUGGEST_COLUMN_CONTENT_TYPE;
  public static final String KEY_IS_LIVE = SearchManager.SUGGEST_COLUMN_IS_LIVE;
  public static final String KEY_VIDEO_WIDTH = SearchManager.SUGGEST_COLUMN_VIDEO_WIDTH;
  public static final String KEY_VIDEO_HEIGHT = SearchManager.SUGGEST_COLUMN_VIDEO_HEIGHT;
  public static final String KEY_AUDIO_CHANNEL_CONFIG =
          SearchManager.SUGGEST_COLUMN_AUDIO_CHANNEL_CONFIG;
  public static final String KEY_PURCHASE_PRICE = SearchManager.SUGGEST_COLUMN_PURCHASE_PRICE;
  public static final String KEY_RENTAL_PRICE = SearchManager.SUGGEST_COLUMN_RENTAL_PRICE;
  public static final String KEY_RATING_STYLE = SearchManager.SUGGEST_COLUMN_RATING_STYLE;
  public static final String KEY_RATING_SCORE = SearchManager.SUGGEST_COLUMN_RATING_SCORE;
  public static final String KEY_PRODUCTION_YEAR = SearchManager.SUGGEST_COLUMN_PRODUCTION_YEAR;
  public static final String KEY_COLUMN_DURATION = SearchManager.SUGGEST_COLUMN_DURATION;
  public static final String KEY_ACTION = SearchManager.SUGGEST_COLUMN_INTENT_ACTION;
...
```

<!--When you build the map from the SearchManager columns to your data fields, you must also specify the _ID to give each row a unique ID.-->
當我們創建從[SearchManager](http://developer.android.com/reference/android/app/SearchManager.html)列填充到我們的數據字段時，我們也必須定義[_ID](http://developer.android.com/reference/android/provider/BaseColumns.html#_ID)去獲得每行的獨一無二的ID。


```java
...
  private static HashMap buildColumnMap() {
    HashMap map = new HashMap();
    map.put(KEY_NAME, KEY_NAME);
    map.put(KEY_DESCRIPTION, KEY_DESCRIPTION);
    map.put(KEY_ICON, KEY_ICON);
    map.put(KEY_DATA_TYPE, KEY_DATA_TYPE);
    map.put(KEY_IS_LIVE, KEY_IS_LIVE);
    map.put(KEY_VIDEO_WIDTH, KEY_VIDEO_WIDTH);
    map.put(KEY_VIDEO_HEIGHT, KEY_VIDEO_HEIGHT);
    map.put(KEY_AUDIO_CHANNEL_CONFIG, KEY_AUDIO_CHANNEL_CONFIG);
    map.put(KEY_PURCHASE_PRICE, KEY_PURCHASE_PRICE);
    map.put(KEY_RENTAL_PRICE, KEY_RENTAL_PRICE);
    map.put(KEY_RATING_STYLE, KEY_RATING_STYLE);
    map.put(KEY_RATING_SCORE, KEY_RATING_SCORE);
    map.put(KEY_PRODUCTION_YEAR, KEY_PRODUCTION_YEAR);
    map.put(KEY_COLUMN_DURATION, KEY_COLUMN_DURATION);
    map.put(KEY_ACTION, KEY_ACTION);
    map.put(BaseColumns._ID, "rowid AS " +
            BaseColumns._ID);
    map.put(SearchManager.SUGGEST_COLUMN_INTENT_DATA_ID, "rowid AS " +
            SearchManager.SUGGEST_COLUMN_INTENT_DATA_ID);
    map.put(SearchManager.SUGGEST_COLUMN_SHORTCUT_ID, "rowid AS " +
            SearchManager.SUGGEST_COLUMN_SHORTCUT_ID);
    return map;
  }
...
```

<!--In the example above, notice the mapping to the SUGGEST_COLUMN_INTENT_DATA_ID field. This is the portion of the URI that points to the content unique to the data in this row — that is, the last part of the URI describing where the content is stored. The first part of the URI, when it is common to all of the rows in the table, is set in the searchable.xml file as the android:searchSuggestIntentData attribute, as described in Handle Search Suggestions, below.-->

在上面的例子中，注意填充[SUGGEST_COLUMN_INTENT_DATA_ID](http://developer.android.com/reference/android/app/SearchManager.html#SUGGEST_COLUMN_INTENT_DATA_ID)字段。這是URI的一部分，指向獨一無二的內容到這一列的數據，那是URI描述的內容被存儲的最後部分。在URI的第一部分，與所有表格的列同樣，是設置[在searchable.xml](http://developer.android.com/guide/topics/search/searchable-config.html)文件，用[android:searchSuggestIntentData](http://developer.android.com/guide/topics/search/searchable-config.html#searchSuggestIntentData)屬性。屬性被描述在[Handle Search Suggestions](http://developer.android.com/training/tv/discovery/searchable.html#suggestions)。

<!--If the first part of the URI is different for each row in the table, you map that value with the SUGGEST_COLUMN_INTENT_DATA field. When the user selects this content, the intent that fires provides the intent data from the combination of the SUGGEST_COLUMN_INTENT_DATA_ID and either the android:searchSuggestIntentData attribute or the SUGGEST_COLUMN_INTENT_DATA field value.-->

如果URI的第一部分是不同於表格的每一列，我們填充[SUGGEST_COLUMN_INTENT_DATA](http://developer.android.com/reference/android/app/SearchManager.html#SUGGEST_COLUMN_INTENT_DATA)字段的值。當用戶選擇這個內容時，這個intent被啟動依據[SUGGEST_COLUMN_INTENT_DATA_ID](http://developer.android.com/reference/android/app/SearchManager.html#SUGGEST_COLUMN_INTENT_DATA_ID)的混合intent數據或者`android:searchSuggestIntentData`屬性和[SUGGEST_COLUMN_INTENT_DATA](http://developer.android.com/reference/android/app/SearchManager.html#SUGGEST_COLUMN_INTENT_DATA)字段值之一。

<!--### Provide Search Suggestion Data-->
### 提供搜索建議數據

<!--Implement a Content Provider to return search term suggestions to the Android TV search dialog. The system queries your content provider for suggestions by calling the query() method each time a letter is typed. In your implementation of query(), your content provider searches your suggestion data and returns a Cursor that points to the rows you have designated for suggestions.-->

實現一個[Content Provider](http://developer.android.com/guide/topics/providers/content-providers.html)去返回搜索術語建議到AndroidTV搜索框。系統需要我們的內容容器提供建議，通過調用每次一個字母類型[query()](http://developer.android.com/reference/android/content/ContentProvider.html#query(android.net.Uri, java.lang.String[], java.lang.String, java.lang.String[], java.lang.String))方法。在[query()](http://developer.android.com/reference/android/content/ContentProvider.html#query(android.net.Uri, java.lang.String[], java.lang.String, java.lang.String[], java.lang.String))的實現中，我們的內容容器搜索我們的建議數據並且返回一個光標指向我們已經指定的建議列。

```java
@Override
  public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs,
                      String sortOrder) {
    // Use the UriMatcher to see what kind of query we have and format the db query accordingly
    switch (URI_MATCHER.match(uri)) {
      case SEARCH_SUGGEST:
          Log.d(TAG, "search suggest: " + selectionArgs[0] + " URI: " + uri);
          if (selectionArgs == null) {
              throw new IllegalArgumentException(
                      "selectionArgs must be provided for the Uri: " + uri);
          }
          return getSuggestions(selectionArgs[0]);
      default:
          throw new IllegalArgumentException("Unknown Uri: " + uri);
    }
  }

  private Cursor getSuggestions(String query) {
    query = query.toLowerCase();
    String[] columns = new String[]{
      BaseColumns._ID,
      VideoDatabase.KEY_NAME,
      VideoDatabase.KEY_DESCRIPTION,
      VideoDatabase.KEY_ICON,
      VideoDatabase.KEY_DATA_TYPE,
      VideoDatabase.KEY_IS_LIVE,
      VideoDatabase.KEY_VIDEO_WIDTH,
      VideoDatabase.KEY_VIDEO_HEIGHT,
      VideoDatabase.KEY_AUDIO_CHANNEL_CONFIG,
      VideoDatabase.KEY_PURCHASE_PRICE,
      VideoDatabase.KEY_RENTAL_PRICE,
      VideoDatabase.KEY_RATING_STYLE,
      VideoDatabase.KEY_RATING_SCORE,
      VideoDatabase.KEY_PRODUCTION_YEAR,
      VideoDatabase.KEY_COLUMN_DURATION,
      VideoDatabase.KEY_ACTION,
      SearchManager.SUGGEST_COLUMN_INTENT_DATA_ID
    };
    return mVideoDatabase.getWordMatch(query, columns);
  }
...
```

<!--In your manifest file, the content provider receives special treatment. Rather than getting tagged as an activity, it is described as a <provider>. The provider includes the android:searchSuggestAuthority attribute to tell the system the namespace of your content provider. Also, you must set its android:exported attribute to "true" so that the Android global search can use the results returned from it.-->

在我們的manifest文件中，內容容器接受特殊處理。相比被標記為一個activity，它是被描述為<[provider](http://developer.android.com/guide/topics/manifest/provider-element.html)>。provider包括`android:searchSuggestAuthority`屬性去告訴系統我們的內容容器的名字空間。並且，我們必須設置它的`android:exported`屬性為`"true"`，這樣Android全局搜索能用它返回的搜索結果。

```xml
<provider android:name="com.example.android.tvleanback.VideoContentProvider"
    android:authorities="com.example.android.tvleanback"
    android:exported="true" />
```

<!--## Handle Search Suggestions ##-->
## 處理搜索建議

<!--Your app must include a res/xml/searchable.xml file to configure the search suggestions settings. It inlcudes the android:searchSuggestAuthority attribute to tell the system the namespace of your content provider. This must match the string value you specify in the android:authorities attribute of the <provider> element in your AndroidManifest.xml file.-->

我們的應用必須包括[res/xml/searchable.xml](http://developer.android.com/guide/topics/search/searchable-config.html)文件去配置搜索建議設置。它包括[android:searchSuggestAuthority](http://developer.android.com/guide/topics/search/searchable-config.html#searchSuggestAuthority)屬性去告訴系統內容容器的名字空間。這必須匹配在`AndroidManifest.xml`文件的<[provider](http://developer.android.com/guide/topics/manifest/provider-element.html)>元素的[android:authorities](http://developer.android.com/guide/topics/manifest/provider-element.html#auth) 屬性的字符串值。

<!--The searchable.xml file must also include the android:searchSuggestIntentAction with the value "android.intent.action.VIEW" to define the intent action for providing a custom suggestion. This is different from the intent action for providing a search term, explained below. See also, Declaring the intent action for other ways to declare the intent action for suggestions.-->

[searchable.xml](http://developer.android.com/guide/topics/search/searchable-config.html)文件必須也包含在`"android.intent.action.VIEW"`的[android:searchSuggestIntentAction](http://developer.android.com/guide/topics/search/searchable-config.html#searchSuggestIntentAction)值去定義提供自定義建議的intent action。這與提供一個搜索術語的intent action不同，下面解釋。查看[Declaring the intent action](http://developer.android.com/guide/topics/search/adding-custom-suggestions.html#IntentAction) 用另一種方式去定義建議的intent action。

<!--Along with the intent action, your app must provide the intent data, which you specify with the android:searchSuggestIntentData attribute. This is the first part of the URI that points to the content. It describes the portion of the URI common to all rows in the mapping table for that content. The portion of the URI that is unique to each row is established with the SUGGEST_COLUMN_INTENT_DATA_ID field, as described above in Identify Columns. See also, Declaring the intent data for other ways to declare the intent data for suggestions.-->

同intent action一起，我們的應用必須提供我們定義的[android:searchSuggestIntentData](http://developer.android.com/guide/topics/search/searchable-config.html#searchSuggestIntentData)屬性的intent數據。這是指向內容的URI的第一部分。它描述在填充的內容表格中URI所有共同列的部分。URI的獨一無二的部分用 [SUGGEST_COLUMN_INTENT_DATA_ID](http://developer.android.com/reference/android/app/SearchManager.html#SUGGEST_COLUMN_INTENT_DATA_ID)字段建立每一列，以上被描述在[識別列](http://developer.android.com/training/tv/discovery/searchable.html#columns)。查看[Declaring the intent data](http://developer.android.com/guide/topics/search/adding-custom-suggestions.html#IntentData)用另一種方式去定義建議的intent數據。

<!--Also, note the android:searchSuggestSelection=" ?" attribute which specifies the value passed as the selection parameter of the query() method where the question mark (?) value is replaced with the query text.-->

並且，注意`android:searchSuggestSelection="?"`屬性為特定的值。這個值作為[query()](http://developer.android.com/reference/android/content/ContentProvider.html#query(android.net.Uri, java.lang.String[], java.lang.String, java.lang.String[], java.lang.String))方法`selection`參數。方法的問題標記(?)值被代替為請求文本。

<!--Finally, you must also include the android:includeInGlobalSearch attribute with the value "true". Here is an example searchable.xml file:-->

最後，我們也必須包含[android:includeInGlobalSearch](http://developer.android.com/guide/topics/search/searchable-config.html#includeInGlobalSearch)屬性值為`"true"`。這是一個[searchable.xml](http://developer.android.com/guide/topics/search/searchable-config.html)文件的例子：
```
<searchable xmlns:android="http://schemas.android.com/apk/res/android"
    android:label="@string/search_label"
        android:hint="@string/search_hint"
        android:searchSettingsDescription="@string/settings_description"
        android:searchSuggestAuthority="com.example.android.tvleanback"
        android:searchSuggestIntentAction="android.intent.action.VIEW"
        android:searchSuggestIntentData="content://com.example.android.tvleanback/video_database_leanback"
        android:searchSuggestSelection=" ?"
        android:searchSuggestThreshold="1"
        android:includeInGlobalSearch="true"
    >
</searchable>
```

<!--## Handle Search Terms ##-->
## 處理搜索術語

<!--As soon as the search dialog has a word which matches the value in one of your app's columns (described in Identifying Columns, above), the system fires the ACTION_SEARCH intent. The activity in your app which handles that intent searches the repository for columns with the given word in their values, and returns a list of content items with those columns. In your AndroidManifest.xml file, you designate the activity which handles the ACTION_SEARCH intent like this:-->

一旦搜索框有一個字匹配到了應用列中的一個（被描述在上文的[識別列](http://developer.android.com/training/tv/discovery/searchable.html#identifying)），系統啟動[ACTION_SEARCH](http://developer.android.com/reference/android/content/Intent.html#ACTION_SEARCH) intent。我們應用的activity處理intent搜索列的給定的字段資源，並且返回一個那些內容項的列表。在我們的`AndroidManifest.xml`文件中，我們指定的activity處理[ACTION_SEARCH](http://developer.android.com/reference/android/content/Intent.html#ACTION_SEARCH) intent，像這樣：

```xml
...
  <activity
      android:name="com.example.android.tvleanback.DetailsActivity"
      android:exported="true">

      <!-- Receives the search request. -->
      <intent-filter>
          <action android:name="android.intent.action.SEARCH" />
          <!-- No category needed, because the Intent will specify this class component -->
      </intent-filter>

      <!-- Points to searchable meta data. -->
      <meta-data android:name="android.app.searchable"
          android:resource="@xml/searchable" />
  </activity>
...
  <!-- Provides search suggestions for keywords against video meta data. -->
  <provider android:name="com.example.android.tvleanback.VideoContentProvider"
      android:authorities="com.example.android.tvleanback"
      android:exported="true" />
...
```

<!--The activity must also describe the searchable configuration with a reference to the searchable.xml file. To use the global search dialog, the manifest must describe which activity should receive search queries. The manifest must also describe the <provider> element, exactly as it is described in the searchable.xml file.-->

activity必須參考[searchable.xml](http://developer.android.com/guide/topics/search/searchable-config.html)文件描述可搜索的設置。用[全局搜索框](http://developer.android.com/guide/topics/search/search-dialog.html)，manifest必須描述activity應該收到的搜索請求。manifest必須描述<[provider](http://developer.android.com/guide/topics/manifest/provider-element.html)>元素，詳細被描述在[searchable.xml](http://developer.android.com/guide/topics/search/searchable-config.html)文件。

<!--## Deep Link to Your App in the Details Screen ##-->
## 深鏈接到應用的詳情頁

<!--If you have set up the search configuration as described in Handle Search Suggestions and mapped the SUGGEST_COLUMN_TEXT_1, SUGGEST_COLUMN_CONTENT_TYPE, and SUGGEST_COLUMN_PRODUCTION_YEAR fields as described in Identify Columns, a deep link to a watch action for your content appears in the details screen that launches when the user selects a search result, as shown in figure 1.-->

如果我們有設置[處理搜索建議](http://developer.android.com/training/tv/discovery/searchable.html#suggestions)描述的搜索配置和填充 [SUGGEST_COLUMN_TEXT_1](http://developer.android.com/reference/android/app/SearchManager.html#SUGGEST_COLUMN_TEXT_1)，[SUGGEST_COLUMN_CONTENT_TYPE](http://developer.android.com/reference/android/app/SearchManager.html#SUGGEST_COLUMN_CONTENT_TYPE)和[SUGGEST_COLUMN_PRODUCTION_YEAR](http://developer.android.com/reference/android/app/SearchManager.html#SUGGEST_COLUMN_PRODUCTION_YEAR)字段到[識別列](http://developer.android.com/training/tv/discovery/searchable.html#columns)，一個[深鏈接](http://developer.android.com/training/app-indexing/deep-linking.html)去查看詳情頁的內容。當用戶選擇一個搜索結果時，詳情頁將打開。如圖1。

![deep-link](deep-link.png)  
<!--**Figure 1.** The details screen displays a deep link for the Videos by Google (Leanback) sample app. Sintel: © copyright Blender Foundation, www.sintel.org.-->
**圖1** 詳情頁顯示一個深鏈接為Google(Leanback)的視頻代碼。Sintel: © copyright Blender Foundation, www.sintel.org.

<!--When the user selects the link for your app, identified by the "Available On" button in the details screen, the system launches the activity which handles the ACTION_VIEW (set as android:searchSuggestIntentAction with the value "android.intent.action.VIEW" in the searchable.xml file).-->

當用戶選擇我們的應用鏈接，`“Available On”`按鈕被標識在詳情頁，系統啟動activity處理[ACTION_VIEW](http://developer.android.com/reference/android/content/Intent.html#ACTION_VIEW)（在[searchable.xml](http://developer.android.com/guide/topics/search/searchable-config.html#searchSuggestIntentAction)文件設置[android:searchSuggestIntentAction](http://developer.android.com/guide/topics/search/searchable-config.html#searchSuggestIntentAction)值為`"android.intent.action.VIEW"`）。

<!--You can also set up a custom intent to launch your activity, and this is demonstrated in the Android Leanback sample app. Note that the sample app launches its own LeanbackDetailsFragment to show the details for the selected media, but you should launch the activity that plays the media immediately to save the user another click or two.-->

我們也能設置用戶intent去啟動我們的activity，這個在[在AndroidLeanback示例代碼應用](https://github.com/googlesamples/androidtv-Leanback)中演示。注意示例應用啟動它自己的`LeanbackDetailsFragment`去顯示被選擇媒體的詳情，但是我們應該啟動activity去播放媒體。立即去保存用戶的另一次或兩次點擊。

----------------
[下一節: 使TV應用是可被搜索的 >](in-app-search.html)
