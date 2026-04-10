# macOS Focus Management

macOS uses a **key view loop** model — focus moves between views via Tab/Shift-Tab in a loop defined by the responder chain. This is fundamentally different from tvOS (geometric/spatial) and iOS (focus groups activated by hardware keyboard).

## Core Concept: Key View Loop

Every `NSWindow` maintains a loop of focusable views. Tab advances forward, Shift-Tab goes backward.

```
┌─ TextField ──► Button ──► PopUpButton ──► TableView ──┐
└───────────────────────────────────────────────────────┘
```

### NSView Focus APIs

```swift
class MyCustomView: NSView {
    // Required: Declare the view can accept focus
    override var acceptsFirstResponder: Bool { true }

    // Required for Tab key navigation
    override var canBecomeKeyView: Bool { true }

    // Called when view gains focus
    override func becomeFirstResponder() -> Bool {
        needsDisplay = true
        return super.becomeFirstResponder()
    }

    // Called when view loses focus
    override func resignFirstResponder() -> Bool {
        needsDisplay = true
        return super.resignFirstResponder()
    }
}
```

### Window First Responder

```swift
// Make a view focused
window.makeFirstResponder(myView)

// Check what's currently focused
if let focused = window.firstResponder as? NSView {
    print("Focused: \(focused)")
}

// Resign focus (focus goes to window itself)
window.makeFirstResponder(nil)
```

### Key View Loop Setup

**Interface Builder:** Connect `nextKeyView` outlets in sequence, with the last view pointing back to the first.

**Programmatic:**
```swift
textField.nextKeyView = button
button.nextKeyView = popUpButton
popUpButton.nextKeyView = textField  // Complete the loop
```

**Auto-recalculation:**
```swift
window.recalculatesKeyViewLoop = true  // System manages the loop
// System uses geometric position (left-to-right, top-to-bottom) to determine order
```

Common mistake: Setting `recalculatesKeyViewLoop = true` AND manually setting `nextKeyView`. The manual chain gets overwritten.

## SwiftUI Focus on macOS

### @FocusState

Works the same as iOS. On macOS, focus is always active (no hardware keyboard requirement like iOS).

```swift
enum Field: Hashable { case search, name, email }
@FocusState private var focusedField: Field?

VStack {
    TextField("Search", text: $search)
        .focused($focusedField, equals: .search)
    TextField("Name", text: $name)
        .focused($focusedField, equals: .name)
    TextField("Email", text: $email)
        .focused($focusedField, equals: .email)
}
.onAppear { focusedField = .search }  // Auto-focus on appear
```

