# Mobile UI Polish Skills

> Claude Code skills that make your mobile interfaces feel **alive** — spring physics, micro-animations, typography polish, haptics, shadows, and more. Works across all four major mobile UI stacks.

---

## Platforms

| Skill | Platform | File |
|-------|----------|------|
| `compose-ui-polish` | Android — Jetpack Compose | [SKILL.md](compose-ui-polish/SKILL.md) |
| `android-xml-ui-polish` | Android — XML Views | [SKILL.md](android-xml-ui-polish/SKILL.md) |
| `swiftui-ui-polish` | iOS — SwiftUI | [SKILL.md](swiftui-ui-polish/SKILL.md) |
| `uikit-ui-polish` | iOS — UIKit / Storyboard | [SKILL.md](uikit-ui-polish/SKILL.md) |

---

## Installation

### Option 1 — npx (recommended)

```bash
npx skills add atoshmatov/mobile-ui-polish-skills
```

### Option 2 — Manual

1. Clone this repo
```bash
git clone https://github.com/atoshmatov/mobile-ui-polish-skills.git
```

2. Copy the skill folder(s) you need into `~/.claude/skills/`

```bash
# macOS / Linux
cp -r mobile-ui-polish-skills/compose-ui-polish ~/.claude/skills/

# Windows (PowerShell)
Copy-Item -Recurse mobile-ui-polish-skills\compose-ui-polish $env:USERPROFILE\.claude\skills\
```

3. Restart Claude Code — the skill is now available.

---

## Usage

Once installed, simply describe what you want to polish in natural language. Claude Code will automatically apply the relevant skill.

### Trigger phrases

```
"make this screen feel better"
"add animations to the UI"
"polish the Compose UI"
"improve micro-interactions"
"this screen feels flat, fix it"
"add press feedback to buttons"
"add stagger animation to the list"
```

### Example — Jetpack Compose

**Before:**
```kotlin
Button(onClick = { submit() }) {
    Text("Submit")
}
```

**After (with compose-ui-polish skill):**
```kotlin
val interactionSource = remember { MutableInteractionSource() }
val isPressed by interactionSource.collectIsPressedAsState()
val scale by animateFloatAsState(
    targetValue = if (isPressed) 0.95f else 1f,
    animationSpec = spring(
        dampingRatio = Spring.DampingRatioMediumBouncy,
        stiffness = Spring.StiffnessHigh
    ),
    label = "scale"
)
Box(
    modifier = Modifier
        .scale(scale)
        .clip(RoundedCornerShape(12.dp))
        .background(MaterialTheme.colorScheme.primary)
        .clickable(interactionSource = interactionSource, indication = null) { submit() }
        .padding(horizontal = 24.dp, vertical = 14.dp),
    contentAlignment = Alignment.Center
) {
    Text("Submit", color = Color.White, fontWeight = FontWeight.Medium, letterSpacing = 0.3.sp)
}
```

### Example — UIKit

**Before:**
```swift
imageView.image = UIImage(named: "photo")
```

**After (with uikit-ui-polish skill):**
```swift
imageView.layer.cornerRadius = 12
imageView.layer.cornerCurve = .continuous
imageView.clipsToBounds = true
imageView.layer.borderWidth = 1
imageView.layer.borderColor = UIColor.black.withAlphaComponent(0.06).cgColor
```

---

## What's Covered

Every skill covers the same set of techniques, adapted to each platform's API:

### Typography
- Custom fonts
- Letter spacing (kern / tracking / letterSpacing)
- Line height / line spacing
- Weight contrast (heading vs body vs caption)
- Negative tracking for large text

### Press & Interaction States
- Scale-on-press with spring physics
- Ripple / highlight feedback
- Disabled, focused, highlighted states

### Animations
| Technique | Compose | XML Views | SwiftUI | UIKit |
|-----------|---------|-----------|---------|-------|
| Spring physics | `spring()` | `SpringAnimation` | `.spring(response:damping:)` | `usingSpringWithDamping` |
| Enter animation | `AnimatedVisibility` | `ObjectAnimator` | `.transition()` | `UIView.animate` |
| Content change | `AnimatedContent` | `TransitionManager` | `withAnimation` | `UIViewPropertyAnimator` |
| Hero / shared element | — | `ActivityOptions` | `matchedGeometryEffect` | `UIViewControllerTransitioning` |
| Stagger | `delay(index * 60L)` | `willDisplay + delay` | `onAppear + asyncAfter` | `willDisplay + delay` |

### Shadow & Elevation
- Ambient + spot shadow
- Colored shadow (for primary elements)
- Shadow path optimization (UIKit)
- MaterialCardView elevation (XML)

### Corner Radius — Concentric Rule
- `inner radius = outer radius − padding`
- Continuous corners (iOS 13+ / Material)
- Per-corner radius

### Image Polish
- Shape clipping (circle, rounded)
- Subtle border overlay
- Gradient overlay for text readability
- Shimmer / skeleton loading placeholder

### Loading States
- Skeleton placeholders
- Animated shimmer (all 4 platforms)
- Smooth transition: skeleton → content

### Haptic Feedback
| Effect | Android | iOS |
|--------|---------|-----|
| Light tap | `HapticFeedbackType.TextHandleMove` | `UIImpactFeedbackGenerator(.light)` |
| Button press | `HapticFeedbackType.LongPress` | `UIImpactFeedbackGenerator(.medium)` |
| Success | `VibrationEffect` | `UINotificationFeedbackGenerator(.success)` |
| Error | `VibrationEffect` | `UINotificationFeedbackGenerator(.error)` |

### Optical Alignment
- Icon + text vertical balance (push icon down 1–2dp)
- Card top padding ≠ bottom padding
- Visual center vs mathematical center

### Spacing Rhythm
`4 → 8 → 12 → 16 → 24 → 32 → 48 dp/pt`

---

## Pre-Ship Checklist

Copy this checklist into your PR before shipping any screen:

```
UI Polish Checklist:
- [ ] Typography: letterSpacing and lineHeight set on all text
- [ ] Buttons: press animation (scale or ripple)
- [ ] Lists: stagger animation on item appear
- [ ] Show/hide: animated (not instant snap)
- [ ] Content change: crossfade or slide transition
- [ ] Cards: shadow with correct color/opacity
- [ ] Nested corners: inner = outer − padding
- [ ] Images: clipped to shape, border if needed
- [ ] Loading: skeleton placeholder present
- [ ] Haptics: on primary CTA actions
- [ ] Spacing: consistent 4dp rhythm
- [ ] Optical alignment: icons visually centered
```

---

## Why These Skills Exist

Most UI code is **functionally correct but feels robotic**:
- Buttons snap with no feedback
- Lists appear all at once
- Text has no breathing room
- Shadows are missing or too heavy
- Corners are inconsistent

These skills encode the small details that separate "it works" from "it feels great" — and make Claude Code apply them consistently across your whole codebase.

> *"Great design is invisible. It guides users without them ever noticing."*

---

## Author

Made by **Alimardon Atoshmatov** — Android & iOS Developer

- GitHub: [@atoshmatov](https://github.com/atoshmatov)
- Project: [Cryon](https://github.com/atoshmatov)

---

## License

MIT — use freely in your projects.
