---
name: android-xml-ui-polish
description: >
  Use this skill to make Android XML View-based interfaces feel polished.
  Covers typography (TextAppearance, custom fonts), StateListDrawable,
  ripple effects, ObjectAnimator / ViewPropertyAnimator, TransitionManager,
  GradientDrawable corner radius (concentric rule), elevation/shadow,
  RecyclerView stagger, ShapeableImageView, MotionLayout basics,
  and optical alignment — all without Jetpack Compose.
metadata:
  author: Alimardon / Cryon
  last-updated: '2026-06-26'
  keywords:
    - Android XML
    - View System
    - UI Polish
    - ObjectAnimator
    - ViewPropertyAnimator
    - TransitionManager
    - StateListDrawable
    - Ripple
    - ShapeableImageView
    - MotionLayout
    - RecyclerView
    - Stagger
---

## When to Use

Use this skill when working with the traditional Android View system (XML
layouts, not Compose). App uses `<LinearLayout>`, `<ConstraintLayout>`,
`<RecyclerView>`, `<TextView>`, `<ImageView>` etc. in `.xml` layout files.

---

## 1. Typography Polish

### Custom fonts (res/font/)

```xml
<!-- res/font/inter_regular.ttf, inter_medium.ttf, inter_bold.ttf -->

<!-- res/values/fonts.xml -->
<font-family xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <font
        app:fontStyle="normal"
        app:fontWeight="400"
        app:font="@font/inter_regular" />
    <font
        app:fontStyle="normal"
        app:fontWeight="500"
        app:font="@font/inter_medium" />
    <font
        app:fontStyle="normal"
        app:fontWeight="700"
        app:font="@font/inter_bold" />
</font-family>
```

### TextAppearance styles

```xml
<!-- res/values/type.xml -->
<resources>
    <!-- Heading -->
    <style name="TextAppearance.App.Heading" parent="TextAppearance.Material3.HeadlineMedium">
        <item name="android:fontFamily">@font/inter</item>
        <item name="android:textSize">28sp</item>
        <item name="android:letterSpacing">-0.02</item>    <!-- negative for large -->
        <item name="android:lineSpacingMultiplier">1.2</item>
    </style>

    <!-- Body -->
    <style name="TextAppearance.App.Body" parent="TextAppearance.Material3.BodyMedium">
        <item name="android:fontFamily">@font/inter</item>
        <item name="android:textSize">16sp</item>
        <item name="android:letterSpacing">0.01</item>
        <item name="android:lineSpacingMultiplier">1.5</item>
    </style>

    <!-- Caption -->
    <style name="TextAppearance.App.Caption" parent="TextAppearance.Material3.LabelSmall">
        <item name="android:fontFamily">@font/inter</item>
        <item name="android:textSize">12sp</item>
        <item name="android:letterSpacing">0.03</item>
        <item name="android:lineSpacingMultiplier">1.4</item>
    </style>
</resources>
```

```xml
<!-- Layout usage -->
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:textAppearance="@style/TextAppearance.App.Body"
    android:text="Hello World" />
```

### SpannableString (programmatic)

```kotlin
val spannable = SpannableString("Hello World")
spannable.setSpan(
    LetterSpacingSpan(0.05f), 0, spannable.length,
    Spannable.SPAN_EXCLUSIVE_EXCLUSIVE
)
textView.text = spannable

// Custom LetterSpacingSpan
class LetterSpacingSpan(private val spacing: Float) : MetricAffectingSpan() {
    override fun updateMeasureState(paint: TextPaint) {
        paint.letterSpacing = spacing
    }
    override fun updateDrawState(paint: TextPaint) {
        paint.letterSpacing = spacing
    }
}
```

---

## 2. Ripple & Press States

### Material ripple (recommended)

```xml
<!-- Button / clickable view -->
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:background="?attr/selectableItemBackground"   <!-- bounded ripple -->
    android:clickable="true"
    android:focusable="true"
    android:padding="12dp"
    android:text="Click me" />

<!-- Borderless ripple (icon buttons) -->
android:background="?attr/selectableItemBackgroundBorderless"
```

