# 識別用戶的當下活動

> 編寫:[penkzhou](https://github.com/penkzhou) - 原文:<http://developer.android.com/training/location/activity-recognition.html>

活動識別會去探測用戶當前的身體活動，比如步行，駕駛以及站立。通過一個不同於請求位置更新或者地理圍欄的活動識別client來請求用戶活動更新，但是請求方式是類似的。根據你設置的更新頻率，Location Services會返回包含一個或者多個活動以及它們出現對應的概率的反饋信息。這一課將會向你展示如何從Location Services請求活動識別更新。

## 1)請求活動識別更新

從Location Services請求活動識別更新的過程與請求週期性的位置更新類似。你通過一個client發送請求，接著Location Services 以 [PendingIntent](http://developer.android.com/reference/android/app/PendingIntent.html)的形式將更新數據返回。然而，你在開始之前必須設置好對應的權限。下面的課程將會教你如何設置權限，連接client以及請求更新。

### 1.1)設置接收更新數據的權限

一個應用想要獲得活動識別數據就必須擁有`com.google.android.gms.permission.ACTIVITY_RECOGNITION`權限。為了讓你的應用有這個權限，在你的manifest文件裡面將如下代碼放到`<manifest>`標籤的裡面。

```java
<uses-permission
    android:name="com.google.android.gms.permission.ACTIVITY_RECOGNITION"/>
```

活動識別不需要[ACCESS_COARSE_LOCATION](http://developer.android.com/reference/android/Manifest.permission.html#ACCESS_COARSE_LOCATION)權限和 [ACCESS_FINE_LOCATION](http://developer.android.com/reference/android/Manifest.permission.html#ACCESS_FINE_LOCATION)權限。

### 1.2)檢查Google Play Services是否可用

位置服務是Google Play services 中的一部分。由於很難預料用戶設備的狀態，所以你在嘗試連接位置服務之前應該要檢測你的設備是否安裝了Google Play services安裝包。為了檢測這個安裝包是否被安裝，你可以調用[GooglePlayServicesUtil.isGooglePlayServicesAvailable()](http://developer.android.com/reference/com/google/android/gms/common/GooglePlayServicesUtil.html#isGooglePlayServicesAvailable(android.content.Context)，這個方法將會返回一個結果代碼。你可以通過查詢[ConnectionResult](http://developer.android.com/reference/com/google/android/gms/common/ConnectionResult.html)的參考文檔中結果代碼列表來理解對應的結果代碼。如果你碰到了錯誤，你可以調用[GooglePlayServicesUtil.getErrorDialog()](http://developer.android.com/reference/com/google/android/gms/common/GooglePlayServicesUtil.html#getErrorDialog(int, android.app.Activity, int))獲取本地化的對話框來提示用戶採取適當地行為，接著你需要將這個對話框置於一個[DialogFragment](http://developer.android.com/reference/android/support/v4/app/DialogFragment.html)中顯示。這個對話框可以讓用戶去糾正這個問題，這個時候Google Services可以將結果返回給你的activity。為了處理這個結果，重寫[onActivityResult()](http://developer.android.com/reference/android/app/Activity.html#onActivityResult(int, int, android.content.Intent))即可。

> **Note:** 為了讓你的應用能夠兼容 Android 1.6 之後的版本，用來顯示DialogFragment的必須是FragmentActivity而不是之前的Activity。使用FragmentActivity同樣可以調用 getSupportFragmentManager() 方法來顯示 DialogFragment。

因為你的代碼裡通常會不止一次地檢測Google Play services是否安裝, 為了方便，可以定義一個方法來封裝這種檢測行為。下面的代碼片段包含了所有檢測Google Play services是否安裝需要用到的代碼：

```java
public class MainActivity extends FragmentActivity {
    ...
    //全局變量
    /*
     * 定義一個發送給Google Play services的請求代碼
     * 這個代碼將會在Activity.onActivityResult的方法中返回
     */
    private final static int
            CONNECTION_FAILURE_RESOLUTION_REQUEST = 9000;
    ...
    // 定義一個顯示錯誤對話框的DialogFragment
    public static class ErrorDialogFragment extends DialogFragment {
        // 表示錯誤對話框的全局屬性
        private Dialog mDialog;
        // 默認的構造函數，將 dialog 屬性設為空
        public ErrorDialogFragment() {
            super();
            mDialog = null;
        }
        // 設置要顯示的dialog
        public void setDialog(Dialog dialog) {
            mDialog = dialog;
        }
        // 返回一個 Dialog 給 DialogFragment.
        @Override
        public Dialog onCreateDialog(Bundle savedInstanceState) {
            return mDialog;
        }
    }
    ...
    /*
     * 處理來自Google Play services 發給FragmentActivity的結果
     *
     */
    @Override
    protected void onActivityResult(
            int requestCode, int resultCode, Intent data) {
        // 根據請求代碼來決定做什麼
        switch (requestCode) {
            ...
            case CONNECTION_FAILURE_RESOLUTION_REQUEST :
            /*
             * 如果結果代碼是 Activity.RESULT_OK, 嘗試重新連接
             *
             */
                switch (resultCode) {
                    case Activity.RESULT_OK :
                    /*
                     * 嘗試重新請求
                     */
                    ...
                    break;
                }
            ...
        }
     }
    ...
    private boolean servicesConnected() {
        // 檢測Google Play services 是否可用
        int resultCode =
                GooglePlayServicesUtil.
                        isGooglePlayServicesAvailable(this);
        // 如果 Google Play services 可用
        if (ConnectionResult.SUCCESS == resultCode) {
            // 在 debug 模式下, 記錄程序日誌
            Log.d("Location Updates",
                    "Google Play services is available.");
            // Continue
            return true;
        // 因為某些原因Google Play services 不可用
        } else {
            // 獲取error code
            int errorCode = connectionResult.getErrorCode();
            // 從Google Play services 獲取 error dialog
            Dialog errorDialog = GooglePlayServicesUtil.getErrorDialog(
                    errorCode,
                    this,
                    CONNECTION_FAILURE_RESOLUTION_REQUEST);

            // 如果 Google Play services可以提供一個error dialog
            if (errorDialog != null) {
                // 為這個error dialog 創建一個新的DialogFragment
                ErrorDialogFragment errorFragment =
                        new ErrorDialogFragment();
                // 在DialogFragment中設置dialog
                errorFragment.setDialog(errorDialog);
                // 在DialogFragment中顯示error dialog
                errorFragment.show(getSupportFragmentManager(),
                        "Geofence Detection");
            }
        }
    }
    ...
}
```

下面的代碼片段使用了這個方法來檢查Google Play services是否可用。

### 1.3)發送活動更新數據請求

一般的更新數據請求都是從一個實現了Location Services回調函數的[Activity](http://developer.android.com/reference/android/app/Activity.html) 或者[Fragment](http://developer.android.com/reference/android/support/v4/app/Fragment.html)發出來的。生成這個請求的過程是一個異步過程，它是在你請求到活動識別client的連接的時候開始的。當這個client連接上的時候，Location Services對調用你對[onConnected()](http://developer.android.com/reference/com/google/android/gms/common/GooglePlayServicesClient.ConnectionCallbacks.html#onConnected(android.os.Bundle)方法的實現。在這個方法裡面，你可以發送更新數據的請求到Location Services；這個請求是異步的。一旦你生成這個請求，你就可以斷開client的連接了。

這個過程會在下面的代碼裡面描述。

### 1.4)定義 Activity 和 Fragment

定義一個實現如下接口的[FragmentActivity](http://developer .android.com/reference/android/support/v4/app/FragmentActivity.html) 或者[Fragment](http://developer.android.com/reference/android/support/v4/app/Fragment.html)：

[ConnectionCallbacks](http://developer.android.com/reference/com/google/android/gms/common/GooglePlayServicesClient.ConnectionCallbacks.html)

* 實現當client連接上或者斷開連接時Location Services 調用的方法。

[OnConnectionFailedListener](http://developer.android.com/reference/com/google/android/gms/common/GooglePlayServicesClient.OnConnectionFailedListener.html)

* 實現當client連接出現錯誤時Location Services 調用的方法。

例如：

```java
public class MainActivity extends FragmentActivity implements
        ConnectionCallbacks, OnConnectionFailedListener {
    ...
}
```

接下來，定義全局變量。為更新頻率定義一個常量，為活動識別client 定義一個變量，為Location Services用來發送更新的[PendingIntent](http://developer.android.com/reference/android/app/PendingIntent.html)添加一個變量：

```java
public class MainActivity extends FragmentActivity implements
        ConnectionCallbacks, OnConnectionFailedListener {
    ...
    // Constants that define the activity detection interval
    public static final int MILLISECONDS_PER_SECOND = 1000;
    public static final int DETECTION_INTERVAL_SECONDS = 20;
    public static final int DETECTION_INTERVAL_MILLISECONDS =
            MILLISECONDS_PER_SECOND * DETECTION_INTERVAL_SECONDS;
    ...
    /*
     * Store the PendingIntent used to send activity recognition events
     * back to the app
     */
    private PendingIntent mActivityRecognitionPendingIntent;
    // Store the current activity recognition client
    private ActivityRecognitionClient mActivityRecognitionClient;
    ...
}
```

在 [onCreate()](http://developer.android.com/reference/android/app/Activity.html#onCreate(android.os.Bundle))方法裡面，為活動識別client和[PendingIntent](http://developer.android.com/reference/android/app/PendingIntent.html)賦值：

```java
public class MainActivity extends FragmentActivity implements
        ConnectionCallbacks, OnConnectionFailedListener {
    ...
    @Override
    onCreate(Bundle savedInstanceState) {
        ...
        /*
         * Instantiate a new activity recognition client. Since the
         * parent Activity implements the connection listener and
         * connection failure listener, the constructor uses "this"
         * to specify the values of those parameters.
         */
        mActivityRecognitionClient =
                new ActivityRecognitionClient(mContext, this, this);
        /*
         * Create the PendingIntent that Location Services uses
         * to send activity recognition updates back to this app.
         */
        Intent intent = new Intent(
                mContext, ActivityRecognitionIntentService.class);
        /*
         * Return a PendingIntent that starts the IntentService.
         */
        mActivityRecognitionPendingIntent =
                PendingIntent.getService(mContext, 0, intent,
                PendingIntent.FLAG_UPDATE_CURRENT);
        ...
    }
    ...
}
```

### 1.5)開啟請求進程

定義一個請求活動識別更新的方法。在這個方法裡面，請求到Location Services的連接。你可以在activity的任何地方調用這個方法；這個方法是用來開啟請求更新數據的方法鏈。


為了避免在你的第一個請求結束之前開啟第二個請求時出現競爭的情況，你可以定義一個boolean標誌位來記錄當前請求的狀態。在開始請求的時候設置標誌位值為```true``` ，在請求結束的時候設置標誌位為```false``` 。

下面的代碼展示瞭如何開始一個更新請求：

```java
public class MainActivity extends FragmentActivity implements
        ConnectionCallbacks, OnConnectionFailedListener {
    ...
    // Global constants
    ...
    // Flag that indicates if a request is underway.
    private boolean mInProgress;
    ...
    @Override
    onCreate(Bundle savedInstanceState) {
        ...
        // Start with the request flag set to false
        mInProgress = false;
        ...
    }
    ...
    /**
     * Request activity recognition updates based on the current
     * detection interval.
     *
     */
     public void startUpdates() {
        // Check for Google Play services

        if (!servicesConnected()) {
            return;
        }
        // If a request is not already underway
        if (!mInProgress) {
            // Indicate that a request is in progress
            mInProgress = true;
            // Request a connection to Location Services
            mActivityRecognitionClient.connect();
        //
        } else {
            /*
             * A request is already underway. You can handle
             * this situation by disconnecting the client,
             * re-setting the flag, and then re-trying the
             * request.
             */
        }
    }
    ...
}
```

下面就實現了[onConnected()](http://developer.android.com/reference/com/google/android/gms/common/GooglePlayServicesClient.ConnectionCallbacks.html#onConnected(android.os.Bundle)方法。在這個方法裡面，從Location Services請求活動識別更新。當Location Services 結束對client的連接過程然後調用[onConnected()](http://developer.android.com/reference/com/google/android/gms/common/GooglePlayServicesClient.ConnectionCallbacks.html#onConnected(android.os.Bundle)方法時，這個更新請求就會直接被調用：

```java
public class MainActivity extends FragmentActivity implements
        ConnectionCallbacks, OnConnectionFailedListener {
    ...
    /*
     * Called by Location Services once the location client is connected.
     *
     * Continue by requesting activity updates.
     */
    @Override
    public void onConnected(Bundle dataBundle) {
        /*
         * Request activity recognition updates using the preset
         * detection interval and PendingIntent. This call is
         * synchronous.
         */
        mActivityRecognitionClient.requestActivityUpdates(
                DETECTION_INTERVAL_MILLISECONDS,
                mActivityRecognitionPendingIntent);
        /*
         * Since the preceding call is synchronous, turn off the
         * in progress flag and disconnect the client
         */
        mInProgress = false;
        mActivityRecognitionClient.disconnect();
    }
    ...
}
```

### 1.6)處理斷開連接

在某些情況下，Location Services可能會在你調用[disconnect()](http://developer.android.com/reference/com/google/android/gms/location/ActivityRecognitionClient.html#disconnect())方法之前斷開與活動識別client的連接。為了處理這種情況，實現[onDisconnected()](http://developer.android.com/reference/com/google/android/gms/common/GooglePlayServicesClient.ConnectionCallbacks.html#onDisconnected())方法即可。在這個方法裡面，設置請求標誌位來表示這個請求是否有效，並根據這個標誌位來刪除client：

```java
public class MainActivity extends FragmentActivity implements
        ConnectionCallbacks, OnConnectionFailedListener {
    ...
    /*
     * Called by Location Services once the activity recognition
     * client is disconnected.
     */
    @Override
    public void onDisconnected() {
        // Turn off the request flag
        mInProgress = false;
        // Delete the client
        mActivityRecognitionClient = null;
    }
    ...
}
```

### 1.7)處理連接錯誤

在處理正常的回調函數之外，你還得提供一個回調函數來處理連接出現錯誤的情況。這個回調函數重用了前面在檢查Google Play service的時候用到的DialogFragment類。它還可以重用之前在onActivityResult()方法裡用來接收當用戶和錯誤對話框交互時產生的結果用到的代碼。下面的代碼展示瞭如何實現回調函數：
```java
public class MainActivity extends FragmentActivity implements
        ConnectionCallbacks,
        OnConnectionFailedListener,
        OnAddGeofencesResultListener {
    ...
    // Implementation of OnConnectionFailedListener.onConnectionFailed
    @Override
    public void onConnectionFailed(ConnectionResult connectionResult) {
        // Turn off the request flag
        mInProgress = false;
        /*
         * If the error has a resolution, start a Google Play services
         * activity to resolve it.
         */
        if (connectionResult.hasResolution()) {
            try {
                connectionResult.startResolutionForResult(
                        this,
                        CONNECTION_FAILURE_RESOLUTION_REQUEST);
            } catch (SendIntentException e) {
                // Log the error
                e.printStackTrace();
            }
        // If no resolution is available, display an error dialog
        } else {
            // Get the error code
            int errorCode = connectionResult.getErrorCode();
            // Get the error dialog from Google Play services
            Dialog errorDialog = GooglePlayServicesUtil.getErrorDialog(
                    errorCode,
                    this,
                    CONNECTION_FAILURE_RESOLUTION_REQUEST);
            // If Google Play services can provide an error dialog
            if (errorDialog != null) {
                // Create a new DialogFragment for the error dialog
                ErrorDialogFragment errorFragment =
                        new ErrorDialogFragment();
                // Set the dialog in the DialogFragment
                errorFragment.setDialog(errorDialog);
                // Show the error dialog in the DialogFragment
                errorFragment.show(
                        getSupportFragmentManager(),
                        "Activity Recognition");
            }
        }
    }
    ...
}
```

## 2)處理活動更新數據

為了處理Location Services每一個週期發送的[Intent](http://developer.android.com/reference/android/content/Intent.html)，你可以定義一個[IntentService](http://developer.android.com/reference/android/app/IntentService.html)以及它的[onHandleIntent()](http://developer.android.com/reference/android/app/IntentService.html#onHandleIntent(android.content.Intent)方法。 Location Services以[Intent](http://developer.android.com/reference/android/content/Intent.html)對象的形式返回活動識別更新數據，並使用了你在調用[requestActivityUpdates()](http://developer.android.com/reference/com/google/android/gms/location/ActivityRecognitionClient.html#requestActivityUpdates(long, android.app.PendingIntent))方法時產生的[PendingIntent](http://developer.android.com/reference/android/app/PendingIntent.html) 。因為你為這個[PendingIntent](http://developer.android.com/reference/android/app/PendingIntent.html)提供了一個單獨的intent，那麼接收這個intent的唯一組件就是[IntentService](http://developer.android.com/reference/android/app/IntentService.html)了。

下面的代碼展示瞭如何來檢查活動識別更新數據。

### 2.1)定義一個IntentService

首先定義這個類以及它的[onHandleIntent()](http://developer.android.com/reference/android/app/IntentService.html#onHandleIntent(android.content.Intent)方法：

```java
/**
 * Service that receives ActivityRecognition updates. It receives
 * updates in the background, even if the main Activity is not visible.
 */
public class ActivityRecognitionIntentService extends IntentService {
    ...
    /**
     * Called when a new activity detection update is available.
     */
    @Override
    protected void onHandleIntent(Intent intent) {
        ...
    }
    ...
}
```


接下啦，在intent裡面檢查數據。你可以從這個數據裡面獲取到所有可能的活動列表以及它們對應的概率。下面的代碼展示瞭如何獲取可能性最大的活動，活動對應的概率以及它的類型：

```java
public class ActivityRecognitionIntentService extends IntentService {
    ...
    @Override
    protected void onHandleIntent(Intent intent) {
        ...
        // If the incoming intent contains an update
        if (ActivityRecognitionResult.hasResult(intent)) {
            // Get the update
            ActivityRecognitionResult result =
                    ActivityRecognitionResult.extractResult(intent);
            // Get the most probable activity
            DetectedActivity mostProbableActivity =
                    result.getMostProbableActivity();
            /*
             * Get the probability that this activity is the
             * the user's actual activity
             */
            int confidence = mostProbableActivity.getConfidence();
            /*
             * Get an integer describing the type of activity
             */
            int activityType = mostProbableActivity.getType();
            String activityName = getNameFromType(activityType);
            /*
             * At this point, you have retrieved all the information
             * for the current update. You can display this
             * information to the user in a notification, or
             * send it to an Activity or Service in a broadcast
             * Intent.
             */
            ...
        } else {
            /*
             * This implementation ignores intents that don't contain
             * an activity update. If you wish, you can report them as
             * errors.
             */
        }
        ...
    }
    ...
}
```

`getNameFromType()` 方法將活動類型轉化成了對應的描述性字符串。在一個正式的應用中，你應該從資源文件中去獲取字符串而不是使用擁有固定值的變量：

```java
public class ActivityRecognitionIntentService extends IntentService {
    ...
    /**
     * Map detected activity types to strings
     *@param activityType The detected activity type
     *@return A user-readable name for the type
     */
    private String getNameFromType(int activityType) {
        switch(activityType) {
            case DetectedActivity.IN_VEHICLE:
                return "in_vehicle";
            case DetectedActivity.ON_BICYCLE:
                return "on_bicycle";
            case DetectedActivity.ON_FOOT:
                return "on_foot";
            case DetectedActivity.STILL:
                return "still";
            case DetectedActivity.UNKNOWN:
                return "unknown";
            case DetectedActivity.TILTING:
                return "tilting";
        }
        return "unknown";
    }
    ...
}
```


### 2.2)在manifest文件裡面添加IntentService

為了讓系統識別這個IntentService，你需要在應用的manifest文件裡面添加```<service>```標籤：

```java
<service
    android:name="com.example.android.location.ActivityRecognitionIntentService"
    android:label="@string/app_name"
    android:exported="false">
</service>
```

注意你不必為這個服務去設置特定的intent filters，因為它只接受特定的intent。定義Activity和Fragment這一段已經描述了活動更新intent是如何被創建的。

## 3)停止活動識別更新

停止活動識別更新的過程與開啟活動識別更新的過程類似，只要將調用的方法[removeActivityUpdates()](http://developer.android.com/reference/com/google/android/gms/location/ActivityRecognitionClient.html#removeActivityUpdates(android.app.PendingIntent)換成[requestActivityUpdates()](http://developer.android.com/reference/com/google/android/gms/location/ActivityRecognitionClient.html#requestActivityUpdates(long, android.app.PendingIntent)即可。

停止更新的過程使用了你在添加請求更新時使用過的幾個方法，開始的時候要為兩種操作定義請求類型：

```java
public class MainActivity extends FragmentActivity implements
        ConnectionCallbacks, OnConnectionFailedListener {
    ...
    public enum REQUEST_TYPE {START, STOP}
    private REQUEST_TYPE mRequestType;
    ...
}
```

更改開始請求活動識別更新的代碼，在裡面使用 ```START``` 請求參數：

```java
public class MainActivity extends FragmentActivity implements
        ConnectionCallbacks, OnConnectionFailedListener {
    ...
    public void startUpdates() {
        // Set the request type to START
        mRequestType = REQUEST_TYPE.START;
        /*
         * Test for Google Play services after setting the request type.
         * If Google Play services isn't present, the proper request type
         * can be restarted.
         */
        if (!servicesConnected()) {
            return;
        }
        ...
    }
    ...
    public void onConnected(Bundle dataBundle) {
        switch (mRequestType) {
            case START :
                /*
                 * Request activity recognition updates using the
                 * preset detection interval and PendingIntent.
                 * This call is synchronous.
                 */
                mActivityRecognitionClient.requestActivityUpdates(
                        DETECTION_INTERVAL_MILLISECONDS,
                        mActivityRecognitionPendingIntent);
                break;
                ...
                /*
                 * An enum was added to the definition of REQUEST_TYPE,
                 * but it doesn't match a known case. Throw an exception.
                 */
                default :
                throw new Exception("Unknown request type in onConnected().");
                break;
        }
        ...
    }
    ...
}
```

### 3.1)開始請求停止更新

定義一個方法來請求停止活動識別更新。在這個方法裡面，設置號請求類型，然後向Location Services發起連接。接著你就可以在activity裡面的任何地方調用這個方法了。這樣做的目的就是開啟停止活動更新的方法鏈：

```java
public class MainActivity extends FragmentActivity implements
        ConnectionCallbacks, OnConnectionFailedListener {
    ...
    /**
     * Turn off activity recognition updates
     *
     */
    public void stopUpdates() {
        // Set the request type to STOP
        mRequestType = REQUEST_TYPE.STOP;
        /*
         * Test for Google Play services after setting the request type.
         * If Google Play services isn't present, the request can be
         * restarted.
         */
        if (!servicesConnected()) {
            return;
        }
        // If a request is not already underway
        if (!mInProgress) {
            // Indicate that a request is in progress
            mInProgress = true;
            // Request a connection to Location Services
            mActivityRecognitionClient.connect();
        //
        } else {
            /*
             * A request is already underway. You can handle
             * this situation by disconnecting the client,
             * re-setting the flag, and then re-trying the
             * request.
             */
        }
        ...
    }
    ...
}
```

在[onConnected()](http://developer.android.com/reference/com/google/android/gms/common/GooglePlayServicesClient.ConnectionCallbacks.html#onConnected(android.os.Bundle)方法裡面，如果請求參數類型是 ```STOP```,則調用 [removeActivityUpdates()](http://developer.android.com/reference/com/google/android/gms/location/ActivityRecognitionClient.html#removeActivityUpdates(android.app.PendingIntent)方法。將你之前用來開啟更新進程的[PendingIntent](http://developer.android.com/reference/android/app/PendingIntent.html)作為一個參數傳給[removeActivityUpdates()](http://developer.android.com/reference/com/google/android/gms/location/ActivityRecognitionClient.html#removeActivityUpdates(android.app.PendingIntent)方法：

```java
public class MainActivity extends FragmentActivity implements
        ConnectionCallbacks, OnConnectionFailedListener {
    ...
    public void onConnected(Bundle dataBundle) {
        switch (mRequestType) {
            ...
            case STOP :
            mActivityRecognitionClient.removeActivityUpdates(
                    mActivityRecognitionPendingIntent);
            break;
            ...
        }
        ...
    }
    ...
}
```

你不需要改變你對onDisconnected()方法和onConnectionFailed()方法的實現，因為這些方法不依賴這些請求類型。

現在你已經擁有了一個實現了活動識別的應用的基本架構了。你還可以與其他幾個基於地理位置的特徵進行結合。
