# Accessibility — deep pass (screen readers, dynamic type, contrast)

`ui-restraint-and-accessibility.md` §4 gives the built-in baseline (label/role/state, target size, no color-only meaning). This file goes deeper: how a screen actually reads under VoiceOver/TalkBack, how to order and group focus, how to announce async results, how to survive the largest text scale, and how to compute a contrast ratio instead of guessing. Apply it to any screen that has compound rows, async state, forms, modals, or charts.

> Cross-ref: `ui-restraint-and-accessibility.md` (baseline a11y + targets), `typography-systems.md` (type scale that must scale), `real-world-constraints.md` (largest-text / smallest-width verification), `color-systems.md` (contrast-aware role mapping).

---

## 1. Screen reader semantics — what the reader actually says

A screen reader announces, per focused element, roughly: **label → role → value → state → hint**. Your job is to make each of those correct and to make the *number of stops* small. A row that reads "Avatar. Jane Doe. image. 2 new messages. Online. button." is broken — it should be one stop: "Jane Doe, 2 new messages, online. button."

Order to get right: **Label** (what it is), **Role/trait** (button, header, image, switch…), **Value** (for adjustable/toggle), **State** (selected, disabled, expanded), **Hint** (what happens — sparingly, last).

### React Native / Expo
```tsx
<Pressable
  accessibilityRole="button"
  accessibilityLabel="Jane Doe, 2 new messages, online"
  accessibilityState={{ selected, disabled, busy: loading }}
  accessibilityHint="Opens the conversation">
  {/* children NOT individually focusable: */}
  <View accessible={false} importantForAccessibility="no-hide-descendants">…</View>
</Pressable>
```
`accessibilityRole` (`button|link|header|image|switch|checkbox|radio|adjustable|alert|summary|menuitem|none`), `accessibilityState` ({selected, disabled, checked, busy, expanded}), `accessibilityValue` ({min,max,now,text}) for sliders. **`accessible` on the parent merges children into one node** (default true on `Pressable`/`Text`). `accessibilityLiveRegion="polite"|"assertive"` (Android) re-announces a region when its text changes.

### Jetpack Compose
```kotlin
Row(Modifier
    .clickable(onClickLabel = "Open conversation") { open() }
    .semantics(mergeDescendants = true) {
        contentDescription = "Jane Doe, 2 new messages, online"
        role = Role.Button
        stateDescription = if (selected) "Selected" else "Not selected"
    }
) { /* decorative icon: */ Icon(painterResource(R.drawable.dot), contentDescription = null) }
```
`mergeDescendants = true` collapses a subtree into one node. `stateDescription` overrides the spoken state; `role` sets the trait; `disabled()` and `selected = true` set state. Hide decorative subtrees with `Modifier.clearAndSetSemantics {}` or `contentDescription = null` on `Icon`/`Image`. `liveRegion = LiveRegionMode.Polite` re-announces on change.

### SwiftUI
```swift
HStack { … }
  .accessibilityElement(children: .combine)          // merge into one element
  .accessibilityLabel("Jane Doe, 2 new messages, online")
  .accessibilityAddTraits(.isButton)
  .accessibilityValue(isOnline ? "Online" : "Offline")
  .accessibilityHint("Opens the conversation")
```
`children: .combine` merges; `.ignore` makes you supply everything; `.contain` keeps a container. Traits via `.accessibilityAddTraits(.isButton/.isHeader/.isSelected/.isToggle)` and `.accessibilityRemoveTraits`. Decorative views → `.accessibilityHidden(true)`. Toggles/sliders expose value automatically when you use native `Toggle`/`Slider`.

### Flutter
```dart
Semantics(
  button: true,
  label: 'Jane Doe, 2 new messages, online',
  hint: 'Opens the conversation',
  selected: selected,
  child: ExcludeSemantics(child: decorativeRow),  // hide visuals from a11y tree
);
// merge a compound widget into one node:
MergeSemantics(child: Row(children: [avatar, name, badge]));
```
`Semantics(label:, button:, enabled:, value:, selected:, header:, liveRegion:)`. `ExcludeSemantics` drops a subtree; `MergeSemantics` collapses descendants into one node. Prefer wrapping the *outer* tappable so the whole row is one stop.

