# 測試UI組件

> 編寫:[huanglizhuo](https://github.com/huanglizhuo) - 原文:<http://developer.android.com/training/activity-testing/activity-ui-testing.html>

通常情況下，[Activity](http://developer.android.com/reference/android/app/Activity.html)，包括用戶界面組件（如按鈕，複選框，可編輯的文本域，和選框）允許用戶與Android應用程序交互。本節介紹如何對一個簡單的帶有按鈕的界面交互測試。我們可以使用相同的步驟來測試其他更復雜的UI組件。

> **注意**: 這一節的測試方法叫做白盒測試，因為我們擁有要測試應用程序的源碼。Android Instrumentation框架適用於創建應用程序中UI部件的白盒測試。用戶界面測試的另一種類型是黑盒測試，即無法得知應用程序源代碼的類型。這種類型的測試可以用來測試應用程序如何與其他應用程序，或與系統進行交互。黑盒測試不包括在本節中。瞭解更多關於如何在你的Android應用程序進行黑盒測試，請閱讀[UI Testing guide](http://developer.android.com/tools/testing/testing_ui.html)。

要參看完整的測試案例，可以查看本節示例代碼中的`ClickFunActivityTest.java`文件。

## 使用 Instrumentation 建立UI測試

當測試擁有UI的Activity時，被測試的Activity在UI線程中運行。然而，測試程序會在程序自己的進程中，單獨的一個線程內運行。這意味著，我們的測試程序可以獲得UI線程的對象，但是如果它嘗試改變UI線程對象的值，會得到`WrongThreadException`錯誤。

為了安全地將`Intent`注入到`Activity`，或是在UI線程中執行測試方法，我們可以讓測試類繼承於[ActivityInstrumentationTestCase2](http://developer.android.com/reference/android/test/ActivityInstrumentationTestCase2.html)。要學習如何在UI線程運行測試方法，請看[在UI線程測試](http://developer.android.com/tools/testing/activity_testing.html#RunOnUIThread)。

### 建立測試數據集（Fixture）

當為UI測試建立測試數據集時，我們應該在<a href="http://developer.android.com/reference/junit/framework/TestCase.html#setUp()">setUp()</a>方法中指定[touch mode](http://developer.android.com/guide/topics/ui/ui-events.html#TouchMode)。把touch mode設置為真可以防止在執行編寫的測試方法時，人為的UI操作獲取到控件的焦點（比如,一個按鈕會觸發它的點擊監聽器）。確保在調用<a href="http://developer.android.com/reference/android/test/ActivityInstrumentationTestCase2.html#getActivity()">getActivity()</a>方法前調用了[setActivityInitialTouchMode](http://developer.android.com/reference/android/test/ActivityInstrumentationTestCase2.html#setActivityInitialTouchMode(boolean))。

比如:

```java
public class ClickFunActivityTest
        extends ActivityInstrumentationTestCase2 {
    ...
    @Override
    protected void setUp() throws Exception {
        super.setUp();

        setActivityInitialTouchMode(true);

        mClickFunActivity = getActivity();
        mClickMeButton = (Button)
                mClickFunActivity
                .findViewById(R.id.launch_next_activity_button);
        mInfoTextView = (TextView)
                mClickFunActivity.findViewById(R.id.info_text_view);
    }
}
```

## 添加測試方法確認UI響應表現

UI測試目標應包括:

*. 檢驗[Activity](http://developer.android.com/reference/android/app/Activity.html)啟動時[Button](http://developer.android.com/reference/android/widget/Button.html)在正確佈局位置顯示。
*. 檢驗[TextView](http://developer.android.com/reference/android/widget/TextView.html)初始化時是隱藏的。
*. 檢驗[TextView](http://developer.android.com/reference/android/widget/TextView.html)在[Button](http://developer.android.com/reference/android/widget/Button.html)點擊時顯示預期的字符串

接下來的部分會演示怎樣實現上述驗證方法

### 驗證Button佈局參數

我們應該像如下添加的測試方法那樣。驗證[Activity](http://developer.android.com/reference/android/app/Activity.html)中的按鈕是否正確顯示:

```java
@MediumTest
public void testClickMeButton_layout() {
    final View decorView = mClickFunActivity.getWindow().getDecorView();

    ViewAsserts.assertOnScreen(decorView, mClickMeButton);

    final ViewGroup.LayoutParams layoutParams =
            mClickMeButton.getLayoutParams();
    assertNotNull(layoutParams);
    assertEquals(layoutParams.width, WindowManager.LayoutParams.MATCH_PARENT);
    assertEquals(layoutParams.height, WindowManager.LayoutParams.WRAP_CONTENT);
}
```

在調用<a href="http://developer.android.com/reference/android/test/ViewAsserts.html#assertOnScreen(android.view.View, android.view.View)">assertOnScreen()</a>方法時，傳遞根視圖以及期望呈現在屏幕上的視圖作為參數。如果想呈現的視圖沒有在根視圖中,該方法會拋出一個[AssertionFailedError](http://developer.android.com/reference/junit/framework/AssertionFailedError.html)異常，否則測試通過。

我們也可以通過獲取一個[ViewGroup.LayoutParams](http://developer.android.com/reference/android/view/ViewGroup.LayoutParams.html)對象的引用驗證[Button](http://developer.android.com/reference/android/widget/Button.html)佈局是否正確，然後調用`assert`方法驗證[Button](http://developer.android.com/reference/android/widget/Button.html)對象的寬高屬性值是否與預期值一致。

`@MediumTest`註解指定測試是如何歸類的（和它的執行時間相關）。要了解更多有關測試的註解，見本節示例。

### 驗證TextView的佈局參數

可以像這樣添加一個測試方法來驗證[TextView](http://developer.android.com/reference/android/widget/TextView.html)最初是隱藏在[Activity](http://developer.android.com/reference/android/app/Activity.html)中的:

```java
@MediumTest
public void testInfoTextView_layout() {
    final View decorView = mClickFunActivity.getWindow().getDecorView();
    ViewAsserts.assertOnScreen(decorView, mInfoTextView);
    assertTrue(View.GONE == mInfoTextView.getVisibility());
}
```

我們可以調用`getDecorView()`方法得到一個[Activity](http://developer.android.com/reference/android/app/Activity.html)中修飾試圖（Decor View）的引用。要修飾的View在佈局層次視圖中是最上層的ViewGroup([FrameLayout](http://developer.android.com/reference/android/widget/FrameLayout.html))

### 驗證按鈕的行為

可以使用如下測試方法來驗證當按下按鈕時[TextView](http://developer.android.com/reference/android/widget/TextView.html)變得可見:

```java
@MediumTest
public void testClickMeButton_clickButtonAndExpectInfoText() {
    String expectedInfoText = mClickFunActivity.getString(R.string.info_text);
    TouchUtils.clickView(this, mClickMeButton);
    assertTrue(View.VISIBLE == mInfoTextView.getVisibility());
    assertEquals(expectedInfoText, mInfoTextView.getText());
}
```

在測試中調用<a href="http://developer.android.com/reference/android/test/TouchUtils.html#clickView(android.test.InstrumentationTestCase, android.view.View)">clickView()</a>可以讓我們用編程方式點擊一個按鈕。我們必須傳遞正在運行的測試用例的一個引用和要操作按鈕的引用。

> **注意**:[TouchUtils](http://developer.android.com/reference/android/test/TouchUtils.html)輔助類提供與應用程序交互的方法可以方便進行模擬觸摸操作。我們可以使用這些方法來模擬點擊，輕敲，或應用程序屏幕拖動View。

> **警告**[TouchUtils](http://developer.android.com/reference/android/test/TouchUtils.html)方法的目的是將事件安全地從測試線程發送到UI線程。我們不可以直接在UI線程或任何標註@UIThread的測試方法中使用[TouchUtils](http://developer.android.com/reference/android/test/TouchUtils.html)這樣做可能會增加錯誤線程異常。

## 應用測試註解

[@SmallTest](http://developer.android.com/reference/android/test/suitebuilder/annotation/SmallTest.html)

    標誌該測試方法是小型測試的一部分。

[@MediumTest](http://developer.android.com/reference/android/test/suitebuilder/annotation/MediumTest.html)

    標誌該測試方法是中等測試的一部分。

[@LargeTest](http://developer.android.com/reference/android/test/suitebuilder/annotation/LargeTest.html)

    標誌該測試方法是大型測試的一部分。

通常情況下，如果測試方法只需要幾毫秒的時間，那麼它應該被標記為[@SmallTest](http://developer.android.com/reference/android/test/suitebuilder/annotation/SmallTest.html)，長時間運行的測試（100毫秒或更多）通常被標記為[@MediumTest](http://developer.android.com/reference/android/test/suitebuilder/annotation/MediumTest.html)或[@LargeTest](http://developer.android.com/reference/android/test/suitebuilder/annotation/LargeTest.html)，這主要取決於測試訪問資源在網絡上或在本地系統。 可以參看[Android Tools Protip](https://plus.google.com/+AndroidDevelopers/posts/TPy1EeSaSg8)，它可以更好地指導我們使用測試註釋

我們可以創建其它的測試註釋來控制測試的組織和運行。要了解更多關於其他註釋的信息，見[Annotation](http://developer.android.com/reference/java/lang/annotation/Annotation.html)類參考。

本節示例代碼[AndroidTestingFun.zip](http://developer.android.com/shareables/training/AndroidTestingFun.zip)
