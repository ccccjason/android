# 按需加載視圖

> 編寫:[allenlsy](https://github.com/allenlsy) - 原文:<http://developer.android.com/training/improving-layouts/loading-ondemand.html>

有時你的 Layout 會用到不怎麼重用的複雜視圖。不管它是列表項 細節，進度顯示器，或是撤銷時的提示信息，你可以僅在需要的時候載入它們，提高 UI 渲染速度。

## 定義 ViewStub

[ViewStub](http://developer.android.com/reference/android/view/ViewStub.html) 是一個輕量的視圖，不需要大小信息，也不會在被加入的 Layout 中繪製任何東西。每個 ViewStub 只需要設置 `android:layout` 屬性來指定需要被 inflate 的 Layout 類型。

以下 ViewStub 是一個半透明的進度條覆蓋層。功能上講，它應該只在新的數據項被導入到應用程序時可見。

```xml
<ViewStub
    android:id="@+id/stub_import"
    android:inflatedId="@+id/panel_import"
    android:layout="@layout/progress_overlay"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:layout_gravity="bottom" />
```

## 載入 ViewStub Layout

當你要載入用 ViewStub 聲明的 Layout 時，要麼用 `setVisibility(View.VISIBLE)` 設置它的可見性，要麼調用其 `inflate()` 方法。

```java
((ViewStub) findViewById(R.id.stub_import)).setVisibility(View.VISIBLE);
// or
View importPanel = ((ViewStub) findViewById(R.id.stub_import)).inflate();
```

> **Notes**：`inflate()` 方法會在渲染完成後返回被 inflate 的視圖，所以如果你需要和這個 Layout 交互的話， 你不需要再調用 `findViewById()` 去查找這個元素，。

一旦 ViewStub 可見或是被 inflate 了，ViewStub 就不再繼續存在View的層級機構中了。取而代之的是被 inflate 的 Layout，其 id 是 ViewStub 上的 `android:inflatedId` 屬性。（ViewStub 的 `android:id` 屬性僅在 ViewStub 可見以前可用）

> **Notes**：ViewStub 的一個缺陷是，它目前不支持使用 `<merge/>` 標籤的 Layout 。
