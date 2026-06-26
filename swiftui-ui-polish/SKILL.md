---
name: swiftui-ui-polish
description: >
  Use this skill to make SwiftUI interfaces feel polished and alive.
  Covers typography refinement, press/tap states with ButtonStyle,
  spring animations, withAnimation transitions, matchedGeometryEffect,
  stagger effects, shadow styling, concentric corner radius, image polish,
  haptics, SF Symbols animations, and optical alignment — all in SwiftUI.
  Includes Android/Kotlin analogies for each technique.
metadata:
  author: Alimardon / Cryon
  last-updated: '2026-06-26'
  keywords:
    - SwiftUI
    - UI Polish
    - Animation
    - Spring
    - ButtonStyle
    - matchedGeometryEffect
    - Haptics
    - Stagger
    - Shadow
    - Corner Radius
    - Typography
---

## When to Use

Use this skill when a SwiftUI screen feels "flat" — taps have no response,
transitions snap, text looks generic, or shadows are missing. Apply when
reviewing or polishing any SwiftUI view.

> **Android analogiyasi**: Bu skill Jetpack Compose uchun
> `compose-ui-polish` skill bilan parallel — bir xil prinsiplar,
> boshqa API.

---

## 1. Typography Polish

Android'da `TextStyle(letterSpacing, lineHeight)` ishlatiladi.
SwiftUI'da `.font()`, `.tracking()`, `.lineSpacing()`.

```swift
// BAD — default system font, no tuning
Text("Hello")

// GOOD — tuned
Text("Hello")
    .font(.system(size: 16, weight: .medium, design: .default))
    .tracking(0.3)           // letter spacing (positive = wider)
    .lineSpacing(6)          // extra space between lines

// Heading — tight tracking for large text
Text("Title")
    .font(.system(size: 28, weight: .bold))
    .tracking(-0.5)          // negative for large text
    .lineSpacing(4)

// Caption
Text("subtitle")
    .font(.system(size: 12, weight: .regular))
    .tracking(0.4)
    .foregroundStyle(.secondary)

// Custom font
Text("Hello")
    .font(.custom("Inter-Medium", size: 16))
    .tracking(0.2)

// Rules (same as Android):
// Large (28pt+)   → tracking: -0.3 to -0.8
// Body (14–18pt)  → tracking: 0.1–0.3
// Caption (10–12) → tracking: 0.3–0.5
```

---

## 2. Press / Tap States with ButtonStyle

Android'da `interactionSource + animateFloatAsState`.
SwiftUI'da `ButtonStyle` protocol.

```swift
// Scale button style — press down, release with spring
struct ScaleButtonStyle: ButtonStyle {
    var pressedScale: CGFloat = 0.95

    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .scaleEffect(configuration.isPressed ? pressedScale : 1.0)
            .animation(
                configuration.isPressed
                    ? .easeOut(duration: 0.1)
                    : .spring(response: 0.3, dampingFraction: 0.6),
                value: configuration.isPressed
            )
    }
}

// Usage
Button("Tap me") { doAction() }
    .buttonStyle(ScaleButtonStyle())

// Opacity + scale combo
struct FeedbackButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .scaleEffect(configuration.isPressed ? 0.96 : 1.0)
            .opacity(configuration.isPressed ? 0.85 : 1.0)
            .animation(.spring(response: 0.25, dampingFraction: 0.7),
                       value: configuration.isPressed)
    }
}

// Spring bounce style (for primary CTA buttons)
struct BounceButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .scaleEffect(configuration.isPressed ? 0.94 : 1.0)
            .animation(
                .spring(response: 0.2, dampingFraction: 0.5, blendDuration: 0),
                value: configuration.isPressed
            )
    }
}
```

---

## 3. Spring Animations

Android'da `spring(dampingRatio, stiffness)`.
SwiftUI'da `.spring(response:dampingFraction:)`.

