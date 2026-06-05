# Anti-Patterns — what NOT to do (the AI UI blocklist)

A short, blunt list of the moves that instantly mark a screen as AI-generated / amateur. Treat each as a hard "don't". When reviewing your own output, scan this list first — removing these does more for quality than adding anything.

> Meta-rule: **when a screen feels off, the fix is almost always to REMOVE, not add.**

---

## Layout & structure

- ❌ **Dashboard with 8–12 equal cards.** A wall of identical cards = no hierarchy. → Make one dominant; group the rest into sections or compact rows.
- ❌ **Everything full-width, evenly spaced** (card, gap, card, gap, button). → Vary density; hierarchical spacing; group related items.
- ❌ **Nested cards** (a card inside a card inside a card). → Flatten; use spacing/sections or one card with internal dividers.
- ❌ **Floating CTA + FAB + bottom tab bar all at once.** → One primary action mechanism per screen.
- ❌ **Centering everything.** → Left-align content (LTR); reserve centering for empty states / single focal moments.
- ❌ **Carousels for primary content** the user needs to compare. → Use a list/grid they can scan.
- ❌ **Everything inside a card.** Not all content needs a card; edge-to-edge lists/images often feel more native.

---

## Color

- ❌ **5 accent colors competing.** → One accent; neutrals carry the UI; functional colors (success/warn/error) for status only.
- ❌ **Accent color on many elements** (every icon, every label). → Accent the single primary thing.
- ❌ **Pure black `#000` / pure white `#FFF` surfaces.** → Off-black (#0B0B0F–#121212), off-white (#FAFAFA); tinted neutrals.
- ❌ **Gradients everywhere** (every card and button). → Gradient as occasional accent (hero/brand), not default.
- ❌ **Red/green used decoratively** in finance/data. → Reserve strictly for gain/loss/status.
- ❌ **Color as the only signal** of state/meaning. → Add icon/label/text too (accessibility).

---

## Shadows & depth

- ❌ **A shadow on every element.** → Shadows only for things that float (FAB, menus, sheets); use borders/tonal tiers otherwise.
- ❌ **Hard black 0.5-opacity drop shadows.** → Soft, low-opacity (0.06–0.14), layered.
- ❌ **Heavy borders + heavy shadows together.** → Pick one separation method per context.

---

## Typography

- ❌ **One flat gray for all text.** → P/S/T opacity ramp (see `visual-hierarchy.md`).
- ❌ **Five arbitrary font sizes.** → 2–3 sizes; differentiate by weight + color.
- ❌ **Display text at default letter-spacing.** → Tighten large text; loosen small caps/labels.
- ❌ **Non-tabular figures for money/counters/timers.** → Tabular/monospaced figures so numbers don't jitter.
- ❌ **Light weight on small text.** → Regular/Medium minimum for legibility.
- ❌ **Three unrelated fonts.** → One pair (display + body) or one good variable font.
- ❌ **Emoji as UI icons.** → A consistent icon set (SF Symbols / Material Symbols / Lucide).

---

## Content & text

- ❌ **Narrating the UI** ("Tap the button below to continue"). → Let the design speak.
- ❌ **Redundant labels** ("Email Address (enter your email)"). → One clear label.
- ❌ **Paragraph empty states.** → One line + one action.
- ❌ **Untruncated long strings** that blow the layout. → `numberOfLines`/ellipsis.
- ❌ **Filler placeholder content** that hides real-data problems (long names, empty lists). → Design for real, messy content.

---

## Components & interaction

- ❌ **A styled `View`/`div` faking a button.** → Use the real interactive component (press state, role, a11y).
- ❌ **No press feedback** (instant, dead taps). → Scale/opacity + haptic on meaningful actions.
- ❌ **Tiny touch targets** (<48dp/44pt) or crowded targets. → Minimum hit area + spacing (hitSlop/padding).
- ❌ **Happy-path only** (no empty/loading/error). → Implement all relevant states.
- ❌ **Spinner on a blank screen** for content load. → Skeletons matching the layout.
- ❌ **Hard-swapping state** (no transition on toggle/expand/select). → Animate state changes.
- ❌ **Long lists in a ScrollView `.map()`.** → Virtualize (LazyColumn/FlatList/FlashList/ListView.builder).

---

## Platform

- ❌ **Material look forced on iOS** (or Cupertino forced on Android). → Adapt per platform.
- ❌ **FAB on an iOS-only screen.** → iOS uses toolbar/nav-bar actions.
- ❌ **Breaking system back/swipe-back** or trapping the user with no exit. → Honor platform navigation.
- ❌ **Ignoring safe areas** (content under notch/home indicator/status bar). → Respect insets.
- ❌ **Locking text size** / breaking Dynamic Type with fixed-height boxes. → Let text scale.

---

## Motion

- ❌ **Animating decoration** / things looping for no reason. → Animate state and navigation, not idle decor.
- ❌ **Slow/janky animations** (>400ms, dropped frames). → 150–300ms, run off main thread, 60fps.
- ❌ **Bounce/wobble everywhere.** → Springs with high damping; reserve playful bounce for intentionally playful apps.
- ❌ **Ignoring Reduce Motion.** → Degrade to fades.

---

## The "would they ship this?" gut check

Before declaring done, ask: would **Linear / Apple / Revolut** ship this screen? If the honest answer is no, it's usually because of an item above — find it and remove it.
