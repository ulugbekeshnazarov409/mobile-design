# Product Maturity — build for the stage the product is actually at

A classic AI mistake: asked for an MVP, it ships Airbnb-level complexity — tab bars with five sections, filters, settings sub-menus, power-user features nobody needs yet. Or the reverse: a scaling product gets a toy UI. **Match the UI complexity to the product's stage.** The right design for Stage 0 is *wrong* for Stage 3 and vice-versa.

> Before building, infer or ask the stage. When unstated, assume the **earliest plausible stage** — it's cheaper to add than to remove, and simplicity is the safer default.

---

## The four stages

### Stage 0 — Pre-product-market-fit (MVP)
**Goal:** prove the core value, fast. **Design:** ruthless simplicity.
- One core flow, one primary CTA per screen. Minimal navigation (maybe no tab bar — a single stack).
- Skip: settings sprawl, customization, onboarding tours, empty dashboards, "nice to have" sections.
- Fast to use, obvious, nothing to learn. Hardcode/defer what isn't core.
- Anti-goal: looking "complete." Looking *focused* is the win.

### Stage 1 — Growth
**Goal:** acquire + retain more users. **Design:** more discovery & structure.
- Real navigation (tab bar / sections), search, richer feed/discovery.
- Onboarding that explains value; analytics/usage surfaces; notifications.
- Profile/settings appear properly. More content types, still focused.

### Stage 2 — Scale
**Goal:** serve a large, varied user base. **Design:** depth & power.
- Power-user features, customization/preferences, advanced workflows, shortcuts.
- Denser information where pros want it; bulk actions; filters/sorting; saved views.
- Performance + consistency at volume matter; design-system maturity.

### Stage 3 — Enterprise
**Goal:** organizations, teams, compliance. **Design:** control & governance.
- Roles & permissions, admin panels, audit logs, team management, SSO.
- Configurability, data export, granular settings, multi-account.
- Trust/compliance signals; conservative, robust patterns over flashy ones.

---

## Stage → UI complexity (quick map)

| | Nav | Per-screen actions | Settings | Customization | Density |
| --- | --- | --- | --- | --- | --- |
| **0 MVP** | minimal / stack | one CTA | almost none | none | airy, simple |
| **1 Growth** | tab bar / sections | a few | basic | minimal | medium |
| **2 Scale** | rich, nested | many, power tools | extensive | yes | can be dense |
| **3 Enterprise** | role-based, admin | workflows | granular + admin | high | dense, structured |

---

## How to detect / decide the stage

- **Signals:** "MVP / prototype / launch fast / validate" → Stage 0. "Add search/onboarding/discovery" → Stage 1. "power users / customization / advanced" → Stage 2. "teams / roles / admin / audit / SSO / compliance" → Stage 3.
- **Repo signals:** tiny codebase / few screens → early; auth+roles, admin modules, feature flags → later.
- **When unstated:** state your assumption in the plan ("assuming Stage 0 MVP — one flow, minimal nav") so the user can correct it. Don't silently over-build.

---

## The rules

- **Don't ship Stage-3 complexity to a Stage-0 product.** No five-tab nav, settings labyrinth, or customization for an MVP.
- **Don't ship a toy to a scaling product.** A Stage-2/3 app needs real structure, power features, and density.
- **You can grow into it:** design Stage-0 so it can *extend* (clean tokens/components) without designing all of Stage-3 now.
- **Complexity is a cost**, not a feature. Every added section/option is something to learn, maintain, and get wrong. Add it when the stage demands it.

---

## Maturity checklist

- [ ] Stage inferred (or asked); assumption stated in the plan.
- [ ] UI complexity matches the stage — no over-building an MVP, no under-building a scaling app.
- [ ] Navigation depth, settings, customization, and density appropriate to the stage.
- [ ] Built to extend cleanly (tokens/components) without front-loading later-stage complexity.
- [ ] Every section/option earns its place at this stage.
