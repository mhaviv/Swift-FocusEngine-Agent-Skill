# Focus Anti-Patterns (All Platforms)

These are critical mistakes that break focus navigation. Flag any occurrence immediately. Patterns 1-14 are primarily tvOS. Patterns 15-21 are macOS-specific.

## Blocking (must fix before ship)

### 1. Using `.disabled()` instead of `.allowsHitTesting(false)`

`.disabled(true)` removes the view from the focus chain entirely on tvOS. Focus jumps unpredictably to a distant view. This is the #1 cause of "focus jumpiness."

```swift
// BAD
Button("Watch") { ... }
    .disabled(isLoading)

// GOOD
Button("Watch") { ... }
    .allowsHitTesting(!isLoading)
    .opacity(isLoading ? 0.5 : 1.0)
```

UIKit equivalent: `UIButton.isEnabled = false` also makes the button unfocusable.

### 2. Missing `.focusSection()` on horizontal ScrollViews in vertical layouts

Without `.focusSection()`, swiping down from a horizontal row jumps focus diagonally to a misaligned item in the next row. The focus engine needs this to treat each row as a logical group.

```swift
// BAD
VStack {
    ScrollView(.horizontal) { HStack { /* row 1 */ } }
    ScrollView(.horizontal) { HStack { /* row 2 */ } }
}

// GOOD
VStack {
    ScrollView(.horizontal) { HStack { /* row 1 */ } }
        .focusSection()
    ScrollView(.horizontal) { HStack { /* row 2 */ } }
        .focusSection()
}
```

### 3. Adding `.focusable()` to Buttons or NavigationLinks

Buttons and NavigationLinks are already focusable. Adding `.focusable()` wraps them in a second focusable layer, causing double-focus artifacts.

```swift
// BAD
Button("Play") { ... }
    .focusable()

// GOOD
Button("Play") { ... }
```

Exception: `.focusable(isEnabled)` on a custom ButtonStyle's inner view is acceptable for conditional focusability.

### 4. Mixing SwiftUI and UIKit focus on same hierarchy

`@FocusState` and UIKit's focus engine (`setNeedsFocusUpdate`) are separate systems. When both are active on the same hierarchy (e.g., UIHostingController in UIKit), you get two "focused" items. Only the last-operated system's focus is real.

Rule: Pick one system per view hierarchy branch.

### 5. Calling `reloadData()` during animations

Animated data source changes cause the focus engine to lose track of the focused item. Focus jumps to the first item or an unexpected location.

```swift
// BAD
UIView.animate(withDuration: 0.3) { ... }
collectionView.reloadData()

// GOOD — gate focus updates
var allowsFocusUpdate = true

override func shouldUpdateFocus(in context: UIFocusUpdateContext) -> Bool {
    return allowsFocusUpdate
}

func safeReload() {
    allowsFocusUpdate = false
    collectionView.reloadData()
    collectionView.layoutIfNeeded()
    allowsFocusUpdate = true
    setNeedsFocusUpdate()
    updateFocusIfNeeded()
}
```

### 6. Using `frame.width` in focus transform calculations

Reading `frame.width` during layout changes causes jitter because the frame updates mid-animation. Cache the resting width.

```swift
// BAD
let scale: CGFloat = 1.13
let scaledWidth = frame.width * scale  // frame.width changes during layout!

// GOOD
private var restingWidth: CGFloat = 0
override func layoutSubviews() {
    super.layoutSubviews()
    if !isFocused { restingWidth = bounds.width }
}
// Use restingWidth in focus calculations
```

### 7. `setNeedsFocusUpdate()` called from wrong environment

`setNeedsFocusUpdate()` only works if the calling focus environment currently contains the focused view. If it doesn't, the call silently does nothing. This is the most common UIKit focus bug.

```swift
// BAD — calling from a VC that doesn't have focus
otherViewController.setNeedsFocusUpdate()

// GOOD — call from the VC that contains the currently focused view
// or from a common ancestor
self.setNeedsFocusUpdate()
self.updateFocusIfNeeded()
```

### 8. Setting `isUserInteractionEnabled = false` on headers/labels

On tvOS, `isUserInteractionEnabled = false` propagates to all descendants and can permanently break focus traversal through that part of the view hierarchy.

```swift
// BAD
sectionHeader.isUserInteractionEnabled = false

// GOOD — just don't make it focusable (it isn't by default)
// UILabel.canBecomeFocused is already false
```

### 9. `remembersLastFocusedIndexPath` + offscreen `reloadData()`

When `remembersLastFocusedIndexPath = true` and `reloadData()` is called while the collection view is NOT visible (e.g., behind a detail view), the remembered index path may not match what was previously focused.

Workaround: Combine with manual tracking via `indexPathForPreferredFocusedView(in:)`.

### 10. Using `UIView.animate` for CALayer properties

Shadow opacity, shadow radius, and other CALayer properties do not animate inside `UIView.animate` blocks. Use `CABasicAnimation`.

```swift
// BAD
UIView.animate(withDuration: 0.3) {
    self.layer.shadowOpacity = 1.0  // Does NOT animate
}

// GOOD
let anim = CABasicAnimation(keyPath: "shadowOpacity")
anim.fromValue = layer.shadowOpacity
anim.toValue = 1.0
anim.duration = 0.3
layer.add(anim, forKey: "shadowOpacity")
layer.shadowOpacity = 1.0
```

