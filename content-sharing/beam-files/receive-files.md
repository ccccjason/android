# 接收其他設備的文件

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/beam-files/receive-files.html>

Android Beam文件傳輸將文件拷貝至接收設備上的某個特殊目錄。同時使用Android Media Scanner掃描拷貝的文件，並在[MediaStore](http://developer.android.com/reference/android/provider/MediaStore.html) provider中為媒體文件添加對應的條目記錄。本課將展示當文件拷貝完成時要如何響應，以及在接收設備上應該如何定位拷貝的文件。

## 響應請求並顯示數據

當Android Beam文件傳輸將文件拷貝至接收設備後，它會發佈一個包含[Intent](http://developer.android.com/reference/android/content/Intent.html)的通知，該Intent擁有：[ACTION_VIEW](http://developer.android.com/reference/android/content/Intent.html#ACTION_VIEW)，首個被傳輸文件的MIME類型，以及一個指向第一個文件的URI。用戶點擊該通知後，Intent會被髮送至系統。為了使我們的應用程序能夠響應該Intent，我們需要為響應的<a href="http://developer.android.com/reference/android/app/Activity.html">Activity</a>所對應的<a href="http://developer.android.com/guide/topics/manifest/activity-element.html">&lt;activity&gt;</a>標籤添加一個[`<intent-filter>`](http://developer.android.com/guide/topics/manifest/intent-filter-element.html)標籤，在[`<intent-filter>`](http://developer.android.com/guide/topics/manifest/intent-filter-element.html)標籤中，添加以下子標籤：

[`<action android:name="android.intent.action.VIEW" />`](http://developer.android.com/guide/topics/manifest/action-element.html)

該標籤用來匹配從通知發出的Intent，這些Intent具有[ACTION_VIEW](http://developer.android.com/reference/android/content/Intent.html#ACTION_VIEW)這一Action。

[`<category android:name="android.intent.category.CATEGORY_DEFAULT" />`](http://developer.android.com/guide/topics/manifest/category-element.html)

該標籤用來匹配不含有顯式Category的[Intent](http://developer.android.com/reference/android/content/Intent.html)對象。

[`<data android:mimeType="mime-type" />`](http://developer.android.com/guide/topics/manifest/data-element.html)

該標籤用來匹配一個MIME類型。僅僅指定那些我們的應用能夠處理的類型。

下例展示瞭如何添加一個intent filter來激活我們的activity：

```xml
    <activity
        android:name="com.example.android.nfctransfer.ViewActivity"
        android:label="Android Beam Viewer" >
        ...
        <intent-filter>
            <action android:name="android.intent.action.VIEW"/>
            <category android:name="android.intent.category.DEFAULT"/>
            ...
        </intent-filter>
    </activity>
```

> **Note：**Android Beam文件傳輸不是含有[ACTION_VIEW](http://developer.android.com/reference/android/content/Intent.html#ACTION_VIEW)的Intent的唯一可能發送者。在接收設備上的其它應用也有可能會發送含有該Action的intent。我們馬上會進一步討論這一問題。

## 請求文件讀權限
要讀取Android Beam文件傳輸所拷貝到設備上的文件，需要請求[READ_EXTERNAL_STORAGE](http://developer.android.com/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE)權限。例如：

```xml
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

如果希望將文件拷貝至應用程序自己的存儲區，那麼需要的權限改為[WRITE_EXTERNAL_STORAGE](http://developer.android.com/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE)，另外，[WRITE_EXTERNAL_STORAGE](http://developer.android.com/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE)權限包含了[READ_EXTERNAL_STORAGE](http://developer.android.com/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE)權限。

> **Note：**對於Android 4.2.2（API Level 17）及之前版本的系統，[READ_EXTERNAL_STORAGE](http://developer.android.com/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE)權限僅在用戶選擇要讀文件時才是強制需要的。而在今後的版本中會在所有情況下都需要該權限。為保證應用程序在未來的穩定性，建議在Manifest清單文件中聲明該權限。

由於我們的應用對於自身的內部存儲區域具有控制權，因此當要將文件拷貝至應用程序自身的的內部存儲區域時，不需要聲明寫權限。

## 獲取拷貝文件的目錄

Android Beam文件傳輸一次性將所有文件拷貝到目標設備的一個目錄中，Android Beam文件傳輸通知所發出的[Intent](http://developer.android.com/reference/android/content/Intent.html)中含有指向了第一個被傳輸的文件的URI。然而，我們的應用程序也有可能接收到除了Android Beam文件傳輸之外的某個來源所發出的含有[ACTION_VIEW](http://developer.android.com/reference/android/content/Intent.html#ACTION_VIEW)這一Action的Intent。為了明確應該如何處理接收的Intent，我們要檢查它的Scheme和Authority。

可以調用<a href="http://developer.android.com/reference/android/net/Uri.html#getScheme()">Uri.getScheme()</a>獲得URI的Scheme，下例展示瞭如何確定Scheme並對URI進行相應的處理：

```java
public class MainActivity extends Activity {
    ...
    // A File object containing the path to the transferred files
    private File mParentPath;
    // Incoming Intent
    private Intent mIntent;
    ...
    /*
     * Called from onNewIntent() for a SINGLE_TOP Activity
     * or onCreate() for a new Activity. For onNewIntent(),
     * remember to call setIntent() to store the most
     * current Intent
     *
     */
    private void handleViewIntent() {
        ...
        // Get the Intent action
        mIntent = getIntent();
        String action = mIntent.getAction();
        /*
         * For ACTION_VIEW, the Activity is being asked to display data.
         * Get the URI.
         */
        if (TextUtils.equals(action, Intent.ACTION_VIEW)) {
            // Get the URI from the Intent
            Uri beamUri = mIntent.getData();
            /*
             * Test for the type of URI, by getting its scheme value
             */
            if (TextUtils.equals(beamUri.getScheme(), "file")) {
                mParentPath = handleFileUri(beamUri);
            } else if (TextUtils.equals(
                    beamUri.getScheme(), "content")) {
                mParentPath = handleContentUri(beamUri);
            }
        }
        ...
    }
    ...
}
```

### 從File URI中獲取目錄

如果接收的[Intent](http://developer.android.com/reference/android/content/Intent.html)包含一個File URI，則該URI包含了一個文件的絕對文件名，它包括了完整的路徑和文件名。對Android Beam文件傳輸來說，目錄路徑指向了其它被傳輸文件的位置（如果有其它傳輸文件的話），要獲得該目錄路徑，需要取得URI的路徑部分（URI中除去“file:”前綴的部分），根據路徑創建一個[File](http://developer.android.com/reference/java/io/File.html)對象，然後獲取這個[File](http://developer.android.com/reference/java/io/File.html)的父目錄：

```java
    ...
    public String handleFileUri(Uri beamUri) {
        // Get the path part of the URI
        String fileName = beamUri.getPath();
        // Create a File object for this filename
        File copiedFile = new File(fileName);
        // Get a string containing the file's parent directory
        return copiedFile.getParent();
    }
    ...
```

### 從Content URI獲取目錄

如果接收的[Intent](http://developer.android.com/reference/android/content/Intent.html)包含一個Content URI，這個URI可能指向的是存儲於[MediaStore](http://developer.android.com/reference/android/provider/MediaStore.html) Content Provider的目錄和文件名。我們可以通過檢測URI的Authority值來判斷它是否是來自於[MediaStore](http://developer.android.com/reference/android/provider/MediaStore.html)的Content URI。一個[MediaStore](http://developer.android.com/reference/android/provider/MediaStore.html)的Content URI可能來自Android Beam文件傳輸也可能來自其它應用程序，但不管怎麼樣，我們都能根據該Content URI獲得一個目錄路徑和文件名。

我們也可以接收一個含有[ACTION_VIEW](http://developer.android.com/reference/android/content/Intent.html#ACTION_VIEW)這一Action的Intent，它包含的Content URI針對於Content Provider，而不是[MediaStore](http://developer.android.com/reference/android/provider/MediaStore.html)，這種情況下，該Content URI不包含[MediaStore](http://developer.android.com/reference/android/provider/MediaStore.html)的Authority，且這個URI一般不指向一個目錄。

> **Note：**對於Android Beam文件傳輸，接收在含有[ACTION_VIEW](http://developer.android.com/reference/android/content/Intent.html#ACTION_VIEW)的Intent中的Content URI時，若第一個接收的文件MIME類型為“audio/*”，“image/*”或者“video/*”，Android Beam文件傳輸會在它存儲傳輸文件的目錄內運行Media Scanner，以此為媒體文件添加索引。同時Media Scanner將結果寫入[MediaStore](http://developer.android.com/reference/android/provider/MediaStore.html)的Content Provider，之後它將第一個文件的Content URI回遞給Android Beam文件傳輸。這個Content URI就是我們在通知[Intent](http://developer.android.com/reference/android/content/Intent.html)中所接收到的。要獲得第一個文件的目錄，需要使用該Content URI從[MediaStore](http://developer.android.com/reference/android/provider/MediaStore.html)中獲取它。

### 確定Content Provider

為了確定是否能從Content URI中獲取文件目錄，可以通過調用<a href="http://developer.android.com/reference/android/net/Uri.html#getAuthority()">Uri.getAuthority()</a>獲取URI的Authority，以此確定與該URI相關聯的Content Provider。其結果有兩個可能的值：

**[MediaStore.AUTHORITY](http://developer.android.com/reference/android/provider/MediaStore.html#AUTHORITY)**

表明該URI關聯了被[MediaStore](http://developer.android.com/reference/android/provider/MediaStore.html)記錄的一個文件或者多個文件。可以從[MediaStore](http://developer.android.com/reference/android/provider/MediaStore.html)中獲取文件的全名，目錄名就自然可以從文件全名中獲取。

**其他值**

來自其他Content Provider的Content URI。可以顯示與該Content URI相關聯的數據，但是不要嘗試去獲取文件目錄。

要從[MediaStore](http://developer.android.com/reference/android/provider/MediaStore.html)的Content URI中獲取目錄，我們需要執行一個查詢操作，它將[Uri](http://developer.android.com/reference/android/net/Uri.html)參數指定為收到的ContentURI，將[MediaColumns.DATA](http://developer.android.com/reference/android/provider/MediaStore.MediaColumns.html#DATA)列作為投影（Projection）。返回的[Cursor](http://developer.android.com/reference/android/database/Cursor.html)對象包含了URI所代表的文件的完整路徑和文件名。該目錄路徑下還包含了由Android Beam文件傳輸傳送到該設備上的其它文件。

下面的代碼展示瞭如何測試Content URI的Authority，並獲取傳輸文件的路徑和文件名：

```java
    ...
    public String handleContentUri(Uri beamUri) {
        // Position of the filename in the query Cursor
        int filenameIndex;
        // File object for the filename
        File copiedFile;
        // The filename stored in MediaStore
        String fileName;
        // Test the authority of the URI
        if (!TextUtils.equals(beamUri.getAuthority(), MediaStore.AUTHORITY)) {
            /*
             * Handle content URIs for other content providers
             */
        // For a MediaStore content URI
        } else {
            // Get the column that contains the file name
            String[] projection = { MediaStore.MediaColumns.DATA };
            Cursor pathCursor =
                    getContentResolver().query(beamUri, projection,
                    null, null, null);
            // Check for a valid cursor
            if (pathCursor != null &&
                    pathCursor.moveToFirst()) {
                // Get the column index in the Cursor
                filenameIndex = pathCursor.getColumnIndex(
                        MediaStore.MediaColumns.DATA);
                // Get the full file name including path
                fileName = pathCursor.getString(filenameIndex);
                // Create a File object for the filename
                copiedFile = new File(fileName);
                // Return the parent directory of the file
                return new File(copiedFile.getParent());
             } else {
                // The query didn't work; return null
                return null;
             }
        }
    }
    ...
```

更多關於從Content Provider獲取數據的知識，請參考：[Retrieving Data from the Provider](http://developer.android.com/guide/topics/providers/content-provider-basics.html#SimpleQuery)。