```swift
// State variable
@State private var isExpanded = false

// Spring toggle
Button("Toggle") {
    withAnimation(.spring(response: 0.4, dampingFraction: 0.6)) {
        isExpanded.toggle()
    }
}

// Spring presets:
// Snappy:  .spring(response: 0.2, dampingFraction: 0.8)   → fast, minimal bounce
// Bouncy:  .spring(response: 0.4, dampingFraction: 0.5)   → clear bounce
// Gentle:  .spring(response: 0.6, dampingFraction: 0.7)   → slow, soft
// Physics: .spring(mass: 1, stiffness: 200, damping: 20)  → manual tuning

// Animate multiple properties
withAnimation(.spring(response: 0.35, dampingFraction: 0.65)) {
    cardOffset = isShowing ? 0 : 300
    cardOpacity = isShowing ? 1 : 0
    cardScale = isShowing ? 1 : 0.85
}

// .animation modifier (implicit — no withAnimation needed)
RoundedRectangle(cornerRadius: isExpanded ? 24 : 8)
    .animation(.spring(response: 0.3, dampingFraction: 0.7), value: isExpanded)
```

---

## 4. Transitions — Enter / Exit

Android'da `AnimatedVisibility(enter=fadeIn+slideIn, exit=fadeOut+slideOut)`.
SwiftUI'da `.transition()` + `withAnimation`.

```swift
// Basic fade
if isVisible {
    CardView()
        .transition(.opacity)
}

// Slide from bottom
if isVisible {
    CardView()
        .transition(.move(edge: .bottom).combined(with: .opacity))
}

// Scale + fade (dialogs, overlays)
if isVisible {
    DialogView()
        .transition(
            .asymmetric(
                insertion: .scale(scale: 0.85).combined(with: .opacity),
                removal: .scale(scale: 0.9).combined(with: .opacity)
            )
        )
}

// Custom transition
extension AnyTransition {
    static var slideUp: AnyTransition {
        .asymmetric(
            insertion: .move(edge: .bottom).combined(with: .opacity),
            removal: .move(edge: .top).combined(with: .opacity)
        )
    }

    static var popIn: AnyTransition {
        .scale(scale: 0.8, anchor: .center).combined(with: .opacity)
    }
}

// Trigger
Button("Show") {
    withAnimation(.spring(response: 0.35, dampingFraction: 0.65)) {
        isVisible.toggle()
    }
}
```

---

## 5. matchedGeometryEffect — Hero Animation

Android'da Shared Element Transition'ga o'xshash.
SwiftUI'da `matchedGeometryEffect`.

```swift
@Namespace private var heroNamespace
@State private var isExpanded = false

// List item → detail view hero
if !isExpanded {
    // List thumbnail
    Image("photo")
        .resizable()
        .scaledToFill()
        .frame(width: 80, height: 80)
        .clipShape(RoundedRectangle(cornerRadius: 12))
        .matchedGeometryEffect(id: "photo", in: heroNamespace)
        .onTapGesture {
            withAnimation(.spring(response: 0.4, dampingFraction: 0.7)) {
                isExpanded = true
            }
        }
} else {
    // Full screen detail
    Image("photo")
        .resizable()
        .scaledToFill()
        .frame(maxWidth: .infinity, maxHeight: 300)
        .clipShape(RoundedRectangle(cornerRadius: 20))
        .matchedGeometryEffect(id: "photo", in: heroNamespace)
        .onTapGesture {
            withAnimation(.spring(response: 0.4, dampingFraction: 0.7)) {
                isExpanded = false
            }
        }
}
```

---

## 6. Stagger Animations

Android'da `LazyColumn + index * 60ms delay`.
SwiftUI'da `ForEach + onAppear + delay`.

```swift
struct StaggeredList: View {
    let items: [Item]
    @State private var visibleItems: Set<Int> = []

    var body: some View {
        ScrollView {
            LazyVStack(spacing: 12) {
                ForEach(Array(items.enumerated()), id: \.offset) { index, item in
                    ItemCard(item: item)
                        .opacity(visibleItems.contains(index) ? 1 : 0)
                        .offset(y: visibleItems.contains(index) ? 0 : 30)
                        .onAppear {
                            DispatchQueue.main.asyncAfter(
                                deadline: .now() + Double(index) * 0.06
                            ) {
                                withAnimation(
                                    .spring(response: 0.4, dampingFraction: 0.7)
                                ) {
                                    visibleItems.insert(index)
                                }
                            }
                        }
                }
            }
            .padding()
        }
    }
}

// Grid stagger
LazyVGrid(columns: [GridItem(.flexible()), GridItem(.flexible())]) {
    ForEach(Array(items.enumerated()), id: \.offset) { index, item in
        GridItemView(item: item)
            .opacity(visible ? 1 : 0)
            .scaleEffect(visible ? 1 : 0.85)
            .onAppear {
                DispatchQueue.main.asyncAfter(
                    deadline: .now() + Double(index) * 0.04
                ) {
                    withAnimation(.spring()) { visible = true }
                }
            }
    }
}
```

