# 接收Activity返回的結果

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/basics/intents/result.html>

啟動另外一個activity並不一定是單向的。我們也可以啟動另外一個activity然後接受一個返回的result。為接受result，我們需要使用<a href="http://developer.android.com/reference/android/app/Activity.html#startActivityForResult(android.content.Intent, int)">startActivityForResult()</a> ，而不是<a href="http://developer.android.com/reference/android/app/Activity.html#startActivity(android.content.Intent)">startActivity()</a>。

例如，我們的app可以啟動一個camera程序並接受拍的照片作為result。或者可以啟動聯繫人程序並獲取其中聯繫的人的詳情作為result。

當然，被啟動的activity需要指定返回的result。它需要把這個result作為另外一個intent對象返回，我們的activity需要在<a href="http://developer.android.com/reference/android/app/Activity.html#onActivityResult(int, int, android.content.Intent)">onActivityResult()</a>的回調方法裡面去接收result。

> **Note:**在執行`startActivityForResult()`時，可以使用explicit 或者 implicit 的intent。當啟動另外一個位於的程序中的activity時，我們應該使用explicit intent來確保可以接收到期待的結果。

<!-- more -->

## 啟動Activity

對於startActivityForResult() 方法中的intent與之前介紹的並無太大差異，不過是需要在這個方法裡面多添加一個int類型的參數。

該integer參數稱為"request code"，用於標識請求。當我們接收到result Intent時，可從回調方法裡面的參數去判斷這個result是否是我們想要的。

例如，下面是一個啟動activity來選擇聯繫人的例子：

```java
static final int PICK_CONTACT_REQUEST = 1;  // The request code
...
private void pickContact() {
    Intent pickContactIntent = new Intent(Intent.ACTION_PICK, Uri.parse("content://contacts"));
    pickContactIntent.setType(Phone.CONTENT_TYPE); // Show user only contacts w/ phone numbers
    startActivityForResult(pickContactIntent, PICK_CONTACT_REQUEST);
}
```

## 接收Result

當用戶完成了啟動之後activity操作之後，系統會調用我們activity中的onActivityResult() 回調方法。該方法有三個參數：

* 通過startActivityForResult()傳遞的request code。
* 第二個activity指定的result code。如果操作成功則是`RESULT_OK` ，如果用戶沒有操作成功，而是直接點擊回退或者其他什麼原因，那麼則是`RESULT_CANCELED`
* 包含了所返回result數據的intent。

例如，下面顯示瞭如何處理pick a contact的result：

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    // Check which request we're responding to
    if (requestCode == PICK_CONTACT_REQUEST) {
        // Make sure the request was successful
        if (resultCode == RESULT_OK) {
            // The user picked a contact.
            // The Intent's data Uri identifies which contact was selected.

            // Do something with the contact here (bigger example below)
        }
    }
}
```

本例中被返回的Intent使用Uri的形式來表示返回的聯繫人。

為正確處理這些result，我們必須瞭解那些result intent的格式。對於自己程序裡面的返回result是比較簡單的。Apps都會有一些自己的api來指定特定的數據。例如，People app (Contacts app on some older versions) 總是返回一個URI來指定選擇的contact，Camera app 則是在`data`數據區返回一個 Bitmap （see the class about [Capturing Photos](http://developer.android.com/training/camera/index.html)).

### 讀取聯繫人數據

上面的代碼展示瞭如何獲取聯繫人的返回結果，但沒有說清楚如何從結果中讀取數據，因為這需要更多關於[content providers](http://developer.android.com/guide/topics/providers/content-providers.html)的知識。但如果想知道的話，下面是一段代碼，展示如何從被選的聯繫人中讀出電話號碼。

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    // Check which request it is that we're responding to
    if (requestCode == PICK_CONTACT_REQUEST) {
        // Make sure the request was successful
        if (resultCode == RESULT_OK) {
            // Get the URI that points to the selected contact
            Uri contactUri = data.getData();
            // We only need the NUMBER column, because there will be only one row in the result
            String[] projection = {Phone.NUMBER};

            // Perform the query on the contact to get the NUMBER column
            // We don't need a selection or sort order (there's only one result for the given URI)
            // CAUTION: The query() method should be called from a separate thread to avoid blocking
            // your app's UI thread. (For simplicity of the sample, this code doesn't do that.)
            // Consider using CursorLoader to perform the query.
            Cursor cursor = getContentResolver()
                    .query(contactUri, projection, null, null, null);
            cursor.moveToFirst();

            // Retrieve the phone number from the NUMBER column
            int column = cursor.getColumnIndex(Phone.NUMBER);
            String number = cursor.getString(column);

            // Do something with the phone number...
        }
    }
}
```

> **Note**:在Android 2.3 (API level 9)之前對`Contacts Provider`的請求(比如上面的代碼)，需要聲明`READ_CONTACTS`權限(更多詳見[Security and Permissions](http://developer.android.com/guide/topics/security/security.html))。但如果是Android 2.3以上的系統就不需要這麼做。但這種臨時權限也僅限於特定的請求，所以仍無法獲取除返回的Intent以外的聯繫人信息，除非聲明瞭`READ_CONTACTS`權限。