Key difference from iOS: Setting `focusedField = nil` on macOS does NOT dismiss the keyboard (there's no virtual keyboard to dismiss). It moves focus to the window itself.

### .focusable() on macOS

Makes non-interactive views focusable for keyboard navigation:

```swift
CardView(item: item)
    .focusable()
    .onKeyPress(.return) {
        openItem(item)
        return .handled
    }
```

On macOS, `.focusable()` views participate in the Tab loop automatically.

### .focusable(interactions:) (macOS 14+)

```swift
.focusable(interactions: .edit)     // For text-editing-like views
.focusable(interactions: .activate) // For button-like views
```

On macOS, `.activate` responds to Tab without requiring a system toggle (unlike iOS which requires "Keyboard Navigation" to be enabled).

### defaultFocus(_:_:priority:) (macOS 14+)

```swift
@FocusState var selectedField: Field?

VStack { ... }
    .defaultFocus($selectedField, .search, priority: .userInitiated)
```

### .focusSection() (macOS 14+)

Groups focusable views for arrow key navigation within a section:

```swift
HStack {
    VStack {
        // Sidebar items
    }
    .focusSection()

    VStack {
        // Content items
    }
    .focusSection()
}
```

Arrow keys move within a section; Tab moves between sections. This is the macOS equivalent of how focus sections work on tvOS, but with keyboard-driven navigation instead of remote-driven.

## Focus Ring (Focus Indicator)

macOS draws a system focus ring (blue glow) around focused views. This is the macOS equivalent of tvOS's focus scale/highlight and iOS's halo effect.

### NSView Focus Ring

```swift
class MyView: NSView {
    // Control focus ring visibility
    override var focusRingType: NSFocusRingType {
        return .exterior  // .exterior (default), .interior, .none
    }

    // Customize focus ring shape (default: bounds)
    override var focusRingMaskBounds: NSRect {
        return bounds.insetBy(dx: 4, dy: 4)
    }

    override func drawFocusRingMask() {
        // Draw a rounded rect focus ring
        NSBezierPath(roundedRect: focusRingMaskBounds,
                     xRadius: 8, yRadius: 8).fill()
    }

    // Tell AppKit when focus ring mask changes
    override func noteFocusRingChanged() {
        // Called when the ring needs to redraw
    }
}
```

### SwiftUI Focus Ring

```swift
// System focus ring (default on macOS)
TextField("Name", text: $name)
    .focusable()

// Suppress system focus ring
TextField("Name", text: $name)
    .focusable()
    .focusEffectDisabled()  // No ring drawn

// Custom focus indicator
@FocusState var isFocused: Bool

TextField("Name", text: $name)
    .focused($isFocused)
    .focusEffectDisabled()
    .overlay(
        RoundedRectangle(cornerRadius: 8)
            .stroke(isFocused ? .blue : .clear, lineWidth: 2)
    )
```

Common mistake: Disabling the focus ring without providing an alternative visual indicator. Users need to see what's focused.

### NSFocusRingPlacement

When drawing views that contain a focus ring:

```swift
// In draw(_:)
NSGraphicsContext.saveGraphicsState()
NSFocusRingPlacement.only.set()
path.fill()   // Only the focus ring is drawn, not the fill
NSGraphicsContext.restoreGraphicsState()
```

## focusedValue / focusedSceneValue (Menu Commands)

On macOS, `focusedValue` is critical for making menu bar commands respond to the currently focused content. This is the primary use case — menus enable/disable and change behavior based on what's focused.

### Menu Bar Integration

```swift
// Define the key
struct SelectedDocumentKey: FocusedValueKey {
    typealias Value = Document
}

extension FocusedValues {
    var selectedDocument: Document? {
        get { self[SelectedDocumentKey.self] }
        set { self[SelectedDocumentKey.self] = newValue }
    }
}

// Set from your view
DocumentView(document: document)
    .focusedValue(\.selectedDocument, document)

// Read in Commands
struct AppCommands: Commands {
    @FocusedValue(\.selectedDocument) var document

    var body: some Commands {
        CommandGroup(after: .pasteboard) {
            Button("Export PDF") { document?.exportPDF() }
                .disabled(document == nil)
                .keyboardShortcut("e", modifiers: [.command, .shift])
        }
    }
}
```

### Multi-Window with focusedSceneValue

macOS apps commonly have multiple windows. `focusedSceneValue` propagates from whichever window is key:

```swift
WindowGroup {
    EditorView()
        .focusedSceneValue(\.activeDocument, document)
}

// Menu commands automatically target the key window's document
@FocusedValue(\.activeDocument) var activeDocument
```

### @FocusedObject (macOS 12+)

Pass an entire ObservableObject through focus:

```swift
.focusedObject(editorModel)

// In Commands:
@FocusedObject var editor: EditorModel?
```

## NSTableView / NSOutlineView / NSCollectionView Focus

### Table View

```swift
// Keyboard navigation is enabled by default
// Arrow keys move selection, Tab moves to next key view
tableView.allowsEmptySelection = false  // Ensure something is always selected

// React to selection changes (which follow focus)
func tableViewSelectionDidChange(_ notification: Notification) {
    // Selection changed via keyboard or mouse
}
```

### Collection View

```swift
// macOS 10.13+: Keyboard focus navigation
collectionView.isSelectable = true
collectionView.allowsEmptySelection = false

// selectionIndexPaths tracks focused/selected items
```

### Type-to-Select

NSTableView and NSOutlineView support type-to-select by default — typing characters jumps to matching rows. This is separate from focus/selection.

```swift
// Disable if it conflicts with your search field
tableView.allowsTypeSelect = false
```

## Full Keyboard Access on macOS

When enabled in System Settings > Keyboard > Keyboard Navigation, ALL controls become focusable via Tab — not just text fields and lists.

```swift
// Check if Full Keyboard Access is on
NSApplication.shared.isFullKeyboardAccessEnabled

// Views can check at runtime
override var canBecomeKeyView: Bool {
    // Return true always, or only when FKA is enabled
    return NSApplication.shared.isFullKeyboardAccessEnabled || alwaysFocusable
}
```

Important: Even without Full Keyboard Access, users can Tab between text fields and lists. FKA adds buttons, checkboxes, sliders, pop-ups, etc.

## Mac Catalyst Focus

Mac Catalyst apps run iOS UIKit code on macOS. Focus behavior bridges both worlds:

### What Works Automatically
- `UIFocusSystem` is active (same as iPad with keyboard)
- `UIFocusHaloEffect` renders as macOS focus ring
- `focusGroupIdentifier` maps to Tab navigation groups
- Tab/Shift-Tab navigation works

### What's Different
- `canBecomeFocused` must return `true` for custom views
- macOS menu bar integration requires `UIMenuBuilder` or SwiftUI Commands
- Focus ring appearance follows macOS system settings, not iOS halo styling
- `UIFocusSystem.focusSystem(for:)` — check it exists before using, nil if focus is unavailable

### Common Catalyst Mistake
Forgetting that Mac Catalyst inherits iPad focus behavior. If your iPad app doesn't support keyboard focus (no `UIFocusHaloEffect`, no `allowsFocus` on collection views), your Mac Catalyst app won't support it either.

## macOS vs Other Platforms

| Feature | macOS | tvOS | iOS/iPadOS |
|---------|-------|------|-----------|
| Focus model | Key view loop | Geometric/spatial | Focus groups (keyboard) |
| Always active | Yes | Yes | No (keyboard required) |
| Tab behavior | Next key view | N/A | Next focus group |
| Arrow keys | Within controls | Directional focus | Within focus group |
| Focus indicator | Blue ring | Scale/highlight | Halo |
| Primary input | Mouse + keyboard | Siri Remote | Touch |
| focusedValue | Menu commands | N/A | Menu commands |
| .focusSection() | macOS 14+ | tvOS 15+ | iOS 17+ |
| @FocusState | macOS 12+ | tvOS 15+ | iOS 15+ |
| FKA toggle | System Settings | N/A | Settings > Accessibility |

## Common macOS Focus Mistakes

### 1. Not completing the key view loop
If the last view's `nextKeyView` doesn't point back to the first, Tab stops working after reaching the end.

### 2. Forgetting acceptsFirstResponder
Custom NSView subclasses return `false` by default. Without overriding to `true`, the view is invisible to Tab navigation.

### 3. Focus ring on custom-drawn views
If you draw content with custom insets or shapes, the default rectangular focus ring looks wrong. Override `drawFocusRingMask()` and `focusRingMaskBounds`.

### 4. Conflicting recalculatesKeyViewLoop with manual nextKeyView
Setting `recalculatesKeyViewLoop = true` overrides all manual `nextKeyView` connections. Pick one approach.

### 5. Assuming Full Keyboard Access is always on
Most users don't enable it. Your custom views that rely on Tab focus for non-text-field controls may not receive focus for most users. Always provide mouse/trackpad interaction as primary.

### 6. Not using focusedValue for menu commands
Menu bar items that don't use `focusedValue` can't respond to the current selection. The menu item stays enabled/disabled regardless of context.

### 7. SwiftUI focus ring doubling
Using `.focusable()` on a view that already has system focus support (like TextField) can cause double focus rings.

## WWDC Sessions

| Session | Year | Key Content |
|---------|------|-------------|
| Support keyboard navigation in macOS | WWDC21 | Key view loop, focus ring customization |
| Direct and reflect focus in SwiftUI | WWDC21 | @FocusState, focusedValue on macOS |
| Bring your iOS app to the Mac | WWDC19 | Mac Catalyst focus behavior |
| The SwiftUI cookbook for focus | WWDC23 | .focusable(interactions:), cross-platform focus |
| What's new in AppKit | WWDC24 | Focus ring improvements, NSFocusRingPlacement updates |
