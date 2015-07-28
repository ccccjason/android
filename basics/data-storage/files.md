# 保存到文件

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/basics/data-storage/files.html>

Android使用與其他平臺類似的基於磁盤的文件系統(disk-based file systems)。本課程將描述如何在Android文件系統上使用 [File](http://developer.android.com/reference/java/io/File.html) 的讀寫APIs對Andorid的file system進行讀寫。

File 對象非常適合於流式順序數據的讀寫。如圖片文件或是網絡中交換的數據等。

本課程將會演示如何在app中執行基本的文件相關操作。假定讀者已對linux的文件系統及[java.io](http://developer.android.com/reference/java/io/package-summary.html)中標準的I/O APIs有一定認識。

## 存儲在內部還是外部

所有的Android設備均有兩個文件存儲區域："internal" 與 "external" 。 這兩個名稱來自於早先的Android系統，當時大多設備都內置了不可變的內存（internal storage)及一個類似於SD card（external storage）這樣的可卸載的存儲部件。之後有一些設備將"internal" 與 "external" 都做成了不可卸載的內置存儲，雖然如此，但是這一整塊還是從邏輯上有被劃分為"internal"與"external"的。只是現在不再以是否可卸載進行區分了。 下面列出了兩者的區別：

* **Internal storage:**
	* 總是可用的
	* 這裡的文件默認只能被我們的app所訪問。
	* 當用戶卸載app的時候，系統會把internal內該app相關的文件都清除乾淨。
	* Internal是我們在想確保不被用戶與其他app所訪問的最佳存儲區域。

* **External storage:**
	* 並不總是可用的，因為用戶有時會通過USB存儲模式掛載外部存儲器，當取下掛載的這部分後，就無法對其進行訪問了。
	* 是大家都可以訪問的，因此保存在這裡的文件可能被其他程序訪問。
	* 當用戶卸載我們的app時，系統僅僅會刪除external根目錄（<a href="http://developer.android.com/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)">getExternalFilesDir()</a>）下的相關文件。
	* External是在不需要嚴格的訪問權限並且希望這些文件能夠被其他app所共享或者是允許用戶通過電腦訪問時的最佳存儲區域。

> **Tip:** 儘管app是默認被安裝到internal storage的，我們還是可以通過在程序的manifest文件中聲明[android:installLocation](http://developer.android.com/guide/topics/manifest/manifest-element.html#install) 屬性來指定程序安裝到external storage。當某個程序的安裝文件很大且用戶的external storage空間大於internal storage時，用戶會傾向於將該程序安裝到external storage。更多安裝信息見[App Install Location](http://developer.android.com/guide/topics/data/install-location.html)。

## 獲取External存儲的權限

為了寫數據到external storage, 必須在你[manifest文件](http://developer.android.com/guide/topics/manifest/manifest-intro.html)中請求[WRITE_EXTERNAL_STORAGE](http://developer.android.com/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE)權限：

```xml
<manifest ...>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    ...
</manifest>
```

> **Caution:**目前，所有的apps都可以在不指定某個專門的權限下做**讀**external storage的動作。但這在以後的安卓版本中會有所改變。如果我們的app只需要**讀**的權限(不是寫), 那麼將需要聲明 [READ_EXTERNAL_STORAGE](http://developer.android.com/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE) 權限。為了確保app能持續地正常工作，我們現在在編寫程序時就需要聲明讀權限。
```xml
<manifest ...>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    ...
</manifest>
```
但是，如果我們的程序有聲明**[WRITE_EXTERNAL_STORAGE](http://developer.android.com/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE) **權限，那麼就默認有了**讀**的權限。

對於internal storage，我們不需要聲明任何權限，因為程序默認就有讀寫程序目錄下的文件的權限。

## 保存到Internal Storage

當保存文件到internal storage時，可以通過執行下面兩個方法之一來獲取合適的目錄作為 [FILE](http://developer.android.com/reference/java/io/File.html) 的對象：

* <a href="http://developer.android.com/reference/android/content/Context.html#getFilesDir()">getFilesDir()</a> : 返回一個[File](http://developer.android.com/reference/java/io/File.html)，代表了我們app的internal目錄。
* <a href="http://developer.android.com/reference/android/content/Context.html#getCacheDir()">getCacheDir()</a> : 返回一個[File](http://developer.android.com/reference/java/io/File.html)，代表了我們app的internal緩存目錄。請確保這個目錄下的文件能夠在一旦不再需要的時候馬上被刪除，並對其大小進行合理限制，例如1MB 。系統的內部存儲空間不夠時，會自行選擇刪除緩存文件。

可以使用<a href="http://developer.android.com/reference/java/io/File.html#File(java.io.File, java.lang.String)">File()</a> 構造器在那些目錄下創建一個新的文件，如下：

```java
File file = new File(context.getFilesDir(), filename);
```

同樣，也可以執行<a href="http://developer.android.com/reference/android/content/Context.html#openFileOutput(java.lang.String, int)">openFileOutput()</a> 獲取一個 [FileOutputStream](http://developer.android.com/reference/java/io/FileOutputStream.html) 用於寫文件到internal目錄。如下：

```java
String filename = "myfile";
String string = "Hello world!";
FileOutputStream outputStream;

try {
  outputStream = openFileOutput(filename, Context.MODE_PRIVATE);
  outputStream.write(string.getBytes());
  outputStream.close();
} catch (Exception e) {
  e.printStackTrace();
}
```

如果需要緩存一些文件，可以使用<a href="http://developer.android.com/reference/java/io/File.html#createTempFile(java.lang.String, java.lang.String)">createTempFile()</a>。例如：下面的方法從[URL](http://developer.android.com/reference/java/net/URL.html)中抽取了一個文件名，然後再在程序的internal緩存目錄下創建了一個以這個文件名命名的文件。

```java
 public File getTempFile(Context context, String url) {
    File file;
    try {
        String fileName = Uri.parse(url).getLastPathSegment();
        file = File.createTempFile(fileName, null, context.getCacheDir());
    catch (IOException e) {
        // Error while creating file
    }
    return file;
}
```

> **Note:** 我們的app的internal storage 目錄以app的包名作為標識存放在Android文件系統的特定目錄下[data/data/com.example.xx]。 從技術上講，如果文件被設置為可讀的，那麼其他app就可以讀取該internal文件。然而，其他app需要知道包名與文件名。若沒有設置為可讀或者可寫，其他app是沒有辦法讀寫的。因此我們只要使用了[MODE_PRIVATE](http://developer.android.com/reference/android/content/Context.html#MODE_PRIVATE) ，那麼這些文件就不可能被其他app所訪問。

## 保存文件到External Storage

因為external storage可能是不可用的，比如遇到SD卡被拔出等情況時。因此在訪問之前應對其可用性進行檢查。我們可以通過執行 <a href="http://developer.android.com/reference/android/os/Environment.html#getExternalStorageState()">getExternalStorageState()</a>來查詢external storage的狀態。若返回狀態為[MEDIA_MOUNTED](http://developer.android.com/reference/android/os/Environment.html#MEDIA_MOUNTED), 則可以讀寫。示例如下：

```java
 /* Checks if external storage is available for read and write */
public boolean isExternalStorageWritable() {
    String state = Environment.getExternalStorageState();
    if (Environment.MEDIA_MOUNTED.equals(state)) {
        return true;
    }
    return false;
}

/* Checks if external storage is available to at least read */
public boolean isExternalStorageReadable() {
    String state = Environment.getExternalStorageState();
    if (Environment.MEDIA_MOUNTED.equals(state) ||
        Environment.MEDIA_MOUNTED_READ_ONLY.equals(state)) {
        return true;
    }
    return false;
}
```

儘管external storage對於用戶與其他app是可修改的，我們可能會保存下面兩種類型的文件。

* **Public files** :這些文件對與用戶與其他app來說是public的，當用戶卸載我們的app時，這些文件應該保留。例如，那些被我們的app拍攝的圖片或者下載的文件。
* **Private files**: 這些文件完全被我們的app所私有，它們應該在app被卸載時刪除。儘管由於存儲在external storage，那些文件從技術上而言可以被用戶與其他app所訪問，但實際上那些文件對於其他app沒有任何意義。因此，當用戶卸載我們的app時，系統會刪除其下的private目錄。例如，那些被我們的app下載的緩存文件。

想要將文件以public形式保存在external storage中，請使用<a href="http://developer.android.com/reference/android/os/Environment.html#getExternalStoragePublicDirectory(java.lang.String)">getExternalStoragePublicDirectory()</a>方法來獲取一個 File 對象，該對象表示存儲在external storage的目錄。這個方法會需要帶有一個特定的參數來指定這些public的文件類型，以便於與其他public文件進行分類。參數類型包括[DIRECTORY_MUSIC](http://developer.android.com/reference/android/os/Environment.html#DIRECTORY_MUSIC) 或者 [DIRECTORY_PICTURES](http://developer.android.com/reference/android/os/Environment.html#DIRECTORY_PICTURES). 如下:

```java
public File getAlbumStorageDir(String albumName) {
    // Get the directory for the user's public pictures directory.
    File file = new File(Environment.getExternalStoragePublicDirectory(
            Environment.DIRECTORY_PICTURES), albumName);
    if (!file.mkdirs()) {
        Log.e(LOG_TAG, "Directory not created");
    }
    return file;
}
```

想要將文件以private形式保存在external storage中，可以通過執行<a href="http://developer.android.com/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)">getExternalFilesDir()</a> 來獲取相應的目錄，並且傳遞一個指示文件類型的參數。每一個以這種方式創建的目錄都會被添加到external storage封裝我們app目錄下的參數文件夾下（如下則是albumName）。這下面的文件會在用戶卸載我們的app時被系統刪除。如下示例：

```java
public File getAlbumStorageDir(Context context, String albumName) {
    // Get the directory for the app's private pictures directory.
    File file = new File(context.getExternalFilesDir(
            Environment.DIRECTORY_PICTURES), albumName);
    if (!file.mkdirs()) {
        Log.e(LOG_TAG, "Directory not created");
    }
    return file;
}
```

如果剛開始的時候，沒有預定義的子目錄存放我們的文件，可以在 getExternalFilesDir()方法中傳遞`null`. 它會返回app在external storage下的private的根目錄。

請記住，getExternalFilesDir() 方法會創建的目錄會在app被卸載時被系統刪除。如果我們的文件想在app被刪除時仍然保留，請使用getExternalStoragePublicDirectory().

無論是使用 getExternalStoragePublicDirectory() 來存儲可以共享的文件，還是使用 getExternalFilesDir() 來儲存那些對於我們的app來說是私有的文件，有一點很重要，那就是要使用那些類似`DIRECTORY_PICTURES` 的API的常量。那些目錄類型參數可以確保那些文件被系統正確的對待。例如，那些以`DIRECTORY_RINGTONES` 類型保存的文件就會被系統的media scanner認為是ringtone而不是音樂。

## 查詢剩餘空間

如果事先知道想要保存的文件大小，可以通過執行<a href="http://developer.android.com/reference/java/io/File.html#getFreeSpace()">getFreeSpace()</a> or <a href="http://developer.android.com/reference/java/io/File.html#getTotalSpace()">getTotalSpace()</a> 來判斷是否有足夠的空間來保存文件，從而避免發生[IOException](http://developer.android.com/reference/java/io/IOException.html)。那些方法提供了當前可用的空間還有存儲系統的總容量。

然而，系統並不能保證可以寫入通過`getFreeSpace()`查詢到的容量文件， 如果查詢的剩餘容量比我們的文件大小多幾MB，或者說文件系統使用率還不足90%，這樣則可以繼續進行寫的操作，否則最好不要寫進去。
> **Note：**並沒有強制要求在寫文件之前去檢查剩餘容量。我們可以嘗試先做寫的動作，然後通過捕獲 IOException 。這種做法僅適合於事先並不知道想要寫的文件的確切大小。例如，如果在把PNG圖片轉換成JPEG之前，我們並不知道最終生成的圖片大小是多少。

## 刪除文件

在不需要使用某些文件的時候應刪除它。刪除文件最直接的方法是直接執行文件的`delete()`方法。

```java
myFile.delete();
```

如果文件是保存在internal storage，我們可以通過`Context`來訪問並通過執行`deleteFile()`進行刪除

```java
myContext.deleteFile(fileName);
```

> **Note:** 當用戶卸載我們的app時，android系統會刪除以下文件：
* 所有保存到internal storage的文件。
* 所有使用getExternalFilesDir()方式保存在external storage的文件。

> 然而，通常來說，我們應該手動刪除所有通過 getCacheDir() 方式創建的緩存文件，以及那些不會再用到的文件。
