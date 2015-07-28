# 發送文件給其他設備

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/beam-files/sending-files.html>

這節課將展示如何通過Android Beam文件傳輸向另一臺設備發送大文件。要發送文件，首先應聲明使用NFC和外部存儲的權限，我們需要測試一下自己的設備是否支持NFC，這樣才能夠將文件的URI提供給Android Beam文件傳輸。

使用Android Beam文件傳輸功能必須滿足以下要求：

1. Android Beam文件傳輸功能傳輸大文件必須在Android 4.1（API Level 16）及以上版本的Android系統中使用。
2. 希望傳送的文件必須放置於外部存儲。更多關於外部存儲的知識，請參考：[Using the External Storage](http://developer.android.com/guide/topics/data/data-storage.html#filesExternal)。
3. 希望傳送的文件必須是全局可讀的。我們可以通過<a href="http://developer.android.com/reference/java/io/File.html#setReadable(boolean)">File.setReadable(true,false)</a>來為文件設置相應的讀權限。
4. 必須提供待傳輸文件的File URI。Android Beam文件傳輸無法處理由<a href="http://developer.android.com/reference/android/support/v4/content/FileProvider.html#getUriForFile(android.content.Context, java.lang.String, java.io.File)">FileProvider.getUriForFile</a>生成的Content URI。

## 在清單文件中聲明

首先，編輯Manifest清單文件來聲明應用程序所需要的權限和功能。

### 聲明權限

為了允許應用程序使用Android Beam文件傳輸控制NFC從外部存儲發送文件，必須在應用程序的Manifest清單文件中聲明下面的權限：

#### [NFC](http://developer.android.com/reference/android/Manifest.permission.html#NFC)
允許應用程序通過NFC發送數據。為聲明該權限，要添加下面的標籤作為一個[`<manifest>`](http://developer.android.com/guide/topics/manifest/manifest-element.html)標籤的子標籤：

```xml
<uses-permission android:name="android.permission.NFC" />
```

#### [READ_EXTERNAL_STORAGE](http://developer.android.com/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE)
允許應用讀取外部存儲。為聲明該權限，要添加下面的標籤作為一個[`<manifest>`](http://developer.android.com/guide/topics/manifest/manifest-element.html)標籤的子標籤：

```xml
<uses-permission
            android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

> **Note：**對於Android 4.2.2（API Level 17）及之前版本的系統，這個權限不是必需的。在後續版本的系統中，若應用程序需要讀取外部存儲，可能會需要申明該權限。為保證將來程序穩定性，建議在該權限申明變成必需的之前，先在清單文件中聲明。

### 指定NFC功能

通過添加[`<uses-feature>`](http://developer.android.com/guide/topics/manifest/uses-feature-element.html)標籤作為一個[`<manifest>`](http://developer.android.com/guide/topics/manifest/manifest-element.html)標籤的子標籤，指定我們的應用程序使用NFC。設置`android:required`屬性字段為`true`，使得我們的應用程序只有在NFC可以使用時才能運行。

下面的代碼展示瞭如何指定[`<uses-feature>`](http://developer.android.com/guide/topics/manifest/uses-feature-element.html)標籤：

```xml
<uses-feature
    android:name="android.hardware.nfc"
    android:required="true" />
```

注意，如果應用程序將NFC作為一個可選的功能，期望在NFC不可使用時程序還能繼續執行，我們就應該將`android:required`屬性字段設為`false`，然後在代碼中測試NFC的可用性。

### 指定Android Beam文件傳輸

由於Android Beam文件傳輸只能在Android 4.1（API Level 16）及以上的平臺使用，如果應用將Android Beam文件傳輸作為一個不可缺少的核心模塊，那麼我們必須指定[`<uses-sdk>`](http://developer.android.com/guide/topics/manifest/uses-sdk-element.html)標籤為：[android:minSdkVersion](http://developer.android.com/guide/topics/manifest/uses-sdk-element.html#min)="16"。或者可以將[android:minSdkVersion](http://developer.android.com/guide/topics/manifest/uses-sdk-element.html#min)設置為其它值，然後在代碼中測試平臺版本，這部分內容將在下一節中展開。

## 測試設備是否支持Android Beam文件傳輸

應使用以下標籤使得在Manifest清單文件中指定NFC是可選的：

```xml
<uses-feature android:name="android.hardware.nfc" android:required="false" />
```

如果設置了[android:required](http://developer.android.com/guide/topics/manifest/uses-feature-element.html#required)="false"，則我們必須在代碼中測試設備是否支持NFC和Android Beam文件傳輸。

為在代碼中測試是否支持Android Beam文件傳輸，我們先通過<a href="http://developer.android.com/reference/android/content/pm/PackageManager.html#hasSystemFeature(java.lang.String)">PackageManager.hasSystemFeature()</a>和參數[FEATURE_NFC](http://developer.android.com/reference/android/content/pm/PackageManager.html#FEATURE_NFC)測試設備是否支持NFC。下一步，通過[SDK_INT](http://developer.android.com/reference/android/os/Build.VERSION.html#SDK_INT)的值測試系統版本是否支持Android Beam文件傳輸。如果設備支持Android Beam文件傳輸，那麼獲得一個NFC控制器的實例，它能允許我們與NFC硬件進行通信，如下所示：

```java
public class MainActivity extends Activity {
    ...
    NfcAdapter mNfcAdapter;
    // Flag to indicate that Android Beam is available
    boolean mAndroidBeamAvailable  = false;
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // NFC isn't available on the device
        if (!PackageManager.hasSystemFeature(PackageManager.FEATURE_NFC)) {
            /*
             * Disable NFC features here.
             * For example, disable menu items or buttons that activate
             * NFC-related features
             */
            ...
        // Android Beam file transfer isn't supported
        } else if (Build.VERSION.SDK_INT <
                Build.VERSION_CODES.JELLY_BEAN_MR1) {
            // If Android Beam isn't available, don't continue.
            mAndroidBeamAvailable = false;
            /*
             * Disable Android Beam file transfer features here.
             */
            ...
        // Android Beam file transfer is available, continue
        } else {
        mNfcAdapter = NfcAdapter.getDefaultAdapter(this);
        ...
        }
    }
    ...
}
```

## 創建一個提供文件的回調函數

一旦確認了設備支持Android Beam文件傳輸，那麼可以添加一個回調函數，當Android Beam文件傳輸監測到用戶希望向另一個支持NFC的設備發送文件時，系統就會調用該函數。在該回調函數中，返回一個[Uri](http://developer.android.com/reference/android/net/Uri.html)對象數組，Android Beam文件傳輸會將URI對應的文件拷貝給要接收這些文件的設備。

要添加這個回調函數，需要實現[NfcAdapter.CreateBeamUrisCallback](http://developer.android.com/reference/android/nfc/NfcAdapter.CreateBeamUrisCallback.html)接口，和它的方法：<a href="http://developer.android.com/reference/android/nfc/NfcAdapter.CreateBeamUrisCallback.html#createBeamUris(android.nfc.NfcEvent)">createBeamUris()</a>，下面是一個例子：

```java
public class MainActivity extends Activity {
    ...
    // List of URIs to provide to Android Beam
    private Uri[] mFileUris = new Uri[10];
    ...
    /**
     * Callback that Android Beam file transfer calls to get
     * files to share
     */
    private class FileUriCallback implements
            NfcAdapter.CreateBeamUrisCallback {
        public FileUriCallback() {
        }
        /**
         * Create content URIs as needed to share with another device
         */
        @Override
        public Uri[] createBeamUris(NfcEvent event) {
            return mFileUris;
        }
    }
    ...
}
```

一旦實現了這個接口，通過調用<a href="http://developer.android.com/reference/android/nfc/NfcAdapter.html#setBeamPushUrisCallback(android.nfc.NfcAdapter.CreateBeamUrisCallback, android.app.Activity)">setBeamPushUrisCallback()</a>將回調函數提供給Android Beam文件傳輸。下面是一個例子：

```java
public class MainActivity extends Activity {
    ...
    // Instance that returns available files from this app
    private FileUriCallback mFileUriCallback;
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Android Beam file transfer is available, continue
        ...
        mNfcAdapter = NfcAdapter.getDefaultAdapter(this);
        /*
         * Instantiate a new FileUriCallback to handle requests for
         * URIs
         */
        mFileUriCallback = new FileUriCallback();
        // Set the dynamic callback for URI requests.
        mNfcAdapter.setBeamPushUrisCallback(mFileUriCallback,this);
        ...
    }
    ...
}
```

> **Note：**我們也可以將[Uri](http://developer.android.com/reference/android/net/Uri.html)對象數組通過應用程序的[NfcAdapter](http://developer.android.com/reference/android/nfc/NfcAdapter.html)實例，直接提供給NFC框架。如果能在NFC觸碰事件發生之前，定義這些URI，那麼可以選擇使用這個方法。更多關於這個方法的知識，請參考：<a href="http://developer.android.com/reference/android/nfc/NfcAdapter.html#setBeamPushUris(android.net.Uri[], android.app.Activity)">NfcAdapter.setBeamPushUris()</a>。

## 指定要發送的文件
為了將一或多個文件發送給其他支持NFC的設備，需要為每一個文件獲取一個File URI（一個具有文件格式（file scheme）的URI），然後將它們添加至一個[Uri](http://developer.android.com/reference/android/net/Uri.html)對象數組中。此外，要傳輸一個文件，我們必須也擁有該文件的讀權限。下例展示瞭如何根據文件名獲取其File URI，然後將URI添加至數組當中：

```java
    /*
     * Create a list of URIs, get a File,
     * and set its permissions
     */
    private Uri[] mFileUris = new Uri[10];
    String transferFile = "transferimage.jpg";
    File extDir = getExternalFilesDir(null);
    File requestFile = new File(extDir, transferFile);
    requestFile.setReadable(true, false);
    // Get a URI for the File and add it to the list of URIs
    fileUri = Uri.fromFile(requestFile);
    if (fileUri != null) {
        mFileUris[0] = fileUri;
    } else {
        Log.e("My Activity", "No File URI available for file.");
    }
```
