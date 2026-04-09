<p align="center">
  <img src="assets/logo.svg" height="120" alt="Swift FocusEngine Pro" />
</p>

<h1 align="center">Swift FocusEngine Pro</h1>

<p align="center">
  <strong>Agent skill for focus management across all Apple platforms</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/tvOS-15+-000000?logo=apple" />
  <img src="https://img.shields.io/badge/iOS-15+-000000?logo=apple" />
  <img src="https://img.shields.io/badge/watchOS-8+-000000?logo=apple" />
  <img src="https://img.shields.io/badge/visionOS-1+-000000?logo=apple" />
  <img src="https://img.shields.io/badge/Swift-5.9+-F05138?logo=swift&logoColor=white" />
  <img src="https://img.shields.io/badge/License-MIT-blue" />
</p>

<p align="center">
  <a href="https://x.com/michael_haviv">
    <img src="https://img.shields.io/badge/Contact-@michael__haviv-1DA1F2?logo=x&logoColor=white" />
  </a>
  <a href="https://www.linkedin.com/in/michaelhaviv/">
    <img src="https://img.shields.io/badge/LinkedIn-Michael_Haviv-0A66C2?logo=linkedin&logoColor=white" />
  </a>
</p>

---

Swift FocusEngine Pro is a free, open-source agent skill that helps AI coding assistants write correct focus management code for **tvOS**, **iOS/iPadOS**, **watchOS**, and **visionOS**. It covers SwiftUI, UIKit, and RealityKit — targeting the mistakes LLMs actually make with Apple's focus engine.

Built from real-world experience shipping production tvOS apps, Apple developer documentation, WWDC sessions (2017-2025), and community best practices from Airbnb, Showmax, and others.

