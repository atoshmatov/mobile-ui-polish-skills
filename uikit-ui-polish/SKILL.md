---
name: uikit-ui-polish
description: >
  Use this skill to make UIKit (Storyboard/XIB/programmatic) interfaces feel
  polished. Covers NSAttributedString typography, UIControl state drawables,
  UIView.animate spring physics, UIViewPropertyAnimator, CALayer corner radius
  (concentric rule), layer.shadow, UICollectionView/UITableView stagger,
  UIImageView clipping, CABasicAnimation, UIVisualEffectView blur,
  haptics, and CAGradientLayer — all in UIKit.
  Includes Android/Kotlin analogies for each technique.
metadata:
  author: Alimardon / Cryon
  last-updated: '2026-06-26'
  keywords:
    - UIKit
    - Storyboard
    - XIB
    - UI Polish
    - CALayer
    - UIViewPropertyAnimator
    - Spring Animation
    - CABasicAnimation
    - CAGradientLayer
    - UIVisualEffectView
    - Haptics
    - Stagger
    - Typography
    - NSAttributedString
---

## When to Use

Use this skill when working with UIKit-based iOS apps (Storyboard, XIB, or
programmatic views without SwiftUI). App uses `UIViewController`,
`UITableView`, `UICollectionView`, `UIImageView`, `UIButton` etc.

> **Android analogiyasi**: Bu skill Android XML Views uchun
> `android-xml-ui-polish` skill bilan parallel.

---

## 1. Typography Polish

Android'da `TextAppearance + SpannableString`.
UIKit'da `NSAttributedString + UIFont`.

```swift
// NSAttributedString — full control
let paragraphStyle = NSMutableParagraphStyle()
paragraphStyle.lineSpacing = 6         // extra space between lines
paragraphStyle.lineBreakMode = .byWordWrapping

let attributes: [NSAttributedString.Key: Any] = [
    .font: UIFont.systemFont(ofSize: 16, weight: .medium),
    .kern: 0.3,                        // letter spacing
    .foregroundColor: UIColor.label,
    .paragraphStyle: paragraphStyle
]
label.attributedText = NSAttributedString(string: "Hello", attributes: attributes)

// Heading — tight tracking for large text
let headingAttributes: [NSAttributedString.Key: Any] = [
    .font: UIFont.systemFont(ofSize: 28, weight: .bold),
    .kern: -0.5,                       // negative for large text
    .foregroundColor: UIColor.label
]

// Caption — loose tracking
let captionAttributes: [NSAttributedString.Key: Any] = [
    .font: UIFont.systemFont(ofSize: 12, weight: .regular),
    .kern: 0.4,
    .foregroundColor: UIColor.secondaryLabel
]

// Custom font
let customFont = UIFont(name: "Inter-Medium", size: 16) ?? UIFont.systemFont(ofSize: 16)
let customAttributes: [NSAttributedString.Key: Any] = [
    .font: customFont,
    .kern: 0.2
]

// UILabel extension helper
extension UILabel {
    func setStyledText(
        _ text: String,
        size: CGFloat,
        weight: UIFont.Weight = .regular,
        kern: CGFloat = 0,
        lineSpacing: CGFloat = 0,
        color: UIColor = .label
    ) {
        let paragraphStyle = NSMutableParagraphStyle()
        paragraphStyle.lineSpacing = lineSpacing
        attributedText = NSAttributedString(
            string: text,
            attributes: [
                .font: UIFont.systemFont(ofSize: size, weight: weight),
                .kern: kern,
                .foregroundColor: color,
                .paragraphStyle: paragraphStyle
            ]
        )
    }
}

// Usage
titleLabel.setStyledText("Hello", size: 28, weight: .bold, kern: -0.5)
bodyLabel.setStyledText("Body text", size: 16, weight: .regular, kern: 0.2, lineSpacing: 6)
```

---

## 2. Button Press States

Android'da `StateListDrawable + TouchListener`.
UIKit'da `UIControl.State + UIView.animate`.

