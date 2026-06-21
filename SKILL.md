---
name: mobile-design
description: >-
  Design-grade mobile UI skill. Builds polished, production-quality, native-feeling
  mobile screens and components across Jetpack Compose (Kotlin), SwiftUI (Swift),
  Flutter (Dart), and React Native / Expo. Grounded in Google Material Design 3,
  Apple Human Interface Guidelines, and real-world app layout patterns (Mobbin-style
  flows). Use for ANY task that designs, styles, or builds a mobile screen, component,
  or design system — "build a chat UI like Telegram", "make this Compose screen look
  premium", "design an onboarding flow", "SwiftUI settings screen", "Flutter card",
  "RN bottom sheet", "mobile design tokens", "make it look native not AI". Triggers —
  mobile ui, mobile design, jetpack compose, kotlin ui, swiftui, swift ui, flutter
  ui, react native ui, expo ui, material 3, material you, apple hig, design tokens,
  bottom sheet, bottom nav, app bar, fab, onboarding, chat screen, feed, profile
  screen, settings screen, native mobile design, liquid glass, glassmorphism, frosted
  glass, blur material, map screen, map overlay, bottom sheet over map, marker, currency
  format, so'm / UZS, bank card UI, Uzcard, Humo, phone mask, region picker, expo router,
  navigation, tabs, deep linking, auth flow, loading state, skeleton, empty state, form
  validation, react-hook-form, tanstack query, optimistic update, push notification,
  permission flow, image picker, design tokens, accessibility, voiceover, talkback,
  dynamic type, performance, 60fps, flashlist, example screen, date picker, calendar,
  otp input, segmented control, toast, snackbar, filter chips, slider, search bar, chart,
  graph, sparkline, dashboard, skia, gradient, shader, custom canvas, i18n, localization,
  uz/ru/en, pluralization, rtl, testing, maestro, detox, e2e, web, responsive, universal,
  expo web, onboarding, empty state, illustration, lottie.
---

# Mobile Design

Build mobile UI that looks **intentional, native, and shipped** — like screens you'd find on Mobbin from Linear, Revolut, Telegram, Airbnb — not generic AI/Bootstrap mush.

This skill spans four target stacks. Pick the user's framework, then apply the same design judgment to all of them:

| Platform | Framework | Design system |
| --- | --- | --- |
| Android | Jetpack Compose (Kotlin) | Material Design 3 |
| iOS | SwiftUI (Swift) | Apple Human Interface Guidelines |
| Cross-platform | Flutter (Dart) | Material 3 + Cupertino |
| Cross-platform | React Native / Expo | Adapt M3/HIG per-OS |

## Two modes

- **Build mode** — create/style a screen or component (default).
- **Review / lint mode** — audit existing UI code for design problems and optionally refactor. Trigger on "review/lint my UI", "is this design correct", "fix the design issues", "make this match guidelines". Use `references/design-review.md`.

## Decision flow — INTAKE → DETECT → DECIDE → PLAN (do this before writing any UI)

Never start coding UI blind. First, if the brief is vague, *ask a quick multiple-choice intake*; then work out *what language/framework the code is in*, then *which UI toolkit + styling system that implies*, then *state the plan*. This is a hard gate, not optional.

### Step 0 — INTAKE (only when the brief is vague)

