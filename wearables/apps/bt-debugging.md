# 通過藍牙進行調試

> 編寫: [kesenhoo](https://github.com/kesenhoo) - 原文: <http://developer.android.com/training/wearables/apps/bt-debugging.html>

我們可以通過藍牙來調試我們的可穿戴應用。即通過藍牙把調試數據輸出到已經連接了開發電腦的手持設備上。

## 搭建好設備用來調試

1. 開啟手持設備的USB調試：
    * 打開設置應用並滑動到底部。
    * 如果在設置裡面沒有開發者選項，點擊**關於手機**（或者**關於平板**），滑動到底部，點擊build number 7次。
    * 返回並點擊**開發者選項**。
    * 開啟**USB調試**。
2. 開啟可穿戴設備的藍牙調試：
    * 點擊主界面2次，來到Wear菜單界面。
    * 滑動到底部，點擊**設置**。
    * 滑動到底部，如果沒有**開發者選項**，點擊**關於**，然後點擊Build Number 7次。
    * 點擊**開發者選項**。
    * 開啟**藍牙調試**。

## 建立調試會話

1. 在手持設備上，打開`Android Wear`配套應用。
2. 點擊右上角的菜單，選擇**設置**。
3. 開啟**藍牙調試**。我們將會在選項下面看到一個小的狀態信息：
```xml
Host: disconnected
Target: connected
```
4. 通過USB連接手持設備到電腦上，並執行下面的命令：
```xml
adb forward tcp:4444 localabstract:/adb-hub
adb connect localhost:4444
```
> **Note:** 我們可以使用任何可用的端口。

在`Android Wear`配套應用上，我們將會看到狀態變為：
```xml
Host: connected
Target: connected
```

## 調試應用

當運行`abd devices`的命令時，我們的可穿戴設備應該表示為localhost:4444。執行任何的`adb`命令，需要使用下面的格式：

```xml
adb -s localhost:4444 <command>
```

如果沒有任何其他的設備通過TCP/IP連接到手持設備（即模擬器），我們可以使用下面的簡短命令：

```xml
adb -e <command>
```

例如：

```xml
adb -e logcat
adb -e shell
adb -e bugreport
```
