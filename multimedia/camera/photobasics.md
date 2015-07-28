# 輕鬆拍攝照片

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/camera/photobasics.html>

這節課將講解如何使用已有的相機應用拍攝照片。

假設我們正在實現一個基於人群的氣象服務，通過應用客戶端拍下的天氣圖片匯聚在一起，可以組成全球氣象圖。整合圖片只是應用的一小部分，我們想要通過最簡單的方式獲取圖片，而不是重新設計並實現一個具有相機功能的組件。幸運的是，通常來說，大多數Android設備都已經安裝了至少一款相機程序。在這節課中，我們會學習如何利用已有的相機應用拍攝照片。

## 請求使用相機權限

如果拍照是應用的必要功能，那麼應該令它在Google Play中僅對有相機的設備可見。為了讓用戶知道我們的應用需要依賴相機，在Manifest清單文件中添加`<uses-feature>`標籤:

```xml
 <manifest ... >
    <uses-feature android:name="android.hardware.camera"
                  android:required="true" />
    ...
</manifest>
```

如果我們的應用使用相機，但相機並不是應用的正常運行所必不可少的組件，可以將`android:required`設置為`"false"`。這樣的話，Google Play 也會允許沒有相機的設備下載該應用。當然我們有必要在使用相機之前通過調用<a href="http://developer.android.com/reference/android/content/pm/PackageManager.html#hasSystemFeature(java.lang.String)">hasSystemFeature(PackageManager.FEATURE_CAMERA)</a>方法來檢查設備上是否有相機。如果沒有，我們應該禁用和相機相關的功能！

## 使用相機應用程序進行拍照

利用一個描述了執行目的Intent對象，Android可以將某些執行任務委託給其他應用。整個過程包含三部分： Intent 本身，一個函數調用來啟動外部的 Activity，當焦點返回到我們的Activity時，處理返回圖像數據的代碼。

下面的函數通過發送一個Intent來捕獲照片：

```java
static final int REQUEST_IMAGE_CAPTURE = 1;

private void dispatchTakePictureIntent() {
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
    }
}
```

注意在調用<a href="http://developer.android.com/reference/android/app/Activity.html#startActivityForResult(android.content.Intent, int)">startActivityForResult()</a>方法之前，先調用<a href="http://developer.android.com/reference/android/content/Intent.html#resolveActivity(android.content.pm.PackageManager)">resolveActivity()</a>，這個方法會返回能處理該Intent的第一個Activity（譯註：即檢查有沒有能處理這個Intent的Activity）。執行這個檢查非常重要，因為如果在調用<a href="http://developer.android.com/reference/android/app/Activity.html#startActivityForResult(android.content.Intent, int)">startActivityForResult()</a>時，沒有應用能處理你的Intent，應用將會崩潰。所以只要返回結果不為null，使用該Intent就是安全的。

## 獲取縮略圖

拍攝照片並不是應用的最終目的，我們還想要從相機應用那裡取回拍攝的照片，並對它執行某些操作。

Android的相機應用會把拍好的照片編碼為縮小的[Bitmap](http://developer.android.com/reference/android/graphics/Bitmap.html)，使用extra value的方式添加到返回的[Intent](http://developer.android.com/reference/android/content/Intent.html)當中，並傳送給<a href="http://developer.android.com/reference/android/app/Activity.html#onActivityResult(int, int, android.content.Intent)">onActivityResult()</a>，對應的Key為`"data"`。下面的代碼展示的是如何獲取這一圖片並顯示在[ImageView](http://developer.android.com/reference/android/widget/ImageView.html)上。

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
        Bundle extras = data.getExtras();
        Bitmap imageBitmap = (Bitmap) extras.get("data");
        mImageView.setImageBitmap(imageBitmap);
    }
}
```

> **Note:** 這張從`"data"`中取出的縮略圖適用於作為圖標，但其他作用會比較有限。而處理一張全尺寸圖片需要做更多的工作。

## 保存全尺寸照片

如果我們提供了一個File對象給Android的相機程序，它會保存這張全尺寸照片到給定的路徑下。另外，我們必須提供存儲圖片所需要的含有後綴名形式的文件名。

一般而言，用戶使用設備相機所拍攝的任何照片都應該被存放在設備的公共外部存儲中，這樣它們就能被所有的應用訪問。將[DIRECTORY_PICTURES](http://developer.android.com/reference/android/os/Environment.html#DIRECTORY_PICTURES)作為參數，傳遞給<a href="http://developer.android.com/reference/android/os/Environment.html#getExternalStoragePublicDirectory(java.lang.String)">getExternalStoragePublicDirectory()</a>方法，可以返回適用於存儲公共圖片的目錄。由於該方法提供的目錄被所有應用共享，因此對該目錄進行讀寫操作分別需要[READ_EXTERNAL_STORAGE](http://developer.android.com/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE)和[WRITE_EXTERNAL_STORAGE](http://developer.android.com/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE)權限。另外，因為寫權限隱含了讀權限，所以如果需要外部存儲的寫權限，那麼僅僅需要請求一項權限就可以了：

```xml
<manifest ...>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    ...
</manifest>
```

然而，如果希望照片對我們的應用而言是私有的，那麼可以使用<a href="http://developer.android.com/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)">getExternalFilesDir()</a>提供的目錄。在Android 4.3及以下版本的系統中，寫這個目錄需要[WRITE_EXTERNAL_STORAGE](http://developer.android.com/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE)權限。從Android 4.4開始，該目錄將無法被其他應用訪問，所以該權限就不再需要了，你可以通過添加[maxSdkVersion](http://developer.android.com/guide/topics/manifest/uses-permission-element.html#maxSdk)屬性，聲明只在低版本的Android設備上請求這個權限。

```xml
<manifest ...>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
                     android:maxSdkVersion="18" />
    ...
