# 獲取聯繫人列表

> 編寫:[spencer198711](https://github.com/spencer198711) - 原文:<http://developer.android.com/training/contacts-provider/retrieve-names.html>

這一課展示瞭如何根據要搜索的字符串去匹配聯繫人的數據，從而得到聯繫人列表，你可以使用以下方法去實現：

**匹配聯繫人名字**

通過搜索字符串來匹配聯繫人名字的全部或者部分來獲得聯繫人列表。因為Contacts Provider允許多個實例擁有相同的名字，所以這種方法能夠返回匹配的列表。

**匹配特定的數據類型，比如電話號碼**

通過搜索字符串來匹配聯繫人的某一特定數據類型（如電子郵件地址），來取得符合要求的聯繫人列表。例如，這種方法可以列出電子郵件地址與搜索字符相匹配的所有聯繫人。

**匹配任意類型的數據**

通過搜索字符串來匹配聯繫人詳情的所有數據類型，包括名字、電話號碼、地址、電子郵件地址等等。例如，這種方法接受任意數據類型的搜索字符串，並列出與這個搜索字符串相匹配的聯繫人。

> **Note：**這一課的所有例子都使用CursorLoader獲取Contacts Provider中的數據。CursorLoader在一個與UI線程相獨立的工作線程進行查詢操作。這保證了數據查詢不會降低UI響應的時間，以免引起槽糕的用戶體驗。更多信息，請參照[在後臺加載數據](http://hukai.me/android-training-course-in-chinese/background-jobs/load-data-background/index.html)。

## 請求讀取聯繫人的權限

為了能夠在Contacts Provider中做任意類型的搜索，我們的應用必須擁有READ_CONTACTS權限。為了擁有這個權限，我們需要在項目的manifest文件的<manifest\>節點中添加<uses-permission>子結點，如下：

```xml
<uses-permission android:name="android.permission.READ_CONTACTS" />
```

## 根據名字取得聯繫人並列出結果

這種方法根據搜索字符串，去匹配Contacts Provider的ContactsContract.Contacts表中的聯繫人名字。通常希望在ListView中展示結果，去讓用戶在所有匹配的聯繫人中做選擇。

### 定義ListView和列表項的佈局

為了能夠將搜索結果展示在列表中，我們需要一個包含ListView以及其他佈局控件的主佈局文件，和一個定義列表中每一項的佈局文件。例如，可以使用以下XML代碼去創建主佈局文件res/layout/contacts\_list\_view.xml：

```xml
<?xml version="1.0" encoding="utf-8"?>
<ListView xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@android:id/list"
      android:layout_width="match_parent"
      android:layout_height="match_parent"/>
```

這個XML代碼使用了Android內建的ListView控件,他的id是android:id/list。

使用以下XML代碼定義列表項佈局文件contacts\_list\_item.xml：

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@android:id/text1"
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:clickable="true"/>
```

這個XML代碼使用了Android內建的TextView控件,他的id是android:text1。

> **Note：**本課並不會描述如何從用戶那裡獲取搜索字符串的界面，因為我們可能會間接地獲取這個字符串。比如說，我們可能會給用戶一個選項去輸入文字信息，把這些文字信息作為搜索字符串去匹配聯繫人的名字。

剛剛寫的這兩個佈局文件定義了一個顯示ListView的用戶界面。下一步是編寫使用這個用戶界面顯示聯繫人列表的代碼。

### 定義一個顯示聯繫人列表的Fragment

為了顯示聯繫人列表，需要定義一個由Activity加載的Fragment。使用Fragment是一個比較靈活的方法，因為我們可以使用一個Fragment去顯示列表，用另一個Fragment顯示用戶在列表中選擇的聯繫人的詳情。使用這種方式，我們可以將本課程中展示的方法和另外一課[獲取聯繫人詳情](retrieve-detail.html)的方法聯繫起來。

想要學習如何在Activity中使用一個或者多個Fragment，請閱讀培訓課程[使用Fragment建立動態UI](http://hukai.me/android-training-course-in-chinese/basics/fragments/index.html)。

為了方便我們編寫對Contacts Provider的查詢，Android框架提供了一個叫做ContactsContract的契約類，這個類定義了一些對查詢Contacts Provider很有用的常量和方法。當我們使用這個類的時候，我們不用自己定義內容URI、表名、列名等常量。使用這個類，只需要引入以下類聲明：

```java
import android.provider.ContactsContract;
```

由於代碼中使用了CursorLoader去獲取provider的數據，所以我們必須實現加載器接口LoaderManager.LoaderCallbacks。同時，為了檢測用戶從結果列表中選擇了哪一個聯繫人，必須實現適配器接口AdapterView.OnItemClickListener。例如：

```java
...
import android.support.v4.app.Fragment;
import android.support.v4.app.LoaderManager.LoaderCallbacks;
import android.widget.AdapterView;
...
public class ContactsFragment extends Fragment implements
    LoaderManager.LoaderCallbacks<Cursor>,
    AdapterView.OnItemClickListener {
```

### 定義全局變量

定義在其他代部分碼中使用的全局變量：

```java
...
/*
 * Defines an array that contains column names to move from
 * the Cursor to the ListView.
 */
@SuppressLint("InlinedApi")
private final static String[] FROM_COLUMNS = {
        Build.VERSION.SDK_INT
                >= Build.VERSION_CODES.HONEYCOMB ?
                Contacts.DISPLAY_NAME_PRIMARY :
                Contacts.DISPLAY_NAME
};
/*
 * Defines an array that contains resource ids for the layout views
 * that get the Cursor column contents. The id is pre-defined in
 * the Android framework, so it is prefaced with "android.R.id"
 */
private final static int[] TO_IDS = {
       android.R.id.text1
};
// Define global mutable variables
// Define a ListView object
ListView mContactsList;
// Define variables for the contact the user selects
// The contact's _ID value
long mContactId;
// The contact's LOOKUP_KEY
String mContactKey;
// A content URI for the selected contact
Uri mContactUri;
// An adapter that binds the result Cursor to the ListView
private SimpleCursorAdapter mCursorAdapter;
...
```

> **Note：**由於Contacts.DISPLAY\_NAME\_PRIMARY需要在Android 3.0（API版本11）或者更高的版本才能使用，如果應用的minSdkVersion是10或者更低，會在eclipse中產生警告信息。為了關閉這個警告，我們可以在FROM_COLUMNS定義之前加上@SuppressLint("InlinedApi")註解。

### 初始化Fragment

為了初始化Fragment，Android系統需要我們為這個Fragment添加空的、公有的構造方法，同時在回調方法onCreateView()中綁定界面。例如：

```java
// Empty public constructor, required by the system
public ContactsFragment() {}
// A UI Fragment must inflate its View
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container,
        Bundle savedInstanceState) {
    // Inflate the fragment layout
    return inflater.inflate(R.layout.contact_list_fragment,
        container, false);
}
```

### 為ListView設置CursorAdapter

設置SimpleCursorAdapter，將搜索結果綁定到ListView。為了獲得顯示聯繫人列表的ListView控件，需要使用Fragment的父Activity調用Activity.findViewById()。當調用setAdapter()的時候，需要使用父Activity的上下文（Context）。

```java
public void onActivityCreated(Bundle savedInstanceState) {
    super.onActivityCreated(savedInstanceState);
    ...
    // Gets the ListView from the View list of the parent activity
    mContactsList =
        (ListView) getActivity().findViewById(R.layout.contact_list_view);
    // Gets a CursorAdapter
    mCursorAdapter = new SimpleCursorAdapter(
            getActivity(),
            R.layout.contact_list_item,
            null,
            FROM_COLUMNS, TO_IDS,
            0);
    // Sets the adapter for the ListView
    mContactsList.setAdapter(mCursorAdapter);
}
```

### 為選擇的聯繫人設置監聽器

當我們顯示搜索列表結果的時候，我們通常會讓用戶選擇某一個聯繫人去做進一步的處理。例如，當用戶選擇某一個聯繫人的時候，可以在地圖上顯示這個聯繫人的地址。為了能夠提供這個功能，我們需要定義當前的Fragment為一個點擊監聽器，這需要這個類實現AdapterView.OnItemClickListener接口，就像前面介紹的**定義顯示聯繫人列表的Fragment**那樣。

繼續設置這個監聽器，需要在onActivityCreated()方法中調用setOnItemClickListener()以使得這個監聽器綁定到ListView。例如：

```java
public void onActivityCreated(Bundle savedInstanceState) {
    ...
    // Set the item click listener to be the current fragment.
    mContactsList.setOnItemClickListener(this);
    ...
}
```

由於指定了當前的Fragment作為ListView的點擊監聽器，現在我們需要實現處理點擊事件的onItemClick()方法。這個會在隨後討論。

### 定義查詢映射

定義一個常量，這個常量包含我們想要從查詢結果中返回的列。Listview中的每一項顯示了一個聯繫人的名字。在Android 3.0（API version 11）或者更高的版本，這個列的名字是Contacts.DISPLAY\_NAME\_PRIMARY；在Android 3.0之前，這個列的名字是Contacts.DISPLAY\_NAME。

在SimpleCursorAdapter綁定過程中會用到Contacts.\_ID列。 Contacts.\_ID和LOOKUP\_KEY一同用來構建用戶選擇的聯繫人的內容URI。

```java
...
@SuppressLint("InlinedApi")
private static final String[] PROJECTION = {
    Contacts._ID,
    Contacts.LOOKUP_KEY,
    Build.VERSION.SDK_INT
            >= Build.VERSION_CODES.HONEYCOMB ?
            Contacts.DISPLAY_NAME_PRIMARY :
            Contacts.DISPLAY_NAME
};
```

### 定義Cursor的列索引常量

為了從Cursor中獲得單獨某一列的數據，我們需要知道這一列在Cursor中的索引值。我們需要定義Cursor列的索引值，這些索引值與我們定義查詢映射的列的順序是一樣的。例如：

```java
// The column index for the _ID column
private static final int CONTACT_ID_INDEX = 0;
// The column index for the LOOKUP_KEY column
private static final int LOOKUP_KEY_INDEX = 1;
```

### 指定查詢標準

為了指定我們想要的數據，我們需要創建一個包含文本表達式和變量的組合，去告訴provider我們需要的數據列和想要的值。

對於文本表達式，定義一個常量，列出所有搜索到的列。儘管這個表達式可以包含變量值，但是建議用"?"佔位符來替代這個值。在搜索的時候，佔位符裡的值會被數組裡的值所取代。使用"?"佔位符確保了搜索條件是由綁定產生而不是由SQL編譯產生。這個方法消除了惡意SQL注入的可能。例如：

```java
// Defines the text expression
@SuppressLint("InlinedApi")
private static final String SELECTION =
        Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB ?
        Contacts.DISPLAY_NAME_PRIMARY + " LIKE ?" :
        Contacts.DISPLAY_NAME + " LIKE ?";
// Defines a variable for the search string
private String mSearchString;
// Defines the array to hold values that replace the ?
private String[] mSelectionArgs = { mSearchString };
```

### 定義onItemClick()方法

在之前的內容中，我們為Listview設置了列表項點擊監聽器，現在需要定義AdapterView.OnItemClickListener.onItemClick()方法以實現監聽器行為：

```java
@Override
public void onItemClick(
    AdapterView<?> parent, View item, int position, long rowID) {
    // Get the Cursor
    Cursor cursor = parent.getAdapter().getCursor();
    // Move to the selected contact
    cursor.moveToPosition(position);
    // Get the _ID value
    mContactId = getLong(CONTACT_ID_INDEX);
    // Get the selected LOOKUP KEY
    mContactKey = getString(CONTACT_KEY_INDEX);
    // Create the contact's content Uri
    mContactUri = Contacts.getLookupUri(mContactId, mContactKey);
    /*
     * You can use mContactUri as the content URI for retrieving
     * the details for a contact.
     */
}
```

### 初始化Loader

由於使用了CursorLoader獲取數據，我們必須初始化後臺線程和其他的控制異步獲取數據的變量。需要在onActivityCreated()方法中做初始化的工作，這個方法是在Fragment的界面顯示之前被調用的，相關代碼展示如下：

```java
public class ContactsFragment extends Fragment implements
    LoaderManager.LoaderCallbacks<Cursor> {
...
	// Called just before the Fragment displays its UI
	@Override
	public void onActivityCreated(Bundle savedInstanceState) {
    	// Always call the super method first
    	super.onActivityCreated(savedInstanceState);
    	...
    	// Initializes the loader
    	getLoaderManager().initLoader(0, null, this);
```

### 實現onCreateLoader()方法

我們需要實現onCreateLoader()方法，這個方法是在調用initLoader()後馬上被loader框架調用的。

在onCreateLoader()方法中，設置搜索字符串模式。為了讓一個字符串符合一個模式，插入"%"字符代表0個或多個字符，插入"_"代表一個字符。例如，模式%Jefferson%將會匹配“Thomas Jefferson”和“Jefferson Davis”。

這個方法返回一個CursorLoader對象。對於內容URI，則使用了Contacts.CONTENT_URI，這個URI關聯到整個表，例子如下所示：

```java
...
@Override
public Loader<Cursor> onCreateLoader(int loaderId, Bundle args) {
    /*
     * Makes search string into pattern and
     * stores it in the selection array
     */
    mSelectionArgs[0] = "%" + mSearchString + "%";
    // Starts the query
    return new CursorLoader(
            getActivity(),
            Contacts.CONTENT_URI,
            PROJECTION,
            SELECTION,
            mSelectionArgs,
            null
    );
}
```

### 實現onLoadFinished()方法和onLoaderReset()方法

實現onLoadFinished()方法。當Contacts Provider返回查詢結果的時候，loader框架會調用onLoadFinished()方法。在這個方法中，將查詢結果Cursor傳給SimpleCursorAdapter，這將會使用這個搜索結果自動更新ListView。

```java
public void onLoadFinished(Loader<Cursor> loader, Cursor cursor) {
    // Put the result Cursor in the adapter for the ListView
    mCursorAdapter.swapCursor(cursor);
}
```

當loader框架檢測到結果集Cursor包含過時的數據時，它會調用onLoaderReset()。我們需要刪除SimpleCursorAdapter對已經存在Cursor的引用。如果不這麼做的話，loader框架將不會回收Cursor對象，這將會導致內存洩漏。例如：

```java
@Override
public void onLoaderReset(Loader<Cursor> loader) {
    // Delete the reference to the existing Cursor
    mCursorAdapter.swapCursor(null);
}
```

我們現在已經實現了一個應用的關鍵部分，即根據搜索字符串匹配聯繫人名字和將獲得的結果展示在ListView中。用戶可以點擊選擇一個聯繫人名字，這將會觸發一個監聽器，在監聽器的回調函數中，你可以使用此聯繫人的數據做進一步的處理。例如，你可以進一步獲取此聯繫人的詳情，想要知道何如獲取聯繫人詳情，請繼續學習下一課[獲取聯繫人詳情](retrieve-detail.html)。

想要了解更多搜索用戶界面的知識，請參考API指南[Creating a Search Interface](http://developer.android.com/guide/topics/search/search-dialog.html)。

這一課的以下內容展示了在Contacts Provider中查找聯繫人的其他方法。

## 根據特定的數據類型匹配聯繫人

這種方法可以讓我們指定想要匹配的數據類型。根據名字去檢索是這種類型的查詢的一個具體例子。但也可以用任何與聯繫人詳情數據相關的數據類型去做查詢。例如，我們可以檢索具有特定郵政編碼聯繫人，在這種情況下，搜索字符串將會去匹配存儲在一個郵政編碼列中的數據。

為了實現這種類型的檢索，首先實現以下的代碼，正如之前的內容所展示的：

* 請求讀取聯繫人的權限
* 定義列表和列表項的佈局
* 定義顯示聯繫人列表的Fragment
* 定義全局變量
* 初始化Fragment
* 為ListView設置CursorAdapter
* 設置選擇聯繫人的監聽器
* 定義Cursor的列索引常量

	儘管我們現在從不同的表中取數據，檢索列的映射順序是一樣的，所以我們可以為這個Cursor使用同樣的索引常量。
* 定義onItemClick()方法
* 初始化loader
* 實現onLoadFinished()方法和onLoaderReset()方法

為了將搜索字符串匹配特定的詳請數據類型並顯示結果，以下的步驟展示了我們需要額外添加的代碼。

### 選擇要查詢的數據類型和數據庫表

為了從特定類型的詳請數據中查詢，我們必須知道的數據類型的自定義MIME類型的值。每一個數據類型擁有唯一的`MIME`類型值，這個值在ContactsContract.CommonDataKinds的子類中被定義為常量`CONTENT_ITEM_TYPE`，並且與實際的數據類型相關。子類的名字會表明它們的實際數據類型。例如，email數據的子類是`ContactsContract.CommonDataKinds.Email`，並且email的自定義MIME類型是`Email.CONTENT_ITEM_TYPE`。

在搜索中需要使用ContactsContract.Data類。同時所有需要的常量，包括數據映射、選擇字句、排序規則都是由這個類定義或繼承自此類。

### 定義查詢映射

為了定義一個查詢映射，請選擇一個或者多個定義在ContactsContract.Data表或其子類的列。Contacts Provider在返回行結果集之前，隱式的連接了ContactsContract.Data表和其他表。例如：

```java
@SuppressLint("InlinedApi")
private static final String[] PROJECTION = {
    /*
     * The detail data row ID. To make a ListView work,
     * this column is required.
     */
    Data._ID,
    // The primary display name
    Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB ?
            Data.DISPLAY_NAME_PRIMARY :
            Data.DISPLAY_NAME,
    // The contact's _ID, to construct a content URI
    Data.CONTACT_ID
    // The contact's LOOKUP_KEY, to construct a content URI
    Data.LOOKUP_KEY (a permanent link to the contact
};
```

### 定義查詢標準

為了根據特定的聯繫人數據類型查詢字符串，請按照以下方法構建查詢選擇子句：

* 包含搜索字符串的列名。這個名字根據數據類型所變化，所以我們需要找到與數據類型對應的ContactsContract.CommonDataKinds的子類，並從這個子類中選擇列名。例如，想要搜索email地址，需要使用Email.ADDRESS列。
* 搜索字符串本身，請在查詢選擇子句裡使用"?"表示。
* 列名包含自定義的MIME類型值。這個列名字總是Data.MIMETYPE。
* 自定義MIME類型值的數據類型。如之前描述，這需要使用ContactsContract.CommonDataKinds子類中的`CONTENT_ITEM_TYPE`常量。例如，email數據的MIME類型值是`Email.CONTENT_ITEM_TYPE`。需要在這個常量值的開頭和結尾加上"'"（單引號）。否則，provider會把這個值翻譯成一個變量而不是一個字符串。我們不需要為這個值提供佔位符，因為我們在使用一個常量而不是用戶提供的值。例如：

```java
/*
 * Constructs search criteria from the search string
 * and email MIME type
 */
private static final String SELECTION =
    /*
     * Searches for an email address
     * that matches the search string
     */
    Email.ADDRESS + " LIKE ? " + "AND " +
    /*
     * Searches for a MIME type that matches
     * the value of the constant
     * Email.CONTENT_ITEM_TYPE. Note the
     * single quotes surrounding Email.CONTENT_ITEM_TYPE.
     */
    Data.MIMETYPE + " = '" + Email.CONTENT_ITEM_TYPE + "'";
```

下一步，定義包含選擇字符串的變量：

```java
String mSearchString;
String[] mSelectionArgs = { "" };
```

### 實現onCreateLoader()方法

現在，我們已經詳述了想要的數據和如何找到這些數據，如何在onCreateLoader()方法中定義一個查詢。使用你的數據映射、查詢選擇表達式和一個數組作為選擇表達式的參數，並從這個方法中返回一個新的CursorLoader對象。而內容URI需要使用Data.CONTENT_URI，例如：

```java
@Override
public Loader<Cursor> onCreateLoader(int loaderId, Bundle args) {
    // OPTIONAL: Makes search string into pattern
    mSearchString = "%" + mSearchString + "%";
    // Puts the search string into the selection criteria
    mSelectionArgs[0] = mSearchString;
    // Starts the query
    return new CursorLoader(
            getActivity(),
            Data.CONTENT_URI,
            PROJECTION,
            SELECTION,
            mSelectionArgs,
            null
    );
}
```

這段代碼片段是基於特定的聯繫人詳情數據類型的簡單反向查找。如果我們的應用關注於某一種特定的數據類型，比如說email地址，並且允許用戶獲得與此數據相關的聯繫人名字，這種形式的查詢是最好的方法。

## 根據任意類型的數據匹配聯繫人

根據任意數據類型獲取聯繫人時，如果聯繫人的數據（這些數據包括名字、email地址、郵件地址和電話號碼等等）能匹配要搜索的字符串，那麼該聯繫人信息將會被返回。這種搜索結果會比較廣泛。例如，如果搜索字符串是"Doe"，搜索任意類型的數據將會返回名字為"Jone Doe"的聯繫人，也會返回一個住在"Doe Street"的聯繫人。

為了完成這種類型的查詢，就像之前展示的那樣，首先需要實現以下代碼：

* 請求讀取聯繫人的權限
* 定義列表和列表項的佈局
* 定義顯示聯繫人列表的Fragment
* 定義全局變量
* 初始化Fragment
* 為ListView設置CursorAdapter
* 設置選擇聯繫人的監聽器
* 定義Cursor的列索引常量

	對於這種形式的查詢，你需要使用與在“使用特定類型的數據匹配聯繫人”那一節中相同的表，也可以使用相同的列索引。
* 定義onItemClick()方法
* 初始化loader
* 實現onLoadFinished()方法和onLoaderReset()方法

以下的步驟展示了為了能夠根據任意的數據類型去匹配查詢字符串並顯示結果列表，我們需要添加的額外代碼。

### 去除查詢標準

不需要為mSelectionArgs定義查詢標準常量SELECTION。這些內容在這種類型的檢索不會被用到。

### 實現onCreateLoader()方法

實現onCreateLoader()方法，返回一個新的CursorLoader對象。我們不需要把搜索字符串轉化成一個搜索模式，因為Contacts Provider會自動做這件事。使用Contacts.CONTENT\_FILTER\_URI作為基礎查詢URI，並使用Uri.withAppendedPath()方法將搜索字符串添加到基礎URI中。使用這個URI會自動觸發對任意數據類型的搜索，就像以下例子所示：

```java
@Override
public Loader<Cursor> onCreateLoader(int loaderId, Bundle args) {
    /*
     * Appends the search string to the base URI. Always
     * encode search strings to ensure they're in proper
     * format.
     */
    Uri contentUri = Uri.withAppendedPath(
            Contacts.CONTENT_FILTER_URI,
            Uri.encode(mSearchString));
    // Starts the query
    return new CursorLoader(
            getActivity(),
            contentUri,
            PROJECTION,
            null,
            null,
            null
    );
}
```

這段代碼片段，是想要在Contacts Provider中建立廣泛搜索類型應用的基礎部分。這種方法對那些想要實現與通訊錄應用聯繫人列表中相似搜索功能的應用，會很有幫助。
