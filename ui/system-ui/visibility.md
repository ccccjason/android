# 響應UI可見性的變化

> 編寫:[K0ST](https://github.com/K0ST) - 原文:<http://developer.android.com/training/system-ui/visibility.html>

本節課將教你如果註冊監聽器來監聽系統UI可見性的變化。這個方法在將系統欄與你自己的UI控件進行同步操作時很有用。

## 註冊監聽器

為了獲取系統UI可見性變化的通知，我們需要對View註冊`View.OnSystemUiVisibilityChangeListener`監聽器。通常上來說，這個View是用來控制導航的可見性的。

例如你可以添加如下代碼在onCreate中

```java
View decorView = getWindow().getDecorView();
decorView.setOnSystemUiVisibilityChangeListener
        (new View.OnSystemUiVisibilityChangeListener() {
    @Override
    public void onSystemUiVisibilityChange(int visibility) {
        // Note that system bars will only be "visible" if none of the
        // LOW_PROFILE, HIDE_NAVIGATION, or FULLSCREEN flags are set.
        if ((visibility & View.SYSTEM_UI_FLAG_FULLSCREEN) == 0) {
            // TODO: The system bars are visible. Make any desired
            // adjustments to your UI, such as showing the action bar or
            // other navigational controls.
        } else {
            // TODO: The system bars are NOT visible. Make any desired
            // adjustments to your UI, such as hiding the action bar or
            // other navigational controls.
        }
    }
});
```

保持系統欄和UI同步是一種很好的實踐方式，比如當狀態欄顯示或隱藏的時候進行ActionBar的顯示和隱藏等等。
