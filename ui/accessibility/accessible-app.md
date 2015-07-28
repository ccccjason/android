# 開發輔助程序

> 編寫:[K0ST](https://github.com/K0ST) - 原文:<http://developer.android.com/training/accessibility/accessible-app.html>

本課程將教您：

1. 添加內容描述(*Content Descriptions*)

2. 設計焦點導航（*Focus Navigation*）

3. 觸發可達性事件(*Accessibility Events*)

4. 測試你的程序

Android平臺本身有一些專注可達性的特性，這些特性可以幫助你專門為那些視覺上或生理上有缺陷的用戶在應用上做特別的優化。然而，正確的優化方式或最簡單利用這個特性的方法往往不是那麼顯而易見的。本課程將給您演示如何利用和實現這些策略和平臺的特性功能，構建一個更友好的具有可達性的Android應用。

## 添加內容描述

一個好的交互界面上的元素通常不需要特別使用一個標籤來表明這個元素的作用。例如對於一個任務型應用來說，一個項目旁邊的勾選框表達的意思就非常明確，或者對於一個文件管理應用，垃圾桶的圖標表達的意思也非常清除。然而對於具有視覺障礙的用戶來說，其他類型的UI交互提示是有必要的。

幸運的是，我們可以很輕鬆的給一個UI元素加上標籤，這樣類似於[TalkBack](https://play.google.com/store/apps/details?id=com.google.android.marvin.talkback)這樣的基於語音的Accessibility Service就可以將標籤的內容朗讀出來。如果你的標籤在整個應用的生命週期中不太可能會發生變化(*比如‘停止’或者‘購買’*)，你就可以在XML佈局文件中對*android:contentDescription*屬性進行設置。代碼如下：

```xml
<Button
    android:id=”@+id/pause_button”
    android:src=”@drawable/pause”
    android:contentDescription=”@string/pause”/>
```
然而，在很多情況下描述的內容是基於上下文環境的，比如說一個開關按鈕的狀態，或者在list中一片可選的數據項。在運行時編輯內容描述可以使用*setContentDescription()*方法，代碼如下：

```java
String contentDescription = "Select " + strValues[position];
label.setContentDescription(contentDescription);
```

將以上功能添加進您的代碼是提高您應用可達性的最簡單的方法。嘗試著將那些有用的地方都加入內容描述，但同時要避免像web開發者那樣將所有的元素都標註，那樣會產生大量的無用信息。比如說，不要將應用圖標的內容描述設置為*‘應用圖標’*。這隻會對用戶的瀏覽產生干擾。

來試試吧！下載TalkBack(谷歌開發的一款可達性應用)，在**Settings > Accessibility > TalkBack**將它開啟。然後使用你的應用聽聽看TalkBack發出的語音提示。


## 設計焦點導航

你的應用除了支持觸摸操作外，更應該支持其他的導航方式。很多Android設備不僅僅提供了觸摸屏，還提供了其他的導航硬件比如說十字鍵、方向鍵、軌跡球等等。除此之外，最新的Android發行版本也支持藍牙或USB的外接設備，比如鍵盤等等。

為了實現這種方式的導航，一切用戶可以用來可導航的元素(*navigational elements*)都需要設置為focusable（*聚焦*）,它可以在運行時通過*View.setFocusable()*方法來進行設定，或者也可以在XML佈局文件中使用*android:focusable*來設置。

每個UI控件有四個屬性，*android:nextFocusUp*,*android:nextFocusDown*,*android:nextFocusLeft*,*android:nextFocusRight*,用戶在導航時可以利用這些屬性來指定下一個焦點的位置。系統會自動根據佈局的方向來確定導航的順序，如果在您的應用中系統提供的方案並不合適，您可以用這些屬性來進行自定義的修改。

比如說，下面就是一個關於按鈕和標籤的例子，他們都是可聚焦的(*focusable*)，按向下鍵會將焦點從按鈕移到文字上，按向上會重新將焦點移到按鈕上。
```xml
<Button android:id="@+id/doSomething"
    android:focusable="true"
    android:nextFocusDown=”@id/label”
    ... />
<TextView android:id="@+id/label"
    android:focusable=”true”
    android:text="@string/labelText"
    android:nextFocusUp=”@id/doSomething”
    ... />
```
證實您的應用運行正確的直觀方法，最簡單的方式就是在Android虛擬機裡運行您的應用，然後使用虛擬器的方向鍵來在各個元素之間導航，使用OK按鈕來代替觸摸操作。

## 觸發可達性事件

如果你在你的Android框架中使用了View組件，當你選中了一個View或者是焦點變化的時候，可達性事件(*AccessibilityEvent*)都會產生。這些事件會被傳遞到Accessibility Service中進行處理，實現一些輔助功能，如語音提示等。

如果你寫了一個自定義的View，請確保它在合適的時候產生事件。使用*sendAccessibilityEvent(int)*函數可以產生可達性事件，其中的參數表示事件的類型。完整的可達性事件類型可查閱[AccessibilityEvent](http://developer.android.com/reference/android/view/accessibility/AccessibilityEvent.html)參考文檔。

比如說，你拓展了一個圖片的View，你希望在它聚焦的時候使用鍵盤打字可以在其中插入題注，這時候發送一個*TYPE_VIEW_TEXT_CHANGED*事件就非常合適，儘管它不是本身就構建在這個圖片View中的。產生事件的代碼如下：
```java
public void onTextChanged(String before, String after) {
    ...
    if (AccessibilityManager.getInstance(mContext).isEnabled()) {
        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_TEXT_CHANGED);
    }
    ...
}
```

## 測試你的程序

請確保您在添加可達性功能後測試它的有效性。為了測試內容描述可達性事件，請安裝並啟用一個Accessibility Service。比如說使用TalkBack，它是一個免費的開源的屏幕讀取軟件，可在Google Play上進行下載。Service啟動後，請測試您應用中所有的功能，同時聽聽TalkBack的語音反饋。

同時，嘗試著用一個方向控制器來控制你的應用，而非使用直接觸摸的方式。你可以使用一個物理設備，比如十字鍵、軌跡球等。如果沒有條件，可以使用android虛擬器，它提供了虛擬的按鍵控制。

在測試導航與反饋的同時，和在沒有任何視覺提示的情況下，應該對你的應用大概是一個什麼樣子有所認識。出現問題就修復優化它們，最終就會開發出一個更易用可達的Android程序。



