# Restraint, Density & Accessibility — the "don't ruin it" pass

The #1 way AI breaks a good UI is **doing too much**: too much text, too many components, too many colors, badges and icons everywhere, every element shouting. A real, premium 1:1 screen is mostly **restraint** — only what's needed, where it's needed, with room to breathe. Run this pass on every screen before declaring done.

---

## 1. Restraint — put only what's needed, where it's needed

**Less is the feature.** Before adding any element, ask: does this earn its place? If it doesn't change what the user can do or understand, cut it.

Hard limits per screen (exceed only with a reason):
- **One** primary action. Everything else is visibly secondary/tertiary.
- **One** accent color used deliberately; the rest is neutral surface/text. Not a rainbow.
- **2–3** type sizes; differentiate with weight/color, not more sizes.
- **≤ ~3** top-bar actions; overflow to a menu.
- **No** decorative icon on every row/label. Icons earn their place (recognition, action), not decoration.
- **No** badge/pill/tag spam. A badge means "unread/urgent", not "looks busy".

Text discipline (the most common AI failure):
- Headlines short (≤ ~5 words). Body ≤ 2 lines in cards/rows; **truncate** long content with `numberOfLines`/ellipsis, don't let it explode the layout.
- One label per thing — no "Email Address (your email)" redundancy. No filler helper text.
- Don't narrate the UI ("Tap the button below to continue"). Let the design speak.
- Empty states: one line + one action, not a paragraph.

Visual density & spacing:
- Group related items tight (8–12), separate groups wide (24–32). Negative space is part of the design, not waste — don't fill it.
- Consistent rhythm: same screen margin, same card padding, same row height. Drifting values read as broken.
- Don't stack many full-width cards of equal weight — vary density, use sections, let the eye rest.
- Alignment: pick a grid and hold it. Stray indents/centering look amateur.

Color & decoration:
- Neutral carries the UI; accent highlights the one thing that matters. Avoid multiple saturated colors competing.
- Subtle elevation/borders over heavy shadows and thick outlines. No gradient-on-everything.

> If a screen feels busy, the fix is almost always **remove**, not rearrange. Cut elements, merge labels, widen spacing, reduce colors.

---

## 2. Pixel-perfect / 1:1 real screens

To reproduce a real screen (from a reference/Figma/screenshot) faithfully:
- **Match exact values**: spacing, font size/weight/line-height, color, radius, border, shadow — read them, don't approximate; snap to the nearest project token and flag drift.
- **Match the box model**: gap between items ≠ margin on each; internal padding ≠ external spacing. Get this right or the whole layout shifts.
- **Match sizing behavior**: fixed vs fill vs intrinsic/grow. Wrong choice breaks on long text and small screens.
- **Match hierarchy & density**: the reference's relative sizes and weights, not just the pixels of one element.
- **Verify on real constraints**: smallest target width + largest Dynamic Type + dark mode. A screen that's 1:1 on one device but breaks on text-scale isn't done.
- **Honor the platform** when it conflicts with the raw design (native control beats a pixel-matched fake), and note it.

---

## 3. "What's underneath" — analyze before you place

For every component placed, the skill should know *what it actually is, which props it needs, and why it sits where it sits*. Don't drop components blindly. Quick analysis per element:

- **Right primitive?** Is this a `Pressable`/`Button`, a `List.Item`, a `Card`, a sheet? Use the semantic component, not a styled `View` faking a button.
- **Required props wired?** Interactive → `onPress`/handler + disabled/loading states. Input → value/onChange + keyboard type + content-type. Image → source + `contentFit` + placeholder + alt/label.
- **Placement justified?** Primary action in the thumb zone (bottom). Back/title top. Destructive isolated. Nav 3–5 items. Each placement traces to a rule in `design-to-code.md`.
- **State coverage?** default / pressed / focused / disabled / loading / selected / error / empty — implement those that apply.
- **Token-bound?** No raw hex / magic number inside the component; values come from the project theme.

State it briefly in the plan ("FAB bottom-right because primary create action; uses theme primary; ripple feedback") so placement is intentional, not decorative.

---

## 4. Accessibility — built in, not bolted on

Every interactive element gets a label, a role, and correct state. Targets meet minimum size. Text scales. Contrast passes.

**Universal rules**
- Hit area ≥ **48dp (Android) / 44pt (iOS)**; expand small visuals via padding/`hitSlop`, not by enlarging the icon.
- Contrast ≥ **4.5:1** body text, **3:1** large text/icons/UI.
- Respect OS text scaling / Dynamic Type — don't lock text in fixed-height boxes that clip.
- Respect Reduce Motion. Provide focus order and don't trap focus.
- Don't convey meaning by color alone (add icon/label/text).

**React Native / Expo**
```tsx
<Pressable
  onPress={onPress}
  accessibilityRole="button"
  accessibilityLabel="Send message"
  accessibilityState={{ disabled, busy: loading }}
  hitSlop={8}>
  …
</Pressable>
```
- `accessibilityRole` (`button`/`link`/`header`/`image`/`switch`/`adjustable`…), `accessibilityLabel`, `accessibilityHint` (sparingly), `accessibilityState` ({disabled, selected, checked, busy, expanded}), `accessibilityValue` for sliders. Group with `accessible`/`accessibilityElementsHidden`. Decorative images → `accessibilityElementsHidden` / no label. Honor `AccessibilityInfo.isReduceMotionEnabled`. Use `allowFontScaling` (default true) — don't disable.

**Jetpack Compose**
- `Modifier.semantics { contentDescription = "…"; role = Role.Button; stateDescription = … }`, `Modifier.clearAndSetSemantics` to hide decorative, `Modifier.minimumInteractiveComponentSize()` for targets, `mergeDescendants` for grouping. Decorative `Icon(contentDescription = null)`. Respect `LocalContext` font scale.

**SwiftUI**
- `.accessibilityLabel`, `.accessibilityHint`, `.accessibilityValue`, `.accessibilityAddTraits(.isButton/.isHeader)`, `.accessibilityHidden(true)` for decorative, `.accessibilityElement(children: .combine)` to group. Dynamic Type via text styles; test largest sizes. `@Environment(\.accessibilityReduceMotion)`. VoiceOver order with `.accessibilitySortPriority`.

**Flutter**
- Wrap in `Semantics(label:, button: true, enabled:, value:)`; `ExcludeSemantics`/`Semantics(excludeSemantics:)` for decorative; `MergeSemantics` to group. Min target via `materialTapTargetSize` / sized `InkWell`. `MediaQuery.textScaler` respected; don't clamp. Honor `MediaQuery.disableAnimations`.

---

## 5. Final pass checklist (run before "done")

Restraint
- [ ] One primary action; one accent; 2–3 type sizes.
- [ ] Removed everything that didn't earn its place; no icon/badge/text spam.
- [ ] Long text truncates; no layout-breaking content.
- [ ] Consistent margins/padding/row heights; generous, intentional whitespace.

Fidelity
- [ ] Values match the reference and snap to tokens; box model correct.
- [ ] Verified at small width, largest text scale, and dark mode.
- [ ] Platform conventions honored where they conflict with the raw design.

Underneath
- [ ] Each element uses the correct semantic component with required props wired.
- [ ] Placement justified by a rule; all relevant states implemented.

Accessibility
- [ ] Labels + roles + states on all interactives; targets ≥ 48dp/44pt.
- [ ] Contrast passes; text scales; reduce-motion respected; no color-only meaning.
