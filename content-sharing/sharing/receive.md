# 接收從其他App傳送來的數據

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/sharing/receive.html>

就像我們的程序能夠分享數據給其他程序一樣，其也能方便的接收來自其他程序的數據。需要考慮的是用戶與我們的程序如何進行交互，以及我們想要從其他程序接收數據的類型。例如，一個社交網絡程序可能會希望能夠從其他程序接受文本數據，比如一個有趣的網址鏈接。Google+的Android客戶端會接受文本數據與單張或者多張圖片。用戶可以簡單的從Gallery程序選擇一張圖片來啟動Google+，並利用其發佈文本或圖片。

<!-- more -->

## 更新我們的manifest文件(Update Your Manifest)

Intent filters告訴Android系統一個程序願意接受的數據類型。類似於上一課，我們可以創建intent filters來表明程序能夠接收的action類型。下面是個例子，對三個activit分別指定接受單張圖片，文本與多張圖片。(Intent filter相關資料，請參考[Intents and Intent Filters](http://developer.android.com/guide/topics/intents/intents-filters.html#ifs))

```xml
<activity android:name=".ui.MyActivity" >
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="image/*" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.SEND_MULTIPLE" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="image/*" />
    </intent-filter>
</activity>
```

當某個程序嘗試通過創建一個intent並將其傳遞給startActivity來分享一些東西時，我們的程序會被呈現在一個列表中讓用戶進行選擇。如果用戶選擇了我們的程序，相應的activity會被調用開啟，這個時候就是我們如何處理獲取到的數據的問題了。

## 處理接受到的數據(Handle the Incoming Content)

為了處理從Intent帶來的數據，可以通過調用getIntent()方法來獲取到Intent對象。拿到這個對象後，我們可以對其中面的數據進行判斷，從而決定下一步行為。請記住，如果一個activity可以被其他的程序啟動，我們需要在檢查intent的時候考慮這種情況(是被其他程序而調用啟動的)。

```java
void onCreate (Bundle savedInstanceState) {
    ...
    // Get intent, action and MIME type
    Intent intent = getIntent();
    String action = intent.getAction();
    String type = intent.getType();

    if (Intent.ACTION_SEND.equals(action) && type != null) {
        if ("text/plain".equals(type)) {
            handleSendText(intent); // Handle text being sent
        } else if (type.startsWith("image/")) {
            handleSendImage(intent); // Handle single image being sent
        }
    } else if (Intent.ACTION_SEND_MULTIPLE.equals(action) && type != null) {
        if (type.startsWith("image/")) {
            handleSendMultipleImages(intent); // Handle multiple images being sent
        }
    } else {
        // Handle other intents, such as being started from the home screen
    }
    ...
}

void handleSendText(Intent intent) {
    String sharedText = intent.getStringExtra(Intent.EXTRA_TEXT);
    if (sharedText != null) {
        // Update UI to reflect text being shared
    }
}

void handleSendImage(Intent intent) {
    Uri imageUri = (Uri) intent.getParcelableExtra(Intent.EXTRA_STREAM);
    if (imageUri != null) {
        // Update UI to reflect image being shared
    }
}

void handleSendMultipleImages(Intent intent) {
    ArrayList<Uri> imageUris = intent.getParcelableArrayListExtra(Intent.EXTRA_STREAM);
    if (imageUris != null) {
        // Update UI to reflect multiple images being shared
    }
}
```

請注意，由於無法知道其他程序發送過來的數據內容是文本還是其他類型的數據，若數據量巨大，則需要大量處理時間，因此我們應避免在UI線程裡面去處理那些獲取到的數據。

更新UI可以像更新EditText一樣簡單，也可以是更加複雜一點的操作，例如過濾出感興趣的圖片。這完全取決於我們的應用接下來要做些什麼。

*********************************
