# 創建 Sync Adpater

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/sync-adapters/creating-sync-adapter.html>

設備和服務器之間執行數據傳輸的代碼會封裝在應用的 Sync Adapter 組件中。Sync Adapter 框架會基於我們的調度和觸發操作，運行 Sync Adapter 組件中的代碼。要將同步適配組件添加到應用當中，我們需要添加下列部件：

Sync Adapter 類

  將我們的數據傳輸代碼封裝到一個與 Sync Adapter 框架兼容的接口當中。

綁定 [Service](http://developer.android.com/reference/android/app/Service.html)

  通過一個綁定服務，允許 Sync Adapter 框架運行 Sync Adapter 類中的代碼。

Sync Adapter 的 XML 元數據文件

  該文件包含了有關 Sync Adapter 的信息。框架會根據該文件確定應該如何加載並調度數據傳輸任務。

應用 manifest 清單文件的聲明

  需要在應用的 manifest 清單文件中聲明綁定服務；同時還需要指出 Sync Adapter 的元數據。

這節課將會向我們展示如何定義他們。

## 創建一個 Sync Adapter 類

在這部分課程中，我們將會學習如何創建封裝了數據傳輸代碼的 Sync Adapter 類。創建該類需要繼承 Sync Adapter 的基類；為該類定義構造函數；以及實現相關的方法。在這些方法中，我們定義數據傳輸任務。

### 繼承 Sync Adapter 基類：AbstractThreadedSyncAdapter

要創建 Sync Adapter 組件，首先繼承 [AbstractThreadedSyncAdapter](http://developer.android.com/reference/android/content/AbstractThreadedSyncAdapter.html)，然後編寫它的構造函數。與使用 <a href="http://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle)">Activity.onCreate()</a> 配置 Activity 時一樣，每次我們重新創建 Sync Adapter 組件的時候，使用構造函數執行相關的配置。例如，如果我們的應用使用一個 Content Provider 來存儲數據，那麼使用構造函數來獲取一個 [ContentResolver](http://developer.android.com/reference/android/content/ContentResolver.html) 實例。由於從 Android 3.0 開始添加了第二種形式的構造函數，來支持 `parallelSyncs` 參數，所以我們需要創建兩種形式的構造函數來保證兼容性。

> **Note：**Sync Adapter 框架是設計成和 Sync Adapter 組件的單例一起工作的。實例化 Sync Adapter 組件的更多細節，會在後面的章節中展開。

下面的代碼展示瞭如何實現 [AbstractThreadedSyncAdapter](http://developer.android.com/reference/android/content/AbstractThreadedSyncAdapter.html) 和它的構造函數：

```java
/**
 * Handle the transfer of data between a server and an
 * app, using the Android sync adapter framework.
 */
public class SyncAdapter extends AbstractThreadedSyncAdapter {
    ...
    // Global variables
    // Define a variable to contain a content resolver instance
    ContentResolver mContentResolver;
    /**
     * Set up the sync adapter
     */
    public SyncAdapter(Context context, boolean autoInitialize) {
        super(context, autoInitialize);
        /*
         * If your app uses a content resolver, get an instance of it
         * from the incoming Context
         */
        mContentResolver = context.getContentResolver();
    }
    ...
    /**
     * Set up the sync adapter. This form of the
     * constructor maintains compatibility with Android 3.0
     * and later platform versions
     */
    public SyncAdapter(
            Context context,
            boolean autoInitialize,
            boolean allowParallelSyncs) {
        super(context, autoInitialize, allowParallelSyncs);
        /*
         * If your app uses a content resolver, get an instance of it
         * from the incoming Context
         */
        mContentResolver = context.getContentResolver();
        ...
    }
```

### 在 onPerformSync() 中添加數據傳輸代碼

Sync Adapter 組件並不會自動地執行數據傳輸。它對我們的數據傳輸代碼進行封裝，使得 Sync Adapter 框架可以在後臺執行數據傳輸，而不會牽連到我們的應用。當框架準備同步我們的應用數據時，它會調用我們所實現的 <a href="http://developer.android.com/reference/android/content/AbstractThreadedSyncAdapter.html#onPerformSync(android.accounts.Account, android.os.Bundle, java.lang.String, android.content.ContentProviderClient, android.content.SyncResult)">onPerformSync()</a> 方法。

為了便於將數據從應用程序轉移到 Sync Adapter 組件中，Sync Adapter 框架調用 <a href="http://developer.android.com/reference/android/content/AbstractThreadedSyncAdapter.html#onPerformSync(android.accounts.Account, android.os.Bundle, java.lang.String, android.content.ContentProviderClient, android.content.SyncResult)">onPerformSync()</a>，它具有下面的參數：

Account

  該 [Account](http://developer.android.com/reference/android/accounts/Account.html) 對象與觸發 Sync Adapter 的事件相關聯。如果服務端不需要使用賬戶，那麼我們不需要使用這個對象內的信息。

Extras

  一個 Bundle 對象，它包含了一些標識，這些標識由觸發 Sync Adapter 的事件所發送。

Authority

  系統中某個 Content Provider 的 Authority。我們的應用必須要有訪問它的權限。通常，該 Authority 對應於應用的 Content Provider。

Content provider client

  [ContentProviderClient](http://developer.android.com/reference/android/content/ContentProviderClient.html) 針對於由 `Authority` 參數所指向的Content Provider。[ContentProviderClient](http://developer.android.com/reference/android/content/ContentProviderClient.html) 是一個 Content Provider 的輕量級共有接口。它的基本功能和 [ContentResolver](http://developer.android.com/reference/android/content/ContentResolver.html) 一樣。如果我們正在使用 Content Provider 來存儲應用數據，那麼我們可以利用它連接 Content Provider。反之，則將其忽略。

Sync result

  一個 [SyncResult](http://developer.android.com/reference/android/content/SyncResult.html) 對象，我們可以使用它將信息發送給 Sync Adapter 框架。

下面的代碼片段展示了 <a href="http://developer.android.com/reference/android/content/AbstractThreadedSyncAdapter.html#onPerformSync(android.accounts.Account, android.os.Bundle, java.lang.String, android.content.ContentProviderClient, android.content.SyncResult)">onPerformSync()</a> 函數的整體結構：

```java
    /*
     * Specify the code you want to run in the sync adapter. The entire
     * sync adapter runs in a background thread, so you don't have to set
     * up your own background processing.
     */
    @Override
    public void onPerformSync(
            Account account,
            Bundle extras,
            String authority,
            ContentProviderClient provider,
            SyncResult syncResult) {
    /*
     * Put the data transfer code here.
     */
    ...
    }
```

雖然實際的<a  href="http://developer.android.com/reference/android/content/AbstractThreadedSyncAdapter.html#onPerformSync(android.accounts.Account, android.os.Bundle, java.lang.String, android.content.ContentProviderClient, android.content.SyncResult)">onPerformSync()</a> 實現是要根據應用數據的同步需求以及服務器的連接協議來制定，但是我們的實現只需要執行一些常規任務：

連接到一個服務器

  儘管我們可以假定在開始傳輸數據時，已經獲取到了網絡連接，但是 Sync Adapter 框架並不會自動地連接到一個服務器。

下載和上傳數據

  Sync Adapter 不會自動執行數據傳輸。如果我們想要從服務器下載數據並將它存儲到 Content Provider 中，我們必須提供請求數據，下載數據和將數據插入到 Provider 中的代碼。類似地，如果我們想把數據發送到服務器，我們需要從一個文件，數據庫或者 Provider 中讀取數據，並且發送必需的上傳請求。同時我們還需要處理在執行數據傳輸時所發生的網絡錯誤。

處理數據衝突或者確定當前數據的狀態

  Sync Adapter 不會自動地解決服務器數據與設備數據之間的衝突。同時，它也不會自動檢測服務器上的數據是否比設備上的數據要新，反之亦然。因此，我們必須自己提供處理這些狀況的算法。

清理

  在數據傳輸的尾聲，記得要關閉網絡連接，清除臨時文件和緩存。

> **Note：**Sync Adapter 框架會在一個後臺線程中執行 <a href="http://developer.android.com/reference/android/content/AbstractThreadedSyncAdapter.html#onPerformSync(android.accounts.Account, android.os.Bundle, java.lang.String, android.content.ContentProviderClient, android.content.SyncResult)">onPerformSync()</a> 方法，所以我們不需要配置後臺處理任務。

除了和同步相關的任務之外，我們還應該嘗試將一些週期性的網絡相關的任務合併起來，並將它們添加到 <a href="http://developer.android.com/reference/android/content/AbstractThreadedSyncAdapter.html#onPerformSync(android.accounts.Account, android.os.Bundle, java.lang.String, android.content.ContentProviderClient, android.content.SyncResult)">onPerformSync()</a> 中。將所有網絡任務集中到該方法內處理，可以減少由啟動和停止網絡接口所造成的電量損失。有關更多如何在進行網絡訪問時更高效地使用電池方面的知識，可以閱讀：[Transferring Data Without Draining the Battery](../efficient-downloads/index.html)，它描述了一些在數據傳輸代碼中可以包含的網絡訪問任務。

## 將 Sync Adapter 綁定到框架上

現在，我們已經將數據傳輸代碼封裝在 Sync Adapter 組件中，但是我們必須讓框架可以訪問我們的代碼。為了做到這一點，我們需要創建一個綁定 [Service](http://developer.android.com/reference/android/app/Service.html)，它將一個特殊的 Android Binder 對象從 Sync Adapter 組件傳遞給框架。有了這一 Binder 對象，框架就可以調用 <a href="http://developer.android.com/reference/android/content/AbstractThreadedSyncAdapter.html#onPerformSync(android.accounts.Account, android.os.Bundle, java.lang.String, android.content.ContentProviderClient, android.content.SyncResult)">onPerformSync()</a> 方法並將數據傳遞給它。

在服務的 <a href="http://developer.android.com/reference/android/app/Service.html#onCreate()">onCreate()</a> 方法中將我們的 Sync Adapter 組件實例化為一個單例。通過在 <a href="http://developer.android.com/reference/android/app/Service.html#onCreate()">onCreate()</a> 方法中實例化該組件，我們可以推遲到服務啟動後再創建它，這會在框架第一次嘗試執行數據傳輸時發生。我們需要通過一種線程安全的方法來實例化組件，以防止 Sync Adapter 框架在響應觸發和調度時，形成含有多個 Sync Adapter 執行的隊列。

下面的代碼片段展示了我們應該如何實現一個綁定 [Service](http://developer.android.com/reference/android/app/Service.html) 的類，實例化我們的 Sync Adapter 組件，並獲取 Android Binder 對象：

```java
package com.example.android.syncadapter;
/**
 * Define a Service that returns an IBinder for the
 * sync adapter class, allowing the sync adapter framework to call
 * onPerformSync().
 */
public class SyncService extends Service {
    // Storage for an instance of the sync adapter
    private static SyncAdapter sSyncAdapter = null;
    // Object to use as a thread-safe lock
    private static final Object sSyncAdapterLock = new Object();
    /*
     * Instantiate the sync adapter object.
     */
    @Override
    public void onCreate() {
        /*
         * Create the sync adapter as a singleton.
         * Set the sync adapter as syncable
         * Disallow parallel syncs
         */
        synchronized (sSyncAdapterLock) {
            if (sSyncAdapter == null) {
                sSyncAdapter = new SyncAdapter(getApplicationContext(), true);
            }
        }
    }
    /**
     * Return an object that allows the system to invoke
     * the sync adapter.
     *
     */
    @Override
    public IBinder onBind(Intent intent) {
        /*
         * Get the object that allows external processes
         * to call onPerformSync(). The object is created
         * in the base class code when the SyncAdapter
         * constructors call super()
         */
        return sSyncAdapter.getSyncAdapterBinder();
    }
}
```

> **Note：**要看更多 Sync Adapter 綁定服務的例子，可以閱讀樣例代碼。

## 添加框架所需的賬戶

Sync Adapter 框架需要每個 Sync Adapter 擁有一個賬戶類型。在[創建 Stub 授權器](create-authenticator.html)章節中，我們已經聲明瞭賬戶類型的值。現在我們需要在 Android 系統中配置該賬戶類型。要配置賬戶類型，通過調用 <a href="http://developer.android.com/reference/android/accounts/AccountManager.html#addAccountExplicitly(android.accounts.Account, java.lang.String, android.os.Bundle)">addAccountExplicitly()</a> 添加一個使用其賬戶類型的虛擬賬戶。

調用該方法最合適的地方是在應用的啟動 Activity 的 <a href="http://developer.android.com/reference/android/app/Service.html#onCreate()">onCreate()</a> 方法中。如下面的代碼樣例所示：

```java
public class MainActivity extends FragmentActivity {
    ...
    ...
    // Constants
    // The authority for the sync adapter's content provider
    public static final String AUTHORITY = "com.example.android.datasync.provider"
    // An account type, in the form of a domain name
    public static final String ACCOUNT_TYPE = "example.com";
    // The account name
    public static final String ACCOUNT = "dummyaccount";
    // Instance fields
    Account mAccount;
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ...
        // Create the dummy account
        mAccount = CreateSyncAccount(this);
        ...
    }
    ...
    /**
     * Create a new dummy account for the sync adapter
     *
     * @param context The application context
     */
    public static Account CreateSyncAccount(Context context) {
        // Create the account type and default account
        Account newAccount = new Account(
                ACCOUNT, ACCOUNT_TYPE);
        // Get an instance of the Android account manager
        AccountManager accountManager =
                (AccountManager) context.getSystemService(
                        ACCOUNT_SERVICE);
        /*
         * Add the account and account type, no password or user data
         * If successful, return the Account object, otherwise report an error.
         */
        if (accountManager.addAccountExplicitly(newAccount, null, null))) {
            /*
             * If you don't set android:syncable="true" in
             * in your <provider> element in the manifest,
             * then call context.setIsSyncable(account, AUTHORITY, 1)
             * here.
             */
        } else {
            /*
             * The account exists or some other error occurred. Log this, report it,
             * or handle it internally.
             */
        }
    }
    ...
}
```

## 添加 Sync Adapter 的元數據文件

要將我們的 Sync Adapter 組件集成到框架中，我們需要向框架提供描述組件的元數據，以及額外的標識信息。元數據指定了我們為 Sync Adapter 所創建的賬戶類型，聲明瞭一個和應用相關聯的 Content Provider Authority，對和 Sync Adapter 相關的一部分系統用戶接口進行控制，同時還聲明瞭其它同步相關的標識。在我們項目的 `/res/xml/` 目錄下的一個特定文件內聲明這一元數據，我們可以為這個文件命名，不過通常來說我們將其命名為 `syncadapter.xml`。

在這一文件中包含了一個 XML 標籤 `<sync-adapter>`，它包含了下列的屬性字段：

`android:contentAuthority`

  Content Provider 的 URI Authority。如果我們在前一節課程中為應用創建了一個 Stub Content Provider，那麼請使用在 manifest 清單文件中添加在  [`<provider>`](http://developer.android.com/guide/topics/manifest/provider-element.html) 標籤內的 [android:authorities](http://developer.android.com/guide/topics/manifest/provider-element.html#auth) 屬性值。這一屬性的更多細節在本章後續章節中有更多的介紹。

  如果我們正使用 Sync Adapter 將數據從 Content Provider 傳輸到服務器上，該屬性的值應該和數據的 Content URI Authority 保持一致。這個值也是我們在 manifest 清單文件中添加在 [`<provider>`](http://developer.android.com/guide/topics/manifest/provider-element.html) 標籤內 `android:authorities` 屬性的值。

`android:accountType`

  Sync Adapter 框架所需要的賬戶類型。這個值必須和我們所創建的驗證器元數據文件內所提供的賬戶類型一致（詳細內容可以閱讀：[創建 Stub 授權器](create-authenticator.html)）。這也是在上一節的代碼片段中。常量 `ACCOUNT_TYPE` 的值。

配置相關屬性

  `android:userVisible`

  該屬性設置 Sync Adapter 框架的賬戶類型是否可見。默認地，和賬戶類型相關聯的賬戶圖標和標籤在系統設置的賬戶選項中可以看見，所以我們應該將 Sync Adapter 設置為對用戶不可見（除非我們確實擁有一個賬戶類型或者域名或者它們可以輕鬆地和我們的應用相關聯）。如果我們將賬戶類型設置為不可見，那麼我們仍然可以允許用戶通過一個 Activity 中的用戶接口來控制 Sync Adapter。

  `android:supportsUploading`

  允許我們將數據上傳到雲。如果應用僅僅下載數據，那麼請將該屬性設置為 `false`。

  `android:allowParallelSyncs`

  允許多個 Sync Adapter 組件的實例同時運行。如果應用支持多個用戶賬戶並且我們希望多個用戶並行地傳輸數據，那麼可以使用該特性。如果我們從不執行多個數據傳輸，那麼這個選項是沒用的。

  `android:isAlwaysSyncable`

  指明 Sync Adapter 框架可以在任何我們指定的時間運行 Sync Adapter。如果我們希望通過代碼來控制 Sync Adapter 的運行時機，請將該屬性設置為 `false`。然後調用 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#requestSync(android.accounts.Account, java.lang.String, android.os.Bundle)">requestSync()</a> 來運行 Sync Adapter。要學習更多關於運行 Sync Adapter 的知識，可以閱讀：[執行 Sync Adapter](running-sync-adapter.html)。

下面的代碼展示了應該如何通過 XML 配置一個使用單個虛擬賬戶，並且只執行下載的 Sync Adapter：

```xml
<?xml version="1.0" encoding="utf-8"?>
<sync-adapter
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:contentAuthority="com.example.android.datasync.provider"
        android:accountType="com.android.example.datasync"
        android:userVisible="false"
        android:supportsUploading="false"
        android:allowParallelSyncs="false"
        android:isAlwaysSyncable="true"/>
```

## 在 Manifest 清單文件中聲明 Sync Adapter

一旦我們將 Sync Adapter 組件集成到應用中，我們需要聲明相關的權限來使用它，並且還需要聲明我們所添加的綁定 [Service](http://developer.android.com/reference/android/app/Service.html)。

由於 Sync Adapter 組件會運行設備與網絡之間傳輸數據的代碼，所以我們需要請求使用網絡的權限。同時，我們的應用還需要讀寫 Sync Adapter 配置信息的權限，這樣我們才能通過應用中的其它組件去控制 Sync Adapter。另外，我們還需要一個特殊的權限，來允許應用使用我們在[創建 Stub 授權器](create-authenticator.html)中所創建的授權器組件。

要請求這些權限，將下列內容添加到應用 manifest 清單文件中，並作為 [`<manifest>`](http://developer.android.com/guide/topics/manifest/manifest-element.html) 標籤的子標籤：

[`android.permission.INTERNET`](http://developer.android.com/reference/android/Manifest.permission.html#INTERNET)

  允許 Sync Adapter 訪問網絡，使得它可以從設備下載和上傳數據到服務器。如果之前已經請求了該權限，那麼就不需要重複請求了。

[`android.permission.READ_SYNC_SETTINGS`](http://developer.android.com/reference/android/Manifest.permission.html#READ_SYNC_SETTINGS)

  允許應用讀取當前的 Sync Adapter 配置。例如，我們需要該權限來調用 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#getIsSyncable(android.accounts.Account, java.lang.String)">getIsSyncable()</a>。

[`android.permission.WRITE_SYNC_SETTINGS`](http://developer.android.com/reference/android/Manifest.permission.html#WRITE_SYNC_SETTINGS)

  允許我們的應用 對Sync Adapter 的配置進行控制。我們需要這一權限來通過 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#addPeriodicSync(android.accounts.Account, java.lang.String, android.os.Bundle, long)">addPeriodicSync()</a> 方法設置執行同步的時間間隔。另外，調用 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#requestSync(android.accounts.Account, java.lang.String, android.os.Bundle)">requestSync()</a> 方法不需要用到該權限。更多信息可以閱讀：[執行 Sync Adapter](running-sync-adapter.html)。

[`android.permission.AUTHENTICATE_ACCOUNTS`](http://developer.android.com/reference/android/Manifest.permission.html#AUTHENTICATE_ACCOUNTS)

  允許我們使用在[創建 Stub 授權器](create-authenticator.html)中所創建的驗證器組件。

下面的代碼片段展示瞭如何添加這些權限：

```xml
<manifest>
...
    <uses-permission
            android:name="android.permission.INTERNET"/>
    <uses-permission
            android:name="android.permission.READ_SYNC_SETTINGS"/>
    <uses-permission
            android:name="android.permission.WRITE_SYNC_SETTINGS"/>
    <uses-permission
            android:name="android.permission.AUTHENTICATE_ACCOUNTS"/>
...
</manifest>
```

最後，要聲明框架用來和 Sync Adapter 進行交互的綁定 [Service](http://developer.android.com/reference/android/app/Service.html)，添加下列的 XML 代碼到應用 manifest  清單文件中，作為 [`<application>`](http://developer.android.com/guide/topics/manifest/application-element.html) 標籤的子標籤：

```xml
        <service
                android:name="com.example.android.datasync.SyncService"
                android:exported="true"
                android:process=":sync">
            <intent-filter>
                <action android:name="android.content.SyncAdapter"/>
            </intent-filter>
            <meta-data android:name="android.content.SyncAdapter"
                    android:resource="@xml/syncadapter" />
        </service>
```

[`<intent-filter>`](http://developer.android.com/guide/topics/manifest/intent-filter-element.html) 標籤配置了一個過濾器，它會被帶有 `android.content.SyncAdapter` 這一 Action 的 Intent 所觸發，該 Intent 一般是由系統為了運行 Sync Adapter 而發出的。當過濾器被觸發後，系統會啟動我們所創建的綁定服務，在本例中它叫做 `SyncService`。屬性 [android:exported="true"](http://developer.android.com/guide/topics/manifest/service-element.html#exported) 允許我們應用之外的其它進程（包括系統）訪問這一 [Service](http://developer.android.com/reference/android/app/Service.html)。屬性 [android:process=":sync"](http://developer.android.com/guide/topics/manifest/service-element.html#proc) 告訴系統應該在一個全局共享的，且名字叫做 `sync` 的進程內運行該 [Service](http://developer.android.com/reference/android/app/Service.html)。如果我們的應用中有多個 Sync Adapter，那麼它們可以共享該進程，這有助於減少開銷。

[`<meta-data>`](http://developer.android.com/guide/topics/manifest/meta-data-element.html) 標籤提供了我們之前為 Sync Adapter 所創建的元數據 XML 文件的文件名。屬性 [android:name](http://developer.android.com/guide/topics/manifest/meta-data-element.html#nm) 指出這一元數據是針對於 Sync Adapter 框架的。而 [android:resource](http://developer.android.com/guide/topics/manifest/meta-data-element.html#rsrc) 標籤則指定了元數據文件的名稱。

現在我們已經為 Sync Adapter 準備好所有相關的組件了。下一節課將講授如何讓 Sync Adapter 框架運行 Sync Adapter。要實現這一點，既可以通過響應一個事件的方式，也可以通過執行一個週期性任務的方式。