---

## 7. Shadow & Depth

Android'da `Modifier.shadow(elevation, shape)`.
SwiftUI'da `.shadow(color:radius:x:y:)`.

```swift
// Card shadow
RoundedRectangle(cornerRadius: 16)
    .fill(.white)
    .shadow(
        color: Color.black.opacity(0.08),   // ambient
        radius: 8, x: 0, y: 2
    )
    .shadow(
        color: Color.black.opacity(0.12),   // spot
        radius: 16, x: 0, y: 8
    )

// Floating button shadow
Circle()
    .fill(Color.accentColor)
    .frame(width: 56, height: 56)
    .shadow(
        color: Color.accentColor.opacity(0.4),
        radius: 12, x: 0, y: 6
    )

// Subtle list item shadow
VStack {
    ForEach(items) { item in
        ItemRow(item: item)
            .background(
                RoundedRectangle(cornerRadius: 12)
                    .fill(.white)
                    .shadow(color: .black.opacity(0.06), radius: 4, y: 2)
            )
    }
}

// Inner shadow (highlight effect)
RoundedRectangle(cornerRadius: 12)
    .fill(.white)
    .overlay(
        RoundedRectangle(cornerRadius: 12)
            .stroke(
                LinearGradient(
                    colors: [.white.opacity(0.5), .clear],
                    startPoint: .topLeading,
                    endPoint: .bottomTrailing
                ),
                lineWidth: 1
            )
    )
```

---

## 8. Corner Radius — Concentric Rule

Android'da `inner = outer - padding`.
SwiftUI'da ayniy qoida.

```swift
// BAD — both same radius (looks wrong)
RoundedRectangle(cornerRadius: 20)
    .fill(Color.gray.opacity(0.2))
    .padding(8)
    .overlay(
        RoundedRectangle(cornerRadius: 20)  // ← too big!
            .fill(.white)
    )

// GOOD — inner = outer - padding = 20 - 8 = 12
let outerRadius: CGFloat = 20
let padding: CGFloat = 8
let innerRadius: CGFloat = outerRadius - padding  // = 12

RoundedRectangle(cornerRadius: outerRadius)
    .fill(Color.gray.opacity(0.2))
    .padding(padding)
    .overlay(
        RoundedRectangle(cornerRadius: innerRadius)   // ← correct
            .fill(.white)
    )

// Helper view modifier
extension View {
    func concentricCard(outerRadius: CGFloat = 20, padding: CGFloat = 12) -> some View {
        self
            .padding(padding)
            .background(
                RoundedRectangle(cornerRadius: outerRadius - padding)
                    .fill(Color(.systemBackground))
            )
            .padding(2)
            .background(
                RoundedRectangle(cornerRadius: outerRadius)
                    .fill(Color(.systemGroupedBackground))
            )
    }
}
```

---

## 9. Image Polish

```swift
// Rounded image with border
AsyncImage(url: URL(string: imageURL)) { image in
    image
        .resizable()
        .scaledToFill()
} placeholder: {
    ShimmerView()
}
.frame(width: 80, height: 80)
.clipShape(RoundedRectangle(cornerRadius: 12))
.overlay(
    RoundedRectangle(cornerRadius: 12)
        .stroke(Color.black.opacity(0.06), lineWidth: 1)
)

// Circle avatar
AsyncImage(url: URL(string: avatarURL)) { image in
    image.resizable().scaledToFill()
} placeholder: {
    Circle().fill(Color.gray.opacity(0.2))
}
.frame(width: 48, height: 48)
.clipShape(Circle())
.overlay(Circle().stroke(Color(.systemGray5), lineWidth: 2))

// Gradient overlay (text over image)
ZStack(alignment: .bottom) {
    AsyncImage(url: URL(string: imageURL)) { image in
        image.resizable().scaledToFill()
    } placeholder: { Color.gray }
    .frame(height: 200)
    .clipped()

    LinearGradient(
        colors: [.clear, .black.opacity(0.7)],
        startPoint: .center,
        endPoint: .bottom
    )
    .frame(height: 120)

    Text(title)
        .foregroundStyle(.white)
        .padding()
        .frame(maxWidth: .infinity, alignment: .leading)
}
.clipShape(RoundedRectangle(cornerRadius: 16))
```

