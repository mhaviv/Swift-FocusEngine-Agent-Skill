# iOS/iPadOS Focus Management

iOS focus is a **secondary** interaction model alongside touch, driven by hardware keyboard (Tab and arrow keys). This is fundamentally different from tvOS where focus is the **primary** interaction.

## Key Difference: Two-Level Navigation

- **Tab key** moves between **focus groups** (significant UI regions)
- **Arrow keys** move *within* a focus group
- **Return** (iPadOS) or **Space** (Mac Catalyst) activates the focused item
- Focus only activates when a hardware keyboard is connected — your app must work without it

This two-level model does NOT exist on tvOS.

## iOS-Only APIs (Not Available on tvOS)

### focusGroupIdentifier (iOS 15+, NOT tvOS)

Assigns a view to a named focus group. Focus groups define what Tab navigates between.

```swift
// UIKit
sidebarContainer.focusGroupIdentifier = "com.myapp.sidebar"
contentContainer.focusGroupIdentifier = "com.myapp.content"
```

Rules:
- Set on the **common ancestor**, not individual focusable items
- UIKit automatically infers groups from the view hierarchy — only override when default grouping is wrong
- Elements with the same identifier belong to the same group
- Tab loop order is derived from focus groups, accounting for reading direction and layout

### UIFocusGroupPriority (iOS 15+, tvOS 15+)

Determines which item is "primary" within a focus group — the item that gets focus when Tab enters the group.

```swift
// Make this the preferred item when Tab enters the group
myButton.focusGroupPriority = .prioritized  // 2000
```

Priority levels:
- `.ignored` (0) — skipped for group primary selection
- `.previouslyFocused` (1000) — was previously focused in this group
- `.prioritized` (2000) — explicitly prioritized
- `.currentlyFocused` (NSIntegerMax) — currently focused

### UIFocusHaloEffect (iOS 15+, NOT tvOS)

System-standard focus ring (halo) for keyboard focus on iPadOS and Mac Catalyst.

```swift
class MyCell: UICollectionViewCell {
    override var focusEffect: UIFocusEffect? {
        // Custom rounded halo matching the image shape
        let halo = UIFocusHaloEffect(
            roundedRect: imageView.bounds,
            cornerRadius: 8,
            curve: .continuous
        )
        halo.referenceView = imageView   // Render above this view
        halo.containerView = contentView // Place halo in this view
        halo.position = .outside         // .inside, .outside, or .automatic
        return halo
    }
}
```

To disable the default halo:
```swift
override var focusEffect: UIFocusEffect? {
    return nil
}
```

Common mistake: Not matching halo shape to content shape (square halo on circular avatar).

### allowsFocus (UICollectionView/UITableView, iOS 15+)

Makes all cells keyboard-focusable.

```swift
collectionView.allowsFocus = true
collectionView.selectionFollowsFocus = true  // Auto-select on focus
```

Without `selectionFollowsFocus`, users must press Return after focusing a cell to select it.

## Shared APIs (iOS + tvOS)

### @FocusState (iOS 15+, tvOS 15+)

Same API, different context. On iOS, primarily used for:
- Keyboard focus management in forms (move between text fields)
- Dismissing keyboard (`focusedField = nil`)

```swift
enum Field: Hashable { case username, password }
@FocusState private var focusedField: Field?

VStack {
    TextField("Username", text: $username)
        .focused($focusedField, equals: .username)
    SecureField("Password", text: $password)
        .focused($focusedField, equals: .password)
    Button("Login") { focusedField = nil }  // Dismiss keyboard
}
.onSubmit { focusedField = .password }  // Tab to next field
```

### .focusSection() (iOS 17+, tvOS 15+)

Available on iOS starting iOS 17. Groups focusable descendants for directional navigation — same as tvOS but arrived later on iOS.

### .focusable(interactions:) (iOS 17+)

Fine-grained control over focus interaction types:

```swift
.focusable(interactions: .edit)     // Text-like (sliders, steppers)
.focusable(interactions: .activate) // Button-like (needs Keyboard Navigation enabled)
```

`.activate` does NOT receive focus on click — requires system "Keyboard Navigation" toggle to be on.

### defaultFocus(_:_:priority:) (iOS 17+, tvOS 16+)

Modern cross-platform default focus API:

```swift
@FocusState var selectedField: Field?

VStack { ... }
    .defaultFocus($selectedField, .name, priority: .userInitiated)
```

