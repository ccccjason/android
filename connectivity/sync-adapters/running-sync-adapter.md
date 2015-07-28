# 執行 Sync Adpater

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/sync-adapters/running-sync-adapter.html>

在本節課之前，我們已經學習瞭如何創建一個封裝了數據傳輸代碼的 Sync Adapter 組件，以及如何添加其它的組件，使得我們可以將 Sync Adapter 集成到系統當中。現在我們已經擁有了所有部件，來安裝一個包含有 Sync Adapter 的應用了，但是這裡還沒有任何代碼是負責去運行 Sync Adapter。

執行 Sync Adapter 的時機，一般應該基於某個計劃任務或者一些事件的間接結果。例如，我們可能希望 Sync Adapter 以一個定期計劃任務的形式運行（比如每隔一段時間或者在每天的一個固定時間運行）。或者也可能希望當設備上的數據發生變化後，執行 Sync Adapter。我們應該避免將運行 Sync Adapter 作為用戶某個行為的直接結果，因為這樣做的話我們就無法利用 Sync Adapter 框架可以按計劃調度的特性。例如，我們應該在 UI 中避免使用刷新按鈕。

下列情況可以作為運行 Sync Adapter 的時機：

當服務端數據變更時：

  當服務端發送消息告知服務端數據發生變化時，運行 Sync Adapter 以響應這一來自服務端的消息。這一選項允許從服務器更新數據到設備上，該方法可以避免由於輪詢服務器所造成的執行效率下降，或者電量損耗。

當設備的數據變更時：

  當設備上的數據發生變化時，運行 Sync Adapter。這一選項允許我們將修改後的數據從設備發送給服務器。如果需要保證服務器端一直擁有設備上最新的數據，那麼這一選項非常有用。如果我們將數據存儲於 Content Provider，那麼這一選項的實現將會非常直接。如果使用的是一個 Stub Content Provider，檢測數據的變化可能會比較困難。

當系統發送了一個網絡消息：

  當 Android 系統發送了一個網絡消息來保持 TCP/IP 連接開啟時，運行 Sync Adapter。這個消息是網絡框架（Networking Framework）的一個基本部分。可以將這一選項作為自動運行 Sync Adapter 的一個方法。另外還可以考慮將它和基於時間間隔運行 Sync Adapter 的策略結合起來使用。

每隔一定時間：

  可以每隔一段指定的時間間隔後，運行 Sync Adapter，或者在每天的固定時間運行它。

根據需求：

  運行 Sync Adapter 以響應用戶的行為。然而，為了提供最佳的用戶體驗，我們應該主要依賴那些更加自動式的選項。使用自動式的選項，可以節省大量的電量以及網絡資源。

本課程的後續部分會詳細介紹每個選項。

## 當服務器數據變化時，運行 Sync Adapter

如果我們的應用從服務器傳輸數據，且服務器的數據會頻繁地發生變化，那麼可以使用一個 Sync Adapter 通過下載數據來響應服務端數據的變化。要運行 Sync Adapter，我們需要讓服務端嚮應用的 [BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html) 發送一條特殊的消息。為了響應這條消息，可以調用 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#requestSync(android.accounts.Account, java.lang.String, android.os.Bundle)">ContentResolver.requestSync()</a> 方法，向 Sync Adapter 框架發出信號，讓它運行 Sync Adapter。