### Custom colored ripple

```xml
<!-- res/drawable/ripple_primary.xml -->
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="@color/ripple_color">
    <item android:id="@android:id/mask">
        <shape android:shape="rectangle">
            <corners android:radius="12dp" />
            <solid android:color="#FFFFFF" />
        </shape>
    </item>
    <item>
        <shape android:shape="rectangle">
            <corners android:radius="12dp" />
            <solid android:color="@color/button_background" />
        </shape>
    </item>
</ripple>
```

```xml
<!-- res/values/colors.xml -->
<color name="ripple_color">#1A6650A4</color>  <!-- primary color @ 10% alpha -->
```

### StateListDrawable (custom press states)

```xml
<!-- res/drawable/selector_button.xml -->
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true">
        <shape android:shape="rectangle">
            <corners android:radius="12dp" />
            <solid android:color="@color/primary_pressed" />  <!-- darker -->
        </shape>
    </item>
    <item android:state_focused="true">
        <shape android:shape="rectangle">
            <corners android:radius="12dp" />
            <solid android:color="@color/primary_focused" />
        </shape>
    </item>
    <item>
        <shape android:shape="rectangle">
            <corners android:radius="12dp" />
            <solid android:color="@color/primary" />
        </shape>
    </item>
</selector>
```

### Scale on press (code)

```kotlin
fun View.addPressScale(pressedScale: Float = 0.95f) {
    setOnTouchListener { _, event ->
        when (event.action) {
            MotionEvent.ACTION_DOWN -> {
                animate().scaleX(pressedScale).scaleY(pressedScale)
                    .setDuration(100)
                    .setInterpolator(DecelerateInterpolator())
                    .start()
            }
            MotionEvent.ACTION_UP, MotionEvent.ACTION_CANCEL -> {
                animate().scaleX(1f).scaleY(1f)
                    .setDuration(200)
                    .setInterpolator(OvershootInterpolator(1.5f))
                    .start()
            }
        }
        false
    }
}

// Usage
binding.btnSubmit.addPressScale()
```

---

## 3. View Animations

### ViewPropertyAnimator (simple)

```kotlin
// Fade in
view.alpha = 0f
view.visibility = View.VISIBLE
view.animate()
    .alpha(1f)
    .setDuration(300)
    .setInterpolator(DecelerateInterpolator())
    .start()

// Fade out
view.animate()
    .alpha(0f)
    .setDuration(200)
    .withEndAction { view.visibility = View.GONE }
    .start()

// Slide in from bottom
view.translationY = view.height.toFloat()
view.visibility = View.VISIBLE
view.animate()
    .translationY(0f)
    .setDuration(350)
    .setInterpolator(DecelerateInterpolator(2f))
    .start()

// Scale + fade in (for popups, cards)
view.scaleX = 0.85f
view.scaleY = 0.85f
view.alpha = 0f
view.visibility = View.VISIBLE
view.animate()
    .scaleX(1f)
    .scaleY(1f)
    .alpha(1f)
    .setDuration(300)
    .setInterpolator(OvershootInterpolator(1.2f))
    .start()
```

### ObjectAnimator (property animation)

```kotlin
// Bounce effect on button
val bounceAnim = ObjectAnimator.ofFloat(view, "scaleX", 1f, 0.9f, 1.05f, 1f).apply {
    duration = 400
    interpolator = DecelerateInterpolator()
}
val bounceAnimY = ObjectAnimator.ofFloat(view, "scaleY", 1f, 0.9f, 1.05f, 1f).apply {
    duration = 400
}
AnimatorSet().apply {
    playTogether(bounceAnim, bounceAnimY)
    start()
}

// Color transition
val colorAnim = ObjectAnimator.ofObject(
    view,
    "backgroundColor",
    ArgbEvaluator(),
    startColor,
    endColor
).apply {
    duration = 300
    start()
}

// Elevation animation (on card expand)
ObjectAnimator.ofFloat(cardView, "cardElevation", 2f.dpToPx(), 12f.dpToPx()).apply {
    duration = 200
    start()
}
```

