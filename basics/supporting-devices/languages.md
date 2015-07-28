# 適配不同的語言

> 編寫:[Lin-H](http://github.com/Lin-H) - 原文:<http://developer.android.com/training/basics/supporting-devices/languages.html>

把UI中的字符串存儲在外部文件，通過代碼提取，這是一種很好的做法。Android可以通過工程中的資源目錄輕鬆實現這一功能。

如果使用Android SDK Tools(詳見[創建Android項目(Creating an Android Project)](../../basics/firstapp/creating-project.html))來創建工程，則在工程的根目錄會創建一個`res/`的目錄，目錄中包含所有資源類型的子目錄。其中包含工程的默認文件比如`res/values/strings.xml`，用於保存字符串值。

## 創建區域設置目錄及字符串文件

為支持多國語言，在`res/`中創建一個額外的`values`目錄以連字符和ISO國家代碼結尾命名，比如`values-es/` 是為語言代碼為"es"的區域設置的簡單的資源文件的目錄。Android會在運行時根據設備的區域設置，加載相應的資源。詳見[Providing Alternative Resources](http://developer.android.com/guide/topics/resources/providing-resources.html#AlternativeResources)。

若決定支持某種語言，則需要創建資源子目錄和字符串資源文件，例如:

```
MyProject/
    res/
       values/
           strings.xml
       values-es/
           strings.xml
       values-fr/
           strings.xml
```

添加不同區域語言的字符串值到相應的文件。

Android系統運行時會根據用戶設備當前的區域設置，使用相應的字符串資源。

例如，下面列舉了幾個不同語言對應不同的字符串資源文件。

英語(默認區域語言)，`/values/strings.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="title">My Application</string>
    <string name="hello_world">Hello World!</string>
</resources>
```

西班牙語，`/values-es/strings.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="title">Mi Aplicación</string>
    <string name="hello_world">Hola Mundo!</string>
</resources>
```

法語，`/values-fr/strings.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="title">Mon Application</string>
    <string name="hello_world">Bonjour le monde !</string>
</resources>
```

> **Note**：可以在任何資源類型中使用區域修飾詞(或者任何配置修飾符)，比如為bitmap提供本地化的版本，更多信息見[Localization](https://developer.android.com/guide/topics/resources/localization.html)。

## 使用字符資源

我們可以在源代碼和其他XML文件中通過`<string>`元素的`name`屬性來引用自己的字符串資源。

在源代碼中可以通過`R.string.<string_name>`語法來引用一個字符串資源，很多方法都可以通過這種方式來接受字符串。

例如:

```java
// Get a string resource from your app's Resources
String hello = getResources().getString(R.string.hello_world);

// Or supply a string resource to a method that requires a string
TextView textView = new TextView(this);
textView.setText(R.string.hello_world);
```

在其他XML文件中，每當XML屬性要接受一個字符串值時，你都可以通過`@string/<string_name>`語法來引用字符串資源。

例如:

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/hello_world" />
```
