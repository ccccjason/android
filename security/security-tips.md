# 安全要點

> 編寫:[craftsmanBai](https://github.com/craftsmanBai) - <http://z1ng.net> - 原文:<http://developer.android.com/training/articles/security-tips.html>

Android內建的安全機制可以顯著地減少了應用程序的安全問題。你可以在默認的系統設置和文件權限設置的環境下建立應用，避免針對一堆頭疼的安全問題尋找解決方案。

一些幫助建立應用的核心安全特性如下：

* Android應用程序沙盒，將應用數據和代碼的執行與其他程序隔離。
* 具有魯棒性的常見安全功能的應用框架，例如加密，權限控制，安全IPC
* 使用ASLR，NX，ProPolice，safe_iop，OpenBSD dlmalloc，OpenBSD calloc，Linux mmap_min_addr等技術，減少了常見內存管理錯誤。
* 加密文件系統可以保護丟失或被盜走的設備數據。
* 用戶權限控制限制訪問系統關鍵信息和用戶數據。
* 應用程序權限以單個應用為基礎控制其數據。

儘管如此，熟悉Android安全特性仍然很重要。遵守這些習慣並將其作為優秀的代碼風格，能夠減少無意間給用戶帶來的安全問題。

## 數據存儲

對於一個Android的應用程序來說，最為常見的安全問題是存放在設備上的數據能否被其他應用獲取。在設備上存放數據基本方式有三種:

### 使用內部存儲

默認情況下，你在[內部存儲](http://developer.android.com/guide/topics/data/data-storage.html#filesInternal)中創建的文件只有你的應用可以訪問。Android實現了這種機制，並且對於大多數應用程序都是有效的。
你應該避免在IPC文件中使用[MODE_WORLD_WRITEABLE](http://developer.android.com/reference/android/content/Context.html#MODE_WORLD_WRITEABLE)或者[MODE_WORLD_READABLE](http://developer.android.com/reference/android/content/Context.html#MODE_WORLD_READABLE)模式，因為它們不為特殊程序提供限制數據訪問的功能，它們也不對數據格式進行任何控制。如果你想與其他應用的進程共享數據，可以使用[Content Provider](http://developer.android.com/guide/topics/providers/content-providers.html)，它可以給其他應用提供了可讀寫權限以及逐項動態獲取權限。

如果想對敏感數據進行特別保護，你可以使用應用程序無法直接獲取的密鑰來加密本地文件。例如，密鑰可以存放在[KeyStore](http://developer.android.com/reference/java/security/KeyStore.html)而非設備上，使用用戶密碼進行保護。儘管這種方式無法防止通過root權限查看用戶輸入的密碼，但是它可以為未進行[文件系統加密](http://source.android.com/tech/encryption/index.html)的丟失設備提供保護。

### 使用外部存儲

創建於[外部存儲](http://developer.android.com/guide/topics/data/data-storage.html#filesExternal)的文件，比如SD卡，是全局可讀寫的。
由於外部存儲器可被用戶移除並且能夠被任何應用修改，因此不應使用外部存儲保存應用的敏感信息。
當處理來自外部存儲的數據時，應用程序應該[執行輸入驗證](http://developer.android.com/training/articles/security-tips.html#InputValidation)（參看輸入驗證章節）
我們強烈建議應用在動態加載之前不要把可執行文件或class文件存儲到外部存儲中。
如果一個應用從外部存儲檢索可執行文件，那麼在動態加載之前它們應該進行簽名與加密驗證。

### 使用Content Providers

[ContentProviders](http://developer.android.com/guide/topics/providers/content-providers.html)提供了一種結構存儲機制，它可以限制你自己的應用，也可以允許其他應用程序進行訪問。
如果你不打算向其他應用提供訪問你的[ContentProvider](http://developer.android.com/reference/android/content/ContentProvider.html)功能，那麼在manifest中標記他們為[android:exported=false](http://developer.android.com/guide/topics/manifest/provider-element.html#exported)即可。
要建立一個給其他應用使用的[ContentProvider](http://developer.android.com/reference/android/content/ContentProvider.html)，你可以為讀寫操作指定一個單一的[permission](http://developer.android.com/guide/topics/manifest/provider-element.html#prmsn)，或者在manifest中為讀寫操作指定確切的權限。我們強烈建議你對要分配的權限進行限制，僅滿足目前有的功能即可。
記住，通常新的權限在新功能加入的時候同時增加，會比把現有權限撤銷並打斷已經存在的用戶更合理。

如果Content Provider僅在自己的應用中共享數據，使用簽名級別[android:protectionLevel](http://developer.android.com/guide/topics/manifest/permission-element.html#plevel)的權限是更可取的。
簽名權限不需要用戶確認，當應用使用同樣的密鑰獲取數據時，這提供了更好的用戶體驗，也更好地控制了Content Provider數據的訪問。
Content Providers也可以通過聲明[android:grantUriPermissions](http://developer.android.com/guide/topics/manifest/provider-element.html#gprmsn)並在觸發組件的Intent對象中使用[FLAG_GRANT_READ_URI_PERMISSION](http://developer.android.com/reference/android/content/Intent.html#FLAG_GRANT_READ_URI_PERMISSION)和[FLAG_GRANT_WRITE_URI_PERMISSION](http://developer.android.com/reference/android/content/Intent.html#FLAG_GRANT_WRITE_URI_PERMISSION)標誌提供更細緻的訪問。
這些許可的作用域可以通過[grant-uri-permission](http://developer.android.com/guide/topics/manifest/grant-uri-permission-element.html)進一步限制。
當訪問一個ContentProvider時，使用參數化的查詢方法，比如<a href="http://developer.android.com/reference/android/content/ContentProvider.html#query(android.net.Uri, java.lang.String[], java.lang.String, java.lang.String[], java.lang.String">query()</a>，<a href="http://developer.android.com/reference/android/content/ContentProvider.html#update(android.net.Uri, android.content.ContentValues, java.lang.String, java.lang.String[]">update()</a>和<a href="http://developer.android.com/reference/android/content/ContentProvider.html#delete(android.net.Uri, java.lang.String, java.lang.String[]">delete()</a>來避免來自不信任源潛在的SQL注入。
注意，如果selection語句是在提交給方法之前先連接用戶數據的，使用參數化的方法或許不夠。
不要對“寫”權限有一個錯誤的觀念。
考慮“寫”權限允許sql語句，它可以通過使用創造性的WHERE子句並且解析結果讓部分數據的確認變為可能。
例如：入侵者可能在通話記錄中通過修改一條記錄來檢測某個特定存在的電話號碼，只要那個電話號碼已經存在。
如果content provider數據有可預見的結構，提供“寫”權限也許等同於同時提供了“讀寫”權限。

## 使用權限

因為安卓沙盒將應用程序隔離，程序必須顯式地共享資源和數據。它們通過聲明他們需要的權限來獲取額外的功能，而基本的沙盒不提供這些功能，比如相機訪問設備。

### 請求權限

我們建議最小化應用請求的權限數量，不具有訪問敏感資料的權限可以減少無意中濫用這些權限的風險，可以增加用戶接受度，並且減少應用被攻擊者攻擊利用的可能性。

如果你的應用可以設計成不需要任何權限，那最好不過。例如：與其請求訪問設備信息來建立一個標識，不如建立一個[GUID](http://developer.android.com/reference/java/util/UUID.html)（這個例子在下文“處理用戶數據”中有說明）。

除了請求權限之外，你的應用可以使用[permissions](http://developer.android.com/guide/topics/manifest/permission-element.html)來保護可能會暴露給其他應用的安全敏感的IPC：比如[ContentProvider](http://developer.android.com/reference/android/content/ContentProvider.html)。通常來說，我們建議使用訪問控制而不是用戶權限確認許可，因為權限會使用戶感到困惑。例如，考慮在權限設置上為應用間的IPC通信使用單一開發者提供的[簽名保護級別](http://developer.android.com/guide/topics/manifest/permission-element.html#plevel)。

不要洩漏受許可保護的數據。只有當應用通過IPC暴露數據才會發生這種情況，因為它具有特殊權限，卻不要求任何客戶端的IPC接口有那樣的權限。更多關於這方面的潛在影響以及這種問題發生的頻率在USENIX: [http://www.cs.be rkeley.edu/~afelt/felt_usenixsec2011.pdf](http://www.cs.berkeley.edu/~afelt/felt_usenixsec2011.pdf)研究論文中都有說明。

### 創建權限

通常，你應該力求建立擁有儘量少權限的應用，直至滿足你的安全需要。建立一個新的權限對於大多數應用相對少見，因為[系統定義的許可](http://developer.android.com/reference/android/Manifest.permission.html)覆蓋很多情況。在適當的地方使用已經存在的許可執行訪問檢查。

如果必須建立一個新的權限，考慮能否使用[signature protection level](http://developer.android.com/guide/topics/manifest/permission-element.html#plevel)來完成你的任務。簽名許可對用戶是透明的並且只允許相同開發者簽名的應用訪問，與應用執行權限檢查一樣。如果你建立一個[dagerous protction level](http://developer.android.com/guide/topics/manifest/permission-element.html#plevel)，那麼用戶需要決定是否安裝這個應用。這會使其他開發者困惑，也使用戶困惑。

如果你要建立一個危險的許可，則會有多種複雜情況需考慮：

*   對於用戶將要做出的安全決定，許可需要用字符串對其進行簡短的表述。
*   許可字符串必須保證語言的國際化。
*   用戶可能對一個許可感到困惑或者知曉風險而選擇不安裝應用
*   當許可的創造者未安裝的時候，應用可能要求許可。

上面每一個因素都為應用開發者帶來了重要的非技術挑戰，同時也使用戶感到困惑，這也是我們不建議使用危險許可的原因。

## 使用網絡

網絡交易具有很高的安全風險，因為它涉及到傳送私人的數據。人們對移動設備的隱私關注日益加深，特別是當設備進行網絡交易時，因此應用採取最佳方式保護用戶數據安全極為重要。

### 使用IP網絡

Android下的網絡與Linux環境下的差別並不大。主要考慮的是確保對敏感數據採用了適當的協議，比如使用[HTTPS進行網絡傳輸](http://developer.android.com/reference/javax/net/ssl/HttpsURLConnection.html)。我們在任何支持HTTPS的服務器上更願意使用HTTPS而不是HTTP，因為移動設備可能會頻繁連接不安全的網絡，比如公共WiFi熱點。

授權且加密的套接層級別的通信可通過使用[SSLSocket](http://developer.android.com/reference/javax/net/ssl/SSLSocket.html)類輕鬆實現。考慮到Android設備使用WiFi連接不安全網絡的頻率，對於所有應用來說，使用安全網絡是極力鼓勵支持的。

我們發現部分應用使用[localhost](http://en.wikipedia.org/wiki/Localhost)端口處理敏感的IPC。我們不鼓勵這種方法，是因為這些接口可被設備上的其他應用訪問。相反，你應該在可認證的地方使用Android IPC機制，例如[Service](http://developer.android.com/reference/android/app/Service.html)（比使用迴環還糟的是綁定INADDR_ANY，因為你的應用可能收到來自任何地方來的請求，我們也已經見識過了）。

一個有必要重複的常見議題是，確保不信任從HTTP或者其他不安全協議下載的數據。這包括在[WebView](http://developer.android.com/reference/android/webkit/WebView.html)中的輸入驗證和對於http的任何響應。

### 使用電話網絡

SMS協議是Android開發者使用最頻繁的電話協議，主要為用戶與用戶之間的通信設計，但對於想要傳送數據的應用來說並不合適。由於SMS的限制性，我們強烈建議使用[Google Cloud Messaging](http://developer.android.com/google/gcm/index.html)（GCM）和IP網絡從web服務器發送數據消息給用戶設備應用。

很多開發者沒有意識到SMS在網絡上或者設備上是不加密的，也沒有牢固驗證。特別是任何SMS接收者應該預料到惡意用戶也許已經給你的應用發送了SMS：不要指望未驗證的SMS數據執行敏感操作。你也應該注意到SMS在網絡上也許會遭到冒名頂替並且/或者攔截，對於Android設備本身，SMS消息是通過廣播intent傳遞的，所以他們也許會被其他擁有[READ_SMS](http://developer.android.com/reference/android/Manifest.permission.html#READ_SMS)許可的應用截獲。

## 輸入驗證

無論應用運行在什麼平臺上，功能不完善的輸入驗證是最常見的影響應用安全問題之一。Android有平臺級別的對策，用於減少應用的公開輸入驗證問題，你應該在可能的地方使用這些功能。同樣需要注意的是，選擇類型安全的語言能減少輸入驗證問題。

如果你使用native代碼，那麼任何從文件讀取的，通過網絡接收的，或者通過IPC接收的數據都有可能引發安全問題。最常見的問題是[buffer overflows](http://en.wikipedia.org/wiki/Buffer_overflow)，[use after free](http://en.wikipedia.org/wiki/Double_free#Use_after_free)，和[off-by-one](http://en.wikipedia.org/wiki/Off-by-one_error)。Android提供安全機制比如ASLR和DEP以減少這些漏洞的可利用性，但是沒有解決基本的問題。小心處理指針和管理緩存可以預防這些問題。

動態、基於字符串的語言，比如JavaScript和SQL，都常受到由轉義字符和[腳本注入](http://en.wikipedia.org/wiki/Code_injection)帶來的輸入驗證問題。

如果你使用提交到SQL Database或者Content Provider的數據，SQL注入也許是個問題。最好的防禦是使用參數化的查詢，就像ContentProviders中討論的那樣。限制權限為只讀或者只寫可以減少SQL注入的潛在危害。

如果你不能使用上面提到的安全功能，我們強烈建議使用結構嚴謹的數據格式並且驗證符合期望的格式。黑名單策略與替換危險字符是有效的，但這些技術在實踐中是易錯的並且當錯誤可能發生的時候應該儘量避免。

## 處理用戶數據

通常來說，處理用戶數據安全最好的方法是最小化獲取敏感數據用戶個人數據的API使用。如果你對數據進行訪問並且可以避免存儲或傳輸，那就不要存儲和傳輸數據。最後，思考是否有一種應用邏輯可能被實現為使用hash或者不可逆形式的數據。例如，你的應用也許使用一個email地址的hash作為主鍵，避免傳輸或存儲email地址，這減少無意間洩漏數據的機會，並且也能減少攻擊者嘗試利用你的應用的機會。

如果你的應用訪問私人數據，比如密碼或者用戶名，記住司法也許要求你提供一個使用和存儲這些數據的隱私策略的解釋。所以遵守最小化訪問用戶數據最佳的安全實踐也許只是簡單的服從。

你也應該考慮到應用是否會疏忽暴露個人信息給其他方，比如廣告第三方組件或者你應用使用的第三方服務。如果你不知道為什麼一個組件或者服務請求個人信息，那麼就不要提供給它。通常來說，通過減少應用訪問個人信息，會減少這個區域潛在的問題。

如果必須訪問敏感數據，評估這個信息是否必須要傳到服務器，或者是否可以被客戶端操作。考慮客戶端上使用敏感數據運行的任何代碼，避免傳輸用戶數據
確保不會無意間通過過渡自由的IPC、world writable文件、或網絡socket暴露用戶數據給其他設備上的應用。這裡有一個洩漏權限保護數據的特別例子，在[Requesting Permissions](http://developer.android.com/training/articles/security-tips.html#RequestingPermissions)章節中討論。

如果需要GUID，建立一個大的、唯一的數字並保存它。不要使用電話標識，比如與個人信息相關的電話號碼或者IMEI。這個話題在[Android Developer Blog](http://android-developers.blogspot.com/2011/03/identifying-app-installations.html)中有更詳細的討論。

應用開發者應謹慎的把log寫到機器上。在Android中，log是共享資源，一個帶有[READ_LOGS](http://developer.android.com/reference/android/Manifest.permission.html#READ_LOGS)許可的應用可以訪問。即使電話log數據是臨時的並且在重啟之後會擦除，不恰當地記錄用戶信息會無意間洩漏用戶數據給其他應用。

## 使用WebView

因為[WebView](http://developer.android.com/reference/android/webkit/WebView.html)能包含HTML和JavaScript瀏覽網絡內容，不恰當的使用會引入常見的web安全問題，比如[跨站腳本攻擊](http://en.wikipedia.org/wiki/Cross_site_scripting)（JavaScript注入）。Android採取一些機制通過限制WebView的能力到應用請求功能最小化來減少這些潛在的問題。

如果你的應用沒有在WebView內直接使用JavaScript，不要調用<a href="http://developer.android.com/reference/android/webkit/WebSettings.html#setJavaScriptEnabled(boolean)">setJavaScriptEnabled()</a>。某些樣本代碼使用這種方法，可能會導致在產品應用中改變用途：所以如果不需要的話移除它。默認情況下WebView不執行JavaScript，所以跨站腳本攻擊不會產生。

使用<a href="http://developer.android.com/reference/android/webkit/WebView.html#addJavascriptInterface(java.lang.Object, java.lang.String)">addJavaScriptInterface()</a>要特別的小心，因為它允許JavaScript執行通常保留給Android應用的操作。只把<a href="http://developer.android.com/reference/android/webkit/WebView.html#addJavascriptInterface(java.lang.Object, java.lang.String)">addJavaScriptInterface()</a>暴露給可靠的輸入源。如果不受信任的輸入是被允許的，不受信任的JavaScript也許會執行Android方法。總得來說，我們建議只把<a href="http://developer.android.com/reference/android/webkit/WebView.html#addJavascriptInterface(java.lang.Object, java.lang.String)">addJavaScriptInterface()</a>暴露給你應用內包含的JavaScript。

如果你的應用通過WebView訪問敏感數據，你也許想要使用<a href="http://developer.android.com/reference/android/webkit/WebView.html#clearCache(boolean)">clearCache()</a>方法來刪除任何存儲到本地的文件。服務端的header，比如no-cache，能用於指示應用不應該緩存特定的內容。

### 處理證書

通常來說，我們建議請求用戶證書頻率最小化--使得釣魚攻擊更明顯，並且降低其成功的可能。取而代之使用授權令牌然後刷新它。

可能的情況下，用戶名和密碼不應該存儲到設備上，而使用用戶提供的用戶名和密碼執行初始認證，然後使用一個短暫的、特定服務的授權令牌。可以被多個應用訪問的service應該使用[AccountManager](http://developer.android.com/reference/android/accounts/AccountManager.htmls)訪問。
如果可能的話，使用AccountManager類來執行基於雲的服務並且不把密碼存儲到設備上。

使用AccountManager獲取[Account](http://developer.android.com/reference/android/accounts/Account.html)之後，進入任何證書前檢查[CREATOR](http://developer.android.com/reference/android/accounts/Account.html#CREATOR)，這樣你就不會因為疏忽而把證書傳遞給錯誤的應用。

如果證書只是用於你創建的應用，那麼你能使用<a href="http://developer.android.com/reference/android/content/pm/PackageManager.html#checkSignatures(int, int)">checkSignature()</a>驗證訪問AccountManager的應用。或者，如果一個應用要使用證書，你可以使用[KeyStore](http://developer.android.com/reference/java/security/KeyStore.html)來儲存。

## 使用加密

除了採取數據隔離，支持完整的文件系統加密，提供安全信道之外。Android提供大量加密算法來保護數據。

通常來說，嘗試使用最高級別的已存在framework的實現來支持，如果你需要安全的從一個已知的位置取回一個文件，一個簡單的HTTPS URI也許就足夠了，並且這部分不要求任何加密知識。如果你需要一個安全信道，考慮使用[HttpsURLConnection](http://developer.android.com/reference/javax/net/ssl/HttpsURLConnection.html)或者[SSLSocket](http://developer.android.com/reference/javax/net/ssl/SSLSocket.html)要比使用你自己的協議好。

如果你發現的確需要實現一個自定義的協議，我們強烈建議你不要自己實現加密算法。使用已經存在的加密算法，比如[Cipher](http://developer.android.com/reference/javax/crypto/Cipher.html)類中提供的AES或者RSA。

使用一個安全的隨機數生成器（[SecureRandom](http://developer.android.com/reference/java/security/SecureRandom.html)）來初始化加密密鑰（[KeyGenerator](http://developer.android.com/reference/javax/crypto/KeyGenerator.html)）。使用一個不安全隨機數生成器生成的密鑰嚴重削弱算法的優點，而且可能遭到離線攻擊。

如果你需要存儲一個密鑰來重複使用，使用類似於[KeyStore](http://developer.android.com/reference/java/security/KeyStore.html)的機制，來提供長期儲存和檢索加密密鑰的功能。

## 使用進程間通信

一些Android應用試圖使用傳統的Linux技術實現IPC，比如網絡socket和共享文件。我們強烈鼓勵使用Android系統IPC功能，比如[Intent](http://developer.android.com/reference/android/content/Intent.html)，[Binder](http://developer.android.com/reference/android/os/Binder.html)，[Messenger](http://developer.android.com/reference/android/os/Messenger.html)和[BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html)。Android IPC機制允許你為每一個IPC機制驗證連接到你的IPC和設置安全策略的應用的身份。

很多安全元素通過IPC機制共享。Broadcast Receiver, Activitie,和Service都在應用的manifest中聲明。如果你的IPC機制不打算給其他應用使用，設置`android:exported`屬性為false。這對於同一個UID內包含多個進程的應用，或者在開發後期決定不想通過IPC暴露功能並且不想重寫代碼的時候非常有用。

如果你的IPC打算讓別的應用訪問，你可以通過使用Permission標記設置一個安全策略。如果IPC是使用同一個密鑰簽名的獨立的應用間的，使用[signature](http://developer.android.com/guide/topics/manifest/permission-element.html#plevel)更好一些。

### 使用Intent

Intent是Android中異步IPC機制的首選。根據你應用的需求，你也許使用<a href="http://developer.android.com/reference/android/content/Context.html#sendBroadcast(android.content.Intent)">sendBroadcast()</a>，<a href="http://developer.android.com/reference/android/content/Context.html#sendOrderedBroadcast(android.content.Intent, java.lang.String)">sendOrderedBroadcast()</a>或者直接的intent來指定一個應用組件。

注意，有序廣播可以被Receiver接收，所以他們也許不會被髮送到所有的應用中。
如果你要發送一個intent給指定的Receiver，這個intent必須被直接的發送給這個Receiver。

Intent的發送者能在發送的時候驗證Receiver是否有一個許可指定了一個non-Null Permission。只有有那個許可的應用才會收到這個intent。如果廣播intent內的數據是敏感的，你應該考慮使用許可來保證惡意應用沒有恰當的許可無法註冊接收那些消息。這種情況下，可以考慮直接執行這個Receiver而不是發起一個廣播。

> **注意：**Intent過濾器不能作為安全特性--組件可被intent顯式調用，可能會沒有符合intent過濾器的數據。你應該在Intent Receiver內執行輸入驗證，確認對於調用Receiver，Service、或Activity來說格式正確合理。

### 使用服務

[Service](http://developer.android.com/reference/android/app/Service.html)經常被用於為其他應用提供服務。每個service類必須在它的manifest文件進行相應的聲明。

默認情況下，Service不能被導出和被其他應用執行。如果你加入了任何Intent過濾器到服務的聲明中，那麼它默認為可以被導出。最好明確聲明[android:exported](http://developer.android.com/guide/topics/manifest/service-element.html#exported)元素來確定它按照你設想的運行。可以使用[android:permission](http://developer.android.com/guide/topics/manifest/service-element.html#prmsn)保護Service。這樣做，其他應用在他們自己的manifest文件中將需要聲明相應的[<uses-permission>](http://developer.android.com/guide/topics/manifest/uses-permission-element.html)元素來啟動、停止或者綁定到這個Service上。

一個Service可以使用許可保護單獨的IPC調用，在執行調用前通過調用<a href="http://developer.android.com/reference/android/content/Context.html#checkCallingPermission(java.lang.String)">checkCallingPermission()</a>來實現。我們建議使用manifest中聲明的許可，因為那些是不容易監管的。

### 使用binder和messenger接口

在Android中，[Binders](http://developer.android.com/reference/android/os/Binder.html)和[Messenger](http://developer.android.com/reference/android/os/Messenger.html)是RPC風格IPC的首選機制。必要的話，他們提供一個定義明確的接口，促進彼此的端點認證。

我們強烈鼓勵在一定程度上，設計不要求指定許可檢查的接口。Binder和[Messenger](http://developer.android.com/reference/android/os/Messenger.html)不在應用的manifest中聲明，因此你不能直接在Binder上應用聲明的許可。它們在應用的manifest中繼承許可聲明，[Service](http://developer.android.com/reference/android/app/Service.html)或者[Activity](http://developer.android.com/reference/android/app/Activity.html)內實現了許可。如果你打算創建一個接口，在一個指定binder接口上要求認證和/或者訪問控制，這些控制必須在Binder和[Messenger](http://developer.android.com/reference/android/os/Messenger.html)的接口中明確添加代碼。

如果提供一個需要訪問控制的接口，使用<a href="http://developer.android.com/reference/android/content/Context.html#checkCallingPermission(java.lang.String)">checkCallingPermission()</a>來驗證調用者是否擁有必要的許可。由於你的應用的id已經被傳遞到別的接口，因此代表調用者訪問一個Service之前這尤其重要。如果調用一個Service提供的接口，如果你沒有對給定的Service訪問許可，<a href="http://developer.android.com/reference/android/content/Context.html#bindService(android.content.Intent, android.content.ServiceConnection, int)">bindService()</a>請求也許會失敗。如果調用你自己的應用提供的本地接口，使用<a href="http://developer.android.com/reference/android/os/Binder.html#clearCallingIdentity()">clearCallingIdentity()</a>來進行內部安全檢查是有用的。

更多關於用服務運行IPC的信息，參見[Bound Services](http://developer.android.com/guide/components/bound-services.html)

### 利用BroadcastReceiver

[Broadcast receivers](http://developer.android.com/reference/android/content/BroadcastReceiver.html)是用來處理通過[intent](http://developer.android.com/reference/android/content/Intent.html)發起的異步請求。

默認情況下，Receiver是導出的，並且可以被任何其他應用執行。如果你的[BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html)打算讓其他應用使用，你也許想在應用的manifest文件中使用[<receiver>](http://developer.android.com/guide/topics/manifest/receiver-element.html)元素對receiver使用安全許可。這將阻止沒有恰當許可的應用發送intent給這個[BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html)。

## 動態加載代碼

我們不鼓勵從應用文件外加載代碼。考慮到代碼注入或者代碼篡改，這樣做顯著增加了應用暴露的可能，同時也增加了版本管理和應用測試的複雜性。最終可能造成無法驗證應用的行為，因此在某些環境下應該被限制。

如果你的應用確實動態加載了代碼，最重要的事情是記住運行動態加載的代碼與應用具有相同的安全許可。用戶決定安裝你的應用是基於你的id，他們期望你提供任何運行在應用內部的代碼，包括動態加載的代碼。

動態加載代碼主要的風險在於代碼來源於可確認的源頭。
如果這個模塊是之間直接包含在你的應用中，那麼它們不能被其他應用修改，不論代碼是本地庫或者是使用[DexClassLoader](http://developer.android.com/reference/dalvik/system/DexClassLoader.html)加載的類這都是事實。我們見過很多應用實例嘗試從不安全的地方加載代碼，比如從網絡上通過非加密的協議或者從全局可寫的位置（比如外部存儲）下載數據。這些地方會允許網絡上其他人在傳輸過程中修改其內容，或者允許用戶設備上的其他應用修改其內容。

## 在虛擬機器安全性

Dalvik是安卓的運行時虛擬機(VM)。Dalvik是特別為安卓建立的，但許多其他虛擬機相關的安全代碼的也適用於安卓。一般來說，你不應該關心與自己有關的虛擬機的安全問題。你的應用程序在一個安全的沙盒環境下運行，所以系統上的其他進程無法訪問你的代碼或私人數據。

如果你想更深入瞭解虛擬機的安全問題，我們建議您熟悉一些現有文獻的主題。推薦兩個比較流行的資源：

*   [http://www.securingjava.com/toc.html](http://www.securingjava.com/toc.html)
*   [https://www.owasp.org/index.php/Java_Security_Resources](https://www.owasp.org/index.php/Java_Security_Resources)

這個文檔集中於安卓與其他VM環境不同地方。對於有在其他環境下有VM編程經驗開發者來說，這裡有兩個普遍的問題可能對於編寫Android應用來說有些不同：

*    一些虛擬機，比如JVM或者.Net，擔任一個安全的邊界作用，代碼與底層操作系統隔離。在Android上，Dalvik VM不是一個安全邊界：應用沙箱是在系統級別實現的，所以Dalvik可以在同一個應用與native代碼相互操作，沒有任何安全約束。
*    已知的手機上的存儲限制，對來發者來說，想要建立模塊化應用和使用動態類加載是很常見的。要這麼做的時候需要考慮兩個資源：一是在哪裡恢復你的應用邏輯，二是在哪裡存儲它們。不要從未驗證的資源使用動態類加載器，比如不安全的網絡資源或者外部存儲，因為那些代碼可能被修改為包含惡意行為。

## 本地代碼的安全

一般來說，對於大多數應用開發，我們鼓勵開發者使用Android SDK而不是使用[Android NDK]（http://developer.android.com/tools/sdk/ndk/index.html) 的native代碼。編譯native代碼的應用更為複雜，移植性差，更容易包含常見的內存崩潰錯誤，比如緩衝區溢出。

Android使用Linux內核編譯並且與Linux開發相似，如果你打算使用native代碼，安全策略尤其有用。與Linux有關的安全問題超出了本文的討論範圍，但讀者可以參考[Secure Programming for Linux and Unix HOWTO](http://www.dwheeler.com/secure-programs)。

與大多數Linux環境的一個重要區別是應用沙箱。在Android中，所有的應用運行在應用沙箱中，包括用native代碼編寫的應用。在最基本的級別中，與Linux相似，對於開發者來說最好的方式是知道每個應用被分配一個權限非常有限的唯一UID。這裡討論的比[Android Security Overview](http://source.android.com/tech/security/index.html)中更細節化，你應該熟悉應用許可，即使你使用的是native代碼。
