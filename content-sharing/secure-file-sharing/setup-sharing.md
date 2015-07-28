# 建立文件分享

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/secure-file-sharing/setup-sharing.html>

為了將文件安全地從我們的應用程序共享給其它應用程序，我們需要對自己的應用進行配置來提供安全的文件句柄（Content URI的形式）。Android的[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)組件會基於在XML文件中的具體配置為文件創建Content URI。本課將介紹如何在應用程序中添加[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)的默認實現，以及如何指定要共享的文件。

> **Note:**[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)類隸屬於[v4 Support Library](http://developer.android.com/tools/support-library/features.html#v4)庫。關於如何在應用程序中包含該庫，請參考：[Support Library Setup](http://developer.android.com/tools/support-library/setup.html)。

## 指定FileProvider

為了給應用程序定義一個[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)，需要在Manifest清單文件中定義一個entry，該entry指明瞭需要使用的創建Content URI的Authority。此外，還需要一個XML文件的文件名，該XML文件指定了我們的應用可以共享的目錄路徑。

下例展示瞭如何在清單文件中添加[`<provider>`](http://developer.android.com/guide/topics/manifest/provider-element.html)標籤，來指定[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)類，Authority及XML文件名：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapp">
    <application
        ...>
        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="com.example.myapp.fileprovider"
            android:grantUriPermissions="true"
            android:exported="false">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/filepaths" />
        </provider>
        ...
    </application>
</manifest>
```
這裡，[android:authorities](http://developer.android.com/guide/topics/manifest/provider-element.html#auth)字段指定了希望使用的Authority，該Authority針對於[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)所生成的content URI。本例中的Authority是“com.example.myapp.fileprovider”。對於自己的應用，要在我們的應用程序包名（[android:package](http://developer.android.com/guide/topics/manifest/manifest-element.html#package)的值）之後繼續追加“fileprovider”來指定Authority。要更多關於Authority的知識，請參考：[Content URIs](http://developer.android.com/guide/topics/providers/content-provider-basics.html#ContentURIs)，以及[android:authorities](http://developer.android.com/guide/topics/manifest/provider-element.html#auth)。

[`<provider>`](http://developer.android.com/guide/topics/manifest/provider-element.html)下的[`<meta-data>`](http://developer.android.com/guide/topics/manifest/meta-data-element.html)指向了一個XML文件，該文件指定了我們希望共享的目錄路徑。“android:resource”屬性字段是這個文件的路徑和名字（無“.xml”後綴）。該文件的內容將在下一節討論。

## 指定可共享目錄路徑

一旦在Manifest清單文件中為自己的應用添加了[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)，就需要指定我們希望共享文件的目錄路徑。為指定該路徑，首先要在“res/xml/”下創建文件“filepaths.xml”。在這個文件中，為每一個想要共享目錄添加一個XML標籤。下面的是一個“res/xml/filepaths.xml”的內容樣例。這個例子也說明了如何在內部存儲區域共享一個“files/”目錄的子目錄：

```xml
<paths>
    <files-path path="images/" name="myimages" />
</paths>
```

在這個例子中，`<files-path>`標籤共享的是在我們應用的內部存儲中“files/”目錄下的目錄。“path”屬性字段指出了該子目錄為“files/”目錄下的子目錄“images/”。“name”屬性字段告知[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)在“files/images/”子目錄中的文件的Content URI添加路徑分段（path segment）標記：“myimages”。

`<paths>`標籤可以有多個子標籤，每一個子標籤用來指定不同的共享目錄。除了`<files-path>`標籤，還可以使用`<external-path>`來共享位於外部存儲的目錄；另外，`<cache-path>`標籤用來共享在內部緩存目錄下的子目錄。更多關於指定共享目錄子標籤的知識請參考：[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)。

> **Note:** XML文件是我們定義共享目錄的唯一方式，不可以用代碼的形式添加目錄。

現在我們有一個完整的[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)聲明，它在應用程序的內部存儲中“files/”目錄或其子目錄下創建文件的Content URI。當我們的應用為一個文件創建了Content URI，該Content URI將會包含下列信息：
- [`<provider>`](http://developer.android.com/guide/topics/manifest/provider-element.html)標籤中指定的Authority（“com.example.myapp.fileprovider”）；
- 路徑“myimages/”；
- 文件的名字。

例如，如果本課的例子定義了一個[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)，然後我們需要一個文件“default_image.jpg”的Content URI，[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)會返回如下URI：

```
content://com.example.myapp.fileprovider/myimages/default_image.jpg
```
