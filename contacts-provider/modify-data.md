# 使用Intent修改聯繫人信息

> 編寫:[spencer198711](https://github.com/spencer198711) - 原文:<http://developer.android.com/training/contacts-provider/modify-data.html>

這一課介紹如何使用[Intent](http://developer.android.com/reference/android/content/Intent.html)去插入一個新的聯繫人或者修改聯繫人的數據。我們不是直接訪問Contacts Provider，而是通過Intent啟動Contacts應用去運行適當的[Activity](http://developer.android.com/reference/android/app/Activity.html)。對於這一課中描述的數據修改行為，如果你向Intent發送擴展的數據，它會自動填充進啟動的Activity頁面中。

使用Intent去插入或者更新一個聯繫人是比較推薦的修改Contacts Provider的做法。原因如下：

* 節省了我們自行開發UI和編寫代碼的時間和精力。
* 避免了由於不按照Contacts Provider的規則去修改而產生的錯誤。
* 減少應用需要申請的權限數量。因為我們的應用把修改行為委託給已經擁有寫Contacts Provider權限的Contacts應用，所以我們的應用不需要再去申請這個權限，。

## 使用Intent插入新的聯繫人

當我們的應用接收到新的數據時，我們通常會允許用戶去插入一個新的聯繫人。例如，一個餐館評論應用可以允許用戶在評論餐館的時候，把這個餐館添加為一個聯繫人。可以使用Intent去做這個任務，使用我們擁有的儘可能多的數據去創建對應的Intent，然後發送這個Intent到Contacts應用。

使用Contacts應用去插入一個聯繫人將會向Contacts Provider中的[ContactsContract.RawContacts](http://developer.android.com/reference/android/provider/ContactsContract.RawContacts.html)表中插入一個原始聯繫人。必要的情況下，在創建原始聯繫人的時候，Contacts應用將會提示用戶選擇賬戶類型和要使用的賬戶。如果聯繫人已經存在，Contacts應用也會告知用戶。用戶將會有取消插入的選項，在這種情況下不會有聯繫人被創建。想要知道更多關於原始聯繫人的信息，請參閱[Contacts Provider](http://developer.android.com/guide/topics/providers/contacts-provider.html)的API指導。

### 創建一個Intent

利用Intents.Insert.ACTION創建一個新的Intent對象，並設置其MIME類型為[RawContacts.CONTENT_TYPE](http://developer.android.com/reference/android/provider/ContactsContract.RawContacts.html#CONTENT_TYPE)。例如：

```java
...
// Creates a new Intent to insert a contact
Intent intent = new Intent(Intents.Insert.ACTION);
// Sets the MIME type to match the Contacts Provider
intent.setType(ContactsContract.RawContacts.CONTENT_TYPE);
```

如果我們已經獲得了此聯繫人的詳細信息，比如說電話號碼或者email地址，那麼我們可以把它們作為擴展數據添加到Intent中。對於鍵值，需要使用[Intents.Insert](http://developer.android.com/reference/android/provider/ContactsContract.Intents.Insert.html)中對應的常量。Contacts應用將會在插入界面顯示這些數據，以便用戶作進一步的數據編輯和數據添加。

```java
/* Assumes EditText fields in your UI contain an email address
 * and a phone number.
 *
 */
private EditText mEmailAddress = (EditText) findViewById(R.id.email);
private EditText mPhoneNumber = (EditText) findViewById(R.id.phone);
...
/*
 * Inserts new data into the Intent. This data is passed to the
 * contacts app's Insert screen
 */
// Inserts an email address
intent.putExtra(Intents.Insert.EMAIL, mEmailAddress.getText())
/*
 * In this example, sets the email type to be a work email.
 * You can set other email types as necessary.
 */
      .putExtra(Intents.Insert.EMAIL_TYPE, CommonDataKinds.Email.TYPE_WORK)
// Inserts a phone number
      .putExtra(Intents.Insert.PHONE, mPhoneNumber.getText())
/*
 * In this example, sets the phone type to be a work phone.
 * You can set other phone types as necessary.
 */
      .putExtra(Intents.Insert.PHONE_TYPE, Phone.TYPE_WORK);
```

一旦我們創建好Intent，調用startActivity()將其發送到Contacts應用。

```java
	/* Sends the Intent
     */
    startActivity(intent);
```

這個調用將會打開Contacts應用的界面，並允許用戶進入一個新的聯繫人。這個聯繫人的賬戶類型和賬戶名字列在屏幕的上方。一旦用戶輸入數據並點擊*確定*，Contacts應用的聯繫人列表則會顯示出來。用戶可以點擊*Back*鍵返回到我們自己創建的應用。

## 使用Intent編輯已經存在的聯繫人

如果用戶已經選擇了一個感興趣的聯繫人，使用Intent去編輯這個已存在的聯繫人會很有用。例如，一個用來查找擁有郵政地址但是缺少郵政編碼的聯繫人的應用，可以給用戶提供查找郵政編碼的選項，然後把找到的郵政編碼添加到這個聯繫人中。

使用Intent編輯已經存在的聯繫人，同插入一個聯繫人的步驟類似。像前面介紹的[使用Intent插入新的聯繫人]()創建一個Intent，但是需要給這個Intent添加對應聯繫人的<a href="http://developer.android.com/reference/android/provider/ContactsContract.Contacts.html#CONTENT_LOOKUP_URI">Contacts.CONTENT\_LOOKUP\_URI</a>和MIME類型<a href="http://developer.android.com/reference/android/provider/ContactsContract.Contacts.html#CONTENT_ITEM_TYPE">Contacts.CONTENT\_ITEM\_TYPE</a>。如果想要使用已經擁有的詳情信息編輯這個聯繫人，我們需要把這些數據放到Intent的擴展數據中。同時注意有些列是不能使用Intent編輯的，這些不可編輯的列在[ContactsContract.Contacts](http://developer.android.com/reference/android/provider/ContactsContract.Contacts.html) 摘要部分“Update”標題下有列出。

最後，發送這個Intent。Contacts應用會顯示一個編輯界面作為迴應。當用戶編輯完成並保存，Contacts應用會顯示一個聯繫人列表。當用戶點擊*Back*，我們自己的應用會出現。

### 創建Intent

為了能夠編輯一個聯繫人，需要調用Intent(action)去創建一個擁有ACTION\_EDIT行為的Intent。調用setDataAndType()去設置這個Intent要編輯的聯繫人的Contacts.CONTENT\_LOOKUP\_URI和MIME類型Contacts.CONTENT\_ITEM\_TYPE。因為調用setType()會重寫Intent當前的數據，所以我們必須同時設置數據和MIME類型。

為了得到聯繫人的Contacts.CONTENT\_LOOKUP\_URI，需要調用Contacts.getLookupUri(id, lookupkey)方法，該方法的參數分別是聯繫人的Contacts.\_ID和Contacts.LOOKUP\_KEY。

以下的代碼片段展示瞭如何創建這個Intent：

```java
// The Cursor that contains the Contact row
    public Cursor mCursor;
    // The index of the lookup key column in the cursor
    public int mLookupKeyIndex;
    // The index of the contact's _ID value
    public int mIdIndex;
    // The lookup key from the Cursor
    public String mCurrentLookupKey;
    // The _ID value from the Cursor
    public long mCurrentId;
    // A content URI pointing to the contact
    Uri mSelectedContactUri;
    ...
    /*
     * Once the user has selected a contact to edit,
     * this gets the contact's lookup key and _ID values from the
     * cursor and creates the necessary URI.
     */
    // Gets the lookup key column index
    mLookupKeyIndex = mCursor.getColumnIndex(Contacts.LOOKUP_KEY);
    // Gets the lookup key value
    mCurrentLookupKey = mCursor.getString(mLookupKeyIndex);
    // Gets the _ID column index
    mIdIndex = mCursor.getColumnIndex(Contacts._ID);
    mCurrentId = mCursor.getLong(mIdIndex);
    mSelectedContactUri =
            Contacts.getLookupUri(mCurrentId, mCurrentLookupKey);
    ...
    // Creates a new Intent to edit a contact
    Intent editIntent = new Intent(Intent.ACTION_EDIT);
    /*
     * Sets the contact URI to edit, and the data type that the
     * Intent must match
     */
    editIntent.setDataAndType(mSelectedContactUri,Contacts.CONTENT_ITEM_TYPE);
```

### 添加導航標誌

在Android 4.0（API版本14）和更高的版本，Contacts應用中的一個問題會導致錯誤的頁面導航。我們的應用發送一個編輯聯繫人的Intent到Contacts應用，用戶編輯並保存這個聯繫人，當用戶點擊*Back*鍵的時候會看到聯繫人列表頁面。用戶需要點擊最近使用的應用，然後選擇我們的應用，才能返回到我們自己的應用。

要在Android 4.0.3（API版本15）及以後的版本解決此問題，需要添加`finishActivityOnSaveCompleted`擴展數據參數到這個Intent，並將它的值設置為true。Android 4.0之前的版本也能夠接受這個參數，但是不起作用。為了設置擴展數據，請按照以下方式去做：

```java
	// Sets the special extended data for navigation
    editIntent.putExtra("finishActivityOnSaveCompleted", true);
```

### 添加其他的擴展數據

對Intent添加額外的擴展數據，需要調用putExtra()。可以為常見的聯繫人數據字段添加擴展數據，這些常見字段的key值可以從[Intents.Insert](http://developer.android.com/reference/android/provider/ContactsContract.Intents.Insert.html) API參考文檔中查到。記住ContactsContract.Contacts表中有些列是不能編輯的，這些列在[ContactsContract.Contacts](http://developer.android.com/reference/android/provider/ContactsContract.Contacts.html)的摘要部分“Update”標題下有列出。

### 發送Intent

最後，發送我們已經構建好的Intent。例如：

```java
	// Sends the Intent
    startActivity(editIntent);
```

## 使用Intent讓用戶去選擇是插入還是編輯聯繫人

我們可以通過發送帶有`ACTION_INSERT_OR_EDIT`行為的Intent，讓用戶去選擇是插入聯繫人還是編輯已有的聯繫人。例如，一個email客戶端應用會允許用戶添加一個收件地址到新的聯繫人，或者僅僅作為額外的郵件地址添加到已有的聯繫人。需要為這個Intent設置MIME類型Contacts.CONTENT\_ITEM\_TYPE，但是不需要設置數據URI。

當我們發送這個Intent後，Contacts應用會展示一個聯繫人列表。用戶可以選擇是插入一個新的聯繫人還是挑選一個存在的聯繫人去編輯。任何添加到Intent中的擴展數據字段都會填充在界面上。我們可以使用任何在[Intents.Insert](http://developer.android.com/reference/android/provider/ContactsContract.Intents.Insert.html)中指定的的key值。以下的代碼片段展示瞭如何構建和發送這個Intent：

```java
// Creates a new Intent to insert or edit a contact
    Intent intentInsertEdit = new Intent(Intent.ACTION_INSERT_OR_EDIT);
    // Sets the MIME type
    intentInsertEdit.setType(Contacts.CONTENT_ITEM_TYPE);
    // Add code here to insert extended data, if desired
    ...
    // Sends the Intent with an request ID
    startActivity(intentInsertEdit);
```