谷歌雲消息（[Google Cloud Messaging](http://developer.android.com/google/gcm/index.html)，GCM）提供了我們需要的服務端組件和設備端組件，來讓上述消息系統能夠運行。使用 GCM 觸發數據傳輸比通過向服務器輪詢的方式要更加可靠，也更加有效。因為輪詢需要一個一直處於活躍狀態的 [Service](http://developer.android.com/reference/android/app/Service.html)，而 GCM 使用的 [BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html) 僅在消息到達時會被激活。另外，即使沒有更新的內容，定期的輪詢也會消耗大量的電池電量，而 GCM 僅在需要時才會發出消息。

> **Note：**如果我們使用 GCM，將廣播消息發送到所有安裝了我們的應用的設備，來激活 Sync Adapter。要記住他們會在同一時間（粗略地）收到我們的消息。這會導致在同一時段內有多個 Sync Adapter 的實例在運行，進而導致服務器和網絡的負載過重。要避免這一情況，我們應該考慮為不同的設備設定不同的 Sync Adapter 來延遲啟動時間。

下面的代碼展示瞭如何通過 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#requestSync(android.accounts.Account, java.lang.String, android.os.Bundle)">requestSync()</a> 響應一個接收到的 GCM 消息：

```java
public class GcmBroadcastReceiver extends BroadcastReceiver {
    ...
    // Constants
    // Content provider authority
    public static final String AUTHORITY = "com.example.android.datasync.provider"
    // Account type
    public static final String ACCOUNT_TYPE = "com.example.android.datasync";
    // Account
    public static final String ACCOUNT = "default_account";
    // Incoming Intent key for extended data
    public static final String KEY_SYNC_REQUEST =
            "com.example.android.datasync.KEY_SYNC_REQUEST";
    ...
    @Override
    public void onReceive(Context context, Intent intent) {
        // Get a GCM object instance
        GoogleCloudMessaging gcm =
                GoogleCloudMessaging.getInstance(context);
        // Get the type of GCM message
        String messageType = gcm.getMessageType(intent);
        /*
         * Test the message type and examine the message contents.
         * Since GCM is a general-purpose messaging system, you
         * may receive normal messages that don't require a sync
         * adapter run.
         * The following code tests for a a boolean flag indicating
         * that the message is requesting a transfer from the device.
         */
        if (GoogleCloudMessaging.MESSAGE_TYPE_MESSAGE.equals(messageType)
            &&
            intent.getBooleanExtra(KEY_SYNC_REQUEST)) {
            /*
             * Signal the framework to run your sync adapter. Assume that
             * app initialization has already created the account.
             */
            ContentResolver.requestSync(ACCOUNT, AUTHORITY, null);
            ...
        }
        ...
    }
    ...
}
```

## 當 Content Provider 的數據變化時，運行 Sync Adapter

如果我們的應用在一個 Content Provider 中收集數據，並且希望當我們更新了 Content Provider 的時候，同時更新服務器的數據，我們可以配置 Sync Adapter 來讓它自動運行。要做到這一點，首先應該為 Content Provider 註冊一個 Observer。當 Content Provider 的數據發生了變化之後，Content Provider 框架會調用 Observer。在 Observer 中，調用 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#requestSync(android.accounts.Account, java.lang.String, android.os.Bundle)">requestSync()</a> 來告訴框架現在應該運行 Sync Adapter 了。

> **Note：**如果我們使用的是一個 Stub Content Provider，那麼在 Content Provider 中不會有任何數據，並且不會調用 <a href="http://developer.android.com/reference/android/database/ContentObserver.html#onChange(boolean)">onChange()</a> 方法。在這種情況下，我們不得不提供自己的某種機制來檢測設備數據的變化。這一機制還要負責在數據發生變化時調用 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#requestSync(android.accounts.Account, java.lang.String, android.os.Bundle)">requestSync()</a>。

為了給 Content Provider 創建一個 Observer，繼承 [ContentObserver](http://developer.android.com/reference/android/database/ContentObserver.html) 類，並且實現 <a href="http://developer.android.com/reference/android/database/ContentObserver.html#onChange(boolean)">onChange()</a> 方法的兩種形式。在 <a href="http://developer.android.com/reference/android/database/ContentObserver.html#onChange(boolean)">onChange()</a> 中，調用 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#requestSync(android.accounts.Account, java.lang.String, android.os.Bundle)">requestSync()</a> 來啟動 Sync Adapter。

要註冊 Observer，需要將它作為參數傳遞給 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#registerContentObserver(android.net.Uri, boolean, android.database.ContentObserver)">registerContentObserver()</a>。在該方法中，我們還要傳遞一個我們想要監視的 Content URI。Content Provider 框架會將這個需要監視的 URI 與其它一些 Content URIs 進行比較，這些其它的 Content URIs 來自於 [ContentResolver](http://developer.android.com/reference/android/content/ContentResolver.html) 中那些可以修改 Provider 的方法（如 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#insert(android.net.Uri, android.content.ContentValues)">ContentResolver.insert()</a>）所傳入的參數。如果出現了變化，那麼我們所實現的 <a href="http://developer.android.com/reference/android/database/ContentObserver.html#onChange(boolean)">ContentObserver.onChange()</a> 將會被調用。

下面的代碼片段展示瞭如何定義一個 [ContentObserver](http://developer.android.com/reference/android/database/ContentObserver.html)，它在表數據發生變化後調用 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#requestSync(android.accounts.Account, java.lang.String, android.os.Bundle)">requestSync()</a>：

```java
public class MainActivity extends FragmentActivity {
    ...
    // Constants
    // Content provider scheme
    public static final String SCHEME = "content://";
    // Content provider authority
    public static final String AUTHORITY = "com.example.android.datasync.provider";
    // Path for the content provider table
    public static final String TABLE_PATH = "data_table";
    // Account
    public static final String ACCOUNT = "default_account";
    // Global variables
    // A content URI for the content provider's data table
    Uri mUri;
    // A content resolver for accessing the provider
    ContentResolver mResolver;
    ...
    public class TableObserver extends ContentObserver {
        /*
         * Define a method that's called when data in the
         * observed content provider changes.
         * This method signature is provided for compatibility with
         * older platforms.
         */
        @Override
        public void onChange(boolean selfChange) {
            /*
             * Invoke the method signature available as of
             * Android platform version 4.1, with a null URI.
             */
            onChange(selfChange, null);
        }
        /*
         * Define a method that's called when data in the
         * observed content provider changes.
         */
        @Override
        public void onChange(boolean selfChange, Uri changeUri) {
            /*
             * Ask the framework to run your sync adapter.
             * To maintain backward compatibility, assume that
             * changeUri is null.
            ContentResolver.requestSync(ACCOUNT, AUTHORITY, null);
        }
        ...
    }
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ...
        // Get the content resolver object for your app
        mResolver = getContentResolver();
        // Construct a URI that points to the content provider data table
        mUri = new Uri.Builder()
                  .scheme(SCHEME)
                  .authority(AUTHORITY)
                  .path(TABLE_PATH)
                  .build();
        /*
         * Create a content observer object.
         * Its code does not mutate the provider, so set
         * selfChange to "false"
         */
        TableObserver observer = new TableObserver(false);
        /*
         * Register the observer for the data table. The table's path
         * and any of its subpaths trigger the observer.
         */
        mResolver.registerContentObserver(mUri, true, observer);
        ...
    }
    ...
}
```

## 在一個網絡消息之後，運行 Sync Adapter

當可以獲得一個網絡連接時，Android 系統會每隔幾秒發送一條消息來保持 TCP/IP 連接處於開啟狀態。這一消息也會傳遞到每個應用的 [ContentResolver](http://developer.android.com/reference/android/content/ContentResolver.html) 中。通過調用 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#setSyncAutomatically(android.accounts.Account, java.lang.String, boolean)">setSyncAutomatically()</a>，我們可以在 [ContentResolver](http://developer.android.com/reference/android/content/ContentResolver.html) 收到消息後，運行 Sync Adapter。

每當網絡消息被髮送後運行 Sync Adapter，通過這樣的調度方式可以保證每次運行 Sync Adapter 時都可以訪問網絡。如果不是每次數據變化時就要以數據傳輸來響應，但是又希望自己的數據會被定期地更新，那麼我們可以用這一選項。類似地，如果我們不想要定期執行 Sync Adapter，但希望經常運行它，我們也可以使用這一選項。

由於 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#setSyncAutomatically(android.accounts.Account, java.lang.String, boolean)">setSyncAutomatically()</a> 方法不會禁用 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#addPeriodicSync(android.accounts.Account, java.lang.String, android.os.Bundle, long)">addPeriodicSync()</a>，所以 Sync Adapter 可能會在一小段時間內重複地被觸發激活。如果我們想要定期地運行 Sync Adapter，應該禁用 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#setSyncAutomatically(android.accounts.Account, java.lang.String, boolean)">setSyncAutomatically()</a>。

下面的代碼片段展示如何配置 [ContentResolver](http://developer.android.com/reference/android/content/ContentResolver.html)，利用它來響應網絡消息，從而運行 Sync Adapter：

```java
public class MainActivity extends FragmentActivity {
    ...
    // Constants
    // Content provider authority
    public static final String AUTHORITY = "com.example.android.datasync.provider";
    // Account
    public static final String ACCOUNT = "default_account";
    // Global variables
    // A content resolver for accessing the provider
    ContentResolver mResolver;
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ...
        // Get the content resolver for your app
        mResolver = getContentResolver();
        // Turn on automatic syncing for the default account and authority
        mResolver.setSyncAutomatically(ACCOUNT, AUTHORITY, true);
        ...
    }
    ...
}
```

## 定期地運行Sync Adapter

我們可以設置一個在運行之間的時間間隔來定期運行 Sync Adapter，或者在每天的固定時間運行它，還可以兩種策略同時使用。定期地運行 Sync Adapter 可以讓服務器的更新間隔大致保持一致。

同樣地，當服務器相對來說比較空閒時，我們可以通過在夜間定期調用 Sync Adapter，把設備上的數據上傳到服務器。大多數用戶在晚上不會關機，併為手機充電，所以這一方法是可行的。而且，通常來說，設備不會在深夜運行除了 Sync Adapter 之外的其他的任務。然而，如果我們使用這個方法的話，我們需要注意讓每臺設備在略微不同的時間觸發數據傳輸。如果所有設備在同一時間運行我們的 Sync Adapter，那麼我們的服務器和移動運營商的網絡將很有可能負載過重。

一般來說，當我們的用戶不需要實時更新，而希望定期更新時，使用定期運行的策咯會很有用。如果我們希望在數據的實時性和 Sync Adapter 的資源消耗之間進行一個平衡，那麼定期執行是一個不錯的選擇。

要定期運行我們的 Sync Adapter，調用 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#addPeriodicSync(android.accounts.Account, java.lang.String, android.os.Bundle, long)">addPeriodicSync()</a>。這樣每隔一段時間，Sync Adapter 就會運行。由於 Sync Adapter 框架會考慮其他 Sync Adapter 的執行，並嘗試最大化電池效率，所以間隔時間會動態地進行細微調整。同時，如果當前無法獲得網絡連接，框架不會運行我們的 Sync Adapter。

注意，<a href="http://developer.android.com/reference/android/content/ContentResolver.html#addPeriodicSync(android.accounts.Account, java.lang.String, android.os.Bundle, long)">addPeriodicSync()</a> 方法不會讓 Sync Adapter 每天在某個時間自動運行。要讓我們的 Sync Adapter 在每天的某個時刻自動執行，可以使用一個重複計時器作為觸發器。重複計時器的更多細節可以閱讀：[AlarmManager](http://developer.android.com/reference/android/app/AlarmManager.html)。如果我們使用 <a href="http://developer.android.com/reference/android/app/AlarmManager.html#setInexactRepeating(int, long, long, android.app.PendingIntent)">setInexactRepeating()</a> 方法設置了一個每天的觸發時刻會有粗略變化的觸發器，我們仍然應該將不同設備 Sync Adapter 的運行時間隨機化，使得它們的執行交錯開來。

<a href="http://developer.android.com/reference/android/content/ContentResolver.html#addPeriodicSync(android.accounts.Account, java.lang.String, android.os.Bundle, long)">addPeriodicSync()</a> 方法不會禁用 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#setSyncAutomatically(android.accounts.Account, java.lang.String, boolean)">setSyncAutomatically()</a>，所以我們可能會在一小段時間內產生多個 Sync Adapter 的運行實例。另外，僅有一部分 Sync Adapter 的控制標識可以在調用 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#addPeriodicSync(android.accounts.Account, java.lang.String, android.os.Bundle, long)">addPeriodicSync()</a> 時使用。不被允許的標識在該方法的<a href="http://developer.android.com/reference/android/content/ContentResolver.html#addPeriodicSync(android.accounts.Account,%20java.lang.String,%20android.os.Bundle,%20long)">文檔</a>中可以查看。

下面的代碼樣例展示瞭如何定期執行 Sync Adapter：

```java
public class MainActivity extends FragmentActivity {
    ...
    // Constants
    // Content provider authority
    public static final String AUTHORITY = "com.example.android.datasync.provider";
    // Account
    public static final String ACCOUNT = "default_account";
    // Sync interval constants
    public static final long SECONDS_PER_MINUTE = 60L;
    public static final long SYNC_INTERVAL_IN_MINUTES = 60L;
    public static final long SYNC_INTERVAL =
            SYNC_INTERVAL_IN_MINUTES *
            SECONDS_PER_MINUTE;
    // Global variables
    // A content resolver for accessing the provider
    ContentResolver mResolver;
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ...
        // Get the content resolver for your app
        mResolver = getContentResolver();
        /*
         * Turn on periodic syncing
         */
        ContentResolver.addPeriodicSync(
                ACCOUNT,
                AUTHORITY,
                Bundle.EMPTY,
                SYNC_INTERVAL);
        ...
    }
    ...
}
```

## 按需求執行 Sync Adapter

以響應用戶請求的方式運行 Sync Adapter 是最不推薦的策略。要知道，該框架是被特別設計的，它可以讓 Sync Adapter 在根據某個調度規則運行時，能夠儘量最高效地使用手機電量。顯然，在數據改變的時候執行同步可以更有效的使用手機電量，因為電量都消耗在了更新新的數據上。

相比之下，允許用戶按照自己的需求運行 Sync Adapter 意味著 Sync Adapter 會自己運行，這將無法有效地使用電量和網絡資源。如果根據需求執行同步，會誘導用戶即便沒有證據表明數據發生了變化也請求一個更新，這些無用的更新會導致對電量的低效率使用。一般來說，我們的應用應該使用其它信號來觸發一個同步更新或者讓它們定期地去執行，而不是依賴於用戶的輸入。

不過，如果我們仍然想要按照需求運行 Sync Adapter，可以將 Sync Adapter 的配置標識設置為手動執行，之後調用 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#requestSync(android.accounts.Account, java.lang.String, android.os.Bundle)">ContentResolver.requestSync()</a> 來觸發一次更新。

通過下列標識來執行按需求的數據傳輸：

[`SYNC_EXTRAS_MANUAL`](http://developer.android.com/reference/android/content/ContentResolver.html#SYNC_EXTRAS_MANUAL)

  強制執行手動的同步更新。Sync Adapter 框架會忽略當前的設置，比如通過 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#setSyncAutomatically(android.accounts.Account, java.lang.String, boolean)">setSyncAutomatically()</a> 方法設置的標識。

[`SYNC_EXTRAS_EXPEDITED`](http://developer.android.com/reference/android/content/ContentResolver.html#SYNC_EXTRAS_EXPEDITED)

  強制同步立即執行。如果我們不設置此項，系統可能會在運行同步請求之前等待一小段時間，因為它會嘗試將一小段時間內的多個請求集中在一起調度，目的是為了優化電量的使用。

下面的代碼片段將展示如何調用 <a href="http://developer.android.com/reference/android/content/ContentResolver.html#requestSync(android.accounts.Account, java.lang.String, android.os.Bundle)">requestSync()</a> 來響應一個按鈕點擊事件：

```java
public class MainActivity extends FragmentActivity {
    ...
    // Constants
    // Content provider authority
    public static final String AUTHORITY =
            "com.example.android.datasync.provider"
    // Account type
    public static final String ACCOUNT_TYPE = "com.example.android.datasync";
    // Account
    public static final String ACCOUNT = "default_account";
    // Instance fields
    Account mAccount;
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ...
        /*
         * Create the dummy account. The code for CreateSyncAccount
         * is listed in the lesson Creating a Sync Adapter
         */

        mAccount = CreateSyncAccount(this);
        ...
    }
    /**
     * Respond to a button click by calling requestSync(). This is an
     * asynchronous operation.
     *
     * This method is attached to the refresh button in the layout
     * XML file
     *
     * @param v The View associated with the method call,
     * in this case a Button
     */
    public void onRefreshButtonClick(View v) {
        ...
        // Pass the settings flags by inserting them in a bundle
        Bundle settingsBundle = new Bundle();
        settingsBundle.putBoolean(
                ContentResolver.SYNC_EXTRAS_MANUAL, true);
        settingsBundle.putBoolean(
                ContentResolver.SYNC_EXTRAS_EXPEDITED, true);
        /*
         * Request the sync for the default account, authority, and
         * manual sync settings
         */
        ContentResolver.requestSync(mAccount, AUTHORITY, settingsBundle);
    }
```
