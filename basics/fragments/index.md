# 使用Fragment建立動態UI

> 編寫:[fastcome1985](https://github.com/fastcome1985) - 原文:<http://developer.android.com/training/basics/fragments/index.html>

* 為了在Android上為用戶提供動態的、多窗口的交互體驗，我們需要將UI組件和Activity操作封裝成模塊進行使用，使得我們可以在activity中對這些模塊進行切入切出操作。可以用[Fragment](http://developer.android.com/intl/zh-cn/reference/android/app/Fragment.html)來創建這些模塊，Fragment就像一個嵌套的activity，擁有自己的佈局（layout）並管理自己的生命週期。

*0 一個fragment定義了自己的佈局後，它可以在activity中與其他的fragment生成不同的組合，從而為不同的屏幕尺寸生成不同的佈局（一個小的屏幕一次也許只能一個fragment，大的屏幕則可以顯示更多）。

* 本章將展示如何用fragment來創建動態界面，並在不同屏幕尺寸的設備上優化APP的用戶體驗。即使是運行著android1.6這樣老版本的設備，也都能夠繼續得到支持。

* 完整的Demo示例：[FragmentBasics.zip](http://developer.android.com/shareables/training/FragmentBasics.zip "FragmentBasics.zip")

## Lessons

* [**創建一個fragment**](creating.html)

  學習如何創建一個fragment，以及實現其生命週期內的基本功能。


* [**構建有彈性的UI**](fragment-ui.html)

  學習在APP內，對不同的屏幕尺寸用fragment構建不同的佈局。


* [**與其他fragments交互**](communicating.html)

  學習fragment與activity及其他fragment間進行交互。