---

## 2. Focus order & grouping

Reading order follows the visual layout top-to-bottom, leading-to-trailing — but absolute positioning, overlays, and z-order can scramble it. Verify by swiping, not by reading code.

- **Merge compound elements** (avatar + name + subtitle + badge = one node). See §1 grouping APIs. A list row, a chip, a stat tile each = one stop.
- **Hide decoration** — dividers, background blobs, gradient overlays, redundant icons. RN `importantForAccessibility="no-hide-descendants"`; Compose `clearAndSetSemantics`; SwiftUI `.accessibilityHidden(true)`; Flutter `ExcludeSemantics`.
- **Override order only when needed.** SwiftUI `.accessibilitySortPriority(Double)` (higher = earlier). Compose `Modifier.semantics { traversalIndex = -1f }` + `isTraversalGroup = true` on the container. RN/Flutter: fix the *render* order or use `Semantics(sortKey: OrdinalSortKey(n))` (Flutter).
- **Headers are landmarks.** Mark section titles as headers (`accessibilityRole="header"` / `Role`-less but `heading()` in Compose / `.accessibilityAddTraits(.isHeader)` / `Semantics(header: true)`) so users jump by heading via the rotor/local navigation.

### Managing focus on transitions
Move focus deliberately on screen change, after submit, and when a modal opens — and return it on close.

| Event | Move focus to | API |
| --- | --- | --- |
| New screen pushed | screen title / first heading | RN `AccessibilityInfo.setAccessibilityFocus(reactTag)`; iOS `UIAccessibility.post(.screenChanged, argument:)`; Android sends `TYPE_WINDOW_STATE_CHANGED` automatically; Flutter focus first node |
| Modal / sheet opens | sheet title; trap focus inside | iOS native sheets trap; RN set focus + `aria-modal`; Compose `Dialog` traps by default |
| After form submit error | the first invalid field (and announce, §3) | per-platform set-focus |
| Item deleted from list | next/previous item, not nowhere | set focus to neighbor |

Don't trap focus except in modals; always provide a way back out (close button reachable, ESC/back honored).

---

## 3. Live regions & announcements

Async results (loading finished, error, "Copied", "5 results") are invisible to a screen reader unless you announce them. Use a **live region** for content that changes in place, a **direct announcement** for transient events.

| Framework | One-off announcement | Live region (re-announce on change) |
| --- | --- | --- |
| RN/Expo | `AccessibilityInfo.announceForAccessibility('Message sent')` | `accessibilityLiveRegion="polite"` (Android) / `accessibilityRole="alert"` (both) |
| Compose | `View.announceForAccessibility(...)` via context, or set `liveRegion` | `Modifier.semantics { liveRegion = LiveRegionMode.Polite }` |
| SwiftUI | `UIAccessibility.post(notification: .announcement, argument: "Sent")` | `.accessibilityLabel` on a region + post `.layoutChanged`; iOS 17+ `AccessibilityNotification.Announcement("…").post()` |
| Flutter | `SemanticsService.announce('Sent', TextDirection.ltr)` | `Semantics(liveRegion: true, child: …)` |

Rules: **polite** for non-urgent (results, status); **assertive/alert** only for errors that interrupt. Announce once — don't fire on every keystroke. For a spinner, announce "Loading" on start and the result on finish; don't leave silence.

---

## 4. Dynamic Type / font scaling

Users scale OS text up to ~200%+ (iOS accessibility sizes, Android font size + display size). Premium UI **reflows**; AI UI clips, truncates headings, or locks text in fixed-height pills. Test every screen at the **largest** setting (see `real-world-constraints.md`).

