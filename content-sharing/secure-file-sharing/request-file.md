# 請求分享一個文件

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/secure-file-sharing/request-file.html>

當一個應用程序希望訪問由其它應用程序所共享的文件時，請求應用程序（客戶端）經常會向其它應用程序（服務端）發送一個文件請求。多數情況下，該請求會導致在服務端應用程序中啟動一個Activity，該Activity中會顯示可以共享的文件。當服務端應用程序向客戶端應用程序返回了文件的Content URI後，用戶即可開始選擇文件。

本課將展示一個客戶端應用程序應該如何向服務端應用程序請求一個文件，接收服務端應用程序發來的Content URI，然後使用這個Content URI打開這個文件。

## 發送一個文件請求

為了向服務端應用程序發送文件請求，在客戶端應用程序中，需要調用[startActivityForResult](http://developer.android.com/reference/android/app/Activity.html#startActivityForResult(android.content.Intent, int))方法，同時傳遞給這個方法一個[Intent](http://developer.android.com/reference/android/content/Intent.html)參數，它包含了客戶端應用程序能處理的某個Action，比如[ACTION_PICK](http://developer.android.com/reference/android/content/Intent.html#ACTION_PICK)及一個MIME類型。

例如，下面的代碼展示瞭如何向服務端應用程序發送一個Intent，來啟動在[分享文件](sharing-file.html#SendURI)中提到的[Activity](http://developer.android.com/reference/android/app/Activity.html)：

```java
public class MainActivity extends Activity {
    private Intent mRequestFileIntent;
    private ParcelFileDescriptor mInputPFD;
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mRequestFileIntent = new Intent(Intent.ACTION_PICK);
        mRequestFileIntent.setType("image/jpg");
        ...
    }
    ...
    protected void requestFile() {
        /**
         * When the user requests a file, send an Intent to the
         * server app.
         * files.
         */
            startActivityForResult(mRequestFileIntent, 0);
        ...
    }
    ...
}
```

## 訪問請求的文件

當服務端應用程序向客戶端應用程序發回包含Content URI的[Intent](http://developer.android.com/reference/android/content/Intent.html)時，該[Intent](http://developer.android.com/reference/android/content/Intent.html)會傳遞給客戶端應用程序重寫的<a href="http://developer.android.com/reference/android/app/Activity.html#onActivityResult(int, int, android.content.Intent)">onActivityResult()</a>方法當中。一旦客戶端應用程序擁有了文件的Content URI，它就可以通過獲取其[FileDescriptor](http://developer.android.com/reference/java/io/FileDescriptor.html)訪問文件了。

這一過程中不用過多擔心文件的安全問題，因為客戶端應用程序所收到的所有數據只有文件的Content URI而已。由於URI不包含目錄路徑信息，客戶端應用程序無法查詢或打開任何服務端應用程序的其他文件。客戶端應用程序僅僅獲取了這個文件的訪問渠道以及由服務端應用程序授予的訪問權限。同時訪問權限是臨時的，一旦這個客戶端應用的任務棧結束了，這個文件將無法再被除服務端應用程序之外的其他應用程序訪問。

下面的例子展示了客戶端應用程序應該如何處理髮自服務端應用程序的[Intent](http://developer.android.com/reference/android/content/Intent.html)，以及客戶端應用程序如何使用Content URI獲取[FileDescriptor](http://developer.android.com/reference/java/io/FileDescriptor.html)：

```java
    /*
     * When the Activity of the app that hosts files sets a result and calls
     * finish(), this method is invoked. The returned Intent contains the
     * content URI of a selected file. The result code indicates if the
     * selection worked or not.
     */
    @Override
    public void onActivityResult(int requestCode, int resultCode,
            Intent returnIntent) {
        // If the selection didn't work
        if (resultCode != RESULT_OK) {
            // Exit without doing anything else
            return;
        } else {
            // Get the file's content URI from the incoming Intent
            Uri returnUri = returnIntent.getData();
            /*
             * Try to open the file for "read" access using the
             * returned URI. If the file isn't found, write to the
             * error log and return.
             */
            try {
                /*
                 * Get the content resolver instance for this context, and use it
                 * to get a ParcelFileDescriptor for the file.
                 */
                mInputPFD = getContentResolver().openFileDescriptor(returnUri, "r");
            } catch (FileNotFoundException e) {
                e.printStackTrace();
                Log.e("MainActivity", "File not found.");
                return;
            }
            // Get a regular file descriptor for the file
            FileDescriptor fd = mInputPFD.getFileDescriptor();
            ...
        }
    }
```

<a href="http://developer.android.com/reference/android/content/ContentResolver.html#openFileDescriptor(android.net.Uri, java.lang.String)">openFileDescriptor()</a>方法返回一個文件的[ParcelFileDescriptor](http://developer.android.com/reference/android/os/ParcelFileDescriptor.html)對象。客戶端應用程序從該對象中獲取[FileDescriptor](http://developer.android.com/reference/java/io/FileDescriptor.html)對象，然後利用該對象讀取這個文件了。