```swift
// UIButton with highlighted state
class PolishedButton: UIButton {
    override var isHighlighted: Bool {
        didSet {
            UIView.animate(
                withDuration: isHighlighted ? 0.1 : 0.25,
                delay: 0,
                usingSpringWithDamping: isHighlighted ? 1.0 : 0.6,
                initialSpringVelocity: 0.8,
                options: [.allowUserInteraction],
                animations: {
                    self.transform = self.isHighlighted
                        ? CGAffineTransform(scaleX: 0.95, y: 0.95)
                        : .identity
                    self.alpha = self.isHighlighted ? 0.9 : 1.0
                }
            )
        }
    }
}

// Extension on UIView for press scale
extension UIView {
    func addPressAnimation(scale: CGFloat = 0.95) {
        let tap = UILongPressGestureRecognizer(target: self, action: #selector(handlePress))
        tap.minimumPressDuration = 0
        addGestureRecognizer(tap)
    }

    @objc private func handlePress(_ gesture: UILongPressGestureRecognizer) {
        switch gesture.state {
        case .began:
            UIView.animate(withDuration: 0.1) {
                self.transform = CGAffineTransform(scaleX: 0.95, y: 0.95)
                self.alpha = 0.85
            }
        case .ended, .cancelled:
            UIView.animate(
                withDuration: 0.3, delay: 0,
                usingSpringWithDamping: 0.5,
                initialSpringVelocity: 0.8,
                animations: {
                    self.transform = .identity
                    self.alpha = 1.0
                }
            )
        default: break
        }
    }
}

// Storyboard button state colors
button.setBackgroundColor(.systemBlue, for: .normal)
button.setBackgroundColor(.systemBlue.withAlphaComponent(0.7), for: .highlighted)
button.setBackgroundColor(.systemGray3, for: .disabled)
```

---

## 3. Spring Animations

Android'da `SpringAnimation(DynamicAnimation)`.
UIKit'da `usingSpringWithDamping`.

```swift
// Basic spring animation
UIView.animate(
    withDuration: 0.5,
    delay: 0,
    usingSpringWithDamping: 0.65,      // 0.0 = max bounce, 1.0 = no bounce
    initialSpringVelocity: 0.5,        // initial velocity
    options: [],
    animations: {
        view.transform = isExpanded
            ? .identity
            : CGAffineTransform(scaleX: 0.85, y: 0.85)
    }
)

// Spring presets:
// Snappy (no bounce):  duration: 0.3, damping: 1.0
// Gentle bounce:       duration: 0.4, damping: 0.75
// Medium bounce:       duration: 0.5, damping: 0.6
// Bouncy:              duration: 0.6, damping: 0.45

// UIViewPropertyAnimator (iOS 10+ — more control)
let animator = UIViewPropertyAnimator(
    duration: 0.5,
    timingParameters: UISpringTimingParameters(
        mass: 1,
        stiffness: 200,
        damping: 20,
        initialVelocity: CGVector(dx: 0, dy: 0)
    )
)
animator.addAnimations {
    view.alpha = 1
    view.transform = .identity
}
animator.startAnimation()

// Slide in from bottom (modal style)
func slideIn(_ view: UIView) {
    view.transform = CGAffineTransform(translationX: 0, y: view.bounds.height)
    view.alpha = 0
    UIView.animate(
        withDuration: 0.45, delay: 0,
        usingSpringWithDamping: 0.75,
        initialSpringVelocity: 0.5,
        animations: {
            view.transform = .identity
            view.alpha = 1
        }
    )
}

// Pop in (for cards, dialogs)
func popIn(_ view: UIView) {
    view.transform = CGAffineTransform(scaleX: 0.85, y: 0.85)
    view.alpha = 0
    UIView.animate(
        withDuration: 0.4, delay: 0,
        usingSpringWithDamping: 0.6,
        initialSpringVelocity: 0.5,
        animations: {
            view.transform = .identity
            view.alpha = 1
        }
    )
}
```

---

## 4. CABasicAnimation — Layer Animations

