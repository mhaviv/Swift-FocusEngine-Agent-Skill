# watchOS Focus Management

watchOS focus exists primarily to route **Digital Crown input** to the correct view. There is no keyboard, remote, or cursor. The Digital Crown is the sole non-touch input device.

## How Focus Works on watchOS

- When a view has focus, the system draws a **green border** around it
- Rotating the Digital Crown changes that focused view's value
- Focus is **sequential** (layout order), not spatial/directional like tvOS
- System controls (List, ScrollView, Picker, Stepper, Toggle) handle Digital Crown automatically — no `.focusable()` needed
- Only SwiftUI focus APIs exist on watchOS — there is **no UIFocusEnvironment** equivalent (WatchKit has no focus protocol)
- `.focusSection()` is **NOT available** on watchOS
- watchOS does NOT support keyboard focus

## Available APIs

### @FocusState (watchOS 8.0+)

Controls which view receives Digital Crown input.

```swift
enum Field: Hashable { case amount, tip }
@FocusState private var focusedField: Field?

Form {
    TextField("Amount", value: $amount, format: .currency(code: "USD"))
        .focused($focusedField, equals: .amount)
    Picker("Tip", selection: $tipPercent) { /* ... */ }
        .focused($focusedField, equals: .tip)
}
```

### .focusable() (watchOS 8.0+)

Makes custom views capable of receiving Digital Crown input. The green focus ring appears when focused.

```swift
Text("Value: \(crownValue, specifier: "%.1f")")
    .focusable()                              // MUST come FIRST
    .digitalCrownRotation($crownValue,
                          from: 0.0,
                          through: 10.0,
                          sensitivity: .low,
                          isContinuous: false,
                          isHapticFeedbackEnabled: true)
```

**Critical ordering rule**: `.focusable()` MUST be placed BEFORE `.digitalCrownRotation()` in the modifier chain. Reversing the order silently breaks crown input.

### .focusable(interactions:) (watchOS 10.0+)

Fine-grained control:
- `.edit` — captures Digital Crown rotation (sliders, steppers)
- `.activate` — primary action via focus (button-like)
- `.automatic` — platform-appropriate defaults

### .digitalCrownRotation() (watchOS 6.0+, enhanced watchOS 9.0+)

Binds Digital Crown hardware to a property.

Parameters: `binding`, `from`/`through` (range), `by` (stride for haptic stops), `sensitivity` (.low/.medium/.high), `isContinuous` (wrap-around), `isHapticFeedbackEnabled`, `onChange`, `onIdle`.

Pattern types:
- **Free-scrolling**: No `by` parameter — smooth continuous rotation
- **Picker-style**: With `by` stride — discrete haptic stops
- **Circular**: With `isContinuous: true` — wraps around at bounds

### .digitalCrownAccessory() (watchOS 9.0+)

Adds or controls visibility of an accessory view near the Digital Crown indicator.

### prefersDefaultFocus(_:in:) (watchOS 7.0+)

Controls which view initially gets Digital Crown focus.

```swift
@Namespace var formScope

Form {
    Picker("Size", selection: $size) { /* ... */ }
        .prefersDefaultFocus(true, in: formScope)
    Stepper("Count", value: $count)
}
.focusScope(formScope)
```

### defaultFocus(_:_:priority:) (watchOS 9.0+)

Modern alternative to `prefersDefaultFocus`:

```swift
@FocusState private var focusedField: Field?

Form { ... }
    .defaultFocus($focusedField, .name)
```

### focusScope(_:) (watchOS 7.0+)

Required companion for `prefersDefaultFocus` and `resetFocus`. Creates a focus namespace scope.

### resetFocus (watchOS 7.0+)

Programmatically re-evaluates default focus within a namespace:

```swift
@Namespace var mainScope
@Environment(\.resetFocus) var resetFocus

Button("Reset") {
    resetFocus(in: mainScope)
}
```

### @Environment(\.isFocused) (watchOS 7.0+)

Read-only — returns true when nearest focusable ancestor has focus. Use for custom focus visuals.

### .focusEffectDisabled() (watchOS 10.0+)

Suppresses the default green focus ring. Use when providing custom focus visuals via `isFocused`.

## Digital Crown + List/ScrollView Interaction

- `List` and `ScrollView` automatically receive Digital Crown scroll input — no focus modifiers needed
- When a `Form` contains focusable controls (Picker, Stepper), the first focusable view may grab focus initially, making Crown control that view instead of scrolling
- If the user scrolls with their finger, Crown transfers to form scrolling — **the picker may not regain focus** (known UX pain point)
- Use `defaultFocus` to control which element initially receives crown focus

## Common Mistakes

### 1. Wrong modifier order
`.digitalCrownRotation()` before `.focusable()` silently breaks crown input. `.focusable()` must come first.

### 2. Adding .focusable() to system controls
Causes double green focus rings and requires extra taps to navigate. Picker, Stepper, Toggle, TextField already handle focus.

### 3. Multiple Steppers in a Form
Multiple steppers can simultaneously show the green focus border, confusing users about which one the crown controls.

### 4. Focus loss in scrollable Forms
Pickers inside scrollable Forms may lose focus permanently after the user finger-scrolls.

### 5. Not using focusScope
Without scoping, `prefersDefaultFocus` and `resetFocus` operate on the entire window — unexpected behavior.

## Not Available on watchOS

| API | Why Not |
|-----|---------|
| `.focusSection()` | No directional navigation — watchOS focus is sequential |
| `UIFocusEnvironment` | WatchKit has no focus protocol |
| `UIFocusGuide` | No spatial navigation model |
| `focusGroupIdentifier` | iOS-only concept |
| `UIFocusHaloEffect` | iOS/Catalyst only |
| Keyboard navigation | No keyboard support |

## WWDC Sessions

| Session | Year | watchOS Content |
|---------|------|----------------|
| SwiftUI on watchOS | WWDC19 | Introduced .focusable() + .digitalCrownRotation() |
| Direct and reflect focus in SwiftUI | WWDC21 | @FocusState, prefersDefaultFocus on watchOS |
| The SwiftUI cookbook for focus | WWDC23 | FocusInteractions (.edit for Digital Crown) |
