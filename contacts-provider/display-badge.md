# 顯示聯繫人頭像

> 編寫:[spencer198711](https://github.com/spencer198711) - 原文:<http://developer.android.com/training/contacts-provider/display-contact-badge.html>

這一課展示瞭如何在我們的應用界面上添加一個[QuickContactBadge]()，以及如何為它綁定數據。
QuickContactBadge是一個在初始情況下顯示聯繫人縮略圖頭像的widget。儘管我們可以使用任何[Bitmap](http://developer.android.com/reference/android/graphics/Bitmap.html)作為縮略圖頭像，但是我們通常會使用從聯繫人照片縮略圖中解碼出來的Bitmap。

這個小的圖片是一個控件，當用戶點擊它時，QuickContactBadge會展開一個包含以下內容的對話框：

* 一個大的聯繫人頭像

	與這個聯繫人關聯的大的頭像，如果此人沒有設置頭像，則顯示預留的圖案。

* 應用程序圖標

	根據聯繫人詳情數據，顯示每一個能夠被手機中的應用所處理的數據的圖標。例如，如果聯繫人的數據包含一個或多個email地址，就會顯示email應用的圖標。當用戶點擊這個圖標的時候，這個聯繫人所有的email地址都會顯示出來。當用戶點擊其中一個email地址時，email應用將會顯示一個界面，讓用戶為選中的地址撰寫郵件。

QuickContactBadge視圖提供了對聯繫人數據的即時訪問，是一種與聯繫人溝通的快捷方式。用戶不用查詢一個聯繫人，查找並複製信息，然後把信息粘貼到合適的應用中。他們可以點擊QuickContactBadge，選擇他們想要的溝通方式，然後直接把信息發送給合適的應用中。

## 添加一個QuickContactBadge視圖

為了添加一個QuickContactBadge視圖，需要在佈局文件中插入一個QuickContactBadge。例如：

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="match_parent">
...
    <QuickContactBadge
               android:id=@+id/quickbadge
               android:layout_height="wrap_content"
               android:layout_width="wrap_content"
               android:scaleType="centerCrop"/>
    ...
</RelativeLayout>
```

## 獲取Contacts Provider的數據

為了能在QuickContactBadge中顯示聯繫人，我們需要這個聯繫人的內容URI和顯示頭像的Bitmap。我們可以從在Contacts Provider中獲取到的數據列中生成這兩個數據。需要指定這些列作為查詢映射去把數據加載到Cursor中。

對於Android 3.0（API版本為11）以及以後的版本，需要在查詢映射中添加以下列：

* [Contacts._ID](http://developer.android.com/reference/android/provider/BaseColumns.html#_ID)
* [Contacts.LOOKUP_KEY](http://developer.android.com/reference/android/provider/ContactsContract.ContactsColumns.html#LOOKUP_KEY)
* [Contacts.PHOTO_THUMBNAIL_URI](http://developer.android.com/reference/android/provider/ContactsContract.ContactsColumns.html#PHOTO_THUMBNAIL_URI)

對於Android 2.3.3（API版本為10）以及之前的版本，則使用以下列：

* [Contacts._ID](http://developer.android.com/reference/android/provider/BaseColumns.html#_ID)
* [Contacts.LOOKUP_KEY](http://developer.android.com/reference/android/provider/ContactsContract.ContactsColumns.html#LOOKUP_KEY)

這一課的剩餘部分假設你已經獲取到了包含這些以及其他你可能選擇的數據列的Cursor對象。想要學習如何獲取這些列對象的Cursor，請參閱課程[獲取聯繫人列表](retrieve-names.html)。

## 設置聯繫人URI和縮略圖

一旦我們已經擁有了所需的數據列，那麼我們就可以為QuickContactBadge視圖綁定數據了。

### 設置聯繫人URI

為了設置聯繫人URI，需要調用[getLookupUri(id, lookupKey)]()去獲取[CONTENT_LOOKUP_URI](http://developer.android.com/reference/android/provider/ContactsContract.Contacts.html#CONTENT_LOOKUP_URI)，然後調用[assignContactUri()](http://developer.android.com/reference/android/widget/QuickContactBadge.html#assignContactUri(android.net.Uri))去為QuickContactBadge設置對應的聯繫人。例如：

```java
// The Cursor that contains contact rows
Cursor mCursor;
// The index of the _ID column in the Cursor
int mIdColumn;
// The index of the LOOKUP_KEY column in the Cursor
int mLookupKeyColumn;
// A content URI for the desired contact
Uri mContactUri;
// A handle to the QuickContactBadge view
QuickContactBadge mBadge;
...
mBadge = (QuickContactBadge) findViewById(R.id.quickbadge);
/*
 * Insert code here to move to the desired cursor row
 */
// Gets the _ID column index
mIdColumn = mCursor.getColumnIndex(Contacts._ID);
// Gets the LOOKUP_KEY index
mLookupKeyColumn = mCursor.getColumnIndex(Contacts.LOOKUP_KEY);
// Gets a content URI for the contact
mContactUri =
        Contacts.getLookupUri(
            mCursor.getLong(mIdColumn),
            mCursor.getString(mLookupKeyColumn)
        );
mBadge.assignContactUri(mContactUri);
```

當用戶點擊QuickContactBadge圖標的時候，這個聯繫人的詳細信息將會自動展現在對話框中。

### 設置聯繫人照片的縮略圖

為QuickContactBadge設置聯繫人URI並不會自動加載聯繫人的縮略圖照片。為了加載聯繫人照片，需要從聯繫人的Cursor對象的一行數據中獲取照片的URI，使用這個URI去打開包含壓縮的縮略圖文件，並把這個文件讀到Bitmap對象中。

> **Note：**<a href="http://developer.android.com/reference/android/provider/ContactsContract.ContactsColumns.html#PHOTO_THUMBNAIL_URI">PHOTO\_THUMBNAIL\_URI</a>這一列在Android 3.0之前的版本是不存在的。對於這些版本，我們必須從[Contacts.Photo](http://developer.android.com/reference/android/provider/ContactsContract.Contacts.Photo.html)表中獲取照片的URI。

首先，為包含Contacts._ID和Contacts.LOOKUP_KEY的Cursor數據列設置對應的變量，這在之前已經有描述：

```java
// The column in which to find the thumbnail ID
int mThumbnailColumn;
/*
 * The thumbnail URI, expressed as a String.
 * Contacts Provider stores URIs as String values.
 */
String mThumbnailUri;
...
/*
 * Gets the photo thumbnail column index if
 * platform version >= Honeycomb
 */
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
    mThumbnailColumn =
            mCursor.getColumnIndex(Contacts.PHOTO_THUMBNAIL_URI);
// Otherwise, sets the thumbnail column to the _ID column
} else {
    mThumbnailColumn = mIdColumn;
}
/*
 * Assuming the current Cursor position is the contact you want,
 * gets the thumbnail ID
 */
mThumbnailUri = mCursor.getString(mThumbnailColumn);
...
```

定義一個方法，使用與這個聯繫人的照片有關的數據和目標視圖的尺寸作為參數，返回一個尺寸合適的縮略圖Bitmap對象。下面先構建一個指向這個縮略圖的URI：

```java
 /**
 * Load a contact photo thumbnail and return it as a Bitmap,
 * resizing the image to the provided image dimensions as needed.
 * @param photoData photo ID Prior to Honeycomb, the contact's _ID value.
 * For Honeycomb and later, the value of PHOTO_THUMBNAIL_URI.
 * @return A thumbnail Bitmap, sized to the provided width and height.
 * Returns null if the thumbnail is not found.
 */
private Bitmap loadContactPhotoThumbnail(String photoData) {
    // Creates an asset file descriptor for the thumbnail file.
    AssetFileDescriptor afd = null;
    // try-catch block for file not found
    try {
        // Creates a holder for the URI.
        Uri thumbUri;
        // If Android 3.0 or later
        if (Build.VERSION.SDK_INT
                >=
            Build.VERSION_CODES.HONEYCOMB) {
            // Sets the URI from the incoming PHOTO_THUMBNAIL_URI
            thumbUri = Uri.parse(photoData);
        } else {
        // Prior to Android 3.0, constructs a photo Uri using _ID
            /*
             * Creates a contact URI from the Contacts content URI
             * incoming photoData (_ID)
             */
            final Uri contactUri = Uri.withAppendedPath(
                    Contacts.CONTENT_URI, photoData);
            /*
             * Creates a photo URI by appending the content URI of
             * Contacts.Photo.
             */
            thumbUri =
                    Uri.withAppendedPath(
                            contactUri, Photo.CONTENT_DIRECTORY);
        }

    /*
     * Retrieves an AssetFileDescriptor object for the thumbnail
     * URI
     * using ContentResolver.openAssetFileDescriptor
     */
    afd = getActivity().getContentResolver().
            openAssetFileDescriptor(thumbUri, "r");
    /*
     * Gets a file descriptor from the asset file descriptor.
     * This object can be used across processes.
     */
    FileDescriptor fileDescriptor = afd.getFileDescriptor();
    // Decode the photo file and return the result as a Bitmap
    // If the file descriptor is valid
    if (fileDescriptor != null) {
        // Decodes the bitmap
        return BitmapFactory.decodeFileDescriptor(
                fileDescriptor, null, null);
        }
    // If the file isn't found
    } catch (FileNotFoundException e) {
        /*
         * Handle file not found errors
         */
    }
    // In all cases, close the asset file descriptor
    } finally {
        if (afd != null) {
            try {
                afd.close();
            } catch (IOException e) {}
        }
    }
    return null;
}
```

在代碼中調用loadContactPhotoThumbnail()去獲取縮略圖Bitmap對象，使用獲取的Bitmap對象去設置QuickContactBadge頭像縮略圖。


```java
...
/*
 * Decodes the thumbnail file to a Bitmap.
 */
