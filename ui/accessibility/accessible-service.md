# 開發輔助服務

> 編寫:[K0ST](https://github.com/K0ST) - 原文:<http://developer.android.com/training/accessibility/service.html>

本課程將教您：

1. 創建可達性服務(Accessibility Service)

2. 配置可達性服務(Accessibility Service)

3. 響應可達性事件(AccessibilityEvents)

4. 從View層級中提取更多信息

Accessibility Service是Android系統框架提供給安裝在設備上應用的一個可選的導航反饋特性。Accessibility Service 可以替代應用與用戶交流反饋，比如將文本轉化為語音提示，或是用戶的手指懸停在屏幕上一個較重要的區域時的觸摸反饋等。本課程將教您如何創建一個Accessibility Service，同時處理來自應用的信息，並將這些信息反饋給用戶。

## 創建Accessibility Service

Accessibility Service可以綁定在一個正常的應用中，或者是單獨的一個Android項目都可以。創建一個Accessibility Service的步驟與創建普通Service的步驟相似，在你的項目中創建一個繼承於[AccessibilityService](http://developer.android.com/reference/android/accessibilityservice/AccessibilityService.html)的類：

```java
package com.example.android.apis.accessibility;

import android.accessibilityservice.AccessibilityService;

public class MyAccessibilityService extends AccessibilityService {
...
    @Override
    public void onAccessibilityEvent(AccessibilityEvent event) {
    }

    @Override
    public void onInterrupt() {
    }

...
}
```

與其他Service類似，你必須在manifest文件當中聲明這個Service。記得標明它監聽處理了`android.accessibilityservice`事件，以便Service在其他應用產生[AccessibilityEvent](http://developer.android.com/reference/android/view/accessibility/AccessibilityEvent.html)的時候被調用。

```xml
<application ...>
...
<service android:name=".MyAccessibilityService">
     <intent-filter>
         <action android:name="android.accessibilityservice.AccessibilityService" />
     </intent-filter>
     . . .
</service>
...
</application>
```

如果你為這個Service創建了一個新項目，且僅僅是一個Service而不準備做成一個應用，那麼你就可以移除啟動的Activity(一般為MainActivity.java)，同樣也記得在manifest中將這個Activity聲明移除。

## 配置Accessibility Service

設置Accessibility Service的配置變量會告訴系統如何讓Service運行與何時運行。你希望響應哪種類型的事件？Service是否對所有的應用有效還是對部分指定包名的應用有效？使用哪些不同類型的反饋？

你有兩種設置這些變量屬性的方法，一種向下兼容的辦法是通過代碼來進行設定，使用`setServiceInfo`([android.accessibilityservice.AccessibilityServiceInfo](http://developer.android.com/reference/android/accessibilityservice/AccessibilityService.html#setServiceInfo(android.accessibilityservice.AccessibilityServiceInfo))。你需要重寫(*override*)`onServiceConnected()`方法，並在這裡進行Service的配置。

```java
@Override
public void onServiceConnected() {
    // Set the type of events that this service wants to listen to.  Others
    // won't be passed to this service.
    info.eventTypes = AccessibilityEvent.TYPE_VIEW_CLICKED |
            AccessibilityEvent.TYPE_VIEW_FOCUSED;

    // If you only want this service to work with specific applications, set their
    // package names here.  Otherwise, when the service is activated, it will listen
    // to events from all applications.
    info.packageNames = new String[]
            {"com.example.android.myFirstApp", "com.example.android.mySecondApp"};

    // Set the type of feedback your service will provide.
    info.feedbackType = AccessibilityServiceInfo.FEEDBACK_SPOKEN;

    // Default services are invoked only if no package-specific ones are present
    // for the type of AccessibilityEvent generated.  This service *is*
    // application-specific, so the flag isn't necessary.  If this was a
    // general-purpose service, it would be worth considering setting the
    // DEFAULT flag.

    // info.flags = AccessibilityServiceInfo.DEFAULT;

    info.notificationTimeout = 100;

    this.setServiceInfo(info);

}
```

在Android 4.0之後，就用另一種方式來設置了：通過設置XML文件來進行配置。一些特性的選項比如`canRetrieveWindowContent`僅僅可以在XML可以配置。對於上面所示的相應的配置，利用XML配置如下：

```xml
<accessibility-service
     android:accessibilityEventTypes="typeViewClicked|typeViewFocused"
     android:packageNames="com.example.android.myFirstApp, com.example.android.mySecondApp"
     android:accessibilityFeedbackType="feedbackSpoken"
     android:notificationTimeout="100"
     android:settingsActivity="com.example.android.apis.accessibility.TestBackActivity"
     android:canRetrieveWindowContent="true"
/>
```
如果你確定是通過XML進行配置，那麼請確保在manifest文件中通過< meta-data >標籤指定這個配置文件。假設此配置文件存放的地址為：`res/xml/serviceconfig.xml`，那麼標籤應該如下:

```xml
<service android:name=".MyAccessibilityService">
     <intent-filter>
         <action android:name="android.accessibilityservice.AccessibilityService" />
     </intent-filter>
     <meta-data android:name="android.accessibilityservice"
     android:resource="@xml/serviceconfig" />
</service>
```

## 響應Accessibility Event

現在你的Service已經配置好並可以監聽Accessibility Event了，來寫一些響應這些事件的代碼吧！首先就是要重寫*onAccessibilityEvent(AccessibilityEvent)*方法，在這個方法中，使用`getEventType()`來確定事件的類型，使用`getContentDescription()`來提取產生事件的View的相關的文本標籤。

```java
@Override
public void onAccessibilityEvent(AccessibilityEvent event) {
    final int eventType = event.getEventType();
    String eventText = null;
    switch(eventType) {
        case AccessibilityEvent.TYPE_VIEW_CLICKED:
            eventText = "Focused: ";
            break;
        case AccessibilityEvent.TYPE_VIEW_FOCUSED:
            eventText = "Focused: ";
            break;
    }

    eventText = eventText + event.getContentDescription();

    // Do something nifty with this text, like speak the composed string
    // back to the user.
    speakToUser(eventText);
    ...
}
```

## 從View層級中提取更多信息

這一步並不是必要步驟，但是卻非常有用。Android 4.0版本中增加了一個新特性，就是能夠用AccessibilityService來遍歷View層級，並從產生Accessibility 事件的組件與它的父子組件中提取必要的信息。為了實現這個目的，你需要在XML文件中進行如下的配置：

```xml
android:canRetrieveWindowContent="true"
```

一旦完成，使用[getSource()](http://developer.android.com/reference/android/view/accessibility/AccessibilityRecord.html#getSource())獲取一個[AccessibilityNodeInfo](http://developer.android.com/reference/android/view/accessibility/AccessibilityNodeInfo.html)對象，如果觸發事件的窗口是活動窗口，該調用只返回一個對象，如果不是,它將返回null，做出相應的反響。下面的示例是一個代碼片段,當它接收到一個事件時,執行以下步驟:


1. 立即獲取到產生這個事件的Parent
2. 在這個Parent中尋找文本標籤或勾選框
3. 如果找到，創建一個文本內容來反饋給用戶，提示內容和是否已勾選。
4. 如果當遍歷View的時候某處返回了null值，那麼就直接結束這個方法。

```java
// Alternative onAccessibilityEvent, that uses AccessibilityNodeInfo

@Override
public void onAccessibilityEvent(AccessibilityEvent event) {

    AccessibilityNodeInfo source = event.getSource();
    if (source == null) {
        return;
    }

    // Grab the parent of the view that fired the event.
    AccessibilityNodeInfo rowNode = getListItemNodeInfo(source);
    if (rowNode == null) {
        return;
    }

    // Using this parent, get references to both child nodes, the label and the checkbox.
    AccessibilityNodeInfo labelNode = rowNode.getChild(0);
    if (labelNode == null) {
        rowNode.recycle();
        return;
    }

    AccessibilityNodeInfo completeNode = rowNode.getChild(1);
    if (completeNode == null) {
        rowNode.recycle();
        return;
    }

    // Determine what the task is and whether or not it's complete, based on
    // the text inside the label, and the state of the check-box.
    if (rowNode.getChildCount() < 2 || !rowNode.getChild(1).isCheckable()) {
        rowNode.recycle();
        return;
    }

    CharSequence taskLabel = labelNode.getText();
    final boolean isComplete = completeNode.isChecked();
    String completeStr = null;

    if (isComplete) {
        completeStr = getString(R.string.checked);
    } else {
        completeStr = getString(R.string.not_checked);
    }
    String reportStr = taskLabel + completeStr;
    speakToUser(reportStr);
}
```
現在你已經實現了一個完整可運行的Accessibility Service。嘗試著調整它與用戶的交互方式吧！比如添加語音引擎，或者添加震動來提供觸覺上的反饋都是不錯的選擇！
