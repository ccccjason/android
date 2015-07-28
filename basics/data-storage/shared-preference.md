# 保存到Preference

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/basics/data-storage/shared-preferences.html>

當有一個相對較小的key-value集合需要保存時，可以使用[SharedPreferences](http://developer.android.com/reference/android/content/SharedPreferences.html) APIs。 SharedPreferences 對象指向一個保存key-value pairs的文件，併為讀寫他們提供了簡單的方法。每個 SharedPreferences 文件均由framework管理，其既可以是私有的，也可以是共享的。
這節課會演示如何使用 SharedPreferences APIs 來存儲與檢索簡單的數據。

> **Note：** SharedPreferences APIs 僅僅提供了讀寫key-value對的功能，請不要與[Preference](http://developer.android.com/reference/android/preference/Preference.html) APIs相混淆。後者可以幫助我們建立一個設置用戶配置的頁面（儘管它實際上是使用SharedPreferences 來實現保存用戶配置的)。更多關於Preference APIs的信息，請參考[Settings](http://developer.android.com/guide/topics/ui/settings.html) 指南。

## 獲取SharedPreference

我們可以通過以下兩種方法之一創建或者訪問shared preference 文件:

* <a href="http://developer.android.com/reference/android/content/Context.html#getSharedPreferences(java.lang.String, int)">getSharedPreferences()</a> — 如果需要多個通過名稱參數來區分的shared preference文件, 名稱可以通過第一個參數來指定。可在app中通過任何一個[Context](http://developer.android.com/reference/android/content/Context.html) 執行該方法。
* <a href="http://developer.android.com/reference/android/app/Activity.html#getPreferences(int)">getPreferences()</a> — 當activity僅需要一個shared preference文件時。因為該方法會檢索activity下默認的shared preference文件，並不需要提供文件名稱。

例：下面的示例在一個 [Fragment](http://developer.android.com/reference/android/app/Fragment.html) 中被執行，它以private模式訪問名為 `R.string.preference_file_key` 的shared preference文件。這種情況下，該文件僅能被我們的app訪問。

```java
Context context = getActivity();
SharedPreferences sharedPref = context.getSharedPreferences(
        getString(R.string.preference_file_key), Context.MODE_PRIVATE);
```

應以與app相關的方式為shared preference文件命名，該名稱應唯一。如本例中可將其命名為 `"com.example.myapp.PREFERENCE_FILE_KEY"` 。

當然，當activity僅需要一個shared preference文件時，我們可以使用<a href="http://developer.android.com/reference/android/app/Activity.html#getPreferences(int)">getPreferences()</a>方法：

```java
SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
```

> **Caution:** 如果創建了一個[MODE_WORLD_READABLE](http://developer.android.com/reference/android/content/Context.html#MODE_WORLD_READABLE)或者[MODE_WORLD_WRITEABLE](http://developer.android.com/reference/android/content/Context.html#MODE_WORLD_WRITEABLE) 模式的shared preference文件，則其他任何app均可通過文件名訪問該文件。

## 寫Shared Preference

為了寫`shared preferences`文件，需要通過執行<a href="http://developer.android.com/reference/android/content/SharedPreferences.html#edit()">edit()</a>創建一個 [SharedPreferences.Editor](http://developer.android.com/reference/android/content/SharedPreferences.Editor.html)。

通過類似<a href="http://developer.android.com/reference/android/content/SharedPreferences.Editor.html#putInt(java.lang.String, int)">putInt()</a>與<a href="http://developer.android.com/reference/android/content/SharedPreferences.Editor.html#putString(java.lang.String, java.lang.String)">putString()</a>等方法傳遞keys與values，接著通過<a href="http://developer.android.com/reference/android/content/SharedPreferences.Editor.html#commit()">commit()</a> 提交改變. 

```java
SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
SharedPreferences.Editor editor = sharedPref.edit();
editor.putInt(getString(R.string.saved_high_score), newHighScore);
editor.commit();
```

## 讀Shared Preference

為了從shared preference中讀取數據，可以通過類似於 getInt() 及 getString()等方法來讀取。在那些方法裡面傳遞我們想要獲取的value對應的key，並提供一個默認的value作為查找的key不存在時函數的返回值。如下：

```java
SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
int defaultValue = getResources().getInteger(R.string.saved_high_score_default);
long highScore = sharedPref.getInt(getString(R.string.saved_high_score), default);
```