Bitmap mThumbnail =
        loadContactPhotoThumbnail(mThumbnailUri);
/*
 * Sets the image in the QuickContactBadge
 * QuickContactBadge inherits from ImageView, so
 */
mBadge.setImageBitmap(mThumbnail);
```

## 把QuickContactBadge添加到ListView


QuickContactBadge對於一個展示聯繫人列表的ListView來說是一個非常有用的添加功能。使用QuickContactBadge去為每一個聯繫人顯示一個縮略圖，當用戶點擊這個縮略圖時，QuickContactBadge對話框將會顯示。

### 為ListView添加QuickContactBadge

首先，在列表項佈局文件中添加QuickContactBadge視圖元素。例如，如果我們想為獲取到的每一個聯繫人顯示QuickContactBadge和名字，把以下的XML內容放到對應的佈局文件中：

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="wrap_content">
    <QuickContactBadge
        android:id="@+id/quickcontact"
        android:layout_height="wrap_content"
        android:layout_width="wrap_content"
        android:scaleType="centerCrop"/>
    <TextView android:id="@+id/displayname"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:layout_toRightOf="@+id/quickcontact"
              android:gravity="center_vertical"
              android:layout_alignParentRight="true"
              android:layout_alignParentTop="true"/>
</RelativeLayout>
```

