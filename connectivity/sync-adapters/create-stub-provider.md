# 創建 Stub Content Provider

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/sync-adapters/creating-stub-provider.html>

Sync Adapter 框架是設計成用來和設備數據一起工作的，而這些設備數據應該被靈活且安全的 Content Provider 框架管理。因此，Sync Adapter 框架會期望應用已經為它的本地數據定義了 Content Provider。如果 Sync Adapter 框架嘗試去運行我們的 Sync Adapter，而我們的應用沒有一個 Content Provider 的話，那麼 Sync Adapter 將會崩潰。

如果我們正在開發一個新的應用，它將數據從服務器傳輸到一臺設備上，那麼我們務必考慮將本地數據存儲於 Content Provider 中。除了它對於 Sync Adapter 的重要性之外，Content Provider 還可以提供許多安全上的好處，更何況它是專門為了在 Android 設備上處理數據存儲而設計的。要學習如何創建一個 Content Provider，可以閱讀：[Creating a Content Provider](http://developer.android.com/guide/topics/providers/content-provider-creating.html)。

然而，如果我們已經通過別的形式來存儲本地數據，我們仍然可以使用 Sync Adapter 來處理數據傳輸。為了滿足 Sync Adapter 框架對於 Content Provider 的要求，我們可以在應用中添加一個 Stub Content Provider。一個 Stub Content Provider 實現了 Content Provider 類，但是所有的方法都返回 `null` 或者 `0`。如果我們添加了一個 Stub Content Provider，那麼無論數據存儲機制是什麼，我們都可以使用 Sync Adapter 來傳輸數據。

如果在我們的應用中已經有了一個 Content Provider，那麼我們就不需要創建 Stub Content Provider 了。在這種情況下，我們可以略過這節課程，直接進入：[創建 Sync Adapter](create-sync-adapter.html)。如果你還沒有創建 Content Provider，這節課將向你展示如何通過添加一個 Stub Content Provider，將你的 Sync Adapter 添加到框架中。

## 添加一個 Stub Content Provider

要為我們的應用創建一個 Stub Content Provider，首先繼承 [ContentProvider](http://developer.android.com/reference/android/content/ContentProvider.html) 類，並且在所有需要重寫的方法中，我們一律不進行任何處理而是直接返回。下面的代碼片段展示了我們應該如何創建一個 Stub Content Provider：

```java
/*
 * Define an implementation of ContentProvider that stubs out
 * all methods
 */
public class StubProvider extends ContentProvider {
    /*
     * Always return true, indicating that the
     * provider loaded correctly.
     */
    @Override
    public boolean onCreate() {
        return true;
    }
    /*
     * Return an empty String for MIME type
     */
    @Override
    public String getType() {
        return new String();
    }
    /*
     * query() always returns no results
     *
     */
    @Override
    public Cursor query(
            Uri uri,
            String[] projection,
            String selection,
            String[] selectionArgs,
            String sortOrder) {
        return null;
    }
    /*
     * insert() always returns null (no URI)
     */
    @Override
    public Uri insert(Uri uri, ContentValues values) {
        return null;
    }
    /*
     * delete() always returns "no rows affected" (0)
     */
    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        return 0;
    }
    /*
     * update() always returns "no rows affected" (0)
     */
    public int update(
            Uri uri,
            ContentValues values,
            String selection,
            String[] selectionArgs) {
        return 0;
    }
}
```

## 在 Manifest 清單文件中聲明 Provider

Sync Adapter 框架會通過查看應用的 manifest 文件中是否聲明瞭 provider，來驗證我們的應用是否使用了 Content Provider。為了在 manifest 清單文件中聲明我們的 Stub Content Provider，添加一個 [`<provider>`](http://developer.android.com/guide/topics/manifest/provider-element.html) 標籤，並讓它擁有下列屬性字段：

`android:name="com.example.android.datasync.provider.StubProvider"`

  指定實現 Stub Content Provider 類的完整包名。

`android:authorities="com.example.android.datasync.provider"`

  指定 Stub Content Provider 的 URI Authority。用應用的包名加上字符串 `".provider"` 作為該屬性字段的值。雖然我們在這裡向系統聲明瞭 Stub Content Provider，但是不會嘗試訪問 Provider 本身。

`android:exported="false"`

  確定其它應用是否可以訪問 Content Provider。對於 Stub Content Provider 而言，由於沒有讓其它應用訪問該 Provider 的必要，所以我們將該值設置為 `false`。該值並不會影響 Sync Adapter 框架和 Content Provider 之間的交互。

`android:syncable="true"`

  該標識指明 Provider 是可同步的。如果將這個值設置為 `true`，那麼將不需要在代碼中調用 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#setIsSyncable(android.accounts.Account, java.lang.String, int)">setIsSyncable()</a>。這一標識將會允許 Sync Adapter 框架和 Content Provider 進行數據傳輸，但是僅僅在我們顯式地執行相關調用時，這一傳輸時才會進行。

下面的代碼片段展示了我們應該如何將 [`<provider>`](http://developer.android.com/guide/topics/manifest/provider-element.html) 標籤添加到應用的 manifest 清單文件中：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.android.network.sync.BasicSyncAdapter"
    android:versionCode="1"
    android:versionName="1.0" >
    <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
    ...
    <provider
        android:name="com.example.android.datasync.provider.StubProvider"
        android:authorities="com.example.android.datasync.provider"
        android:exported="false"
        android:syncable="true"/>
    ...
    </application>
</manifest>
```

現在我們已經創建了所有 Sync Adapter 框架所需要的依賴項，接下來我們可以創建封裝數據傳輸代碼的組件了。該組件就叫做 Sync Adapter。在下節課中，我們將會展示如何將這一組件添加到應用中。
