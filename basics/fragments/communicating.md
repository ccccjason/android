# Fragments之間的交互

> 編寫:[fastcome1985](https://github.com/fastcome1985) - 原文:<http://developer.android.com/training/basics/fragments/communicating.html>

* 為了重用Fragment UI組件，我們應該把每一個fragment都構建成完全的自包含的、模塊化的組件，定義他們自己的佈局與行為。定義好這些模塊化的Fragment後，就可以讓他們關聯activity，使他們與application的邏輯結合起來，實現全局的複合的UI。

* 通常fragment之間可能會需要交互，比如基於用戶事件改變fragment的內容。所有fragment之間的交互需要通過他們關聯的activity，兩個fragment之間不應該直接交互。

## 定義一個接口

* 為了讓fragment與activity交互，可以在Fragment 類中定義一個接口，並在activity中實現。Fragment在他們生命週期的onAttach()方法中獲取接口的實現，然後調用接口的方法來與Activity交互。

下面是一個fragment與activity交互的例子：

```java
public class HeadlinesFragment extends ListFragment {
    OnHeadlineSelectedListener mCallback;

    // Container Activity must implement this interface
    public interface OnHeadlineSelectedListener {
        public void onArticleSelected(int position);
    }

    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);

        // This makes sure that the container activity has implemented
        // the callback interface. If not, it throws an exception
        try {
            mCallback = (OnHeadlineSelectedListener) activity;
        } catch (ClassCastException e) {
            throw new ClassCastException(activity.toString()
                    + " must implement OnHeadlineSelectedListener");
        }
    }

    ...
}
```

* 現在Fragment就可以通過調用`OnHeadlineSelectedListener`接口實例的`mCallback`中的`onArticleSelected()`（也可以是其它方法）方法與activity傳遞消息。

* 舉個例子，在fragment中的下面的方法在用戶點擊列表條目時被調用，fragment 用回調接口來傳遞事件給父Activity.

```java
 @Override
    public void onListItemClick(ListView l, View v, int position, long id) {
        // Send the event to the host activity
        mCallback.onArticleSelected(position);
    }
```

## 實現接口

* 為了接收回調事件，宿主activity必須實現在Fragment中定義的接口。

* 舉個例子，下面的activity實現了上面例子中的接口。

```java
public static class MainActivity extends Activity
        implements HeadlinesFragment.OnHeadlineSelectedListener{
    ...

    public void onArticleSelected(int position) {
        // The user selected the headline of an article from the HeadlinesFragment
        // Do something here to display that article
    }
}
```


## 傳消息給Fragment

* 宿主activity通過<a href="http://developer.android.com/reference/android/support/v4/app/FragmentManager.html#findFragmentById(int)">findFragmentById()</a>方法獲取[fragment](http://developer.android.com/reference/android/support/v4/app/Fragment.html)的實例，然後直接調用Fragment的public方法來向fragment傳遞消息。

* 例如，假設上面所示的activity可能包含另外一個fragment,這個fragment用來展示從上面的回調方法中返回的指定的數據。在這種情況下，activity可以把從回調方法中接收到的信息傳遞給這個展示數據的Fragment.

```java
public static class MainActivity extends Activity
        implements HeadlinesFragment.OnHeadlineSelectedListener{
    ...

    public void onArticleSelected(int position) {
        // The user selected the headline of an article from the HeadlinesFragment
        // Do something here to display that article

        ArticleFragment articleFrag = (ArticleFragment)
                getSupportFragmentManager().findFragmentById(R.id.article_fragment);

        if (articleFrag != null) {
            // If article frag is available, we're in two-pane layout...

            // Call a method in the ArticleFragment to update its content
            articleFrag.updateArticleView(position);
        } else {
            // Otherwise, we're in the one-pane layout and must swap frags...

            // Create fragment and give it an argument for the selected article
            ArticleFragment newFragment = new ArticleFragment();
            Bundle args = new Bundle();
            args.putInt(ArticleFragment.ARG_POSITION, position);
            newFragment.setArguments(args);

            FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();

            // Replace whatever is in the fragment_container view with this fragment,
            // and add the transaction to the back stack so the user can navigate back
            transaction.replace(R.id.fragment_container, newFragment);
            transaction.addToBackStack(null);

            // Commit the transaction
            transaction.commit();
        }
    }
}
```
