# 使用HTTPS與SSL

> 編寫:[craftsmanBai](https://github.com/craftsmanBai) - <http://z1ng.net> - 原文: <http://developer.android.com/training/articles/security-ssl.html>

SSL，安全套接層([TSL](http://en.wikipedia.org/wiki/Transport_Layer_Security))，是一個常見的用來加密客戶端和服務器通信的模塊。
但是應用程序錯誤地使用SSL可能會導致應用程序的數據在網絡中被惡意攻擊者攔截。為了確保這種情況不在我們的應用中發生，這篇文章主要說明使用網絡安全協議常見的陷阱和使用[Public-Key Infrastructure(PKI)](http://en.wikipedia.org/wiki/Public-key_infrastructure)時一些值得關注的問題。

## 概念

一個典型的SSL使用場景是，服務器配置中包含了一個證書，有匹配的公鑰和私鑰。作為SSL客戶端和服務端握手的一部分，服務端通過使用[public-key cryptography(公鑰加密算法)](http://en.wikipedia.org/wiki/Public-key_cryptography)進行證書籤名來證明它有私鑰。

然而，任何人都可以生成他們自己的證書和私鑰，因此一次簡單的握手不能證明服務端具有匹配證書公鑰的私鑰。一種解決這個問題的方法是讓客戶端擁有一套或者更多可信賴的證書。如果服務端提供的證書不在其中，那麼它將不能得到客戶端的信任。

這種簡單的方法有一些缺陷。服務端應該根據時間升級到強壯的密鑰(key rotation)，更新證書中的公鑰。不幸的是，現在客戶端應用需要根據服務端配置的變化來進行更新。如果服務端不在應用程序開發者的控制下，問題將變得更加麻煩，比如它是一個第三方網絡服務。如果程序需要和任意的服務器進行對話，例如web瀏覽器或者email應用，這種方法也會帶來問題。

為了解決這個問題，服務端通常配置了知名的的發行者證書(稱為[Certificate Authorities(CAs)](http://en.wikipedia.org/wiki/Certificate_authority)。提供的平臺通常包含了一系列知名可信賴的CAs。Android4.2(Jelly Bean)包含了超過100CAs並在每個發行版中更新。和服務端相似的是，一個CA擁有一個證書和一個私鑰。當為一個服務端發佈頒發證書的時候，CA用它的私鑰為服務端簽名。客戶端可以通過服務端擁有被已知平臺CA簽名的證書來確認服務端。

然而，使用CAs又帶來了其他的問題。因為CA為許多服務端證書籤名，你仍然需要其他的方法來確保你對話的是你想要的服務器。為了解決這個問題，使用CA簽名的的證書通過特殊的名字如 gmail.com 或者帶有通配符的域名如 *.google.com 來確認服務端。
下面這個例子會使這些概念具體化一些。[openssl](http://www.openssl.org/docs/apps/openssl.html)工具的客戶端命令關注Wikipedia服務端證書信息。端口為443（默認為HTTPS）。這條命令將open s_client的輸出發送給openssl x509，根據[X.509 standard](http://en.wikipedia.org/wiki/X.509)格式化證書中的內容。特別的是，這條命令需要對象（subject），包含服務端名字和簽發者（issuer）來確認CA。

```
$ openssl s_client -connect wikipedia.org:443 | openssl x509 -noout -subject -issuer
subject= /serialNumber=sOrr2rKpMVP70Z6E9BT5reY008SJEdYv/C=US/O=*.wikipedia.org/OU=GT03314600/OU=See www.rapidssl.com/resources/cps (c)11/OU=Domain Control Validated - RapidSSL(R)/CN=*.wikipedia.org
issuer= /C=US/O=GeoTrust, Inc./CN=RapidSSL CA
```

可以看到由RapidSSL CA頒發給匹配*.wikipedia.org的服務端證書。

## 一個HTTP的例子

假設我們有一個知名CA頒發證書的web服務器，那麼可以使用下面的代碼發送一個安全請求:

```java
URL url = new URL("https://wikipedia.org");
URLConnection urlConnection = url.openConnection();
InputStream in = urlConnection.getInputStream();
copyInputStreamToOutputStream(in, System.out);
```

是的，它就是這麼簡單。如果我們想要修改HTTP的請求，可以把它交付給 [HttpURLConnection](http://developer.android.com/reference/java/net/HttpURLConnection.html)。Android關於[HttpURLConnetcion](http://developer.android.com/reference/java/net/HttpURLConnection.html)文檔中還有更貼切的關於怎樣去處理請求、響應頭、posting的內容、cookies管理、使用代理、獲取responses等例子。但是就這些確認證書和域名的細節而言，Android框架已經通過API為我們考慮到了這些細節。下面是其他需要關注的問題。

## 服務器普通問題的驗證

假設從[getInputStream()](http://developer.android.com/reference/java/net/URLConnection.html#getInputStream()接受內容，會拋出一個異常：

```java
javax.net.ssl.SSLHandshakeException: java.security.cert.CertPathValidatorException: Trust anchor for certification path not found.
        at org.apache.harmony.xnet.provider.jsse.OpenSSLSocketImpl.startHandshake(OpenSSLSocketImpl.java:374)
        at libcore.net.http.HttpConnection.setupSecureSocket(HttpConnection.java:209)
        at libcore.net.http.HttpsURLConnectionImpl$HttpsEngine.makeSslConnection(HttpsURLConnectionImpl.java:478)
        at libcore.net.http.HttpsURLConnectionImpl$HttpsEngine.connect(HttpsURLConnectionImpl.java:433)
        at libcore.net.http.HttpEngine.sendSocketRequest(HttpEngine.java:290)
        at libcore.net.http.HttpEngine.sendRequest(HttpEngine.java:240)
        at libcore.net.http.HttpURLConnectionImpl.getResponse(HttpURLConnectionImpl.java:282)
        at libcore.net.http.HttpURLConnectionImpl.getInputStream(HttpURLConnectionImpl.java:177)
        at libcore.net.http.HttpsURLConnectionImpl.getInputStream(HttpsURLConnectionImpl.java:271)
```

這種情況發生的原因包括：

1.[頒佈證書給服務器的CA不是知名的。](http://developer.android.com/training/articles/security-ssl.html#UnknownCa)

2.[服務器證書不是CA簽名的而是自己簽名的。](http://developer.android.com/training/articles/security-ssl.html#SelfSigned)

3.[服務器配置缺失了中間CA](http://developer.android.com/training/articles/security-ssl.html#MissingCa)

下面將會分別討論當我們和服務器安全連接時如何去解決這些問題。


## 無法識別證書機構

在這種情況中，[SSLHandshakeException](http://developer.android.com/reference/javax/net/ssl/SSLHandshakeException.html)異常產生的原因是我們有一個不被系統信任的CA。可能是我們的證書來源於新CA而不被安卓信任，也可能是應用運行版本較老沒有CA。更多的時候，一個CA不知名是因為它不是公開的CA，而是政府，公司，教育機構等組織私有的。

幸運的是，我們可以讓[HttpsURLConnection](http://developer.android.com/reference/javax/net/ssl/HttpsURLConnection.html)學會信任特殊的CA。過程可能會讓人感到有一些費解，下面這個例子是從[InputStream](http://developer.android.com/reference/java/io/InputStream.html)中獲得特殊的CA，使用它去創建一個密鑰庫，用來創建和初始化[TrustManager](http://developer.android.com/reference/javax/net/ssl/TrustManager.html)。[TrustManager](http://developer.android.com/reference/javax/net/ssl/TrustManager.html)是系統用來驗證服務器證書的，這些證書通過使用[TrustManager](http://developer.android.com/reference/javax/net/ssl/TrustManager.html)信任的CA和密鑰庫中的密鑰創建。
給定一個新的TrustManager，下面這個例子初始化了一個新的[SSLContext](http://developer.android.com/reference/javax/net/ssl/SSLContext.html)，提供了一個[SSLSocketFactory](http://developer.android.com/reference/javax/net/ssl/SSLSocketFactory.html)，我們可以覆蓋來自[HttpsURLConnection](http://developer.android.com/reference/javax/net/ssl/HttpsURLConnection.html)的默認[SSLSocketFactory](http://developer.android.com/reference/javax/net/ssl/SSLSocketFactory.html)。這樣連接時會使用我們的CA來進行證書驗證。

下面是一個華盛頓的大學的組織性的CA的使用例子

```java
// Load CAs from an InputStream
// (could be from a resource or ByteArrayInputStream or ...)
CertificateFactory cf = CertificateFactory.getInstance("X.509");
// From https://www.washington.edu/itconnect/security/ca/load-der.crt
InputStream caInput = new BufferedInputStream(new FileInputStream("load-der.crt"));
Certificate ca;
try {
    ca = cf.generateCertificate(caInput);
    System.out.println("ca=" + ((X509Certificate) ca).getSubjectDN());
} finally {
    caInput.close();
}

// Create a KeyStore containing our trusted CAs
String keyStoreType = KeyStore.getDefaultType();
KeyStore keyStore = KeyStore.getInstance(keyStoreType);
keyStore.load(null, null);
keyStore.setCertificateEntry("ca", ca);

// Create a TrustManager that trusts the CAs in our KeyStore
String tmfAlgorithm = TrustManagerFactory.getDefaultAlgorithm();
TrustManagerFactory tmf = TrustManagerFactory.getInstance(tmfAlgorithm);
tmf.init(keyStore);

// Create an SSLContext that uses our TrustManager
SSLContext context = SSLContext.getInstance("TLS");
context.init(null, tmf.getTrustManagers(), null);

// Tell the URLConnection to use a SocketFactory from our SSLContext
URL url = new URL("https://certs.cac.washington.edu/CAtest/");
HttpsURLConnection urlConnection =
    (HttpsURLConnection)url.openConnection();
urlConnection.setSSLSocketFactory(context.getSocketFactory());
InputStream in = urlConnection.getInputStream();
copyInputStreamToOutputStream(in, System.out);
```

使用一個常用的瞭解你CA的TrustManager，系統可以確認你的服務器證書來自於一個可信任的發行者。

> **注意：**許多網站會提供一個可選解決方案：即讓用戶安裝一個無用的TrustManager。如果你這樣做還不如不加密通訊過程，因為任何人都可以在公共wifi熱點下，使用偽裝成你的服務器的代理髮送你的用戶流量，進行DNS欺騙，來攻擊你的用戶。然後攻擊者便可記錄用戶密碼和其他個人資料。這種方式可以奏效的原因是因為攻擊者可以生成一個證書，並且缺少可以驗證該證書是否來自受信任的來源的TrustManager。你的應用可以同任何人會話。所以不要這樣做，即使是暫時性的也不行。除非你能始終讓你的應用信任服務器證書的簽發者。

## 自簽名服務器證書

第二種[SSLHandshakeException](http://developer.android.com/reference/javax/net/ssl/SSLHandshakeException.html)取決於自簽名證書，意味著服務器就是它自己的CA。這同未知證書權威機構類似，因此你同樣可以用前面部分中提到的方法。

你可以創建自己的TrustManager，這一次直接信任服務器證書。將應用於證書直接捆綁會有一些缺點，不過我們依然可以確保其安全性。我們應該小心確保我們的自簽名證書擁有合適的強密鑰。到2012年，一個65537指數位且一年到期的2048位RSA簽名是合理的。當輪換密鑰時，我們應該查看權威機構（比如[NIST](http://www.nist.gov/)）的建議（[recommendation](http://csrc.nist.gov/groups/ST/key_mgmt/index.html)）來了解哪種密鑰是合適的。


## 缺少中間證書頒發機構

第三種SSLHandshakeException情況的產生於缺少中間CA。大多數公開的CA不直接給服務器簽名。相反，他們使用它們主要的機構（簡稱根認證機構）證書來給中間認證機構簽名，根認證機構這樣做，可以離線存儲減少危險。然而，像安卓等操作系統通常只直接信任根認證機構，在服務器證書（由中間證書頒發機構簽名）和證書驗證者（只知道根認證機構）之間留下了一個缺口。為了解決這個問題，服務器並不在SSL握手的過程中只向客戶端發送它的證書，而是一系列的從服務器到必經的任何中間機構到達根認證機構的證書。

下面是一個 mail.google.com證書鏈，以openssls_client命令顯示：

```
$ openssl s_client -connect mail.google.com:443
---
Certificate chain
 0 s:/C=US/ST=California/L=Mountain View/O=Google Inc/CN=mail.google.com
   i:/C=ZA/O=Thawte Consulting (Pty) Ltd./CN=Thawte SGC CA
 1 s:/C=ZA/O=Thawte Consulting (Pty) Ltd./CN=Thawte SGC CA
   i:/C=US/O=VeriSign, Inc./OU=Class 3 Public Primary Certification Authority
---
```
這裡顯示了一臺服務器發送了一個由Thawte SGC CA為mail.google.com頒發的證書，Thawte SGC CA是一箇中間證書頒發機構，Thawte SGC CA的證書由被安卓信任的Verisign CA頒發。
然而，配置一臺服務器不包括中間證書機構是不常見的。例如，一臺服務器導致安卓瀏覽器的錯誤和應用的異常:


```
$ openssl s_client -connect egov.uscis.gov:443
---
Certificate chain
 0 s:/C=US/ST=District Of Columbia/L=Washington/O=U.S. Department of Homeland Security/OU=United States Citizenship and Immigration Services/OU=Terms of use at www.verisign.com/rpa (c)05/CN=egov.uscis.gov
   i:/C=US/O=VeriSign, Inc./OU=VeriSign Trust Network/OU=Terms of use at https://www.verisign.com/rpa (c)10/CN=VeriSign Class 3 International Server CA - G3
---
```
更有趣的是，用大多數桌面瀏覽器訪問這臺服務器不會導致類似於完全未知CA的或者自簽名的服務器證書導致的錯誤。這是因為大多數桌面瀏覽器緩存隨著時間的推移信任中間證書機構。一旦瀏覽器訪問並且從一個網站了解到的一箇中間證書機構，下一次它將不需要中間證書機構包含證書鏈。

一些站點會有意讓用來提供資源服務的二級服務器像上述所述的那樣。比如，他們可能會讓他們的主HTML頁面用一臺擁有全部證書鏈的服務器來提供，但是像圖片，CSS，或者JavaScript等這樣的資源用不包含CA的服務器來提供，以此節省帶寬。不幸的是，有時這些服務器可能會提供一個在應用中調用的web服務。
這裡有兩種解決這些問題的方法：

*	配置服務器使它包含服務器鏈中的中間證書頒發機構

*	或者，像對待不知名的CA一樣對待中間CA，並且創建一個TrustManager來直接信任它，就像在前兩節中做的那樣。


## 驗證主機名常見問題

就像在文章開頭提到的那樣，有兩個關鍵的部分來確認SSL的連接。第一個是確認證書來源於信任的源，這也是前一個部分關注的焦點。這一部分關注第二部分：確保你當前對話的服務器有正確的證書。當情況不是這樣時，你可能會看到這樣的典型錯誤：

```java
java.io.IOException: Hostname 'example.com' was not verified
        at libcore.net.http.HttpConnection.verifySecureSocketHostname(HttpConnection.java:223)
        at libcore.net.http.HttpsURLConnectionImpl$HttpsEngine.connect(HttpsURLConnectionImpl.java:446)
        at libcore.net.http.HttpEngine.sendSocketRequest(HttpEngine.java:290)
        at libcore.net.http.HttpEngine.sendRequest(HttpEngine.java:240)
        at libcore.net.http.HttpURLConnectionImpl.getResponse(HttpURLConnectionImpl.java:282)
        at libcore.net.http.HttpURLConnectionImpl.getInputStream(HttpURLConnectionImpl.java:177)
        at libcore.net.http.HttpsURLConnectionImpl.getInputStream(HttpsURLConnectionImpl.java:271)

```
服務器配置錯誤可能會導致這種情況發生。服務器配置了一個證書，這個證書沒有匹配的你想連接的服務器的subject或者subject可選的命名域。一個證書被許多不同的服務器使用是可能的。比如，使用 [openssl](http://www.openssl.org/docs/apps/openssl.html) s_client -connect google.com:443 |openssl x509 -text 查看google證書，你可以看到一個subject支持 *google.con *.youtube.com, *.android.com或者其他的。這種錯誤只會發生在你所連接的服務器名稱沒有被證書列為可接受。

不幸的是另外一種原因也會導致這種情況發生：[虛擬化服務](http://en.wikipedia.org/wiki/Virtual_hosting)。當用HTTP同時擁有一個以上主機名的服務器共享時，web服務器可以從 HTTP/1.1請求中找到客戶端需要的目標主機名。不行的是，使用HTTPS會使情況變得複雜，因為服務器必須知道在發現HTTP請求前返回哪一個證書。為了解決這個問題，新版本的SSL，特別是TLSV.1.0和之後的版本，支持[服務器名指示(SNI)](http://en.wikipedia.org/wiki/Server_Name_Indication)，允許SSL客戶端為服務端指定目標主機名，從而返回正確的證書。
幸運的是，從安卓2.3開始，[HttpsURLConnection](http://developer.android.com/reference/javax/net/ssl/HttpsURLConnection.html)支持SNI。不幸的是，Apache HTTP客戶端不這樣，這也是我們不鼓勵用它的原因之一。如果你需要支持安卓2.2或者更老的版本或者Apache HTTP客戶端，一個解決方法是建立一個可選的虛擬化服務並且使用特別的端口，這樣服務端就能夠清楚該返回哪一個證書。


採用不使用你的虛擬服務的主機名[HostnameVerifier](http://developer.android.com/reference/javax/net/ssl/HostnameVerifier.html)而不是服務器默認的來替換，是很重要的選擇。

注意：替換[HostnameVerifier](http://developer.android.com/reference/javax/net/ssl/HostnameVerifier.html)可能會非常危險，如果另外一個虛擬服務不在你的控制下，中間人攻擊可能會直接使流量到達另外一臺服務器而超出你的預想。
如果你仍然確定你想覆蓋主機名驗證，這裡有一個為單[URLConnection](http://developer.android.com/reference/java/net/URLConnection.html)替換驗證過程的例子：



```java
// Create an HostnameVerifier that hardwires the expected hostname.
// Note that is different than the URL's hostname:
// example.com versus example.org
HostnameVerifier hostnameVerifier = new HostnameVerifier() {
    @Override
    public boolean verify(String hostname, SSLSession session) {
        HostnameVerifier hv =
            HttpsURLConnection.getDefaultHostnameVerifier();
        return hv.verify("example.com", session);
    }
};

// Tell the URLConnection to use our HostnameVerifier
URL url = new URL("https://example.org/");
HttpsURLConnection urlConnection =
    (HttpsURLConnection)url.openConnection();
urlConnection.setHostnameVerifier(hostnameVerifier);
InputStream in = urlConnection.getInputStream();
copyInputStreamToOutputStream(in, System.out);
```
但是請記住，如果你發現你在替換主機名驗證，特別是虛擬服務，另外一個虛擬主機不在你的控制的情況是非常危險的，你應該找到一個避免這種問題產生的託管管理。

## 關於直接使用SSL Socket的警告

到目前為止，這些例子聚焦於使用HttpsURLConnection上。有時一些應用需要讓SSL和HTTP分開。舉個例子，一個email應用可能會使用SSL的變種，SMTP,POP3,IMAP等。在那些例子中，應用程序會想使用[SSLSocket](http://developer.android.com/reference/javax/net/ssl/SSLSocket.html)直接連接，與HttpsURLConnection做的方法相似。
這種技術到目前為止處理了證書驗證問題，也應用於SSLSocket中。事實上，當使用常規的TrustManager時，傳遞給HttpsURLConnection的是SSLSocketFactory。如果你需要一個帶常規的SSLSocket的TrustManager，採取下面的步驟使用SSLSocketFactory來創建你的SSLSocket。

> **注意：** SSLSocket不具有主機名驗證功能。它取決於它自己的主機名驗證，通過傳入預期的主機名調用[getDefaultHostNameVerifier()](http://developer.android.com/reference/javax/net/ssl/HttpsURLConnection.html#getDefaultHostnameVerifier())。進一步需要注意的是，當發生錯誤時，<a href="http://developer.android.com/reference/javax/net/ssl/HostnameVerifier.html#verify(java.lang.String, javax.net.ssl.SSLSession">HostnameVerifier.verify()</a>不知道拋出異常，而是返回一個布爾值，你需要進一步明確的檢查。
下面是一個演示的方法。這個例子演示了當它連接gmail.com 443端口並且沒有SNI支持的時候，你將會收到一個mail.google.com的證書。你需要確保證書的確是mail.google.com的。


```java
// Open SSLSocket directly to gmail.com
SocketFactory sf = SSLSocketFactory.getDefault();
SSLSocket socket = (SSLSocket) sf.createSocket("gmail.com", 443);
HostnameVerifier hv = HttpsURLConnection.getDefaultHostnameVerifier();
SSLSession s = socket.getSession();

// Verify that the certicate hostname is for mail.google.com
// This is due to lack of SNI support in the current SSLSocket.
if (!hv.verify("mail.google.com", s)) {
    throw new SSLHandshakeException("Expected mail.google.com, "
                                    "found " + s.getPeerPrincipal());
}

// At this point SSLSocket performed certificate verificaiton and
// we have performed hostname verification, so it is safe to proceed.

// ... use socket ...
socket.close();
```
## 黑名單

SSL 主要依靠CA來確認證書來自正確無誤服務器和域名的所有者。少數情況下，CA被欺騙，或者在[Comodo](http://en.wikipedia.org/wiki/Comodo_Group#Breach_of_security)和[DigiNotar](http://en.wikipedia.org/wiki/DigiNotar)的例子中，一個主機名的證書被頒發給了除了服務器和域名的擁有者之外的人，導致被破壞。

為了減少這種危險，安卓可以將一些黑名單或者整個CA列入黑名單。儘管名單是以前是嵌入操作系統的，從安卓4.2開始，這個名單在以後的方案中可以遠程更新。

## 阻塞

一個應用可以通過阻塞技術保護它自己免於受虛假證書的欺騙。這是簡單運用使用未知CA的例子，限制應用信任的CA僅來自被應用使用的服務器。阻止了來自系統中另外一百多個CA的欺騙而導致的應用安全通道的破壞。

## 客戶端驗證

這篇文章聚焦在SSL的使用者同服務器的安全對話上。SSL也支持服務端通過驗證客戶端的證書來確認客戶端的身份。這種技術也與TrustManager的特性相似。可以參考在[HttpsURLConnection](http://developer.android.com/reference/javax/net/ssl/HttpsURLConnection.html)文檔中關於創建一個常規的[KeyManager](http://developer.android.com/reference/javax/net/ssl/KeyManager.html)的討論。


## nogotofail：網絡流量安全測試工具

對於已知的TLS／SSL漏洞和錯誤，nogotofail提供了一個簡單的方法來確認你的應用程序是安全的。它是一個自動化的、強大的、用於測試網絡的安全問題可擴展性的工具，任何設備的網絡流量都可以通過它。
nogotofail主要應用於三種場景：

*	發現錯誤和漏洞。

*	驗證修補程序和等待迴歸。

*	瞭解應用程序和設備產生的交通。

nogotofail 可以工作在Android，iOS，Linux，Windows，Chrome OS，OSX環境下，事實上任何需要連接到Internet的設備都可以。Android和Linux環境下有簡單易用獲取通知的客戶端配置設置，以及本身可以作為靶機，部署為一個路由器，VPN服務器，或代理。
你可以在nogotofail開源項目訪問該工具。

