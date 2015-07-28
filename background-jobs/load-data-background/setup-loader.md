# 使用CursorLoader執行查詢任務

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/load-data-background/setup-loader.html>

CursorLoader通過ContentProvider在後臺執行一個異步的查詢操作，並且返回數據給調用它的Activity或者FragmentActivity。這使得Activity或者FragmentActivity能夠在查詢任務正在執行的同時繼續與用戶進行其他的交互操作。

## 定義使用CursorLoader的Activity

為了在Activity或者FragmentActivity中使用CursorLoader，它們需要實現`LoaderCallbacks<Cursor>`接口。CursorLoader會調用`LoaderCallbacks<Cursor>`定義的這些回調方法與Activity進行交互；這節課與下節課會詳細介紹每一個回調方法。

<!-- More -->

例如，下面演示了FragmentActivity如何使用CursorLoader。

```java
public class PhotoThumbnailFragment extends FragmentActivity implements
        LoaderManager.LoaderCallbacks<Cursor> {
...
}
```

## 初始化查詢

為了初始化查詢，需要調用`LoaderManager.initLoader()`。這個方法可以初始化LoaderManager的後臺查詢框架。你可以在用戶輸入查詢條件之後觸發初始化的操作，如果你不需要用戶輸入數據作為查詢條件，你可以在`onCreate()`或者`onCreateView()`裡面觸發這個方法。例如：

```java
// Identifies a particular Loader being used in this component
private static final int URL_LOADER = 0;
...
/* When the system is ready for the Fragment to appear, this displays
 * the Fragment's View
 */
public View onCreateView(
        LayoutInflater inflater,
        ViewGroup viewGroup,
        Bundle bundle) {
    ...
    /*
     * Initializes the CursorLoader. The URL_LOADER value is eventually passed
     * to onCreateLoader().
     */
    getLoaderManager().initLoader(URL_LOADER, null, this);
    ...
}
```

> **Note:** `getLoaderManager()`僅僅是在Fragment類中可以直接訪問。為了在FragmentActivity中獲取到LoaderManager，需要執行`getSupportLoaderManager()`.

## 開始查詢

一旦後臺任務被初始化好，它會執行你實現的回調方法`onCreateLoader()`。為了啟動查詢任務，會在這個方法裡面返回CursorLoader。你可以初始化一個空的CursorLoader然後使用它的方法來定義你的查詢條件，或者你可以在初始化CursorLoader對象的時候就同時定義好查詢條件：

```java
/*
* Callback that's invoked when the system has initialized the Loader and
* is ready to start the query. This usually happens when initLoader() is
* called. The loaderID argument contains the ID value passed to the
* initLoader() call.
*/
@Override
public Loader<Cursor> onCreateLoader(int loaderID, Bundle bundle)
{
    /*
     * Takes action based on the ID of the Loader that's being created
     */
    switch (loaderID) {
        case URL_LOADER:
            // Returns a new CursorLoader
            return new CursorLoader(
                        getActivity(),   // Parent activity context
                        mDataUrl,        // Table to query
                        mProjection,     // Projection to return
                        null,            // No selection clause
                        null,            // No selection arguments
                        null             // Default sort order
        );
        default:
            // An invalid id was passed in
            return null;
    }
}
```

一旦後臺查詢任務獲取到了這個Loader對象，就開始在後臺執行查詢的任務。當查詢完成之後，會執行`onLoadFinished()`這個回調函數，關於這些內容會在下一節講到。

