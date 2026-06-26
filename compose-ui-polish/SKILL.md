---
name: compose-ui-polish
description: >
  Use this skill to make Jetpack Compose interfaces feel polished and alive.
  Covers typography refinement, press/interaction states, micro-animations,
  spring physics, enter/exit transitions, stagger effects, shadows, concentric
  corner radius, image polish, haptics, and optical alignment — all in Compose.
metadata:
  author: Alimardon / Cryon
  last-updated: '2026-06-26'
  keywords:
    - Jetpack Compose
    - UI Polish
    - Animation
    - Micro-interactions
    - Typography
    - Shadow
    - Corner Radius
    - Stagger
    - Spring
    - Haptics
---

## When to Use

Use this skill when a Compose UI looks "flat" or "lifeless" — buttons don't
respond, screens snap instead of animate, text feels generic, corners look
inconsistent, or shadows are missing. Apply it component-by-component or
as a full-screen polish pass.

Trigger phrase examples:
- "make this screen feel better"
- "add animations to the UI"
- "polish the Compose UI"
- "improve micro-interactions"

---

## 1. Typography Polish

Poor typography makes UI feel unfinished. Fix letter spacing, line height,
and weight contrast.

```kotlin
// BAD — default text, no tuning
Text("Hello", style = TextStyle(fontSize = 16.sp))

// GOOD — tuned typography
Text(
    text = "Hello",
    style = TextStyle(
        fontSize = 16.sp,
        fontWeight = FontWeight.Medium,
        letterSpacing = 0.15.sp,      // subtle breathing room
        lineHeight = 24.sp,            // 1.5x line height
        color = MaterialTheme.colorScheme.onSurface
    )
)

// Heading — tight tracking for large sizes
Text(
    text = "Title",
    style = TextStyle(
        fontSize = 28.sp,
        fontWeight = FontWeight.Bold,
        letterSpacing = (-0.5).sp,    // negative tracking for big text
        lineHeight = 34.sp
    )
)

// Caption — slightly loose tracking
Text(
    text = "subtitle",
    style = TextStyle(
        fontSize = 12.sp,
        fontWeight = FontWeight.Normal,
        letterSpacing = 0.4.sp,
        color = MaterialTheme.colorScheme.onSurfaceVariant
    )
)
```

**Rules:**
- Large text (28sp+) → negative letter spacing (–0.3 to –0.8)
- Body text (14–18sp) → 0.1–0.25sp
- Small/caption (10–12sp) → 0.3–0.5sp
- Line height = fontSize × 1.4–1.6

---

## 2. Press / Interaction States

Buttons must respond to touch immediately — scale down on press, scale back
on release. Use `interactionSource` + `animateFloatAsState`.

```kotlin
@Composable
fun PolishedButton(
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
    val interactionSource = remember { MutableInteractionSource() }
    val isPressed by interactionSource.collectIsPressedAsState()

    val scale by animateFloatAsState(
        targetValue = if (isPressed) 0.95f else 1f,
        animationSpec = spring(
            dampingRatio = Spring.DampingRatioMediumBouncy,
            stiffness = Spring.StiffnessHigh
        ),
        label = "button_scale"
    )

    Box(
        modifier = modifier
            .scale(scale)
            .clickable(
                interactionSource = interactionSource,
                indication = null,  // remove default ripple if using scale
                onClick = onClick
            ),
        contentAlignment = Alignment.Center
    ) {
        content()
    }
}

// OR — keep ripple + add scale
val scale by animateFloatAsState(
    targetValue = if (isPressed) 0.97f else 1f,
    animationSpec = spring(stiffness = Spring.StiffnessHigh),
    label = "scale"
)
Box(
    modifier = Modifier
        .scale(scale)
        .clip(RoundedCornerShape(12.dp))
        .clickable(interactionSource = interactionSource, indication = rememberRipple())
        { onClick() }
)
```

---

## 3. Spring Animations (Physics-Based)

Replace `tween()` with `spring()` for natural, physical feel.

```kotlin
// BAD — linear/tween feels robotic
val offset by animateDpAsState(
    targetValue = if (visible) 0.dp else 20.dp,
    animationSpec = tween(300)
)

// GOOD — spring feels alive
val offset by animateDpAsState(
    targetValue = if (visible) 0.dp else 20.dp,
    animationSpec = spring(
        dampingRatio = Spring.DampingRatioMediumBouncy,  // slight bounce
        stiffness = Spring.StiffnessMedium
    ),
    label = "offset"
)

// Spring presets:
// DampingRatioNoBouncy     = 1.0f  → smooth, no overshoot
// DampingRatioLowBouncy    = 0.75f → gentle bounce
// DampingRatioMediumBouncy = 0.5f  → clear bounce
// DampingRatioHighBouncy   = 0.2f  → strong bounce (use carefully)

// StiffnessHigh   = 10_000f → snappy
// StiffnessMedium = 400f    → balanced
// StiffnessLow    = 200f    → floaty
// StiffnessVeryLow= 50f     → sluggish (avoid for most UI)
```

