# 創建與執行測試用例

> 編寫:[huanglizhuo](https://github.com/huanglizhuo) - 原文:<http://developer.android.com/training/activity-testing/activity-basic-testing.html>

為了驗證應用的佈局設計和功能是否符合預期，為應用的每個Activity建立測試非常重要。對於每一個測試，我們需要在測試用例中創建一個個獨立的部分，包括測試數據，前提條件和[Activity](http://developer.android.com/reference/android/app/Activity.html)的測試方法。之後我們就可以運行測試並得到測試報告。如果有任何測試沒有通過，這表明在我們代碼中可能有潛在的缺陷。

> **注意**: 在測試驅動開發（TDD）方法中, 不推薦先編寫大部分或整個應用，並在開發完成後再運行測試。而是應該先編寫測試，然後及時編寫正確的代碼，以通過測試。通過更新測試案例來反映新的功能需求，並以此反覆。

## 創建一個測試用例

[Activity](http://developer.android.com/reference/android/app/Activity.html)測試都是通過結構化的方式編寫的。請務必把測試代碼放在一個單獨的包內，從而與被測試的代碼分開。

按照慣例，測試包的名稱應該遵循與應用包名相同的命名方式，在應用包名後接“.tests”。在創建的測試包中，為我們的測試用例添加Java類。按照慣例，測試用例名稱也應遵循要測試的Java或Android的類相同的名稱，並增加後綴“Test”。

要在Eclipse中創建一個新的測試用例可遵循如下步驟：

a. 在Package Explorer中，右鍵點擊待測試工程的src/文件夾，**New > Package**。

b. 設置文件夾名稱`<你的包名稱>.tests`（比如, `com.example.android.testingfun.tests`）並點擊**Finish**。

c. 右鍵點擊創建的測試包，並選擇**New > Calss**。

d. 設置文件名稱`<你的Activity名稱>Test`（比如, `MyFirstTestActivityTest`），然後點擊**Finish**。

## 建立測試數據集(Fixture)

測試數據集包含運行測試前必須生成的一些對象。要建立測試數據集，可以在我們的測試中覆寫<a href="http://developer.android.com/reference/junit/framework/TestCase.html#setUp()">setUp()</a>和<a href="http://developer.android.com/reference/junit/framework/TestCase.html#tearDown()">tearDown()</a>方法。測試會在運行任何其它測試方法之前自動執行<a href="http://developer.android.com/reference/junit/framework/TestCase.html#setUp()">setUp()</a>方法。我們可以用這些方法使得被測試代碼與測試初始化和清理是分開的。

在你的Eclipse中建立測試數據集:

1 . 在 Package Explorer中雙擊測試打開之前編寫的測試用例，然後修改測試用例使它繼承[ActivityTestCase](http://developer.android.com/reference/android/test/ActivityTestCase.html)的子類。比如：

```java
public class MyFirstTestActivityTest
        extends ActivityInstrumentationTestCase2<MyFirstTestActivity> {
```

2 . 下一步，給測試用例添加構造函數和setUp()方法，併為我們想測試的Activity添加變量聲明。比如:

```java
public class MyFirstTestActivityTest
        extends ActivityInstrumentationTestCase2<MyFirstTestActivity> {

    private MyFirstTestActivity mFirstTestActivity;
    private TextView mFirstTestText;

    public MyFirstTestActivityTest() {
        super(MyFirstTestActivity.class);
    }

    @Override
    protected void setUp() throws Exception {
        super.setUp();
        mFirstTestActivity = getActivity();
        mFirstTestText =
                (TextView) mFirstTestActivity
                .findViewById(R.id.my_first_test_text_view);
    }
}
```

構造函數是由測試用的Runner調用，用於初始化測試類的，而<a href="http://developer.android.com/reference/junit/framework/TestCase.html#setUp()">setUp()</a>方法是由測試Runner在其他測試方法開始前運行的。

通常在`setUp()`方法中，我們應該:

* 為`setUp()` 調用父類構造函數，這是JUnit要求的。
* 初始化測試數據集的狀態，具體而言：
    * 定義保存測試數據及狀態的實例變量
    * 創建並保存正在測試的[Activity](http://developer.android.com/reference/android/app/Activity.html)的引用實例。
    * 獲得想要測試的[Activity](http://developer.android.com/reference/android/app/Activity.html)中任何UI組件的引用。

我們可以使用<a href="http://developer.android.com/reference/android/test/ActivityInstrumentationTestCase2.html#getActivity()">getActivity()</a>方法得到正在測試的[Activity](http://developer.android.com/reference/android/app/Activity.html)的引用。

## 增加一個測試前提

我們最好在執行測試之前，檢查測試數據集的設置是否正確，以及我們想要測試的對象是否已經正確地初始化。這樣，測試就不會因為有測試數據集的設置錯誤而失敗。按照慣例，驗證測試數據集的方法被稱為`testPreconditions()`。

例如，我們可能想添加一個像這樣的`testPreconditons()`方法:

```java
public void testPreconditions() {
    assertNotNull(“mFirstTestActivity is null”, mFirstTestActivity);
    assertNotNull(“mFirstTestText is null”, mFirstTestText);
}
```

Assertion（斷言，譯者注）方法源自於Junit[Assert](http://developer.android.com/reference/junit/framework/Assert.html)類。通常，我們可以使用斷言來驗證某一特定的條件是否是真的。

* 如果條件為假，斷言方法拋出一個AssertionFailedError異常，通常會由測試Runner報告。我們可以在斷言失敗時給斷言方法添加一個字符串作為第一個參數從而給出一些上下文詳細信息。
* 如果條件為真，測試通過。

在這兩種情況下，Runner都會繼續運行其它測試用例的測試方法。

## 添加一個測試方法來驗證Activity

下一步，添加一個或多個測試方法來驗證[Activity](http://developer.android.com/reference/android/app/Activity.html)佈局和功能。

例如，如果我們的Activity含有一個[TextView](http://developer.android.com/reference/android/widget/TextView.html)，可以添加如下方法來檢查它是否有正確的標籤文本:

```java
public void testMyFirstTestTextView_labelText() {
    final String expected =
            mFirstTestActivity.getString(R.string.my_first_test);
    final String actual = mFirstTestText.getText().toString();
    assertEquals(expected, actual);
}
```

該 `testMyFirstTestTextView_labelText()` 方法只是簡單的檢查Layout中[TextView](http://developer.android.com/reference/android/widget/TextView.html)的默認文本是否和`strings.xml`資源中定義的文本一樣。

>**注意**：當命名測試方法時，我們可以使用下劃線將被測試的內容與測試用例區分開。這種風格使得我們可以更容易分清哪些是測試用例。

做這種類型的字符串比較時，推薦從資源文件中讀取預期字符串，而不是在代碼中硬性編寫字符串做比較。這可以防止當資源文件中的字符串定義被修改時，會影響到測試的效果。

為了進行比較，預期的和實際的字符串都要做為<a href="http://developer.android.com/reference/junit/framework/Assert.html#assertEquals(java.lang.String, java.lang.String)">assertEquals()</a>方法的參數。如果值是不一樣的，斷言將拋出一個[AssertionFailedError](http://developer.android.com/reference/junit/framework/AssertionFailedError.html)異常。

如果添加了一個`testPreconditions()`方法，我們可以把測試方法放在testPreconditions之後。

要參看一個完整的測試案例，可以參考本節示例中的MyFirstTestActivityTest.java。

##構建和運行測試

我們可以在Eclipse中的包瀏覽器（Package Explorer）中運行我們的測試。

利用如下步驟構建和運行測試:

1. 連接一個Android設備，在設備或模擬器中，打開設置菜單，選擇開發者選項並確保啟用USB調試。

2. 在包瀏覽器(Package Explorer)中，右鍵單擊測試類，並選擇**Run As > Android Junit Test**。

3. 在Android設備選擇對話框，選擇剛才連接的設備，然後單擊“確定”。

4. 在JUnit視圖，驗證測試是否通過,有無錯誤或失敗。

本節示例代碼[AndroidTestingFun.zip](http://developer.android.com/shareables/training/AndroidTestingFun.zip)
