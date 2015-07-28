<!-- # Creating TV Navigation # -->
# 創建TV導航

> 編寫:[awong1900](https://github.com/awong1900) - 原文:<http://developer.android.com/training/tv/start/navigation.html>

<!-- TV devices provide a limited set of navigation controls for apps. Creating an effective navigation scheme for your TV app depends on understanding these limited controls and the limits of users' perception while operating your app. As you build your Android app for TVs, pay special attention to how the user actually Android navigates around your app when using remote control buttons instead of a touch screen.
-->

TV設備為應用程序提供一組有限的導航控件。為我們的TV應用創建有效的導航方案取決於理解這些有限的控件和用戶操作應用時的限制。因此當我們為TV創建Android應用時，額外注意用戶是用遙控器按鍵,而不是用觸摸屏導航我們的應用程序。

<!-- This lesson explains the minimum requirements for creating effective TV app navigation scheme and how to apply those requirements to your app. -->

這節課解釋了創建有效的TV應用導航方案的最低要求和如何對應用程序使用這些要求。

<!-- ## Enable D-pad Navigation ## -->
## 使用D-pad導航

<!-- On a TV device, users navigate with controls on a remote control device, using either a directional pad (D-pad) or arrow keys. This type of control limits movement to up, down, left, and right. To build a great TV-optimized app, you must provide a navigation scheme where the user can quickly learn how to navigate your app using these limited controls. -->

在TV設備上，用戶用遙控器設備的方向手柄（D-pad）或者方向鍵去控制控件。這類控制器限制為上下左右移動。為了創建最優化的TV應用，我們必須提供一個用戶能快速學習如何使用有限控件導航的方案。

<!-- The Android framework handles directional navigation between layout elements automatically, so you typically do not need to do anything extra for your app. However, you should thoroughly test navigation with a D-pad controller to discover any navigation problems. Follow these guidelines to test that your app's navigation system works well with a D-pad on a TV device: -->

Android framework自動地處理佈局元素之間的方向導航操作，因此我們不需要在應用中做額外的事情。不管怎樣，我們也應該用D-pad控制器實際測試去發現任何導航問題。接下來的指引是如何在TV設備上用D-pad測試應用的導航。

<!-- 
- Ensure that a user with a D-pad controller can navigate to all visible controls on the screen.
- For scrolling lists with focus, make sure that the D-pad up and down keys scroll the list, and the Enter key selects an item in the list. Verify that users can select an element in the list and that the list still scrolls when an element is selected.
- Ensure that switching between controls between controls is straightforward and predictable.
-->

- 確保用戶能用D-pad控制器導航所有屏幕可見的控件。
- 對於滾動列表上的焦點，確保D-pad上下鍵能滾動列表，並且確定鍵能選擇列表中的項。檢查用戶可以選擇列表中的元素並且選中元素後仍可以滾動列表。
- 確保在控件之間切換是直接的和可預測的。

<!-- ### Modifying directional navigation ### -->
### 修改導航的方向

<!-- The Android framework automatically applies a directional navigation scheme based on the relative position of focusable elements in your layouts. You should test the generated navigation scheme in your app using a D-pad controller. After testing, if you decide you want users to move through your layouts in a specific way, you can set up explicit directional navigation for your controls. -->

基於佈局元素中可選中的元素的相對位置，Android framwork自動應用導航方向方案。我們應該用D-pad控制器測試生成的導航方案。在測試後，如果我們想規定用戶以一個特定的方式在佈局中移動，我們可以在控件中設置明確的導航方向。

<!-- >**Note**: You should only use these attributes to modify the navigation order if the default order that the system applies does not work well. -->

>**Note**: 如果系統使用的默認順序不是很好，我們應該僅用這些屬性去修改導航順序。

<!-- The following code sample shows how to define the next control to receive focus for a TextView layout object: -->
接下來的示例代碼展示如何為TextView佈局控件定義下一個控件焦點。

```xml
<TextView android:id="@+id/Category1"
        android:nextFocusDown="@+id/Category2"\>
```

<!-- The following table lists all of the available navigation attributes for Android user interface widgets: -->
接下來的列表展示了用戶接口控件所有可用的導航屬性。

屬性          |	功能
:-----------|:----------------
[nextFocusDown](http://developer.android.com/reference/android/R.attr.html#nextFocusDown) |定義用戶按下導航時的焦點
[nextFocusLeft](http://developer.android.com/reference/android/R.attr.html#nextFocusLeft) |定義用戶按左導航時的焦點
[nextFocusRight](http://developer.android.com/reference/android/R.attr.html#nextFocusRight)|定義用戶按右導航時的焦點
[nextFocusUp](http://developer.android.com/reference/android/R.attr.html#nextFocusUp)   |定義用戶按上導航時的焦點

<!-- To use one of these explicit navigation attributes, set the value to the ID (android:id value) of another widget in the layout. You should set up the navigation order as a loop, so that the last control directs focus back to the first one. -->
去使用這些明確的導航屬性，設置另一個佈局控件的ID值（`android:id`值）。我們應該設置導航順序為一個循環，因此最後一個控件返回至第一個焦點。

<!-- ## Provide Clear Focus and Selection ## -->
## 提供清楚的焦點和選中狀態

<!-- The success of an app's navigation scheme on TV devices is depends on how easy it is for a user to determine what user interface element is in focus on screen. If you do not provide clear indications of focused items (and therefore what item a user can take action on), they can quickly become frustrated and exit your app. For the same reason, it is important to always have an item in focus that a user can take action on immediately after your app starts, or any time it is idle. -->

在TV設備上的應用導航方案的成功是基於用戶如何容易的決定屏幕上界面元素的焦點。如果我們不提供清晰的焦點項顯示（和用戶能操作的選項），他們會很快洩氣並退出我們的應用。同樣的原因，重要的是當我們的應用開始或者任何無操作的時間中，總是有焦點項可以立即操作。

<!-- Your app layout and implementation should use color, size, animation, or a combination of these attributes to help users easily determine what actions they can take next. Use a uniform scheme for indicating focus across your application. -->

我們的應用佈局和實現應該用顏色，大小，動畫或者它們組在一起來幫助用戶容易地決定下一步操作。在應用中用一致的焦點顯示方案。

<!-- Android provides Drawable State List Resources to implement highlights for focused and selected controls. The following code example demonstrates how to enable visual behavior for a button to indicate that a user has navigated to the control and then selected it: -->
Android提供[Drawable State List Resources](http://developer.android.com/guide/topics/resources/drawable-resource.html#StateList)來實現高亮選中的焦點。接下來的示例代碼展示瞭如何為用戶導航到控件並選擇它時使用視覺化按鈕顯示：

```xml
<!-- res/drawable/button.xml -->
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true"
          android:drawable="@drawable/button_pressed" /> <!-- pressed -->
    <item android:state_focused="true"
          android:drawable="@drawable/button_focused" /> <!-- focused -->
    <item android:state_hovered="true"
          android:drawable="@drawable/button_focused" /> <!-- hovered -->
    <item android:drawable="@drawable/button_normal" /> <!-- default -->
</selector>
```

<!-- The following layout XML sample code applies the previous state list drawable to a Button: -->
接下來的XML示例代碼對按鈕控件應用了上面的按鍵狀態列表drawable：

```xml
<Button
    android:layout_height="wrap_content"
    android:layout_width="wrap_content"
    android:background="@drawable/button" />
```

<!-- Make sure to provide sufficient padding within the focusable and selectable controls so that the highlights around them are clearly visible. -->
確保在可定為焦點的和可選中的控件中提供了充分的填充，以便圍繞它們的高亮是清楚的。

<!-- For more recommendations on designing effective selection and focus for your TV app, see Patterns for TV. -->
更多建議關於TV應用中設計有效的選中和焦點，看[Patterns of TV](http://developer.android.com/design/tv/patterns.html)。

-------------
[下一節: 創建TV播放應用 >](../playback/index.html)
