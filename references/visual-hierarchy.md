# Visual Hierarchy — the #1 thing AI gets wrong

AI writes good components and bad screens. The reason is almost always **hierarchy**: everything has the same weight, so the eye has nowhere to land. Premium design isn't "more polish on each element" — it's deciding *what the user sees first, second, and what should nearly disappear*, then making the pixels obey that.

Run this **before** styling and again as a check after building.

---

## Stop thinking in "H1 / H2 / Body"

That's a document model, not a screen model. Replace it with three questions:

1. **What should the user see FIRST?** (the one thing — primary action or key content)
2. **What should they see SECOND?** (supporting context)
3. **What should DISAPPEAR into the background?** (metadata, chrome, the rest)

Every element gets ranked: **Primary / Secondary / Tertiary**. If two things are both "primary", you haven't decided yet — decide.

---

## The 3-second rule

A good screen reveals itself in layers over time:

```
0.5 sec  →  Primary action / the hero is obvious
1 sec    →  Screen purpose is clear ("this is my balance", "this is a chat")
3 sec    →  Full understanding without effort
```

If at 0.5s the eye doesn't snap to one thing, the hierarchy is flat. Fix it before anything else.

---

## The tools that create hierarchy (in order of power)

You rank elements using these — usually **combine 2–3**, don't rely on size alone:

1. **Size** — bigger = more important. The hero number, the title.
2. **Weight** — bold pulls forward; regular recedes. Often better than size for subtle rank.
3. **Color / contrast** — full-contrast text = primary; muted (60%) = secondary; faint (38%) = tertiary. The single most underused tool.
4. **Spacing / isolation** — whitespace around an element elevates it. Crowded = less important.
5. **Position** — top and thumb-zone bottom are high-attention; middle-edges recede.
6. **Accent color** — reserve it for the ONE primary action. The moment two things use the accent, neither is primary.

> Mantra: **one primary, muted secondary, hidden tertiary.** Differentiate by *weight + color* first, size second.

---

## The text opacity ramp (copy this)

A flat single-gray text is an instant AI tell. Use a ramp:

| Rank | Light mode | Dark mode | Use |
| --- | --- | --- | --- |
| Primary | ~87–100% ink | ~87–100% white | titles, key content |
| Secondary | ~60% | ~60% | supporting text, labels |
| Tertiary | ~38–45% | ~38–45% | metadata, timestamps, hints |
| Disabled | ~24–30% | ~24–30% | inactive |

(Map to your theme's `onSurface` / `onSurfaceVariant` / `.secondary` / `.tertiary` roles — don't hardcode opacity if the system provides roles.)

---

## One-primary-action rule

- Each screen has **exactly one** primary CTA (filled, accent, full-width, thumb zone).
- Secondary actions are outlined/tonal/text — visibly lower.
- Tertiary actions are quiet text links or icons.
- Two filled accent buttons = no primary. Demote one.

Same for navigation: one clear "what do I do here", everything else supports it.

---

## Hierarchy per pattern (quick reference)

- **Feed/list:** the item's title is primary; thumbnail supports; author/time/stats are tertiary (muted, small).
- **Detail:** hero image + title primary; the buy/book CTA is the action-primary (pinned bottom); description secondary; meta tertiary.
- **Chat:** message text primary; sender (in groups) secondary; time/ticks tertiary (faint).
- **Profile:** name + avatar primary; stats secondary (count bold, label muted); actions secondary.
- **Settings:** row label primary; value/icon secondary; section header tertiary (uppercase muted); destructive isolated.
- **Balance/finance:** the number is the hero — biggest, boldest, tabular; label above it tiny/muted; actions below.

---

## Common hierarchy failures → fix

| Failure | Fix |
| --- | --- |
| Everything same size/weight/color | Rank into P/S/T; apply the opacity ramp + weight |
| Two primary buttons | Keep one filled-accent; demote the other to outlined/text |
| Accent color on many elements | Accent only the one primary thing; neutral the rest |
| Metadata as loud as content | Mute timestamps/counts to tertiary, shrink them |
| No focal point (flat grid of equal cards) | Make one item/section dominant; vary density |
| Title and body indistinguishable | Increase title weight + size, mute body color |

---

## Hierarchy check (run after building)

- [ ] At 0.5s, one element clearly wins the eye.
- [ ] Exactly one primary action; secondary/tertiary visibly lower.
- [ ] Text uses the P/S/T opacity ramp, not one flat gray.
- [ ] Accent color appears on the primary thing only.
- [ ] Metadata/chrome recedes; content leads.
- [ ] Spacing isolates what matters; crowding de-emphasizes the rest.

If any fails, fix hierarchy before adding polish — polish on a flat screen is lipstick.
