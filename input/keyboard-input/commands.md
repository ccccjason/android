# 處理按鍵動作

> 編寫:[zhaochunqi](https://github.com/zhaochunqi) - 原文:<http://developer.android.com/training/keyboard-input/commands.html>

當用戶選中一個可編輯的文本 view（如 [EditText](http://developer.android.com/reference/android/widget/EditText.html) 組件），而且用戶連接了一個實體鍵盤時，所有輸入由系統處理。然而，如果我們想接管或直接處理鍵盤輸入，那麼可以通過實現 [KeyEvent.Callback](http://developer.android.com/reference/android/view/KeyEvent.Callback.html) 接口的回調方法，如 <a href="http://developer.android.com/reference/android/view/KeyEvent.Callback.html#onKeyDown(int, android.view.KeyEvent)">onKeyDown()</a> 和 <a href="http://developer.android.com/reference/android/view/KeyEvent.Callback.html#onKeyMultiple(int, int, android.view.KeyEvent)">onKeyMultiple()</a> 來完成上述目的。

因為 Activity 和 View 類都實現了 [KeyEvent.Callback](http://developer.android.com/reference/android/view/KeyEvent.Callback.html) 接口，所以通常我們應該在這些類的繼承中重寫回調方法。

> **Note:** 當使用 KeyEvent 類和相關的 API 處理鍵盤事件時，我們應該期望這種鍵盤事件只從實體鍵盤發出。我們永遠不應該依賴從一個軟輸入法（如屏幕鍵盤）來接收按鍵事件。

## 處理單個按鍵事件

處理單個的按鍵點擊，需要適當地實現 <a href="http://developer.android.com/reference/android/view/KeyEvent.Callback.html#onKeyDown(int, android.view.KeyEvent)">onKeyDown()</a> 或 <a href="http://developer.android.com/reference/android/view/KeyEvent.Callback.html#onKeyUp(int, android.view.KeyEvent)">onKeyUp()</a>。通常，我們使用 <a href="http://developer.android.com/reference/android/view/KeyEvent.Callback.html#onKeyUp(int, android.view.KeyEvent)">onKeyUp()</a> 來確保我們只接收一個事件。如果用戶點擊並按住按鈕不放，<a href="http://developer.android.com/reference/android/view/KeyEvent.Callback.html#onKeyDown(int, android.view.KeyEvent)">onKeyDown()</a> 會被調用多次。

舉例，這個實現響應一些鍵盤按鍵來控制遊戲：

```java
@Override
public boolean onKeyUp(int keyCode, KeyEvent event) {
    switch (keyCode) {
        case KeyEvent.KEYCODE_D:
            moveShip(MOVE_LEFT);
            return true;
        case KeyEvent.KEYCODE_F:
            moveShip(MOVE_RIGHT);
            return true;
        case KeyEvent.KEYCODE_J:
            fireMachineGun();
            return true;
        case KeyEvent.KEYCODE_K:
            fireMissile();
            return true;
        default:
            return super.onKeyUp(keyCode, event);
    }
}
```

## 處理修飾鍵

為了對修飾鍵（例如將一個按鍵與 Shift 或者 Control 鍵組合）進行迴應，我們可以查詢 [KeyEvent](http://developer.android.com/reference/android/view/KeyEvent.html) 來傳遞到回調方法。一些方法，如 getModifiers() 和 getMetaState()，提供一些關於修飾鍵的信息。然而，最簡單的解決方案是用像 <a href="http://developer.android.com/reference/android/view/KeyEvent.html#isShiftPressed()">isShiftPressed()</a> 和 <a href="http://developer.android.com/reference/android/view/KeyEvent.html#isCtrlPressed()">isCtrlPressed()</a> 等方法，檢查我們關心的修飾鍵是否正在被按下。

例如，有一個 <a href="http://developer.android.com/reference/android/view/KeyEvent.Callback.html#onKeyDown(int, android.view.KeyEvent)">onKeyDown()</a> 的實現，當Shift鍵和一個其他按鍵按下時，做一些額外的處理:

```java
@Override
public boolean onKeyUp(int keyCode, KeyEvent event) {
    switch (keyCode) {
        ...
        case KeyEvent.KEYCODE_J:
            if (event.isShiftPressed()) {
                fireLaser();
            } else {
                fireMachineGun();
            }
            return true;
        case KeyEvent.KEYCODE_K:
            if (event.isShiftPressed()) {
                fireSeekingMissle();
            } else {
                fireMissile();
            }
            return true;
        default:
            return super.onKeyUp(keyCode, event);
    }
}
```
