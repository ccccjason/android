# 使得ListView滑動順暢

> 編寫:[allenlsy](https://github.com/allenlsy) - 原文:<http://developer.android.com/training/improving-layouts/smooth-scrolling.html>

保持程序流暢的關鍵，是讓主線程（UI 線程）不要進行大量運算。你要確保在其他線程執行磁盤讀寫、網絡讀寫或是 SQL 操作等。為了測試你的應用的狀態，你可以啟用 [StrictMode](http://developer.android.com/reference/android/os/StrictMode.html)。

## 使用後臺線程

你應該把主線程中的耗時間的操作，提取到一個後臺線程（也叫做“worker thread工作線程”）中，使得主線程只關注 UI 繪畫。很多時候，使用 [AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) 是一個簡單的在主線程以外進行操作的方法。系統會自動把`execute()`的請求放入隊列中併線性調用執行。這個行為是全局的，這意味著你不需要考慮自己定義線程池的事情。

在下面的例子中，一個 AsyncTask 被用於在後臺線程載入圖片，並在載入完成後把圖片顯示到 UI  上。當圖片正在載入時，它還會顯示一個進度提示。

```java
// Using an AsyncTask to load the slow images in a background thread
new AsyncTask<ViewHolder, Void, Bitmap>() {
    private ViewHolder v;

    @Override
    protected Bitmap doInBackground(ViewHolder... params) {
        v = params[0];
        return mFakeImageLoader.getImage();
    }

    @Override
    protected void onPostExecute(Bitmap result) {
        super.onPostExecute(result);
        if (v.position == position) {
            // If this item hasn't been recycled already, hide the
            // progress and set and show the image
            v.progress.setVisibility(View.GONE);
            v.icon.setVisibility(View.VISIBLE);
            v.icon.setImageBitmap(result);
        }
    }
}.execute(holder);
```

從 Android 3.0 (API level 11) 開始, AsyncTask 有個新特性，那就是它可以在多個 CPU 核上運行。你可以調用 `executeOnExecutor()`而不是`execute()`，前者可以根據CPU的核心數來觸發多個任務同時進行。

## 在 ViewHolder 中填入視圖對象

你的代碼可能在 ListView 滑動時經常使用 `findViewById()`，這樣會降低性能。即使是 Adapter 返回一個用於回收的 inflate 後的視圖，你仍然需要查看這個元素並更新它。避免頻繁調用 `findViewById()` 的方法之一，就是使用 ViewHolder（視圖佔位符）的設計模式。

一個 ViewHolder 對象存儲了他的標籤下的每個視圖。這樣你不用頻繁查找這個元素。第一，你需要創建一個類來存儲你會用到的視圖。比如：

```java
static class ViewHolder {
  TextView text;
  TextView timestamp;
  ImageView icon;
  ProgressBar progress;
  int position;
}
```

然後，在 Layout 的類中生成一個 ViewHolder 對象：

```java
ViewHolder holder = new ViewHolder();
holder.icon = (ImageView) convertView.findViewById(R.id.listitem_image);
holder.text = (TextView) convertView.findViewById(R.id.listitem_text);
holder.timestamp = (TextView) convertView.findViewById(R.id.listitem_timestamp);
holder.progress = (ProgressBar) convertView.findViewById(R.id.progress_spinner);
convertView.setTag(holder);
```

這樣你就可以輕鬆獲取每個視圖，而不是使用 `findViewById()` 來不斷查找子視圖，節省了寶貴的運算時間。
