# Changelog

All notable changes to Swift FocusEngine Pro are documented here.

## [1.3.1] - 2026-04-10

### Added
- **3 new tvOS anti-patterns from production** (#15–17) — `LazyVStack` focus escape, vertical `.focusSection()`, allocation in focus callbacks
- **VStack + inner LazyHStack pattern** — lightweight outer container stays in hierarchy, heavy content stays lazy inside each row
- **Tab bar focus escape detection** — `didUpdateFocus` pattern for detecting when focus escapes content to tab bar
- **VoiceOver card composition pattern** — `.accessibilityElement(children: .ignore)` with composed labels for multi-element focusable cards
- Total anti-patterns: 24 (up from 21)

## [1.3.0] - 2026-04-10

### Added
- **macOS focus coverage** — new `macos-focus.md` reference file (650+ lines)
  - AppKit: NSResponder chain, `acceptsFirstResponder`, `canBecomeKeyView`, key view loop
  - Key window vs main window, NSPanel focus behavior, `becomesKeyOnlyIfNeeded`
  - Focus ring: `NSFocusRingType`, `drawFocusRingMask()`, custom shapes
  - SwiftUI on macOS: `@FocusState`, `.focusable()`, `.focusSection()`, `.onKeyPress`
  - `focusedValue` / `focusedSceneValue` for menu bar commands
  - NSToolbar, NSPopover, sheets, NSAlert focus
  - Multi-window, multi-screen, external display
  - Mac Catalyst bridging
  - Full Keyboard Access
- **7 macOS-specific anti-patterns** (#15–21) in `anti-patterns.md`
- macOS focus ring styling in `focus-styling.md`
- macOS VoiceOver, NSAccessibility, Voice Control in `accessibility-focus.md`
- macOS first responder debugging in `debugging.md`
- macOS layout patterns (sidebar, toolbar, multi-window, inspector, three-column) in `layout-patterns.md`
- macOS focus restoration (sheets, NSDocument revert) in `focus-restoration.md`

## [1.2.0] - 2026-04-08

### Added
- **Expanded iOS focus coverage** — game controller focus, Stage Manager multi-window, `.onKeyPress`, pointer hover effects, `focusedValue` / `focusedSceneValue` deep dive
- **Expanded watchOS focus coverage** — `.digitalCrownAccessory()`, nested scrolling conflicts, managing multiple focusable controls
- FAQ section with 20 collapsible questions (tvOS, iOS, iPadOS, watchOS, visionOS, macOS)
- `llms.txt` for AI model discovery
- SKILL.md keyword metadata for registry indexing
- Community health files: CONTRIBUTING.md, issue templates, PR template

## [1.1.0] - 2026-04-07

### Added
- **watchOS focus reference** — Digital Crown routing, sequential focus, `.focusable()` ordering, Crown conflicts
- **RealityKit focus reference** — `HoverEffectComponent`, collision shapes, shader effects, mixed SwiftUI + RealityKit hierarchies
- **Accessibility focus reference** — `@AccessibilityFocusState`, VoiceOver coordination, Full Keyboard Access, Switch Control, Reduce Motion

## [1.0.0] - 2026-04-06

### Added
- Initial release with 10 reference files covering tvOS, iOS/iPadOS, and visionOS
- SwiftUI and UIKit focus management
- 14 critical anti-patterns
- Focus styling, restoration, layout patterns, async coordination, debugging
- Agent Skills format (SKILL.md) for Claude Code, Codex, Cursor, Copilot, Gemini CLI
