# 保存到數據庫

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/basics/data-storage/databases.html>

對於重複或者結構化的數據（如聯繫人信息）等保存到DB是個不錯的主意。本課假定讀者已經熟悉SQL數據庫的常用操作。在Android上可能會使用到的APIs，可以從[android.database.sqlite](http://developer.android.com/reference/android/database/sqlite/package-summary.html)包中找到。

## 定義Schema與Contract

SQL中一個重要的概念是schema：一種DB結構的正式聲明，用於表示database的組成結構。schema是從創建DB的SQL語句中生成的。我們會發現創建一個伴隨類（companion class）是很有益的，這個類稱為合約類（contract class）,它用一種系統化並且自動生成文檔的方式，顯示指定了schema樣式。

Contract Clsss是一些常量的容器。它定義了例如URIs，表名，列名等。這個contract類允許在同一個包下與其他類使用同樣的常量。 它讓我們只需要在一個地方修改列名，然後這個列名就可以自動傳遞給整個code。

組織contract類的一個好方法是在類的根層級定義一些全局變量，然後為每一個table來創建內部類。

> **Note：**通過實現 [BaseColumns](http://developer.android.com/reference/android/provider/BaseColumns.html) 的接口，內部類可以繼承到一個名為_ID的主鍵，這個對於Android裡面的一些類似cursor adaptor類是很有必要的。這麼做不是必須的，但這樣能夠使得我們的DB與Android的framework能夠很好的相容。

例如，下面的例子定義了表名與該表的列名：

```java
public final class FeedReaderContract {
    // To prevent someone from accidentally instantiating the contract class,
    // give it an empty constructor.
    public FeedReaderContract() {}

    /* Inner class that defines the table contents */
    public static abstract class FeedEntry implements BaseColumns {
        public static final String TABLE_NAME = "entry";
        public static final String COLUMN_NAME_ENTRY_ID = "entryid";
        public static final String COLUMN_NAME_TITLE = "title";
        public static final String COLUMN_NAME_SUBTITLE = "subtitle";
        ...
    }
}
```

## 使用SQL Helper創建DB

定義好了的DB的結構之後，就應該實現那些創建與維護db和table的方法。下面是一些典型的創建與刪除table的語句。

```java
private static final String TEXT_TYPE = " TEXT";
private static final String COMMA_SEP = ",";
private static final String SQL_CREATE_ENTRIES =
    "CREATE TABLE " + FeedReaderContract.FeedEntry.TABLE_NAME + " (" +
    FeedReaderContract.FeedEntry._ID + " INTEGER PRIMARY KEY," +
    FeedReaderContract.FeedEntry.COLUMN_NAME_ENTRY_ID + TEXT_TYPE + COMMA_SEP +
    FeedReaderContract.FeedEntry.COLUMN_NAME_TITLE + TEXT_TYPE + COMMA_SEP +
    ... // Any other options for the CREATE command
    " )";

private static final String SQL_DELETE_ENTRIES =
    "DROP TABLE IF EXISTS " + TABLE_NAME_ENTRIES;
```

類似於保存文件到設備的[internal storage](http://developer.android.com/guide/topics/data/data-storage.html#filesInternal) ，Android會將db保存到程序的private的空間。我們的數據是受保護的，因為那些區域默認是私有的，不可被其他程序所訪問。

在[SQLiteOpenHelper](http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html)類中有一些很有用的APIs。當使用這個類來做一些與db有關的操作時，系統會對那些有可能比較耗時的操作（例如創建與更新等）在真正需要的時候才去執行，而不是在app剛啟動的時候就去做那些動作。我們所需要做的僅僅是執行<a href="http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html#getWritableDatabase()">getWritableDatabase()</a>或者<a href="http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html#getReadableDatabase()">getReadableDatabase()</a>.

> **Note：**因為那些操作可能是很耗時的，請確保在background thread（AsyncTask or IntentService）裡面去執行 getWritableDatabase() 或者 getReadableDatabase() 。

為了使用 SQLiteOpenHelper, 需要創建一個子類並重寫<a href="http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html#onCreate(android.database.sqlite.SQLiteDatabase)">onCreate()</a>, <a href="http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html#onUpgrade(android.database.sqlite.SQLiteDatabase, int, int)">onUpgrade()</a>與<a href="http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html#onOpen(android.database.sqlite.SQLiteDatabase)">onOpen()</a>等callback方法。也許還需要實現<a href="http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html#onDowngrade(android.database.sqlite.SQLiteDatabase, int, int)">onDowngrade()</a>, 但這並不是必需的。

例如，下面是一個實現了SQLiteOpenHelper 類的例子：

```java
public class FeedReaderDbHelper extends SQLiteOpenHelper {
    // If you change the database schema, you must increment the database version.
    public static final int DATABASE_VERSION = 1;
    public static final String DATABASE_NAME = "FeedReader.db";

    public FeedReaderDbHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(SQL_CREATE_ENTRIES);
    }
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        // This database is only a cache for online data, so its upgrade policy is
        // to simply to discard the data and start over
        db.execSQL(SQL_DELETE_ENTRIES);
        onCreate(db);
    }
    public void onDowngrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        onUpgrade(db, oldVersion, newVersion);
    }
}
```

為了訪問我們的db，需要實例化 SQLiteOpenHelper的子類：

```java
FeedReaderDbHelper mDbHelper = new FeedReaderDbHelper(getContext());
```

## 添加信息到DB

通過傳遞一個 [ContentValues](http://developer.android.com/reference/android/content/ContentValues.html) 對象到<a href="http://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html#insert(java.lang.String, java.lang.String, android.content.ContentValues)">insert()</a>方法：

```java
// Gets the data repository in write mode
SQLiteDatabase db = mDbHelper.getWritableDatabase();

// Create a new map of values, where column names are the keys
ContentValues values = new ContentValues();
values.put(FeedReaderContract.FeedEntry.COLUMN_NAME_ENTRY_ID, id);
values.put(FeedReaderContract.FeedEntry.COLUMN_NAME_TITLE, title);
values.put(FeedReaderContract.FeedEntry.COLUMN_NAME_CONTENT, content);

// Insert the new row, returning the primary key value of the new row
long newRowId;
newRowId = db.insert(
         FeedReaderContract.FeedEntry.TABLE_NAME,
         FeedReaderContract.FeedEntry.COLUMN_NAME_NULLABLE,
         values);
```

`insert()`方法的第一個參數是table名，第二個參數會使得系統自動對那些`ContentValues` 沒有提供數據的列填充數據為`null`，如果第二個參數傳遞的是null，那麼系統則不會對那些沒有提供數據的列進行填充。

## 從DB中讀取信息

為了從DB中讀取數據，需要使用<a href="http://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html#query(boolean, java.lang.String, java.lang.String[], java.lang.String, java.lang.String[], java.lang.String, java.lang.String, java.lang.String, java.lang.String)">query()</a>方法，傳遞需要查詢的條件。查詢後會返回一個 [Cursor](http://developer.android.com/reference/android/database/Cursor.html) 對象。

```java
SQLiteDatabase db = mDbHelper.getReadableDatabase();

// Define a projection that specifies which columns from the database
// you will actually use after this query.
String[] projection = {
    FeedReaderContract.FeedEntry._ID,
    FeedReaderContract.FeedEntry.COLUMN_NAME_TITLE,
    FeedReaderContract.FeedEntry.COLUMN_NAME_UPDATED,
    ...
    };

// How you want the results sorted in the resulting Cursor
String sortOrder =
    FeedReaderContract.FeedEntry.COLUMN_NAME_UPDATED + " DESC";

Cursor c = db.query(
    FeedReaderContract.FeedEntry.TABLE_NAME,  // The table to query
    projection,                               // The columns to return
    selection,                                // The columns for the WHERE clause
    selectionArgs,                            // The values for the WHERE clause
    null,                                     // don't group the rows
    null,                                     // don't filter by row groups
    sortOrder                                 // The sort order
    );
```
要查詢在cursor中的行，使用cursor的其中一個move方法，但必須在讀取值之前調用。一般來說應該先調用`moveToFirst()`函數，將讀取位置置於結果集最開始的位置。對每一行，我們可以使用cursor的其中一個get方法如`getString()`或`getLong()`獲取列的值。對於每一個get方法必須傳遞想要獲取的列的索引位置(index position)，索引位置可以通過調用`getColumnIndex()`或`getColumnIndexOrThrow()`獲得。

下面演示如何從course對象中讀取數據信息：

```java
cursor.moveToFirst();
long itemId = cursor.getLong(
    cursor.getColumnIndexOrThrow(FeedReaderContract.FeedEntry._ID)
);
```

## 刪除DB中的信息

和查詢信息一樣，刪除數據同樣需要提供一些刪除標準。DB的API提供了一個防止SQL注入的機制來創建查詢與刪除標準。

> **SQL Injection：**(*隨著B/S模式應用開發的發展，使用這種模式編寫應用程序的程序員也越來越多。但由於程序員的水平及經驗也參差不齊，相當大一部分程序員在編寫代碼時沒有對用戶輸入數據的合法性進行判斷，使應用程序存在安全隱患。用戶可以提交一段數據庫查詢代碼，根據程序返回的結果，獲得某些他想得知的數據，這就是所謂的SQL Injection，即SQL注入*)

該機制把查詢語句劃分為選項條件與選項參數兩部分。條件定義了查詢的列的特徵，參數用於測試是否符合前面的條款。由於處理的結果不同於通常的SQL語句，這樣可以避免SQL注入問題。

```java
// Define 'where' part of query.
String selection = FeedReaderContract.FeedEntry.COLUMN_NAME_ENTRY_ID + " LIKE ?";
// Specify arguments in placeholder order.
String[] selelectionArgs = { String.valueOf(rowId) };
// Issue SQL statement.
db.delete(table_name, mySelection, selectionArgs);
```

## 更新數據

當需要修改DB中的某些數據時，使用 update() 方法。

update結合了插入與刪除的語法。

```java
SQLiteDatabase db = mDbHelper.getReadableDatabase();

// New value for one column
ContentValues values = new ContentValues();
values.put(FeedReaderContract.FeedEntry.COLUMN_NAME_TITLE, title);

// Which row to update, based on the ID
String selection = FeedReaderContract.FeedEntry.COLUMN_NAME_ENTRY_ID + " LIKE ?";
String[] selectionArgs = { String.valueOf(rowId) };

int count = db.update(
    FeedReaderDbHelper.FeedEntry.TABLE_NAME,
    values,
    selection,
    selectionArgs);
```
