# 創建一個Fragment

> 編寫:[fastcome1985](https://github.com/fastcome1985) - 原文:<http://developer.android.com/training/basics/fragments/creating.html>

* 我們可以把fragment想象成activity中一個模塊化的部分，它擁有自己的生命週期，接收自己的輸入事件，可以在acvitity運行過程中添加或者移除（有點像"子activity"，可以在不同的activity裡面重複使用）。這一課教我們將學習繼承[Support Library ](http://developer.android.com/tools/support-library/index.html)中的[Fragment](http://developer.android.com/reference/android/support/v4/app/Fragment.html)，使應用在Android1.6這樣的低版本上仍能保持兼容。

> **Note：** 如果APP的最低API版本是11或以上，則不必使用Support Library，我們可以直接使用API框架中的[Fragment](http://developer.android.com/reference/android/app/Fragment.html)，本課主要講解基於Support Library的API，Support Library有一個特殊的包名，有時與平臺版本的API名字存在略微不同。

* 在開始這節課前，必須先讓在項目中引用Support Library。如果沒有使用過Support Library，可以根據文檔  [Support Library Setup](http://developer.android.com/intl/zh-cn/tools/support-library/setup.html) 來設置項目使用Support Library。當然，也可以使用包含[action bar](http://developer.android.com/guide/topics/ui/actionbar.html)的 **v7 appcompat** library。v7 appcompat library 兼容Android2.1(API level 7)，也包含了[Fragment](http://developer.android.com/reference/android/support/v4/app/Fragment.html) APIs。

## 創建一個Fragment類

* 創建一個fragment，首先需要繼承[Fragment](http://developer.android.com/reference/android/support/v4/app/Fragment.html)類，然後在關鍵的生命週期方法中插入APP的邏輯，就像[activity](http://developer.android.com/reference/android/app/Activity.html)一樣。

* 其中一個區別是當創建[Fragment](http://developer.android.com/reference/android/support/v4/app/Fragment.html)的時，必須重寫<a href="http://developer.android.com/reference/android/support/v4/app/Fragment.html#onCreateView(android.view.LayoutInflater, android.view.ViewGroup, android.os.Bundle)">onCreateView()</a>回調方法來定義佈局。事實上，這是使Fragment運行起來，唯一一個需要我們重寫的回調方法。比如，下面是一個自定義佈局的示例fragment.

```java
import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.ViewGroup;

public class ArticleFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
        Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.article_view, container, false);
    }
}
```

* 就像activity一樣，當fragment從activity添加或者移除、當activity生命週期發生變化時，fragment通過生命週期回調函數管理其狀態。例如，當activity的onPause()被調用時，它裡面的所有fragment的onPause()方法也會被觸發。

更多關於fragment的聲明週期和回調方法，詳見[Fragments](http://developer.android.com/guide/components/fragments.html) developer guide.

## 用XML將fragment添加到activity


* fragments是可重用的，模塊化的UI組件，每個Fragment的實例都必須與一個[FragmentActivity](http://developer.android.com/reference/android/support/v4/app/FragmentActivity.html)關聯。我們可以在activity的XML佈局文件中定義每一個fragment來實現這種關聯。

> **Notes：**[FragmentActivity](http://developer.android.com/reference/android/support/v4/app/FragmentActivity.html)是Support Library提供的一個特殊activity ，用於處理API11版本以下的fragment。如果我們APP中的最低版本大於等於11，則可以使用普通的[Activity](http://developer.android.com/reference/android/app/Activity.html)。

* 下面是一個XML佈局的例子，當屏幕被認為是large(用目錄名稱中的`large`字符來區分)時，它在佈局中增加了兩個fragment.

res/layout-large/news_articles.xml

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent">

    <fragment android:name="com.example.android.fragments.HeadlinesFragment"
              android:id="@+id/headlines_fragment"
              android:layout_weight="1"
              android:layout_width="0dp"
              android:layout_height="match_parent" />

    <fragment android:name="com.example.android.fragments.ArticleFragment"
              android:id="@+id/article_fragment"
              android:layout_weight="2"
              android:layout_width="0dp"
              android:layout_height="match_parent" />

</LinearLayout>
```

> **Notes：**更多關於不同屏幕尺寸創建不同佈局的信息，請閱讀[Supporting Different Screen Sizes](../../ui/multiscreen/screen-sizes.html)

* 然後將這個佈局文件用到activity中。

```java
import android.os.Bundle;
import android.support.v4.app.FragmentActivity;

public class MainActivity extends FragmentActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.news_articles);
    }
}
```

* 如果用的是[ v7 appcompat library](http://developer.android.com/intl/zh-cn/tools/support-library/features.html#v7-appcompat)，activity應該改為繼承[ActionBarActivity](http://developer.android.com/reference/android/support/v7/app/ActionBarActivity.html)，ActionBarActivity是FragmentActivity的一個子類（更多關於這方面的內容，請閱讀[Adding the Action Bar](http://developer.android.com/training/basics/actionbar/index.html)）。

> **Note：**當通過XML佈局文件的方式將Fragment添加進activity時，Fragment是不能被動態移除的。如果想要在用戶交互的時候把fragment切入與切出，必須在activity啟動後，再將fragment添加進activity。這部分內容將在下節課闡述。