```swift
// Shake animation (error feedback)
func shake(_ view: UIView) {
    let animation = CAKeyframeAnimation(keyPath: "transform.translation.x")
    animation.timingFunction = CAMediaTimingFunction(name: .linear)
    animation.duration = 0.5
    animation.values = [-12, 12, -8, 8, -4, 4, 0]
    view.layer.add(animation, forKey: "shake")
}

// Pulse animation (for attention)
func pulse(_ view: UIView) {
    let animation = CAKeyframeAnimation(keyPath: "transform.scale")
    animation.values = [1.0, 1.12, 1.0]
    animation.duration = 0.4
    animation.timingFunctions = [
        CAMediaTimingFunction(name: .easeOut),
        CAMediaTimingFunction(name: .easeIn)
    ]
    view.layer.add(animation, forKey: "pulse")
}

// Fade in layer
let fadeIn = CABasicAnimation(keyPath: "opacity")
fadeIn.fromValue = 0
fadeIn.toValue = 1
fadeIn.duration = 0.3
fadeIn.timingFunction = CAMediaTimingFunction(name: .easeOut)
layer.add(fadeIn, forKey: "fadeIn")

// Border color animation
let borderAnim = CABasicAnimation(keyPath: "borderColor")
borderAnim.fromValue = UIColor.systemGray4.cgColor
borderAnim.toValue = UIColor.systemBlue.cgColor
borderAnim.duration = 0.2
view.layer.add(borderAnim, forKey: "borderColor")
view.layer.borderColor = UIColor.systemBlue.cgColor
```

---

## 5. Shadow & Elevation

Android'da `ViewCompat.setElevation()` + `layer.shadowColor`.
UIKit'da `CALayer` shadow properties.

```swift
// Card shadow
view.layer.shadowColor = UIColor.black.cgColor
view.layer.shadowOpacity = 0.12
view.layer.shadowRadius = 16        // blur radius
view.layer.shadowOffset = CGSize(width: 0, height: 4)
view.layer.masksToBounds = false    // MUST be false for shadow

// IMPORTANT: shadow + corner radius (use separate layers)
// Method 1: shadow on container, clip on content
containerView.layer.shadowColor = UIColor.black.cgColor
containerView.layer.shadowOpacity = 0.1
containerView.layer.shadowRadius = 12
containerView.layer.shadowOffset = CGSize(width: 0, height: 4)
containerView.layer.masksToBounds = false

contentView.layer.cornerRadius = 16
contentView.layer.masksToBounds = true   // clips content
containerView.addSubview(contentView)

// Method 2: UIBezierPath shadow (best performance)
view.layer.cornerRadius = 16
view.layer.masksToBounds = false
view.layer.shadowPath = UIBezierPath(
    roundedRect: view.bounds,
    cornerRadius: 16
).cgPath
view.layer.shadowColor = UIColor.black.cgColor
view.layer.shadowOpacity = 0.12
view.layer.shadowRadius = 8
view.layer.shadowOffset = CGSize(width: 0, height: 4)

// Colored shadow (for primary buttons)
button.layer.shadowColor = UIColor.systemBlue.withAlphaComponent(0.4).cgColor
button.layer.shadowOpacity = 1.0
button.layer.shadowRadius = 12
button.layer.shadowOffset = CGSize(width: 0, height: 6)
```

---

## 6. Corner Radius — Concentric Rule

Android'da `inner = outer - padding`.
UIKit'da ayniy qoida: `layer.cornerRadius`.

```swift
// BAD
outerView.layer.cornerRadius = 20
innerView.layer.cornerRadius = 20   // ← same = looks wrong

// GOOD
let outerRadius: CGFloat = 20
let padding: CGFloat = 8
let innerRadius = outerRadius - padding   // = 12

outerView.layer.cornerRadius = outerRadius
innerView.layer.cornerRadius = innerRadius   // ← correct

// Specific corners (e.g., top only)
view.layer.cornerRadius = 16
view.layer.maskedCorners = [.layerMinXMinYCorner, .layerMaxXMinYCorner]

// All corners
view.layer.maskedCorners = [
    .layerMinXMinYCorner,  // top-left
    .layerMaxXMinYCorner,  // top-right
    .layerMinXMaxYCorner,  // bottom-left
    .layerMaxXMaxYCorner   // bottom-right
]

// Continuous corners (iOS 13+ — smoother, Apple-style)
view.layer.cornerCurve = .continuous
view.layer.cornerRadius = 20
```

---

## 7. Image Polish

Android'da `ShapeableImageView`.
UIKit'da `CALayer + clipsToBounds`.

