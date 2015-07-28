# 支持多個遊戲控制器

> 編寫:[heray1990](https://github.com/heray1990) - 原文:<http://developer.android.com/training/game-controllers/multiple-controllers.html>

儘管大部分的遊戲都被設計成一臺 Android 設備支持一個用戶，但是仍然有可能支持在同一臺 Android 設備上同時連接的多個遊戲控制器（即多個用戶）。

這節課覆蓋了一些處理單個設備多個玩家（或者多個控制器）輸入的基本技術。這包括維護一個在玩家化身和每個控制器之間的映射，以及適當地處理控制器的輸入事件。

## 映射玩家到控制器的設備 ID

當一個遊戲控制器連接到一臺 Android 設備，系統會為控制器指定一個整型的設備 ID。我們可以通過調用 <a href="http://developer.android.com/reference/android/view/InputDevice.html#getDeviceIds()">`InputDevice.getDeviceIds()`</a> 來取得已連接的遊戲控制器的設備 ID，如[驗證遊戲控制器是否已連接](http://developer.android.com/training/game-controllers/controller-input.html#input)介紹的一樣。我們可以將每個設備 ID 與遊戲中的玩家關聯起來，然後分別處理每個玩家的遊戲動作。

> **Note：**在運行著 Android 4.1（API level 16）或者更高版本的設備上，我們可以通過使用 <a href="http://developer.android.com/reference/android/view/InputDevice.html#getDescriptor()">`getDescriptor()`</a> 來取得輸入設備的描述符。上述函數為輸入設備返回一個唯一連續的字符串值。不同於設備 ID，即使在輸入設備斷開、重連或者重新配置時，描述符都不會變化。

下面的代碼介紹瞭如何使用 [SparseArray](http://developer.android.com/reference/android/util/SparseArray.html) 來關聯玩家化身與一個特定的控制器。在這個例子中，`mShips` 變量保存了一個 `Ship` 對象的集合。當一個新的控制器連接到一個用戶時，會創建一個新的玩家化身。當已關聯的控制器被移除時，對應的玩家化身會被移除。

`onInputDeviceAdded()` 和 `onInputDeviceRemoved()` 回調函數是[在不同的 Android 系統版本支持控制器](compatibility.html)中介紹的抽象層的一部分。通過實現這些 listener 回調，我們的遊戲可以在添加或者移除控制器的時候，識別出遊戲控制器的設備 ID。這個檢測兼容 Android 2.3（API level 9）和更高的版本。

```java
private final SparseArray<Ship> mShips = new SparseArray<Ship>();

@Override
public void onInputDeviceAdded(int deviceId) {
    getShipForID(deviceId);
}

@Override
public void onInputDeviceRemoved(int deviceId) {
    removeShipForID(deviceId);
}

private Ship getShipForID(int shipID) {
    Ship currentShip = mShips.get(shipID);
    if ( null == currentShip ) {
        currentShip = new Ship();
        mShips.append(shipID, currentShip);
    }
    return currentShip;
}

private void removeShipForID(int shipID) {
    mShips.remove(shipID);
}
```

## 處理多個控制器輸入

我們的遊戲應該執行下面的循環來處理多個控制器的輸入：

1. 檢測是否出現一個輸入事件。

2. 識別輸入源和它的設備 ID。

3. 根據以輸入事件按鍵碼或者座標值的形式表示的 action，更新玩家化身與設備 ID 的關聯關係。

4. 渲染和更新用戶界面。

[keyEvent](http://developer.android.com/reference/android/view/KeyEvent.html) 和 [MotionEvent](http://developer.android.com/reference/android/view/MotionEvent.html) 輸入事件與設備 ID 相關聯。我們的遊戲可以利用這個關聯來確定輸入事件從哪個控制器發出，並且更新玩家化身與控制器的關聯。

下面的代碼介紹了我們如何將一個玩家化身引用相應的遊戲控制器設備 ID，並且根據用戶按下控制器的按鍵來更新遊戲。

```java
@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {
    if ((event.getSource() & InputDevice.SOURCE_GAMEPAD)
                == InputDevice.SOURCE_GAMEPAD) {
        int deviceId = event.getDeviceId();
        if (deviceId != -1) {
            Ship currentShip = getShipForId(deviceId);
            // Based on which key was pressed, update the player avatar
            // (e.g. set the ship headings or fire lasers)
            ...
            return true;
        }
    }
    return super.onKeyDown(keyCode, event);
}
```

> **Note：**一個最佳做法，當用戶的遊戲控制器斷開時，我們應該停止遊戲並詢問用戶是否像要重新連接。