If the request leaves the domain, purpose, visual style, audience, or scope unstated/ambiguous ("build me a fitness app", "make a delivery screen"), run a **fast, batched multiple-choice intake** before anything else — use `references/requirements-intake.md`. Each question = 2-4 concrete options + a recommended default, asked together (prefer the host's structured question UI), so the user confirms in one step instead of you guessing. **Skip intake** when the brief is already specific or it's a tiny one-component ask — never block a clear request with questions. Feed the answers straight into the PLAN block below.

### Step A — DETECT the language & framework (from the repo, not assumptions)

Look at the actual project files and infer the stack. Signals → conclusion:

| Files / signals found | Language | Framework | UI toolkit |
| --- | --- | --- | --- |
| `*.kt`, `build.gradle.kts`, `androidx.compose.*` | Kotlin | Jetpack Compose | Material 3 |
| `*.kt`/`*.java`, `res/layout/*.xml` (no Compose) | Kotlin/Java | Android Views (XML) | Material Components |
| `*.swift`, `import SwiftUI` | Swift | SwiftUI | Apple HIG |
| `*.swift`, `*.storyboard`/`*.xib` | Swift | UIKit | Apple HIG |
| `*.dart`, `pubspec.yaml` | Dart | Flutter | Material 3 / Cupertino |
| `package.json` + `react-native`/`expo`, `*.tsx` | TS/JS | React Native / Expo | (see Step B) |

If no repo (greenfield) or genuinely ambiguous after looking → **ask one question**: which platform/framework. Don't guess between Compose and SwiftUI.

### Step B — DECIDE the UI & styling approach the detected stack dictates

The framework alone doesn't fully decide *how* to style — especially React Native. Inspect deeper and pick the approach the project already uses:

- **Compose** → M3 components + the repo's `Theme.kt`/`Color.kt`/`Type.kt` tokens. (M2 vs M3: match existing imports.)
- **SwiftUI** → system components + semantic colors / any `Theme`/asset color set in the repo.
- **Flutter** → `ThemeData` (`useMaterial3`?) tokens; Material vs Cupertino per platform intent.
- **React Native / Expo** → **detect the UI/styling library and lock into it for the whole task** (this is the big one — see `references/frameworks/react-native-ui-libraries.md` for the full catalog + per-library cheat-sheets):
  - `nativewind` + `tailwind.config.js` → **className-only Tailwind**, no `StyleSheet`.
  - `react-native-paper` → Paper components + Paper theme. `tamagui` → `styled()` + tokens. `@rneui/themed` → RN Elements. `@ui-kitten` → Eva. `@gluestack-ui` → gluestack. `@shopify/restyle` → typed theme box/text. `dripsy` → `sx`. `react-native-unistyles` → themed StyleSheet.
  - none → `StyleSheet.create` + a token module (or suggest a fitting lib if warranted).
  - **One system per app, used to the end** — never mix two (no raw StyleSheet in a Tamagui app, no Paper button in a NativeWind screen).
  - Also detect companions (`@gorhom/bottom-sheet`, FlashList, reanimated/moti), nav (Expo Router vs React Navigation), existing `components/ui` folder, and the design-token source — reuse them.

Full detection tables: `references/project-awareness.md`, `references/frameworks/react-native-expo.md`, `references/frameworks/react-native-ui-libraries.md`.

### Step C — PLAN out loud before coding (one short block)

Once detected and decided, state a brief plan so the user sees the chosen path before any code. Template:

```
Stack:    <language> / <framework>   (detected from <signal>)
Styling:  <system in use>            (e.g. NativeWind className-only)
Tokens:   <reuse existing X / define from foundations>
Visual:   <accent + color ladder; type voice>   (color-systems / typography-systems)
Platform: <iOS calm / Android expressive personality, or both>   (platform-personality)
Maturity: <Stage 0 MVP / 1 / 2 / 3 — assumed if unstated>   (product-maturity)
Screen:   <pattern from screen-patterns.md, e.g. chat scaffold>
Components: <list, reused-or-new>
Nav/deps: <existing | add <pkg> via <installer> because <gap>>
States:   <empty / loading / error to include>
```

Keep it 5–7 lines, then build. For a tiny one-component ask, a one-line version is fine — but still name the stack + styling system explicitly. **The point: the user always knows which language, which UI toolkit, which styling system, and what will be built, before code appears.**

## How to use this skill (build mode)

0. **Project pre-flight (do this first on any existing repo).** Load `references/project-awareness.md`. Detect the framework, the **styling system in use** (e.g. RN with NativeWind → className-only; Tamagui; restyle; or StyleSheet), the existing tokens, and existing components — then **conform to them**. Reuse tokens/components; don't impose a different system. Add a package only when there's a real gap, the ecosystem way (`npx expo install`, `flutter pub add`, etc.).
1. **Identify the framework.** If the user named one (Compose, SwiftUI, Flutter, RN/Expo) use it. If the repo makes it obvious (build.gradle.kts → Compose; *.xcodeproj/Package.swift → SwiftUI; pubspec.yaml → Flutter; package.json + react-native → RN), match it. If genuinely ambiguous, ask once.
2. **Read the foundations + set the visual language.** Load `references/design-foundations.md` (tokens: spacing, type, color, shape, elevation, motion) and `references/design-to-code.md` (token architecture, pixel-perfect handoff, mobile UX laws). Then set the **visual language** — the layer that separates premium from "AI look": `references/color-systems.md` (one accent expanded into the semantic ladder; tinted neutrals; true dark mode — not raw hex/3-tints) and `references/typography-systems.md` (a type *voice* + weight/tracking/tabular figures). And the **platform personality** via `references/platform-personality.md` (iOS calm/content-first vs Android expressive/surface-rich — adapt the *soul*, not just syntax). Decide the **maturity stage** with `references/product-maturity.md` so complexity fits the product (don't over-build an MVP). Every screen starts from tokens mapped onto the project's real names — never hardcoded magic numbers. Greenfield or no token system yet? Pull a complete premium default from `references/design-tokens-starter.md` (light+dark accent ladder, type ramp, spacing/radius/elevation/motion — ready in all four frameworks) and adapt the seed accent.
3. **Scope: screen or flow?** If the request implies a multi-step task ("checkout", "sign up", "send money", "onboarding", "booking"), you're building a **flow**, not one screen. Load `references/flow-architecture.md` — map the screen sequence, branches, carried state, and navigation; confirm scope; then build the whole journey. For any single screen, you'll still cover all its states in step 6.
4. **Decide hierarchy & density (do this before laying out).** Load `references/visual-hierarchy.md` — rank every element Primary/Secondary/Tertiary, pick the one primary action, apply the text opacity ramp. Then `references/screen-density.md` — choose the density that fits the screen's job (dense chat vs airy paywall) and use *hierarchical* spacing, not even gaps. These two fix the #1 and #2 AI failures. Use `references/psychology.md` to place the primary element on the eye's scan path (F/Z), use the strongest attention anchor, and group by proximity/alignment. For a premium reference point, pull the closest app's DNA from `references/case-studies.md` ("make it like Linear/Revolut/Telegram…").
5. **Identify the screen pattern.** Load `references/screen-patterns.md` to map the request ("chat", "feed", "onboarding", "profile", "settings", "paywall") to a proven layout skeleton, and pattern-match against a full worked implementation in `references/example-screens.md` (sign-in/feed/settings built to the premium bar). For **onboarding/first-run/empty states**, use `references/onboarding-and-illustration.md` (value-first, skippable, empty states that explain + act). If the screen is **map-based** (ride/delivery/real-estate/nearby), load `references/map-ui-patterns.md` — overlay chrome, marker/cluster design, and the map-padding↔bottom-sheet sync that AI map UIs always get wrong.
6. **Build — every state, not just the happy path.** Load `references/components.md` for cross-framework component recipes (and `references/components-advanced.md` for composite controls — date/calendar picker, OTP, segmented control, toast/snackbar system, filter chips, slider, search bar), the matching `references/frameworks/<framework>.md` for idioms/navigation/theming, and `references/interaction-patterns.md` to enumerate and build **all** of the screen's states (initial/loading/empty/error/offline/submitting/success/disabled) and its exits — the happy path is ~30% of the work. Design for `references/real-world-constraints.md` too: offline, RTL, largest text scale, small phones + tablets, long content — so it survives the wild, not just the demo. For UZ/CIS-market apps, apply `references/locale-uz.md`: so'm/UZS money formatting, `+998` phone masks, Uzcard/Humo card UI, viloyat→tuman region pickers, and uz-Latin/uz-Cyrillic/ru language handling. **Wire UI to real async state** with `references/data-and-forms.md` — every data surface gets loading(skeleton)/empty/error(retry)/success, mutations get pending + optimistic + success-haptic, forms get validation + no double-submit (TanStack Query, react-hook-form + zod). Structure non-trivial apps with `references/state-and-architecture.md` (server vs client state, feature folders, auth/session, theme provider, secure token storage). For **charts/dashboards** (FinTech, analytics) use `references/charts-and-dataviz.md` — premium, anti-chartjunk, currency-aware. Multilingual app? Apply `references/i18n-and-localization.md` (expo-localization + i18next, pluralization, text-expansion layout) on top of `locale-uz.md` formatting. Targeting **web too**? Build universal with `references/web-and-universal.md` (responsive breakpoints, hover/focus/keyboard, web-vs-native guards). For native capabilities (push, camera, location, share) run the in-context permission flow from `references/notifications-and-native.md` — prime → request → handle denied → fallback, never request on launch.
7. **Restraint + accessibility pass (mandatory).** Run `references/ui-restraint-and-accessibility.md`: cut anything that didn't earn its place (the most common failure is *too much* — text/icons/badges/colors breaking the UI), verify 1:1 fidelity, confirm every element uses the right semantic component with required props + a11y (labels, roles, states, ≥48dp/44pt targets). For anything beyond the basics — screen-reader semantics (VoiceOver/TalkBack), focus order, live-region announcements, Dynamic Type at largest scale, WCAG contrast math — go deep with `references/accessibility-deep.md`.
8. **Premium polish pass.** Run `references/premium-polish.md` to push from "fine" to top-studio tier: kill AI-slop tells (off-black/white, text ramp, single accent, soft layered shadows), apply optical adjustments, tune typography, true dark mode, and one or two signature moves. Add **motion + tactile feel** from `references/motion-recipes.md` (timing/spring tiers, press scale, choreography, **every enter gets an exit**, hero/numeric/skeleton recipes) and `references/gestures-and-haptics.md` (swipe/drag/pull-refresh + the haptic-by-action map; gesture+motion+haptic trinity). For **one signature visual** (gradient mesh, glow, animated canvas, custom progress ring), reach for `references/skia-and-graphics.md` — sparingly, with a perf budget. If the design calls for translucent "glass" chrome (floating bar/sheet, iOS Liquid Glass), use `references/glassmorphism-and-materials.md` — disciplined, accessibility-aware, not by default. Scan `references/anti-patterns.md` and remove any hits.
8b. **Performance pass (premium feel = 60fps).** Run `references/performance-and-quality.md`: virtualize long lists, kill re-render storms, size/cache images, drive motion on the UI thread, cap blur/shadow layers, keep startup lean — jank reads as cheap, so treat dropped frames as a design defect.

9. **Diagnostics pass (zero-tolerance).** Run `references/error-handling-and-diagnostics.md`: type-check + lint/`analyze` + run, and **fix every error AND warning in place** — read the exact message, fix the root cause, re-run until compile is clean, lint/analyze is zero, and the console/logcat is clean (no red, no yellow, no stray logs). Warnings are treated as bugs. Then **verify with evidence** via `references/testing-and-verification.md`: the manual matrix (device, light/dark, small phone + tablet, largest Dynamic Type, RTL, offline, screen-reader swipe-through) plus component/E2E tests (@testing-library/react-native, Maestro) where they fit — prove states render and the primary action works, don't assume.
10. **Critique + score (final gate).** Run `references/design-critique-engine.md` with its anti-gaming sequence: **forced-failure search** (5 reasons not to ship) → **3-persona adversarial review** (Designer / Staff Engineer / Product Manager — all must be satisfied; the PM lens checks the screen actually achieves its goal) → fix → then **evidence-based** scoring. Do ≥1 full critique→revise cycle. Grade with `references/premium-scorecard.md` (evidence per dimension); any dimension below threshold → revise and re-score (accessibility + diagnostics must be 10). Report the scorecard. Only ship at "premium". Self-check against the Quality bar below.

> Progressive disclosure: keep this file in context, open reference files only when the task needs them. Don't dump all references for a one-component ask.

## Design philosophy — read before writing any UI

Most AI-generated mobile UI fails the same way: flat, evenly-spaced, default-colored, no hierarchy, no motion, ignores platform conventions. Avoid every one of these.

**1. Hierarchy over uniformity.** A screen has one primary action, a few secondary ones, and supporting content. Make that visible through size, weight, color, and spacing — not by making everything the same.

**2. Spacing is a system, not a guess.** Use a 4dp/4pt base grid (4, 8, 12, 16, 24, 32, 48). Group related elements tight, separate unrelated ones wide. White space is a design tool, not waste.

**3. Type scale, not random sizes.** Use the platform type ramp (M3 type roles / iOS text styles). Two or three sizes per screen, differentiated by weight and color, beats five arbitrary sizes.

**4. Color with restraint.** One brand/accent color used deliberately. Neutral surfaces carry most of the UI. Derive on-colors, containers, and states from the role system (M3 color roles / HIG semantic colors) — never pick raw hex per element. Always ship light **and** dark.

**5. Respect the platform.** iOS feels like iOS (SF font, large titles, swipe-back, grouped lists, bottom sheets with grabbers). Android feels like Android (Material components, FABs, ripple, top app bar). Cross-platform code should adapt, not pick one and force it everywhere.

**6. Touch targets & ergonomics.** Minimum 48×48dp (Android) / 44×44pt (iOS) hit area. Primary actions reachable by thumb (bottom of screen). Respect safe areas / notches / nav bars.

**7. Motion with purpose.** Use the platform's standard easing and durations. Animate state changes (press, expand, navigate), not decoration. Default to subtle: 150–300ms, ease-out for entrances.

**8. Real content, real states.** Design for long names, empty lists, loading, and errors — not just the happy path with perfect placeholder data. Include skeletons/empty states when building a full screen.

## Quality bar — self-check before declaring done

- [ ] Spacing comes from the grid scale; no stray magic numbers.
- [ ] Type uses the platform scale; clear primary/secondary/tertiary hierarchy.
- [ ] Color uses semantic roles; one deliberate accent; light + dark both work.
- [ ] Components match platform conventions (right component for the OS).
- [ ] Touch targets ≥ 48dp/44pt; safe-area insets respected.
- [ ] At least the obvious state changes are animated with platform easing.
- [ ] Empty/loading/error states considered for full screens.
- [ ] **Hierarchy:** at 0.5s one element wins the eye; elements ranked Primary/Secondary/Tertiary; text uses the opacity ramp. (The #1 AI failure.)
- [ ] **Density:** chosen to fit the screen's job; spacing is hierarchical (related tight, groups wide), not even gaps; item weight varies.
- [ ] **Restraint:** nothing on screen that didn't earn its place; one primary action, one accent, 2–3 type sizes; long text truncates; whitespace intentional. (Most AI UI fails by doing *too much*.)
- [ ] **No anti-patterns:** scanned `anti-patterns.md`, zero hits.
- [ ] **Zero diagnostics:** compiles with no errors; lint/`analyze` clean; app runs with a clean console (no red, no yellow, no stray logs); every warning fixed at the source, not suppressed.
- [ ] **Visual language:** one accent expanded into a semantic ladder + a deliberate type voice (tabular figures for numbers); not raw-hex / default-font AI look.
- [ ] **Platform personality:** feels native to each OS (iOS calm / Android expressive) — adapted soul, not one look in two syntaxes.
- [ ] **Maturity-appropriate:** UI complexity fits the product stage (no over-built MVP); assumed stage stated.
- [ ] **Real-world:** offline / RTL / largest text / small phone + tablet / long content considered — survives beyond the demo.
- [ ] **Flow-aware:** if multi-step, the whole journey is built (screens + nav + carried state + exits), not one screen.
- [ ] **All states built:** loading/empty/error/offline/submitting/success/disabled — not just the happy path; no dead-ends or double-submit.
- [ ] **Motion + tactile:** press feedback + haptics on commits; transitions choreographed; every enter has an exit; Reduce Motion handled.
- [ ] **Critiqued + scored:** ran the anti-gaming sequence (forced-failure search + 3-persona Designer/Engineer/PM review) → ≥1 critique→revise cycle; premium scorecard passes with **evidence per dimension** (all dims ≥ threshold; a11y + diagnostics = 10); remaining items are stated taste trade-offs.
- [ ] Every element uses the right semantic component with required props + accessibility (label/role/state); 1:1 fidelity verified at small width, largest text scale, and dark mode.
- [ ] Code is idiomatic for the framework (composables/views/widgets/components named and structured the way that ecosystem expects) and reusable, not one giant function.
- [ ] (RN) One UI/styling library, used to the end — no mixing systems.

## Reference map

- `references/requirements-intake.md` — **Intake (vague brief).** Fast batched multiple-choice questions (domain, key features, platform, visual mood, color, reference apps, maturity, audience/locale, scope) with recommended defaults; maps answers straight to the PLAN block. Skip when the brief is already specific.
- `references/design-tokens-starter.md` — **Copy-paste token system.** Premium light+dark defaults (accent ladder, tinted neutrals, type ramp, spacing/radius/elevation/motion) ready in RN/NativeWind, Compose, SwiftUI, Flutter. Adapt the seed accent; map onto existing names if present.
- `references/data-and-forms.md` — **Wire UI to async state.** TanStack Query v5 (loading/empty/error/success, refetch, infinite scroll, optimistic mutations), react-hook-form + zod, skeletons, no double-submit, phone/card/OTP fields; cross-framework notes.
- `references/state-and-architecture.md` — **Structure that scales.** Server vs client vs navigation state, feature-folder layout, Zustand + Context, auth/session + `expo-secure-store`, theme provider, env/secrets, a `components/ui` design-system layer.
- `references/notifications-and-native.md` — **Permission flows + native UI.** The in-context permission pattern (prime→request→denied→Settings→fallback), expo-notifications (push/badge/channels/tap deep-link), image-picker/camera, share/clipboard; dev-build + config-plugin callouts.
- `references/accessibility-deep.md` — **A11y deep pass.** Screen-reader semantics per framework (VoiceOver/TalkBack), focus order/grouping, live-region announcements, Dynamic Type at largest scale, WCAG contrast math, OS flags (Reduce Motion/Transparency/Bold Text), how to actually test.
- `references/performance-and-quality.md` — **60fps as polish.** List virtualization, re-render control, image/caching, UI-thread motion, startup/bundle budget, blur/shadow cost, per-framework profiling tools; jank treated as a design defect.
- `references/example-screens.md` — **Complete gold-standard screens.** Full end-to-end implementations (sign-in RN/Expo, feed Compose, settings SwiftUI/Flutter) with tokens + hierarchy + all states + motion + a11y wired together — concrete targets to pattern-match, not copy.
- `references/components-advanced.md` — **Composite controls.** Date/calendar picker, OTP input, segmented control, stepper, toast/snackbar system, filter chips, slider, switch-row, search bar, sticky action bar, avatar group — beyond the basic `components.md` catalog.
- `references/charts-and-dataviz.md` — **Premium charts.** Data-ink/anti-chartjunk, library selection (victory-native-xl, gifted-charts, Skia, Swift Charts, Vico, fl_chart), sparkline/bar/donut/candlestick, currency axis, scrub/tooltip, 60fps, accessible summaries.
- `references/skia-and-graphics.md` — **Signature visuals.** `@shopify/react-native-skia` — gradients/mesh/blur/glow/shaders/animated canvas with Reanimated; Skia-vs-SVG-vs-view decision; one signature moment, with restraint + dev-build note.
- `references/i18n-and-localization.md` — **Multilingual engineering.** expo-localization + i18next, resource files (uz/ru/en), pluralization (Russian one/few/many), Intl formatting, text-expansion layout, RTL, font script coverage — the engine behind `locale-uz.md`.
- `references/testing-and-verification.md` — **Prove it works.** Manual verification matrix (device/dark/sizes/Dynamic Type/RTL/offline/screen reader), @testing-library/react-native, Maestro/Detox E2E, snapshot/golden, a11y testing — evidence before ship.
- `references/web-and-universal.md` — **One codebase, three platforms.** Expo web/react-native-web, responsive breakpoints + max content width, hover/focus/keyboard nav, real anchor links + SEO, web-vs-native pitfalls — when targeting web too.
- `references/onboarding-and-illustration.md` — **First impression.** Value-first onboarding (2-3 slides, parallax, skippable), empty states that explain + act, on-brand SVG/Lottie illustration (dark-mode aware), restrained delight moments.
- `references/project-awareness.md` — **Pre-flight.** Detect the repo's framework, styling system, tokens, and components; conform to them; add-package policy. Run first on any existing repo.
- `references/design-foundations.md` — Tokens: spacing, type ramps, color roles (M3 + HIG), shape, elevation, motion.
- `references/design-to-code.md` — Token architecture (primitive→semantic→component), pixel-perfect handoff, mobile UX laws (thumb zone, touch targets, navigation), clean component code.
- `references/color-systems.md` — **Visual language: color.** The semantic accent ladder (accent/subtle/surface/border/text/on-accent), palette from seed→tonal→roles, tinted neutrals, functional colors, true dark-mode tuning, tasteful gradients.
- `references/typography-systems.md` — **Visual language: type.** Type voice/DNA per brand, font pairing, weight/tracking/line-height per role, tabular figures, optical sizing, custom-font setup per framework.
- `references/platform-personality.md` — **Same app, different soul.** iOS calm/content-first vs Android expressive/surface-rich; what changes per OS (motion/depth/components/nav/type); adapt personality, not just syntax.
- `references/product-maturity.md` — **Build for the stage.** Stage 0 MVP → 1 Growth → 2 Scale → 3 Enterprise; match UI complexity to maturity; don't over-build an MVP.
- `references/visual-hierarchy.md` — **The #1 AI fix.** Rank Primary/Secondary/Tertiary, the 3-second rule, text opacity ramp, one-primary-action, per-pattern hierarchy.
- `references/screen-density.md` — Density spectrum (dense↔airy) per screen job, spacing as hierarchical rhythm, anti-uniformity moves.
- `references/case-studies.md` — *Why* top apps feel premium: Linear, Revolut, Telegram, Airbnb, Notion, Stripe, Arc, Coinbase, Duolingo — hierarchy/density/radius/shadow/motion/color signatures + per-app AI pitfalls. Use for "make it like X".
- `references/anti-patterns.md` — The AI-UI blocklist: what NOT to do (slop tells), scanned in the polish pass.
- `references/error-handling-and-diagnostics.md` — **Zero-tolerance diagnostics.** Per-framework common error/warning/log → fix tables (Compose, SwiftUI, Flutter, RN/Expo), the diagnostics loop, and the fix-in-place workflow. Warnings treated as bugs.
- `references/design-critique-engine.md` — **Self-review gate.** The 8 ship-test questions ("would Linear/Apple ship this?"), dimension-by-dimension critique, and the BUILD→CRITIQUE→REVISE loop.
- `references/premium-scorecard.md` — **Measurable quality.** 0–10 rubric per dimension with weights + thresholds; revise any sub-threshold dimension; report format. A11y + diagnostics must be 10.
- `references/ui-restraint-and-accessibility.md` — **Mandatory final pass.** Restraint/anti-overcrowd, density & whitespace, 1:1 pixel fidelity, "what's underneath" component analysis, accessibility props per framework (label/role/state, hit area, dynamic type, contrast).
- `references/premium-polish.md` — **Premium tier.** Anti-AI-slop tells + fixes, optical adjustments, micro-interactions & haptics, motion choreography, layered depth/materials, typography craft, true dark mode, signature moves, performance-as-polish.
- `references/glassmorphism-and-materials.md` — **Translucent glass / iOS Liquid Glass.** When glass works vs fails, contrast/Reduce-Transparency/perf rules, native-material-first recipes per framework (SwiftUI `.glassEffect`/Material, Compose Haze, Flutter BackdropFilter, RN expo-blur). Apply from the polish pass, sparingly.
- `references/map-ui-patterns.md` — **UI over maps.** Overlay chrome, the map-padding↔bottom-sheet sync (the #1 map bug), marker/cluster/callout design, location-permission flow, dark map style — for Yandex MapKit, Google Maps, react-native-maps, Mapbox.
- `references/locale-uz.md` — **UZ/CIS market conventions.** UZS so'm formatting (space grouping, no decimals), `+998` phone masks + OTP, Uzcard/Humo/Visa card detection & visuals, viloyat→tuman cascading region pickers, uz-Latin/uz-Cyrillic/ru language handling.
- `references/psychology.md` — Perception laws: F/Z scan patterns, attention anchors, Gestalt grouping, cognitive load (Hick/Miller/Jakob), Fitts/thumb zone, perceived performance. Diagnose "feels off".
- `references/screen-patterns.md` — Mobbin-style screen skeletons: onboarding, auth, feed/list, detail, chat, profile, settings, search, checkout/paywall, tab scaffold. Layout anatomy per pattern.
- `references/interaction-patterns.md` — **Screens are state machines.** All states per screen (loading/empty/error/offline/submitting/success/disabled), per-flow state recipes (auth/forms/checkout/chat/search), classic flow-bug prevention.
- `references/real-world-constraints.md` — **Survive the wild.** Offline/slow network, RTL + i18n, Dynamic Type scaling, small phones + tablets/foldables, long content, permissions — the conditions AI forgets.
- `references/flow-architecture.md` — **Build the journey, not one screen.** Canonical flows (booking/messaging/transfer/onboarding/checkout) with sequence, branches, carried state, navigation shape.
- `references/motion-recipes.md` — Premium *feel*: timing tiers, spring personalities, press/choreography/sheet/hero/numeric/skeleton recipes per framework, **enter↔exit pairing**, Reduce Motion.
- `references/gestures-and-haptics.md` — Haptic-by-action map + swipe/drag/pull-refresh/long-press/reorder recipes per framework; the gesture+motion+haptic trinity.
- `references/components.md` — Cross-framework component catalog: buttons, text fields, cards, chips, list items, app bars, bottom nav/tab bar, FAB, bottom sheet, dialogs, avatars, badges.
- `references/design-review.md` — **Review/lint mode.** Audit checklist + report format for finding and refactoring design issues in existing UI code.
- `references/frameworks/jetpack-compose.md` — Compose + M3 idioms, theming, navigation, animation.
- `references/frameworks/swiftui.md` — SwiftUI + HIG idioms, theming, navigation, animation.
- `references/frameworks/flutter.md` — Flutter Material 3 + Cupertino, theming, navigation, animation.
- `references/frameworks/react-native-expo.md` — **RN/Expo platform + design playbook (SDK 56).** Styling detection, **Expo Router** (file tree, Stack/Tabs/groups/modals, typed routes, auth gating), safe-area/edge-to-edge, expo-image/font/splash/status-bar, FlashList perf + New Architecture, Reanimated 3 + Gesture Handler + haptics, expo-blur/linear-gradient/bottom-sheet, EAS/dev-build/config-plugins, package policy, per-OS adaptation.
- `references/frameworks/react-native-ui-libraries.md` — Full RN UI-library catalog: detect → lock into one (NativeWind, Paper, Tamagui, RN Elements, UI Kitten, gluestack, Restyle, Dripsy, Unistyles, vanilla) + per-library theming/component cheat-sheets + companions.
- `references/frameworks/expo-router.md` — **Deep Expo Router.** Nested layouts, route groups + `(auth)`/`(app)` split, protected routes / auth gating, dynamic + typed routes, modals/sheets, deep linking + `+not-found`, header customization, tab badges/custom bar, navigation lifecycle hooks, transitions.
