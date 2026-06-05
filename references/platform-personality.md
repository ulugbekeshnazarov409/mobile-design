# Platform Personality — same app, different soul (not just different syntax)

AI's cross-platform tell: it writes "the same UI in different syntax" — identical layout, identical motion, just Compose vs SwiftUI. But iOS and Android have different *personalities*. A great app feels like it belongs on the platform: Telegram-on-iOS feels Apple-made; Telegram-on-Android feels Material-made — same product, different soul. This file makes adaptation about *character*, not translation.

> Cross-ref: the framework references (per-platform components) and `case-studies.md`.

---

## The two personalities

### iOS — *calm, quiet, content-first*
- **Whitespace-forward**, softer, lighter chrome. Content leads; the UI gets out of the way.
- **Soft, physical motion** — gentle springs, fluid, restrained.
- **Depth via blur/materials & background tiers** (`.ultraThinMaterial`, grouped backgrounds), thin separators — not heavy shadows.
- **Large titles** that collapse on scroll; **inset-grouped lists**; bottom sheets with grabbers + detents.
- **No FAB** — primary actions live in the nav bar or a pinned bottom button.
- **Swipe-back** is sacred; swipe actions on rows.
- Type: SF, Dynamic Type, large titles.

### Android — *expressive, motion-rich, surface-rich*
- **Material expressiveness** — bolder color, stronger surfaces/elevation, more visible structure.
- **Ripple on everything**; touch feedback is loud and physical.
- **FAB** for the primary creative action; **tonal/elevated surfaces** (M3 surface-container tiers).
- **Top app bar** (small or large/collapsing); **navigation bar** with active indicator pill.
- **Emphasized motion** — Material's expressive easing, larger more characterful transitions.
- Type: Roboto/brand, Material type scale.

> Telegram principle: the *product* (chats, features, flows) is identical; the *personality* (components, motion, depth, density) is native to each OS. Aim for that — not one look forced onto both.

---

## What actually changes per platform

| Dimension | iOS | Android |
| --- | --- | --- |
| **Primary action** | nav-bar button / pinned bottom CTA | FAB / filled button |
| **Navigation** | `NavigationStack`, large titles, tab bar, swipe-back | top app bar, nav bar w/ indicator, system back |
| **Lists** | inset-grouped, thin separators | full-width rows, dividers, surface tiers |
| **Depth** | blur/materials, bg tiers, subtle shadow | tonal elevation, ripple, stronger shadow on floats |
| **Sheets** | `.presentationDetents`, grabber | M3 ModalBottomSheet, 28dp top corners |
| **Motion** | gentle springs, soft, quiet | emphasized easing, expressive, ripple |
| **Density** | a touch airier, content-first | can be a touch denser/structured |
| **Icons** | SF Symbols | Material Symbols |
| **Type** | SF + large titles + Dynamic Type | Roboto/brand + Material scale |
| **Switches/controls** | iOS toggle, segmented control | Material switch, segmented buttons |

Cross-platform code (Flutter, RN) should branch on these where it matters — use `.adaptive` widgets / `Platform.select` / `Theme.of(context).platform`, not one hardcoded look.

---

## How much to adapt (by app type)

- **Native-feel apps** (utilities, productivity, finance) → adapt strongly; users expect platform conventions. Telegram/Things model.
- **Brand-forward apps** (games, highly branded consumer) → keep a consistent brand identity across platforms, but still respect platform *navigation + ergonomics* (back behavior, safe areas, target sizes). Don't force a FAB onto iOS even in a branded app.
- **Never** ship iOS with Material ripple + FAB, or Android with iOS grouped insets + no ripple. That's the "same UI, wrong platform" smell.

The rule: **brand can be shared; navigation, components, depth, and motion personality should be native.**

---

## Per-framework application

- **Compose / SwiftUI:** you're already native — just lean into that platform's personality fully (don't make Compose look like iOS or vice-versa).
- **Flutter:** use Material 3 by default, Cupertino widgets for iOS feel; `.adaptive` constructors (`Switch.adaptive`, `CircularProgressIndicator.adaptive`); branch on `Theme.of(context).platform`; consider per-platform navigation patterns and motion.
- **RN/Expo:** `Platform.select({ ios, android })` for components, motion, shadows-vs-elevation, FAB-vs-navbar-action; Expo Router header options per platform; pick the icon set per OS. Match each platform's personality, not a single merged look.

---

## Platform personality checklist

- [ ] The app feels native to each OS — not one look in two syntaxes.
- [ ] iOS: no FAB, soft motion, blur/grouped depth, large titles, swipe-back intact.
- [ ] Android: ripple everywhere, FAB/Material surfaces, app bar + nav indicator, system back.
- [ ] Components, navigation, depth, and motion adapt per platform (brand may stay shared).
- [ ] Cross-platform code branches (`.adaptive` / `Platform.select`), doesn't hardcode one personality.
- [ ] Right icon set + type per OS.
