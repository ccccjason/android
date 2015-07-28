# 創建單元測試

> 編寫:[huanglizhuo](https://github.com/huanglizhuo) - 原文:<http://developer.android.com/training/activity-testing/activity-unit-testing.html>

[Activity](http://developer.android.com/reference/android/app/Activity.html)單元測試可以快速且獨立地（和系統其它部分分離）驗證一個[Activity](http://developer.android.com/reference/android/app/Activity.html)的狀態以及其與其它組件交互的正確性。一個單元測試通常用來測試代碼中最小單位的代碼塊（可以是一個方法，類，或者組件），而且也不依賴於系統或網絡資源。比如說，你可以寫一個單元測試去檢查Activity是否正確地佈局或者是否可以正確地觸發一個Intent對象。

單元測試一般不適合測試與系統有複雜交互的UI。我們應該使用如同[測試UI組件](activity-ui-testing.md)所描述的`ActivityInstrumentationTestCase2`來對這類UI交互進行測試。

這節內容將會講解如何編寫一個單元測試來驗證一個[Intent](http://developer.android.com/reference/android/content/Intent.html)是否正確地觸發了另一個[Activity](http://developer.android.com/reference/android/app/Activity.html)。由於測試是與環境獨立的，所以[Intent](http://developer.android.com/reference/android/content/Intent.html)實際上並沒有發送給Android系統，但我們可以檢查Intent對象的載荷數據是否正確。讀者可以參考一下示例代碼中的`LaunchActivityTest.java`，將它作為一個例子，瞭解完備的測試用例是怎麼樣的。

> **注意**: 如果要針對系統或者外部依賴進行測試，我們可以使用Mocking Framework的Mock類，並把它集成到我們的你的單元測試中。要了解更多關於Android提供的Mocking Framework內容請參考[Mock Object Classes](http://developer.android.com/tools/testing/testing_android.html#MockObjectClasses)。

## 編寫一個Android單元測試例子

ActiviUnitTestCase類提供對於單個[Activity](http://developer.android.com/reference/android/app/Activity.html)進行分離測試的支持。要創建單元測試，我們的測試類應該繼承自`ActiviUnitTestCase`。繼承`ActiviUnitTestCase`的Activity不會被Android自動啟動。要單獨啟動Activity，我們需要顯式的調用startActivity()方法，並傳遞一個[Intent](http://developer.android.com/reference/android/content/Intent.html)來啟動我們的目標[Activity](http://developer.android.com/reference/android/app/Activity.html)。

例如：

```java
public class LaunchActivityTest
        extends ActivityUnitTestCase<LaunchActivity> {
    ...

    @Override
    protected void setUp() throws Exception {
        super.setUp();
        mLaunchIntent = new Intent(getInstrumentation()
                .getTargetContext(), LaunchActivity.class);
        startActivity(mLaunchIntent, null, null);
        final Button launchNextButton =
                (Button) getActivity()
                .findViewById(R.id.launch_next_activity_button);
    }
}
```

## 驗證另一個Activity的啟動

我們的單元測試目標可能包括:

* 驗證當Button被按下時，啟動的LaunchActivity是否正確。
* 驗證啟動的Intent是否包含有效的數據。

為了驗證一個觸發[Intent](http://developer.android.com/reference/android/content/Intent.html)的Button的事件，我們可以使用<a href="http://developer.android.com/reference/android/test/ActivityUnitTestCase.html#getStartedActivityIntent()">getStartedActivityIntent()</a>方法。通過使用斷言方法，我們可以驗證返回的[Intent](http://developer.android.com/reference/android/content/Intent.html)是否為空，以及是否包含了預期的數據來啟動下一個Activity。如果兩個斷言值都是真，那麼我們就成功地驗證了Activity發送的Intent是正確的了。

我們可以這樣實現測試方法:

```java
@MediumTest
public void testNextActivityWasLaunchedWithIntent() {
    startActivity(mLaunchIntent, null, null);
    final Button launchNextButton =
            (Button) getActivity()
            .findViewById(R.id.launch_next_activity_button);
    launchNextButton.performClick();

    final Intent launchIntent = getStartedActivityIntent();
    assertNotNull("Intent was null", launchIntent);
    assertTrue(isFinishCalled());

    final String payload =
            launchIntent.getStringExtra(NextActivity.EXTRAS_PAYLOAD_KEY);
    assertEquals("Payload is empty", LaunchActivity.STRING_PAYLOAD, payload);
}
```

因為LaunchActivity是獨立運行的，所以不可以使用[TouchUtils](http://developer.android.com/reference/android/test/TouchUtils.html)庫來操作UI。如果要直接進行[Button](http://developer.android.com/reference/android/widget/Button.html)點擊，我們可以調用<a href="http://developer.android.com/reference/android/view/View.html#performClick()">perfoemClick()</a>方法。

本節示例代碼[AndroidTestingFun.zip](http://developer.android.com/shareables/training/AndroidTestingFun.zip)