```swift
// Circle avatar
imageView.layer.cornerRadius = imageView.frame.width / 2
imageView.clipsToBounds = true
imageView.layer.borderWidth = 2
imageView.layer.borderColor = UIColor.systemGray5.cgColor

// Rounded rectangle image
imageView.layer.cornerRadius = 12
imageView.layer.cornerCurve = .continuous
imageView.clipsToBounds = true
imageView.layer.borderWidth = 1
imageView.layer.borderColor = UIColor.black.withAlphaComponent(0.06).cgColor

// Gradient overlay on image (for text readability)
func addGradientOverlay(to imageView: UIImageView) {
    let gradientLayer = CAGradientLayer()
    gradientLayer.frame = imageView.bounds
    gradientLayer.colors = [
        UIColor.clear.cgColor,
        UIColor.black.withAlphaComponent(0.7).cgColor
    ]
    gradientLayer.locations = [0.4, 1.0]
    gradientLayer.startPoint = CGPoint(x: 0.5, y: 0)
    gradientLayer.endPoint = CGPoint(x: 0.5, y: 1)
    imageView.layer.addSublayer(gradientLayer)
}

// Shimmer placeholder
class ShimmerView: UIView {
    private let gradientLayer = CAGradientLayer()

    override init(frame: CGRect) {
        super.init(frame: frame)
        setup()
    }

    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setup()
    }

    private func setup() {
        gradientLayer.colors = [
            UIColor.systemGray5.cgColor,
            UIColor.systemGray6.cgColor,
            UIColor.systemGray5.cgColor
        ]
        gradientLayer.locations = [0, 0.5, 1]
        gradientLayer.startPoint = CGPoint(x: 0, y: 0.5)
        gradientLayer.endPoint = CGPoint(x: 1, y: 0.5)
        layer.addSublayer(gradientLayer)

        let animation = CABasicAnimation(keyPath: "locations")
        animation.fromValue = [-1.0, -0.5, 0.0]
        animation.toValue = [1.0, 1.5, 2.0]
        animation.duration = 1.2
        animation.repeatCount = .infinity
        gradientLayer.add(animation, forKey: "shimmer")
    }

    override func layoutSubviews() {
        super.layoutSubviews()
        gradientLayer.frame = bounds
    }
}
```

---

## 8. UITableView / UICollectionView Stagger

Android'da `StaggerAnimator` (DefaultItemAnimator subclass).
UIKit'da `willDisplay` + animation delay.

```swift
// UITableView stagger
func tableView(_ tableView: UITableView,
               willDisplay cell: UITableViewCell,
               forRowAt indexPath: IndexPath) {
    cell.alpha = 0
    cell.transform = CGAffineTransform(translationX: 0, y: 30)

    UIView.animate(
        withDuration: 0.4,
        delay: Double(indexPath.row) * 0.05,
        usingSpringWithDamping: 0.75,
        initialSpringVelocity: 0.5,
        animations: {
            cell.alpha = 1
            cell.transform = .identity
        }
    )
}

// UICollectionView stagger
func collectionView(_ collectionView: UICollectionView,
                    willDisplay cell: UICollectionViewCell,
                    forItemAt indexPath: IndexPath) {
    cell.alpha = 0
    cell.transform = CGAffineTransform(scaleX: 0.85, y: 0.85)

    UIView.animate(
        withDuration: 0.35,
        delay: Double(indexPath.item) * 0.04,
        usingSpringWithDamping: 0.7,
        initialSpringVelocity: 0.5,
        animations: {
            cell.alpha = 1
            cell.transform = .identity
        }
    )
}

// Reset cells for proper stagger on reload
func tableView(_ tableView: UITableView,
               cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(...)
    cell.alpha = 0   // reset for stagger
    return cell
}
```

---

## 9. CAGradientLayer

Android'da `Brush.verticalGradient`.
UIKit'da `CAGradientLayer`.

