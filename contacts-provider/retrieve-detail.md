# 獲取聯繫人詳情

> 編寫:[spencer198711](https://github.com/spencer198711) - 原文:<http://developer.android.com/training/contacts-provider/retrieve-details.html>

這一課展示瞭如何取得一個聯繫人的詳細信息，比如email地址、電話號碼等。當使用者去獲取聯繫人信息的時候，這些信息正是他們所查找的。我們可以給他們關於一個聯繫人的所有信息，或者僅僅顯示一個特定的數據類型，比如email地址。

這一課假設你已經獲取到了一個用戶所選取的聯繫人的[ContactsContract.Contacts](http://developer.android.com/reference/android/provider/ContactsContract.Contacts.html)數據項。在[獲取聯繫人名字](retrieve-names.html)那一課展示瞭如何獲取聯繫人列表。

## 獲取聯繫人的所有詳細信息

為了取得一個聯繫人的所有詳情，查找[ContactsContract.Data](http://developer.android.com/reference/android/provider/ContactsContract.Data.html)表中包含聯繫人[LOOKUP_KEY](http://developer.android.com/reference/android/provider/ContactsContract.ContactsColumns.html#LOOKUP_KEY)列的任意行。因為Contacts Provider隱式地連接了ContactsContract.Contacts表和ContactsContract.Data表，所以這個[LOOKUP_KEY](http://developer.android.com/reference/android/provider/ContactsContract.ContactsColumns.html#LOOKUP_KEY)列在ContactsContract.Data表中是可用的。關於[LOOKUP_KEY](http://developer.android.com/reference/android/provider/ContactsContract.ContactsColumns.html#LOOKUP_KEY)列，在[獲取聯繫人名字](retrieve-names.html)那一課有詳細的描述。

> **Note：**檢索一個聯繫人的所有信息會降低設備的性能，因為這需要檢索ContactsContract.Data表的所有列。在使用這種方法之前，請認真考慮對性能影響。

### 請求權限

為了能夠讀Contacts Provider，我們的應用必須擁有[READ_CONTACTS](http://developer.android.com/reference/android/Manifest.permission.html#READ_CONTACTS)權限。為了請求這個權限，需要在manifest文件的<manifest\>中添加如下子節點：

```xml
<uses-permission android:name="android.permission.READ_CONTACTS" />
```

### 設置查詢映射

根據一行數據的數據類型，它可能會使用很多列或者只使用幾列。另外，數據會根據不同的數據類型而出現在不同的列中。為了確保能夠獲取所有數據類型的所有可能的數據列，需要在查詢映射中添加所有列的名字。如果要把Cursor綁定到ListView，記得要獲取Data._ID，否則的話，界面綁定就不會起作用。同時也需要獲取[Data.MIMETYPE](http://developer.android.com/reference/android/provider/ContactsContract.DataColumns.html#MIMETYPE)列，這樣才能識別我們獲取到的每一行數據的數據類型。例如：

```java
private static final String PROJECTION =
            {
                Data._ID,
                Data.MIMETYPE,
                Data.DATA1,
                Data.DATA2,
                Data.DATA3,
                Data.DATA4,
                Data.DATA5,
                Data.DATA6,
                Data.DATA7,
                Data.DATA8,
                Data.DATA9,
                Data.DATA10,
                Data.DATA11,
                Data.DATA12,
                Data.DATA13,
                Data.DATA14,
                Data.DATA15
            };
```

這個查詢映射使用了ContactsContract.Data類中定義的列名字，去獲取ContactsContract.Data表中一行的所有數據列。

我們也可以使用由ContactsContract.Data或其子類定義的列常量去設置查詢映射。需要注意的是，從SYNC1到SYNC4的數據列是sync adapter同步數據所使用的，它們的值對我們沒有意義。

### 定義查詢標準

為查詢選擇子句定義一個常量，一個包含查詢選擇參數的數組，以及一個保存查詢選擇值的變量。使用Contacts.LOOKUP_KEY列去查找這個聯繫人。例如：

```java
	// Defines the selection clause
    private static final String SELECTION = Data.LOOKUP_KEY + " = ?";
    // Defines the array to hold the search criteria
    private String[] mSelectionArgs = { "" };
    /*
     * Defines a variable to contain the selection value. Once you
     * have the Cursor from the Contacts table, and you've selected
     * the desired row, move the row's LOOKUP_KEY value into this
     * variable.
     */
    private String mLookupKey;
```

在查詢選擇表達式中使用 “?”佔位符，確保了搜索是由綁定生成而不是由SQL編譯生成。這種方法消除了惡意SQL注入的可能性。

### 定義排序順序

定義在查詢結果Cursor中希望的排序順序。按照Data.MIMETYPE去排序，可以讓特定數據類型的所有行排列在一起。這種形式的查詢排序參數讓所有具有email的行排在一起，讓所有具有電話的行排在一起……例如：

```java
	/*
     * Defines a string that specifies a sort order of MIME type
     */
    private static final String SORT_ORDER = Data.MIMETYPE;
```

> **Note：**一些數據類型不使用子類型，所以不能按照子類型來排序。作為替代方法，我們不得不遍歷返回的Cursor，去判定當前行的數據類型，為那些使用子類型的數據行保存數據。當讀取完cursor後，我們可以根據子類型去排序每一個數據類型並顯示結果。

### 初始化查詢loader

永遠在後臺線程中去檢索Contacts Provider(或者其他content provider)的數據。使用Loader框架中的LoaderManager類和LoaderManager.LoaderCallbacks在後臺去做獲取數據的工作。

當我們已經準備好去獲取數據行，需要通過調用initLoader()方法去初始化loader框架。傳遞一個Integer類型的標識符給initLoader()方法，這個標識符會傳遞給LoaderManager.LoaderCallbacks方法。當在一個應用中使用多個loader時，這個標識符能夠幫助我們區分它們。

以下的代碼片段展示瞭如何初始化loader框架：

```java
public class DetailsFragment extends Fragment implements
        LoaderManager.LoaderCallbacks<Cursor> {
    ...
    // Defines a constant that identifies the loader
    DETAILS_QUERY_ID = 0;
    ...
    /*
     * Invoked when the parent Activity is instantiated
     * and the Fragment's UI is ready. Put final initialization
     * steps here.
     */
    @Override
    onActivityCreated(Bundle savedInstanceState) {
        ...
        // Initializes the loader framework
        getLoaderManager().initLoader(DETAILS_QUERY_ID, null, this);
```

### 實現onCreateLoader()方法

實現onCreateLoader()方法。loader框架會在我們調用initLoader()方法後立即調用onCreateLoader()方法。這個方法會返回一個CursorLoader對象。由於搜索的是ContactsContract.Data表，所以需要使用常量Data.CONTENT_URI作為內容URI。例如：


```java
	@Override
    public Loader<Cursor> onCreateLoader(int loaderId, Bundle args) {
        // Choose the proper action
        switch (loaderId) {
            case DETAILS_QUERY_ID:
            // Assigns the selection parameter
            mSelectionArgs[0] = mLookupKey;
            // Starts the query
            CursorLoader mLoader =
                    new CursorLoader(
                            getActivity(),
                            Data.CONTENT_URI,
                            PROJECTION,
                            SELECTION,
                            mSelectionArgs,
                            SORT_ORDER
                    );
            ...
    }
```

### 實現onLoadFinished()方法和onLoaderReset()方法

實現onLoadFinished()方法。當Contacts Provider返回查詢結果的時候，loader框架會調用onLoadFinished()方法。例如：

```java
public void onLoadFinished(Loader<Cursor> loader, Cursor cursor) {
        switch (loader.getId()) {
            case DETAILS_QUERY_ID:
                    /*
                     * Process the resulting Cursor here.
                     */
                }
                break;
            ...
        }
    }
```

當loader框架檢測到結果集Cursor所對應的數據已經發生變化的時候，會調用onLoaderReset()方法。這時，需要通過把Cursor設置為null來移除對已經存在Cursor對象的引用。否則，loader框架就不會銷燬舊的Cursor對象，從而導致內存洩漏。例如：

```java
	@Override
    public void onLoaderReset(Loader<Cursor> loader) {
        switch (loader.getId()) {
            case DETAILS_QUERY_ID:
                /*
                 * If you have current references to the Cursor,
                 * remove them here.
                 */
                }
                break;
    }
```

## 獲取聯繫人的特定類型的信息

獲取聯繫人的特定類型的信息，例如所有的email信息，跟獲取聯繫人的所有詳細信息類似。下面的內容是在[獲取聯繫人的所有詳細信息]()列出的代碼的基礎上作出的修改：

查詢映射

修改查詢映射使得能夠針對特定的數據類型去獲取列。同時需要修改查詢映射，來把在ContactsContract.CommonDataKinds子類中定義的列常量與數據類型對應起來。

查詢選擇

修改查詢選擇子句去搜索特定類型的MIMETYPE值。

排序順序

由於僅僅搜索一種類型的詳細數據，所以不需要將返回的Cursor按照Data.MIMETYPE進行分組。

這些修改將會在下面的小節中詳細描述。

### 設置查詢映射

使用ContactsContract.CommonDataKinds的特定類型子類所定義的列名稱常量，定義我們想要獲取的數據列。如果我們打算把Cursor綁定到ListView，確保要獲取`_ID`列。例如，為了獲取email數據，需要定義以下數據映射：

```java
private static final String[] PROJECTION =
            {
                Email._ID,
                Email.ADDRESS,
                Email.TYPE,
                Email.LABEL
            };
```

需要注意的是，這個查詢映射使用在ContactsContract.CommonDataKinds.Email類中定義的列名稱，來替代ContactsContract.Data類中定義的列名稱。使用email類型的列名稱使得代碼更具可讀性。

在查詢映射中，我們也可以使用ContactsContract.CommonDataKinds子類所定義的其他數據列。

### 定義查詢標準

根據我們想要找的特定聯繫人的LOOKUP_KEY和聯繫人詳細信息的Data.MIMETYPE定義一個搜索表達式，去獲取數據。把MIMETYPE的值從頭到尾用單引號括住，否則的話，content provider將會把這個常量當成變量名而不是字符串。因為我們使用的是常量，而不是用戶提供的值，所以這裡不需要使用佔位符。例如：

```java
/*
     * Defines the selection clause. Search for a lookup key
     * and the Email MIME type
     */
    private static final String SELECTION =
            Data.LOOKUP_KEY + " = ?" +
            " AND " +
            Data.MIMETYPE + " = " +
            "'" + Email.CONTENT_ITEM_TYPE + "'";
    // Defines the array to hold the search criteria
    private String[] mSelectionArgs = { "" };
```


### 定義排序規則

為查詢返回的[Cursor](http://developer.android.com/reference/android/database/Cursor.html)定義一個排序規則。由於是檢索特定的數據類型，刪除根據[MIMETYPE](http://developer.android.com/reference/android/provider/ContactsContract.DataColumns.html#MIMETYPE)來排序的部分。而如果查詢的詳細數據類型包含子類型，可以根據這個子類型去排序。例如，對於email數據，我們可以根據[Email.TYPE](http://developer.android.com/reference/android/provider/ContactsContract.CommonDataKinds.CommonColumns.html#TYPE)排序：

```java
private static final String SORT_ORDER = Email.TYPE + " ASC ";
```





