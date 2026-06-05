<div align="center">

# 📱 mobile-design

### A Claude Code skill that builds **premium, native-feeling, production-grade mobile UI** — across Jetpack Compose, SwiftUI, Flutter, and React Native / Expo.

> Not just *component-aware* — it's **flow-aware, motion-aware, and self-critical**: it builds whole journeys, ships real motion + haptics, fixes every error/warning, then critiques and scores its own work before shipping.

Grounded in **Google Material Design 3**, **Apple Human Interface Guidelines**, and real-world app layout patterns. It detects your project's stack, locks into your styling system, and ships screens that look **1:1 with real apps** — not generic AI mush.

</div>

---

## ✨ What it is

`mobile-design` is an [Agent Skill](https://docs.claude.com/en/docs/claude-code/skills) for **Claude Code**. When you ask Claude to build or style a mobile screen, this skill loads a deep knowledge base of design tokens, screen patterns, per-framework idioms, and a strict restraint + accessibility discipline — so the output looks intentional and shipped, like the screens you'd find on Mobbin from Linear, Revolut, Telegram, or Airbnb.

It is **project-aware**: before writing a single line, it inspects your repo, figures out the language/framework and the UI library you already use, and conforms to it — instead of dumping a parallel design system on top of your code.

---

## 🎯 Why it exists

Most AI-generated mobile UI fails the same way:

- ❌ Flat, evenly-spaced, default-colored, no hierarchy
- ❌ **Too much** — text everywhere, an icon on every row, badges and colors competing, layout breaking
- ❌ Ignores the platform (Material look forced on iOS, no native conventions)
- ❌ Hardcoded hex and magic numbers, no dark mode, broken on long text
- ❌ Tiny touch targets, missing accessibility, no empty/loading/error states

This skill encodes the opposite of every one of those into a repeatable workflow.

---

## 🧩 Supported stacks

| Platform | Framework | Design system |
| --- | --- | --- |
| 🤖 Android | **Jetpack Compose** (Kotlin) | Material Design 3 |
| 🍏 iOS | **SwiftUI** (Swift) | Apple Human Interface Guidelines |
| 💙 Cross-platform | **Flutter** (Dart) | Material 3 + Cupertino |
| ⚛️ Cross-platform | **React Native / Expo** (TS/JS) | Adapts M3/HIG per-OS |

### React Native UI libraries — auto-detected & locked in

NativeWind · React Native Paper · Tamagui · RN Elements · UI Kitten · gluestack-ui · Restyle · Dripsy · Unistyles · vanilla StyleSheet — plus companions like `@gorhom/bottom-sheet`, FlashList, Reanimated/Moti, Expo Router.

> **One styling system per app, used to the end.** The skill never mixes a raw `StyleSheet` into a Tamagui app or a Paper button into a NativeWind screen.

---

## 🚀 Install

A skill is a folder under `~/.claude/skills/`. Drop this repo there and Claude Code picks it up automatically.

```bash
# Clone straight into your skills directory
git clone https://github.com/ulugbekeshnazarov409/mobile-design.git \
  ~/.claude/skills/mobile-design
```

**Windows (PowerShell):**
```powershell
git clone https://github.com/ulugbekeshnazarov409/mobile-design.git `
  "$env:USERPROFILE\.claude\skills\mobile-design"
```

That's it. Open Claude Code — the skill is active. No build step, no config.

> 💡 Project-scoped install: put it in `<your-project>/.claude/skills/mobile-design` instead, to ship the skill with a specific repo.

---

## 🕹️ How to use it

Just talk to Claude Code in natural language. The skill **activates on its own** when your request is about mobile UI (keywords like *mobile ui, jetpack compose, swiftui, flutter, react native, expo, material 3, bottom sheet, onboarding, chat screen, settings screen…*), or invoke it explicitly with `/mobile-design`.

It runs in **two modes**:

### 🔨 Build mode (default)
Create or style a screen / component. The skill follows a hard gate before any code:

```
DETECT  →  DECIDE  →  PLAN  →  BUILD  →  RESTRAINT + A11Y PASS
```

1. **DETECT** — reads your repo to identify language + framework (`*.kt`+Compose → Kotlin/Compose; `import SwiftUI` → SwiftUI; `pubspec.yaml` → Flutter; `react-native`/`expo` → RN).
2. **DECIDE** — picks the UI + styling approach your project dictates (e.g. NativeWind → className-only; Paper → Paper theme).
3. **PLAN** — states a short plan (stack, styling, tokens, screen pattern, components, deps, states) so you see the path before code appears.
4. **BUILD** — generates idiomatic, token-driven, reusable code.
5. **RESTRAINT + A11Y PASS** — cuts anything that didn't earn its place, verifies 1:1 fidelity, and confirms accessibility (labels, roles, states, ≥48dp/44pt targets, contrast, dynamic type).

### 🔍 Review / lint mode
Audit existing UI code for design problems and optionally refactor — a mobile equivalent of design-guideline linting. Findings come back as `file:line — severity — problem — fix`.

Trigger it with phrases like *"review my UI", "is this design correct", "fix the design issues", "make this match guidelines"*.

---

## 💬 Example prompts

Copy, adapt, and go. The more context (framework, reference, constraints) you give, the closer to 1:1 the result.

**Build a screen**
```
Build a chat screen like Telegram in SwiftUI — bubbles, date separators,
growing input bar with a send/mic button.
```
```
Create an onboarding flow (3 pages, dots, Get Started CTA) in Jetpack Compose,
Material 3, with light + dark.
```
```
Design a settings screen in Flutter — grouped sections, a dark-mode toggle,
and a destructive Log out row.
```
```
Build a profile screen in React Native. We use NativeWind — className only.
```

**Match a real design / screenshot**
```
Here's a screenshot of a paywall. Reproduce it 1:1 in our Expo app
(we use Tamagui). Match spacing, type, and the selected-plan state.
```

**Work inside an existing project**
```
Add a bottom-sheet filter to the feed screen. Use whatever sheet library
the project already has, and match our existing components and theme.
```

**Review / refactor**
```
Review this Compose screen for design issues — touch targets, contrast,
hardcoded colors, missing states — and fix them.
```

**Design system / tokens**
```
Set up a design-token system (primitive → semantic → component) for our
Flutter app and refactor the home screen to use it.
```

> 🧠 **Tip:** name your **framework** and (for RN) your **styling library** when you can. If you don't, the skill will detect them from the repo — or ask once if it's genuinely ambiguous.

---

## 📂 What's inside

A 10-gate pipeline backed by **30 reference files**, loaded on demand:

```
mobile-design/
├── SKILL.md                          # Entry: modes, DETECT→DECIDE→PLAN→…→CRITIQUE pipeline, quality bar
└── references/
    # — Project & foundations —
    ├── project-awareness.md          # Detect repo stack/styling/tokens/components — conform to them
    ├── design-foundations.md         # Tokens: spacing, type, color roles (M3 + HIG), shape, elevation, motion
    ├── design-to-code.md             # Token architecture, pixel-perfect handoff, mobile UX laws (thumb zone, targets)
    # — Visual language —
    ├── color-systems.md              # Semantic accent ladder, palette from seed, tinted neutrals, true dark mode
    ├── typography-systems.md         # Type voice/DNA, pairing, tracking/line-height, tabular figures, custom fonts
    ├── visual-hierarchy.md           # P/S/T ranking, 3-second rule, text opacity ramp (the #1 AI fix)
    ├── screen-density.md             # Density spectrum + spacing as hierarchical rhythm
    ├── psychology.md                 # F/Z scan, attention anchors, Gestalt, cognitive load, perceived speed
    ├── case-studies.md               # Why Linear/Revolut/Telegram/Airbnb… feel premium — DNA + pitfalls
    # — Build —
    ├── screen-patterns.md            # Mobbin-style skeletons (onboarding, feed, chat, profile, settings, paywall…)
    ├── components.md                 # Cross-framework component recipes
    ├── interaction-patterns.md       # Every screen as a state machine (loading/empty/error/offline/success…)
    ├── flow-architecture.md          # Build the whole journey: sequence, branches, carried state, navigation
    ├── platform-personality.md       # Same app, different soul — iOS calm vs Android expressive
    ├── product-maturity.md           # Build for the stage (MVP → Growth → Scale → Enterprise)
    ├── real-world-constraints.md     # Offline, RTL, large text, tablets/foldables, long content, permissions
    # — Polish & motion —
    ├── premium-polish.md             # Anti-slop tells, optical adjustments, depth, signature moves, performance
    ├── motion-recipes.md             # Timing/spring tiers, press, choreography, enter↔exit, hero/numeric/skeleton
    ├── gestures-and-haptics.md       # Haptic-by-action map + swipe/drag/pull-refresh/long-press/reorder
    ├── ui-restraint-and-accessibility.md  # Anti-overcrowd + 1:1 fidelity + a11y per framework
    ├── anti-patterns.md              # The AI-UI blocklist (what NOT to do)
    # — Verify —
    ├── error-handling-and-diagnostics.md  # Zero-tolerance: per-framework error/warning/log → fix tables
    ├── design-critique-engine.md     # Self-review: 3-persona (Designer/Engineer/PM) adversarial review + REVISE loop
    ├── premium-scorecard.md          # 0–10 weighted rubric with evidence; a11y + diagnostics must = 10
    ├── design-review.md              # Review/lint mode: audit checklist + report format
    └── frameworks/
        ├── jetpack-compose.md        # Compose + M3 idioms, theming, navigation, animation
        ├── swiftui.md                # SwiftUI + HIG idioms
        ├── flutter.md                # Flutter Material 3 + Cupertino
        ├── react-native-expo.md      # RN/Expo internals, styling detection, package policy
        └── react-native-ui-libraries.md  # Full RN UI-library catalog + per-library cheat-sheets
```

Progressive disclosure: `SKILL.md` stays in context; reference files load only when a task needs them.

---

## 🧭 Design principles it enforces

- **Hierarchy over uniformity** — one primary action, clear primary/secondary/tertiary.
- **Spacing is a system** — 4dp/4pt grid; tight groups, wide gaps; whitespace is a tool.
- **Type scale, not random sizes** — 2–3 sizes, differentiated by weight + color.
- **Color with restraint** — one deliberate accent; neutral surfaces; semantic roles; always light + dark.
- **Respect the platform** — iOS feels like iOS, Android like Android; cross-platform adapts.
- **Touch & ergonomics** — ≥48dp/44pt targets; primary actions in the thumb zone; safe areas.
- **Motion with purpose** — platform easing, 150–300ms, animate state not decoration.
- **Real states** — empty, loading (skeletons), and error, not just the happy path.
- **Restraint above all** — when a screen feels busy, the fix is *remove*, not rearrange.

---

## ❓ FAQ

**Does it work without naming a framework?**
Yes — it detects from your repo. It only asks when truly ambiguous (e.g. greenfield with no files).

**Will it change my styling system?**
No. It detects and conforms. It only suggests adding a library when there's a real gap, installs it the ecosystem way (`npx expo install`, `flutter pub add`…), and notes any wiring.

**Does it handle accessibility?**
Yes — labels, roles, states, hit-area minimums, contrast, and dynamic type are part of the mandatory final pass, with per-framework code patterns.

**Can it review existing code, not just generate?**
Yes — that's Review / lint mode.

---

## 🤝 Contributing

Issues and PRs welcome — new screen patterns, component recipes, framework idioms, or UI-library cheat-sheets. Keep additions token-driven, platform-correct, and restraint-first.

## 📄 License

MIT.

<div align="center">

**Built for [Claude Code](https://claude.com/claude-code).** Design like you mean it.

</div>