</manifest>
```

> **Note:** 所有存儲在<a href="http://developer.android.com/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)">getExternalFilesDir()</a>提供的目錄中的文件會在用戶卸載你的app後被刪除。

一旦選定了存儲文件的目錄，我們還需要設計一個保證文件名不會衝突的命名規則。當然我們還可以將路徑存儲在一個成員變量裡以備在將來使用。下面的例子使用日期時間戳作為新照片的文件名：

```java
String mCurrentPhotoPath;

private File createImageFile() throws IOException {
    // Create an image file name
    String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
    String imageFileName = "JPEG_" + timeStamp + "_";
    File storageDir = Environment.getExternalStoragePublicDirectory(
            Environment.DIRECTORY_PICTURES);
    File image = File.createTempFile(
        imageFileName,  /* prefix */
        ".jpg",         /* suffix */
        storageDir      /* directory */
    );

    // Save a file: path for use with ACTION_VIEW intents
    mCurrentPhotoPath = "file:" + image.getAbsolutePath();
    return image;
}
```

有了上面的方法，我們就可以給新照片創建文件對象了，現在我們可以像這樣創建並觸發一個[Intent](http://developer.android.com/reference/android/content/Intent.html)：

```java
static final int REQUEST_TAKE_PHOTO = 1;

private void dispatchTakePictureIntent() {
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    // Ensure that there's a camera activity to handle the intent
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        // Create the File where the photo should go
        File photoFile = null;
        try {
            photoFile = createImageFile();
        } catch (IOException ex) {
            // Error occurred while creating the File
            ...
        }
        // Continue only if the File was successfully created
        if (photoFile != null) {
            takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT,
                    Uri.fromFile(photoFile));
            startActivityForResult(takePictureIntent, REQUEST_TAKE_PHOTO);
        }
    }
}
```

## 將照片添加到相冊中

由於我們通過Intent創建了一張照片，因此圖片的存儲位置我們是知道的。對其他人來說，也許查看我們的照片最簡單的方式是通過系統的Media Provider。

> **Note:** 如果將圖片存儲在<a href="http://developer.android.com/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)">getExternalFilesDir()</a>提供的目錄中，Media Scanner將無法訪問到我們的文件，因為它們隸屬於應用的私有數據。

下面的例子演示瞭如何觸發系統的Media Scanner，將我們的照片添加到Media Provider的數據庫中，這樣就可以使得Android相冊程序與其他程序能夠讀取到這些照片。

```java
private void galleryAddPic() {
    Intent mediaScanIntent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
    File f = new File(mCurrentPhotoPath);
    Uri contentUri = Uri.fromFile(f);
    mediaScanIntent.setData(contentUri);
    this.sendBroadcast(mediaScanIntent);
}
```

## 解碼一幅縮放圖片

在有限的內存下，管理許多全尺寸的圖片會很棘手。如果發現應用在展示了少量圖片後消耗了所有內存，我們可以通過縮放圖片到目標視圖尺寸，之後再載入到內存中的方法，來顯著降低內存的使用，下面的例子演示了這個技術：

```java
private void setPic() {
    // Get the dimensions of the View
    int targetW = mImageView.getWidth();
    int targetH = mImageView.getHeight();

    // Get the dimensions of the bitmap
    BitmapFactory.Options bmOptions = new BitmapFactory.Options();
    bmOptions.inJustDecodeBounds = true;
    BitmapFactory.decodeFile(mCurrentPhotoPath, bmOptions);
    int photoW = bmOptions.outWidth;
    int photoH = bmOptions.outHeight;

    // Determine how much to scale down the image
    int scaleFactor = Math.min(photoW/targetW, photoH/targetH);

    // Decode the image file into a Bitmap sized to fill the View
    bmOptions.inJustDecodeBounds = false;
    bmOptions.inSampleSize = scaleFactor;
    bmOptions.inPurgeable = true;

    Bitmap bitmap = BitmapFactory.decodeFile(mCurrentPhotoPath, bmOptions);
    mImageView.setImageBitmap(bitmap);
}
```

***
