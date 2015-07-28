# 分享文件

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/secure-file-sharing/sharing-file.html>

對應用程序進行配置，使得它可以使用Content URI來共享文件後，其就可以響應其他應用程序的獲取文件的請求了。一種響應這些請求的方法是在服務端應用程序提供一個可以由其他應用激活的文件選擇接口。該方法可以允許客戶端應用程序讓用戶從服務端應用程序選擇一個文件，然後接收這個文件的Content URI。

本課將會展示如何在應用中創建一個用於選擇文件的[Activity](http://developer.android.com/reference/android/app/Activity.html)，來響應這些獲取文件的請求。

## 接收文件請求

為了從客戶端應用程序接收一個文件獲取請求並以Content URI的形式進行響應，我們的應用程序應該提供一個選擇文件的[Activity](http://developer.android.com/reference/android/app/Activity.html)。客戶端應用程序通過調用<a href="http://developer.android.com/reference/android/app/Activity.html#startActivityForResult(android.content.Intent, int)">startActivityForResult()</a>方法啟動這一[Activity](http://developer.android.com/reference/android/app/Activity.html)。該方法包含了一個具有[ACTION_PICK](http://developer.android.com/reference/android/content/Intent.html#ACTION_PICK)Action的[Intent](http://developer.android.com/reference/android/content/Intent.html)參數。當客戶端應用程序調用了<a href="http://developer.android.com/reference/android/app/Activity.html#startActivityForResult(android.content.Intent, int)">startActivityForResult()</a>，我們的應用可以向客戶端應用程序返回一個結果，該結果即用戶所選擇的文件所對應的Content URI。

關於如何在客戶端應用程序實現文件獲取請求，請參考：[請求分享一個文件](request-file.html)。

## 創建一個選擇文件的Activity

為建立一個選擇文件的[Activity](http://developer.android.com/reference/android/app/Activity.html)，首先需要在Manifest清單文件中定義[Activity](http://developer.android.com/reference/android/app/Activity.html)，在其Intent過濾器中，匹配[ACTION_PICK](http://developer.android.com/reference/android/content/Intent.html#ACTION_PICK)Action及[CATEGORY_DEFAULT](http://developer.android.com/reference/android/content/Intent.html#CATEGORY_DEFAULT)和[CATEGORY_OPENABLE](http://developer.android.com/reference/android/content/Intent.html#CATEGORY_OPENABLE)這兩種Category。另外，還需要為應用程序設置MIME類型過濾器，來表明我們的應用程序可以向其他應用程序提供哪種類型的文件。下面這段代碼展示瞭如何在清單文件中定義新的[Activity](http://developer.android.com/reference/android/app/Activity.html)和Intent過濾器：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    ...
        <application>
        ...
            <activity
                android:name=".FileSelectActivity"
                android:label="File Selector" >
                <intent-filter>
                    <action
                        android:name="android.intent.action.PICK"/>
                    <category
                        android:name="android.intent.category.DEFAULT"/>
                    <category
                        android:name="android.intent.category.OPENABLE"/>
                    <data android:mimeType="text/plain"/>
                    <data android:mimeType="image/*"/>
                </intent-filter>
            </activity>
```

### 在代碼中定義文件選擇Activity

下面，定義一個[Activity](http://developer.android.com/reference/android/app/Activity.html)子類，用於顯示在內部存儲的“files/images/”目錄下可以獲得的文件，然後允許用戶選擇期望的文件。下面代碼展示瞭如何定義該[Activity](http://developer.android.com/reference/android/app/Activity.html)，並令其響應用戶的選擇：

```java
public class MainActivity extends Activity {
    // The path to the root of this app's internal storage
    private File mPrivateRootDir;
    // The path to the "images" subdirectory
    private File mImagesDir;
    // Array of files in the images subdirectory
    File[] mImageFiles;
    // Array of filenames corresponding to mImageFiles
    String[] mImageFilenames;
    // Initialize the Activity
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Set up an Intent to send back to apps that request a file
        mResultIntent =
                new Intent("com.example.myapp.ACTION_RETURN_FILE");
        // Get the files/ subdirectory of internal storage
        mPrivateRootDir = getFilesDir();
        // Get the files/images subdirectory;
        mImagesDir = new File(mPrivateRootDir, "images");
        // Get the files in the images subdirectory
        mImageFiles = mImagesDir.listFiles();
        // Set the Activity's result to null to begin with
        setResult(Activity.RESULT_CANCELED, null);
        /*
         * Display the file names in the ListView mFileListView.
         * Back the ListView with the array mImageFilenames, which
         * you can create by iterating through mImageFiles and
         * calling File.getAbsolutePath() for each File
         */
         ...
    }
    ...
}
```


## 響應一個文件選擇

一旦用戶選擇了一個被共享的文件，我們的應用程序必須明確哪個文件被選擇了，併為該文件生成一個對應的Content URI。如果我們的[Activity](http://developer.android.com/reference/android/app/Activity.html)在[ListView](http://developer.android.com/reference/android/widget/ListView.html)中顯示了可獲得文件的清單，那麼當用戶點擊了一個文件名時，系統會調用方法<a href="http://developer.android.com/reference/android/widget/AdapterView.OnItemClickListener.html#onItemClick(android.widget.AdapterView&lt;?&gt;, android.view.View, int, long)">onItemClick()</a>，在該方法中可以獲取被選擇的文件。

在<a href="http://developer.android.com/reference/android/widget/AdapterView.OnItemClickListener.html#onItemClick(android.widget.AdapterView&lt;?&gt;, android.view.View, int, long)">onItemClick()</a>中，根據被選中文件的文件名獲取一個[File](http://developer.android.com/reference/java/io/File.html)對象，然後將其作為參數傳遞給<a href="http://developer.android.com/reference/android/support/v4/content/FileProvider.html#getUriForFile(android.content.Context, java.lang.String, java.io.File)">getUriForFile()</a>，另外還需傳入的參數是在[`<provider>`](http://developer.android.com/guide/topics/manifest/provider-element.html)標籤中為[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)所指定的Authority，函數返回的Content URI包含了相應的Authority，一個對應於文件目錄的路徑標記（如在XML meta-data中定義的），以及包含擴展名的文件名。有關[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)如何基於XML meta-data將目錄路徑與路徑標記進行匹配的知識，可以閱讀：[指定可共享目錄路徑](setup-sharing.html#DefineMetaData)。

下例展示瞭如何檢測選中的文件並且獲得它的Content URI：

```java
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Define a listener that responds to clicks on a file in the ListView
        mFileListView.setOnItemClickListener(
                new AdapterView.OnItemClickListener() {
            @Override
            /*
             * When a filename in the ListView is clicked, get its
             * content URI and send it to the requesting app
             */
            public void onItemClick(AdapterView<?> adapterView,
                    View view,
                    int position,
                    long rowId) {
                /*
                 * Get a File for the selected file name.
                 * Assume that the file names are in the
                 * mImageFilename array.
                 */
                File requestFile = new File(mImageFilename[position]);
                /*
                 * Most file-related method calls need to be in
                 * try-catch blocks.
                 */
                // Use the FileProvider to get a content URI
                try {
                    fileUri = FileProvider.getUriForFile(
                            MainActivity.this,
                            "com.example.myapp.fileprovider",
                            requestFile);
                } catch (IllegalArgumentException e) {
                    Log.e("File Selector",
                          "The selected file can't be shared: " +
                          clickedFilename);
                }
                ...
            }
        });
        ...
    }
```

記住，我們能生成的那些Content URI所對應的文件，必須是那些在meta-data文件中包含`<paths>`標籤的目錄內的文件，這方面知識在[Specify Sharable Directories](http://developer.android.com/training/secure-file-sharing/setup-sharing.html#DefineMetaData)中已經討論過。如果調用<a href="http://developer.android.com/reference/android/support/v4/content/FileProvider.html#getUriForFile(android.content.Context, java.lang.String, java.io.File)">getUriForFile()</a>方法所要獲取的文件不在我們指定的目錄中，會收到一個[IllegalArgumentException](http://developer.android.com/reference/java/lang/IllegalArgumentException.html)。

## 為文件授權

現在已經有了想要共享給其他應用程序的文件所對應的Content URI，我們需要允許客戶端應用程序訪問這個文件。為了達到這一目的，可以通過將Content URI添加至一個[Intent](http://developer.android.com/reference/android/content/Intent.html)中，然後為該[Intent](http://developer.android.com/reference/android/content/Intent.html)設置權限標記。所授予的權限是臨時的，並且當接收文件的應用程序的任務棧終止後，會自動過期。

下例展示瞭如何為文件設置讀權限：

```java
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Define a listener that responds to clicks in the ListView
        mFileListView.setOnItemClickListener(
                new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView,
                    View view,
                    int position,
                    long rowId) {
                ...
                if (fileUri != null) {
                    // Grant temporary read permission to the content URI
                    mResultIntent.addFlags(
                        Intent.FLAG_GRANT_READ_URI_PERMISSION);
                }
                ...
             }
             ...
        });
    ...
    }
