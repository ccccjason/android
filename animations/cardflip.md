# 展示Card翻轉動畫

> 編寫:[XizhiXu](https://github.com/XizhiXu) - 原文:<http://developer.android.com/training/animation/cardflip.html>

這節課展示如何使用自定義Fragment動畫實現Card翻轉動畫。Card翻轉動畫通過模擬Card翻轉的效果實現view內容的切換。

下面是card翻轉動畫的樣子：

<div style="
  background: transparent url(device_galaxynexus_blank_land_span8.png) no-repeat
scroll top left; padding: 26px 68px 38px 72px; overflow: hidden;">

<video style="width: 320px; height: 180px;" controls="" autoplay="">
    <source src="anim_card_flip.mp4" type="video/mp4">
    <source src="anim_card_flip.webm" type="video/webm">
    <source src="anim_card_flip.ogv" type="video/ogg">
</video>

</div>

如果你想直接查看整個例子，[下載](http://developer.android.com/shareables/training/Animations.zip)並運行App樣例然後選擇Card翻轉例子。查看下列文件中的代碼實現：

* `src/CardFlipActivity.java`
* `animator/card_flip_right_in.xml`
* `animator/card_flip_right_out.xml`
* `animator/card_flip_left_in.xml`
* `animator/card_flip_left_out.xml`
* `layout/fragment_card_back.xml`
* `layout/fragment_card_front.xml`

## 創建Animator

創建Card翻轉動畫，我們需要兩個Animator。一個讓正面的card的右側向左翻轉漸出，一個讓背面的Card向右翻轉漸入。我們還需要兩個 Animator讓背面的card的右側向左翻轉漸入，一個讓向右翻轉漸入。

**card_flip_left_in.xml**

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Before rotating, immediately set the alpha to 0. -->
    <objectAnimator
        android:valueFrom="1.0"
        android:valueTo="0.0"
        android:propertyName="alpha"
        android:duration="0" />

    <!-- Rotate. -->
    <objectAnimator
        android:valueFrom="-180"
        android:valueTo="0"
        android:propertyName="rotationY"
        android:interpolator="@android:interpolator/accelerate_decelerate"
        android:duration="@integer/card_flip_time_full" />

    <!-- Half-way through the rotation (see startOffset), set the alpha to 1. -->
    <objectAnimator
        android:valueFrom="0.0"
        android:valueTo="1.0"
        android:propertyName="alpha"
        android:startOffset="@integer/card_flip_time_half"
        android:duration="1" />
</set>
```

**card_flip_left_out.xml**

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Rotate. -->
    <objectAnimator
        android:valueFrom="0"
        android:valueTo="180"
        android:propertyName="rotationY"
        android:interpolator="@android:interpolator/accelerate_decelerate"
        android:duration="@integer/card_flip_time_full" />

    <!-- Half-way through the rotation (see startOffset), set the alpha to 0. -->
    <objectAnimator
        android:valueFrom="1.0"
        android:valueTo="0.0"
        android:propertyName="alpha"
        android:startOffset="@integer/card_flip_time_half"
        android:duration="1" />
</set>
```

**card_flip_right_in.xml**

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Before rotating, immediately set the alpha to 0. -->
    <objectAnimator
        android:valueFrom="1.0"
        android:valueTo="0.0"
        android:propertyName="alpha"
        android:duration="0" />

    <!-- Rotate. -->
    <objectAnimator
        android:valueFrom="180"
        android:valueTo="0"
        android:propertyName="rotationY"
        android:interpolator="@android:interpolator/accelerate_decelerate"
        android:duration="@integer/card_flip_time_full" />

    <!-- Half-way through the rotation (see startOffset), set the alpha to 1. -->
    <objectAnimator
        android:valueFrom="0.0"
        android:valueTo="1.0"
        android:propertyName="alpha"
        android:startOffset="@integer/card_flip_time_half"
        android:duration="1" />
</set>
```

**card_flip_right_out.xml**

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Rotate. -->
    <objectAnimator
        android:valueFrom="0"
        android:valueTo="-180"
        android:propertyName="rotationY"
        android:interpolator="@android:interpolator/accelerate_decelerate"
        android:duration="@integer/card_flip_time_full" />

    <!-- Half-way through the rotation (see startOffset), set the alpha to 0. -->
    <objectAnimator
        android:valueFrom="1.0"
        android:valueTo="0.0"
        android:propertyName="alpha"
        android:startOffset="@integer/card_flip_time_half"
        android:duration="1" />
</set>
```

## 創建View

Card的每一面是一個獨立的佈局，比如兩屏文字，兩張圖片，或者任何View的組合。然後我們將在應用動畫的Fragment裡面用到這兩個佈局。下面的佈局創建了展示文本Card的一面：

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="#a6c"
    android:padding="16dp"
    android:gravity="bottom">

    <TextView android:id="@android:id/text1"
        style="?android:textAppearanceLarge"
        android:textStyle="bold"
        android:textColor="#fff"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/card_back_title" />

    <TextView style="?android:textAppearanceSmall"
        android:textAllCaps="true"
        android:textColor="#80ffffff"
        android:textStyle="bold"
        android:lineSpacingMultiplier="1.2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/card_back_description" />

</LinearLayout>
```

Card另一面顯示一個 [`ImageView`](http://developer.android.com/reference/android/widget/ImageView.html)：

```xml
<ImageView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:src="@drawable/image1"
    android:scaleType="centerCrop"
    android:contentDescription="@string/description_image_1" />
```

## 創建Fragment

為Card正反面創建Fragment，這些類從<a href="http://developer.android.com/reference/android/app/Fragment.html#onCreateView(android.view.LayoutInflater, android.view.ViewGroup, android.os.Bundle)"> `onCreateView()` </a>方法中分別為每個Framgent返回你之前創建的佈局。在想要顯示Card的父Activity中，我們可以創建對應的Fragment 實例。下面的例子展示父Activity內嵌套的Fragment：

```java
public class CardFlipActivity extends Activity {
    ...
    /**
     * A fragment representing the front of the card.
     */
    public class CardFrontFragment extends Fragment {
        @Override
        public View onCreateView(LayoutInflater inflater, ViewGroup container,
                Bundle savedInstanceState) {
            return inflater.inflate(R.layout.fragment_card_front, container, false);
        }
    }

    /**
     * A fragment representing the back of the card.
     */
    public class CardBackFragment extends Fragment {
        @Override
        public View onCreateView(LayoutInflater inflater, ViewGroup container,
                Bundle savedInstanceState) {
            return inflater.inflate(R.layout.fragment_card_back, container, false);
        }
    }
}
```

## 應用card翻轉動畫

現在，我們需要在父Activity中展示Fragment。為此，首先創建Activity的佈局。下面例子創建了一個可以在運行時添加Fragment的 [`FrameLayout`](http://developer.android.com/reference/android/widget/FrameLayout.html)。

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```
在Activity代碼中，把先前創建的佈局設置成其ContentVew。當Activity創建時展示一個默認的Fragment是個不錯的注意。所以下面的Activity樣例表明瞭如何默認顯示卡片正面：

```java
public class CardFlipActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_activity_card_flip);

        if (savedInstanceState == null) {
            getFragmentManager()
                    .beginTransaction()
                    .add(R.id.container, new CardFrontFragment())
                    .commit();
        }
    }
    ...
}
```

既然現在顯示了卡片的正面，我們可以在合適時機用翻轉動畫顯示卡片背面了。創建一個方法來顯示背面，它需要做下面這些事情：

* 將Fragment轉換設置我們剛做的自定義動畫

* 用新Fragment替換當前顯示的Fragment，並且應用之前創建的動畫到該事件中。

* 添加之前顯示的Fragment到Fragment的回退棧（back stack）中，所以當用戶按下 *Back* 鍵時，Card會翻轉回來。

```java
private void flipCard() {
    if (mShowingBack) {
        getFragmentManager().popBackStack();
        return;
    }

    // Flip to the back.

    mShowingBack = true;

    // Create and commit a new fragment transaction that adds the fragment for the back of
    // the card, uses custom animations, and is part of the fragment manager's back stack.

    getFragmentManager()
            .beginTransaction()

            // Replace the default fragment animations with animator resources representing
            // rotations when switching to the back of the card, as well as animator
            // resources representing rotations when flipping back to the front (e.g. when
            // the system Back button is pressed).
            .setCustomAnimations(
                    R.animator.card_flip_right_in, R.animator.card_flip_right_out,
                    R.animator.card_flip_left_in, R.animator.card_flip_left_out)

            // Replace any fragments currently in the container view with a fragment
            // representing the next page (indicated by the just-incremented currentPage
            // variable).
            .replace(R.id.container, new CardBackFragment())

            // Add this transaction to the back stack, allowing users to press Back
            // to get to the front of the card.
            .addToBackStack(null)

            // Commit the transaction.
            .commit();
}
```
