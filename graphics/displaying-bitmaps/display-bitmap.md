# 在UI上顯示Bitmap

> 編寫:[kesenhoo](https://github.com/kesenhoo) - 原文:<http://developer.android.com/training/displaying-bitmaps/display-bitmap.html>

這一課會演示如何運用前面幾節課的內容，使用後臺線程與緩存機制將圖片加載到ViewPager與GridView控件，並且學習處理併發與配置改變問題。

## 實現加載圖片到ViewPager

[Swipe View Pattern](http://developer.android.com/design/patterns/swipe-views.html)是一個使用滑動來切換顯示不同詳情頁面的設計模型。（關於這種效果請先參看[Android Design: Swipe Views](http://developer.android.com/design/patterns/swipe-views.html)）。我們可以通過[PagerAdapter](http://developer.android.com/reference/android/support/v4/view/PagerAdapter.html)與[ViewPager](http://developer.android.com/reference/android/support/v4/view/ViewPager.html)控件來實現這個效果。 不過，一個更加合適的Adapter是PagerAdapter的一個子類，叫做[FragmentStatePagerAdapter](http://developer.android.com/reference/android/support/v4/app/FragmentStatePagerAdapter.html)：它可以在某個ViewPager中的子視圖切換出屏幕時自動銷燬與保存Fragments的狀態。這樣能夠保持更少的內存消耗。

> **Note:** 如果只有為數不多的圖片並且確保不會超出程序內存限制，那麼使用PagerAdapter或 FragmentPagerAdapter會更加合適。

下面是一個使用ViewPager與ImageView作為子視圖的示例。主Activity包含有ViewPager和Adapter。

```java
public class ImageDetailActivity extends FragmentActivity {
    public static final String EXTRA_IMAGE = "extra_image";

    private ImagePagerAdapter mAdapter;
    private ViewPager mPager;

    // A static dataset to back the ViewPager adapter
    public final static Integer[] imageResIds = new Integer[] {
            R.drawable.sample_image_1, R.drawable.sample_image_2, R.drawable.sample_image_3,
            R.drawable.sample_image_4, R.drawable.sample_image_5, R.drawable.sample_image_6,
            R.drawable.sample_image_7, R.drawable.sample_image_8, R.drawable.sample_image_9};

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.image_detail_pager); // Contains just a ViewPager

        mAdapter = new ImagePagerAdapter(getSupportFragmentManager(), imageResIds.length);
        mPager = (ViewPager) findViewById(R.id.pager);
        mPager.setAdapter(mAdapter);
    }

    public static class ImagePagerAdapter extends FragmentStatePagerAdapter {
        private final int mSize;

        public ImagePagerAdapter(FragmentManager fm, int size) {
            super(fm);
            mSize = size;
        }

        @Override
        public int getCount() {
            return mSize;
        }

        @Override
        public Fragment getItem(int position) {
            return ImageDetailFragment.newInstance(position);
        }
    }
}
```

Fragment裡面包含了ImageView控件:

```java
public class ImageDetailFragment extends Fragment {
    private static final String IMAGE_DATA_EXTRA = "resId";
    private int mImageNum;
    private ImageView mImageView;

    static ImageDetailFragment newInstance(int imageNum) {
        final ImageDetailFragment f = new ImageDetailFragment();
        final Bundle args = new Bundle();
        args.putInt(IMAGE_DATA_EXTRA, imageNum);
        f.setArguments(args);
        return f;
    }

    // Empty constructor, required as per Fragment docs
    public ImageDetailFragment() {}

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mImageNum = getArguments() != null ? getArguments().getInt(IMAGE_DATA_EXTRA) : -1;
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
            Bundle savedInstanceState) {
        // image_detail_fragment.xml contains just an ImageView
        final View v = inflater.inflate(R.layout.image_detail_fragment, container, false);
        mImageView = (ImageView) v.findViewById(R.id.imageView);
        return v;
    }

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        final int resId = ImageDetailActivity.imageResIds[mImageNum];
        mImageView.setImageResource(resId); // Load image into ImageView
    }
}
```

希望你有發現上面示例存在的問題：在UI線程中讀取圖片可能會導致應用無響應。因此使用在第二課中學習的AsyncTask會更好。

```java
public class ImageDetailActivity extends FragmentActivity {
    ...

    public void loadBitmap(int resId, ImageView imageView) {
        mImageView.setImageResource(R.drawable.image_placeholder);
        BitmapWorkerTask task = new BitmapWorkerTask(mImageView);
        task.execute(resId);
    }

    ... // include BitmapWorkerTask class
}

public class ImageDetailFragment extends Fragment {
    ...

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        if (ImageDetailActivity.class.isInstance(getActivity())) {
            final int resId = ImageDetailActivity.imageResIds[mImageNum];
            // Call out to ImageDetailActivity to load the bitmap in a background thread
            ((ImageDetailActivity) getActivity()).loadBitmap(resId, mImageView);
        }
    }
}
```

在BitmapWorkerTask中做一些例如重設圖片大小，從網絡拉取圖片的任務，可以確保不會阻塞UI線程。如果後臺線程不僅僅是一個簡單的加載操作，增加一個內存緩存或者磁盤緩存會比較好（請參考第三課：緩存Bitmap），下面是一些為了內存緩存而附加的內容：

```java
public class ImageDetailActivity extends FragmentActivity {
    ...
    private LruCache mMemoryCache;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        ...
        // initialize LruCache as per Use a Memory Cache section
    }

    public void loadBitmap(int resId, ImageView imageView) {
        final String imageKey = String.valueOf(resId);

        final Bitmap bitmap = mMemoryCache.get(imageKey);
        if (bitmap != null) {
            mImageView.setImageBitmap(bitmap);
        } else {
            mImageView.setImageResource(R.drawable.image_placeholder);
            BitmapWorkerTask task = new BitmapWorkerTask(mImageView);
            task.execute(resId);
        }
    }

    ... // include updated BitmapWorkerTask from Use a Memory Cache section
}
```

把前面學習到的所有技巧合並起來，我們將得到一個響應性良好的ViewPager實現：它擁有最小的加載延遲，同時可以根據實際需求執行不同的後臺處理任務。

## 實現加載圖片到GridView

[Grid List Building Block](http://developer.android.com/design/building-blocks/grid-lists.html)是一種有效顯示大量圖片的方式。它能夠一次顯示許多圖片，同時即將被顯示的圖片會處於準備顯示的狀態。如果我們想要實現這種效果，必須確保UI是流暢的，能夠控制內存使用，並且正確處理併發問題（因為GridView會循環使用子視圖)。

下面是一個典型的使用場景，在Fragment裡面內置GridView，其中GridView的子視圖是ImageView：

```java
public class ImageGridFragment extends Fragment implements AdapterView.OnItemClickListener {
    private ImageAdapter mAdapter;

    // A static dataset to back the GridView adapter
    public final static Integer[] imageResIds = new Integer[] {
            R.drawable.sample_image_1, R.drawable.sample_image_2, R.drawable.sample_image_3,
            R.drawable.sample_image_4, R.drawable.sample_image_5, R.drawable.sample_image_6,
            R.drawable.sample_image_7, R.drawable.sample_image_8, R.drawable.sample_image_9};

    // Empty constructor as per Fragment docs
    public ImageGridFragment() {}

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mAdapter = new ImageAdapter(getActivity());
    }

    @Override
    public View onCreateView(
            LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        final View v = inflater.inflate(R.layout.image_grid_fragment, container, false);
        final GridView mGridView = (GridView) v.findViewById(R.id.gridView);
        mGridView.setAdapter(mAdapter);
        mGridView.setOnItemClickListener(this);
        return v;
    }

    @Override
    public void onItemClick(AdapterView parent, View v, int position, long id) {
        final Intent i = new Intent(getActivity(), ImageDetailActivity.class);
        i.putExtra(ImageDetailActivity.EXTRA_IMAGE, position);
        startActivity(i);
    }

    private class ImageAdapter extends BaseAdapter {
        private final Context mContext;

        public ImageAdapter(Context context) {
            super();
            mContext = context;
        }

        @Override
        public int getCount() {
            return imageResIds.length;
        }

        @Override
        public Object getItem(int position) {
            return imageResIds[position];
        }

        @Override
        public long getItemId(int position) {
            return position;
        }

        @Override
        public View getView(int position, View convertView, ViewGroup container) {
            ImageView imageView;
            if (convertView == null) { // if it's not recycled, initialize some attributes
                imageView = new ImageView(mContext);
                imageView.setScaleType(ImageView.ScaleType.CENTER_CROP);
                imageView.setLayoutParams(new GridView.LayoutParams(
                        LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT));
            } else {
                imageView = (ImageView) convertView;
            }
        //請注意下面的代碼
        imageView.setImageResource(imageResIds[position]); // Load image into ImageView
        return imageView;
    }
}
```

這裡同樣有一個問題，上面的代碼實現中，犯了把圖片加載放在UI線程進行處理的錯誤。如果只是加載一些很小的圖片，或者是經過Android系統縮放並緩存過的圖片，上面的代碼在運行時不會有太大問題，但是如果加載的圖片稍微複雜耗時一點，這都會導致你的UI卡頓甚至應用無響應。

與前面加載圖片到ViewPager一樣，如果`setImageResource`的操作會比較耗時，也有可能會阻塞UI線程。不過我們可以使用類似前面異步處理圖片與增加緩存的方法來解決這個問題。然而，我們還需要考慮GridView的循環機制所帶來的併發問題。為了處理這個問題，可以參考前面的課程 。下面是一個更新過後的解決方案：

```java
public class ImageGridFragment extends Fragment implements AdapterView.OnItemClickListener {
    ...

    private class ImageAdapter extends BaseAdapter {
        ...

        @Override
        public View getView(int position, View convertView, ViewGroup container) {
            ...
            loadBitmap(imageResIds[position], imageView)
            return imageView;
        }
    }

    public void loadBitmap(int resId, ImageView imageView) {
        if (cancelPotentialWork(resId, imageView)) {
            final BitmapWorkerTask task = new BitmapWorkerTask(imageView);
            final AsyncDrawable asyncDrawable =
                    new AsyncDrawable(getResources(), mPlaceHolderBitmap, task);
            imageView.setImageDrawable(asyncDrawable);
            task.execute(resId);
        }
    }

    static class AsyncDrawable extends BitmapDrawable {
        private final WeakReference bitmapWorkerTaskReference;

        public AsyncDrawable(Resources res, Bitmap bitmap,
                BitmapWorkerTask bitmapWorkerTask) {
            super(res, bitmap);
            bitmapWorkerTaskReference =
                new WeakReference(bitmapWorkerTask);
        }

        public BitmapWorkerTask getBitmapWorkerTask() {
            return bitmapWorkerTaskReference.get();
        }
    }

    public static boolean cancelPotentialWork(int data, ImageView imageView) {
        final BitmapWorkerTask bitmapWorkerTask = getBitmapWorkerTask(imageView);

        if (bitmapWorkerTask != null) {
            final int bitmapData = bitmapWorkerTask.data;
            if (bitmapData != data) {
                // Cancel previous task
                bitmapWorkerTask.cancel(true);
            } else {
                // The same work is already in progress
                return false;
            }
        }
        // No task associated with the ImageView, or an existing task was cancelled
        return true;
    }

    private static BitmapWorkerTask getBitmapWorkerTask(ImageView imageView) {
       if (imageView != null) {
           final Drawable drawable = imageView.getDrawable();
           if (drawable instanceof AsyncDrawable) {
               final AsyncDrawable asyncDrawable = (AsyncDrawable) drawable;
               return asyncDrawable.getBitmapWorkerTask();
           }
        }
        return null;
    }

    ... // include updated BitmapWorkerTask class
```
> **Note:**對於ListView同樣可以套用上面的方法。

上面的方法提供了足夠的彈性，使得我們可以做從網絡下載圖片，並對大尺寸大的數碼照片做縮放等操作而不至於阻塞UI線程。
