# 傳輸資源

> 編寫:[wly2014](https://github.com/wly2014) - 原文: <http://developer.android.com/training/wearables/data-layer/assets.html>

為了通過藍牙發送大量的二進制數據，比如圖片，要將一個[Asset](http://developer.android.com/reference/com/google/android/gms/wearable/Asset.html)附加到數據元上，並放入複製而來的數據庫中。

Assets 能夠自動地處理數據緩存以避免重複發送，保護藍牙帶寬。一般的模式是：手持設備下載圖像，將圖片壓縮到適合在可穿戴設備上顯示的大小，並以Asset傳給可穿戴設備。下面的例子演示此模式。

> **Note:** 儘管數據元的大小限制在100KB，但資源可以任意大。然而，傳輸大量資源會多方面地影響用戶體驗，因此，當傳輸大量資源時，要測試我們的應用以保證它有良好的用戶體驗。

## 傳輸資源

在Asset類中使用`creat..()`方法創建資源。下面，我們將一個bitmap轉化為字節流，然後調用[creatFromBytes()](http://developer.android.com/reference/com/google/android/gms/wearable/Asset.html#createFromBytes(byte[]))方法創建資源。

```java
private static Asset createAssetFromBitmap(Bitmap bitmap) {
    final ByteArrayOutputStream byteStream = new ByteArrayOutputStream();
    bitmap.compress(Bitmap.CompressFormat.PNG, 100, byteStream);
    return Asset.createFromBytes(byteStream.toByteArray());
}
```

創建資源後，使用 [DataMap](DataMap.html) 或者 [PutDataRepuest](PutDataRequest.html) 類中的 `putAsset()` 方法將其附加到數據元上，然後用 <a href="http://developer.android.com/reference/com/google/android/gms/wearable/DataApi.html#putDataItem(com.google.android.gms.common.api.GoogleApiClient, com.google.android.gms.wearable.PutDataRequest)">putDataItem()</a> 方法將數據元放入數據庫。

### 使用 PutDataRequest

```java
Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.image);
Asset asset = createAssetFromBitmap(bitmap);
PutDataRequest request = PutDataRequest.create("/image");
request.putAsset("profileImage", asset);
Wearable.DataApi.putDataItem(mGoogleApiClient, request);
```

### 使用 PutDataMapRequest

```java
Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.image);
Asset asset = createAssetFromBitmap(bitmap);
PutDataMapRequest dataMap = PutDataMapRequest.create("/image");
dataMap.getDataMap().putAsset("profileImage", asset)
PutDataRequest request = dataMap.asPutDataRequest();
PendingResult<DataApi.DataItemResult> pendingResult = Wearable.DataApi
        .putDataItem(mGoogleApiClient, request);
```

## 接收資源

創建資源後，我們可能需要在另一連接端讀取資源。以下是如何實現回調以發現資源變化和提取Asset對象。

```java
@Override
public void onDataChanged(DataEventBuffer dataEvents) {
  for (DataEvent event : dataEvents) {
    if (event.getType() == DataEvent.TYPE_CHANGED &&
        event.getDataItem().getUri().getPath().equals("/image")) {
      DataMapItem dataMapItem = DataMapItem.fromDataItem(event.getDataItem());
      Asset profileAsset = dataMapItem.getDataMap().getAsset("profileImage");
      Bitmap bitmap = loadBitmapFromAsset(profileAsset);
      // Do something with the bitmap
    }
  }
}

public Bitmap loadBitmapFromAsset(Asset asset) {
    if (asset == null) {
        throw new IllegalArgumentException("Asset must be non-null");
    }
    ConnectionResult result =
           mGoogleApiClient.blockingConnect(TIMEOUT_MS, TimeUnit.MILLISECONDS);
    if (!result.isSuccess()) {
        return null;
    }
    // convert asset into a file descriptor and block until it's ready
    InputStream assetInputStream = Wearable.DataApi.getFdForAsset(
            mGoogleApiClient, asset).await().getInputStream();
            mGoogleApiClient.disconnect();

    if (assetInputStream == null) {
        Log.w(TAG, "Requested an unknown Asset.");
        return null;
    }
    // decode the stream into a bitmap
    return BitmapFactory.decodeStream(assetInputStream);
}
```



