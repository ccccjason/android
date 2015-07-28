# 獲取文件信息

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/secure-file-sharing/retrieve-info.html>

當一個客戶端應用程序擁有了文件的Content URI之後，它就可以獲取該文件並進行下一步的工作了，但在此之前，客戶端應用程序還可以向服務端應用程序獲取關於文件的信息，包括文件的數據類型和文件大小等等。數據類型可以幫助客戶端應用程序確定自己能否處理該文件，文件大小能幫助客戶端應用程序為文件設置合理的緩衝區。

本課將展示如何通過查詢服務端應用程序的[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)來獲取文件的MIME類型和文件大小。

## 獲取文件的MIME類型

客戶端應用程序可以通過文件的數據類型判斷自己應該如何處理這個文件的內容。客戶端應用程序可以通過調用<a href="http://developer.android.com/reference/android/content/ContentResolver.html#getType(android.net.Uri)">ContentResolver.getType()</a>方法獲得Content URI所對應的文件數據類型。該方法返回文件的MIME類型。默認情況下，一個[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)通過文件的後綴名來確定其MIME類型。

下例展示了當服務端應用程序將Content URI返回給客戶端應用程序後，客戶端應用程序應該如何獲取文件的MIMIE類型：

```java
    ...
    /*
     * Get the file's content URI from the incoming Intent, then
     * get the file's MIME type
     */
    Uri returnUri = returnIntent.getData();
    String mimeType = getContentResolver().getType(returnUri);
    ...
```

## 獲取文件名及文件大小
[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)類有一個<a href="http://developer.android.com/reference/android/support/v4/content/FileProvider.html#query(android.net.Uri, java.lang.String[], java.lang.String, java.lang.String[], java.lang.String)">query()</a>方法的默認實現，它返回一個[Cursor](http://developer.android.com/reference/android/database/Cursor.html)對象，該Cursor對象包含了Content URI所關聯的文件的名稱和大小。默認的實現返回下面兩列信息：

[**DISPLAY_NAME**](http://developer.android.com/reference/android/provider/OpenableColumns.html#DISPLAY_NAME)

文件名，[String](http://developer.android.com/reference/java/lang/String.html)類型。這個值和<a href="http://developer.android.com/reference/java/io/File.html#getName()">File.getName()</a>所返回的值一樣。

[**SIZE**](http://developer.android.com/reference/android/provider/OpenableColumns.html#SIZE)

文件大小，以字節為單位，long類型。這個值和<a href="http://developer.android.com/reference/java/io/File.html#length()">File.length()</a>所返回的值一樣。

客戶端應用可以通過將<a href="http://developer.android.com/reference/android/support/v4/content/FileProvider.html#query(android.net.Uri, java.lang.String[], java.lang.String, java.lang.String[], java.lang.String)">query()</a>的除了Content URI之外的其他參數都設置為“null”，來同時獲取文件的[名稱](http://developer.android.com/reference/android/provider/OpenableColumns.html#DISPLAY_NAME)（DISPLAY_NAME）和[大小](http://developer.android.com/reference/android/provider/OpenableColumns.html#SIZE)（SIZE）。例如，下面的代碼獲取一個文件的[名稱](http://developer.android.com/reference/android/provider/OpenableColumns.html#DISPLAY_NAME)和[大小](http://developer.android.com/reference/android/provider/OpenableColumns.html#SIZE)，然後在兩個[TextView](http://developer.android.com/reference/android/widget/TextView.html)中將他們顯示出來：

```java
    ...
    /*
     * Get the file's content URI from the incoming Intent,
     * then query the server app to get the file's display name
     * and size.
     */
    Uri returnUri = returnIntent.getData();
    Cursor returnCursor =
            getContentResolver().query(returnUri, null, null, null, null);
    /*
     * Get the column indexes of the data in the Cursor,
     * move to the first row in the Cursor, get the data,
     * and display it.
     */
    int nameIndex = returnCursor.getColumnIndex(OpenableColumns.DISPLAY_NAME);
    int sizeIndex = returnCursor.getColumnIndex(OpenableColumns.SIZE);
    returnCursor.moveToFirst();
    TextView nameView = (TextView) findViewById(R.id.filename_text);
    TextView sizeView = (TextView) findViewById(R.id.filesize_text);
    nameView.setText(returnCursor.getString(nameIndex));
    sizeView.setText(Long.toString(returnCursor.getLong(sizeIndex)));
    ...
```
