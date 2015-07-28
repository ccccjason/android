# 更新你的Security Provider來對抗SSL漏洞利用

> 編寫:[craftsmanBai](https://github.com/craftsmanBai) - <http://z1ng.net> - 原文: <http://developer.android.com/training/articles/security-gms-provider.html>

安卓依靠security provider保障網絡通信安全。然而有時默認的security provider存在安全漏洞。為了防止這些漏洞被利用，Google Play services 提供了一個自動更新設備的security provider的方法來對抗已知的漏洞。通過調用Google Play services方法，可以確保你的應用運行在可以抵抗已知漏洞的設備上。

舉個例子，OpenSSL的漏洞(CVE-2014-0224)會導致中間人攻擊，在通信雙方不知情的情況下解密流量。Google Play services 5.0提供了一個補丁，但是必須確保應用安裝了這個補丁。通過調用Google Play services方法，可以確保你的應用運行在可抵抗攻擊的安全設備上。

**注意**：更新設備的security provider不是更新[android.net.SSLCertificateSocketFactory](http://developer.android.com/reference/android/net/SSLCertificateSocketFactory.html).比起使用這個類，我們更鼓勵應用開發者使用融入密碼學的高級方法。大多數應用可以使用類似[HttpsURLConnection](http://developer.android.com/reference/javax/net/ssl/HttpsURLConnection.html)，[HttpClient](http://developer.android.com/reference/org/apache/http/client/HttpClient.html)，[AndroidHttpClient](http://developer.android.com/reference/android/net/http/AndroidHttpClient.html)這樣的API，而不必去設置[TrustManager](http://developer.android.com/reference/javax/net/ssl/TrustManager.html)或者創建一個[SSLCertificateSocketFactory](http://developer.android.com/reference/android/net/SSLCertificateSocketFactory.html)。


## 使用ProviderInstaller給Security Provider打補丁

使用providerinstaller類來更新設備的security provider。你可以通過調用該類的方法[installIfNeeded()]()(或者[ installifneededasync]())來驗證security provider是否為最新的(必要的話更新它)。

當你調用[installifneeded]()時，[providerinstaller]()會做以下事情：

*	如果設備的Provider成功更新(或已經是最新的)，該方法返回正常。

*	如果設備的Google Play services 庫已經過時了，這個方法拋出[googleplayservicesrepairableexception]()異常表明無法更新Provider。應用程序可以捕獲這個異常並向用戶彈出合適的對話框提示更新Google Play services。

*	如果產生了不可恢復的錯誤，該方法拋出[googleplayservicesnotavailableexception]()表示它無法更新[Provider]()。應用程序可以捕獲異常並選擇合適的行動，如顯示標準問題解決流程圖。

[installifneededasync]()方法類似，但它不拋出異常，而是通過相應的回調方法，以提示成功或失敗。

如果[installifneeded]()需要安裝一個新的[Provider]()，可能耗費30-50毫秒（較新的設備）到350毫秒（舊設備）。如果security provider已經是最新的，該方法需要的時間量可以忽略不計。為了避免影響用戶體驗：

*	線程加載後立即在後臺網絡線程中調用[installifneeded]()，而不是等待線程嘗試使用網絡。（多次調用該方法沒有害處，如果安全提供程序不需要更新它會立即返回。）

*	如果用戶體驗會受線程阻塞的影響——比如從UI線程中調用，那麼使用[installifneededasync()]()調用該方法的異步版本。（當然，如果你要這樣做，在嘗試任何安全通信之前必須等待操作完成。[providerinstaller]()調用監聽者的[onproviderinstalled()]()方法發出成功信號。

**警告**：如果[providerinstaller]()無法安裝更新Provider，您的設備security provider會容易受到已知漏洞的攻擊。你的程序等同於所有HTTP通信未被加密。
一旦[Provider]()更新，所有安全API（包括SSL API）的調用會經過它(但這並不適用於[android.net.sslcertificatesocketfactory]()，面對[cve-2014-0224]()這種漏洞仍然是脆弱的)。


## 同步修補

修補security provider最簡單的方法就是調用同步方法[installIfNeeded()](http://developer.android.com/reference/com/google/android/gms/security/ProviderInstaller.html##installIfNeeded(android.content.Context).如果用戶體驗不會被線程阻塞影響的話，這種方法很合適。

舉個例子，這裡有一個sync adapter會更新security provider。由於它運行在後臺，因此在等待security provider更新的時候線程阻塞是可以的。sync adapter調用installifneeded()更新security provider。如果返回正常，sync adapter可以確保security provider是最新的。如果返回異常，sync adapter可以採取適當的行動（如提示用戶更新Google Play services）。

```java

/**
 * Sample sync adapter using {@link ProviderInstaller}.
 */
public class SyncAdapter extends AbstractThreadedSyncAdapter {

  ...

  // This is called each time a sync is attempted; this is okay, since the
  // overhead is negligible if the security provider is up-to-date.
  @Override
  public void onPerformSync(Account account, Bundle extras, String authority,
      ContentProviderClient provider, SyncResult syncResult) {
    try {
      ProviderInstaller.installIfNeeded(getContext());
    } catch (GooglePlayServicesRepairableException e) {

      // Indicates that Google Play services is out of date, disabled, etc.

      // Prompt the user to install/update/enable Google Play services.
      GooglePlayServicesUtil.showErrorNotification(
          e.getConnectionStatusCode(), getContext());

      // Notify the SyncManager that a soft error occurred.
      syncResult.stats.numIOExceptions++;
      return;

    } catch (GooglePlayServicesNotAvailableException e) {
      // Indicates a non-recoverable error; the ProviderInstaller is not able
      // to install an up-to-date Provider.

      // Notify the SyncManager that a hard error occurred.
      syncResult.stats.numAuthExceptions++;
      return;
    }

    // If this is reached, you know that the provider was already up-to-date,
    // or was successfully updated.
  }
}

```


### 異步修補

更新security provider可能耗費350毫秒（舊設備）。如果在一個會直接影響用戶體驗的線程中更新，如UI線程，那麼你不會希望進行同步更新，因為這可能導致應用程序或設備凍結直到操作完成。因此你應該使用異步方法[installifneededasync()](http://developer.android.com/reference/com/google/android/gms/security/ProviderInstaller.html#installIfNeededAsync(android.content.Context, com.google.android.gms.security.ProviderInstaller.ProviderInstallListener)。方法通過調用回調函數來反饋其成功或失敗。
例如，下面是一些關於更新security provider在UI線程中的活動的代碼。調用installifneededasync()來更新security provider，並指定自己為監聽器接收成功或失敗的通知。如果security provider是最新的或更新成功，會調用[onproviderinstalled()](http://developer.android.com/reference/com/google/android/gms/security/ProviderInstaller.ProviderInstallListener.html#onProviderInstalled()方法，並且知道通信是安全的。如果security provider無法更新，會調用[onproviderinstallfailed()](http://developer.android.com/reference/com/google/android/gms/security/ProviderInstaller.ProviderInstallListener.html#onProviderInstallFailed(int, android.content.Intent)方法，並採取適當的行動（如提示用戶更新Google Play services）


```java
/**
 * Sample activity using {@link ProviderInstaller}.
 */
public class MainActivity extends Activity
    implements ProviderInstaller.ProviderInstallListener {

  private static final int ERROR_DIALOG_REQUEST_CODE = 1;

  private boolean mRetryProviderInstall;

  //Update the security provider when the activity is created.
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    ProviderInstaller.installIfNeededAsync(this, this);
  }

  /**
   * This method is only called if the provider is successfully updated
   * (or is already up-to-date).
   */
  @Override
  protected void onProviderInstalled() {
    // Provider is up-to-date, app can make secure network calls.
  }

  /**
   * This method is called if updating fails; the error code indicates
   * whether the error is recoverable.
   */
  @Override
  protected void onProviderInstallFailed(int errorCode, Intent recoveryIntent) {
    if (GooglePlayServicesUtil.isUserRecoverableError(errorCode)) {
      // Recoverable error. Show a dialog prompting the user to
      // install/update/enable Google Play services.
      GooglePlayServicesUtil.showErrorDialogFragment(
          errorCode,
          this,
          ERROR_DIALOG_REQUEST_CODE,
          new DialogInterface.OnCancelListener() {
            @Override
            public void onCancel(DialogInterface dialog) {
              // The user chose not to take the recovery action
              onProviderInstallerNotAvailable();
            }
          });
    } else {
      // Google Play services is not available.
      onProviderInstallerNotAvailable();
    }
  }

  @Override
  protected void onActivityResult(int requestCode, int resultCode,
      Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (requestCode == ERROR_DIALOG_REQUEST_CODE) {
      // Adding a fragment via GooglePlayServicesUtil.showErrorDialogFragment
      // before the instance state is restored throws an error. So instead,
      // set a flag here, which will cause the fragment to delay until
      // onPostResume.
      mRetryProviderInstall = true;
    }
  }

  /**
   * On resume, check to see if we flagged that we need to reinstall the
   * provider.
   */
  @Override
  protected void onPostResume() {
    super.onPostResult();
    if (mRetryProviderInstall) {
      // We can now safely retry installation.
      ProviderInstall.installIfNeededAsync(this, this);
    }
    mRetryProviderInstall = false;
  }

  private void onProviderInstallerNotAvailable() {
    // This is reached if the provider cannot be updated for some reason.
    // App should consider all HTTP communication to be vulnerable, and take
    // appropriate action.
  }
}

```
