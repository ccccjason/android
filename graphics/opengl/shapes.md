# 定義形狀

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/graphics/opengl/shapes.html>

在一個OpenGL ES View的上下文（Context）中定義形狀，是創建你的傑作所需要的第一步。在瞭解關於OpenGL ES如何定義圖形對象的基本知識之前，通過OpenGL ES 繪圖可能會有些困難。

這節課將講解OpenGL ES相對於Android設備屏幕的座標系，定義形狀和形狀表面的基本知識，如定義一個三角形和一個矩形。

## 定義一個三角形

OpenGL ES允許我們使用三維空間的座標來定義繪畫對象。所以在我們能畫三角形之前，必須先定義它的座標。在OpenGL 中，典型的辦法是為座標定義一個浮點型的頂點數組。為了高效起見，我們可以將座標寫入一個[ByteBuffer](http://developer.android.com/reference/java/nio/ByteBuffer.html)，它將會傳入OpenGl ES的圖形處理流程中：

```java
public class Triangle {

    private FloatBuffer vertexBuffer;

    // number of coordinates per vertex in this array
    static final int COORDS_PER_VERTEX = 3;
    static float triangleCoords[] = {   // in counterclockwise order:
             0.0f,  0.622008459f, 0.0f, // top
            -0.5f, -0.311004243f, 0.0f, // bottom left
             0.5f, -0.311004243f, 0.0f  // bottom right
    };

    // Set color with red, green, blue and alpha (opacity) values
    float color[] = { 0.63671875f, 0.76953125f, 0.22265625f, 1.0f };

    public Triangle() {
        // initialize vertex byte buffer for shape coordinates
        ByteBuffer bb = ByteBuffer.allocateDirect(
                // (number of coordinate values * 4 bytes per float)
                triangleCoords.length * 4);
        // use the device hardware's native byte order
        bb.order(ByteOrder.nativeOrder());

        // create a floating point buffer from the ByteBuffer
        vertexBuffer = bb.asFloatBuffer();
        // add the coordinates to the FloatBuffer
        vertexBuffer.put(triangleCoords);
        // set the buffer to read the first coordinate
        vertexBuffer.position(0);
    }
}
```

默認情況下，OpenGL ES會假定一個座標系，在這個座標系中，[0, 0, 0]（分別對應X軸座標, Y軸座標, Z軸座標）對應的是GLSurfaceView的中心。[1, 1, 0]對應的是右上角，[-1, -1, 0]對應的則是左下角。如果想要看此座標系的插圖說明，可以閱讀[OpenGL ES](http://developer.android.com/guide/topics/graphics/opengl.html#faces-winding)開發手冊。

注意到這個形狀的座標是以逆時針順序定義的。繪製的順序非常關鍵，因為它定義了哪一面是形狀的正面（希望繪製的一面），以及背面（使用OpenGL ES的Cull Face功能可以讓背面不要繪製）。更多關於該方面的信息，可以閱讀[OpenGL ES](http://developer.android.com/guide/topics/graphics/opengl.html#faces-winding)開發手冊。

## 定義一個矩形

在OpenGL中定義三角形非常簡單，那麼你是否想要來點更復雜的呢？比如，定義一個矩形？有很多方法可以用來定義矩形，不過在OpenGL ES中最典型的辦法是使用兩個三角形拼接在一起：

![ccw-square](ccw-square.png "使用兩個三角形畫一個矩形")

再一次地，我們需要逆時針地為三角形頂點定義座標來表現這個圖形，並將值放入一個[ByteBuffer](http://developer.android.com/reference/java/nio/ByteBuffer.html)中。為了避免由兩個三角形重合的那條邊的頂點被重複定義，可以使用一個繪製列表來告訴OpenGL ES圖形處理流程應該如何畫這些頂點。下面是代碼樣例：

```java
public class Square {

    private FloatBuffer vertexBuffer;
    private ShortBuffer drawListBuffer;

    // number of coordinates per vertex in this array
    static final int COORDS_PER_VERTEX = 3;
    static float squareCoords[] = {
            -0.5f,  0.5f, 0.0f,   // top left
            -0.5f, -0.5f, 0.0f,   // bottom left
             0.5f, -0.5f, 0.0f,   // bottom right
             0.5f,  0.5f, 0.0f }; // top right

    private short drawOrder[] = { 0, 1, 2, 0, 2, 3 }; // order to draw vertices

    public Square() {
        // initialize vertex byte buffer for shape coordinates
        ByteBuffer bb = ByteBuffer.allocateDirect(
        // (# of coordinate values * 4 bytes per float)
                squareCoords.length * 4);
        bb.order(ByteOrder.nativeOrder());
        vertexBuffer = bb.asFloatBuffer();
        vertexBuffer.put(squareCoords);
        vertexBuffer.position(0);

        // initialize byte buffer for the draw list
        ByteBuffer dlb = ByteBuffer.allocateDirect(
        // (# of coordinate values * 2 bytes per short)
                drawOrder.length * 2);
        dlb.order(ByteOrder.nativeOrder());
        drawListBuffer = dlb.asShortBuffer();
        drawListBuffer.put(drawOrder);
        drawListBuffer.position(0);
    }
}
```

該樣例可以看作是一個如何使用OpenGL創建複雜圖形的啟發，通常來說，我們需要使用三角形的集合來繪製對象。在下一節課中，我們將學習如何在屏幕上畫這些形狀。
