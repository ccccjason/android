# 創建List

> 編寫: [roya](https://github.com/RoyaAoki) 原文:<https://developer.android.com/training/wearables/ui/lists.html>

List讓用戶在可穿戴設備上很容易地從一組選項中選擇一個項目。這個課程介紹瞭如何在Android Wear應用中創建List。

Wearable UI庫包含了`WearableListView`類，該類是對可穿戴設備進行優化的List實現。

> **Note:** Android SDK 中的`Notifications`例子示範瞭如何在應用中使用 `WearableListView`。這個例子的位於`android-sdk/samples/android-20/wearable/Notifications`目錄。

為了在Android Wear應用中創建List，我們需要:

1.  添加`WearableListView`元素到activity的layout定義中。
2.  為List選項創建一個自定義的layout實現。
3.  使用這個實現為List選項創建一個layout定義文件。
4.  創建一個adapter以填充List。
5.  指定這個adapter到`WearableListView`元素。

下面的章節有這些步驟的詳細描述。

![](06_uilib.png)

**Figure 3:** 在Android Wear上的List View.

## 添加List View

下面的layout使用`BoxInsetLayout`添加了一個List view到activity中，所以這個List可以正確地顯示在圓形和方形兩種設備上：

```xml
<android.support.wearable.view.BoxInsetLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:background="@drawable/robot_background"
    android:layout_height="match_parent"
    android:layout_width="match_parent">

    <FrameLayout
        android:id="@+id/frame_layout"
        android:layout_height="match_parent"
        android:layout_width="match_parent"
        app:layout_box="left|bottom|right">

        <android.support.wearable.view.WearableListView
            android:id="@+id/wearable_list"
            android:layout_height="match_parent"
            android:layout_width="match_parent">
        </android.support.wearable.view.WearableListView>
    </FrameLayout>
</android.support.wearable.view.BoxInsetLayout>
```
	
## 為List選項創建一個Layou實現

在許多例子中，每個List選項都由一個圖標和一個描述組成。Android SDK中的*Notifications* 例子實現了一個自定義layout：繼承[LinearLayout](https://developer.android.com/reference/android/widget/LinearLayout.html)以合併兩元素到每個List選項。這個layout也實現了 `WearableListView.OnCenterProximityListener`接口裡的方法，以實現在用戶在List中滾動時，因`WearableListView`的事件而改變選項圖標顏色和漸隱文字：

```java
public class WearableListItemLayout extends LinearLayout
             implements WearableListView.OnCenterProximityListener {

    private ImageView mCircle;
    private TextView mName;

    private final float mFadedTextAlpha;
    private final int mFadedCircleColor;
    private final int mChosenCircleColor;

    public WearableListItemLayout(Context context) {
        this(context, null);
    }

    public WearableListItemLayout(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public WearableListItemLayout(Context context, AttributeSet attrs,
                                  int defStyle) {
        super(context, attrs, defStyle);

        mFadedTextAlpha = getResources()
                         .getInteger(R.integer.action_text_faded_alpha) / 100f;
        mFadedCircleColor = getResources().getColor(R.color.grey);
        mChosenCircleColor = getResources().getColor(R.color.blue);
    }

    // Get references to the icon and text in the item layout definition
    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        // These are defined in the layout file for list items
        // (see next section)
        mCircle = (ImageView) findViewById(R.id.circle);
        mName = (TextView) findViewById(R.id.name);
    }

    @Override
    public void onCenterPosition(boolean animate) {
        mName.setAlpha(1f);
        ((GradientDrawable) mCircle.getDrawable()).setColor(mChosenCircleColor);
    }

    @Override
    public void onNonCenterPosition(boolean animate) {
        ((GradientDrawable) mCircle.getDrawable()).setColor(mFadedCircleColor);
        mName.setAlpha(mFadedTextAlpha);
    }
}
```

我們也可以創建animator對象以放大List中間選項的圖標。我們可以使用`WearableListView.OnCenterProximityListener`接口的`onCenterPosition()`和  `onNonCenterPosition()`回調方法來管理animator對象。更多關於animator對象的信息請查看[Animating with ObjectAnimator](https://developer.android.com/guide/topics/graphics/prop-animation.html#object-animator)

##為Items創建Layout解釋

在為List選項實現自定義layout之後，我們需要提供一個layout解釋文件以具體說明list item中的組件參數。下面的layout使用先前的自定義layout實現，並且定義圖標和文本view，這兩個view的ID對應layout實現類的ID：

`res/layout/list_item.xml`

```xml
<com.example.android.support.wearable.notifications.WearableListItemLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:gravity="center_vertical"
    android:layout_width="match_parent"
    android:layout_height="80dp">
    <ImageView
        android:id="@+id/circle"
        android:layout_height="20dp"
        android:layout_margin="16dp"
        android:layout_width="20dp"
        android:src="@drawable/wl_circle"/>
    <TextView
        android:id="@+id/name"
        android:gravity="center_vertical|left"
        android:layout_width="wrap_content"
        android:layout_marginRight="16dp"
        android:layout_height="match_parent"
        android:fontFamily="sans-serif-condensed-light"
        android:lineSpacingExtra="-4sp"
        android:textColor="@color/text_color"
        android:textSize="16sp"/>
</com.example.android.support.wearable.notifications.WearableListItemLayout>
```
	
## 創建Adapter以填充List

Adapter用內容填充`WearableListView`。下面的adapter基於strings數組元素填充了List：

```java
private static final class Adapter extends WearableListView.Adapter {
    private String[] mDataset;
    private final Context mContext;
    private final LayoutInflater mInflater;

    // Provide a suitable constructor (depends on the kind of dataset)
    public Adapter(Context context, String[] dataset) {
        mContext = context;
        mInflater = LayoutInflater.from(context);
        mDataset = dataset;
    }

    // Provide a reference to the type of views you're using
    public static class ItemViewHolder extends WearableListView.ViewHolder {
        private TextView textView;
        public ItemViewHolder(View itemView) {
            super(itemView);
            // find the text view within the custom item's layout
            textView = (TextView) itemView.findViewById(R.id.name);
        }
    }

    // Create new views for list items
    // (invoked by the WearableListView's layout manager)
    @Override
    public WearableListView.ViewHolder onCreateViewHolder(ViewGroup parent,
                                                          int viewType) {
        // Inflate our custom layout for list items
        return new ItemViewHolder(mInflater.inflate(R.layout.list_item, null));
    }

    // Replace the contents of a list item
    // Instead of creating new views, the list tries to recycle existing ones
    // (invoked by the WearableListView's layout manager)
    @Override
    public void onBindViewHolder(WearableListView.ViewHolder holder,
                                 int position) {
        // retrieve the text view
        ItemViewHolder itemHolder = (ItemViewHolder) holder;
        TextView view = itemHolder.textView;
        // replace text contents
        view.setText(mDataset[position]);
        // replace list item's metadata
        holder.itemView.setTag(position);
    }

    // Return the size of your dataset
    // (invoked by the WearableListView's layout manager)
    @Override
    public int getItemCount() {
        return mDataset.length;
    }
}
```

## 連接Adapter和設置Click Listener

在我們的activity中，從layout中取得`WearableListView`元素的引用，分配一個adapter實例以填充List，然後設置一個click listener以完成當用戶選擇了一個特定的List選項的動作。

```java
public class WearActivity extends Activity
                          implements WearableListView.ClickListener {

    // Sample dataset for the list
    String[] elements = { "List Item 1", "List Item 2", ... };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.my_list_activity);

        // Get the list component from the layout of the activity
        WearableListView listView =
            (WearableListView) findViewById(R.id.wearable_list);

        // Assign an adapter to the list
        listView.setAdapter(new Adapter(this, elements));

        // Set a click listener
        listView.setClickListener(this);
    }

    // WearableListView click listener
    @Override
    public void onClick(WearableListView.ViewHolder v) {
        Integer tag = (Integer) v.itemView.getTag();
        // use this data to complete some action ...
    }

    @Override
    public void onTopEmptyRegionClick() {
    }
}
```