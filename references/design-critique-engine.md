# Design Critique Engine — self-review before you ship

Building the screen is half the job. The other half is **looking at it as a senior designer would and being honest about what's wrong.** This is the step AI skips — it generates, declares done, and moves on. Instead: critique your own output, find the weaknesses, and **revise until it passes.** This runs after the polish + diagnostics passes, as the final gate.

> Mindset: you are not defending your work — you are trying to *break* it. Assume something is off and go find it. The first version is a draft.

---

## The ship-test questions

Ask these out loud about the screen you just built. A single honest "no" means **REVISE**, not ship.

1. **Would Linear ship this?** (precision, restraint, hierarchy)
2. **Would Revolut ship this?** (premium feel, depth, confident numerics)
3. **Would Apple accept this?** (HIG, accessibility, native conventions)
4. **At 0.5s, is the primary thing obvious?** (hierarchy)
5. **Did I do too much?** (restraint — can I remove anything?)
6. **Does it work in dark mode, at large text, on a small screen, with long content?** (robustness)
7. **Is there any error/warning/stray log?** (diagnostics)
8. **Does it feel generic — could this be any app's default kit?** (the slop test)

If you can't confidently say yes to all, you're not done.

---

## Structured critique (go dimension by dimension)

For each, state what's wrong (be specific, cite the element) → then fix. Don't be vague ("looks fine"); name the defect.

### Hierarchy
- Is there one clear focal point? Two competing primaries? Flat equal weights?
- Does metadata recede, or is it as loud as content?
- *Fix:* re-rank P/S/T, apply opacity ramp, demote extra primaries.

### Restraint
- What can I delete and lose nothing? (decorative icons, redundant labels, extra cards, badges)
- Too many colors? More than one accent?
- *Fix:* remove. The fix for "busy" is subtraction.

### Density & spacing
- Even gaps everywhere? Floaty/empty? Wrong density for the job?
- Are related items actually grouped (tight) and unrelated separated (wide)?
- *Fix:* hierarchical spacing, group with sections, vary item weight.

### Premium feel
- Pure black/white? Flat single gray text? Hard shadows? Default radius?
- Any AI-slop tell from `anti-patterns.md`?
- *Fix:* off-black/white, text ramp, soft layered shadows, optical tracking.

### Motion & feedback
- Do taps respond (scale/opacity)? Haptics on meaningful actions? State changes animated?
- Anything slow/janky/bouncy-for-no-reason?
- *Fix:* add press states + haptics; tune to 150–300ms springs; Reduce Motion.

### Platform fit
- Right component for the OS? FAB on iOS? Material forced on iOS? Back/swipe works?
- *Fix:* adapt per platform.

### States
- Empty / loading (skeleton) / error implemented? Disabled/selected states?
- *Fix:* add the missing states.

### Accessibility
- Labels/roles/states on interactives? Targets ≥48dp/44pt? Contrast passes? Text scales?
- *Fix:* wire a11y props; expand hit areas.

### Diagnostics
- Compiles clean? Lint/analyze zero? Console clean?
- *Fix:* resolve every error and warning at the source.

---

## The REVISE loop

```
BUILD → CRITIQUE (find defects) → REVISE (fix them) → CRITIQUE again
```

- Run at least **one** full critique-and-revise cycle on every non-trivial screen. The first pass almost always finds 2–4 real issues.
- Keep looping until a critique pass finds **nothing material** — or you've made it measurably better and remaining items are genuine taste trade-offs (state them).
- Pair with `premium-scorecard.md`: score the screen; any dimension below threshold → revise that dimension specifically, then re-score.
- Don't loop forever on diminishing returns. Two solid cycles + a clean scorecard is the practical bar.

---

## Anti-gaming mechanisms (so self-review isn't a student grading their own exam)

A model reviewing itself tends to rubber-stamp ("Hierarchy 9, Premium 10") when the real screen is a 6. These three mechanisms force honesty. Use all three on non-trivial screens.

### 1. Evidence-based scoring (no evidence → can't score)
Every score must cite **concrete, specific evidence** pointing at actual elements — not adjectives. Format:
```
Hierarchy: 8/10
  Evidence:
    - Primary CTA "Reserve" is filled-accent, full-width, pinned bottom (only accent on screen)
    - Title 22pt semibold dominates the first viewport
    - Timestamp muted to ~38% — recedes
  Missing / weak:
    - The price chip also uses accent → mild competition with the CTA
```
If you cannot produce evidence for a score, **you cannot give that score** — go look harder or score lower. "Looks good" is not evidence.

### 2. Forced failure search (assume it's broken)
Before scoring, run this explicitly:
```
Find 5 concrete reasons NOT to ship this screen.
→ Fix them.
→ Re-score.
```
The rule forces the model to find ≥5 real defects even when it "feels done" — there are almost always 5. No vague reasons; each must name an element and a fix.

### 3. Adversarial review (two personas must agree)
Critique from two opposed roles, surface their disagreement, then resolve it:
```
DESIGNER persona     → "Too crowded; the secondary actions fight the CTA; spacing is uneven."
STAFF ENGINEER persona → "Too many bespoke components; this won't scale; a11y labels missing on the icon row."
RESOLVE → reconcile both sets of objections into concrete fixes, then re-critique.
```
The designer hunts visual/UX flaws; the engineer hunts structural/scalability/a11y flaws. A screen only passes when **both** are satisfied. This catches what a single viewpoint misses.

> Order: forced-failure search → fix → adversarial review → fix → evidence-based scorecard. Only then ship.

## Honesty rules

- **Name specifics**, not "looks good." "The timestamp is the same weight as the title — demote it" beats "nice hierarchy."
- **Default to finding problems.** If a critique pass finds nothing on the first build, you probably didn't look hard enough — re-read against `anti-patterns.md`.
- **Cite the reference.** "Linear would mute this to ~38%." "Apple HIG wants a 44pt target here."
- **Separate facts from taste.** Errors/a11y/contrast are non-negotiable; some spacing/color choices are taste — flag those as choices, fix the rest.

---

## Critique gate (must pass to ship)

- [ ] Ran ≥1 full critique → revise cycle.
- [ ] All 8 ship-test questions answered "yes" honestly.
- [ ] Every dimension critiqued with specific defects named and fixed.
- [ ] Scorecard run; no dimension below threshold (`premium-scorecard.md`).
- [ ] Remaining items are explicit taste trade-offs, stated — not unaddressed defects.
