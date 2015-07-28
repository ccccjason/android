# 建立OpenGL ES的環境

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/graphics/opengl/environment.html>

要在應用中使用OpenGL ES繪製圖像，我們必須為它們創建一個View容器。一種比較直接的方法是實現[GLSurfaceView](http://developer.android.com/reference/android/opengl/GLSurfaceView.html)類和[GLSurfaceView.Renderer](http://developer.android.com/reference/android/opengl/GLSurfaceView.Renderer.html)類。其中，[GLSurfaceView](http://developer.android.com/reference/android/opengl/GLSurfaceView.html)是一個View容器，它用來存放使用OpenGL繪製的圖形，而[GLSurfaceView.Renderer](http://developer.android.com/reference/android/opengl/GLSurfaceView.Renderer.html)則用來控制在該View中繪製的內容。關於這兩個類的更多信息，你可以閱讀：[OpenGL ES](http://developer.android.com/guide/topics/graphics/opengl.html)開發手冊。

使用[GLSurfaceView](http://developer.android.com/reference/android/opengl/GLSurfaceView.html)是一種將OpenGL ES集成到應用中的方法之一。對於一個全屏的或者接近全屏的圖形View，使用它是一個理想的選擇。開發者如果希望把OpenGL ES的圖形集成在佈局的一小部分裡面，那麼可以考慮使用[TextureView](http://developer.android.com/reference/android/view/TextureView.html)。對於喜歡自己動手實現的開發者來說，還可以通過使用[SurfaceView](http://developer.android.com/reference/android/view/SurfaceView.html)搭建一個OpenGL ES View，但這將需要編寫更多的代碼。

在這節課中，我們將展示如何在一個的Activity中完成[GLSurfaceView](http://developer.android.com/reference/android/opengl/GLSurfaceView.html)和[GLSurfaceView.Renderer](http://developer.android.com/reference/android/opengl/GLSurfaceView.Renderer.html)的最簡單的實現。

## 在Manifest配置文件中聲明使用OpenGL ES

為了讓應用能夠使用OpenGL ES 2.0接口，我們必須將下列聲明添加到Manifest配置文件當中：

```xml
<uses-feature android:glEsVersion="0x00020000" android:required="true" />
```

如果我們的應用使用紋理壓縮（Texture Compression），那麼我們必須對支持的壓縮格式也進行聲明，確保應用僅安裝在可以兼容的設備上：

```xml
<supports-gl-texture android:name="GL_OES_compressed_ETC1_RGB8_texture" />
<supports-gl-texture android:name="GL_OES_compressed_paletted_texture" />
```

更多關於紋理壓縮的內容，可以閱讀：[OpenGL](http://developer.android.com/guide/topics/graphics/opengl.html#textures)開發手冊。

## 為OpenGL ES圖形創建一個activity

使用OpenGL ES的安卓應用就像其它類型的應用一樣有自己的用戶接口，即也擁有多個Activity。主要的區別體現在Acitivity佈局內容上的差異。在許多應用中你可能會使用[TextView](http://developer.android.com/reference/android/widget/TextView.html)，[Button](http://developer.android.com/reference/android/widget/Button.html)和[ListView](http://developer.android.com/reference/android/widget/ListView.html)等，而在使用OpenGL ES的應用中，我們還可以添加一個[GLSurfaceView](http://developer.android.com/reference/android/opengl/GLSurfaceView.html)。

下面的代碼展示了一個使用[GLSurfaceView](http://developer.android.com/reference/android/opengl/GLSurfaceView.html)作為其主View的Activity：

```java
public class OpenGLES20Activity extends Activity {

    private GLSurfaceView mGLView;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Create a GLSurfaceView instance and set it
        // as the ContentView for this Activity.
        mGLView = new MyGLSurfaceView(this);
        setContentView(mGLView);
    }
}
```

> **Note：**OpenGL ES 2.0需要Android 2.2（API Level 8）或更高版本的系統，所以確保你的Android項目的API版本滿足該要求。

## 構建一個GLSurfaceView對象

[GLSurfaceView](http://developer.android.com/reference/android/opengl/GLSurfaceView.html)是一種比較特殊的View，我們可以在該View中繪製OpenGL ES圖形，不過它自己並不做太多和繪製圖形相關的任務。繪製對象的任務是由你在該View中配置的[GLSurfaceView.Renderer](http://developer.android.com/reference/android/opengl/GLSurfaceView.Renderer.html)所控制的。事實上，這個對象的代碼非常簡短，你可能會希望不要繼承它，直接創建一個未經修改的GLSurfaceView實例，不過請不要這麼做，因為我們需要繼承該類來捕捉觸控事件，這方面知識會在[響應觸摸事件](touch.html)（該系列課程的最後一節課）中做進一步的介紹。

[GLSurfaceView](http://developer.android.com/reference/android/opengl/GLSurfaceView.html)的核心代碼非常簡短，所以對於一個快速的實現而言，我們通常可以在Acitvity中創建一個內部類並使用它：

```java
class MyGLSurfaceView extends GLSurfaceView {

    private final MyGLRenderer mRenderer;

    public MyGLSurfaceView(Context context){
        super(context);

        // Create an OpenGL ES 2.0 context
        setEGLContextClientVersion(2);

        mRenderer = new MyGLRenderer();

        // Set the Renderer for drawing on the GLSurfaceView
        setRenderer(mRenderer);
    }
}
```

另一個對於[GLSurfaceView](http://developer.android.com/reference/android/opengl/GLSurfaceView.html)實現的可選選項，是將渲染模式設置為：[GLSurfaceView.RENDERMODE_WHEN_DIRTY](http://developer.android.com/reference/android/opengl/GLSurfaceView.html#RENDERMODE_WHEN_DIRTY)，其含義是：僅在你的繪製數據發生變化時才在視圖中進行繪製操作：

```java
// Render the view only when there is a change in the drawing data
setRenderMode(GLSurfaceView.RENDERMODE_WHEN_DIRTY);
```

如果選用這一配置選項，那麼除非調用了<a href="http://developer.android.com/reference/android/opengl/GLSurfaceView.html#requestRender()">requestRender()</a>，否則[GLSurfaceView](http://developer.android.com/reference/android/opengl/GLSurfaceView.html)不會被重新繪製，這樣做可以讓應用的性能及效率得到提高。

## 構建一個渲染類

在一個使用OpenGL ES的應用中，一個[GLSurfaceView.Renderer](http://developer.android.com/reference/android/opengl/GLSurfaceView.Renderer.html)類的實現（或者我們將其稱之為渲染器），正是事情變得有趣的地方。該類會控制和其相關聯的[GLSurfaceView](http://developer.android.com/reference/android/opengl/GLSurfaceView.html)，具體而言，它會控制在[GLSurfaceView](http://developer.android.com/reference/android/opengl/GLSurfaceView.html)上繪製的內容。在渲染器中，一共有三個方法會被Android系統調用，以此來明確要在[GLSurfaceView](http://developer.android.com/reference/android/opengl/GLSurfaceView.html)上繪製的內容以及如何繪製：
* <a href="http://developer.android.com/reference/android/opengl/GLSurfaceView.Renderer.html#onSurfaceCreated(javax.microedition.khronos.opengles.GL10, javax.microedition.khronos.egl.EGLConfig)">onSurfaceCreated()</a>：調用一次，用來配置View的OpenGL ES環境。
* <a href="http://developer.android.com/reference/android/opengl/GLSurfaceView.Renderer.html#onDrawFrame(javax.microedition.khronos.opengles.GL10)">onDrawFrame()</a>：每次重新繪製View時被調用。
* <a href="http://developer.android.com/reference/android/opengl/GLSurfaceView.Renderer.html#onDrawFrame(javax.microedition.khronos.opengles.GL10)">onSurfaceChanged()</a>：如果View的幾何形態發生變化時會被調用，例如當設備的屏幕方向發生改變時。

下面是一個非常基本的OpenGL ES渲染器的實現，它僅僅在[GLSurfaceView](http://developer.android.com/reference/android/opengl/GLSurfaceView.html)中畫一個黑色的背景：

```java
public class MyGLRenderer implements GLSurfaceView.Renderer {

    public void onSurfaceCreated(GL10 unused, EGLConfig config) {
        // Set the background frame color
        GLES20.glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    }

    public void onDrawFrame(GL10 unused) {
        // Redraw background color
        GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT);
    }

    public void onSurfaceChanged(GL10 unused, int width, int height) {
        GLES20.glViewport(0, 0, width, height);
    }
}
```

就是這樣！上面的代碼創建了一個簡單地應用程序，它使用OpenGL讓屏幕呈現為黑色。雖然它的代碼看上去並沒有做什麼非常有意思的事情，但是通過創建這些類，我們已經對使用OpenGL繪製圖形有了基本的認識和鋪墊。

> **Note：**你可能想知道，自己明明使用的是OpenGL ES 2.0接口，為什麼這些方法會有一個[GL10](http://developer.android.com/reference/javax/microedition/khronos/opengles/GL10.html)的參數。這是因為這些方法的簽名（Method Signature）在2.0接口中被簡單地重用了，以此來保持Android框架的代碼儘量簡單。

如果你對OpenGL ES接口很熟悉，那麼你現在就可以在你的應用中構建一個OpenGL ES的環境並繪製圖形了。當然， 如果你希望獲取更多的幫助來學會使用OpenGL，那麼請繼續學習下一節課程獲取更多的知識。