在以下的章節中，這個文件被稱為`contact_item_layout.xml`。

### 設置自定義的CursorAdapter

定義一個繼承自CursorAdapter的adapter來將CursorAdapter綁定到一個包含QuickContactBadge的ListView中。這種方式允許我們在綁定數據到QuickContactBadge之前對Cursor中的數據進行處理。同時也能將多個Cursor中的列綁定到QuickContactBadge。而使用普通的CursorAdapter是不能完成這些操作的。

我們定義的CursorAdapter的子類必須重寫以下方法：

* [CursorAdapter.newView()]()

	填充一個View對象去持有列表項佈局。在重寫這個方法的過程中，需要保存這個佈局的子View的handles，包括QuickContactBadge的handles。通過採用這種方法，避免了每次在填充新的佈局時都去獲取子View的handles。

	我們必須重寫這個方法以便能夠獲取每個子View對象的handles。這種方法允許我們控制這些子View對象在CursorAdapter.bindView()方法中的綁定。

* [CursorAdapter.bindView()]()

	將數據從當前Cursor行綁定到列表項佈局的子View對象中。必須重寫這個方法以便能夠將聯繫人的URI和縮略圖信息綁定到QuickContactBadge。這個方法的默認實現僅僅允許在數據列和View之間的一對一映射。


以下的代碼片段是一個包含了自定義CursorAdapter子類的例子。

### 定義自定義的列表Adapter

定義CursorAdapter的子類包括編寫這個類的構造方法，以及重寫newView()和bindView():

```java
private class ContactsAdapter extends CursorAdapter {
    private LayoutInflater mInflater;
    ...
    public ContactsAdapter(Context context) {
        super(context, null, 0);

        /*
         * Gets an inflater that can instantiate
         * the ListView layout from the file.
         */
        mInflater = LayoutInflater.from(context);
        ...
    }
    ...
    /**
     * Defines a class that hold resource IDs of each item layout
     * row to prevent having to look them up each time data is
     * bound to a row.
     */
    private class ViewHolder {
        TextView displayname;
        QuickContactBadge quickcontact;
    }
    ..
    @Override
    public View newView(
            Context context,
            Cursor cursor,
            ViewGroup viewGroup) {
        /* Inflates the item layout. Stores resource IDs in a
         * in a ViewHolder class to prevent having to look
         * them up each time bindView() is called.
         */
        final View itemView =
                mInflater.inflate(
                        R.layout.contact_list_layout,
                        viewGroup,
                        false
                );
        final ViewHolder holder = new ViewHolder();
        holder.displayname =
                (TextView) view.findViewById(R.id.displayname);
        holder.quickcontact =
                (QuickContactBadge)
                        view.findViewById(R.id.quickcontact);
        view.setTag(holder);
        return view;
    }
    ...
    @Override
    public void bindView(
            View view,
            Context context,
            Cursor cursor) {
        final ViewHolder holder = (ViewHolder) view.getTag();
        final String photoData =
                cursor.getString(mPhotoDataIndex);
        final String displayName =
                cursor.getString(mDisplayNameIndex);
        ...
        // Sets the display name in the layout
        holder.displayname = cursor.getString(mDisplayNameIndex);
        ...
        /*
         * Generates a contact URI for the QuickContactBadge.
         */
        final Uri contactUri = Contacts.getLookupUri(
                cursor.getLong(mIdIndex),
                cursor.getString(mLookupKeyIndex));
        holder.quickcontact.assignContactUri(contactUri);
        String photoData = cursor.getString(mPhotoDataIndex);
        /*
         * Decodes the thumbnail file to a Bitmap.
         * The method loadContactPhotoThumbnail() is defined
         * in the section "Set the Contact URI and Thumbnail"
         */
        Bitmap thumbnailBitmap =
                loadContactPhotoThumbnail(photoData);
        /*
         * Sets the image in the QuickContactBadge
         * QuickContactBadge inherits from ImageView
         */
        holder.quickcontact.setImageBitmap(thumbnailBitmap);
}
```

