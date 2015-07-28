# 管理音頻播放

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/managing-audio/index.html>

如果我們的應用能夠播放音頻，那麼讓用戶能夠以自己預期的方式控制音頻是很重要的。為了保證良好的用戶體驗，我們應該讓應用能夠管理當前的音頻焦點，因為這樣才能確保多個應用不會在同一時刻一起播放音頻。

在學習本系列課程中，我們將會創建可以對音量按鈕進行響應的應用，該應用會在播放音頻的時候請求獲取音頻焦點，並且在當前音頻焦點被系統或其他應用所改變的時候，做出正確的響應。

## Lessons

* [**控制音量與音頻播放(Controlling Your App’s Volume and Playback)**](volume-playback.html)

  學習如何確保用戶能通過硬件或軟件音量控制器調節應用的音量（通常這些控制器上還具有播放、停止、暫停、跳過以及回放等功能按鍵）。


* [**管理音頻焦點(Managing Audio Focus)**](audio-focus.html)

  由於可能會有多個應用具有播放音頻的功能，考慮他們如何交互非常重要。為了防止多個音樂應用同時播放音頻，Android使用音頻焦點（Audio Focus）來控制音頻的播放。在這節課中可以學習如何請求音頻焦點，監聽音頻焦點的丟失，以及在這種情況發生時應該如何做出響應。


* [**兼容音頻輸出設備(Dealing with Audio Output Hardware)**](audio-output.html)

  音頻有多種輸出設備，在這節課中可以學習如何找出播放音頻的設備，以及處理播放時耳機被拔出的情況。
