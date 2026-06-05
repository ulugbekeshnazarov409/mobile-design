# Premium Polish — what separates "shipped by a top studio" from "AI default"

Two screens can use the same components, tokens, and layout and still feel worlds apart. The difference is **polish**: optical correctness, micro-interactions, depth, motion choreography, typographic care, and tasteful restraint. This file is the layer that pushes output from "fine" to "premium" (Linear / Revolut / Airbnb / Things / Arc tier). Apply it on top of the foundations after the screen works.

---

## 1. The AI-slop tells — and their premium fix

Premium design is partly defined by the absence of these. Audit for each:

| ❌ AI / generic tell | ✅ Premium move |
| --- | --- |
| Everything full-width, equal weight, evenly spaced | Deliberate hierarchy; vary size/density; one focal element |
| Pure black `#000` / pure white `#FFF` surfaces | Off-black `#0B0B0F`–`#121212`, off-white `#FAFAFA`; tinted neutrals |
| One flat gray for all text | A text ramp: primary / secondary / tertiary (e.g. 100% / 60% / 38% opacity) |
| Saturated brand color used everywhere | Accent on the **one** thing that matters; neutrals carry the rest |
| Hard drop shadows (`#000` 0.5) | Soft, low-opacity, layered shadows (y 2–8, blur 8–24, 0.06–0.14) or tonal elevation |
| Default 8px radius on everything | Intentional radius scale; larger continuous corners on cards/sheets |
| Icons on every label, badges everywhere | Icons/badges earn their place; mostly text + space |
| Instant, no feedback on tap | Press scale/opacity + haptic; state transitions animated |
| Centered everything | A real grid; left-aligned content; optical alignment |
| Emoji as UI icons | A consistent icon set (SF Symbols / Material Symbols / Lucide) |
| Generic system font at default tracking | Tuned type: right weight, line-height, letter-spacing, tabular figures for numbers |
| Stock gradients on cards/buttons | Flat or barely-there gradients; gradient as accent, not default |

> Rule of thumb: if it looks like a Bootstrap admin template or a default Figma community kit, it isn't premium yet.

---

## 2. Optical adjustments (the "designer eye")

Math-correct often looks wrong. Premium UIs correct optically:

- **Optical alignment > metric alignment.** Icons next to text often need a 0.5–1px nudge to sit on the baseline. Triangular/circular shapes (play icon, chevrons) look centered when slightly offset.
- **Optical padding.** A pill with an icon needs slightly less trailing padding than leading; text-only buttons need a hair more vertical than they "should".
- **Border compensation.** A 1px border eats interior space — add it back so bordered and unbordered cards read the same size.
- **Type optical sizing.** Large display text needs **tighter** letter-spacing (−1 to −2%); small caps/labels need **looser** (+2 to +5%). Never leave display text at default tracking.
- **Icon weight matching.** Icon stroke weight should match adjacent text weight; 24px icon next to 17pt body, not a chunky 28px.
- **Even visual rhythm.** Equal *pixel* gaps can look uneven when elements differ in visual weight — adjust so it *reads* even.

---

## 3. Micro-interactions & feedback (premium lives here)

Every interactive element should respond. Subtle, fast, physical.

- **Press states:** scale to ~0.96–0.98 + slight opacity on tap-down, spring back on release. (Compose: `animateFloatAsState` on press via `interactionSource`; SwiftUI: `.scaleEffect` + `.animation` or `ButtonStyle`; Flutter: `AnimatedScale`/`InkWell`; RN: `Pressable` `({pressed})` + Reanimated.)
- **Haptics on meaningful actions:** selection change, toggle, success, pull-to-refresh trigger, sheet snap, destructive confirm. Light/selection feedback — not on every tap. (iOS: `.sensoryFeedback` / `UIImpactFeedbackGenerator`; Android: `HapticFeedbackConstants` / `view.performHapticFeedback`; Flutter: `HapticFeedback.selectionClick`; RN/Expo: `expo-haptics`.)
- **State transitions:** toggles slide, checkmarks draw, counts roll (numeric content transition), chips fill — don't hard-swap.
- **List motion:** staggered fade/slide-in on first load (30–50ms per item, cap the stagger); animate insert/remove; swipe actions with resistance.
- **Loading:** skeletons that match the real layout with a subtle shimmer sweep — not a centered spinner on a blank screen. Spinner only for the very first cold load.
- **Pull-to-refresh & overscroll:** native rubber-banding; custom refresh control that feels weighted.
- **Empty/success moments:** a small, tasteful illustration or animated check — a moment of delight, used sparingly.

---

## 4. Motion choreography

Beyond "animate the thing" — orchestrate it.

- **Enter decelerate, exit accelerate.** Things arrive softly (ease-out / spring), leave quickly (ease-in).
- **Stagger related groups**, don't fire everything at once. Parent moves, children follow.
- **Shared-element / hero transitions** for list → detail (image and title persist across the push). (Compose shared-element transitions; SwiftUI `matchedGeometryEffect`; Flutter `Hero`; RN reanimated shared transitions.)
- **Spring feel for interaction**, tween for content. Springs: medium response (~0.3–0.4s), high damping (0.8–0.9) — no wobble unless intentional.
- **Continuity:** the element you tapped is the element that expands. Motion explains *where things came from and went*.
- **Restraint:** 150–300ms, one or two things moving at once. Respect Reduce Motion — degrade to fades.

