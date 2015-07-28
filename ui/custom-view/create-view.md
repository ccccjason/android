# 創建自定義的View類

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/custom-views/create-view.html>

設計良好的類總是相似的。它使用一個好用的接口來封裝一個特定的功能，它有效的使用CPU與內存，等等。為了成為一個設計良好的類，自定義的view應該:

* 遵守Android標準規則。
* 提供自定義的風格屬性值並能夠被Android XML Layout所識別。
* 發出可訪問的事件。
* 能夠兼容Android的不同平臺。

Android的framework提供了許多基類與XML標籤用來幫助你創建一個符合上面要求的View。這節課會介紹如何使用Android framework來創建一個view的核心功能。


## 繼承一個View
Android framework裡面定義的view類都繼承自View。你自定義的view也可以直接繼承View，或者你可以通過繼承既有的一個子類(例如Button)來節約一點時間。

為了讓Android Developer Tools能夠識別你的view，你必須至少提供一個constructor，它包含一個Contenx與一個AttributeSet對象作為參數。這個constructor允許layout editor創建並編輯你的view的實例。

```java
class PieChart extends View {
    public PieChart(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
}
```

## 定義自定義屬性
為了添加一個內置的View到你的UI上，你需要通過XML屬性來指定它的樣式與行為。良好的自定義views可以通過XML添加和改變樣式，為了讓你的自定義的view也有如此的行為，你應該:

* 為你的view在<declare-styleable>資源標籤下定義自設的屬性
* 在你的XML layout中指定屬性值
* 在運行時獲取屬性值
* 把獲取到的屬性值應用在你的view上

這一節討論如何定義自定義屬性以及指定屬性值，下一節將會實現在運行時獲取屬性值並將它應用。

為了定義自設的屬性，添加 <declare-styleable> 資源到你的項目中。放置於res/values/attrs.xml文件中。下面是一個attrs.xml文件的示例:

```xml
<resources>
   <declare-styleable name="PieChart">
       <attr name="showText" format="boolean" />
       <attr name="labelPosition" format="enum">
           <enum name="left" value="0"/>
           <enum name="right" value="1"/>
       </attr>
   </declare-styleable>
</resources>
```

上面的代碼聲明瞭2個自設的屬性，**showText**與**labelPosition**，它們都歸屬於PieChart的項目下的styleable實例。styleable實例的名字，通常與自定義的view名字一致。儘管這並沒有嚴格規定要遵守這個convention，但是許多流行的代碼編輯器都依靠這個命名規則來提供statement completion。

一旦你定義了自設的屬性，你可以在layout XML文件中使用它們，就像內置屬性一樣。唯一不同的是你自設的屬性是歸屬於不同的命名空間。不是屬於`http://schemas.android.com/apk/res/android`的命名空間，它們歸屬於`http://schemas.android.com/apk/res/[your package name]`。例如，下面演示瞭如何為PieChart使用上面定義的屬性：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
   xmlns:custom="http://schemas.android.com/apk/res/com.example.customviews">
 <com.example.customviews.charting.PieChart
     custom:showText="true"
     custom:labelPosition="left" />
</LinearLayout>
```

為了避免輸入長串的namespace名字，示例上面使用了`xmlns`指令，這個指令可以指派`custom`作為`http://schemas.android.com/apk/res/com.example.customviews`namespace的別名。你也可以選擇其他的別名作為你的namespace。

請注意，如果你的view是一個inner class，你必須指定這個view的outer class。同樣的，如果PieChart有一個inner class叫做PieView。為了使用這個類中自設的屬性，你應該使用com.example.customviews.charting.PieChart$PieView.

## 應用自定義屬性
當view從XML layout被創建的時候，在xml標籤下的屬性值都是從resource下讀取出來並傳遞到view的constructor作為一個AttributeSet參數。儘管可以從AttributeSet中直接讀取數值，可是這樣做有些弊端：

