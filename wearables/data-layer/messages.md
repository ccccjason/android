# 發送與接收消息

> 編寫:[wly2014](https://github.com/wly2014) - 原文: <http://developer.android.com/training/wearables/data-layer/messages.html>

使用[MessageApi](MessageApi.html)發送消息，要附加以下幾項：

* 任一payload（可選）
* 唯一標識消息動作的路徑

不像數據元，Messages（消息）在手持和可穿戴應用之間沒有同步。Messages是單向交流機制，這有利於遠程進程調用(RPC)，比如：發送消息到可穿戴設備以開啟activity。

多個可穿戴設備可以連接到一臺用戶的手持設備。在網絡中每個已連接的設備被視為一個*節點*（*node*）。由於有多個已連接的設備，我們必須考慮哪個節點收到消息。例如，在一個在可穿戴設備上接收語音數據的語音轉錄應用中，我們應該發送消息到一個具有處理能力和電池容量的節點來處理請求，例如一個手持式設備。

> **Note:** Google Play services 7.3.0版之前，一次只有一個可穿戴設備可以連接到手持設備。我們需要將現有的代碼升級，以考慮到多個連接節點的功能。如果我們不作出修改，那麼我們的消息可能不會傳到想要的設備。

## 發送消息

一個可穿戴應用可以為用戶提供如語音轉錄等功能。用戶可以對著他們可穿戴設備的麥克風說話，然後就會將語音保存成一個筆記。由於一個可穿戴設備通常沒有足夠的處理能力和電池容量來處理語音轉錄activity，所以應用應該將這個工作留給一個更加有能力的、已連接的設備來處理。

下面幾個小節介紹如何通知那些可以處理activity請求的設備節點，發現有能力滿足請求的節點，併發送消息給那些節點。

### 通知節點功能

使用 [MessageApi](http://developer.android.com/reference/com/google/android/gms/wearable/MessageApi.html) 類發送請求，來從一個可穿戴設備啟動一個手持設備的activity。由於一個手持式設備可以連接多個可穿戴設備，所以可穿戴應用需要確定一個已連接的節點是否有能力啟動activity。在我們的手持式應用中，通知其它節點：我們的手持式應用所在的節點提供了上述指定的功能。

為了把我們的手持式應用的功能通知其它節點，需要：

1. 在工程的 `res/values/` 目錄下創建一個名為 `wear.xml` 的 XML 文件。
2. 在 `wear.xml` 文件中添加一個名為 `android_wear_capabilities` 的資源。
3. 定義設備可以提供的功能。

> **Note:** 功能是我們自定義的字符串，它在我們的應用中必須是唯一的。

下面這個例子介紹瞭如何將一個名為 `voice_transcription` 的功能添加到 `wear.xml`中：

```xml
<resources>
    <string-array name="android_wear_capabilities">
        <item>voice_transcription</item>
    </string-array>
</resources>
```

### 檢索具有相關功能的節點

首先，我們可以通過調用 <a href="http://developer.android.com/reference/com/google/android/gms/wearable/CapabilityApi.html#getCapability(com.google.android.gms.common.api.GoogleApiClient, java.lang.String, int)">CapabilityApi.getCapability()</a> 方法來檢測具有相關功能的節點。下面的例子介紹瞭如何手動檢索具有 `voice_transcription` 功能的節點：

```java
private static final String
        VOICE_TRANSCRIPTION_CAPABILITY_NAME = "voice_transcription";

private GoogleApiClient mGoogleApiClient;

...

private void setupVoiceTranscription() {
    CapabilityApi.GetCapabilityResult result =
            Wearable.CapabilityApi.getCapability(
                    mGoogleApiClient, VOICE_TRANSCRIPTION_CAPABILITY_NAME,
                    CapabilityApi.FILTER_REACHABLE).await();

    updateTranscriptionCapability(result.getCapability());
}
```

為了在連接到可穿戴設備的時候檢測有能力的節點，註冊一個 [CapabilityApi.CapabilityListener()](http://developer.android.com/reference/com/google/android/gms/wearable/CapabilityApi.CapabilityListener.html) 實例到 [GoogleApiClient](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.html)。下面的例子介紹瞭如何註冊該監聽器和檢索具有 `voice_transcription` 功能的節點。

```java
private void setupVoiceTranscription() {
    ...

    CapabilityApi.CapabilityListener capabilityListener =
            new CapabilityApi.CapabilityListener() {
                @Override
                public void onCapabilityChanged(CapabilityInfo capabilityInfo) {
                    updateTranscriptionCapability(capabilityInfo);
                }
            };

    Wearable.CapabilityApi.addCapabilityListener(
            mGoogleApiClient,
            capabilityListener,
            VOICE_TRANSCRIPTION_CAPABILITY_NAME);
}
```

> **Note:** 如果我們創建一個繼承 [WearableListenerService](http://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService.html) 的 service 來檢測功能的變化，我們可能要重寫 [onConnectedNodes()](http://developer.android.com/reference/com/google/android/gms/wearable/WearableListenerService.html#onConnectedNodes(java.util.List<com.google.android.gms.wearable.Node>)) 方法來監聽細微的連接細節，例如，一個可穿戴設備與手持式設備從Wi-Fi連接切換到藍牙連接。關於一個實現的例子，請查看在 [FindMyPhone](https://github.com/googlesamples/android-FindMyPhone/) 示例中的 `DisconnectListenerService` 類。更多關於如何監聽重要事件的內容，請見[監聽數據層事件](events.html#Listen)。

檢測到有能力的節點之後，需要確定將消息發送到哪裡。我們需要選擇與可穿戴設備鄰近的節點，這樣可以最小化多個節點間的消息路由。一個鄰近的節點被定義為一個直接與設備連接的節點。調用 [Node.isNearby()](http://developer.android.com/reference/com/google/android/gms/wearable/Node.html#isNearby()) 來確定一個節點是否是鄰近的。

下面的例子介紹瞭如何確定最佳節點：

```java
private String transcriptionNodeId = null;

private void updateTranscriptionCapability(CapabilityInfo capabilityInfo) {
    Set<Node> connectedNodes = capabilityInfo.getNodes();

    transcriptionNodeId = pickBestNodeId(connectedNodes);
}

private String pickBestNodeId(Set<Node> nodes) {
    String bestNodeId = null;
    // Find a nearby node or pick one arbitrarily
    for (Node node : nodes) {
        if (node.isNearby()) {
            return node.getId();
         }
         bestNodeId = node.getId();
    }
    return bestNodeId;
}
```

### 傳送消息

一旦我們確定了最佳節點，使用 [MessageApi](http://developer.android.com/reference/com/google/android/gms/wearable/MessageApi.html) 發送消息。

下面的例子介紹瞭如何從一個可穿戴設備發送消息到具有語音轉錄功能的節點。在我們試圖發送消息之前，需要判斷節點是否可用。這個調用是同步的，它在系統將傳送的消息放到隊列前會一直阻塞。

> **Note:** 一個成功結果碼並不保證消息是否傳送成功。如果我們的應用需要數據的可靠性，那麼使用 [DataItem](http://developer.android.com/reference/com/google/android/gms/wearable/DataItem.html) 對象或者 [ChannelApi](http://developer.android.com/reference/com/google/android/gms/wearable/ChannelApi.html) 類在設備間發送數據。

```java
public static final String VOICE_TRANSCRIPTION_MESSAGE_PATH = "/voice_transcription";

private void requestTranscription(byte[] voiceData) {
    if (transcriptionNodeId != null) {
        Wearable.MessageApi.sendMessage(googleApiClient, transcriptionNodeId,
            VOICE_TRANSCRIPTION_MESSAGE_PATH, voiceData).setResultCallback(
                  new ResultCallback() {
                      @Override
                      public void onResult(SendMessageResult sendMessageResult) {
                          if (!sendMessageResult.getStatus().isSuccess()) {
                              // Failed to send message
                          }
                      }
                  }
            );
    } else {
        // Unable to retrieve node with transcription capability
    }
}
```
> **Note:** 閱讀 [Communicate with Google Play Services](http://developer.android.com/google/auth/api-client.html#Communicating) 瞭解更多關於異步和同步調用，以及何時使用哪個。

我們還可以廣播消息給所有已連接的節點。為了獲得我們可以發送消息的已連接節點，需要實現下面的代碼：

```java
private Collection<String> getNodes() {
    HashSet <String>results = new HashSet<String>();
    NodeApi.GetConnectedNodesResult nodes =
            Wearable.NodeApi.getConnectedNodes(mGoogleApiClient).await();
    for (Node node : nodes.getNodes()) {
        results.add(node.getId());
    }
    return results;
}
```

## 接收消息

為了在收到消息時被提醒，我們可以實現 [MessageListener](http://developer.android.com/reference/com/google/android/gms/wearable/MessageApi.MessageListener.html) 接口來提供消息事件的監聽。然後，我們需要在 <a href="http://developer.android.com/reference/com/google/android/gms/wearable/MessageApi.html#addListener(com.google.android.gms.common.api.GoogleApiClient, com.google.android.gms.wearable.MessageApi.MessageListener)">MessageApi.addListener()</a> 方法中註冊監聽。這個例子展示如何通過檢查 `VOICE_TRANSCRIPTION_MESSAGE_PATH` 來實現監聽器。如果該條件是true，就會啟動特定的activity來處理語音數據。

```java
@Override
public void onMessageReceived(MessageEvent messageEvent) {
    if (messageEvent.getPath().equals(VOICE_TRANSCRIPTION_MESSAGE_PATH)) {
        Intent startIntent = new Intent(this, MainActivity.class);
        startIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        startIntent.putExtra("VOICE_DATA", messageEvent.getData());
        startActivity(startIntent);
    }
}
```

這僅是實現更多細節的一小段。關於如何在 service 或 activity 實現完整的監聽，請參見 [監聽數據傳輸層事件](events.html#Listen) 。