---

## 4. AnimatedVisibility — Enter / Exit Transitions

Never show/hide elements instantly. Always animate in and out.

```kotlin
// BAD — instant show/hide
if (visible) {
    Card { content() }
}

// GOOD — animated
AnimatedVisibility(
    visible = visible,
    enter = fadeIn(animationSpec = tween(200)) +
            slideInVertically(
                initialOffsetY = { it / 4 },       // slide from 25% below
                animationSpec = spring(
                    dampingRatio = Spring.DampingRatioMediumBouncy,
                    stiffness = Spring.StiffnessMedium
                )
            ),
    exit = fadeOut(animationSpec = tween(150)) +
           slideOutVertically(targetOffsetY = { it / 4 })
) {
    Card { content() }
}

// Scale in (for dialogs, bottom sheets, tooltips)
AnimatedVisibility(
    visible = visible,
    enter = scaleIn(
        initialScale = 0.85f,
        transformOrigin = TransformOrigin.Center,
        animationSpec = spring(dampingRatio = Spring.DampingRatioMediumBouncy)
    ) + fadeIn(),
    exit = scaleOut(targetScale = 0.9f) + fadeOut()
) {
    DialogContent()
}

// Expand from top (for dropdowns, accordions)
AnimatedVisibility(
    visible = expanded,
    enter = expandVertically(expandFrom = Alignment.Top) + fadeIn(),
    exit = shrinkVertically(shrinkTowards = Alignment.Top) + fadeOut()
) {
    DropdownContent()
}
```

---

## 5. AnimatedContent — Content Transitions

When content CHANGES (not appears/disappears), use `AnimatedContent`.

```kotlin
// BAD — content snaps
Text(if (loading) "Loading..." else "Done")

// GOOD — content animates
AnimatedContent(
    targetState = uiState,
    transitionSpec = {
        fadeIn(tween(200)) + slideInVertically { it / 4 } togetherWith
        fadeOut(tween(150)) + slideOutVertically { -it / 4 }
    },
    label = "content"
) { state ->
    when (state) {
        is Loading -> CircularProgressIndicator()
        is Success -> SuccessContent(state.data)
        is Error   -> ErrorContent(state.message)
    }
}

// Crossfade (simpler — just fades between content)
Crossfade(
    targetState = currentTab,
    animationSpec = tween(200),
    label = "tab"
) { tab ->
    when (tab) {
        Tab.Home    -> HomeScreen()
        Tab.Profile -> ProfileScreen()
    }
}
```

---

## 6. Stagger Animations

List items should appear one by one, not all at once.

```kotlin
// LazyColumn with stagger
LazyColumn {
    itemsIndexed(items) { index, item ->
        var visible by remember { mutableStateOf(false) }

        LaunchedEffect(Unit) {
            delay(index * 60L)   // 60ms between each item
            visible = true
        }

        AnimatedVisibility(
            visible = visible,
            enter = fadeIn(tween(300)) +
                    slideInVertically(
                        initialOffsetY = { it / 3 },
                        animationSpec = spring(
                            dampingRatio = Spring.DampingRatioMediumBouncy
                        )
                    )
        ) {
            ItemCard(item)
        }
    }
}

// Grid stagger (2 columns — alternate left/right)
LazyVerticalGrid(columns = GridCells.Fixed(2)) {
    itemsIndexed(items) { index, item ->
        var visible by remember { mutableStateOf(false) }
        LaunchedEffect(Unit) {
            delay(index * 40L)
            visible = true
        }
        AnimatedVisibility(
            visible = visible,
            enter = fadeIn() + scaleIn(initialScale = 0.9f)
        ) {
            GridItem(item)
        }
    }
}
```

---

## 7. Shadow & Elevation

Material3 uses `Modifier.shadow()` for custom shapes; `elevation` for cards.