* 擁有屬性的資源並沒有經過解析
* Styles並沒有運用上

> 翻譯註：通過 attrs 的方法是可以直接獲取到屬性值的，但是不能確定值類型，如:
```java
String title = attrs.getAttributeValue(null, "title");
int resId = attrs.getAttributeResourceValue(null, "title", 0);
title = context.getText(resId));
```
>都能獲取到 "title" 屬性，但你不知道值是字符串還是resId，處理起來就容易出問題，下面的方法則能在編譯時就發現問題

取而代之的是，通過obtainStyledAttributes()來獲取屬性值。這個方法會傳遞一個[TypedArray](http://developer.android.com/reference/android/content/res/TypedArray.html)對象，它是間接referenced並且styled的。

Android資源編譯器幫你做了許多工作來使調用[obtainStyledAttributes()](http://developer.android.com/reference/android/content/res/Resources.Theme.html#obtainStyledAttributes(android.util.AttributeSet, int[], int, int))更簡單。對res目錄裡的每一個`<declare-styleable>`資源，自動生成的R.java文件定義了存放屬性ID的數組和常量，常量用來索引數組中每個屬性。你可以使用這些預先定義的常量來從[TypedArray](http://developer.android.com/reference/android/content/res/TypedArray.html)中讀取屬性。這裡就是`PieChart`類如何讀取它的屬性:

```java
public PieChart(Context context, AttributeSet attrs) {
   super(context, attrs);
   TypedArray a = context.getTheme().obtainStyledAttributes(
        attrs,
        R.styleable.PieChart,
        0, 0);

   try {
       mShowText = a.getBoolean(R.styleable.PieChart_showText, false);
       mTextPos = a.getInteger(R.styleable.PieChart_labelPosition, 0);
   } finally {
       a.recycle();
   }
}
```

清注意TypedArray對象是一個共享資源，必須被在使用後進行回收。

## 添加屬性和事件
Attributes是一個強大的控制view的行為與外觀的方法，但是他們僅僅能夠在view被初始化的時候被讀取到。為了提供一個動態的行為，需要暴露出一些合適的getter 與setter方法。下面的代碼演示瞭如何使用這個技巧:

```java
public boolean isShowText() {
   return mShowText;
}

public void setShowText(boolean showText) {
   mShowText = showText;
   invalidate();
   requestLayout();
}
```

請注意，在setShowText方法裡面有調用[invalidate()](http://developer.android.com/reference/android/view/View.html#invalidate()) and [requestLayout()](http://developer.android.com/reference/android/view/View.html#requestLayout()). 這兩個調用是確保穩定運行的關鍵。當view的某些內容發生變化的時候，需要調用invalidate來通知系統對這個view進行redraw，當某些元素變化會引起組件大小變化時，需要調用requestLayout方法。調用時若忘了這兩個方法，將會導致hard-to-find bugs。

自定義的view也需要能夠支持響應事件的監聽器。例如，`PieChart`暴露了一個自定義的事件`OnCurrentItemChanged`來通知監聽器，用戶已經切換了焦點到一個新的組件上。

我們很容易忘記了暴露屬性與事件，特別是當你是這個view的唯一用戶時。請花費一些時間來仔細定義你的view的交互。一個好的規則是總是暴露任何屬性與事件。

## 設計可訪問性

自定義view應該支持廣泛的用戶群體，包含一些不能看到或使用觸屏的殘障人士。為了支持殘障人士，我們應該：

* 使用`android:contentDescription`屬性標記輸入字段。
* 在適當的時候通過調用`sendAccessibilityEvent()` 發送訪問事件。
* 支持備用控制器，如方向鍵（D-pad）和軌跡球（trackball）等。


對於創建使用的 views的更多消息, 請參見Android Developers Guide中的 [Making Applications Accessible](http://developer.android.com/guide/topics/ui/accessibility/apps.html#custom-views) 。
