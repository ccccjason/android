# 聯繫人信息

> 編寫:[spencer198711](https://github.com/spencer198711) - 原文:<http://developer.android.com/training/contacts-provider/index.html>

**[Contacts Provider](http://developer.android.com/guide/topics/providers/contacts-provider.html)**是用戶聯繫人信息的集中倉庫， 它包含了來自聯繫人應用與社交應用的聯繫人數據。在我們的應用中，我們可以通過調用[**ContentResolver**](http://developer.android.com/reference/android/content/ContentResolver.html)方法或者通過發送Intent給聯繫人應用來訪問Contacts Provider的信息。

這個章節會講解獲取聯繫人列表，顯示指定聯繫人詳情以及通過intent來修改聯繫人信息。這裡介紹的基礎技能能夠擴展到執行更復雜的任務。另外，這個章節也會幫助我們瞭解Contacts Provider的整個架構與操作方法。

## Lessons

[**獲取聯繫人列表**](retrieve-names.html)

學習如何獲取聯繫人列表。你可以使用下面的技術來篩選需要的信息：

  * 通過聯繫人名字進行篩選
  * 通過聯繫人類型進行篩選
  * 通過類似電話號碼等指定的一類信息進行篩選。


[**獲取聯繫人詳情**](retrieve-detail.html)

學習如何獲取單個聯繫人的詳情。一個聯繫人的詳細信息包括電話號碼與郵件地址等等。你可以獲取所有的詳細信息，也有可以只獲取指定類型的詳細數據，例如郵件地址。


[**使用Intents修改聯繫人信息**](modify-data.html)

學習如何通過發送intent給聯繫人應用來修改聯繫人信息。


[**顯示聯繫人頭像**](display-badge.html)

學習如何顯示**QuickContactBadge**小組件。當用戶點擊聯繫人臂章(頭像)組件時，會打開一個對話框，這個對話框會顯示聯繫人詳情，並提供操作按鈕來處理詳細信息。例如，如果聯繫人信息有郵件地址，這個對話框可以顯示一個啟動默認郵件應用的操作按鈕。