---

## 5. Depth & materials (tasteful, layered)

- **Layered elevation**, not one big shadow: a small ambient shadow + a slightly larger key shadow reads richer than a single hard one.
- **Tonal depth** (M3): use surface-container tiers for elevation; reserve real shadows for floating elements (FAB, menus, sheets).
- **Glass / blur** for bars and sheets over content (iOS `.ultraThinMaterial`; Android/RN blur views) — sparingly, where content scrolls underneath.
- **Subtle borders** (`outlineVariant` / 1px hairline at ~8–12% contrast) to separate without shadow.
- **Backgrounds with life:** a faint radial/linear tint, a very low-opacity noise/grain, or a soft gradient behind a hero — barely perceptible, adds richness. Never a loud gradient.

---

## 6. Typography craft

- **Pair intentionally:** one expressive display/heading face + one highly legible UI/body face, or a single great variable font across weights. Don't ship three random fonts.
- **Weights with purpose:** typically Regular (body), Medium (labels/emphasis), Semibold/Bold (titles). Avoid Light for small text.
- **Line-height:** ~1.3–1.5 for body, tighter (~1.1–1.2) for large headings.
- **Letter-spacing:** tighten big text, loosen small caps/labels (see §2).
- **Tabular / monospaced figures** for prices, counters, timers, tables — so numbers don't jitter.
- **Measure & truncation:** comfortable line length; ellipsize gracefully; never let a long string blow the layout.
- **Custom fonts the right way:** Compose `FontFamily` + `Font(...)`; SwiftUI custom font + Dynamic Type via `relativeTo`; Flutter `pubspec` fonts + `TextTheme`; RN/Expo `expo-font` + `useFonts` gating render.

---

## 7. Color & dark mode mastery

- **Build a real palette**, don't grab raw hex per element: pick a brand hue → derive a tonal scale → map semantic roles. (M3 `ColorScheme.fromSeed`; or a hand-tuned scale.)
- **Dark mode is not inverted light mode:** raise surface lightness with elevation (not pure black), reduce accent saturation slightly, lower text to ~87/60/38% white, soften shadows (they barely show — lean on tonal surfaces and borders).
- **One accent, plus functional colors** (success/warn/error/info) used only for status.
- **Contrast is non-negotiable** (4.5:1 text, 3:1 large/icon) — premium *and* accessible.
- **Tinted neutrals:** nudge grays slightly toward the brand hue for cohesion instead of dead neutral gray.

---

## 8. Imagery & iconography

- **Content-first imagery:** real photos/avatars with consistent aspect ratios, rounded with the shape scale, always with a placeholder (blurhash/skeleton) and a failure state.
- **One icon set, one style** (all outlined or all filled within a context); match size and stroke to text.
- **Gradient/scrim over images** behind overlaid text for legibility (bottom-up dark scrim).
- **Empty states** get a small, on-brand illustration or icon — a reason to feel the product is cared for.

---

## 9. Signature "premium move" recipes

Drop-in upgrades that instantly read high-end (use where they fit, don't pile them all on):

- **Collapsing/parallax header:** large title + hero that shrinks into the bar on scroll.
- **Sticky section headers** in long lists; floating date pills in chat.
- **Segmented control / animated tab indicator** that slides between options.
- **Bottom sheet with detents** that snap (medium → large) with a grabber and haptic on snap.
- **Numeric roll / content transition** on counters, balances, prices.
- **Card press depth:** card lifts slightly and brightens on press.
- **Frosted bottom bar** with content scrolling beneath.
- **Subtle entrance:** screen content fades/slides up 8–12px on mount.
- **Pull-to-refresh with a custom, weighted control** + haptic at the trigger threshold.

---

## 10. Performance is polish

Jank kills the premium feeling instantly. Keep it 60fps (or 120 on ProMotion):

- Virtualize long lists (`LazyColumn` / `FlatList`/`FlashList` / `ListView.builder`) — never map huge data into a scroll view.
- Run animations on the UI/compositor thread (Reanimated worklets; Compose `graphicsLayer`; avoid animating layout where a transform works).
- Memoize/stabilize list items and keys; avoid rebuilding the world on every state change.
- Optimize images (correct resolution, caching, blurhash placeholder); don't decode huge bitmaps.
- Debounce expensive work; keep the main thread free during gestures.

---

## Premium pass checklist

- [ ] No slop tells (§1): off-black/white, text ramp, single accent, soft layered shadows, consistent icons.
- [ ] Optical adjustments applied to type tracking, icon/text alignment, padding.
- [ ] Every interactive has a press state + feedback; key actions have haptics.
- [ ] Motion choreographed (enter/exit, stagger, shared element); springs tuned; Reduce Motion handled.
- [ ] Depth is layered/tonal; blur/gradient used sparingly and tastefully.
- [ ] Typography tuned (weights, line-height, tracking, tabular figures); fonts loaded correctly.
- [ ] Real palette + a true dark mode (not inverted); contrast passes.
- [ ] Imagery has placeholders + scrims; empty states feel cared for.
- [ ] One or two signature moves where they fit — not all at once.
- [ ] 60fps: lists virtualized, animations off the main thread, images optimized.
