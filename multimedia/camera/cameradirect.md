# 控制相機

> 編寫:[kesenhoo](https://github.com/kesenhoo) - :<http://developer.android.com/training/camera/cameradirect.html>

在這一節課，我們會討論如何通過使用Android框架所提供的API來直接控制相機硬件。

直接控制相機，比起向已有的相機應用請求圖片或視頻，要複雜一些。這節課將會講解如何創建一個特殊的相機應用或將相機整合在我們的應用當中。

## 打開相機對象

獲取一個 [Camera](http://developer.android.com/reference/android/hardware/Camera.html) 對象是直接控制相機的第一步。正如Android自帶的相機程序一樣，比較好的訪問相機的方式是在<a href="http://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle)">onCreate()</a>方法裡面另起一個線程來打開相機。這種辦法可以避免因為啟動時間較長導致UI線程被阻塞。另外還有一種更好的方法：可以把打開相機的操作延遲到<a href="http://developer.android.com/reference/android/app/Activity.html#onResume()">onResume()</a>方法裡面去執行，這樣可以使得代碼更容易重用，還能保持控制流程更為簡單。

如果我們在執行<a href="http://developer.android.com/reference/android/hardware/Camera.html#open()">Camera.open()</a>方法的時候相機正在被另外一個應用使用，那麼函數會拋出一個Exception，我們可以利用`try`語句塊進行捕獲：

```java
private boolean safeCameraOpen(int id) {
    boolean qOpened = false;
  
    try {
        releaseCameraAndPreview();
        mCamera = Camera.open(id);
        qOpened = (mCamera != null);
    } catch (Exception e) {
        Log.e(getString(R.string.app_name), "failed to open Camera");
        e.printStackTrace();
    }

    return qOpened;    
}

private void releaseCameraAndPreview() {
    mPreview.setCamera(null);
    if (mCamera != null) {
        mCamera.release();
        mCamera = null;
    }
}
```

自從API Level 9開始，相機框架可以支持多個相機。如果使用舊的API，在調用<a href="http://developer.android.com/reference/android/hardware/Camera.html#open()">open()</a>時不傳入參數 ，那麼我們會獲取後置攝像頭。

## 創建相機預覽界面

拍照通常需要向用戶提供一個預覽界面來顯示待拍攝的事物。我們可以使用[SurfaceView](http://developer.android.com/reference/android/view/SurfaceView.html)來展現照相機採集的圖像。

### Preview類

我們需要使用Preview類來顯示預覽界面。這個類需要實現`android.view.SurfaceHolder.Callback`接口，用這個接口把相機硬件獲取的數據傳遞給應用。

```java
class Preview extends ViewGroup implements SurfaceHolder.Callback {

    SurfaceView mSurfaceView;
    SurfaceHolder mHolder;

    Preview(Context context) {
        super(context);

        mSurfaceView = new SurfaceView(context);
        addView(mSurfaceView);

        // Install a SurfaceHolder.Callback so we get notified when the
        // underlying surface is created and destroyed.
        mHolder = mSurfaceView.getHolder();
        mHolder.addCallback(this);
        mHolder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
    }
...
}
```

Preview類必須在實時圖像預覽開始之前傳遞給[Camera](http://developer.android.com/reference/android/hardware/Camera.html)對象。

### 設置和啟動Preview

一個Camera實例與它相關的Preview必須以特定的順序來創建，其中Camera對象首先被創建。在下面的示例中，初始化Camera的動作被封裝了起來，這樣，無論用戶想對Camera做什麼樣的改變，<a href="http://developer.android.com/reference/android/hardware/Camera.html#startPreview()">Camera.startPreview()</a>都會被`setCamera()`調用。另外，Preview對象必須在`surfaceChanged()`這一回調方法裡面重新啟用（restart）。

```java
public void setCamera(Camera camera) {
    if (mCamera == camera) { return; }
    
    stopPreviewAndFreeCamera();
    
    mCamera = camera;
    
    if (mCamera != null) {
        List<Size> localSizes = mCamera.getParameters().getSupportedPreviewSizes();
        mSupportedPreviewSizes = localSizes;
        requestLayout();
      
        try {
            mCamera.setPreviewDisplay(mHolder);
        } catch (IOException e) {
            e.printStackTrace();
        }
      
        // Important: Call startPreview() to start updating the preview
        // surface. Preview must be started before you can take a picture.
        mCamera.startPreview();
    }
}
```

## 修改相機設置

相機設置可以改變拍照的方式，從縮放級別到曝光補償等。下面的例子僅僅演示瞭如何改變預覽大小，更多設置請參考相機應用的源代碼。

```java
public void surfaceChanged(SurfaceHolder holder, int format, int w, int h) {
    // Now that the size is known, set up the camera parameters and begin
    // the preview.
    Camera.Parameters parameters = mCamera.getParameters();
    parameters.setPreviewSize(mPreviewSize.width, mPreviewSize.height);
    requestLayout();
    mCamera.setParameters(parameters);

    // Important: Call startPreview() to start updating the preview surface.
    // Preview must be started before you can take a picture.
    mCamera.startPreview();
}
```

## 設置預覽方向

大多數相機程序會鎖定預覽為橫屏狀態，因為該方向是相機傳感器的自然方向。當然這一設定並不會阻止我們去拍豎屏的照片，因為設備的方向信息會被記錄在EXIF頭中。<a href="http://developer.android.com/reference/android/hardware/Camera.html#setDisplayOrientation(int)">setCameraDisplayOrientation()</a>方法可以讓你在不影響照片拍攝過程的情況下，改變預覽的方向。然而，對於Android API Level 14及以下版本的系統，在改變方向之前，我們必須先停止預覽，然後再去重啟它。

## 拍攝照片

只要預覽開始之後，可以使用<a href="http://developer.android.com/reference/android/hardware/Camera.html#takePicture(android.hardware.Camera.ShutterCallback, android.hardware.Camera.PictureCallback, android.hardware.Camera.PictureCallback)">Camera.takePicture()</a>方法拍攝照片。我們可以創建<a href="http://developer.android.com/reference/android/hardware/Camera.PictureCallback.html">Camera.PictureCallback</a>與<a href="http://developer.android.com/reference/android/hardware/Camera.ShutterCallback.html">Camera.ShutterCallback</a>對象並將他們傳遞到<a href="http://developer.android.com/reference/android/hardware/Camera.html#takePicture(android.hardware.Camera.ShutterCallback, android.hardware.Camera.PictureCallback, android.hardware.Camera.PictureCallback)">Camera.takePicture()</a>中。

如果我們想要進行連拍，可以創建一個[Camera.PreviewCallback](http://developer.android.com/reference/android/hardware/Camera.PreviewCallback.html)並實現<a href="http://developer.android.com/reference/android/hardware/Camera.PreviewCallback.html#onPreviewFrame(byte[], android.hardware.Camera)">onPreviewFrame()</a>方法。我們可以拍攝選中的預覽幀，或是為調用<a href="http://developer.android.com/reference/android/hardware/Camera.html#takePicture(android.hardware.Camera.ShutterCallback, android.hardware.Camera.PictureCallback, android.hardware.Camera.PictureCallback)">takePicture()</a>建立一個延遲。

## 重啟Preview

在拍攝好圖片後，我們必須在用戶拍下一張圖片之前重啟預覽。下面的示例使用快門按鈕來實現重啟。

```java
@Override
public void onClick(View v) {
    switch(mPreviewState) {
    case K_STATE_FROZEN:
        mCamera.startPreview();
        mPreviewState = K_STATE_PREVIEW;
        break;

    default:
        mCamera.takePicture( null, rawCallback, null);
        mPreviewState = K_STATE_BUSY;
    } // switch
    shutterBtnConfig();
}
```

## 停止預覽並釋放相機

當應用使用好相機後，我們有必要進行清理操作。特別地，我們必須釋放[Camera](http://developer.android.com/reference/android/hardware/Camera.html)對象，不然的話可能會引起其他應用崩潰，包括我們自己應用的新實例。

那麼何時應該停止預覽並釋放相機呢？在預覽的Surface被摧毀之後，可以做停止預覽與釋放相機的操作。如下面Preview類中的方法所做的那樣：

```java
public void surfaceDestroyed(SurfaceHolder holder) {
    // Surface will be destroyed when we return, so stop the preview.
    if (mCamera != null) {
        // Call stopPreview() to stop updating the preview surface.
        mCamera.stopPreview();
    }
}

/**
 * When this function returns, mCamera will be null.
 */
private void stopPreviewAndFreeCamera() {

    if (mCamera != null) {
        // Call stopPreview() to stop updating the preview surface.
        mCamera.stopPreview();
    
        // Important: Call release() to release the camera for use by other
        // applications. Applications should release the camera immediately
        // during onPause() and re-open() it during onResume()).
        mCamera.release();
    
        mCamera = null;
    }
}
```

在這節課的前部分中，這一些系列的動作也是`setCamera()`方法的一部分，因此初始化一個相機的動作，總是從停止預覽開始的。
