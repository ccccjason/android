# 輕鬆錄製視頻

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/camera/videobasics.html>

這節課會介紹如何使用已有的相機應用來錄製視頻。

假設在我們應用的所有功能當中，整合視頻只是其中的一小部分，我們想要以最簡單的方法錄製視頻，而不是重新實現一個攝像機組件。幸運的是，大多數Android設備已經安裝了一個能錄製視頻的相機應用。在本節課當中，我們將會讓它為我們完成這一任務。

## 請求相機權限

為了讓用戶知道我們的應用依賴照相機，在Manifest清單文件中添加`<uses-feature>`標籤:

```xml
<manifest ... >
    <uses-feature android:name="android.hardware.camera"
                  android:required="true" />
    ...
</manifest>
```

如果應用使用相機，但相機並不是應用正常運行所必不可少的組件，可以將`android:required`設置為`"false"`。這樣的話，Google Play 也會允許沒有相機的設備下載該應用。當然我們有必要在使用相機之前通過調用<a href="http://developer.android.com/reference/android/content/pm/PackageManager.html#hasSystemFeature(java.lang.String)">hasSystemFeature(PackageManager.FEATURE_CAMERA)</a>方法來檢查設備上是否有相機。如果沒有，那麼和相機相關的功能應該禁用！

## 使用相機程序來錄製視頻

利用一個描述了執行目的的Intent對象，Android可以將某些執行任務委託給其他應用。整個過程包含三部分： Intent 本身，一個函數調用來啟動外部的 Activity，當焦點返回到Activity時，處理返回圖像數據的代碼。

下面的函數將會發送一個Intent來錄製視頻

```java
static final int REQUEST_VIDEO_CAPTURE = 1;

private void dispatchTakeVideoIntent() {
    Intent takeVideoIntent = new Intent(MediaStore.ACTION_VIDEO_CAPTURE);
    if (takeVideoIntent.resolveActivity(getPackageManager()) != null) {
        startActivityForResult(takeVideoIntent, REQUEST_VIDEO_CAPTURE);
    }
}
```

注意在調用<a href="http://developer.android.com/reference/android/app/Activity.html#startActivityForResult(android.content.Intent, int)">startActivityForResult()</a>方法之前，先調用<a href="http://developer.android.com/reference/android/content/Intent.html#resolveActivity(android.content.pm.PackageManager)">resolveActivity()</a>，這個方法會返回能處理該Intent的第一個Activity（譯註：即檢查有沒有能處理這個Intent的Activity）。執行這個檢查非常重要，因為如果在調用<a href="http://developer.android.com/reference/android/app/Activity.html#startActivityForResult(android.content.Intent, int)">startActivityForResult()</a>時，沒有應用能處理你的Intent，應用將會崩潰。所以只要返回結果不為null，使用該Intent就是安全的。


## 查看視頻

Android的相機程序會把指向視頻存儲地址的[Uri](http://developer.android.com/reference/android/net/Uri.html)添加到[Intent](http://developer.android.com/reference/android/content/Intent.html)中，並傳送給<a href="http://developer.android.com/reference/android/app/Activity.html#onActivityResult(int, int, android.content.Intent)">onActivityResult()</a>方法。下面的代碼獲取該視頻並顯示到一個VideoView當中：

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_VIDEO_CAPTURE && resultCode == RESULT_OK) {
        Uri videoUri = intent.getData();
        mVideoView.setVideoURI(videoUri);
    }
}
```
