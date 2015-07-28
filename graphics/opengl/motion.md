# 添加移動

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/graphics/opengl/motion.html>

在屏幕上繪製圖形是OpenGL的一個基本特性，當然我們也可以通過其它的Android圖形框架類做這些事情，包括[Canvas](http://developer.android.com/reference/android/graphics/Canvas.html)和[Drawable](http://developer.android.com/reference/android/graphics/drawable/Drawable.html)對象。OpenGL ES的特別之處在於，它還提供了其它的一些功能，比如在三維空間中對繪製圖形進行移動和變換操作，或者通過其它獨有的方法創建出引人入勝的用戶體驗。

在這節課中，我們會更深入地學習OpenGL ES的知識：對一個圖形添加旋轉動畫。

## 旋轉一個形狀

使用OpenGL ES 2.0 旋轉一個繪製圖形是比較簡單的。在渲染器中，創建另一個變換矩陣（一個旋轉矩陣），並且將它和我們的投影變換矩陣以及相機視角變換矩陣結合在一起：

```java
private float[] mRotationMatrix = new float[16];
public void onDrawFrame(GL10 gl) {
    float[] scratch = new float[16];

    ...

    // Create a rotation transformation for the triangle
    long time = SystemClock.uptimeMillis() % 4000L;
    float angle = 0.090f * ((int) time);
    Matrix.setRotateM(mRotationMatrix, 0, angle, 0, 0, -1.0f);

    // Combine the rotation matrix with the projection and camera view
    // Note that the mMVPMatrix factor *must be first* in order
    // for the matrix multiplication product to be correct.
    Matrix.multiplyMM(scratch, 0, mMVPMatrix, 0, mRotationMatrix, 0);

    // Draw triangle
    mTriangle.draw(scratch);
}
```

如果完成了這些變更以後，你的三角形還是沒有旋轉的話，確認一下你是否將啟用[GLSurfaceView.RENDERMODE_WHEN_DIRTY](http://developer.android.com/reference/android/opengl/GLSurfaceView.html#RENDERMODE_WHEN_DIRTY)的這一配置所對應的代碼註釋掉了，有關該方面的知識會在下一節中展開。

## 啟用連續渲染

如果嚴格按照這節課的樣例代碼走到了現在這一步，那麼請確認一下是否將設置渲染模式為`RENDERMODE_WHEN_DIRTY`的那行代碼註釋了，不然的話OpenGL只會對這個形狀執行一次旋轉，然後就等待[GLSurfaceView](http://developer.android.com/reference/android/opengl/GLSurfaceView.html)容器的[requestRender()](http://developer.android.com/reference/android/opengl/GLSurfaceView.html#requestRender())方法被調用後才會繼續執行渲染操作。

```java
public MyGLSurfaceView(Context context) {
    ...
    // Render the view only when there is a change in the drawing data.
    // To allow the triangle to rotate automatically, this line is commented out:
    //setRenderMode(GLSurfaceView.RENDERMODE_WHEN_DIRTY);
}
```

除非某個對象，它的變化和用戶的交互無關，不然的話一般還是建議將這個配置打開。在下一節課中的內容將會把這個註釋放開，再次設定這一配置選項。
