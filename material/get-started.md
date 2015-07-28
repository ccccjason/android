# 開始使用Material Design

> 編寫: [allenlsy](https://github.com/allenlsy) - 原文: <https://developer.android.com/training/material/get-started.html>

要創建一個 Material Design 應用：

1. 學習 [Material Design 規格標準](http://www.google.com/design/spec/material-design/introduction.html)
2. 應用 Material Design 主題
3. 創建符合 Material Design 的 Layout 文件
4. 定義視圖的 elevation 值來修改陰影
5. 使用系統組件來創建列表和卡片
6. 自定義動畫

#### 維護向下兼容性

你可以添加 Material Design 特性，同時保持對 Android 5.0 之前版本的兼容。更多信息，請參見[維護兼容性章節](https://developer.android.com/training/material/compatibility.html)。

#### 使用 Material Design 更新現有應用

要更新現有應用，使其使用 Material Design，你需要翻新你的 layout 文件來遵從 Material Design 標準，並確保其包含了正確的元素高度，觸摸反饋和動畫。

#### 使用 Material Design 創建新的應用

如果你要創建使用 Material Design 的新的應用，Material Design 指南提供了一套跨平臺統一的設計。請遵從指南，使用新功能來進行 Android 應用的設計和開發。

## 應用 Material 主題

要在應用中使用 Material 主題，需要定義一個繼承於 `android:Theme.Material` 的 style 文件：

```xml
<!-- res/values/styles.xml -->
<resources>
  <!-- your theme inherits from the material theme -->
  <style name="AppTheme" parent="android:Theme.Material">
    <!-- theme customizations -->
  </style>
</resources>
```

Material 主題提供了更新後的系統組件，使你可以設置調色板和在觸摸和 Activity 切換時使用默認的動畫。更多信息，請參見 [Material 主題](http://developer.android.com/training/material/theme.html) 章節。

## 設計你的 Layouts

另外，要應用自定義的 Material 主題，你的 layout 應該要符合 [Material 設計規範](http://www.google.com/design/spec)。在設計 Layout 時，尤其要注意一下方面：

* 基準線網格
* Keyline
* 間隙
* 觸摸目標的大小
* Layout 結構

## 定義視圖的 Elevation

視圖可以投射陰影， elevation 值決定了陰影的大小和繪製順序。要設定 elevation 值，請使用 `android:elevation` 屬性：

```xml
<TextView
    android:id="@+id/my_textview"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/next"
    android:background="@color/white"
    android:elevation="5dp" />
```

新的 `translationZ` 屬性使得你可以設計臨時變更 elevation 的動畫。elevation 變化在做觸摸反饋時很有用。

更多信息，請參見定義陰影和 Clipping 視圖章節。

## 創建列表和卡片

[RecyclerView](http://developer.android.com/reference/android/support/v7/widget/RecyclerView.html) 是一個植入性更強的 ListView，它支持不同的 layout 類型，並可以提升性能。[CardView](http://developer.android.com/reference/android/support/v7/widget/CardView.html) 使得你可以在卡片內顯示一部分內容，並且和其他應用保持外觀一致。以下是一段樣例代碼展示如何在 layout 中添加 CardView

```xml
<android.support.v7.widget.CardView
    android:id="@+id/card_view"
    android:layout_width="200dp"
    android:layout_height="200dp"
    card_view:cardCornerRadius="3dp">
    ...
</android.support.v7.widget.CardView>
```

更多信息，請參見列表和卡片章節。

## 自定義動畫

Android 5.0 (API level 21) 包含了新的創建自定義動畫 API。比如，你可以在 activity 中定義進入和退出 activity 時的動畫。


```java
public class MyActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // enable transitions
        getWindow().requestFeature(Window.FEATURE_CONTENT_TRANSITIONS);
        setContentView(R.layout.activity_my);
    }

    public void onSomeButtonClicked(View view) {
        getWindow().setExitTransition(new Explode());
        Intent intent = new Intent(this, MyOtherActivity.class);
        startActivity(intent,
                      ActivityOptions
                          .makeSceneTransitionAnimation(this).toBundle());
    }
}
```

當你從當前 activity 進入另一個 activity 時，退出切換動畫會被調用。

想學習更多新的動畫 API，參見[自定義動畫章節](http://developer.android.com/reference/android/view/View.html#setSystemUiVisibility(int))。
