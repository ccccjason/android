# 避免出現程序無響應ANR(Keeping Your App Responsive)

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/articles/perf-anr.html>

可能你寫的代碼在性能測試上表現良好，但是你的應用仍然有時候會反應遲緩(sluggish)，停頓(hang)或者長時間卡死(frezze)，或者是應用處理輸入的數據花費時間過長。對於你的應用來說最槽糕的事情是出現"程序無響應(Application Not Responding)" (ANR)的警示框。

在Android中，系統通過顯示ANR警示框來保護程序的長時間無響應。對話框如下：

![anr](anr.png)

此時，你的應用已經經歷過一段時間的無法響應了，因此係統提供用戶可以退出應用的選擇。為你的程序提供良好的響應性是至關重要的，這樣才能夠避免系統為用戶顯示ANR的警示框。

這節課描述了Android系統是如何判斷一個應用不可響應的。這節課還會提供程序編寫的指導原則，確保你的程序保持響應性。

## 是什麼導致了ANR?(What Triggers ANR?)

通常來說，系統會在程序無法響應用戶的輸入事件時顯示ANR。例如，如果一個程序在UI線程執行I/O操作(通常是網絡請求或者是文件讀寫)，這樣系統就無法處理用戶的輸入事件。或者是應用在UI線程花費了太多的時間用來建立一個複雜的在內存中的數據結構，又或者是在一個遊戲程序的UI線程中執行了一個複雜耗時的計算移動的操作。確保那些計算操作高效是很重要的，不過即使是最高效的代碼也是需要花時間執行的。

**對於你的應用中任何可能長時間執行的操作，你都不應該執行在UI線程**。你可以創建一個工作線程，把那些操作都執行在工作線程中。這確保了UI線程(這個線程會負責處理UI事件) 能夠順利執行，也預防了系統因代碼僵死而崩潰。因為UI線程是和類級別相關聯的，你可以把相應性作為一個類級別(class-level)的問題(相比來說，代碼性能則屬於方法級別(method-level)的問題)

在Android中，程序的響應性是由Activity Manager與Window Manager系統服務來負責監控的。當系統監測到下面的條件之一時會顯示ANR的對話框:

* 對輸入事件(例如硬件點擊或者屏幕觸摸事件)，5秒內都無響應。
* BroadReceiver不能夠在10秒內結束接收到任務。

## 如何避免ANRs(How to Avoid ANRs)

Android程序通常是執行在默認的UI線程(也就是main線程)中的。這意味著在UI線程中執行的任何長時間的操作都可能觸發ANR，因為程序沒有給自己處理輸入事件或者broadcast事件的機會。

因此，任何執行在UI線程的方法都應該儘可能的簡短快速。特別是，在activity的生命週期的關鍵方法`onCreate()`與`onResume()`方法中應該儘可能的做比較少的事情。類似網絡或者DB操作等可能長時間執行的操作，或者是類似調整bitmap大小等需要長時間計算的操作，都應該執行在工作線程中。(在DB操作中，可以通過異步的網絡請求)。

為了執行一個長時間的耗時操作而創建一個工作線程最方便高效的方式是使用`AsyncTask`。只需要繼承AsyncTask並實現`doInBackground()`方法來執行任務即可。為了把任務執行的進度呈現給用戶，你可以執行`publishProgress()`方法，這個方法會觸發`onProgressUpdate()`的回調方法。在`onProgressUpdate()`的回調方法中(它執行在UI線程)，你可以執行通知用戶進度的操作，例如：

```java
private class DownloadFilesTask extends AsyncTask<URL, Integer, Long> {
    // Do the long-running work in here
    protected Long doInBackground(URL... urls) {
        int count = urls.length;
        long totalSize = 0;
        for (int i = 0; i < count; i++) {
            totalSize += Downloader.downloadFile(urls[i]);
            publishProgress((int) ((i / (float) count) * 100));
            // Escape early if cancel() is called
            if (isCancelled()) break;
        }
        return totalSize;
    }

    // This is called each time you call publishProgress()
    protected void onProgressUpdate(Integer... progress) {
        setProgressPercent(progress[0]);
    }

    // This is called when doInBackground() is finished
    protected void onPostExecute(Long result) {
        showNotification("Downloaded " + result + " bytes");
    }
}
```

為了能夠執行這個工作線程，只需要創建一個實例並執行`execute()`:

```java
new DownloadFilesTask().execute(url1, url2, url3);
```

相比起AsycnTask來說，創建自己的線程或者HandlerThread稍微複雜一點。如果你想這樣做，**你應該通過`Process.setThreadPriority()`並傳遞`THREAD_PRIORITY_BACKGROUND`來設置線程的優先級為"background"。**如果你不通過這個方式來給線程設置一個低的優先級，那麼這個線程仍然會使得你的應用顯得卡頓，因為這個線程默認與UI線程有著同樣的優先級。

如果你實現了Thread或者HandlerThread，請確保你的UI線程不會因為等待工作線程的某個任務而去執行Thread.wait()或者Thread.sleep()。UI線程不應該去等待工作線程完成某個任務，你的UI線程應該提供一個Handler給其他工作線程，這樣工作線程能夠通過這個Handler在任務結束的時候通知UI線程。使用這樣的方式來設計你的應用程序可以使得你的程序UI線程保持響應性，以此來避免ANR。

BroadcastReceiver有特定執行時間的限制說明了broadcast receivers應該做的是：簡短快速的任務，避免執行費時的操作，例如保存數據或者註冊一個Notification。正如在UI線程中執行的方法一樣，程序應該避免在broadcast receiver中執行費時的長任務。但不是採用通過工作線程來執行復雜的任務的方式，你的程序應該啟動一個IntentService來響應intent broadcast的長時間任務。

> **Tip:** 你可以使用StrictMode來幫助尋找因為不小心加入到UI線程的潛在的長時間執行的操作，例如網絡或者DB相關的任務。

## 增加響應性(Reinforce Responsiveness)

通常來說，100ms - 200ms是用戶能夠察覺到卡頓的上限。這樣的話，下面有一些避免ANR的技巧：

* 如果你的程序需要響應正在後臺加載的任務，在你的UI中可以顯示ProgressBar來顯示進度。
* 對遊戲程序，在工作線程執行計算的任務。
* 如果你的程序在啟動階段有一個耗時的初始化操作，可以考慮顯示一個閃屏，要麼儘快的顯示主界面，然後馬上顯示一個加載的對話框，異步加載數據。無論哪種情況，你都應該顯示一個進度信息，以免用戶感覺程序有卡頓的情況。
* 使用性能測試工具，例如Systrace與Traceview來判斷程序中影響響應性的瓶頸。

