# 保存並搜索數據

> 編寫:[Lin-H](https://github.com/Lin-H) - 原文:<http://developer.android.com/training/search/search.html>

有很多方法可以儲存你的數據，比如儲存在線上的數據庫，本地的SQLite數據庫，甚至是文本文件。你自己來選擇最適合你應用的存儲方式。本節課程會向你展示如何創建一個健壯的可以提供全文搜索的SQLite虛擬表。並從一個每行有一組單詞-解釋對的文件中將數據填入。

##創建虛擬表

虛擬表與SQLite表的運行方式類似，但虛擬表是通過回調來向內存中的對象進行讀取和寫入，而不是通過數據庫文件。要創建一個虛擬表，首先為該表創建一個類:

```java
public class DatabaseTable {
    private final DatabaseOpenHelper mDatabaseOpenHelper;

    public DatabaseTable(Context context) {
        mDatabaseOpenHelper = new DatabaseOpenHelper(context);
    }
}
```

在`DatabaseTable`類中創建一個繼承[SQLiteOpenHelper](http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html)的內部類。你必須重寫類[SQLiteOpenHelper](http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html)中定義的abstract方法，才能在必要的時候創建和更新你的數據庫表。例如，下面一段代碼聲明瞭一個數據庫表，用來儲存字典app所需的單詞。

```java
public class DatabaseTable {

    private static final String TAG = "DictionaryDatabase";

    //字典的表中將要包含的列項
    public static final String COL_WORD = "WORD";
    public static final String COL_DEFINITION = "DEFINITION";

    private static final String DATABASE_NAME = "DICTIONARY";
    private static final String FTS_VIRTUAL_TABLE = "FTS";
    private static final int DATABASE_VERSION = 1;

    private final DatabaseOpenHelper mDatabaseOpenHelper;

    public DatabaseTable(Context context) {
        mDatabaseOpenHelper = new DatabaseOpenHelper(context);
    }

    private static class DatabaseOpenHelper extends SQLiteOpenHelper {

        private final Context mHelperContext;
        private SQLiteDatabase mDatabase;

        private static final String FTS_TABLE_CREATE =
                    "CREATE VIRTUAL TABLE " + FTS_VIRTUAL_TABLE +
                    " USING fts3 (" +
                    COL_WORD + ", " +
                    COL_DEFINITION + ")";

        DatabaseOpenHelper(Context context) {
            super(context, DATABASE_NAME, null, DATABASE_VERSION);
            mHelperContext = context;
        }

        @Override
        public void onCreate(SQLiteDatabase db) {
            mDatabase = db;
            mDatabase.execSQL(FTS_TABLE_CREATE);
        }

        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
            Log.w(TAG, "Upgrading database from version " + oldVersion + " to "
                    + newVersion + ", which will destroy all old data");
            db.execSQL("DROP TABLE IF EXISTS " + FTS_VIRTUAL_TABLE);
            onCreate(db);
        }
    }
}
```

##填入虛擬表

現在，表需要數據來儲存。下面的代碼會向你展示如何讀取一個內容為單詞和解釋的文本文件(位於`res/raw/definitions.txt`)，如何解析文件與如何將文件中的數據按行插入虛擬表中。為防止UI鎖死這些操作會在另一條線程中執行。將下面的一段代碼添加到你的`DatabaseOpenHelper`內部類中。

>**Tip**:你也可以設置一個回調來通知你的UI activity線程的完成結果。

```java
private void loadDictionary() {
        new Thread(new Runnable() {
            public void run() {
                try {
                    loadWords();
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        }).start();
    }

private void loadWords() throws IOException {
    final Resources resources = mHelperContext.getResources();
    InputStream inputStream = resources.openRawResource(R.raw.definitions);
    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));

    try {
        String line;
        while ((line = reader.readLine()) != null) {
            String[] strings = TextUtils.split(line, "-");
            if (strings.length < 2) continue;
            long id = addWord(strings[0].trim(), strings[1].trim());
            if (id < 0) {
                Log.e(TAG, "unable to add word: " + strings[0].trim());
            }
        }
    } finally {
        reader.close();
    }
}

public long addWord(String word, String definition) {
    ContentValues initialValues = new ContentValues();
    initialValues.put(COL_WORD, word);
    initialValues.put(COL_DEFINITION, definition);

    return mDatabase.insert(FTS_VIRTUAL_TABLE, null, initialValues);
}
```

任何恰當的地方，都可以調用`loadDictionary()`方法向表中填入數據。一個比較好的地方是`DatabaseOpenHelper`類的[onCreate()](http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html#onCreate(android.database.sqlite.SQLiteDatabase))方法中，緊隨創建表之後:

```java
@Override
public void onCreate(SQLiteDatabase db) {
    mDatabase = db;
    mDatabase.execSQL(FTS_TABLE_CREATE);
    loadDictionary();
}
```

##搜索請求

當你的虛擬表創建好並填入數據後，根據[SearchView](http://developer.android.com/reference/android/widget/SearchView.html)提供的請求搜索數據。將下面的方法添加到`DatabaseTable`類中，用來創建搜索請求的SQL語句:

```java
public Cursor getWordMatches(String query, String[] columns) {
    String selection = COL_WORD + " MATCH ?";
    String[] selectionArgs = new String[] {query+"*"};

    return query(selection, selectionArgs, columns);
}

private Cursor query(String selection, String[] selectionArgs, String[] columns) {
    SQLiteQueryBuilder builder = new SQLiteQueryBuilder();
    builder.setTables(FTS_VIRTUAL_TABLE);

    Cursor cursor = builder.query(mDatabaseOpenHelper.getReadableDatabase(),
            columns, selection, selectionArgs, null, null, null);

    if (cursor == null) {
        return null;
    } else if (!cursor.moveToFirst()) {
        cursor.close();
        return null;
    }
    return cursor;
}
```

調用`getWordMatches()`來搜索請求。任何符合的結果返回到[Cursor](http://developer.android.com/reference/android/database/Cursor.html)中，可以直接遍歷或是建立一個[ListView](http://developer.android.com/reference/android/widget/ListView.html)。這個例子是在檢索activity的`handleIntent()`方法中調用`getWordMatches()`。請記住，因為之前創建的intent filter，檢索activity會在[ACTION_SEARCH](http://developer.android.com/reference/android/content/Intent.html#ACTION_SEARCH) intent中額外接收請求作為變量存儲:

```java
DatabaseTable db = new DatabaseTable(this);

...

private void handleIntent(Intent intent) {

    if (Intent.ACTION_SEARCH.equals(intent.getAction())) {
        String query = intent.getStringExtra(SearchManager.QUERY);
        Cursor c = db.getWordMatches(query, null);
        //執行Cursor並顯示結果
    }
}
```
