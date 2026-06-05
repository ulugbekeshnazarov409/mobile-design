# Screen Density — real apps aren't evenly spaced

The default AI layout is: card → big gap → card → big gap → button. Uniform, airy, lifeless. Real apps choose a **density** that fits their purpose and hold it with rhythm. A chat is dense; a paywall is sparse; a banking dashboard is compact-premium. Pick the right density, then make spacing *deliberate*, not evenly sprinkled.

---

## The density spectrum

| Density | Feel | Row/section spacing | Use / examples |
| --- | --- | --- | --- |
| **Very dense** | fast, information-rich | 4–8 between items, tight rows (44–52) | chat, message lists, tables — *Telegram* |
| **Compact-premium** | confident, efficient | 8–12 items, hero gets room | finance dashboards, pro tools — *Revolut, Linear* |
| **Medium** | balanced, comfortable | 12–16 items, 24 sections | feeds, marketplaces — *Airbnb, Coinbase* |
| **Airy** | calm, focused | 16–24 items, 32 sections | onboarding, paywalls, settings — *Notion, Duolingo* |

Decide density **first**, from the screen's job:
- Lots of scannable items / frequent use → denser.
- One decision / emotional moment / first-run → airier.
- Money/hero number → compact-premium (dense context, isolated hero).

---

## Spacing is rhythm, not padding

Uniform gaps are the tell. Use a **hierarchy of spacing**:

```
4–8    inside a unit (icon↔label, title↔subtitle)      ← tight, "these belong together"
12     between related items in a group
16     standard card padding / screen margin
24     between groups
32     between major sections
```

Rule: **related = tight, unrelated = wide.** A title and its subtitle sit 4 apart; two unrelated cards sit 24 apart. When everything is 16, the eye can't tell what groups with what.

> Proximity is meaning. Things close together read as related; the gap *is* the grouping.

---

## Density recipes by pattern

**Chat (very dense)**
- Row/message vertical padding 6–8; group consecutive messages (tighten to ~2, show avatar/time once).
- List rows 56-ish but content-tight; separators thin or none.

**Finance dashboard (compact-premium)**
- Hero balance gets generous isolation (24–32 around it).
- Below: dense rows of transactions/actions (8–12 apart). Contrast of airy-hero + dense-list = premium.

**Feed (medium)**
- Card padding 16; 12–16 between cards; section headers with 24 above.
- Vary item density: not every item a giant equal card — mix compact rows with occasional feature cards.

**Settings (airy-ish, grouped)**
- Rows 48–56 tall, 0 gap *within* a group (use separators), 24–32 *between* groups. Section header muted above each group.

**Onboarding / paywall (airy)**
- Big hero, generous vertical breathing (32+), one CTA pinned bottom. Don't fill the space — emptiness is intentional focus.

---

## Anti-uniformity moves

- **Vary item weight:** combine compact rows + occasional larger feature cards instead of N identical cards.
- **Group with sections**, not equal gaps — labeled sections create structure denser/cleaner than floating cards.
- **Let the hero breathe, pack the rest:** isolate the one important thing; tighten the supporting list.
- **Use separators in dense lists** instead of gaps (saves space, stays clean).
- **Edge-to-edge where it fits:** full-bleed images/lists feel more app-like than everything inset in cards.

---

## Don't-over-space checklist

- [ ] Density chosen to match the screen's job (dense vs airy), not defaulted to airy.
- [ ] Spacing is hierarchical (4/8/12/16/24/32), not one repeated gap.
- [ ] Related elements are tight; only unrelated groups are wide.
- [ ] Not every element is a full-width equal-weight card — density varies.
- [ ] Hero isolated; supporting content packed efficiently.
- [ ] Consistent row heights / paddings within a section (rhythm holds).

If the screen feels "floaty" or empty, it's probably over-spaced and under-grouped — tighten related items, add sections, vary density.
