# 退出全屏的Activity

> 編寫: [roya](https://github.com/RoyaAoki) 原文:<https://developer.android.com/training/wearables/ui/exit.html>

默認情況下，用戶通過從左到右滑動退出Android Wear activities。如果應用含有水平滾動的內容，用戶首先滑動到內容邊緣，然後再次從左到右滑動即退出app。

對於更加沉浸式的體驗，比如在應用中可以任意方向地滾動地圖，這時我們可以在應用中禁用滑動退出手勢。然而，如果我們禁用了這個功能，那麼我們必須使用Wearable UI庫中的`DismissOverlayView`類實現長按退出UI模式讓用戶退出應用。當然，我們需要在用戶第一次運行我們應用的時候提醒用戶可以通過長按退出應用。

更多關於退出Android Wear activities的設計指南，請查看[Breaking out of the card](https://developer.android.com/design/wear/structure.html#Custom)。

## 禁用滑動退出手勢

如果我們應用的用戶交互模型與滑動退出手勢相沖突，那麼我們可以在應用中禁用它。為了禁用滑動退出手勢，需要繼承默認的theme，然後設置`android:windowSwipeToDismiss` 屬性為`false`：

```xml
<style name="AppTheme" parent="Theme.DeviceDefault">
    <item name="android:windowSwipeToDismiss">false</item>
</style>
```
	
如果我們禁用了這個手勢，那麼我們需要實現長按退出UI模型來讓用戶退出我們的應用，下面的章節會介紹相關內容。

## 實現長按退出模式

要在activity中使用`DissmissOverlayView`類，添加下面這個節點到layout文件，讓它全屏且覆蓋在所有其他view上。例如：

```xml
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_height="match_parent"
    android:layout_width="match_parent">

    <!-- other views go here -->

    <android.support.wearable.view.DismissOverlayView
        android:id="@+id/dismiss_overlay"
        android:layout_height="match_parent"
        android:layout_width="match_parent"/>
<FrameLayout>
```
	
在我們的activity中，取得`DismissOverlayView`元素並設置一些提示文字。這些文字會在用戶第一次運行我們的應用時提醒用戶可以使用長按手勢退出應用。接著用`GestureDetector`檢測長按動作：

```java
public class WearActivity extends Activity {

    private DismissOverlayView mDismissOverlay;
    private GestureDetector mDetector;

    public void onCreate(Bundle savedState) {
        super.onCreate(savedState);
        setContentView(R.layout.wear_activity);

        // Obtain the DismissOverlayView element
        mDismissOverlay = (DismissOverlayView) findViewById(R.id.dismiss_overlay);
        mDismissOverlay.setIntroText(R.string.long_press_intro);
        mDismissOverlay.showIntroIfNecessary();

        // Configure a gesture detector
        mDetector = new GestureDetector(this, new SimpleOnGestureListener() {
            public void onLongPress(MotionEvent ev) {
                mDismissOverlay.show();
            }
        });
    }

    // Capture long presses
    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        return mDetector.onTouchEvent(ev) || super.onTouchEvent(ev);
    }
}
```
	
當系統檢測到一個長按動作，`DismissOverlayView`會顯示一個**退出**按鈕。如果用戶點擊它，那麼我們的activity會被終止。