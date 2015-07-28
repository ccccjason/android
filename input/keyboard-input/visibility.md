# 處理輸入法可見性

> 編寫:[zhaochunqi](https://github.com/zhaochunqi) - 原文:<http://developer.android.com/training/keyboard-input/visibility.html>

當輸入焦點移入或移出可編輯的文本框時，Android會相應的顯示或隱藏輸入法（如虛擬鍵盤）。系統也會決定輸入法上方的 UI 和文本框的顯示方式。舉例來說，當屏幕上垂直空間被壓縮時，文本框可能填充輸入法上方所有的空間。對於多數的應用來說，這些默認的行為基本就足夠了。

然而，在一些事例中，我們可能會想要更加直接地控制輸入法的顯示，指定在輸入法顯示的時候，如何顯示我們的佈局。這節課會解釋如何控制和響應輸入法的可見性。

<a name="ShowOnStart"></a>
## 在Activity啟動時顯示輸入法

儘管Android會在Activity啟動時將焦點放在佈局中的第一個文本框，但是並不會顯示輸入法。因為輸入文本可能並不是activity中的首要任務，所以不顯示輸入法是很合理的。可是，如果輸入文本確實是首要的任務（如在登錄界面中），那麼可能需要默認顯示輸入法。

為了在activity啟動時顯示輸入法，添加 [android:windowSoftInputMode](http://developer.android.com/guide/topics/manifest/activity-element.html#wsoft)  屬性到 &lt;activity&gt; 節點中，並將該屬性的值設為 `"stateVisible"`。如下：

```xml
<application ... >
    <activity
        android:windowSoftInputMode="stateVisible" ... >
        ...
    </activity>
    ...
</application>
```

> **Note:** 如果用戶的設備有一個實體鍵盤，那麼*不會*顯示軟輸入法。

## 根據需要顯示輸入法

如果我們想要確保輸入法在activity生命週期的某個方法中是可見的，那麼可以使用 [InputMethodManager](http://developer.android.com/reference/android/view/inputmethod/InputMethodManager.html) 來實現。

舉例來說，下面的方法調用了一個需要用戶填寫文本的[View](http://developer.android.com/reference/android/view/View.html)，調用了 <a href="http://developer.android.com/reference/android/view/View.html#requestFocus()">requestFocus()</a> 來獲取焦點，然後調用 <a href="http://developer.android.com/reference/android/view/inputmethod/InputMethodManager.html#showSoftInput(android.view.View, int)">showSoftInput()</a> 來打開輸入法。

```java
public void showSoftKeyboard(View view) {
    if (view.requestFocus()) {
        InputMethodManager imm = (InputMethodManager)
                getSystemService(Context.INPUT_METHOD_SERVICE);
        imm.showSoftInput(view, InputMethodManager.SHOW_IMPLICIT);
    }
}
```

> **Note:** 一旦輸入法可見，我們不應該以編程的方式來隱藏它。系統會在用戶結束文本框的任務時隱藏輸入法，或者可以使用系統控制（如*返回*鍵）來隱藏。

## 指定 UI 的響應方式

當輸入法顯示在屏幕上時，會減少 app UI 中的可用空間。系統會決定如何調整 UI 可見的部分，但是這樣做不一定正確。為了確保應用的最佳表現，我們應該在 UI 的剩餘空間中展示我們想要展示的系統界面。

為了在activity中聲明合適的處理方法，可以在 manifest 文件的 &lt;activity&gt; 節點中使用 [android:windowSoftInputMode](http://developer.android.com/guide/topics/manifest/activity-element.html#wsoft) 屬性，並將該屬性的值設為"adjust"。

舉例來說，為了確保系統會在可用空間中重新調整佈局的大小——確保所有的佈局內容都可以被使用（儘管可能需要滑動）——使用 `"adjustResize"`:

```xml
<application ... >
    <activity
        android:windowSoftInputMode="adjustResize" ... >
        ...
    </activity>
    ...
</application>
```

我們可以結合上述調整說明和[初始化輸入法可見性](#ShowOnStart)說明：

```xml
    <activity
        android:windowSoftInputMode="stateVisible|adjustResize" ... >
        ...
    </activity>
```

如果 UI 中包含用戶可能需要在文本輸入時立即執行的事情，那麼使用 `"adjustResize"` 是很重要的。例如，如果我們使用相對佈局（relative layout）在屏幕底部放置一個按鈕，用 `"adjustResize"` 來重新調整大小，使得按鈕欄出現在輸入法上方。