```

> **Caution：**調用<a href="http://developer.android.com/reference/android/content/Intent.html#setFlags(int)">setFlags()</a>來為文件授予臨時被訪問權限是唯一的安全的方法。儘量避免對文件的Content URI調用<a href="http://developer.android.com/reference/android/content/Context.html#grantUriPermission(java.lang.String, android.net.Uri, int)">Context.grantUriPermission()</a>，因為通過該方法授予的權限，只能通過調用<a href="http://developer.android.com/reference/android/content/Context.html#revokeUriPermission(android.net.Uri, int)">Context.revokeUriPermission()</a>來撤銷。

## 與請求應用共享文件

為了向請求文件的應用程序提供其需要的文件，我們將包含了Content URI和相應權限的[Intent](http://developer.android.com/reference/android/content/Intent.html)傳遞給<a href="http://developer.android.com/reference/android/app/Activity.html#setResult(int)">setResult()</a>。當定義的[Activity](http://developer.android.com/reference/android/app/Activity.html)結束後，系統會把這個包含了Content URI的[Intent](http://developer.android.com/reference/android/content/Intent.html)傳遞給客戶端應用程序。下例展示了其中的核心步驟：

```java
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Define a listener that responds to clicks on a file in the ListView
        mFileListView.setOnItemClickListener(
                new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView,
                    View view,
                    int position,
                    long rowId) {
                ...
                if (fileUri != null) {
                    ...
                    // Put the Uri and MIME type in the result Intent
                    mResultIntent.setDataAndType(
                            fileUri,
                            getContentResolver().getType(fileUri));
                    // Set the result
                    MainActivity.this.setResult(Activity.RESULT_OK,
                            mResultIntent);
                    } else {
                        mResultIntent.setDataAndType(null, "");
                        MainActivity.this.setResult(RESULT_CANCELED,
                                mResultIntent);
                    }
                }
        });
```

當用戶選擇好文件後，我們應該向用戶提供一個能夠立即回到客戶端應用程序的方法。一種實現的方法是向用戶提供一個勾選框或者一個完成按鈕。可以使用按鈕的[android:onClick](http://developer.android.com/reference/android/view/View.html#attr_android:onClick)屬性字段為它關聯一個方法。在該方法中，調用<a href="http://developer.android.com/reference/android/app/Activity.html#finish()">finish()</a>。例如：

```java
    public void onDoneClick(View v) {
        // Associate a method with the Done button
        finish();
    }
```