Works with [Claude Code](https://claude.ai/code), [Codex](https://openai.com/codex), [Cursor](https://cursor.sh), [GitHub Copilot](https://github.com/features/copilot), [Gemini CLI](https://github.com/google-gemini/gemini-cli), and any tool supporting the [Agent Skills](https://agentskills.io) format.

## Table of Contents

- [Who This Is For](#who-this-is-for)
- [Why Use an Agent Skill for Focus?](#why-use-an-agent-skill-for-focus)
- [Installing](#installing)
- [Using](#using)
- [What It Covers](#what-it-covers)
- [Anti-Patterns It Catches](#anti-patterns-it-catches)
- [Sources](#sources)
- [Complementary Skills](#complementary-skills)
- [Contributing](#contributing)
- [License](#license)

## Who This Is For

- **tvOS developers** — building apps where every interaction depends on the focus engine working correctly
- **iOS/iPadOS developers** — adding keyboard, game controller, or external display support with focus groups
- **visionOS developers** — navigating the differences between gaze, hover, and focus in spatial computing

## Why Use an Agent Skill for Focus?

Focus management on Apple platforms is one of the hardest things to get right — and one of the hardest things to debug when it breaks.

The focus engine is geometric, not hierarchical. It doesn't follow your view tree. When a user swipes right and focus jumps two rows away instead of to the next item, there's no error, no crash, no log — it just looks broken. The item wasn't perfectly vertically aligned, so the engine picked a different candidate. You'll spend hours in `UIFocusDebugger` before you figure out why.

Apple's documentation covers the APIs but not the real-world edge cases: what happens when you reload data and focus resets to the top, why `.disabled()` silently removes views from the focus chain on tvOS, why `.focusSection()` is the difference between a usable scroll view and chaos, or why `onHover` doesn't fire from eye gaze on visionOS.

LLMs generate focus code that compiles and looks reasonable — but breaks in ways you only discover on a real device with a Siri Remote in your hand. This skill is built from my experience getting focus to actually work in a complex, production tvOS app. Every anti-pattern in here is something I hit, debugged, and fixed.

## Installing

### Claude Code

```bash
# Global (all projects)
npx skills add https://github.com/mhaviv/Apple-Focus-Agent-Skill --skill swift-focusengine-pro -g -y

# Project-level only
npx skills add https://github.com/mhaviv/Apple-Focus-Agent-Skill --skill swift-focusengine-pro -y
```

### Codex

```bash
npx skills add https://github.com/mhaviv/Apple-Focus-Agent-Skill --skill swift-focusengine-pro --agent codex
```

### Cursor

```bash
npx skills add https://github.com/mhaviv/Apple-Focus-Agent-Skill --skill swift-focusengine-pro --agent cursor
```

### GitHub Copilot

```bash
npx skills add https://github.com/mhaviv/Apple-Focus-Agent-Skill --skill swift-focusengine-pro --agent github-copilot
```

### Gemini CLI

```bash
npx skills add https://github.com/mhaviv/Apple-Focus-Agent-Skill --skill swift-focusengine-pro --agent gemini
```

### Other Agents

Any agent that supports the [Agent Skills](https://agentskills.io) format can use this skill. See [agentskills.io](https://agentskills.io) for instructions on adding skills to your agent.

<details>
<summary>Don't have Node installed?</summary>

```bash
brew install node
```

Or download from [nodejs.org](https://nodejs.org).
</details>

## Using

### Claude Code
```
/swift-focusengine-pro Review this view for tvOS focus issues
```

### Codex
```
$swift-focusengine-pro Check my SwiftUI code for focus anti-patterns
```

### Cursor
```
/swift-focusengine-pro Review this view for tvOS focus issues
```

### GitHub Copilot
```
/swift-focusengine-pro Review this view for tvOS focus issues
```

### Gemini CLI
```
Use the swift-focusengine-pro skill to review my focus handling code
```

### Any Agent
> Use the Swift FocusEngine Pro skill to audit my project for focus management problems

### Example Prompts

- *"Why isn't the first item focused when my view appears?"*
- *"Focus is jumping to a completely different row when I swipe right — the items aren't perfectly aligned vertically"*
- *"How do I keep focus position after my data reloads?"*
- *"I added .disabled() to a button but now focus skips over the entire section"*
- *"What's the difference between gaze and focus on visionOS?"*
- *"My Digital Crown rotation stopped working after I reordered my view modifiers"*

## What It Covers

### 2,100+ lines of focus expertise across 10 reference files

| Reference | Platform | Coverage |
|-----------|----------|----------|
| **anti-patterns.md** | All | 14 critical mistakes that break focus navigation |
| **swiftui-focus.md** | tvOS | @FocusState, focusSection, prefersDefaultFocus, AutoFocusManager pattern |
| **uikit-focus.md** | tvOS | UIFocusEnvironment, UIFocusGuide, shouldUpdateFocus, didUpdateFocus |
| **ios-focus.md** | iOS/iPadOS | SwiftUI + UIKit: focus groups, focusGroupIdentifier, UIFocusHaloEffect, keyboard navigation |
| **watchos-focus.md** | watchOS | SwiftUI: Digital Crown routing, sequential focus, digitalCrownRotation |
| **visionos-focus.md** | visionOS | SwiftUI + UIKit + RealityKit: gaze vs hover vs focus, HoverEffect, HoverEffectComponent |
| **focus-styling.md** | All | ButtonStyle + isFocused, FocusBorder, CABasicAnimation, CardButtonStyle |
| **focus-restoration.md** | All | Data reload handling, safe reload pattern, row offset tracking |
| **layout-patterns.md** | tvOS | Table-of-collections, sidebar+content, tab bar, hero+catalog |
| **debugging.md** | All | UIFocusDebugger, _whyIsThisViewNotFocusable, launch arguments |

## Anti-Patterns It Catches

### Blocking (must fix before ship)

1. **`.disabled()` removes views from the focus chain on tvOS** — use `.allowsHitTesting(false)` instead
2. **Missing `.focusSection()` on horizontal ScrollViews** — causes cross-row focus jumping in vertical layouts
3. **Adding `.focusable()` to Buttons or NavigationLinks** — creates double-focus artifacts
4. **Mixing SwiftUI and UIKit focus in the same hierarchy** — focus environment conflicts
5. **Calling `reloadData()` during animations** — focus resets to the top of the screen
6. **Using `frame.width` in focus transform calculations** — dimensions change when focused
7. **`setNeedsFocusUpdate()` called from wrong environment** — silently fails with no error
8. **Setting `isUserInteractionEnabled = false` on headers/labels** — removes them and their children from focus chain
9. **`remembersLastFocusedIndexPath` + offscreen `reloadData()`** — remembered index may no longer exist
10. **Using `UIView.animate` for CALayer properties** — animations won't work, use `CABasicAnimation`

### Warning (should fix)

11. **Non-optional `@FocusState` with `focused(_:equals:)`** — can't represent "nothing focused" state
12. **Missing `prepareForReuse()` cleanup for focus state** — stale focus styling on reused cells
13. **`prefersDefaultFocus` inside ScrollView** — may not work as expected, use `defaultFocus` instead
14. **LazyVStack/LazyVGrid performance on Apple TV HD** — A8 chip can't handle lazy layout recalculation during fast scrolling

## Sources

Built from:
- Apple Developer Documentation (UIFocusEnvironment, UIFocusGuide, FocusState, focusSection, HoverEffect)
- WWDC17: Focus Interaction in tvOS 11
- WWDC21: Direct and reflect focus in SwiftUI + Focus on iPad keyboard navigation
- WWDC23: The SwiftUI cookbook for focus
- WWDC24: Create custom hover effects in visionOS
- WWDC25: Design hover interactions for visionOS
- Production tvOS apps with complex focus requirements
- Community guides (Airbnb, Showmax, Fatbobman, Big Nerd Ranch)

## Complementary Skills

Swift FocusEngine Pro pairs well with these skills:

- [SwiftUI Pro](https://github.com/twostraws/SwiftUI-Agent-Skill) by Paul Hudson — SwiftUI best practices and patterns
- [Swift Concurrency Pro](https://github.com/twostraws/Swift-Concurrency-Agent-Skill) by Paul Hudson — async/await, actors, Sendable
- [Swift Concurrency](https://github.com/AvdLee/Swift-Concurrency-Agent-Skill) by Antoine van der Lee — Swift 6 migration, data race prevention
- [Xcode Build Optimization](https://github.com/AvdLee/Xcode-Build-Optimization-Agent-Skill) by Antoine van der Lee — build benchmarking and optimization

See the [Swift Agent Skills](https://github.com/twostraws/Swift-Agent-Skills) directory for more.

## Contributing

Contributions are welcome! Focus on:

- **Edge cases** — non-obvious focus behaviors that catch developers off guard
- **New platform APIs** — iOS 19, tvOS 19, visionOS 3, watchOS 12 additions
- **Real-world patterns** — battle-tested solutions from production apps
- **Anti-patterns** — mistakes LLMs commonly generate

Keep reference files focused and under 300 lines each. Don't repeat things LLMs already know — focus on what they get wrong. All contributions must be MIT licensed.

Please read the [Code of Conduct](CODE_OF_CONDUCT.md) before contributing.

## License

Swift FocusEngine Pro was created by [Michael Haviv](https://github.com/mhaviv) and is licensed under the [MIT License](LICENSE).