### .focusEffectDisabled() (iOS 17+)

Suppresses the default system focus ring:
```swift
MyView()
    .focusable()
    .focusEffectDisabled()
```

### focusedValue / focusedSceneValue

Propagate data based on where focus is in the hierarchy. Used for menu/command systems:

```swift
.focusedValue(\.selectedItem, item)

// In Commands:
@FocusedValue(\.selectedItem) var selectedItem
```

`focusedSceneValue` for multi-window iPad apps.

### UIFocusItemDeferralMode (iOS 15+, tvOS 15+)

Controls whether focus updates are deferred when the user isn't actively using keyboard:

- `.automatic` — system decides (default)
- `.always` — always defer (use for loading indicators that shouldn't steal focus)
- `.never` — never defer (use for items that need immediate focus after programmatic updates)

### UIFocusItemScrollableContainer (iOS 12+, tvOS 12+)

Protocol for custom scrollable containers. UIScrollView already conforms. Implement on custom containers so the focus system can auto-scroll to reveal focused items.

## iOS vs tvOS Quick Reference

| Feature | tvOS | iOS/iPadOS |
|---------|------|-----------|
| Focus always active | Yes | No (keyboard-driven) |
| Focus groups (Tab nav) | No | Yes (iOS 15+) |
| focusGroupIdentifier | No | Yes (iOS 15+) |
| UIFocusHaloEffect | No | Yes (iOS 15+) |
| UIFocusGuide | tvOS 9+ | iOS 15+ |
| @FocusState | tvOS 15+ | iOS 15+ |
| .focusSection() | tvOS 15+ | iOS 17+ |
| .focusable(interactions:) | No | iOS 17+ |
| canBecomeFocused | tvOS 9+ | iOS 15+ |
| Responder chain sync | N/A | Yes |
| .hoverEffect | tvOS 17+ | N/A (use pointer effects) |
| Parallax tilt | tvOS | N/A |
| Siri Remote | Yes | N/A |
| allowsFocus on CV/TV | N/A | iOS 15+ |
| selectionFollowsFocus | N/A | iOS 15+ |

## Common iOS Focus Mistakes

### 1. Assuming focus works like tvOS
On iOS, focus groups constrain arrow key movement. Forgetting to set up proper groups leads to broken keyboard navigation.

### 2. Not testing without keyboard
The focus system only activates when a keyboard is connected. Your app must work perfectly with touch alone.

### 3. Overriding canBecomeFocused carelessly
This affects both regular Tab navigation AND Full Keyboard Access (accessibility). Test with both.

### 4. Modifier order in SwiftUI
`.focused()` must come AFTER `.focusable()`, not before:
```swift
// BAD
MyView().focused($field, equals: .name).focusable()

// GOOD
MyView().focusable().focused($field, equals: .name)
```

### 5. Ignoring selectionFollowsFocus
Without this on collection/table views, users must press Return after focusing a cell to select it — feels clunky.

### 6. Using focusGroupIdentifier on tvOS
It doesn't exist there. Use `.focusSection()` (SwiftUI) or `UIFocusGuide` (UIKit) on tvOS instead.

### 7. Not handling responder chain
On iOS, the focused item must be inside the first responder chain. Detached views cannot receive focus.

## VoiceOver Focus vs UI Focus

These are **completely separate systems**:
- **UI Focus** (`UIFocusSystem`): Keyboard-driven, only active with hardware keyboard
- **VoiceOver Focus** (`UIAccessibilityFocus`): Assistive technology, `isAccessibilityElement` + `accessibilityElements` ordering
- **Full Keyboard Access**: Uses the SAME focus system as Tab navigation, so `canBecomeFocused` affects both
- SwiftUI: `@AccessibilityFocusState` controls VoiceOver focus independently of `@FocusState`

## WWDC Sessions

| Session | Year | Key Content |
|---------|------|-------------|
| Focus on iPad keyboard navigation | WWDC21 | Primary iOS focus session: focus groups, tab loop, halo, allowsFocus |
| Direct and reflect focus in SwiftUI | WWDC21 | @FocusState, .focused, focusSection |
| Support Full Keyboard Access | WWDC21 | FKA uses same focus system as Tab nav |
| The SwiftUI cookbook for focus | WWDC23 | .focusable(interactions:), focus sections on iOS 17 |