```swift
// Background gradient
let gradientLayer = CAGradientLayer()
gradientLayer.frame = view.bounds
gradientLayer.colors = [
    UIColor.systemBlue.cgColor,
    UIColor.systemPurple.cgColor
]
gradientLayer.startPoint = CGPoint(x: 0, y: 0)
gradientLayer.endPoint = CGPoint(x: 1, y: 1)
view.layer.insertSublayer(gradientLayer, at: 0)

// Gradient border
func addGradientBorder(to view: UIView, cornerRadius: CGFloat) {
    let gradient = CAGradientLayer()
    gradient.frame = view.bounds
    gradient.colors = [UIColor.systemBlue.cgColor, UIColor.systemPurple.cgColor]
    gradient.startPoint = CGPoint(x: 0, y: 0)
    gradient.endPoint = CGPoint(x: 1, y: 1)

    let shapeLayer = CAShapeLayer()
    shapeLayer.lineWidth = 2
    shapeLayer.path = UIBezierPath(roundedRect: view.bounds, cornerRadius: cornerRadius).cgPath
    shapeLayer.fillColor = nil
    shapeLayer.strokeColor = UIColor.black.cgColor
    gradient.mask = shapeLayer

    view.layer.addSublayer(gradient)
}

// Animated gradient
class AnimatedGradientView: UIView {
    private let gradientLayer = CAGradientLayer()
    private let colors: [[CGColor]] = [
        [UIColor.systemBlue.cgColor, UIColor.systemPurple.cgColor],
        [UIColor.systemPurple.cgColor, UIColor.systemPink.cgColor],
        [UIColor.systemPink.cgColor, UIColor.systemBlue.cgColor]
    ]
    private var currentIndex = 0

    override init(frame: CGRect) {
        super.init(frame: frame)
        gradientLayer.colors = colors[0]
        layer.addSublayer(gradientLayer)
        animate()
    }

    private func animate() {
        currentIndex = (currentIndex + 1) % colors.count
        let animation = CABasicAnimation(keyPath: "colors")
        animation.toValue = colors[currentIndex]
        animation.duration = 3.0
        animation.fillMode = .forwards
        animation.isRemovedOnCompletion = false
        gradientLayer.add(animation, forKey: "gradientAnimation")
        gradientLayer.colors = colors[currentIndex]
        DispatchQueue.main.asyncAfter(deadline: .now() + 3) { [weak self] in
            self?.animate()
        }
    }
}
```

---

## 10. UIVisualEffectView — Blur

```swift
// Frosted glass card
let blurEffect = UIBlurEffect(style: .systemMaterial)
let blurView = UIVisualEffectView(effect: blurEffect)
blurView.frame = card.bounds
blurView.layer.cornerRadius = 16
blurView.clipsToBounds = true
card.insertSubview(blurView, at: 0)

// Vibrancy (text on blur)
let vibrancyEffect = UIVibrancyEffect(blurEffect: blurEffect)
let vibrancyView = UIVisualEffectView(effect: vibrancyEffect)
vibrancyView.frame = blurView.contentView.bounds
blurView.contentView.addSubview(vibrancyView)

let label = UILabel()
label.text = "Blurred card text"
vibrancyView.contentView.addSubview(label)

// Animated blur (appear/disappear)
func showBlur() {
    blurView.effect = nil
    UIView.animate(withDuration: 0.3) {
        blurView.effect = UIBlurEffect(style: .systemMaterial)
    }
}
```

---

## 11. Haptic Feedback

Android'da `Vibrator + VibrationEffect`.
UIKit'da `UIImpactFeedbackGenerator`.

```swift
// Light tap
UIImpactFeedbackGenerator(style: .light).impactOccurred()

// Medium (button press)
UIImpactFeedbackGenerator(style: .medium).impactOccurred()

// Heavy (destructive)
UIImpactFeedbackGenerator(style: .heavy).impactOccurred()

// Success (form submit)
UINotificationFeedbackGenerator().notificationOccurred(.success)

// Error (form validation fail)
UINotificationFeedbackGenerator().notificationOccurred(.error)

// Warning
UINotificationFeedbackGenerator().notificationOccurred(.warning)

// Selection changed (tab switch, picker)
UISelectionFeedbackGenerator().selectionChanged()

// Pre-warm for low latency
class ButtonVC: UIViewController {
    private let impactGenerator = UIImpactFeedbackGenerator(style: .medium)

    override func viewDidLoad() {
        super.viewDidLoad()
        impactGenerator.prepare()   // pre-warm
    }

    @IBAction func buttonTapped() {
        impactGenerator.impactOccurred()
        impactGenerator.prepare()   // re-warm for next tap
    }
}
```

---

## 12. Optical Alignment