### Spring animation

```kotlin
// Add dependency: androidx.dynamicanimation:dynamicanimation
val springAnim = SpringAnimation(view, DynamicAnimation.SCALE_X, 1f).apply {
    spring.apply {
        dampingRatio = SpringForce.DAMPING_RATIO_MEDIUM_BOUNCY
        stiffness = SpringForce.STIFFNESS_MEDIUM
    }
}
// Trigger on press
springAnim.animateToFinalPosition(0.95f)   // press
springAnim.animateToFinalPosition(1f)      // release
```

---

## 4. TransitionManager — Screen / Layout Changes

```kotlin
// Basic transition when layout changes
TransitionManager.beginDelayedTransition(
    rootLayout,
    TransitionSet().apply {
        addTransition(Fade())
        addTransition(ChangeBounds())
        ordering = TransitionSet.ORDERING_TOGETHER
        duration = 300
    }
)
// Now change visibility / layout params — they will animate
view.visibility = View.VISIBLE

// Custom transition
val transition = TransitionSet().apply {
    addTransition(Fade(Fade.IN))
    addTransition(Slide(Gravity.BOTTOM))
    addTarget(R.id.card_container)
    duration = 350
    interpolator = DecelerateInterpolator(2f)
}
TransitionManager.beginDelayedTransition(container, transition)
card.visibility = View.VISIBLE
```

### Shared element transitions (Activity to Activity)

```kotlin
// Source Activity
val options = ActivityOptionsCompat.makeSceneTransitionAnimation(
    this,
    imageView,
    "shared_image"
)
startActivity(intent, options.toBundle())

// Source layout
android:transitionName="shared_image"

// Destination layout
android:transitionName="shared_image"

// Destination Activity
window.sharedElementEnterTransition = ChangeBounds().apply {
    duration = 400
    interpolator = DecelerateInterpolator()
}
```

---

## 5. Shadow & Elevation

```xml
<!-- MaterialCardView with shadow -->
<com.google.android.material.card.MaterialCardView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="16dp"
    app:cardCornerRadius="16dp"
    app:cardElevation="4dp"
    app:cardMaxElevation="8dp"
    app:strokeWidth="0dp">
    <!-- content -->
</com.google.android.material.card.MaterialCardView>
```

```kotlin
// Programmatic elevation
ViewCompat.setElevation(view, 8f.dpToPx())

// Elevation on press (feedback)
card.setOnTouchListener { _, event ->
    when (event.action) {
        MotionEvent.ACTION_DOWN -> card.cardElevation = 8f.dpToPx()
        MotionEvent.ACTION_UP   -> card.cardElevation = 2f.dpToPx()
    }
    false
}

// Custom shadow color (API 28+)
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
    view.outlineAmbientShadowColor = Color.parseColor("#14000000")
    view.outlineSpotShadowColor = Color.parseColor("#1A000000")
}
```

---

## 6. Corner Radius — Concentric Rule

**Rule**: inner radius = outer radius − padding

```xml
<!-- Outer container: 16dp radius, 8dp padding -->
<!-- Inner element must use: 16 - 8 = 8dp radius -->

<!-- res/drawable/bg_card_outer.xml -->
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <corners android:radius="16dp" />
    <solid android:color="#F5F5F5" />
</shape>

<!-- res/drawable/bg_card_inner.xml -->
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <corners android:radius="8dp" />  <!-- 16 - 8 = 8 -->
    <solid android:color="#FFFFFF" />
</shape>
```

```xml
<LinearLayout
    android:background="@drawable/bg_card_outer"
    android:padding="8dp">

    <LinearLayout
        android:background="@drawable/bg_card_inner">
        <!-- inner content -->
    </LinearLayout>
</LinearLayout>
```

### ShapeAppearance (Material)