- **Use scalable units and don't cap them.** Compose `sp` for text (not `dp`); never wrap in a hardcoded scale. SwiftUI use **text styles** (`.font(.body/.headline)`) or `.dynamicTypeSize(...DynamicTypeSize.accessibility5)` — not fixed `.system(size:)`. RN keep `allowFontScaling` at its default `true`; only the rare numeric/branding lockup may set it false, and never on body. Flutter respect `MediaQuery.textScaler` (don't override to `1.0`).
- **Let height grow.** No fixed-height containers around text; use min-height + wrap. Replace `height:` with `minHeight:`/padding. Buttons grow with their label.
- **Reflow at large sizes.** Switch horizontal rows to vertical when text is huge: SwiftUI `@Environment(\.dynamicTypeSize) ` + `ViewThatFits`; Compose check `LocalDensity`/`fontScale`; RN `useWindowDimensions` + `PixelRatio.getFontScale()`; Flutter `MediaQuery.textScalerOf(context).scale(...)`.
- **Cap only with intent.** If a layout truly can't take `accessibility5`, clamp the *scaler* (e.g. SwiftUI `.dynamicTypeSize(...(.accessibility2))`) rather than disabling scaling — and only on dense controls, never on primary reading content.

```swift
// reflow a stat row to a column at accessibility sizes
@Environment(\.dynamicTypeSize) private var size
var body: some View {
  let big = size >= .accessibility1
  return ViewThatFits(in: .horizontal) { HStack { label; value } ; VStack(alignment:.leading){ label; value } }
}
```

See `typography-systems.md` for the scalable type ramp these sizes ride on.

---

## 5. Contrast — compute it, don't eyeball it

WCAG AA targets: **4.5:1** for normal text, **3:1** for large text (≥ 24px / 18.66px bold) and for **UI components / icons / focus indicators / graphical objects**. AAA is 7:1 (use for long-form reading where you can).

**Relative luminance** of an sRGB color: for each channel `c ∈ {R,G,B}` normalized to 0–1, `c_lin = c/12.92 if c ≤ 0.03928 else ((c+0.055)/1.055)^2.4`; then `L = 0.2126·R_lin + 0.7152·G_lin + 0.0722·B_lin`. **Contrast ratio** `= (L_lighter + 0.05) / (L_darker + 0.05)`, range 1:1 → 21:1.

```ts
const lin = (c: number) => { c /= 255; return c <= 0.03928 ? c/12.92 : ((c+0.055)/1.055)**2.4; };
const lum = (r:number,g:number,b:number) => 0.2126*lin(r)+0.7152*lin(g)+0.0722*lin(b);
const ratio = (a:[number,number,number], b:[number,number,number]) => {
  const [L1,L2] = [lum(...a), lum(...b)].sort((x,y)=>y-x);
  return (L1 + 0.05) / (L2 + 0.05);              // ≥4.5 text, ≥3 large/UI
};
```

- **Check against the actual background**, including translucency: a 60%-opacity label over a blurred photo can fail. Composite the real pixel first (see `glassmorphism-and-materials.md` for legibility scrims).
- **Check both light and dark** — a role that passes in light often fails in dark and vice-versa. Map contrast-safe steps per mode (`color-systems.md` §2).
- **Don't rely on color alone.** Error/success/selected must carry a second cue: icon, text label, weight, underline, or shape. Color-blind users and grayscale screenshots must still parse the state. (Restated from `ui-restraint-and-accessibility.md` §4 because it's the most-violated rule.)
- **Placeholder, disabled, and helper text** still need contrast (disabled is exempt from WCAG but should remain *perceivable*). Don't ship 2:1 gray-on-gray captions.

---

## 6. Touch targets, hit slop, spacing