### 設置變量

在代碼中，設置相關變量，添加一個包括必須數據列的Cursor。

> **Note：**以下的代碼片段使用了方法`loadContactPhotoThumbnail()`，這個方法是在[設置聯繫人URI和縮略圖]()那一節中定義的。

例如：

```java
public class ContactsFragment extends Fragment implements
        LoaderManager.LoaderCallbacks<Cursor> {
...
// Defines a ListView
private ListView mListView;
// Defines a ContactsAdapter
private ContactsAdapter mAdapter;
...
// Defines a Cursor to contain the retrieved data
private Cursor mCursor;
/*
 * Defines a projection based on platform version. This ensures
 * that you retrieve the correct columns.
 */
private static final String[] PROJECTION =
        {
            Contacts._ID,
            Contacts.LOOKUP_KEY,
            (Build.VERSION.SDK_INT >=
             Build.VERSION_CODES.HONEYCOMB) ?
                    Contacts.DISPLAY_NAME_PRIMARY :
                    Contacts.DISPLAY_NAME
            (Build.VERSION.SDK_INT >=
             Build.VERSION_CODES.HONEYCOMB) ?
                    Contacts.PHOTO_THUMBNAIL_ID :
                    /*
                     * Although it's not necessary to include the
                     * column twice, this keeps the number of
                     * columns the same regardless of version
                     */
                    Contacts_ID
            ...
        };
/*
 * As a shortcut, defines constants for the
 * column indexes in the Cursor. The index is
 * 0-based and always matches the column order
 * in the projection.
 */
// Column index of the _ID column
private int mIdIndex = 0;
// Column index of the LOOKUP_KEY column
private int mLookupKeyIndex = 1;
// Column index of the display name column
private int mDisplayNameIndex = 3;
/*
 * Column index of the photo data column.
 * It's PHOTO_THUMBNAIL_URI for Honeycomb and later,
 * and _ID for previous versions.
 */
private int mPhotoDataIndex =
        Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB ?
        3 :
        0;
...
```

### 設置ListView

在[Fragment.onCreate()](http://developer.android.com/reference/android/support/v4/app/Fragment.html#onCreate(android.os.Bundle))方法中，實例化自定義的adapter對象，獲得一個ListView的handle。

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    ...
    /*
     * Instantiates the subclass of
     * CursorAdapter
     */
    ContactsAdapter mContactsAdapter =
            new ContactsAdapter(getActivity());
    /*
     * Gets a handle to the ListView in the file
     * contact_list_layout.xml
     */
    mListView = (ListView) findViewById(R.layout.contact_list_layout);
    ...
}
...
```

在[onActivityCreated()](http://developer.android.com/reference/android/support/v4/app/Fragment.html#onActivityCreated(android.os.Bundle))方法中，將ContactsAdapter綁定到ListView。

```java
@Override
public void onActivityCreated(Bundle savedInstanceState) {
    ...
    // Sets up the adapter for the ListView
    mListView.setAdapter(mAdapter);
    ...
}
...
```

當獲取到一個包含聯繫人數據的Cursor時（通常在onLoadFinished()的時候），調用swapCursor()把Cursor中的數據綁定到ListView。這將會為聯繫人列表中的每一項都顯示一個QuickContactBadge。

```java
public void onLoadFinished(Loader<Cursor> loader, Cursor cursor) {
    // When the loader has completed, swap the cursor into the adapter.
    mContactsAdapter.swapCursor(cursor);
}
```

當我們使用CursorAdapter或其子類中將Cursor中的數據綁定到ListView，並且使用了CursorLoader去加載Cursor數據時，記得要在onLoaderReset()方法的實現中清理對Cursor對象的引用。例如：

```java
@Override
public void onLoaderReset(Loader<Cursor> loader) {
    // Removes remaining reference to the previous Cursor
    mContactsAdapter.swapCursor(null);
}
```




