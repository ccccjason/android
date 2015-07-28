# 創建 Stub 授權器

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/sync-adapters/creating-authenticator.html>

Sync Adapter 框架假定我們的 Sync Adapter 在同步數據時，設備存儲端關聯了一個賬戶，且服務器端需要進行登錄驗證。因此，我們需要提供一個叫做授權器（Authenticator）的組件作為 Sync Adapter 的一部分。該組件會集成在 Android 賬戶及認證框架中，並提供一個標準的接口來處理用戶憑據，比如登錄信息。

即使我們的應用不使用賬戶，我們仍然需要提供一個授權器組件。在這種情況下，授權器所處理的信息將被忽略，所以我們可以提供一個包含了方法存根（Stub Method）的授權器組件。同時我們需要提供一個綁定 [Service](http://developer.android.com/reference/android/app/Service.html)，來允許 Sync Adapter 框架調用授權器的方法。

這節課將展示如何定義一個能夠滿足 Sync Adapter 框架要求的 Stub 授權器。如果我們想要提供可以處理用戶賬戶的實際的授權器，可以閱讀：[AbstractAccountAuthenticator](http://developer.android.com/reference/android/accounts/AbstractAccountAuthenticator.html)。

## 添加一個 Stub 授權器組件

要在應用中添加一個 Stub 授權器，首先我們需要創建一個繼承 [AbstractAccountAuthenticator](http://developer.android.com/reference/android/accounts/AbstractAccountAuthenticator.html) 的類，在所有需要重寫的方法中，我們不進行任何處理，僅返回 null 或者拋出異常。

下面的代碼片段是一個 Stub 授權器的例子：

```java
/*
 * Implement AbstractAccountAuthenticator and stub out all
 * of its methods
 */
public class Authenticator extends AbstractAccountAuthenticator {
    // Simple constructor
    public Authenticator(Context context) {
        super(context);
    }
    // Editing properties is not supported
    @Override
    public Bundle editProperties(
            AccountAuthenticatorResponse r, String s) {
        throw new UnsupportedOperationException();
    }
    // Don't add additional accounts
    @Override
    public Bundle addAccount(
            AccountAuthenticatorResponse r,
            String s,
            String s2,
            String[] strings,
            Bundle bundle) throws NetworkErrorException {
        return null;
    }
    // Ignore attempts to confirm credentials
    @Override
    public Bundle confirmCredentials(
            AccountAuthenticatorResponse r,
            Account account,
            Bundle bundle) throws NetworkErrorException {
        return null;
    }
    // Getting an authentication token is not supported
    @Override
    public Bundle getAuthToken(
            AccountAuthenticatorResponse r,
            Account account,
            String s,
            Bundle bundle) throws NetworkErrorException {
        throw new UnsupportedOperationException();
    }
    // Getting a label for the auth token is not supported
    @Override
    public String getAuthTokenLabel(String s) {
        throw new UnsupportedOperationException();
    }
    // Updating user credentials is not supported
    @Override
    public Bundle updateCredentials(
            AccountAuthenticatorResponse r,
            Account account,
            String s, Bundle bundle) throws NetworkErrorException {
        throw new UnsupportedOperationException();
    }
    // Checking features for the account is not supported
    @Override
    public Bundle hasFeatures(
        AccountAuthenticatorResponse r,
        Account account, String[] strings) throws NetworkErrorException {
        throw new UnsupportedOperationException();
    }
}
```

## 將授權器綁定到框架

為了讓 Sync Adapter 框架可以訪問我們的授權器，我們必須為它創建一個綁定服務。這一服務提供一個 Android Binder 對象，允許框架調用我們的授權器，並且在授權器和框架間傳遞數據。

因為框架會在它第一次需要訪問授權器時啟動該 [Service](http://developer.android.com/reference/android/app/Service.html)，所以我們也可以使用該服務來實例化授權器。具體而言，我們需要在服務的 <a href="http://developer.android.com/reference/android/app/Service.html#onCreate()">Service.onCreate()</a> 方法中調用授權器的構造函數。

下面的代碼樣例展示瞭如何定義綁定 [Service](http://developer.android.com/reference/android/app/Service.html)：

```java
/**
 * A bound Service that instantiates the authenticator
 * when started.
 */
public class AuthenticatorService extends Service {
    ...
    // Instance field that stores the authenticator object
    private Authenticator mAuthenticator;
    @Override
    public void onCreate() {
        // Create a new authenticator object
        mAuthenticator = new Authenticator(this);
    }
    /*
     * When the system binds to this Service to make the RPC call
     * return the authenticator's IBinder.
     */
    @Override
    public IBinder onBind(Intent intent) {
        return mAuthenticator.getIBinder();
    }
}
```

## 添加授權器的元數據（Metadata）文件

若要將我們的授權器組件集成到 Sync Adapter 框架和賬戶框架中，我們需要為這些框架提供帶有描述組件信息的元數據。該元數據聲明瞭我們為 Sync Adapter 創建的賬戶類型以及系統所顯示的 UI 元素（如果希望用戶可以看到我們創建的賬戶類型）。在我們的項目目錄 `/res/xml/` 下，將元數據聲明於一個 XML 文件中。我們可以自己為該文件按命名，通常我們將它命名為 `authenticator.xml`。

在這個 XML 文件中，包含了一個 `<account-authenticator>` 標籤，它有下列一些屬性：

**android:accountType**

Sync Adapter 框架要求每一個適配器都有一個域名形式的賬戶類型。框架會將它作為 Sync Adapter 內部標識的一部分。如果服務端需要登陸，賬戶類型會和賬戶一起發送到服務端作為登錄憑據的一部分。

如果我們的服務端不需要登錄，我們仍然需要提供一個賬戶類型（該屬性的值用我們能控制的一個域名即可）。雖然框架會使用它來管理 Sync Adapter，但該屬性的值不會發送到服務端。

**android:icon**

指向一個包含圖標的 [Drawable](http://developer.android.com/guide/topics/resources/drawable-resource.html) 資源。如果我們在 `res/xml/syncadapter.xml` 中通過指定 `android:userVisible="true"` 讓 Sync Adapter 可見，那麼我們必須提供圖標資源。它會在系統的設置中的賬戶（Accounts）這一欄內顯示。

**android:smallIcon**

指向一個包含微小版本圖標的 [Drawable](http://developer.android.com/guide/topics/resources/drawable-resource.html) 資源。當屏幕尺寸較小時，這一資源可能會替代 `android:icon` 中所指定的圖標資源。

**android:label**

指明瞭用戶賬戶類型的本地化字符串。如果我們在 `res/xml/syncadapter.xml` 中通過指定 `android:userVisible="true"` 讓 Sync Adapter 可見，那麼我們需要提供該字符串。它會在系統的設置中的賬戶這一欄內顯示，就在我們為授權器定義的圖標旁邊。

下面的代碼樣例展示了我們之前為授權器創建的 XML 文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<account-authenticator
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:accountType="example.com"
        android:icon="@drawable/ic_launcher"
        android:smallIcon="@drawable/ic_launcher"
        android:label="@string/app_name"/>
```

## 在 Manifest 文件中聲明授權器

在之前的步驟中，我們已經創建了一個綁定服務，將授權器和 Sync Adapter 框架連接了起來。為了讓系統可以識別該服務，我們需要在 Manifest 文件中添加 [`<service>`](http://developer.android.com/guide/topics/manifest/service-element.html) 標籤，將它作為 [`<application>`](http://developer.android.com/guide/topics/manifest/application-element.html) 的子標籤：

```xml
    <service
            android:name="com.example.android.syncadapter.AuthenticatorService">
        <intent-filter>
            <action android:name="android.accounts.AccountAuthenticator"/>
        </intent-filter>
        <meta-data
            android:name="android.accounts.AccountAuthenticator"
            android:resource="@xml/authenticator" />
    </service>
```

[`<intent-filter>`](http://developer.android.com/guide/topics/manifest/intent-filter-element.html) 標籤配置了一個可以被 `android.accounts.AccountAuthenticator` 這一 Action 所激活的過濾器，這一 Intent 會在系統要運行授權器時由系統發出。當過濾器被激活後，系統會啟動 `AuthenticatorService`，即之前用來封裝授權器的 [Service](http://developer.android.com/reference/android/app/Service.html)。

[`<meta-data>`](http://developer.android.com/guide/topics/manifest/meta-data-element.html) 標籤聲明瞭授權器的元數據。[android:name](http://developer.android.com/guide/topics/manifest/meta-data-element.html#nm) 屬性將元數據和授權器框架連接起來。[android:resource](http://developer.android.com/guide/topics/manifest/meta-data-element.html#rsrc) 指定了我們之前所創建的授權器元數據文件的名字。

除了授權器之外，Sync Adapter 框架也需要一個 Content Provider。如果我們的應用並沒有使用 Content Provider，那麼可以閱讀下一節課程學習如何創建一個 Stub Content Provider；如果我們的應用已經使用了 ContentProvider，可以直接閱讀：[創建 Sync Adapter](create-sync-adapter.html)。
