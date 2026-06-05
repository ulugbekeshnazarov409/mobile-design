# Premium Scorecard — score every screen, revise the weak dimensions

A concrete rubric to grade a screen 0–10 per dimension, so "premium" becomes measurable instead of vibes. Score honestly, then **revise any dimension below its threshold** and re-score. Pairs with `design-critique-engine.md` (critique finds the *what*; the scorecard tracks the *how much*).

> Be a harsh grader. A 7 means "fine, not premium." Premium screens score **8+ everywhere**, with accessibility and diagnostics at **10** (non-negotiable).

---

## The dimensions & thresholds

| Dimension | Weight | Min to ship | What it measures |
| --- | --- | --- | --- |
| **Hierarchy** | ×3 | ≥ 8 | One clear focal point; P/S/T ranking; eye lands in 0.5s |
| **Restraint** | ×3 | ≥ 8 | Nothing unneeded; one accent; not "too much" |
| **Spacing & density** | ×2 | ≥ 8 | Right density; hierarchical, grouped spacing (not even gaps) |
| **Premium feel** | ×3 | ≥ 8 | No slop tells; off-black/white, text ramp, soft depth, optical type |
| **Motion & feedback** | ×2 | ≥ 7 | Press states, haptics, choreographed transitions |
| **Platform fit** | ×2 | ≥ 8 | Right components per OS; native conventions; back/nav correct |
| **States** | ×2 | ≥ 8 | Empty / loading / error / disabled all handled |
| **Accessibility** | ×3 | **= 10** | Labels/roles/states, ≥48dp/44pt, contrast, dynamic type |
| **Diagnostics** | ×3 | **= 10** | Zero errors, zero warnings, clean console |
| **Code quality** | ×2 | ≥ 8 | Token-driven, reusable, idiomatic, matches project conventions |

Weights reflect impact: hierarchy, restraint, premium feel, a11y, and diagnostics matter most.

---

## Scoring rubric (per dimension)

- **10** — would ship at a top studio; nothing to improve.
- **8–9** — premium; minor taste-level nits only.
- **6–7** — competent but generic; clear improvements available → **revise**.
- **4–5** — noticeably off (flat hierarchy, slop tells, missing states) → **revise**.
- **0–3** — broken (errors, no hierarchy, inaccessible) → **stop and rebuild that part**.

Accessibility and Diagnostics are pass/fail at 10 — anything less is an automatic revise, regardless of how pretty the screen is.

---

## How to use it

1. After build + polish + diagnostics, **score each dimension** with a one-line justification (no inflating).
2. **Any dimension below its threshold → revise that specific dimension** (use the matching reference: hierarchy → `visual-hierarchy.md`, premium feel → `premium-polish.md`, etc.), then re-score just that one.
3. Repeat until all thresholds pass.
4. Report a short scorecard to the user so quality is transparent.

---

## Report format (show the user)

```
Premium Scorecard — <screen name>
  Hierarchy          9/10   one clear focal point, clean P/S/T
  Restraint          8/10   removed 2 decorative icons + a redundant card
  Spacing & density  9/10   compact-premium; hero isolated, list tight
  Premium feel       8/10   off-black surfaces, soft layered shadows, tabular figures
  Motion & feedback  8/10   press scale + selection haptic; sheet snaps
  Platform fit       9/10   native bottom sheet detents, swipe-back intact
  States            10/10   empty + skeleton + error all handled
  Accessibility     10/10   labels/roles, 48dp targets, AA contrast, dynamic type
  Diagnostics       10/10   compiles clean, lint zero, console clean
  Code quality       9/10   token-driven, reusable composables
  ─────────────────────────
  Weighted: 92/100  →  PREMIUM ✓
  Notes: gradient on the balance card is a taste choice; can flatten if preferred.
```

Compute weighted total if useful (sum of score×weight ÷ max). Thresholds: **< 80 → revise**, **80–89 → good, polish the lowest dims**, **90+ → premium**. But a single sub-threshold critical dimension (a11y/diagnostics < 10) fails regardless of total.

---

## Evidence is mandatory (anti-gaming)

A score without evidence is invalid — default it to the failing range. For **every** dimension, cite specific elements:
```
Hierarchy 9/10 — evidence: only the "Reserve" CTA uses accent; title 22pt dominates; time muted 38%.
Premium feel 6/10 — evidence: surfaces are pure #FFF (slop tell); text is one flat gray; hard 0.5 shadow on cards. → revise.
```
Adjectives ("clean", "nice") are not evidence. If you can't point at the pixels, you can't claim the score.

Run the `design-critique-engine.md` anti-gaming sequence **before** scoring: forced-failure search (5 reasons not to ship) → adversarial review (designer vs staff engineer) → fix → then score with evidence. Scoring before that sequence tends to inflate.

## Don't game it

- Score what's *actually there*, not what you intended — with evidence per dimension.
- If unsure between two scores, pick the lower and improve.
- A high total with a failing critical dimension (a11y/diagnostics < 10) is **not** a pass.
- If every dimension is 8+ on the first try with no revisions, you under-critiqued — re-run forced-failure search.
- The goal isn't the number — it's the revisions the number forces.