---

## 10. Haptic Feedback

```swift
// Light tap
let generator = UIImpactFeedbackGenerator(style: .light)
generator.impactOccurred()

// Medium (button press)
UIImpactFeedbackGenerator(style: .medium).impactOccurred()

// Heavy (destructive action)
UIImpactFeedbackGenerator(style: .heavy).impactOccurred()

// Success
UINotificationFeedbackGenerator().notificationOccurred(.success)

// Warning
UINotificationFeedbackGenerator().notificationOccurred(.warning)

// Error
UINotificationFeedbackGenerator().notificationOccurred(.error)

// Selection change (picker, toggle)
UISelectionFeedbackGenerator().selectionChanged()

// SwiftUI wrapper
extension View {
    func hapticFeedback(_ style: UIImpactFeedbackGenerator.FeedbackStyle = .light) -> some View {
        self.onTapGesture {
            UIImpactFeedbackGenerator(style: style).impactOccurred()
        }
    }
}

// Usage
Button("Submit") { submit() }
    .hapticFeedback(.medium)
```

---

## 11. SF Symbols Animations (iOS 17+)

```swift
// Bounce on tap
Image(systemName: "heart.fill")
    .symbolEffect(.bounce, value: isLiked)
    .onTapGesture { isLiked.toggle() }

// Pulse (for loading/active indicators)
Image(systemName: "wifi")
    .symbolEffect(.pulse)

// Scale
Image(systemName: "bell.fill")
    .symbolEffect(.scale.up, value: notified)

// Replace symbol (animated icon swap)
Image(systemName: isPlaying ? "pause.fill" : "play.fill")
    .contentTransition(.symbolEffect(.replace))
    .animation(.spring(), value: isPlaying)
```

---

## 12. Optical Alignment

```swift
// Icon + Text — optical push down
HStack(spacing: 8) {
    Image(systemName: "star.fill")
        .frame(width: 20, height: 20)
        .padding(.top, 1)    // optical: push icon down 1pt
    Text("Rating")
}

// Card — more top padding (optical center)
VStack(alignment: .leading, spacing: 8) {
    Text("Title")
    Text("Subtitle").foregroundStyle(.secondary)
}
.padding(.horizontal, 16)
.padding(.top, 20)          // slightly more top
.padding(.bottom, 16)

// Visual center in scroll (content feels better with more bottom space)
ScrollView {
    content
        .padding(.bottom, 32)   // more breathing room at bottom
}
```

---

## 13. Loading States — Shimmer

```swift
struct ShimmerView: View {
    @State private var phase: CGFloat = -1

    var body: some View {
        RoundedRectangle(cornerRadius: 8)
            .fill(
                LinearGradient(
                    stops: [
                        .init(color: Color(.systemGray5), location: phase),
                        .init(color: Color(.systemGray6), location: phase + 0.3),
                        .init(color: Color(.systemGray5), location: phase + 0.6)
                    ],
                    startPoint: .leading,
                    endPoint: .trailing
                )
            )
            .onAppear {
                withAnimation(
                    .linear(duration: 1.2).repeatForever(autoreverses: false)
                ) {
                    phase = 1.5
                }
            }
    }
}

// Skeleton layout
VStack(alignment: .leading, spacing: 12) {
    ShimmerView().frame(height: 200)                       // image
    ShimmerView().frame(width: 200, height: 20)           // title
    ShimmerView().frame(width: 140, height: 16)           // subtitle
}
```

---

## Checklist — Before / After (SwiftUI)

- [ ] Typography: `.tracking()` and `.lineSpacing()` set on all Text
- [ ] Buttons: custom `ButtonStyle` with press scale/spring
- [ ] Lists: stagger animation on `.onAppear`
- [ ] Show/hide: `.transition()` + `withAnimation` (not instant if/else)
- [ ] Hero transitions: `matchedGeometryEffect` where needed
- [ ] Shadows: double shadow (ambient + spot) on cards
- [ ] Concentric corners: inner = outer - padding
- [ ] Images: `.clipShape()` + `.overlay(stroke)` for border
- [ ] Loading: `ShimmerView` placeholder
- [ ] Haptics: `UIImpactFeedbackGenerator` on primary actions
- [ ] SF Symbols: `.symbolEffect()` on interactive icons (iOS 17+)
- [ ] Optical alignment: icon 1pt lower than text
