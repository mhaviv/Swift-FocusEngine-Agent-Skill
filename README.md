<p align="center">
  <img src="assets/logo.svg" height="120" alt="Apple Focus Pro" />
</p>

<h1 align="center">Apple Focus Pro</h1>

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
</p>

---

Apple Focus Pro is a free, open-source agent skill that helps AI coding assistants write correct focus management code for **tvOS**, **iOS/iPadOS**, **watchOS**, and **visionOS**. It covers SwiftUI, UIKit, and RealityKit — targeting the mistakes LLMs actually make with Apple's focus engine.

Built from real-world experience shipping tvOS apps at Fox News and Fox Weather, Apple developer documentation, WWDC sessions (2017-2025), and community best practices from Airbnb, Showmax, and others.

Works with [Claude Code](https://claude.ai/code), [Codex](https://openai.com/codex), [Cursor](https://cursor.sh), [Gemini CLI](https://github.com/google-gemini/gemini-cli), and any tool supporting the [Agent Skills](https://agentskills.io) format.

## Installing

### Claude Code

```bash
# Project-level (recommended)
npx skills add https://github.com/mhaviv/Apple-Focus-Agent-Skill --skill apple-focus-pro

# Global (all projects)
npx skills add https://github.com/mhaviv/Apple-Focus-Agent-Skill --skill apple-focus-pro -g

# Non-interactive
npx skills add https://github.com/mhaviv/Apple-Focus-Agent-Skill --skill apple-focus-pro -g -y
```

### Codex

```bash
npx skills add https://github.com/mhaviv/Apple-Focus-Agent-Skill --skill apple-focus-pro --agent codex
```

### Cursor

```bash
npx skills add https://github.com/mhaviv/Apple-Focus-Agent-Skill --skill apple-focus-pro --agent cursor
```

Or manually copy the skill's `SKILL.md` and `references/` directory into `.cursor/rules/` in your project.

### Gemini CLI

```bash
npx skills add https://github.com/mhaviv/Apple-Focus-Agent-Skill --skill apple-focus-pro --agent gemini
```

### Other Agents

Any agent that supports the [Agent Skills](https://agentskills.io) format can use this skill. Copy `SKILL.md` and the `references/` directory into your agent's skill/rules directory.

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
/apple-focus-pro Review this view for tvOS focus issues
```
Or naturally: *"Check my SwiftUI code for focus management problems"*

### Codex
```
$apple-focus-pro Check my SwiftUI code for focus anti-patterns
```

### Cursor
Reference the skill in chat or use natural language:
> Use the Apple Focus Pro skill to audit my project for focus management problems

### Gemini CLI
```
Use the apple-focus-pro skill to review my focus handling code
```

### Any Agent
> Use the Apple Focus Pro skill to audit my project for focus management problems

## What It Covers

### 2,100+ lines of focus expertise across 10 reference files

| Reference | Platform | Coverage |
|-----------|----------|----------|
| **anti-patterns.md** | All | 14 critical mistakes that break focus navigation |
| **swiftui-focus.md** | tvOS | @FocusState, focusSection, prefersDefaultFocus, AutoFocusManager pattern |
| **uikit-focus.md** | tvOS | UIFocusEnvironment, UIFocusGuide, shouldUpdateFocus, didUpdateFocus |
| **ios-focus.md** | iOS/iPadOS | Focus groups, focusGroupIdentifier, UIFocusHaloEffect, keyboard navigation |
| **watchos-focus.md** | watchOS | Digital Crown routing, sequential focus, digitalCrownRotation |
| **visionos-focus.md** | visionOS | Gaze vs hover vs focus, HoverEffect, HoverEffectGroup, RealityKit |
| **focus-styling.md** | All | ButtonStyle + isFocused, FocusBorder, CABasicAnimation, CardButtonStyle |
| **focus-restoration.md** | All | Data reload handling, safe reload pattern, row offset tracking |
| **layout-patterns.md** | tvOS | Table-of-collections, sidebar+content, tab bar, hero+catalog |
| **debugging.md** | All | UIFocusDebugger, _whyIsThisViewNotFocusable, launch arguments |

### Key Anti-Patterns It Catches

- `.disabled()` on tvOS removes views from the focus chain (use `.allowsHitTesting(false)`)
- Missing `.focusSection()` on horizontal ScrollViews causes cross-row focus jumping
- Adding `.focusable()` to Buttons creates double-focus artifacts
- `onHover(perform:)` does NOT fire from eye gaze on visionOS
- `.focusable()` MUST come before `.digitalCrownRotation()` on watchOS
- `focusGroupIdentifier` is iOS-only (not available on tvOS)
- `setNeedsFocusUpdate()` silently fails when called from wrong environment
- And 7 more...

### Sources

Built from:
- Apple Developer Documentation (UIFocusEnvironment, UIFocusGuide, FocusState, focusSection, HoverEffect)
- WWDC17: Focus Interaction in tvOS 11
- WWDC21: Direct and reflect focus in SwiftUI + Focus on iPad keyboard navigation
- WWDC23: The SwiftUI cookbook for focus
- WWDC24: Create custom hover effects in visionOS
- WWDC25: Design hover interactions for visionOS
- Production tvOS apps (Fox News, Fox Weather)
- Community guides (Airbnb, Showmax, Fatbobman, Big Nerd Ranch)

## Contributing

Contributions are welcome! Focus on:

- **Edge cases** — non-obvious focus behaviors that catch developers off guard
- **New platform APIs** — iOS 19, tvOS 19, visionOS 3, watchOS 12 additions
- **Real-world patterns** — battle-tested solutions from production apps
- **Anti-patterns** — mistakes LLMs commonly generate

Keep reference files focused and under 300 lines each. All contributions must be MIT licensed.

Please read the [Code of Conduct](CODE_OF_CONDUCT.md) before contributing.

## License

Apple Focus Pro was created by [Michael Haviv](https://github.com/mhaviv) and is licensed under the [MIT License](LICENSE).