```xml
<!-- res/values/shapes.xml -->
<resources>
    <style name="ShapeAppearance.App.Small" parent="">
        <item name="cornerFamily">rounded</item>
        <item name="cornerSize">8dp</item>
    </style>
    <style name="ShapeAppearance.App.Medium" parent="">
        <item name="cornerFamily">rounded</item>
        <item name="cornerSize">12dp</item>
    </style>
    <style name="ShapeAppearance.App.Large" parent="">
        <item name="cornerFamily">rounded</item>
        <item name="cornerSize">16dp</item>
    </style>
</resources>
```

```xml
<com.google.android.material.card.MaterialCardView
    app:shapeAppearance="@style/ShapeAppearance.App.Large" />
```

---

## 7. Image Polish — ShapeableImageView

```xml
<!-- Circle avatar -->
<com.google.android.material.imageview.ShapeableImageView
    android:id="@+id/avatar"
    android:layout_width="48dp"
    android:layout_height="48dp"
    android:scaleType="centerCrop"
    app:shapeAppearanceOverlay="@style/ShapeAppearance.Circle"
    app:strokeWidth="2dp"
    app:strokeColor="@color/surface_variant" />
```

```xml
<!-- res/values/shapes.xml -->
<style name="ShapeAppearance.Circle" parent="">
    <item name="cornerFamily">rounded</item>
    <item name="cornerSize">50%</item>
</style>

<style name="ShapeAppearance.Rounded" parent="">
    <item name="cornerFamily">rounded</item>
    <item name="cornerSize">12dp</item>
</style>
```

```kotlin
// Programmatic clip
imageView.shapeAppearanceModel = ShapeAppearanceModel.builder()
    .setAllCornerSizes(12f.dpToPx())
    .build()

// Glide with rounded corners
Glide.with(context)
    .load(url)
    .transform(RoundedCorners(12.dpToPx()))
    .placeholder(R.drawable.placeholder_shimmer)
    .error(R.drawable.ic_broken_image)
    .into(imageView)

// Glide with circle crop
Glide.with(context)
    .load(avatarUrl)
    .circleCrop()
    .into(avatarImageView)
```

---

## 8. RecyclerView Stagger Animation

```kotlin
class StaggerAnimator : DefaultItemAnimator() {
    override fun animateAdd(holder: RecyclerView.ViewHolder): Boolean {
        holder.itemView.apply {
            alpha = 0f
            translationY = 50f
            animate()
                .alpha(1f)
                .translationY(0f)
                .setDuration(350)
                .setStartDelay(holder.adapterPosition * 60L)
                .setInterpolator(DecelerateInterpolator(2f))
                .start()
        }
        return super.animateAdd(holder)
    }
}

// Usage
recyclerView.itemAnimator = StaggerAnimator()

// In Adapter.onBindViewHolder — reset for proper stagger
override fun onBindViewHolder(holder: VH, position: Int) {
    holder.itemView.alpha = 0f
    holder.itemView.translationY = 50f
    // bind data...
}
```

---

## 9. Loading States — Shimmer

```xml
<!-- build.gradle -->
implementation 'com.facebook.shimmer:shimmer:0.5.0'
```

```xml
<!-- Layout -->
<com.facebook.shimmer.ShimmerFrameLayout
    android:id="@+id/shimmer"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:shimmer_duration="1000"
    app:shimmer_base_color="#E0E0E0"
    app:shimmer_highlight_color="#F5F5F5">

    <!-- Skeleton layout (mirrors real content) -->
    <include layout="@layout/skeleton_item_card" />
</com.facebook.shimmer.ShimmerFrameLayout>
```

```kotlin
// Show shimmer while loading
shimmerLayout.startShimmer()
shimmerLayout.visibility = View.VISIBLE
recyclerView.visibility = View.GONE

// Hide shimmer when data arrives
shimmerLayout.stopShimmer()
shimmerLayout.visibility = View.GONE
recyclerView.visibility = View.VISIBLE
```

---

## 10. Smooth Scroll & NestedScroll

