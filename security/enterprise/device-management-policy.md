# 使用設備管理策略增強安全性

> 編寫:[craftsmanBai](https://github.com/craftsmanBai) - <http://z1ng.net> - 原文: <http://developer.android.com/training/enterprise/device-management-policy.html>

Android 2.2(API Level 8)之後，Android平臺通過設備管理API提供系統級的設備管理能力。

在這一小節中，你將學到如何通過使用設備管理策略創建安全敏感的應用程序。比如某應用可被配置為：在給用戶顯示受保護的內容之前，確保已設置一個足夠強度的鎖屏密碼。

## 定義並聲明你的策略

首先，你需要定義多種在功能層面提供支持的策略。這些策略可以包括屏幕鎖密碼強度、密碼過期時間以及加密等等方面。

你須在res/xml/device_admin.xml中聲明選擇的策略集，它將被應用強制實行。在Android manifest也需要引用聲明的策略集。

每個聲明的策略對應[DevicePolicyManager](http://developer.android.com/reference/android/app/admin/DevicePolicyManager.html)中一些相關設備的策略方法（例如定義最小密碼長度或最少大寫字母字符數）。如果一個應用嘗試調用XML中沒有對應策略的方法，程序在會運行時拋出一個[SecurityException](http://developer.android.com/reference/java/lang/SecurityException.html)異常。

如果應用程序試圖管理其他策略，那麼強制鎖force-lock之類的其他權限就會發揮作用。正如你將看到的，作為設備管理權限激活過程的一部分，聲明策略的列表會在系統屏幕上顯示給用戶。
如下代碼片段在res/xml/device_admin.xml中聲明瞭密碼限制策略：

```xml
<device-admin xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-policies>
        <limit-password />
    </uses-policies>
</device-admin>
```
在Android manifest引用XML策略聲明：

```xml
<receiver android:name=".Policy$PolicyAdmin"
    android:permission="android.permission.BIND_DEVICE_ADMIN">
    <meta-data android:name="android.app.device_admin"
        android:resource="@xml/device_admin" />
    <intent-filter>
        <action android:name="android.app.action.DEVICE_ADMIN_ENABLED" />
    </intent-filter>
</receiver>
```

## 創建一個設備管理接受端

創建一個設備管理廣播接收端（broadcast receiver），可以接收到與你聲明的策略有關的事件通知。也可以對應用程序有選擇地重寫回調函數。

在同樣的應用程序（Device Admin）中，當設備管理（device administrator）權限被用戶設為禁用時，已配置好的策略就會從共享偏好設置（shared preference）中擦除。

你應該考慮實現與你的應用業務邏輯相關的策略。例如，你的應用可以採取一些措施來降低安全風險，如：刪除設備上的敏感數據，禁用遠程同步，對管理員的通知提醒等等。

為了讓廣播接收端能夠正常工作，請務必在Android manifest中註冊下面代碼片段所示內容。

```xml
<receiver android:name=".Policy$PolicyAdmin"
    android:permission="android.permission.BIND_DEVICE_ADMIN">
    <meta-data android:name="android.app.device_admin"
        android:resource="@xml/device_admin" />
    <intent-filter>
        <action android:name="android.app.action.DEVICE_ADMIN_ENABLED" />
    </intent-filter>
</receiver>
```

## 激活設備管理器

在執行任何策略之前，用戶需要手動將程序激活為具有設備管理權限，下面的程序片段顯示瞭如何觸發設置框以便讓用戶為你的程序激活權限。

通過指定[EXTRA_ADD_EXPLANATION](http://developer.android.com/reference/android/app/admin/DevicePolicyManager.html#EXTRA_ADD_EXPLANATION)給出明確的說明信息，以告知用戶為應用程序激活設備管理權限的好處。

```java
if (!mPolicy.isAdminActive()) {

    Intent activateDeviceAdminIntent =
        new Intent(DevicePolicyManager.ACTION_ADD_DEVICE_ADMIN);

    activateDeviceAdminIntent.putExtra(
        DevicePolicyManager.EXTRA_DEVICE_ADMIN,
        mPolicy.getPolicyAdmin());

    // It is good practice to include the optional explanation text to
    // explain to user why the application is requesting to be a device
    // administrator. The system will display this message on the activation
    // screen.
    activateDeviceAdminIntent.putExtra(
        DevicePolicyManager.EXTRA_ADD_EXPLANATION,
        getResources().getString(R.string.device_admin_activation_message));

    startActivityForResult(activateDeviceAdminIntent,
        REQ_ACTIVATE_DEVICE_ADMIN);
}
```

![](device-mgmt-activate-device-admin.png)

如果用戶選擇"Activate"，程序就會獲取設備管理員權限並可以開始配置和執行策略。
當然，程序也需要做好處理用戶選擇放棄激活的準備，比如用戶點擊了“取消”按鈕，返回鍵或者HOME鍵的情況。因此，如果有必要的話，策略設置中的*[onResume()](http://developer.android.com/reference/android/app/Activity.html#onResume())*方法需要加入重新評估的邏輯判斷代碼，以便將設備管理激活選項展示給用戶。

## 實施設備策略控制

在設備管理權限成功激活後，程序就會根據請求的策略來配置設備策略管理器。要牢記，新策略會被添加到每個版本的Android中。所以你需要在程序中做好平臺版本的檢測，以便新策略能被老版本平臺很好的支持。例如，“密碼中含有的最少大寫字符數”這個安全策略只有在高於API Level 11（Honeycomb）的平臺才被支持，以下代碼則演示瞭如何在運行時檢查版本：

```java
DevicePolicyManager mDPM = (DevicePolicyManager)
        context.getSystemService(Context.DEVICE_POLICY_SERVICE);
ComponentName mPolicyAdmin = new ComponentName(context, PolicyAdmin.class);
...
mDPM.setPasswordQuality(mPolicyAdmin, PASSWORD_QUALITY_VALUES[mPasswordQuality]);
mDPM.setPasswordMinimumLength(mPolicyAdmin, mPasswordLength);
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
    mDPM.setPasswordMinimumUpperCase(mPolicyAdmin, mPasswordMinUpperCase);
}
```

這樣程序就可以執行策略了。當程序無法訪問正確的鎖屏密碼的時候，通過設備策略管理器（Device Policy Manager）API可以判斷當前密碼是否適用於請求的策略。如果當前鎖屏密碼滿足策略，設備管理API不會採取糾正措施。明確地啟動設置程序中的系統密碼更改界面是應用程序的責任。例如：

```java
if (!mDPM.isActivePasswordSufficient()) {
    ...
    // Triggers password change screen in Settings.
    Intent intent =
        new Intent(DevicePolicyManager.ACTION_SET_NEW_PASSWORD);
    startActivity(intent);
}
```

一般來說，用戶可以從可用的鎖屏機制中任選一個，例如“無”、“圖案”、“PIN碼”（數字）或密碼（字母數字）。當一個密碼策略配置好後，那些比已定義密碼策略弱的密碼會被禁用。比如，如果配置了密碼級別為“Numeric”，那麼用戶只可以選擇PIN碼（數字）或者密碼（字母數字）。

一旦設備通過設置適當的鎖屏密碼處於被保護的狀態，應用程序便允許訪問受保護的內容。

```java
if (!mDPM.isAdminActive(..)) {
    // Activates device administrator.
    ...
} else if (!mDPM.isActivePasswordSufficient()) {
    // Launches password set-up screen in Settings.
    ...
} else {
    // Grants access to secure content.
    ...
    startActivity(new Intent(context, SecureActivity.class));
}
```
