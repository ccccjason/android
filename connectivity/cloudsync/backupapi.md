# 使用備份API

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/cloudsync/backupapi.html>

當用戶購買了一臺新的設備或者是對當前的設備做了恢復出廠設置的操作，用戶會希望在進行初始化設置的時候，Google Play 能夠把之前安裝過的應用恢復到設備上。默認情況是，用戶的這些期望並不會發生，他們之前的設置與數據都會丟失。

對於一些數據量相對較少的情況（通常少於1MB），例如用戶偏好設置、筆記、遊戲分數或者是其他的一些狀態數據，可以使用 Backup API 來提供一個輕量級的解決方案。這節課會介紹如何將 Backup API 集成到我們的應用當中，以及如何利用 Backup API 將數據恢復到新的設備上。

## 註冊 Android Backup Service

這節課中所使用的 [Android Backup Service](http://developer.android.com/google/backup/index.html) 需要進行註冊。我們可以點擊[這裡](http://code.google.com/android/backup/signup.html)進行註冊。註冊成功後，服務器會提供一段類似於下面的代碼，我們需要將它添加到應用的 Manifest 文件中:

<!-- More -->

```xml
<meta-data android:name="com.google.android.backup.api_key"
android:value="ABcDe1FGHij2KlmN3oPQRs4TUvW5xYZ" />
```

請注意，每一個備份 Key 都只能在特定的包名下工作。如果我們有不同的應用需要使用這個方法進行備份，那麼需要分別為他們進行註冊。

## 配置 Manifest 文件

使用 Android 的備份服務需要將兩個額外的內容添加到應用的 Manifest 文件中。首先，聲明備份代理的類名，然後添加一段類似上面的代碼作為 Application 標籤的子標籤。假設我們的備份代理叫作 `TheBackupAgent`，下面的例子展示瞭如何在 Manifest 文件中添加這些信息:

```xml
<application android:label="MyApp"
             android:backupAgent="TheBackupAgent">
    ...
    <meta-data android:name="com.google.android.backup.api_key"
    android:value="ABcDe1FGHij2KlmN3oPQRs4TUvW5xYZ" />
    ...
</application>
```

## 編寫備份代理

創建備份代理最簡單的方法是繼承 [BackupAgentHelper](http://developer.android.com/reference/android/app/backup/BackupAgentHelper.html)。 創建這個幫助類實際上非常簡便。首先創建一個類，其類名和上述 Manifest 文件中聲明的類名一致（本例中，它叫做 `TheBackupAgent`），然後繼承 `BackupAgentHelper`，之後重寫 <a href="http://developer.android.com/reference/android/app/backup/BackupAgent.html#onCreate()">onCreate()</a> 方法。

在 <a href="http://developer.android.com/reference/android/app/backup/BackupAgent.html#onCreate()">onCreate()</a> 中創建一個 [BackupHelper](http://developer.android.com/reference/android/app/backup/BackupHelper.html)。這些幫助類是專門用來備份某些數據的。目前 Android Framework 包含了兩種幫助類：[FileBackupHelper](http://developer.android.com/reference/android/app/backup/FileBackupHelper.html) 與 [SharedPreferencesBackupHelper](http://developer.android.com/reference/android/app/backup/SharedPreferencesBackupHelper.html)。在我們創建一個幫助類並且指向需要備份的數據的時候，僅僅需要使用 <a href="http://developer.android.com/reference/android/app/backup/BackupAgentHelper.html#addHelper(java.lang.String, android.app.backup.BackupHelper)">addHelper()</a> 方法將它們添加到 `BackupAgentHelper` 當中， 之後再增加一個 Key 用來恢復數據。大多數情況下，完整的實現差不多隻需要10行左右的代碼。

下面是一個對高分數據進行備份的例子：

```java
 import android.app.backup.BackupAgentHelper;
 import android.app.backup.FileBackupHelper;


 public class TheBackupAgent extends BackupAgentHelper {
    // The name of the SharedPreferences file
    static final String HIGH_SCORES_FILENAME = "scores";

    // A key to uniquely identify the set of backup data
    static final String FILES_BACKUP_KEY = "myfiles";

    // Allocate a helper and add it to the backup agent
    @Override
    void onCreate() {
        FileBackupHelper helper = new FileBackupHelper(this, HIGH_SCORES_FILENAME);
        addHelper(FILES_BACKUP_KEY, helper);
    }
}
```

為了使得程序更加靈活，[FileBackupHelper](http://developer.android.com/reference/android/app/backup/FileBackupHelper.html) 的構造函數可以帶有任意數量的文件名。我們只需簡單地通過增加一個額外的參數，就能實現同時對最高分文件與遊戲進度文件進行備份，如下所述：

```java
    @Override
    void onCreate() {
        FileBackupHelper helper = new FileBackupHelper(this, HIGH_SCORES_FILENAME, PROGRESS_FILENAME);
        addHelper(FILES_BACKUP_KEY, helper);
    }
```

備份用戶偏好同樣比較簡單。和創建 [FileBackupHelper](http://developer.android.com/reference/android/app/backup/FileBackupHelper.html) 一樣來創建一個 [SharedPreferencesBackupHelper](http://developer.android.com/reference/android/app/backup/SharedPreferencesBackupHelper.html)。在這種情況下, 不是添加文件名到構造函數當中，而是添加被應用所使用的 Shared Preference Groups 的名稱。下面的例子展示的是，如果高分數據是以 Preference 的形式而非文件的形式存儲的，備份代理幫助類應該如何設計：

```java
 import android.app.backup.BackupAgentHelper;
 import android.app.backup.SharedPreferencesBackupHelper;

 public class TheBackupAgent extends BackupAgentHelper {
     // The names of the SharedPreferences groups that the application maintains.  These
     // are the same strings that are passed to getSharedPreferences(String, int).
     static final String PREFS_DISPLAY = "displayprefs";
     static final String PREFS_SCORES = "highscores";

     // An arbitrary string used within the BackupAgentHelper implementation to
     // identify the SharedPreferencesBackupHelper's data.
     static final String MY_PREFS_BACKUP_KEY = "myprefs";

     // Simply allocate a helper and install it
     void onCreate() {
         SharedPreferencesBackupHelper helper =
                 new SharedPreferencesBackupHelper(this, PREFS_DISPLAY, PREFS_SCORES);
         addHelper(MY_PREFS_BACKUP_KEY, helper);
     }
 }
```

雖然我們可以根據喜好增加任意數量的備份幫助類到備份代理幫助類中，但是請記住每一種類型的備份幫助類只需要一個就夠了。一個 [FileBackupHelper](http://developer.android.com/reference/android/app/backup/FileBackupHelper.html) 可以處理所有我們想要備份的文件, 而一個 [SharedPreferencesBackupHelper](http://developer.android.com/reference/android/app/backup/SharedPreferencesBackupHelper.html) 則能夠處理所有我們想要備份的 Shared Preference Groups。

## 請求備份

為了請求一個備份，僅僅需要創建一個 [BackupManager](http://developer.android.com/reference/android/app/backup/BackupManager.html) 實例，然後調用它的 <a href="http://developer.android.com/reference/android/app/backup/BackupManager.html#dataChanged()">dataChanged()</a> 方法即可：

```java
 import android.app.backup.BackupManager;
 ...

 public void requestBackup() {
   BackupManager bm = new BackupManager(this);
   bm.dataChanged();
 }
```

該調用會告知備份管理器即將有數據會被備份到雲端。在之後的某個時間點，備份管理器會執行備份代理的 <a href="http://developer.android.com/reference/android/app/backup/BackupAgent.html#onBackup(android.os.ParcelFileDescriptor, android.app.backup.BackupDataOutput, android.os.ParcelFileDescriptor)">onBackup()</a> 方法。無論任何時候，只要數據發生了改變，我們都可以去調用它，並且不用擔心這樣會增加網絡的負荷。如果我們在備份正式發生之前請求了兩次備份，那麼最終備份操作僅僅會出現一次。

## 恢復備份數據

一般而言，我們不應該手動去請求恢復，而是應該讓應用安裝到設備上的時候自動進行恢復。然而，如果確實有必要手動去觸發恢復，只需要調用 <a href="http://developer.android.com/reference/android/app/backup/BackupManager.html#requestRestore(android.app.backup.RestoreObserver)">requestRestore()</a> 方法就可以了。
