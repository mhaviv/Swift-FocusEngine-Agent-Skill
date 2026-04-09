# Debugging tvOS Focus Issues

## UIFocusDebugger (LLDB Commands)

Available tvOS 11+. Use in Xcode debugger during a breakpoint.

### Check current focus status
```
(lldb) po UIFocusDebugger.status()
```
Shows the currently focused item and its focus environment chain.

### Check why a view can't receive focus
```
(lldb) po UIFocusDebugger.checkFocusability(for: myView)
```
Returns detailed explanation of why the view is or isn't focusable. Checks:
- `canBecomeFocused` return value
- `isHidden`
- `alpha == 0`
- `isUserInteractionEnabled`
- Whether the view is in a window
- Whether an ancestor blocks focus

### Simulate a focus update
```
(lldb) po UIFocusDebugger.simulateFocusUpdateRequest(from: myEnvironment)
```
Walks the preferred focus chain without actually moving focus. Shows which view WOULD receive focus.

### Check focus group tree
```
(lldb) po UIFocusDebugger.checkFocusGroupTree(for: focusSystem)
```
Prints the entire focus group hierarchy.

### List all commands
```
(lldb) po UIFocusDebugger.help()
```

## _whyIsThisViewNotFocusable (Hidden Debug Method)

Not in public API but invaluable for debugging:

```
(lldb) po [myView _whyIsThisViewNotFocusable]
```

Returns human-readable list of issues:
- "userInteractionEnabled set to NO"
- "canBecomeFocused returns NO"
- "view is hidden"
- "alpha is 0"
- "view is obscured by another view"
- "ancestor has userInteractionEnabled = NO"
- "not in a window"

## Launch Arguments

### -UIFocusLoggingEnabled YES

Add to scheme's launch arguments. Logs every focus update to the console with:
- The preferred focus environment search chain
- Which view was selected and why
- Which views were considered and rejected

### How to add
Xcode -> Product -> Scheme -> Edit Scheme -> Run -> Arguments -> "+" -> `-UIFocusLoggingEnabled YES`

## Quick Look on UIFocusUpdateContext

In `shouldUpdateFocus(in:)` or `didUpdateFocus(in:with:)`:
1. Set breakpoint
2. Select the `context` parameter
3. Click Quick Look (eye icon) or press Space

Shows visual diagram:
- **Red**: previously focused view (search start)
- **Dotted red line**: search path
- **Purple**: focusable UIView regions in search path
- **Blue**: focusable UIFocusGuide regions in search path

## Common Focus Issues Checklist

### Focus doesn't move at all
1. Is the target view focusable? (`canBecomeFocused`, or is it a Button/Cell?)
2. Is the target view visible? (not hidden, alpha > 0, in window)
3. Is `isUserInteractionEnabled = true` on the target AND all ancestors?
4. Is there a geometric path from current focus to target? (use Quick Look to see)
5. Is `shouldUpdateFocus(in:)` returning false somewhere in the chain?

### Focus jumps to wrong item
1. Missing `.focusSection()` on horizontal ScrollViews?
2. Are items geometrically aligned? Focus follows nearest-neighbor in swipe direction.
3. Is `remembersLastFocusedIndexPath` conflicting with manual focus management?
4. Is `preferredFocusEnvironments` returning stale references?

### Focus jitters / visual glitch
1. Are you using `frame.width` in transform calculations? Cache the resting width.
2. Is `prepareForReuse()` resetting all focus-related visual state?
3. Are focus animations using `addCoordinatedFocusingAnimations` (not plain UIView.animate)?
4. Is `layer.zPosition` managed to prevent overlap?
5. Are shadow animations using `CABasicAnimation` (not UIView.animate)?

### Focus lost after data reload
1. Is `reloadData()` called during an animation?
2. Is `remembersLastFocusedIndexPath` + offscreen reload causing stale index?
3. Is `shouldUpdateFocus(in:)` blocking during reload?
4. Did you call `setNeedsFocusUpdate()` + `updateFocusIfNeeded()` after reload?
5. Are you calling from the right focus environment (one that contains the focused view)?

### SwiftUI focus not working
1. Is `@FocusState` Optional when using `focused($binding, equals:)`?
2. Is `.focusScope(namespace)` on an ancestor of `prefersDefaultFocus`?
3. Is `.focusSection()` applied to the container, not individual buttons?
4. Is `.disabled()` being used? Replace with `.allowsHitTesting(false)`.
5. Is `.focusable()` added to a Button? Remove it.

## Testing Focus

### UI Test Utilities

```swift
extension XCUIRemote {
    func press(_ button: XCUIRemote.Button, times: Int) {
        for _ in 0..<times {
            press(button)
            Thread.sleep(forTimeInterval: 0.3)
        }
    }
}

extension XCUIApplication {
    func focusedElement() -> XCUIElement {
        return descendants(matching: .any).element(matching: NSPredicate(format: "hasFocus == true"))
    }
}
```

### Focus navigation test pattern

```swift
func testFocusNavigationBetweenRows() {
    let remote = XCUIRemote.shared
    let app = XCUIApplication()

    // Navigate to first row item
    remote.press(.select)
    XCTAssertTrue(app.buttons["Row 0 Item 0"].hasFocus)

    // Move down to second row
    remote.press(.down)
    XCTAssertTrue(app.buttons.matching(NSPredicate(format: "identifier BEGINSWITH 'Row 1'")).firstMatch.hasFocus)

    // Move back up
    remote.press(.up)
    XCTAssertTrue(app.buttons.matching(NSPredicate(format: "identifier BEGINSWITH 'Row 0'")).firstMatch.hasFocus)
}
```

### Important: Simulator vs Hardware

Focus behavior differs between Simulator and Apple TV hardware:
- Simulator allows arrow key "press and hold" — hardware uses swipe gestures
- Timing of focus animations differs
- Some focus edge cases only reproduce on hardware
- Always verify critical focus flows on a physical device
