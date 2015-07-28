# 兼容音頻輸出設備

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/managing-audio/audio-output.html>

當用戶想要通過Android設備欣賞音樂的時候，他可以有多種選擇，大多數設備擁有內置的揚聲器，有線耳機，也有其它很多設備支持藍牙連接，有些甚至還支持A2DP藍牙音頻傳輸模型協定。（譯註：A2DP全名是Advanced Audio Distribution Profile 藍牙音頻傳輸模型協定! A2DP是能夠採用耳機內的芯片來堆棧數據，達到聲音的高清晰度。有A2DP的耳機就是藍牙立體聲耳機。聲音能達到44.1kHz，一般的耳機只能達到8kHz。如果手機支持藍牙，只要裝載A2DP協議，就能使用A2DP耳機了。還有消費者看到技術參數提到藍牙V1.0 V1.1 V1.2 V2.0 - 這些是指藍牙的技術版本，是指通過藍牙傳輸的速度，他們是否支持A2DP具體要看藍牙產品製造商是否使用這個技術。來自[百度百科](http://baike.baidu.com/view/551149.htm)）

<!-- more -->

## 檢測目前正在使用的硬件設備(Check What Hardware is Being Used)

使用不同的硬件播放聲音會影響到應用的行為。可以使用[AudioManager](http://developer.android.com/reference/android/media/AudioManager.html)來查詢當前音頻是輸出到揚聲器，有線耳機還是藍牙上，如下所示：

```java
if (isBluetoothA2dpOn()) {
    // Adjust output for Bluetooth.
} else if (isSpeakerphoneOn()) {
    // Adjust output for Speakerphone.
} else if (isWiredHeadsetOn()) {
    // Adjust output for headsets
} else { 
    // If audio plays and noone can hear it, is it still playing?
}
```

## 處理音頻輸出設備的改變(Handle Changes in the Audio Output Hardware)

當有線耳機被拔出或者藍牙設備斷開連接的時候，音頻流會自動輸出到內置的揚聲器上。假設播放聲音很大，這個時候突然轉到揚聲器播放會顯得非常嘈雜。

幸運的是，系統會在這種情況下廣播帶有[ACTION_AUDIO_BECOMING_NOISY](http://developer.android.com/reference/android/media/AudioManager.html#ACTION_AUDIO_BECOMING_NOISY)的Intent。無論何時播放音頻，我們都應該註冊一個[BroadcastReceiver](http://developer.android.com/reference/android/content/BroadcastReceiver.html)來監聽這個Intent。在使用音樂播放器時，用戶通常會希望此時能夠暫停當前歌曲的播放。而在遊戲當中，用戶通常會希望可以減低音量。

```java
private class NoisyAudioStreamReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        if (AudioManager.ACTION_AUDIO_BECOMING_NOISY.equals(intent.getAction())) {
            // Pause the playback
        }
    }
}

private IntentFilter intentFilter = new IntentFilter(AudioManager.ACTION_AUDIO_BECOMING_NOISY);

private void startPlayback() {
    registerReceiver(myNoisyAudioStreamReceiver(), intentFilter);
}

private void stopPlayback() {
    unregisterReceiver(myNoisyAudioStreamReceiver);
}
```
