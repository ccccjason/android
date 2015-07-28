# 打印自定義文檔

> 編寫:[jdneo](https://github.com/jdneo) - 原文:<http://developer.android.com/training/printing/custom-docs.html>

對於有些應用，比如繪圖應用，頁面佈局應用和其它一些關注於圖像輸出的應用，創造出精美的打印頁面將是它的核心功能。在這種情況下，僅僅打印一幅圖片或一個HTML文檔就不夠了。這類應用的打印輸出需要精確地控制每一個會在頁面中顯示的對象，包括字體，文本流，分頁符，頁眉，頁腳和一些圖像元素等等。

想要創建一個完全自定義的打印文檔，需要投入比之前討論的方法更多的編程精力。我們必須構建可以和打印框架相互通信的組件，調整打印參數，繪製頁面元素並管理多個頁面的打印。

這節課將展示如何連接打印管理器，創建一個打印適配器以及如何構建出需要打印的內容。

## 連接打印管理器

當我們的應用直接管理打印進程時，在收到來自用戶的打印請求後，第一步要做的是連接Android打印框架並獲取一個[PrintManager](http://developer.android.com/reference/android/print/PrintManager.html)類的實例。該類允許我們初始化一個打印任務並開始打印任務的生命週期。下面的代碼展示瞭如何獲得打印管理器並開始打印進程。

```java
private void doPrint() {
    // Get a PrintManager instance
    PrintManager printManager = (PrintManager) getActivity()
            .getSystemService(Context.PRINT_SERVICE);

    // Set job name, which will be displayed in the print queue
    String jobName = getActivity().getString(R.string.app_name) + " Document";

    // Start a print job, passing in a PrintDocumentAdapter implementation
    // to handle the generation of a print document
    printManager.print(jobName, new MyPrintDocumentAdapter(getActivity()),
            null); //
}
```

上面的代碼展示瞭如何命名一個打印任務以及如何設置一個[PrintDocumentAdapter](http://developer.android.com/reference/android/print/PrintDocumentAdapter.html)類的實例，它負責處理打印生命週期的每一步。打印適配器的實現會在下一節中進行討論。

> **Note：**<a href="http://developer.android.com/reference/android/print/PrintManager.html#print(java.lang.String, android.print.PrintDocumentAdapter, android.print.PrintAttributes)">print()</a>方法的最後一個參數接收一個[PrintAttributes](http://developer.android.com/reference/android/print/PrintAttributes.html)對象。我們可以使用這個參數向打印框架進行一些打印設置，以及基於前一個打印週期的預設，從而改善用戶體驗。我們也可以使用這個參數對打印內容進行一些更符合實際情況的設置，比如當打印一幅照片時，設置打印的方向與照片方向一致。

## 創建打印適配器

打印適配器負責與Android打印框架交互並處理打印過程的每一步。這個過程需要用戶在創建打印文檔前選擇打印機和打印選項。由於用戶可以選擇不同性能的打印機，不同的頁面尺寸或不同的頁面方向，因此這些選項可能會影響最終的打印效果。當這些選項配置好之後，打印框架會尋求適配器進行佈局並生成一個打印文檔，以此作為打印的前期準備。一旦用戶點擊了打印按鈕，框架會將最終的打印文檔傳遞給Print Provider進行打印輸出。在打印過程中，用戶可以選擇取消打印，所以打印適配器必須監聽並響應取消打印的請求。

[PrintDocumentAdapter](http://developer.android.com/reference/android/print/PrintDocumentAdapter.html)抽象類負責處理打印的生命週期，它有四個主要的回調方法。我們必須在打印適配器中實現這些方法，以此來正確地和Android打印框架進行交互：
* <a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.html#onStart()">onStart()</a>：一旦打印進程開始，該方法就將被調用。如果我們的應用有任何一次性的準備任務要執行，比如獲取一個要打印數據的快照，那麼讓它們在此處執行。在你的適配器中，這個回調方法不是必須實現的。
* <a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.html#onLayout(android.print.PrintAttributes, android.print.PrintAttributes, android.os.CancellationSignal, android.print.PrintDocumentAdapter.LayoutResultCallback, android.os.Bundle)">onLayout()</a>：每當用戶改變了影響打印輸出的設置時（比如改變了頁面的尺寸，或者頁面的方向）該函數將會被調用，以此給我們的應用一個機會去重新計算打印頁面的佈局。另外，該方法必須返回打印文檔包含多少頁面。
* <a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.html#onWrite(android.print.PageRange[], android.os.ParcelFileDescriptor, android.os.CancellationSignal, android.print.PrintDocumentAdapter.WriteResultCallback)">onWrite()</a>：該方法調用後，會將打印頁面渲染成一個待打印的文件。該方法可以在<a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.html#onLayout(android.print.PrintAttributes, android.print.PrintAttributes, android.os.CancellationSignal, android.print.PrintDocumentAdapter.LayoutResultCallback, android.os.Bundle)">onLayout()</a>方法被調用後調用一次或多次。
* <a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.html#onFinish()">onFinish()</a>：一旦打印進程結束後，該方法將會被調用。如果我們的應用有任何一次性銷燬任務要執行，讓這些任務在該方法內執行。這個回調方法不是必須實現的。

下面將介紹如何實現`onLayout()`以及`onWrite()`方法，他們是打印適配器的核心功能。

> **Note：**這些適配器的回調方法會在應用的主線程上被調用。如果這些方法的實現在執行時可能需要花費大量的時間，那麼應該將他們放在另一個線程裡執行。例如：我們可以將佈局或者寫入打印文檔的操作封裝在一個[AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html)對象中。

### 計算打印文檔信息

在實現[PrintDocumentAdapter](http://developer.android.com/reference/android/print/PrintDocumentAdapter.html)類時，我們的應用必須能夠指定出所創建文檔的類型，計算出打印任務所需要打印的總頁數，並提供打印頁面的尺寸信息。在實現適配器的<a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.html#onLayout(android.print.PrintAttributes, android.print.PrintAttributes, android.os.CancellationSignal, android.print.PrintDocumentAdapter.LayoutResultCallback, android.os.Bundle)">onLayout()</a>方法時，我們執行這些計算，並提供與理想的輸出相關的一些信息，這些信息可以在[PrintDocumentInfo](http://developer.android.com/reference/android/print/PrintDocumentInfo.html)類中獲取，包括頁數和內容類型。下面的例子展示了[PrintDocumentAdapter](http://developer.android.com/reference/android/print/PrintDocumentAdapter.html)中<a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.html#onLayout(android.print.PrintAttributes, android.print.PrintAttributes, android.os.CancellationSignal, android.print.PrintDocumentAdapter.LayoutResultCallback, android.os.Bundle)">onLayout()</a>方法的基本實現：

```java
@Override
public void onLayout(PrintAttributes oldAttributes,
                     PrintAttributes newAttributes,
                     CancellationSignal cancellationSignal,
                     LayoutResultCallback callback,
                     Bundle metadata) {
    // Create a new PdfDocument with the requested page attributes
    mPdfDocument = new PrintedPdfDocument(getActivity(), newAttributes);

    // Respond to cancellation request
    if (cancellationSignal.isCancelled() ) {
        callback.onLayoutCancelled();
        return;
    }

    // Compute the expected number of printed pages
    int pages = computePageCount(newAttributes);

    if (pages > 0) {
        // Return print information to print framework
        PrintDocumentInfo info = new PrintDocumentInfo
                .Builder("print_output.pdf")
                .setContentType(PrintDocumentInfo.CONTENT_TYPE_DOCUMENT)
                .setPageCount(pages);
                .build();
        // Content layout reflow is complete
        callback.onLayoutFinished(info, true);
    } else {
        // Otherwise report an error to the print framework
        callback.onLayoutFailed("Page count calculation failed.");
    }
}
```

<a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.html#onLayout(android.print.PrintAttributes, android.print.PrintAttributes, android.os.CancellationSignal, android.print.PrintDocumentAdapter.LayoutResultCallback, android.os.Bundle)">onLayout()</a>方法的執行結果有三種：完成，取消或失敗（計算佈局無法順利完成時會失敗）。我們必須通過調用[PrintDocumentAdapter.LayoutResultCallback](http://developer.android.com/reference/android/print/PrintDocumentAdapter.LayoutResultCallback.html)對象中的適當方法來指出這些結果中的一個。

> **Note：**<a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.LayoutResultCallback.html#onLayoutFinished(android.print.PrintDocumentInfo, boolean)">onLayoutFinished()</a>方法的布爾類型參數明確了這個佈局內容是否和上一次打印請求相比發生了改變。恰當地設定了這個參數將避免打印框架不必要地調用<a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.html#onWrite(android.print.PageRange[], android.os.ParcelFileDescriptor, android.os.CancellationSignal, android.print.PrintDocumentAdapter.WriteResultCallback)">onWrite()</a>方法，緩存之前的打印文檔，提升執行性能。

<a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.html#onLayout(android.print.PrintAttributes, android.print.PrintAttributes, android.os.CancellationSignal, android.print.PrintDocumentAdapter.LayoutResultCallback, android.os.Bundle)">onLayout()</a>的主要任務是計算打印文檔的頁數，並將它作為打印參數交給打印機。如何計算頁數則高度依賴於應用是如何對打印頁面進行佈局的。下面的代碼展示了頁數是如何根據打印方向確定的：

```java
private int computePageCount(PrintAttributes printAttributes) {
    int itemsPerPage = 4; // default item count for portrait mode

    MediaSize pageSize = printAttributes.getMediaSize();
    if (!pageSize.isPortrait()) {
        // Six items per page in landscape orientation
        itemsPerPage = 6;
    }

    // Determine number of print items
    int printItemCount = getPrintItemCount();

    return (int) Math.ceil(printItemCount / itemsPerPage);
}
```

### 將打印文檔寫入文件

當需要將打印內容輸出到一個文件時，Android打印框架會調用[PrintDocumentAdapter](http://developer.android.com/reference/android/print/PrintDocumentAdapter.html)類的<a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.html#onWrite(android.print.PageRange[], android.os.ParcelFileDescriptor, android.os.CancellationSignal, android.print.PrintDocumentAdapter.WriteResultCallback)">onWrite()</a>方法。這個方法的參數指定了哪些頁面要被寫入以及要使用的輸出文件。該方法的實現必須將每一個請求頁的內容渲染成一個含有多個頁面的PDF文件。當這個過程結束以後，你需要調用callback對象的<a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.WriteResultCallback.html#onWriteFinished(android.print.PageRange[])">onWriteFinished()</a>方法。

> **Note：** Android打印框架可能會在每次調用<a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.html#onLayout(android.print.PrintAttributes, android.print.PrintAttributes, android.os.CancellationSignal, android.print.PrintDocumentAdapter.LayoutResultCallback, android.os.Bundle)">onLayout()</a>後，調用<a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.html#onWrite(android.print.PageRange[], android.os.ParcelFileDescriptor, android.os.CancellationSignal, android.print.PrintDocumentAdapter.WriteResultCallback)">onWrite()</a>方法一次甚至更多次。請務必牢記：當打印內容的佈局沒有變化時，可以將<a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.LayoutResultCallback.html#onLayoutFinished(android.print.PrintDocumentInfo, boolean)">onLayoutFinished()</a>方法的布爾參數設置為“false”，以此避免對打印文檔進行不必要的重寫操作。

> **Note：**<a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.LayoutResultCallback.html#onLayoutFinished(android.print.PrintDocumentInfo, boolean)">onLayoutFinished()</a>方法的布爾類型參數明確了這個佈局內容是否和上一次打印請求相比發生了改變。恰當地設定了這個參數將避免打印框架不必要的調用<a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.html#onLayout(android.print.PrintAttributes, android.print.PrintAttributes, android.os.CancellationSignal, android.print.PrintDocumentAdapter.LayoutResultCallback, android.os.Bundle)">onLayout()</a>方法，緩存之前的打印文檔，提升執行性能。

下面的代碼展示了使用[PrintedPdfDocument](http://developer.android.com/reference/android/print/pdf/PrintedPdfDocument.html)類創建了PDF文件的基本原理：

```java
@Override
public void onWrite(final PageRange[] pageRanges,
                    final ParcelFileDescriptor destination,
                    final CancellationSignal cancellationSignal,
                    final WriteResultCallback callback) {
    // Iterate over each page of the document,
    // check if it's in the output range.
    for (int i = 0; i < totalPages; i++) {
        // Check to see if this page is in the output range.
        if (containsPage(pageRanges, i)) {
            // If so, add it to writtenPagesArray. writtenPagesArray.size()
            // is used to compute the next output page index.
            writtenPagesArray.append(writtenPagesArray.size(), i);
            PdfDocument.Page page = mPdfDocument.startPage(i);

            // check for cancellation
            if (cancellationSignal.isCancelled()) {
                callback.onWriteCancelled();
                mPdfDocument.close();
                mPdfDocument = null;
                return;
            }

            // Draw page content for printing
            drawPage(page);

            // Rendering is complete, so page can be finalized.
            mPdfDocument.finishPage(page);
        }
    }

    // Write PDF document to file
    try {
        mPdfDocument.writeTo(new FileOutputStream(
                destination.getFileDescriptor()));
    } catch (IOException e) {
        callback.onWriteFailed(e.toString());
        return;
    } finally {
        mPdfDocument.close();
        mPdfDocument = null;
    }
    PageRange[] writtenPages = computeWrittenPages();
    // Signal the print framework the document is complete
    callback.onWriteFinished(writtenPages);

    ...
}
```
代碼中將PDF頁面遞交給了drawPage()方法，這個方法會在下一部分介紹。

就佈局而言，<a href="http://developer.android.com/reference/android/print/PrintDocumentAdapter.html#onWrite(android.print.PageRange[], android.os.ParcelFileDescriptor, android.os.CancellationSignal, android.print.PrintDocumentAdapter.WriteResultCallback)">onWrite()</a>方法的執行可以有三種結果：完成，取消或者失敗（內容無法被寫入）。我們必須通過調用[PrintDocumentAdapter.WriteResultCallback](http://developer.android.com/reference/android/print/PrintDocumentAdapter.WriteResultCallback.html)對象中的適當方法來指明這些結果中的一個。

> **Note：**渲染打印文檔是一個可能耗費大量資源的操作。為了避免阻塞應用的主UI線程，我們應該考慮將頁面的渲染和寫操作放在另一個線程中執行，比如在[AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html)中執行。關於更多異步任務線程的知識，可以閱讀：[Processes and Threads](http://developer.android.com/guide/components/processes-and-threads.html)。

## 繪製PDF頁面內容

當我們的應用進行打印時，應用必須生成一個PDF文檔並將它傳遞給Android打印框架以進行打印。我們可以使用任何PDF生成庫來協助完成這個操作。本節將展示如何使用[PrintedPdfDocument](http://developer.android.com/reference/android/print/pdf/PrintedPdfDocument.html)類將打印內容生成為PDF頁面。

[PrintedPdfDocument](http://developer.android.com/reference/android/print/pdf/PrintedPdfDocument.html)類使用一個[Canvas](http://developer.android.com/reference/android/graphics/Canvas.html)對象來在PDF頁面上繪製元素，這一點和在activity佈局上進行繪製很類似。我們可以在打印頁面上使用[Canvas](http://developer.android.com/reference/android/graphics/Canvas.html)類提供的相關繪圖方法繪製頁面元素。下面的代碼展示瞭如何使用這些方法在PDF頁面上繪製一些簡單的元素：

```java
private void drawPage(PdfDocument.Page page) {
    Canvas canvas = page.getCanvas();

    // units are in points (1/72 of an inch)
    int titleBaseLine = 72;
    int leftMargin = 54;

    Paint paint = new Paint();
    paint.setColor(Color.BLACK);
    paint.setTextSize(36);
    canvas.drawText("Test Title", leftMargin, titleBaseLine, paint);

    paint.setTextSize(11);
    canvas.drawText("Test paragraph", leftMargin, titleBaseLine + 25, paint);

    paint.setColor(Color.BLUE);
    canvas.drawRect(100, 100, 172, 172, paint);
}
```

當使用[Canvas](http://developer.android.com/reference/android/graphics/Canvas.html)在一個PDF頁面上繪圖時，元素通過單位“點（point）”來指定大小，一個點相當於七十二分之一英寸。在編寫程序時，請確保使用該測量單位來指定頁面上的元素大小。在定位繪製的元素時，座標系的原點（即(0,0)點）在頁面的最左上角。

> **Tip：**雖然[Canvas](http://developer.android.com/reference/android/graphics/Canvas.html)對象允許我們將打印元素放置在一個PDF文檔的邊緣，但許多打印機無法在紙張的邊緣打印。所以當我們使用這個類構建一個打印文檔時，需要考慮到那些無法打印的邊緣區域。
