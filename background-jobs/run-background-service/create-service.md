# 創建後臺服務

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/run-background-service/create-service.html>

IntentService為在單一後臺線程中執行任務提供了一種直接的實現方式。它可以處理一個耗時的任務並確保不影響到UI的響應性。另外IntentService的執行還不受UI生命週期的影響，以此來確保AsyncTask能夠順利運行。

但是IntentService有下面幾個侷限性：

* 不可以直接和UI做交互。為了把他執行的結果體現在UI上，需要把結果返回給Activity。
* 工作任務隊列是順序執行的，如果一個任務正在IntentService中執行，此時你再發送一個新的任務請求，這個新的任務會一直等待直到前面一個任務執行完畢才開始執行。
* 正在執行的任務無法打斷。

雖然有上面那些限制，然而在在大多數情況下，IntentService都是執行簡單後臺任務操作的理想選擇。

這節課會演示如何創建繼承的IntentService。同樣也會演示如何創建必須的回調方法`onHandleIntent()`。最後，還會解釋如何在manifest文件中定義這個IntentService。

<!-- More -->

## 1)創建IntentService

為你的app創建一個IntentService組件，需要自定義一個新的類，它繼承自IntentService，並重寫onHandleIntent()方法，如下所示：

```java
public class RSSPullService extends IntentService {
    @Override
    protected void onHandleIntent(Intent workIntent) {
        // Gets data from the incoming Intent
        String dataString = workIntent.getDataString();
        ...
        // Do work here, based on the contents of dataString
        ...
    }
}
```

注意一個普通Service組件的其他回調，例如`onStartCommand()`會被IntentService自動調用。在IntentService中，要避免重寫那些回調。

## 2)在Manifest文件中定義IntentService

IntentService需要在manifest文件添加相應的條目，將此條目`<service>`作為`<application>`元素的子元素下進行定義，如下所示：

```xml
<application
    android:icon="@drawable/icon"
    android:label="@string/app_name">
    ...
    <!--
        Because android:exported is set to "false",
        the service is only available to this app.
    -->
    <service
        android:name=".RSSPullService"
        android:exported="false"/>
    ...
<application/>
```

`android:name`屬性指明瞭IntentService的名字。

注意`<service>`標籤並沒有包含任何intent filter。因為發送任務給IntentService的Activity需要使用顯式Intent，所以不需要filter。這也意味著只有在同一個app或者其他使用同一個UserID的組件才能夠訪問到這個Service。

至此，你已經有了一個基本的IntentService類，你可以通過構造Intent對象向它發送操作請求。構造這些對象以及發送它們到你的IntentService的方式，將在接下來的課程中描述。
