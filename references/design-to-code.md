# Design → Code — principles for engineering UI correctly

Distilled, framework-agnostic rules for turning design intent into clean, correct mobile code. Synthesized from the practices the field treats as canonical (UX Collective design-implementation guidelines, Design Systems Collective token/component architecture, Smashing Magazine mobile UX & touch ergonomics). Apply these on top of the platform references.

---

## 1. Token architecture — three tiers (Design Systems Collective)

Don't scatter raw values. Structure tokens in tiers so themes and changes propagate:

1. **Primitive (global) tokens** — the raw palette/scale: `blue-500 = #3B82F6`, `space-4 = 16`, `radius-md = 12`. No meaning, just values.
2. **Semantic (alias) tokens** — intent mapped to primitives: `color.primary → blue-500`, `color.surface`, `color.text.secondary`, `space.screen-margin → space-4`. **Components reference these**, never primitives.
3. **Component tokens** (optional) — per-component overrides: `button.height`, `button.radius → radius-md`, `card.padding → space-4`.

Result: change one semantic mapping → whole app reflows. Dark mode = swap the semantic layer, not every component. This is exactly how M3 color roles and iOS semantic colors work — mirror it in your own token files.

Rule: **a component should never contain a raw hex or magic number.** If you typed `#`, `0xFF`, or a stray pixel value inside a component, route it through a token instead.

---

## 2. Pixel-perfect handoff (UX Collective)

When implementing from a design (Figma/spec):
- **Read the real values**, don't approximate: spacing, font size/weight/line-height, color, radius, border, shadow. Pull them from the design's tokens/inspect panel, then map to the nearest project token (and flag if the design drifts off-grid).
- **Match the box model:** padding vs margin vs gap. A "16 gap" between cards is `gap`, not a margin on each child. Internal padding ≠ external spacing.
- **Respect intrinsic vs fixed sizing:** does this grow with content, fill width, or have a fixed size? Wrong choice = broken layout on long text / small screens.
- **States are part of the design:** default, pressed, focused, disabled, loading, selected, error. Implement all that apply, even if only "default" was drawn.
- **Optical alignment** over mathematical when they conflict (icon baselines, text inside pills).
- **Density & breakpoints:** verify on a small device and with the largest Dynamic Type. Don't hardcode for one screen size.

When the design and platform convention conflict, prefer the platform convention and note it — a pixel-matched but non-native control feels wrong.

---

## 3. Mobile UX laws (Smashing Magazine)

### Thumb zone & reachability
- One-handed use is the default. The **bottom third** of the screen is the easy-reach zone for the thumb; the top corners are hardest.
- Put **primary actions and primary navigation at the bottom** (bottom CTA, bottom tab bar, bottom-sheet actions). Reserve the top for titles, back, and low-frequency actions.
- On tall phones, avoid critical actions stranded at the very top. If a destructive/primary action must be top (e.g. nav-bar Save on iOS), keep it where the platform expects it.

### Touch targets
- Minimum **48×48 dp (Android)** / **44×44 pt (iOS)** interactive area, with ≥ 8dp spacing between targets. Visual size can be smaller if hit area (padding/`hitSlop`) meets the minimum.
- Don't place tiny tap targets at screen edges or crowd them together.

### Navigation patterns
- 3–5 top-level destinations in a bottom bar; more → "More" tab or a different pattern. Labels visible.
- Keep back/dismiss predictable: iOS swipe-back + back button; Android system back must work. Don't trap the user in a flow without an exit.
- Match depth to platform: tabs for peers, push/stack for drill-down, modal/sheet for focused sub-tasks, full-screen modal for create flows.

### Forms & input
- Right keyboard per field (email/number/phone); autofill/content-type hints; inline validation that never blocks typing; show, don't hide, requirements.
- Keep the focused field and primary button visible above the keyboard.

---

## 4. Clean component code (Design Systems Collective)

- **Single responsibility:** a component does one thing; compose small pieces rather than one giant configurable monster.
- **Props are an API:** name them by intent (`variant="primary"`, `size="lg"`, `tone="danger"`), not by raw style. Sensible defaults; few required props.
- **Stateless presentation + thin stateful wrapper:** keep visuals pure and reusable; isolate data/side-effects.
- **Variants over duplication:** one `Button` with variants beats `PrimaryButton`, `RedButton`, `BigButton`.
- **No copy-paste styling:** shared values come from tokens; shared layout from shared components.
- **Accessibility built in:** labels/roles, focus order, contrast, dynamic type — part of the component, not an afterthought.

---

## 5. Quick implementation rubric

Before shipping a screen or component, confirm:
- [ ] Every value traces to a token (no raw hex/magic numbers in components).
- [ ] All relevant states implemented (default/pressed/disabled/loading/error/empty).
- [ ] Primary action/nav in the thumb zone; targets ≥ 48dp/44pt.
- [ ] Works at small width and largest text size; safe areas respected.
- [ ] Platform convention honored where it conflicts with the raw design.
- [ ] Component is small, variant-driven, stateless-where-possible, accessible.

Use `design-review.md` to audit existing code against these rules.