```swift
// Icon + Label optical alignment
let stack = UIStackView()
stack.axis = .horizontal
stack.spacing = 8
stack.alignment = .center

let icon = UIImageView(image: UIImage(systemName: "star.fill"))
icon.contentMode = .scaleAspectFit
icon.frame = CGRect(x: 0, y: 0, width: 20, height: 20)
// Optical push: add 1pt bottom inset to icon
let iconContainer = UIView()
iconContainer.frame = CGRect(x: 0, y: 0, width: 20, height: 20)
iconContainer.addSubview(icon)
icon.frame.origin.y = 1   // optical push down 1pt

stack.addArrangedSubview(iconContainer)
stack.addArrangedSubview(label)

// Storyboard: add 1pt top constraint to icon
// (icon top constraint = container.top + 1)
```

---

## 13. Spacing Rhythm

```swift
// Constants
enum Spacing {
    static let xs:  CGFloat = 4
    static let sm:  CGFloat = 8
    static let md:  CGFloat = 16
    static let lg:  CGFloat = 24
    static let xl:  CGFloat = 32
    static let xxl: CGFloat = 48
}

// UIStackView spacing
stackView.spacing = Spacing.sm

// Insets
contentView.layoutMargins = UIEdgeInsets(
    top: Spacing.lg,      // more top (optical)
    left: Spacing.md,
    bottom: Spacing.md,
    right: Spacing.md
)

// Subtle divider
let divider = UIView()
divider.backgroundColor = UIColor.black.withAlphaComponent(0.08)
divider.frame = CGRect(x: Spacing.md, y: y, width: width - Spacing.md * 2, height: 1)
```

---

## Helper Extensions

```swift
extension UIView {
    // Animate visibility
    func fadeIn(duration: TimeInterval = 0.3) {
        alpha = 0; isHidden = false
        UIView.animate(withDuration: duration) { self.alpha = 1 }
    }

    func fadeOut(duration: TimeInterval = 0.2, completion: (() -> Void)? = nil) {
        UIView.animate(withDuration: duration, animations: { self.alpha = 0 }) { _ in
            self.isHidden = true
            completion?()
        }
    }

    func popIn(duration: TimeInterval = 0.35) {
        transform = CGAffineTransform(scaleX: 0.85, y: 0.85)
        alpha = 0; isHidden = false
        UIView.animate(
            withDuration: duration, delay: 0,
            usingSpringWithDamping: 0.6, initialSpringVelocity: 0.5
        ) {
            self.transform = .identity
            self.alpha = 1
        }
    }

    func slideUp(duration: TimeInterval = 0.4) {
        transform = CGAffineTransform(translationX: 0, y: 40)
        alpha = 0; isHidden = false
        UIView.animate(
            withDuration: duration, delay: 0,
            usingSpringWithDamping: 0.75, initialSpringVelocity: 0.5
        ) {
            self.transform = .identity
            self.alpha = 1
        }
    }

    // Apply card shadow
    func applyCardShadow(
        radius: CGFloat = 8,
        opacity: Float = 0.12,
        offset: CGSize = CGSize(width: 0, height: 4),
        cornerRadius: CGFloat = 12
    ) {
        layer.shadowColor = UIColor.black.cgColor
        layer.shadowOpacity = opacity
        layer.shadowRadius = radius
        layer.shadowOffset = offset
        layer.masksToBounds = false
        layer.shadowPath = UIBezierPath(
            roundedRect: bounds,
            cornerRadius: cornerRadius
        ).cgPath
    }
}

extension CGFloat {
    var dpToPt: CGFloat { return self }   // on iOS, pt ≈ dp (same unit)
}
```

---

## Checklist — Before / After (UIKit)

- [ ] Labels: `NSAttributedString` with kern + lineSpacing
- [ ] Buttons: `isHighlighted` override with spring animation
- [ ] Views: press scale via gesture recognizer
- [ ] Show/hide: `fadeIn()` / `popIn()` (not `isHidden = true/false`)
- [ ] Cards: `CALayer` shadow (masksToBounds = false!)
- [ ] Images: `layer.cornerRadius + clipsToBounds`
- [ ] Concentric corners: inner = outer - padding
- [ ] TableView/CollectionView: stagger in `willDisplay`
- [ ] Images: shimmer `ShimmerView` placeholder
- [ ] Haptics: `UIImpactFeedbackGenerator` on primary actions
- [ ] Gradients: `CAGradientLayer` for backgrounds/overlays
- [ ] Blur: `UIVisualEffectView` for glass cards
- [ ] Spacing: `Spacing` enum used consistently
- [ ] Shadow path: `UIBezierPath` used for performance
