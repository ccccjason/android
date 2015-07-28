# 分享文件

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/secure-file-sharing/index.html>

一個程序經常需要向其他程序提供一個甚至多個文件。例如，當我們用圖片編輯器編輯圖片時，被編輯的圖片往往由圖庫應用程序所提供；再比如，文件管理器會允許用戶在外部存儲的不同區域之間複製粘貼文件。這裡，我們提出一種讓應用程序可以分享文件的方法：即令發送文件的應用程序對索取文件的應用程序所發出的文件請求進行響應。

在任何情況下，將文件從我們的應用程序發送至其它應用程序的唯一的安全方法是向接收文件的應用程序發送這個文件的content URI，並對該URI授予臨時訪問權限。具有URI臨時訪問權限的content URI是安全的，因為他們僅應用於接收這個URI的應用程序，並且會自動過期。Android的[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)組件提供了<a href="http://developer.android.com/reference/android/support/v4/content/FileProvider.html#getUriForFile(android.content.Context, java.lang.String, java.io.File)">getUriForFile()</a>方法創建一個文件的content URI。

如果希望在應用之間共享少量的文本或者數字等類型的數據，應使用包含該數據的Intent。要學習如何通過Intent發送簡單數據，可以閱讀：[Sharing Simple Data](../sharing/index.html)。

本課主要介紹瞭如何使用Android的[FileProvider](http://developer.android.com/reference/android/support/v4/content/FileProvider.html)組件所創建的content URI在應用之間安全的共享文件。當然，要做到這一點，還需要給接收文件的應用程序訪問的這些content URI授予臨時訪問權限。

## Lessons

* [**建立文件分享**](setup-sharing.html)

  學習如何配置應用程序使得它們可以分享文件。


* [**分享文件**](sharing-file.html)

  學習分享文件的三個步驟：
  - 生成文件的content URI；
  - 授予URI的臨時訪問權限；
  - 將URI發送給接收文件的應用程序。


* [**請求分享一個文件**](request-file.html)

  學習如何向其他應用程序請求文件，如何接收該文件的content URI，以及如何使用content URI打開該文件。


* [**獲取文件信息**](retrieve-info.html)

  學習應用程序如何通過FileProvider提供的content URI獲取文件的信息：例如MIME類型，文件大小等。
