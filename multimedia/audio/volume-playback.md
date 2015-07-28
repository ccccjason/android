# 控制音量與音頻播放

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/managing-audio/volume-playback.html>

良好的用戶體驗應該是可預期且可控的。如果我們的應用可以播放音頻，那麼顯然我們需要做到能夠通過硬件按鈕，軟件按鈕，藍牙耳麥等來控制音量。
同樣地，我們需要能夠對應用的音頻流進行播放（Play），停止（Stop），暫停（Pause），跳過（Skip），以及回放（Previous）等動作，並且並確保其正確性。

<!-- more -->

## 鑑別使用的是哪個音頻流(Identify Which Audio Stream to Use)

為了創建一個良好的音頻體驗，我們首先需要知道應用會使用到哪些音頻流。Android為播放音樂，鬧鈴，通知鈴，來電聲音，系統聲音，打電話聲音與撥號聲音分別維護了一個獨立的音頻流。這樣做的主要目的是讓用戶能夠單獨地控制不同的種類的音頻。上述音頻種類中，大多數都是被系統限制。例如，除非你的應用需要做替換鬧鐘的鈴聲的操作，不然的話你只能通過[STREAM_MUSIC](http://developer.android.com/reference/android/media/AudioManager.html#STREAM_MUSIC)來播放你的音頻。

## 使用硬件音量鍵來控制應用的音量(Use Hardware Volume Keys to Control Your App’s Audio Volume)

默認情況下，按下音量控制鍵會調節當前被激活的音頻流，如果我們的應用當前沒有播放任何聲音，那麼按下音量鍵會調節響鈴的音量。對於遊戲或者音樂播放器而言，即使是在歌曲之間無聲音的狀態，或是當前遊戲處於無聲的狀態，用戶按下音量鍵的操作通常都意味著他們希望調節遊戲或者音樂的音量。你可能希望通過監聽音量鍵被按下的事件，來調節音頻流的音量。其實我們不必這樣做。Android提供了<a href="http://developer.android.com/reference/android/app/Activity.html#setVolumeControlStream(int)">setVolumeControlStream()</a>方法來直接控制指定的音頻流。在鑑別出應用會使用哪個音頻流之後，我們需要在應用生命週期的早期階段調用該方法，因為該方法只需要在Activity整個生命週期中調用一次，通常，我們可以在負責控制多媒體的[Activity](http://developer.android.com/reference/android/app/Activity.html)或者[Fragment](http://developer.android.com/reference/android/app/Fragment.html)的`onCreate()`方法中調用它。這樣能確保不管應用當前是否可見，音頻控制的功能都能符合用戶的預期。

```java
setVolumeControlStream(AudioManager.STREAM_MUSIC);
```

自此之後，不管目標Activity或Fragment是否可見，按下設備的音量鍵都能夠影響我們指定的音頻流（在這個例子中，音頻流是"music"）。

## 使用硬件的播放控制按鍵來控制應用的音頻播放(Use  Hardware Playback Control Keys to Control Your App’s Audio Playback)

許多線控或者無線耳機都會有許多媒體播放控制按鈕，例如：播放，停止，暫停，跳過，以及回放等。無論用戶按下設備上任意一個控制按鈕，系統都會廣播一個帶有[ACTION_MEDIA_BUTTON](http://developer.android.com/reference/android/content/Intent.html#ACTION_MEDIA_BUTTON)的Intent。為了正確地響應這些操作，需要在Manifest文件中註冊一個針對於該Action的[BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html)，如下所示：

```xml
<receiver android:name=".RemoteControlReceiver">
    <intent-filter>
        <action android:name="android.intent.action.MEDIA_BUTTON" />
    </intent-filter>
</receiver>
```

在Receiver的實現中，需要判斷這個廣播來自於哪一個按鈕，Intent通過[EXTRA_KEY_EVENT](http://developer.android.com/reference/android/content/Intent.html#EXTRA_KEY_EVENT)這一Key包含了該信息，另外，[KeyEvent](http://developer.android.com/reference/android/view/KeyEvent.html)類包含了一系列諸如`KEYCODE_MEDIA_*`的靜態變量來表示不同的媒體按鈕，例如[KEYCODE_MEDIA_PLAY_PAUSE](http://developer.android.com/reference/android/view/KeyEvent.html#KEYCODE_MEDIA_PLAY_PAUSE) 與 [KEYCODE_MEDIA_NEXT](http://developer.android.com/reference/android/view/KeyEvent.html#KEYCODE_MEDIA_NEXT)。

```java
public class RemoteControlReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        if (Intent.ACTION_MEDIA_BUTTON.equals(intent.getAction())) {
            KeyEvent event = (KeyEvent)intent.getParcelableExtra(Intent.EXTRA_KEY_EVENT);
            if (KeyEvent.KEYCODE_MEDIA_PLAY == event.getKeyCode()) {
                // Handle key press.
            }
        }
    }
}
```

因為可能會有多個程序在監聽與媒體按鈕相關的事件，所以我們必須在代碼中控制應用接收相關事件的時機。下面的例子顯示瞭如何使用[AudioManager](http://developer.android.com/reference/android/media/AudioManager.html)來為我們的應用註冊監聽與取消監聽媒體按鈕事件，當Receiver被註冊上時，它將是唯一一個能夠響應媒體按鈕廣播的Receiver。

```java
AudioManager am = mContext.getSystemService(Context.AUDIO_SERVICE);
...

// Start listening for button presses
am.registerMediaButtonEventReceiver(RemoteControlReceiver);
...

// Stop listening for button presses
am.unregisterMediaButtonEventReceiver(RemoteControlReceiver);
```

通常，應用需要在他們失去焦點或者不可見的時候（比如在<a href="http://developer.android.com/reference/android/app/Activity.html#onStop()">onStop()</a>方法裡面）取消註冊監聽。但是對於媒體播放應用來說並沒有那麼簡單，實際上，在應用不可見（不能通過可見的UI控件進行控制）的時候，仍然能夠響應媒體播放按鈕事件是極其重要的。為了實現這一點，有一個更好的方法，我們可以在程序獲取與失去音頻焦點的時候註冊與取消對音頻按鈕事件的監聽。這個內容會在後面的課程中詳細講解。