- Minimum hit area **44pt (iOS) / 48dp (Android)** even when the glyph is 16–24. Expand via padding/`hitSlop`, not by scaling the visual.
- **Space between adjacent targets** ≥ ~8dp so a tap doesn't hit the neighbor; WCAG 2.5.8 wants 24×24 CSS-px minimum with spacing. Icon-only toolbars are the usual offender — give them real padding.
- RN `hitSlop={{top:8,…}}`; Compose `Modifier.minimumInteractiveComponentSize()` (and don't shrink with a tiny `size()`); SwiftUI `.frame(minWidth:44,minHeight:44)` + `.contentShape(Rectangle())` so the whole frame is tappable; Flutter `materialTapTargetSize: MaterialTapTargetSize.padded` or wrap in a 48-sized `InkWell`/`GestureDetector`.

---

## 7. OS accessibility flags — read and respond

Respond to system preferences; don't hardcode motion/opacity/weight.

| Preference | RN | Compose | SwiftUI | Flutter |
| --- | --- | --- | --- | --- |
| Reduce Motion | `AccessibilityInfo.isReduceMotionEnabled()` + `'reduceMotionChanged'` | `LocalAccessibilityManager` / check `Settings.Global` animator scale | `@Environment(\.accessibilityReduceMotion)` | `MediaQuery.disableAnimationsOf(context)` |
| Reduce Transparency | `AccessibilityInfo.isReduceTransparencyEnabled()` (iOS) | — (use opaque fallback) | `@Environment(\.accessibilityReduceTransparency)` | `MediaQuery.of(context).accessibleNavigation` (proxy) |
| Bold Text | `'boldTextChanged'` (iOS) | follows font weight settings | `@Environment(\.legibilityWeight) == .bold` | `MediaQuery.boldTextOf(context)` |
| High contrast / Increase Contrast | — | system high-contrast theme | `@Environment(\.colorSchemeContrast) == .increased` | `MediaQuery.highContrastOf(context)` |
| Screen reader on | `AccessibilityInfo.isScreenReaderEnabled()` | `AccessibilityManager.isEnabled` | `UIAccessibility.isVoiceOverRunning` | `MediaQuery.accessibleNavigationOf(context)` |

Responses: **Reduce Motion** → swap large transitions/parallax/auto-play for a cross-fade or instant change (see `motion-recipes.md`). **Reduce Transparency** → replace blur/glass with an opaque tinted surface. **Bold Text / Increase Contrast** → bump weight and use the high-contrast color step. Never *remove* a feature — degrade it gracefully.

---

## 8. Testing — actually verify, don't assume

Code that "has labels" still reads wrong. Turn the reader on and swipe.

- **iOS VoiceOver:** Settings → Accessibility → VoiceOver, or triple-click side button. Swipe right through every element; confirm order, that each interactive is reachable, labels make sense, decoration is silent, and modals trap+return focus. Use **Xcode → Open Developer Tool → Accessibility Inspector** (audit + element inspector + Dynamic Type slider) on simulator.
- **Android TalkBack:** Settings → Accessibility → TalkBack (or volume-key shortcut). Swipe through; check the same. Run **Accessibility Scanner** (Play Store) for target-size/contrast/label flags, and Compose UI tests with `assertContentDescriptionEquals` / `printToLog()` of the semantics tree.
- **Dynamic Type:** crank OS text to the largest accessibility size and re-scan every screen for clipping, truncated headings, overlapping rows, off-screen buttons. Repeat in dark mode.
- **Contrast:** sample real composited pixels (over images/blur) with the §5 formula or a checker; verify light + dark.
- **Color-blind / grayscale:** view key screens desaturated — every state must still be distinguishable.
- **Keyboard / switch (where relevant):** ensure focus is visible and nothing is reachable-but-not-operable.

---

## Quality checklist (a11y deep)

Screen reader
- [ ] Every interactive reads as one stop: correct label → role → value → state; hints sparse and last.
- [ ] Compound rows/chips/tiles merged into a single node; decoration silent.
- [ ] Headings marked as headers; reading order verified by swiping (not by code).

Focus & async
- [ ] Focus moves to title on screen change, to first invalid field on submit error, to a neighbor after delete; returns on modal close.
- [ ] Modals trap focus and are escapable; nothing else traps.
- [ ] Loading/result/error announced via live region or announcement (polite vs assertive chosen correctly).

Type & layout
- [ ] Text uses scalable units, `allowFontScaling`/Dynamic Type honored, not capped on body.
- [ ] Verified at largest accessibility text size: no clipping, reflows instead of truncating; rechecked in dark mode.

Contrast & color
- [ ] Text ≥ 4.5:1, large/UI/icons/focus ≥ 3:1, computed against the *real* (composited) background, in light **and** dark.
- [ ] No state conveyed by color alone — second cue present; passes desaturated.

Targets & OS flags
- [ ] Hit area ≥ 44pt/48dp via padding/hitSlop; ≥ ~8dp between adjacent targets.
- [ ] Reduce Motion, Reduce Transparency, Bold Text, Increase Contrast read and responded to.

Tested
- [ ] Actually navigated with VoiceOver and TalkBack; ran Accessibility Inspector / Accessibility Scanner.
