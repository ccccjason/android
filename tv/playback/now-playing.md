# 顯示正在播放卡片

> 編寫:[huanglizhuo](https://github.com/huanglizhuo) - 原文:<http://developer.android.com/training/tv/playback/now-playing.html>

TV應用允許用戶在使用其他應用時後臺播放音樂或其他媒體。如果我們的應用程序允許後臺，它必須要為用戶提供返回該應用暫停音樂或切換到一個新的歌曲的方法。 Android框架允許TV應用通過在主屏幕上顯示正在播放卡做到這一點。

正在播放卡片是系統的組建,它可以在推薦的行上顯示正在播放的媒體會話它包括了媒體元數據，如專輯封面，標題和應用程序圖標。當用戶選擇它，系統將打開擁有該會話的應用程序。

這節課將演示如何使用[ MediaSession ](http://developer.android.com/reference/android/media/session/MediaSession.html) 類實現正在播放卡片。

##開啟媒體會話

一個播放應用可以作為[ activity ](http://developer.android.com/guide/components/activities) 或者[ service ](http://developer.android.com/guide/components/services/index.html)運行。[ service ](http://developer.android.com/guide/components/services/index.html)是當[ activity ](http://developer.android.com/guide/components/activities) 結束時依然可以後臺播放的。在這節討論中,媒體播放應用是假設在[ MediaBrowserService ](http://developer.android.com/reference/android/service/media/MediaBrowserService.html)下運行的。

在service的[onCreate()](http://developer.android.com/reference/android/service/media/MediaBrowserService.html#onCreate())方法中創建一個新的[ MediaSession ](http://developer.android.com/reference/android/media/session/MediaSession.html#MediaSession(android.content.Context, java.lang.String)),設置適當的回調函數和標誌,並設置 [MediaBrowserService](http://developer.android.com/reference/android/service/media/MediaBrowserService.html) 令牌。

```xml
mSession = new MediaSession(this, "MusicService");
mSession.setCallback(new MediaSessionCallback());
mSession.setFlags(MediaSession.FLAG_HANDLES_MEDIA_BUTTONS |
        MediaSession.FLAG_HANDLES_TRANSPORT_CONTROLS);

// for the MediaBrowserService
setSessionToken(mSession.getSessionToken());
```

> **注意:**正在播放卡片只有在媒體會話設置了[FLAG_HANDLES_TRANSPORT_CONTROLS](http://developer.android.com/reference/android/media/session/MediaSession.html#FLAG_HANDLES_TRANSPORT_CONTROLS)標誌時在可以顯示。

##顯示正在播放卡片

如果會話是系統最高優先級的會話那麼正在播放卡片將在[setActivity(true)](http://developer.android.com/reference/android/media/session/MediaSession.html#setActive(boolean))調用後顯示。同時我們的應用必須像在[Managing Audio Focus](http://developer.android.com/training/managing-audio/audio-focus/index.html)一節中那樣請求音頻焦點。

```xml
private void handlePlayRequest() {

    tryToGetAudioFocus();

    if (!mSession.isActive()) {
        mSession.setActive(true);
    }
...
```

如果另一個應用發起媒體播放請求並調用[setActivity(false)](http://developer.android.com/reference/android/media/session/MediaSession.html#setActive(boolean))後這個卡片將從主屏上移除。

##更新播放狀態

正如任何媒體的應用程序，在[ MediaSession ](http://developer.android.com/reference/android/media/session/MediaSession.html)中更新播放狀態，使卡片可以顯示當前的元數據，如在下面的例子：

```xml
private void updatePlaybackState() {
    long position = PlaybackState.PLAYBACK_POSITION_UNKNOWN;
    if (mMediaPlayer != null && mMediaPlayer.isPlaying()) {
        position = mMediaPlayer.getCurrentPosition();
    }
    PlaybackState.Builder stateBuilder = new PlaybackState.Builder()
            .setActions(getAvailableActions());
    stateBuilder.setState(mState, position, 1.0f);
    mSession.setPlaybackState(stateBuilder.build());
}
private long getAvailableActions() {
    long actions = PlaybackState.ACTION_PLAY |
            PlaybackState.ACTION_PLAY_FROM_MEDIA_ID |
            PlaybackState.ACTION_PLAY_FROM_SEARCH;
    if (mPlayingQueue == null || mPlayingQueue.isEmpty()) {
        return actions;
    }
    if (mState == PlaybackState.STATE_PLAYING) {
        actions |= PlaybackState.ACTION_PAUSE;
    }
    if (mCurrentIndexOnQueue > 0) {
        actions |= PlaybackState.ACTION_SKIP_TO_PREVIOUS;
    }
    if (mCurrentIndexOnQueue < mPlayingQueue.size() - 1) {
        actions |= PlaybackState.ACTION_SKIP_TO_NEXT;
    }
    return actions;
}
```

##顯示媒體元數據

為當前正在播放通過[setMetadata()](http://developer.android.com/reference/android/media/session/MediaSession.html#setMetadata(android.media.MediaMetadata))方法設置[  MediaMetadata ](http://developer.android.com/reference/android/media/MediaMetadata.html)。.這個方法可以讓我們為正在播放卡提供有關軌道，如標題，副標題，和各種圖標等信息。下面的例子假設我們的播放數據存儲在自定義的MediaData類中。

```xml
private void updateMetadata(MediaData myData) {
    MediaMetadata.Builder metadataBuilder = new MediaMetadata.Builder();
    // To provide most control over how an item is displayed set the
    // display fields in the metadata
    metadataBuilder.putString(MediaMetadata.METADATA_KEY_DISPLAY_TITLE,
            myData.displayTitle);
    metadataBuilder.putString(MediaMetadata.METADATA_KEY_DISPLAY_SUBTITLE,
            myData.displaySubtitle);
    metadataBuilder.putString(MediaMetadata.METADATA_KEY_DISPLAY_ICON_URI,
            myData.artUri);
    // And at minimum the title and artist for legacy support
    metadataBuilder.putString(MediaMetadata.METADATA_KEY_TITLE,
            myData.title);
    metadataBuilder.putString(MediaMetadata.METADATA_KEY_ARTIST,
            myData.artist);
    // A small bitmap for the artwork is also recommended
    metadataBuilder.putString(MediaMetadata.METADATA_KEY_ART,
            myData.artBitmap);
    // Add any other fields you have for your data as well
    mSession.setMetadata(metadataBuilder.build());
}
```

##響應用戶的動作

當用戶選擇正在播放卡片時,系統打開應用並擁有會話。如果我們的應用在[setSessionActivity()](http://developer.android.com/reference/android/media/session/MediaSession.html#setSessionActivity(android.app.PendingIntent))有[PendingIntent](http://developer.android.com/reference/android/app/PendingIntent.html)要傳遞,系統將會像下面演示的那樣開啟activity。如果不是，則系統默認的Intent打開。您指定的活動必須提供播放控制，允許用戶暫停或停止播放。

```xml
Intent intent = new Intent(mContext, MyActivity.class);
    PendingIntent pi = PendingIntent.getActivity(context, 99 /*request code*/,
            intent, PendingIntent.FLAG_UPDATE_CURRENT);
    mSession.setSessionActivity(pi);
```