```kotlin
// Card with custom shadow
Card(
    modifier = Modifier
        .fillMaxWidth()
        .shadow(
            elevation = 8.dp,
            shape = RoundedCornerShape(16.dp),
            clip = false,
            ambientColor = Color.Black.copy(alpha = 0.08f),
            spotColor = Color.Black.copy(alpha = 0.12f)
        ),
    shape = RoundedCornerShape(16.dp),
    colors = CardDefaults.cardColors(containerColor = MaterialTheme.colorScheme.surface)
) { ... }

// Floating element (FAB-like) shadow
Box(
    modifier = Modifier
        .size(56.dp)
        .shadow(
            elevation = 12.dp,
            shape = CircleShape,
            spotColor = MaterialTheme.colorScheme.primary.copy(alpha = 0.3f)
        )
        .background(MaterialTheme.colorScheme.primary, CircleShape)
)

// Subtle card shadow (for lists)
Modifier.shadow(
    elevation = 2.dp,
    shape = RoundedCornerShape(12.dp),
    ambientColor = Color(0x14000000),
    spotColor = Color(0x1A000000)
)
```

---

## 8. Corner Radius — Concentric Rule

When a container has rounded corners, nested elements must have SMALLER
radius. This is called "concentric radius". Formula:
`inner radius = outer radius - padding`

```kotlin
// BAD — both have same 16.dp radius (looks wrong)
Box(
    modifier = Modifier
        .clip(RoundedCornerShape(16.dp))
        .background(Color.Gray)
        .padding(8.dp)
) {
    Box(
        modifier = Modifier
            .clip(RoundedCornerShape(16.dp))  // ← too big!
            .background(Color.White)
    )
}

// GOOD — inner = outer - padding = 16 - 8 = 8
Box(
    modifier = Modifier
        .clip(RoundedCornerShape(16.dp))
        .background(Color.Gray)
        .padding(8.dp)
) {
    Box(
        modifier = Modifier
            .clip(RoundedCornerShape(8.dp))   // ← correct
            .background(Color.White)
    )
}

// Concentric helper function
fun concentricRadius(outerRadius: Dp, padding: Dp): Dp =
    (outerRadius - padding).coerceAtLeast(0.dp)
```

---

## 9. Image Polish

Images look better with shape clipping, overlays, and subtle borders.

```kotlin
// Rounded image with subtle border
AsyncImage(
    model = imageUrl,
    contentDescription = null,
    contentScale = ContentScale.Crop,
    modifier = Modifier
        .size(80.dp)
        .clip(RoundedCornerShape(12.dp))
        .border(
            width = 1.dp,
            color = Color.Black.copy(alpha = 0.06f),
            shape = RoundedCornerShape(12.dp)
        )
)

// Avatar — circle with border
AsyncImage(
    model = avatarUrl,
    contentDescription = null,
    contentScale = ContentScale.Crop,
    modifier = Modifier
        .size(48.dp)
        .clip(CircleShape)
        .border(2.dp, MaterialTheme.colorScheme.surfaceVariant, CircleShape)
)

// Image with gradient overlay (for text readability)
Box {
    AsyncImage(
        model = imageUrl,
        contentDescription = null,
        contentScale = ContentScale.Crop,
        modifier = Modifier.fillMaxSize()
    )
    Box(
        modifier = Modifier
            .fillMaxWidth()
            .align(Alignment.BottomCenter)
            .height(120.dp)
            .background(
                Brush.verticalGradient(
                    colors = listOf(Color.Transparent, Color.Black.copy(0.7f))
                )
            )
    )
}

// Loading shimmer placeholder (skeleton)
val shimmerColors = listOf(
    Color.LightGray.copy(alpha = 0.6f),
    Color.LightGray.copy(alpha = 0.2f),
    Color.LightGray.copy(alpha = 0.6f)
)
val transition = rememberInfiniteTransition(label = "shimmer")
val translateAnim by transition.animateFloat(
    initialValue = 0f, targetValue = 1000f,
    animationSpec = infiniteRepeatable(tween(1000, easing = LinearEasing)),
    label = "shimmer_translate"
)
Box(
    modifier = Modifier
        .size(80.dp)
        .clip(RoundedCornerShape(12.dp))
        .background(Brush.horizontalGradient(shimmerColors, startX = translateAnim - 200, endX = translateAnim))
)
```

---

## 10. Haptic Feedback

UI should feel tactile — add haptics on important interactions.

```kotlin
// Setup
val haptic = LocalHapticFeedback.current

// Light tap (button press)
Button(onClick = {
    haptic.performHapticFeedback(HapticFeedbackType.TextHandleMove)
}) { Text("Tap") }

// Success action
haptic.performHapticFeedback(HapticFeedbackType.LongPress)

// Error / warning (use sparingly)
val vibrator = context.getSystemService(Vibrator::class.java)
vibrator?.vibrate(VibrationEffect.createWaveform(longArrayOf(0, 50, 50, 50), -1))
```

