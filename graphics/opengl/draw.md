# 繪製形狀

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/graphics/opengl/draw.html>

在定義了使用OpenGL繪製的形狀之後，你可能希望繪製出它們。使用OpenGL ES 2.0繪製圖形可能會比你想象當中更復雜一些，因為API中提供了大量對於圖形渲染流程的控制。

這節課將解釋如何使用OpenGL ES 2.0接口畫出在上一節課中定義的形狀。

## 初始化形狀

在你開始繪畫之前，你需要初始化並加載你期望繪製的圖形。除非你所使用的形狀結構（原始座標）在執行過程中發生了變化，不然的話你應該在渲染器的<a href="http://developer.android.com/reference/android/opengl/GLSurfaceView.Renderer.html#onSurfaceCreated(javax.microedition.khronos.opengles.GL10, javax.microedition.khronos.egl.EGLConfig)">onSurfaceCreated()</a>方法中初始化它們，這樣做是出於內存和執行效率的考量。

```java
public class MyGLRenderer implements GLSurfaceView.Renderer {

    ...
    private Triangle mTriangle;
    private Square   mSquare;

    public void onSurfaceCreated(GL10 unused, EGLConfig config) {
        ...

        // initialize a triangle
        mTriangle = new Triangle();
        // initialize a square
        mSquare = new Square();
    }
    ...
}
```

### 畫一個形狀

使用OpenGL ES 2.0畫一個定義好的形狀需要較多代碼，因為你需要提供很多圖形渲染流程的細節。具體而言，你必須定義如下幾項：
* 頂點著色器（Vertex Shader）：用來渲染形狀頂點的OpenGL ES代碼。
* 片段著色器（Fragment Shader）：使用顏色或紋理渲染形狀表面的OpenGL ES代碼。
* 程式（Program）：一個OpenGL ES對象，包含了你希望用來繪製一個或更多圖形所要用到的著色器。

你需要至少一個頂點著色器來繪製一個形狀，以及一個片段著色器為該形狀上色。這些著色器必須被編譯然後添加到一個OpenGL ES Program當中，並利用它來繪製形狀。下面的代碼在Triangle類中定義了基本的著色器，我們可以利用它們繪製出一個圖形：

```java
public class Triangle {

    private final String vertexShaderCode =
        "attribute vec4 vPosition;" +
        "void main() {" +
        "  gl_Position = vPosition;" +
        "}";

    private final String fragmentShaderCode =
        "precision mediump float;" +
        "uniform vec4 vColor;" +
        "void main() {" +
        "  gl_FragColor = vColor;" +
        "}";

    ...
}
```

著色器包含了OpenGL Shading Language（GLSL）代碼，它必須先被編譯然後才能在OpenGL環境中使用。要編譯這些代碼，需要在你的渲染器類中創建一個輔助方法：

```java
public static int loadShader(int type, String shaderCode){

    // create a vertex shader type (GLES20.GL_VERTEX_SHADER)
    // or a fragment shader type (GLES20.GL_FRAGMENT_SHADER)
    int shader = GLES20.glCreateShader(type);

    // add the source code to the shader and compile it
    GLES20.glShaderSource(shader, shaderCode);
    GLES20.glCompileShader(shader);

    return shader;
}
```

為了繪製你的圖形，你必須編譯著色器代碼，將它們添加至一個OpenGL ES Program對象中，然後執行鏈接。在你的繪製對象的構造函數裡做這些事情，這樣上述步驟就只用執行一次。

> **Note：**編譯OpenGL ES著色器及鏈接操作對於CPU週期和處理時間而言，消耗是巨大的，所以你應該避免重複執行這些事情。如果在執行期間不知道著色器的內容，那麼你應該在構建你的應用時，確保它們只被創建了一次，並且緩存以備後續使用。

```java
public class Triangle() {
    ...

    private final int mProgram;

    public Triangle() {
        ...

        int vertexShader = MyGLRenderer.loadShader(GLES20.GL_VERTEX_SHADER,
                                        vertexShaderCode);
        int fragmentShader = MyGLRenderer.loadShader(GLES20.GL_FRAGMENT_SHADER,
                                        fragmentShaderCode);

        // create empty OpenGL ES Program
        mProgram = GLES20.glCreateProgram();

        // add the vertex shader to program
        GLES20.glAttachShader(mProgram, vertexShader);

        // add the fragment shader to program
        GLES20.glAttachShader(mProgram, fragmentShader);

        // creates OpenGL ES program executables
        GLES20.glLinkProgram(mProgram);
    }
}
```

至此，你已經完全準備好添加實際的調用語句來繪製你的圖形了。使用OpenGL ES繪製圖形需要你定義一些變量來告訴渲染流程你需要繪製的內容以及如何繪製。既然繪製屬性會根據形狀的不同而發生變化，把繪製邏輯包含在形狀類裡面將是一個不錯的主意。

創建一個`draw()`方法來繪製圖形。下面的代碼為形狀的頂點著色器和形狀著色器設置了位置和顏色值，然後執行繪製函數：

```java
private int mPositionHandle;
private int mColorHandle;

private final int vertexCount = triangleCoords.length / COORDS_PER_VERTEX;
private final int vertexStride = COORDS_PER_VERTEX * 4; // 4 bytes per vertex

public void draw() {
    // Add program to OpenGL ES environment
    GLES20.glUseProgram(mProgram);

    // get handle to vertex shader's vPosition member
    mPositionHandle = GLES20.glGetAttribLocation(mProgram, "vPosition");

    // Enable a handle to the triangle vertices
    GLES20.glEnableVertexAttribArray(mPositionHandle);

    // Prepare the triangle coordinate data
    GLES20.glVertexAttribPointer(mPositionHandle, COORDS_PER_VERTEX,
                                 GLES20.GL_FLOAT, false,
                                 vertexStride, vertexBuffer);

    // get handle to fragment shader's vColor member
    mColorHandle = GLES20.glGetUniformLocation(mProgram, "vColor");

    // Set color for drawing the triangle
    GLES20.glUniform4fv(mColorHandle, 1, color, 0);

    // Draw the triangle
    GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, vertexCount);

    // Disable vertex array
    GLES20.glDisableVertexAttribArray(mPositionHandle);
}
```

一旦你完成了上述所有代碼，僅需要在你渲染器的<a href="http://developer.android.com/reference/android/opengl/GLSurfaceView.Renderer.html#onDrawFrame(javax.microedition.khronos.opengles.GL10)">onDrawFrame()</a>方法中調用`draw()`方法就可以畫出我們想要畫的對象了：

```java
public void onDrawFrame(GL10 unused) {
    ...

    mTriangle.draw();
}
```

當你運行這個應用時，它看上去會像是這樣：

![ogl-triangle](ogl-triangle.png "不使用投影或者相機視圖畫出來的三角形")

在這個代碼樣例中，還存在一些問題。首先，它無法給用戶帶來什麼深刻的印象。其次，這個三角形看上去有一些扁，另外當你改變屏幕方向時，它的形狀也會隨之改變。發生形變的原因是因為對象的頂點沒有根據顯示[GLSurfaceView](http://developer.android.com/reference/android/opengl/GLSurfaceView.html)的屏幕區域的長寬比進行修正。你可以在下一節課中使用投影（Projection）或者相機視角（Camera View）來解決這個問題。

最後，這個三角形是靜止的，這看上去有些無聊。在[添加移動](motion.html)課程當中（後續課程），你會讓這個形狀發生旋轉，並使用一些OpenGL ES圖形處理流程中更加新奇的用法。
