# 使用Google Cloud Messaging（已廢棄）

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/cloudsync/gcm.html>

谷歌雲消息（GCM）是一個用來給Android設備發送消息的免費服務，它可以極大地提升用戶體驗。利用GCM消息，你的應用可以一直保持更新的狀態，同時不會使你的設備在服務器端沒有可用更新時，喚醒無線電並對服務器發起輪詢（這會消耗大量的電量）。同時，GCM可以讓你最多一次性將一條消息發送給1,000個人，使得你可以在恰當地時機很輕鬆地聯繫大量的用戶，同時大量地減輕你的服務器負擔。

這節課將包含一些把GCM集成到應用中的最佳實踐方法，前提是假定你已經對該服務的基本實現有了一個瞭解。如果不是這樣的話，你可以先閱讀一下：[GCM demo app tutorial](http://developer.android.com/google/gcm/demo.html)。

## 高效地發送多播消息

一個GCM最有用的特性之一是單條消息最多可以發送給1,000個接收者。這個功能可以更加簡單地將重要消息發送給你的所有用戶群體。例如，比方說你有一條消息需要發送給1,000,000個人，而你的服務器每秒能發送500條消息。如果你的每條消息只能發送給一個接收者，那麼整個消息發送過程將會耗時1,000,000/500=2,000秒，大約半小時。然而，如果一條消息可以一次性地發送給1,000個人的話，那麼耗時將會是(1,000,000/1,000)/500=2秒。這不僅僅體現出了GCM的實用性，同時對於一些實時消息而言，其重要性也是不言而喻的。就比如災難預警或者體育比分播報，如果延遲了30分鐘，消息的價值就大打折扣了。

想要利用這一功能非常簡單。如果你使用的是Java語言版本的[GCM helper library](http://developer.android.com/google/gcm/gs.html#libs)，只需要向`send`或者`sendNoRetry`方法提供一個註冊ID的List就行了（不要只給單個的註冊ID）：

```java
// This method name is completely fabricated, but you get the idea.
List regIds = whoShouldISendThisTo(message);

// If you want the SDK to automatically retry a certain number of times, use the
// standard send method.
MulticastResult result = sender.send(message, regIds, 5);

// Otherwise, use sendNoRetry.
MulticastResult result = sender.sendNoRetry(message, regIds);
```

如果想用除了Java之外的語言實現GCM支持，可以構建一個帶有下列頭部信息的HTTP POST請求：

```
Authorization: key=YOUR_API_KEY
Content-type: application/json
```

之後將你想要使用的參數編碼成一個JSON對象，列出所有在`registration_ids`這個Key下的註冊ID。下面的代碼片段是一個例子。除了`registration_ids`之外的所有參數都是可選的，在`data`內的項目代表了用戶定義的載荷數據，而非GCM定義的參數。這個HTTP POST消息將會發送到：`https://android.googleapis.com/gcm/send`：

```
{ "collapse_key": "score_update",
   "time_to_live": 108,
   "delay_while_idle": true,
   "data": {
       "score": "4 x 8",
       "time": "15:16.2342"
   },
   "registration_ids":["4", "8", "15", "16", "23", "42"]
}
```
關於更多GCM多播消息的格式，可以閱讀：[Sending Messages](http://developer.android.com/google/gcm/gcm.html#send-msg)。

## 對可替換的消息執行摺疊

GCM經常被用作為一個觸發器，它告訴移動應用向服務器發起鏈接並更新數據。在GCM中，可以（也推薦）在新消息要替代舊消息時，使用可摺疊的消息（Collapsible Messages）。我們用體育比賽作為例子，如果你向所有用戶發送了一條包含了當前比賽比分的消息，15分鐘之後，又發送了一條消息更新比分，那麼第一條消息就沒有意義了。對於那些還沒有收到第一條消息的用戶，就沒有必要將這兩條消息全部接收下來，何況如果要接收兩條消息，那麼設備不得不進行兩次響應（比如對用戶發出通知或警告），但實際上兩條消息中只有一條是重要的。

當你定義了一個摺疊Key，此時如果有多個消息在GCM服務器中，以隊列的形式等待發送給同一個用戶，那麼只有最後的那一條消息會被髮出。對於之前所說的體育比分的例子，這樣做能讓設備免於處理不必要的任務，也不會讓設備對用戶造成太多打擾。對於其他的一些場景比如與服務器同步數據（檢查郵件接收），這樣做的話可以減少設備需要執行同步的次數。例如，如果有10封郵件在服務器中等待被接收，並且有10條GCM消息發送到設備提醒它有新的郵件，那麼實際上只需要一個GCM就夠了，因為設備可以一次性把10封郵件都同步了。

為了使用這一特性，只需要在你要發出的消息中添加一個消息摺疊Key。如果你在使用[GCM helper library](http://developer.android.com/google/gcm/gs.html#libs)，那麼就使用Message類的`collapseKey(String key)`方法。

```java
Message message = new Message.Builder(regId)
    .collapseKey("game4_scores") // The key for game 4.
    .ttl(600) // Time in seconds to keep message queued if device offline.
    .delayWhileIdle(true) // Wait for device to become active before sending.
    .addPayload("key1", "value1")
    .addPayload("key2", "value2")
    .build();
```

如果你沒有使用[GCM helper library](http://developer.android.com/google/gcm/gs.html#libs)，那麼就直接在你要構建的POST頭部中添加一個字段。將`collapse_key`作為字段名，並將Key的名稱作為該字段的值。

## 在GCM消息中嵌入數據


通常， GCM消息被用作為一個觸發器，或者用來告訴設備，在服務器或者別的地方有一些待更新的數據。然而，一條GCM消息的大小最大可以有4kb，因此，有時候可以在GCM消息中放置一些簡單的數據，這樣的話設備就不需要再去和服務器發起連接了。在下列條件都滿足的情況下，我們可以將數據放置在GCM消息中：

* 數據的總大小在4kb以內。
* 每一條消息都很重要，且需要保留。
* 這些消息不適用於消息摺疊的使用情形。

例如，短消息或者回合制網遊中玩家的移動數據等都是將數據直接嵌入在GCM消息中的例子。而電子郵件就是反面例子了，因為電子郵件的數據量一般都大於4kb，而且用戶一般不需要對每一封新郵件都收到一個GCM提醒的消息。

同時在發送多播消息時，也可以考慮這一方法，這樣的話就不會導致大量用戶在接收到GCM的更新提醒後，同時向你的服務器發起連接。

這一策略不適用於發送大量的數據，有這麼一些原因：

* 為了防止惡意軟件發送垃圾消息，GCM有發送頻率的限制。
* 無法保證消息按照既定的發送順序到達。
* 無法保證消息可以在你發送後立即到達。假設設備每一秒都接收一條消息，消息的大小限制在1K，那麼傳輸速率為8kbps，或者說是1990年代的家庭撥號上網的速度。那麼如此大量的消息，一定會讓你的應用在Google Play上的評分非常尷尬。

如果恰當地使用，直接將數據嵌入到GCM消息中，可以加速你的應用的“感知速度”，因為這樣一來它就不必再去服務器獲取數據了。

## 智能地響應GCM消息

你的應用不應該僅僅對收到的GCM消息進行響應就夠了，還應該響應地更智能一些。至於如何響應需要結合具體情況而定。

**不要太過激進**

當提醒用戶去更新數據時，很容易不小心從“有用的消息”變成“干擾消息”。如果你的應用使用狀態欄通知，那麼應該[更新現有的通知](http://developer.android.com/guide/topics/ui/notifiers/notifications.html#Updating)，而不是創建第二個。如果你通過鈴聲或者震動的方式提醒用戶，一定要設置一個計時器。不要讓應用每分鐘的提醒頻率超過1次，不然的話用戶很可能會不堪其擾而卸載你的應用，關機，甚至把手機扔到河裡。

**用聰明的辦法同步數據，別用笨辦法**

當使用GCM告知設備有數據需要從服務器下載時，記住你有4kb大小的數據可以和消息一起發出，這可以幫助你的應用做出更智能的響應。例如，如果你有一個支持訂閱的閱讀應用，而你的用戶訂閱了100個源，那麼這就可以幫助你的應用更智能地決定應該去服務器下載什麼數據。下面的例子說明了在GCM載荷中可以發送什麼樣的數據，以及設備可以做出什麼樣的反應：

* `refresh` - 你的應用被告知向每一個源請求數據。此時你的應用可以向100個不同的服務器發起獲取訂閱內容的請求，或者如果你在服務器上有一個聚合服務，那麼可以只發送一個請求，將100個源的數據進行打包並讓設備獲取，這樣一次性就完成更新。
* `refresh, freshID` - 一種更好的解決方案，你的應用可以有針對性的完成更新。
* `refresh, freshID, timestamp` - 三種方案中最好的，如果正好用戶在收到GCM消息之前手動做了更新，那麼應用可以利用時間戳和當前的更新時間進行對比，並決定是否有必要執行下一步的行動。
