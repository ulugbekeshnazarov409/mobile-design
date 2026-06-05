# Perception & Psychology — design for how the eye and brain actually work

Premium layout isn't taste — it's aligned with how humans scan, group, and decide. Knowing these laws turns "looks off, not sure why" into a fixable diagnosis. Use them to place the important thing where the eye already goes, and to make groupings feel effortless.

---

## 1. Scanning patterns — where the eye goes

- **F-pattern** (text-heavy: feeds, lists, articles): eyes sweep across the top, then down the left, scanning. → Put the most important info **top and left**; front-load labels; left-align scannable content. Don't bury the key line mid-right.
- **Z-pattern** (sparse, focal screens: onboarding, landing, paywalls): eye goes top-left → top-right → diagonally to bottom-left → bottom-right. → Logo/title top, secondary top-right, content middle, **primary CTA bottom-right / bottom**.
- **Layer-cake / scanning lists:** users read headings + first words, skip the rest. → Strong section headers; meaningful first words; truncate predictably.
- **Center-stage** (single focal): the largest, most central element gets attention first. → Make the hero genuinely dominant.

Match the pattern to the screen type; place primary content/action on the eye's natural path, not against it.

---

## 2. Attention anchors — what grabs the eye (in order)

The eye is pulled by, roughly in order of strength:
1. **Faces & people** (then where they look — gaze direction leads the eye).
2. **Motion** (anything moving wins instantly — use sparingly).
3. **High contrast** (a bright element on muted ground).
4. **Color** against neutral (the lone accent).
5. **Size** (the biggest thing).
6. **Isolation** (whitespace around an element).

Use anchors deliberately: the **one** thing you want seen first should own the strongest available anchor (size + accent + isolation). Conversely, don't accidentally anchor a trivial element (a bright badge on a minor row steals attention).

---

## 3. Gestalt grouping — how the brain bundles elements

The brain auto-groups by these; use them instead of boxes/lines:
- **Proximity:** near = related. The single most powerful grouping tool. Tight gaps bind; wide gaps separate. (See `screen-density.md`.)
- **Similarity:** same shape/size/color reads as a set. Style peers identically; differentiate non-peers.
- **Common region:** a shared background/card groups items — but proximity often does it with less visual weight (prefer space over more cards).
- **Continuity:** aligned elements read as connected — a clean grid/edge feels ordered; stray indents break the line and feel broken.
- **Closure:** the mind completes shapes — you can imply structure without drawing every border.
- **Figure/ground:** clear separation of content from background; insufficient contrast makes things feel muddy.

Diagnosis tip: if a layout "feels messy", it's usually a **proximity** or **alignment (continuity)** violation — related things too far apart, or edges not aligned.

---

## 4. Cognitive load — don't make them think

- **Hick's Law:** more choices = slower decisions. → Limit options per screen; one primary action; progressive disclosure over showing everything.
- **Miller's ~7±2 / chunking:** group long info into chunks (phone numbers, steps, sections). Don't present 15 ungrouped items.
- **Recognition over recall:** show options/icons-with-labels rather than making users remember. Persist context across steps.
- **Progressive disclosure:** reveal complexity only when needed (advanced settings behind a tap). First view = the 80% case.
- **Jakob's Law:** users expect your app to work like the apps they know. → Follow platform + category conventions; novelty in *core* navigation costs you.

---

## 5. Fitts's Law & ergonomics — make targets easy to hit

- Time-to-hit grows with distance and shrinks with target size. → **Big targets for frequent/important actions**, placed close to the thumb.
- **Thumb zone:** bottom-center is easiest one-handed; top corners are hardest. Primary actions + nav at the bottom. (Cross-ref `design-to-code.md`.)
- Screen **edges and corners** are "infinite" targets (you can't overshoot) — good for persistent controls.
- Min 48dp/44pt; don't crowd targets.

---

## 6. Perceived performance — feel fast, not just be fast

Perception of speed often matters more than raw speed:
- **Skeletons** make waits feel shorter than spinners (progress feels like motion toward done).
- **Optimistic UI** (act now, reconcile later) feels instant.
- **Instant feedback** on tap (press state + haptic) tells the brain "it heard you" even before work completes.
- **Show progress** for long tasks; **animate transitions** so state changes feel intentional, not laggy.
- Avoid layout shift/jank — jank reads as "broken/slow" regardless of actual speed.

---

## 7. Emotion & trust signals

- **Polish = trust.** Tight alignment, consistent spacing, and crisp type signal "this is made by people who care" — especially for finance/health.
- **Tasteful delight** (a success animation, a satisfying haptic) builds affinity — sparingly, at meaningful moments, never blocking.
- **Honest states** (clear errors, real empty states) build trust more than hiding problems.
- **Restraint reads as premium**; clutter reads as cheap/anxious.

---

## Applying it — quick diagnosis

When a screen feels off, check in this order:
1. **Hierarchy/anchor:** does the strongest anchor point at the right thing? (else re-rank)
2. **Proximity/alignment:** are related items grouped and edges aligned? (most "messy" bugs)
3. **Scan path:** is the key info/action on the F or Z path?
4. **Load:** too many choices/items unchunked? (simplify, group, disclose)
5. **Ergonomics:** are frequent targets big and thumb-reachable?
6. **Perceived speed:** instant feedback, skeletons, no jank?

---

## Psychology checklist

- [ ] Layout matches the natural scan pattern (F for content, Z for focal); key items on the path.
- [ ] The strongest attention anchor points at the intended primary element; no accidental anchors.
- [ ] Grouping uses proximity + alignment (not just boxes); related tight, edges aligned.
- [ ] Choices limited; long info chunked; complexity progressively disclosed.
- [ ] Frequent/important targets are large and in the thumb zone.
- [ ] Feels fast: instant tap feedback, skeletons, optimistic updates, no jank.
