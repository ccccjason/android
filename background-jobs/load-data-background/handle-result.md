# 處理查詢的結果

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/load-data-background/handle-results.html>

正如前面一節課講到的，你應該在 [onCreateLoader()](1)的回調裡面使用CursorLoader執行加載數據的操作。Loader查詢完後會調用Activity或者FragmentActivity的[LoaderCallbacks.onLoadFinished()](2)將結果回調回來。這個回調方法的參數之一是[Cursor](4)，它包含了查詢的數據。你可以使用Cursor對象來更新需要顯示的數據或者進行下一步的處理。

除了[onCreateLoader()](1)與[onLoadFinished()](2)，你也需要實現[onLoaderReset()](3)。這個方法在CursorLoader檢測到[Cursor](4)上的數據發生變化的時候會被觸發。當數據發生變化時，系統也會觸發重新查詢的操作。

<!-- More -->

## 處理查詢結果

為了顯示CursorLoader返回的Cursor數據，需要使用實現AdapterView的視圖組件，，併為這個組件綁定一個實現了CursorAdapter的Adapter。系統會自動把Cursor中的數據顯示到View上。

你可以在顯示數據之前建立View與Adapter的關聯。然後在[onLoadFinished()](2)的時候把Cursor與Adapter進行綁定。一旦你把Cursor與Adapter進行綁定之後，系統會自動更新View。當Cursor上的內容發生改變的時候，也會觸發這些操作。

例如:

```java
public String[] mFromColumns = {
    DataProviderContract.IMAGE_PICTURENAME_COLUMN
};
public int[] mToFields = {
    R.id.PictureName
};
// Gets a handle to a List View
ListView mListView = (ListView) findViewById(R.id.dataList);
/*
 * Defines a SimpleCursorAdapter for the ListView
 *
 */
SimpleCursorAdapter mAdapter =
    new SimpleCursorAdapter(
            this,                // Current context
            R.layout.list_item,  // Layout for a single row
            null,                // No Cursor yet
            mFromColumns,        // Cursor columns to use
            mToFields,           // Layout fields to use
            0                    // No flags
    );
// Sets the adapter for the view
mListView.setAdapter(mAdapter);
...
/*
 * Defines the callback that CursorLoader calls
 * when it's finished its query
 */
@Override
public void onLoadFinished(Loader<Cursor> loader, Cursor cursor) {
    ...
    /*
     * Moves the query results into the adapter, causing the
     * ListView fronting this adapter to re-display
     */
    mAdapter.changeCursor(cursor);
}
```

## 刪除廢舊的Cursor引用

當Cursor失效的時候，CursorLoader會被重置。這通常發生在Cursor相關的數據改變的時候。在重新執行查詢操作之前，系統會執行你的[onLoaderReset()](3)回調方法。在這個回調方法中，你應該刪除當前Cursor上的所有數據，避免發生內存洩露。一旦onLoaderReset()執行結束，CursorLoader就會重新執行查詢操作。

例如:

```java
/*
 * Invoked when the CursorLoader is being reset. For example, this is
 * called if the data in the provider changes and the Cursor becomes stale.
 */
@Override
public void onLoaderReset(Loader<Cursor> loader) {

    /*
     * Clears out the adapter's reference to the Cursor.
     * This prevents memory leaks.
     */
    mAdapter.changeCursor(null);
}
```

***

[1]: http://developer.android.com/reference/android/support/v4/app/LoaderManager.LoaderCallbacks.html "onCreateLoader()"
[2]: http://developer.android.com/reference/android/support/v4/app/LoaderManager.LoaderCallbacks.html  "onLoadFinished()"
[3]: http://developer.android.com/reference/android/support/v4/app/LoaderManager.LoaderCallbacks.html  "onLoaderReset()"
[4]: http://developer.android.com/reference/android/database/Cursor.html  "Cursor"

