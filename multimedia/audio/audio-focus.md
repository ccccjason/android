# 管理音頻焦點

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/managing-audio/audio-focus.html>

由於可能會有多個應用可以播放音頻，所以我們應當考慮一下他們應該如何交互。為了防止多個音樂播放應用同時播放音頻，Android使用音頻焦點（Audio Focus）來控制音頻的播放——即只有獲取到音頻焦點的應用才能夠播放音頻。

在我們的應用開始播放音頻之前，它需要先請求音頻焦點，然後再獲取到音頻焦點。另外，它還需要知道如何監聽失去音頻焦點的事件並對此做出合適的響應。

<!-- more -->

## 請求獲取音頻焦點(Request the Audio Focus)

在我們的應用開始播放音頻之前，它需要獲取將要使用的音頻流的音頻焦點。通過使用<a href="http://developer.android.com/reference/android/media/AudioManager.html#requestAudioFocus(android.media.AudioManager.OnAudioFocusChangeListener, int, int)">requestAudioFocus()</a> 方法可以獲取我們希望得到的音頻流焦點。如果請求成功，該方法會返回[AUDIOFOCUS_REQUEST_GRANTED](http://developer.android.com/reference/android/media/AudioManager.html#AUDIOFOCUS_REQUEST_GRANTED)。

另外我們必須指定正在使用的音頻流，而且需要確定所請求的音頻焦點是短暫的（Transient）還是永久的（Permanent）。

* 短暫的焦點鎖定：當計劃播放一個短暫的音頻時使用（比如播放導航指示）。
* 永久的焦點鎖定：當計劃播放一個較長但時長可預期的音頻時使用（比如播放音樂）。

下面的代碼片段是一個在播放音樂時請求永久音頻焦點的例子，我們必須在開始播放之前立即請求音頻焦點，比如在用戶點擊播放或者遊戲中下一關的背景音樂開始前。

```java
AudioManager am = mContext.getSystemService(Context.AUDIO_SERVICE);
...

// Request audio focus for playback
int result = am.requestAudioFocus(afChangeListener,
                                 // Use the music stream.
                                 AudioManager.STREAM_MUSIC,
                                 // Request permanent focus.
                                 AudioManager.AUDIOFOCUS_GAIN);
   
if (result == AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
    am.registerMediaButtonEventReceiver(RemoteControlReceiver);
    // Start playback.
}
```

一旦結束了播放，需要確保調用了<a href="http://developer.android.com/reference/android/media/AudioManager.html#abandonAudioFocus(android.media.AudioManager.OnAudioFocusChangeListener)">abandonAudioFocus()</a>方法。這樣相當於告知系統我們不再需要獲取焦點並且註銷所關聯的[AudioManager.OnAudioFocusChangeListener](http://developer.android.com/reference/android/media/AudioManager.OnAudioFocusChangeListener.html)監聽器。對於另一種釋放短暫音頻焦點的情況，這會允許任何被我們打斷的應用可以繼續播放。

```java
// Abandon audio focus when playback complete    
am.abandonAudioFocus(afChangeListener);
```

當請求短暫音頻焦點的時候，我們可以選擇是否開啟“Ducking”。通常情況下，一個應用在失去音頻焦點時會立即關閉它的播放聲音。如果我們選擇在請求短暫音頻焦點的時候開啟了Ducking，那意味著其它應用可以繼續播放，僅僅是在這一刻降低自己的音量，直到重新獲取到音頻焦點後恢復正常音量（譯註：也就是說，不用理會這個短暫焦點的請求，這並不會打斷目前正在播放的音頻。比如在播放音樂的時候突然出現一個短暫的短信提示聲音，此時僅僅是把歌曲的音量暫時調低，使得用戶能夠聽到短信提示聲，在此之後便立馬恢復正常播放）。

```java
// Request audio focus for playback
int result = am.requestAudioFocus(afChangeListener,
                             // Use the music stream.
                             AudioManager.STREAM_MUSIC,
                             // Request permanent focus.
                             AudioManager.AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK);
   
if (result == AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
    // Start playback.
}
```

Ducking對於那些間歇性使用音頻焦點的應用來說特別合適，比如語音導航。

如果有另一個應用像上述那樣請求音頻焦點，它所請求的永久音頻焦點或者短暫音頻焦點（支持Ducking或不支持Ducking），都會被你在請求獲取音頻焦點時所註冊的監聽器接收到。

## 處理失去音頻焦點(Handle the Loss of Audio Focus)

如果應用A請求獲取了音頻焦點，那麼在應用B請求獲取音頻焦點的時候，A獲取到的焦點就會失去。如何響應失去焦點事件，取決於失去焦點的方式。

在音頻焦點的監聽器裡面，當接受到描述焦點改變的事件時會觸發<a href="http://developer.android.com/reference/android/media/AudioManager.OnAudioFocusChangeListener.html#onAudioFocusChange(int)">onAudioFocusChange()</a>回調方法。如之前提到的，獲取焦點有三種類型，我們同樣會有三種失去焦點的類型：永久失去，短暫失去，允許Ducking的短暫失去。

* 失去短暫焦點：通常在失去短暫焦點的情況下，我們會暫停當前音頻的播放或者降低音量，同時需要準備在重新獲取到焦點之後恢復播放。

* 失去永久焦點：假設另外一個應用開始播放音樂，那麼我們的應用就應該有效地將自己停止。在實際場景當中，這意味著停止播放，移除媒體按鈕監聽，允許新的音頻播放器可以唯一地監聽那些按鈕事件，並且放棄自己的音頻焦點。此時，如果想要恢復自己的音頻播放，我們需要等待某種特定用戶行為發生（例如按下了我們應用當中的播放按鈕）。

在下面的代碼片段當中，如果焦點的失去是短暫型的，我們將音頻播放對象暫停，並在重新獲取到焦點後進行恢復。如果是永久型的焦點失去事件，那麼我們的媒體按鈕監聽器會被註銷，並且不再監聽音頻焦點的改變。

```java
OnAudioFocusChangeListener afChangeListener = new OnAudioFocusChangeListener() {
    public void onAudioFocusChange(int focusChange) {
        if (focusChange == AUDIOFOCUS_LOSS_TRANSIENT
            // Pause playback
        } else if (focusChange == AudioManager.AUDIOFOCUS_GAIN) {
            // Resume playback 
        } else if (focusChange == AudioManager.AUDIOFOCUS_LOSS) {
            am.unregisterMediaButtonEventReceiver(RemoteControlReceiver);
            am.abandonAudioFocus(afChangeListener);
            // Stop playback
        }
    }
};
```

在上面失去短暫焦點的例子中，如果允許Ducking，那麼除了暫停當前的播放之外，我們還可以選擇使用“Ducking”。

## Duck! 

在使用Ducking時，正常播放的歌曲會降低音量來凸顯這個短暫的音頻聲音，這樣既讓這個短暫的聲音比較突出，又不至於打斷正常的聲音。

下面的代碼片段讓我們的播放器在暫時失去音頻焦點時降低音量，並在重新獲得音頻焦點之後恢復原來音量。

```java
OnAudioFocusChangeListener afChangeListener = new OnAudioFocusChangeListener() {
    public void onAudioFocusChange(int focusChange) {
        if (focusChange == AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK) {
            // Lower the volume
        } else if (focusChange == AudioManager.AUDIOFOCUS_GAIN) {
            // Raise it back to normal
        }
    }
};
```

音頻焦點的失去是我們需要響應的最重要的事件廣播之一，但除此之外還有很多其他重要的廣播需要我們正確地做出響應。系統會廣播一系列的Intent來向你告知用戶在使用音頻過程當中的各種變化。下節課會演示如何監聽這些廣播並提升用戶的整體體驗。