## Warning (should fix)

### 11. Non-optional `@FocusState` with `focused(_:equals:)`

`@FocusState` MUST be Optional when using the `focused($binding, equals:)` overload. Non-optional only works with `focused($bool)`.

```swift
// BAD
@FocusState var field: Field  // Non-optional
TextField("Email", text: $email).focused($field, equals: .email)

// GOOD
@FocusState var field: Field?  // Optional
TextField("Email", text: $email).focused($field, equals: .email)
```

### 12. Missing `prepareForReuse()` cleanup for focus state

Reused UIKit cells retain focused visual state (scale, shadow, highlight) from previous use, causing visual jitter.

```swift
override func prepareForReuse() {
    super.prepareForReuse()
    transform = .identity
    layer.shadowOpacity = 0
    layer.zPosition = 0
    layer.removeAllAnimations()
}
```

### 13. `prefersDefaultFocus` inside ScrollView

`prefersDefaultFocus(_:in:)` does NOT work reliably inside `ScrollView` on tvOS. This is a documented Apple limitation.

Workaround: Use `@FocusState` + `defaultFocus(_:_:priority:)` or set focus programmatically after layout completes.

### 14. LazyVStack/LazyVGrid performance on Apple TV HD

`LazyVStack` and `LazyVGrid` inside `ScrollView` have severe lag on tvOS 18 (Apple TV HD). Consider using `List` or a custom List-based grid instead.

## macOS-Specific Anti-Patterns

### 15. Not overriding `acceptsFirstResponder` on custom NSView

Custom NSView subclasses default to `acceptsFirstResponder = false`. The view silently ignores Tab navigation and `makeFirstResponder` calls. This is the #1 macOS focus bug.

```swift
// BAD — view never receives focus
class MyCustomView: NSView {
    // acceptsFirstResponder defaults to false
}

// GOOD
class MyCustomView: NSView {
    override var acceptsFirstResponder: Bool { true }
    override var canBecomeKeyView: Bool { true }
}
```

### 16. Incomplete key view loop

If the last view's `nextKeyView` doesn't loop back to the first view, Tab navigation stops working after reaching the end. Shift-Tab from the first view also fails.

```swift
// BAD — Tab reaches buttonC and stops
textField.nextKeyView = buttonA
buttonA.nextKeyView = buttonB
buttonB.nextKeyView = buttonC
// No loop back!

// GOOD — complete the loop
buttonC.nextKeyView = textField
```

Alternative: Set `window.recalculatesKeyViewLoop = true` and let the system manage the loop geometrically. But never mix manual `nextKeyView` with `recalculatesKeyViewLoop`.

### 17. Calling `becomeFirstResponder()` directly

Never call `becomeFirstResponder()` on a view directly. It's meant to be called by the system during `makeFirstResponder(_:)`.

```swift
// BAD — bypasses the resign/become handshake
myTextField.becomeFirstResponder()

// GOOD — proper focus handshake via window
view.window?.makeFirstResponder(myTextField)
```

Direct calls skip `resignFirstResponder()` on the current first responder, which can leave the previous view in a bad state (e.g., text editing still active).

### 18. NSPanel stealing focus from main window

Panels (inspectors, tool windows) default to becoming the key window, stealing focus from the document. Users lose their cursor position in text editors.

```swift
// BAD — inspector panel steals focus on every show
let panel = NSPanel(...)
panel.makeKeyAndOrderFront(nil)

// GOOD — panel only takes focus when user explicitly clicks inside it
panel.becomesKeyOnlyIfNeeded = true
panel.orderFront(nil)  // Show without stealing focus
```

### 19. Not restoring focus after sheet dismissal

When an NSAlert or sheet is dismissed, focus should return to the view that was focused before the sheet appeared. SwiftUI handles this automatically, but AppKit requires manual tracking.

```swift
// BAD — focus goes to window, not original view
alert.runModal()

// GOOD — save and restore
let savedFirstResponder = window.firstResponder
alert.beginSheetModal(for: window) { _ in
    self.window.makeFirstResponder(savedFirstResponder)
}
```

### 20. Using `.focusable()` on NSViewRepresentable without bridging

Adding `.focusable()` to a SwiftUI view wrapping AppKit via `NSViewRepresentable` creates a SwiftUI focus layer that doesn't coordinate with AppKit's first responder. The AppKit view handles its own focus.

```swift
// BAD — double focus, SwiftUI ring + AppKit ring
struct MyAppKitView: NSViewRepresentable { ... }
MyAppKitView()
    .focusable()  // Don't add this

// GOOD — let AppKit handle focus natively
struct MyAppKitView: NSViewRepresentable {
    func makeNSView(context: Context) -> NSView {
        let view = MyNSView()
        // NSView handles its own acceptsFirstResponder
        return view
    }
}
```

### 21. Not disabling menu items when no document is focused

Menu items that depend on `focusedValue` but don't check for nil remain enabled when no window is key (e.g., all windows minimized), leading to crashes or no-ops.

```swift
// BAD — crashes when document is nil
Button("Save") { document!.save() }

// GOOD — disable when no focused document
Button("Save") { document?.save() }
    .disabled(document == nil)
```
