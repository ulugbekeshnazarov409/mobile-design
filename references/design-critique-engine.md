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
