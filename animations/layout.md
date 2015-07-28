# 佈局變更動畫

> 編寫:[XizhiXu](https://github.com/XizhiXu) - 原文:<http://developer.android.com/training/animation/layout.html>

佈局動畫是一種預加載動畫，系統在每次改變佈局配置時運行它。我們需要做的僅是在佈局文件裡設置屬性告訴Android系統為這些佈局的變更應用動畫，然後系統的默認動畫便會執行。

> **小貼士:** 如果你想補充自定義佈局動畫，創建 [`LayoutTransition`](http://developer.android.com/reference/android/animation/LayoutTransition.html) 對象，然後用 <a href="http://developer.android.com/reference/android/view/ViewGroup.html#setLayoutTransition(android.animation.LayoutTransition)"> `setLayoutTransition()` </a> 方法把它加到佈局中。

下面的例子在一個list中添加一項的默認佈局動畫：

<div style="
  background: transparent url(device_galaxynexus_blank_land_span8.png) no-repeat
scroll top left; padding: 26px 68px 38px 72px; overflow: hidden;">

<video style="width: 320px; height: 180px;" controls="" autoplay="">
    <source src="anim_layout_changes.mp4" type="video/mp4">
    <source src="anim_layout_changes.mp4" type="video/mp4">
    <source src="anim_layout_changes.ogv" type="video/ogg">
</video>

</div>

如果你想直接查看整個例子，[下載](http://developer.android.com/shareables/training/Animations.zip) App 樣例並運行然後選擇佈局漸變的例子。查看下列文件中的代碼實現：

* `src/LayoutChangesActivity.java`
* `layout/activity_layout_changes.xml`
* `menu/activity_layout_changes.xml`

## 創建佈局

在Activity的XML佈局文件中，為想開啟動畫的佈局設置`android:animateLayoutChanges`屬性為`true`。例如：

```xml
<LinearLayout android:id="@+id/container"
    android:animateLayoutChanges="true"
    ...
/>
```

## 從佈局中添加，更新或刪除項目

現在，我們需要做的就是添加，刪除或更新佈局裡的項目，然後這些項目就會自動顯示動畫：

```java
private ViewGroup mContainerView;
...
private void addItem() {
    View newView;
    ...
    mContainerView.addView(newView, 0);
}
```
