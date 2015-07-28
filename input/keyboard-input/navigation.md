# 支持鍵盤導航

> 編寫:[zhaochunqi](https://github.com/zhaochunqi) - 原文:<http://developer.android.com/training/keyboard-input/navigation.html>

除了軟鍵盤輸入法（如虛擬鍵盤）以外，Android支持將物理鍵盤連接到設備上。鍵盤不僅方便輸入文本，而且提供一種方法來導航和與應用交互。儘管多數的手持設備（如手機）使用觸摸作為主要的交互方式，但是隨著平板和一些類似的設備正在逐步流行起來，許多用戶開始喜歡外接鍵盤。

隨著更多的Android設備提供這種體驗，優化應用以支持通過鍵盤與應用進行交互變得越來越重要。這節課介紹了怎樣為鍵盤導航提供更好的支持。

> **Note:** 對那些沒有使用可見導航提示的應用來說，在應用中支持方向性的導航對於應用的可用性也是很重要的。在我們的應用中完全支持方向導航還可以幫助我們使用諸如 [uiautomator](http://developer.android.com/tools/help/uiautomator/index.html) 等工具進行[自動化用戶界面測試](http://developer.android.com/tools/testing/testing_ui.html)。

## 測試應用

因為Android系統默認開啟了大多必要的行為，所以用戶可能已經可以在我們的應用中使用鍵盤導航了。

所有由Android framework（如Button和EditText）提供的交互部件是可獲得焦點的。這意味著用戶可以使用如D-pad或鍵盤等控制設備，並且當某個部件被選中時，部件會發光或者改變外觀。

為了測試我們的應用：

1. 將應用安裝到一個帶有實體鍵盤的設備上。

	如果我們沒有帶實體鍵盤的設備，連接一個藍牙鍵盤或者USB鍵盤(儘管並不是所有的設備都支持USB連接)

	我們還可以使用Android模擬器：

	1. 在AVD管理器中，要麼點擊**New Device**，要麼選擇一個已存在的文檔點擊**Clone**。

	2. 在出現的窗口中，確保**Keyboard**和**D-pad**開啟。

2. 為了驗證我們的應用，只是用Tab鍵來進行UI導航，確保每一個UI控制的焦點與預期的一致。

	找到任何不在預期焦點的實例。

3. 從頭開始，使用方向鍵(鍵盤上的箭頭鍵)來控制應用的導航。

	在 UI 中每一個被選中的元素上，按上、下、左、右。

	找到每個不在預期焦點的實例。

如果我們找到任何使用Tab鍵或方向鍵後導航的效果不如預期的實例，那麼在佈局中指定焦點應該聚焦在哪裡，如下面幾部分所討論的。

## 處理Tab導航

當用戶使用鍵盤上的Tab鍵導航我們的應用時，系統會根據組件在佈局中的顯示順序，在組件之間傳遞焦點。如果我們使用相對佈局（relative layout），例如，在屏幕上的組件順序與佈局文件中組件的順序不一致，那麼我們可能需要手動指定焦點順序。

舉例來說，在下面的佈局文件中，兩個對齊右邊的按鈕和一個對齊第二個按鈕左邊的文本框。為了把焦點從第一個按鈕傳遞到文本框，然後再傳遞到第二個按鈕，佈局文件需要使用屬性 [android:nextFocusForward](http://developer.android.com/reference/android/view/View.html#attr_android:nextFocusForward)，清楚地為每一個可被選中的組件定義焦點順序：

```xml
<RelativeLayout ...>
    <Button
        android:id="@+id/button1"
        android:layout_alignParentTop="true"
        android:layout_alignParentRight="true"
        android:nextFocusForward="@+id/editText1"
        ... />
    <Button
        android:id="@+id/button2"
        android:layout_below="@id/button1"
        android:nextFocusForward="@+id/button1"
        ... />
    <EditText
        android:id="@id/editText1"
        android:layout_alignBottom="@+id/button2"
        android:layout_toLeftOf="@id/button2"
        android:nextFocusForward="@+id/button2"
        ...  />
    ...
</RelativeLayout>
```

現在焦點從 `button1` 到 `button2` 再到 `editText1`，改成了按照在屏幕上出現的順序：從 `button1` 到 `editText1` 再到 `button2`。

## 處理方向導航

用戶也能夠使用鍵盤上的方向鍵在我們的app中導航(這種行為與在D-pad和軌跡球中的導航一致)。系統提供了一個最佳猜測：根據屏幕上 view 的佈局，在給定的方向上，應該將交掉放在哪個 view 上。然而有時，系統會猜測錯誤。

當在給定的方向進行導航時，如果系統沒有傳遞焦點給合適的 View，那麼指定接收焦點的 view 來使用如下的屬性：

* [android:nextFocusUp](http://developer.android.com/reference/android/view/View.html#attr_android:nextFocusUp)
* [android:nextFocusDown](http://developer.android.com/reference/android/view/View.html#attr_android:nextFocusDown)
* [android:nextFocusLeft](http://developer.android.com/reference/android/view/View.html#attr_android:nextFocusLeft)
* [android:nextFocusRight](http://developer.android.com/reference/android/view/View.html#attr_android:nextFocusRight)

當用戶導航到那個方向時，每一個屬性指定了下一個接收焦點的 view，如根據 view ID 來指定。舉例來說：

```xml
<Button
    android:id="@+id/button1"
    android:nextFocusRight="@+id/button2"
    android:nextFocusDown="@+id/editText1"
    ... />
<Button
    android:id="@id/button2"
    android:nextFocusLeft="@id/button1"
    android:nextFocusDown="@id/editText1"
    ... />
<EditText
    android:id="@id/editText1"
    android:nextFocusUp="@id/button1"
    ...  />
```