```kotlin
// Smooth scroll to position
recyclerView.smoothScrollToPosition(0)

// Smooth scroll to top with offset
val layoutManager = recyclerView.layoutManager as LinearLayoutManager
layoutManager.scrollToPositionWithOffset(0, 0)

// AppBar hide on scroll
val appBarLayout = binding.appBarLayout
recyclerView.addOnScrollListener(object : RecyclerView.OnScrollListener() {
    override fun onScrolled(rv: RecyclerView, dx: Int, dy: Int) {
        if (dy > 0) appBarLayout.animate().translationY(-appBarLayout.height.toFloat())
        else appBarLayout.animate().translationY(0f)
    }
})
```

---

## 11. Optical Alignment

```xml
<!-- Icon + Text row: push icon down 1–2dp for optical balance -->
<LinearLayout
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:gravity="center_vertical"
    android:orientation="horizontal">

    <ImageView
        android:layout_width="20dp"
        android:layout_height="20dp"
        android:layout_marginTop="1dp"    <!-- optical push down -->
        android:layout_marginEnd="8dp"
        android:src="@drawable/ic_star" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Rating" />
</LinearLayout>

<!-- Card: top padding slightly more than bottom -->
<LinearLayout
    android:paddingStart="16dp"
    android:paddingEnd="16dp"
    android:paddingTop="20dp"      <!-- optical: more top -->
    android:paddingBottom="16dp">
```

---

## 12. Dividers & Spacing Rhythm

```xml
<!-- Consistent spacing: 4, 8, 12, 16, 24, 32dp -->
<!-- res/values/dimens.xml -->
<resources>
    <dimen name="spacing_xs">4dp</dimen>
    <dimen name="spacing_sm">8dp</dimen>
    <dimen name="spacing_md">16dp</dimen>
    <dimen name="spacing_lg">24dp</dimen>
    <dimen name="spacing_xl">32dp</dimen>
    <dimen name="spacing_xxl">48dp</dimen>
</resources>

<!-- Subtle divider -->
<View
    android:layout_width="match_parent"
    android:layout_height="1dp"
    android:layout_marginStart="@dimen/spacing_md"
    android:layout_marginEnd="@dimen/spacing_md"
    android:background="#1A000000" />  <!-- black @ 10% -->
```

---

## 13. Helper Extensions

```kotlin
// Extensions — add to Extensions.kt
fun Float.dpToPx(): Float =
    this * Resources.getSystem().displayMetrics.density

fun Int.dpToPx(): Int =
    (this * Resources.getSystem().displayMetrics.density).toInt()

fun View.fadeIn(duration: Long = 300) {
    alpha = 0f
    visibility = View.VISIBLE
    animate().alpha(1f).setDuration(duration).start()
}

fun View.fadeOut(duration: Long = 200, onEnd: (() -> Unit)? = null) {
    animate().alpha(0f).setDuration(duration)
        .withEndAction { visibility = View.GONE; onEnd?.invoke() }
        .start()
}

fun View.slideUp(duration: Long = 350) {
    translationY = height.toFloat()
    visibility = View.VISIBLE
    animate().translationY(0f).setDuration(duration)
        .setInterpolator(DecelerateInterpolator(2f)).start()
}

fun View.popIn(duration: Long = 300) {
    scaleX = 0.85f; scaleY = 0.85f; alpha = 0f
    visibility = View.VISIBLE
    animate().scaleX(1f).scaleY(1f).alpha(1f)
        .setDuration(duration)
        .setInterpolator(OvershootInterpolator(1.2f)).start()
}
```

---

## Checklist — Before / After (XML Views)

- [ ] TextAppearance applied on all TextViews
- [ ] Custom font loaded via res/font/
- [ ] Buttons: ripple drawable or selectableItemBackground
- [ ] Important buttons: press scale via onTouchListener
- [ ] TransitionManager used when visibility/layout changes
- [ ] RecyclerView: StaggerAnimator applied
- [ ] Cards: MaterialCardView with elevation
- [ ] Concentric corners: inner = outer - padding
- [ ] Images: ShapeableImageView with shape + stroke
- [ ] Loading: ShimmerFrameLayout present
- [ ] Spacing: dimens.xml values used consistently
- [ ] Optical alignment: icon 1dp lower than text