---

## 11. Optical Alignment

Visual center ≠ mathematical center. Text and icons need visual padding
adjustments.

```kotlin
// Icon + Text — vertically optical
Row(
    verticalAlignment = Alignment.CenterVertically,
    horizontalArrangement = Arrangement.spacedBy(8.dp)
) {
    Icon(
        imageVector = Icons.Default.Star,
        contentDescription = null,
        modifier = Modifier
            .size(20.dp)
            .padding(top = 1.dp)   // optical: push icon down 1dp
    )
    Text("Rating")
}

// Card padding — optical (top more than bottom)
Card {
    Column(
        modifier = Modifier.padding(
            start = 16.dp,
            end = 16.dp,
            top = 20.dp,    // slightly more on top
            bottom = 16.dp
        )
    ) { ... }
}

// Bottom nav — visual center with icon+label
NavigationBarItem(
    icon = { Icon(..., modifier = Modifier.padding(bottom = 2.dp)) }, // optical push
    label = { Text(...) },
    selected = selected,
    onClick = onClick
)
```

---

## 12. Loading States — Skeleton & Shimmer

Never show empty screens. Use skeleton placeholders.

```kotlin
@Composable
fun SkeletonBox(modifier: Modifier = Modifier) {
    val infiniteTransition = rememberInfiniteTransition(label = "skeleton")
    val alpha by infiniteTransition.animateFloat(
        initialValue = 0.3f,
        targetValue = 0.7f,
        animationSpec = infiniteRepeatable(
            animation = tween(700, easing = FastOutSlowInEasing),
            repeatMode = RepeatMode.Reverse
        ),
        label = "alpha"
    )
    Box(
        modifier = modifier
            .clip(RoundedCornerShape(8.dp))
            .background(MaterialTheme.colorScheme.onSurface.copy(alpha = alpha))
    )
}

// Usage
Column(verticalArrangement = Arrangement.spacedBy(8.dp)) {
    SkeletonBox(Modifier.fillMaxWidth().height(200.dp))        // image
    SkeletonBox(Modifier.fillMaxWidth(0.7f).height(20.dp))    // title
    SkeletonBox(Modifier.fillMaxWidth(0.5f).height(16.dp))    // subtitle
}
```

---

## 13. Dividers & Spacing

Use consistent spacing rhythm (4dp, 8dp, 12dp, 16dp, 24dp, 32dp).

```kotlin
// Subtle divider
HorizontalDivider(
    modifier = Modifier.padding(horizontal = 16.dp),
    thickness = 1.dp,
    color = MaterialTheme.colorScheme.outlineVariant.copy(alpha = 0.5f)
)

// Spacing rhythm constants
object Spacing {
    val xs = 4.dp
    val sm = 8.dp
    val md = 16.dp
    val lg = 24.dp
    val xl = 32.dp
    val xxl = 48.dp
}

// Use in layouts
Column(verticalArrangement = Arrangement.spacedBy(Spacing.sm)) {
    ...
}
```

---

## 14. Color & Contrast

```kotlin
// Surface hierarchy (background → surface → surfaceVariant)
// Background: MaterialTheme.colorScheme.background    (farthest back)
// Card:       MaterialTheme.colorScheme.surface       (middle)
// Chip:       MaterialTheme.colorScheme.surfaceVariant (top layer)

// Text contrast
// Primary text:   onSurface         (full opacity)
// Secondary text: onSurfaceVariant  (muted)
// Disabled text:  onSurface.copy(alpha = 0.38f)

// Subtle overlays
Box(
    modifier = Modifier
        .background(
            color = MaterialTheme.colorScheme.primary.copy(alpha = 0.08f),
            shape = RoundedCornerShape(8.dp)
        )
)
```

---

## Checklist — Before / After

Before shipping any screen, verify:

- [ ] Typography: letterSpacing and lineHeight set on all Text
- [ ] Buttons: press scale or ripple animation present
- [ ] Lists: stagger animation on appear
- [ ] Show/hide: AnimatedVisibility (not if/else snap)
- [ ] Content change: AnimatedContent or Crossfade
- [ ] Cards: shadow with correct ambientColor/spotColor
- [ ] Nested corners: inner radius = outer radius - padding
- [ ] Images: clipped to shape, border if needed
- [ ] Loading states: skeleton placeholders present
- [ ] Haptics: on primary actions
- [ ] Spacing: consistent 4dp rhythm
- [ ] Color: surface hierarchy used correctly
