# Charts & Data Viz — premium, not default

AI charts ship the library's demo: a full gridded box, four garish auto-colors, every axis tick labeled, a legend nobody reads, and no thought for what happens when the array is empty. Premium charts (Revolut, Coinbase, Apple Health) do the opposite — they *erase* ink until only the data and one or two labels remain, use **one accent on neutrals**, format the money axis in the user's locale, animate the line in, and have real empty/loading/error states. This file makes charts read like a designed product feature, not a plotted dataset.

> Cross-ref: `color-systems.md` (one accent + tinted neutrals), `typography-systems.md` (tabular figures), `skia-and-graphics.md` (custom rendering), `design-tokens-starter.md` (chart tokens), `locale-uz.md` (UZS axis), `motion-recipes.md` (animate-in), `performance-and-quality.md` (60fps, no per-frame re-render).

---

## 1. Chart design principles — maximize data-ink

Tufte's rule: every pixel that isn't data is a candidate for deletion. AI charts are mostly chartjunk. Strip it:

| Kill | Keep / replace with |
| --- | --- |
| Full gridline mesh | 2–3 faint horizontal references (`accent-border`/neutral at ~8–12%), no verticals |
| Chart border / plot box | nothing — let the chart breathe into the card |
| 3D, shadows on bars, bevels | flat fills; depth comes from the card, not the bar |
| Legend (when ≤2 series) | label the line ends directly, or rely on the section title |
| Every axis tick labeled | first / last / min / max / "now" only — label what matters |
| Rainbow auto-palette | **one accent** for the primary series, neutrals for the rest |
| Y axis starting at 0 always | for *trends* (sparklines) zoom to the data range; keep 0-baseline for **bars/volume** (truncating bars lies) |
| Dense numeric ticks | format with locale + tabular figures; abbreviate (`1.2M`, `25K so'm`) |

Core moves: **one accent + neutrals** (never green AND blue AND orange unless each encodes meaning — finance green/red = gain/loss only, per `color-systems.md`), **tabular figures on every axis and tooltip** (`typography-systems.md`), padding so end labels aren't clipped, and an **animate-in** on first render (§6). Every chart needs **empty / loading / error** states (§5) — "0 transactions this month" is the common real state AI forgets.

---

## 2. Library selection (React Native)

| Library | Renderer | Use when | Avoid when |
| --- | --- | --- | --- |
| **victory-native (XL, v40+)** | Skia (`@shopify/react-native-skia`) | line / area / bar / scatter with many points, smooth pan & scrub, you want a real chart API + 60fps | tiny inline sparkline (overkill); you need candlesticks out of the box (build via Skia) |
| **react-native-gifted-charts** | `react-native-svg` | fast to ship line/bar/pie/donut, built-in pointer/tooltip, decent defaults you'll still need to tame | thousands of points (SVG node count hurts); fully bespoke shapes |
| **raw `@shopify/react-native-skia`** | Skia | bespoke visuals — candlesticks, custom gauges, gradient area with glow, animated scrub line at 60fps | a standard bar chart (don't reinvent; cost = time) |
| **`react-native-svg` (hand-rolled `Path`)** | SVG | small static sparkline, progress ring, simple shapes, no extra dep | interaction-heavy or large datasets |

Rule of thumb: **sparkline → svg path or Skia**, **standard chart fast → gifted-charts**, **standard chart smooth/scalable → victory-native-xl**, **bespoke/financial → raw Skia**. victory-native-xl and Skia both need the Skia package; on Expo use a dev client (not plain Expo Go).

### Other platforms

| Platform | Default | Notes |
| --- | --- | --- |
| **SwiftUI** | **Swift Charts** (`import Charts`, iOS 16+) | `Chart { LineMark/BarMark/AreaMark/SectorMark }`; native, themable, accessible. `.chartXAxis`/`.chartYAxis` to strip ticks; `.foregroundStyle(.tint)`. First choice on iOS. |
| **Compose** | **Vico** (`com.patrykandpatrick.vico`) or raw `Canvas` | Vico = batteries-included M3-friendly charts. For bespoke, draw on `Canvas`/`drawScope` (paths, gradients). `androidx.graphics.shapes` for shapes. |
| **Flutter** | **fl_chart** | `LineChart`/`BarChart`/`PieChart` with `LineChartData`; turn off `gridData`/`borderData`, set `lineTouchData` for tooltips. `syncfusion_flutter_charts` if you need many chart types. |

```kotlin
// Compose — Vico line chart, gridless, single accent
CartesianChartHost(
  chart = rememberCartesianChart(
    rememberLineCartesianLayer(/* line color = colorScheme.primary */),
    startAxis = rememberStartAxis(guideline = null),   // no gridlines
    bottomAxis = rememberBottomAxis(guideline = null),
  ),
  modelProducer = producer,
)
```
```swift
// SwiftUI — Swift Charts, stripped chrome
Chart(points) { p in
  LineMark(x: .value("Day", p.date), y: .value("Balance", p.amount))
    .interpolationMethod(.monotone)
    .foregroundStyle(.tint)
}
.chartYAxis { AxisMarks(values: .automatic(desiredCount: 3)) { AxisValueLabel() } } // no grid lines
.chartXAxis(.hidden)
```

---

## 3. Sparkline (balance trend) — line + soft area

The hero of any FinTech home: a clean trend line with a faint gradient fill, no axes, one accent. Restraint on the gradient is the whole game — it should *fade to nothing*, not be a solid block.

```tsx
// victory-native-xl — balance sparkline with restrained gradient area
import { CartesianChart, Line, Area } from 'victory-native';
import { LinearGradient, vec } from '@shopify/react-native-skia';
import { useFont } from '@shopify/react-native-skia';

function BalanceTrend({ data }: { data: { x: number; y: number }[] }) {
  if (!data.length) return <ChartEmpty />;            // §5 — never render an empty plot
  return (
    <CartesianChart
      data={data}
      xKey="x"
      yKey={['y']}
      domainPadding={{ top: 24, bottom: 8 }}           // headroom so the peak isn't clipped
      // no axisOptions => no gridlines, no ticks: pure trend
    >
      {({ points, chartBounds }) => (
        <>
          <Area
            points={points.y}
            y0={chartBounds.bottom}
            animate={{ type: 'timing', duration: 500 }}
            curveType="natural"
          >
            <LinearGradient
              start={vec(0, 0)}
              end={vec(0, chartBounds.bottom)}
              colors={[tokens.accentAlpha(0.22), tokens.accentAlpha(0)]} // fade to transparent
            />
          </Area>
          <Line
            points={points.y}
            color={tokens.accent}
            strokeWidth={2.5}
            curveType="natural"
            animate={{ type: 'timing', duration: 500 }}
          />
        </>
      )}
    </CartesianChart>
  );
}
```

Gradient restraint: **one** accent fading to transparent, ~20% top opacity max. No second hue, no hard band. The line is `2–2.5px`, rounded caps, `curveType="natural"`/monotone (never jagged linear for a balance trend). For a static mini-sparkline in a list row, skip the library entirely — build a `react-native-svg` `<Path>` from the points (cheapest possible).

---

## 4. Bar chart — spending by category

```tsx
// victory-native-xl — bars, zero-baseline, single accent, minimal axis
import { CartesianChart, Bar } from 'victory-native';

<CartesianChart
  data={months}
  xKey="label"
  yKey={['amount']}
  domainPadding={{ left: 24, right: 24, top: 16 }}
  axisOptions={{
    font,
    lineColor: 'transparent',                 // kill axis lines
    tickCount: { x: months.length, y: 3 },    // 3 y refs, not a mesh
    formatYLabel: (v) => fmtUzsShort(v),       // "25K so'm" — see locale-uz.md
    formatXLabel: (l) => l,                    // month initials only
    labelColor: tokens.textTertiary,
  }}
>
  {({ points, chartBounds }) => (
    <Bar
      points={points.amount}
      chartBounds={chartBounds}
      color={tokens.accent}
      roundedCorners={{ topLeft: 6, topRight: 6 }}   // soft top, flat base
      barWidth={18}
      animate={{ type: 'timing', duration: 450 }}
    />
  )}
</CartesianChart>
```

Bars stay zero-baselined (truncating bars misleads). One accent fill; gray the rest. Highlight a single bar (the selected/current month) by giving it `accent` and the others `accentAlpha(0.25)` — selection through color weight, not a new hue. Currency axis abbreviated and locale-formatted (`fmtUzsShort` → `locale-uz.md`).

---

## 5. Empty / loading / error states (do not skip)

A chart with no data is the most common real state and the one AI forgets.

```tsx
function ChartEmpty() {
  return (
    <View style={styles.chartState}>
      <SparkIcon color={tokens.textTertiary} />
      <Text style={styles.stateTitle}>No activity yet</Text>
      <Text style={styles.stateSub}>Your spending trend appears here after your first transaction.</Text>
    </View>
  );
}
// Loading: a shimmer placeholder shaped like the chart (a faint baseline + skeleton bars),
//   NOT a centered spinner over blank space.
// Error: short message + Retry, same footprint so layout doesn't jump.
```

Rules: keep the **same height/footprint** across loading → empty → error → loaded so the card never reflows (`performance-and-quality.md`). Skeleton mimics the chart's silhouette (ghost line/bars), not a spinner. Empty copy says what unlocks the chart, not "No data".

---

## 6. Interaction — scrub, tooltip, haptic

Touch-to-scrub is what separates a premium chart from a static image. victory-native-xl ships `useChartPressState`; pair the active point with a moving indicator, a value readout, and a **haptic tick when crossing data points**.

```tsx
import { CartesianChart, Line, useChartPressState } from 'victory-native';
import { Circle } from '@shopify/react-native-skia';
import * as Haptics from 'expo-haptics';

function ScrubbableTrend({ data }) {
  const { state, isActive } = useChartPressState({ x: 0, y: { y: 0 } });
  const lastIdx = useSharedValue(-1);

  useAnimatedReaction(
    () => state.x.position.value,
    () => {
      const i = Math.round(state.x.value.value);
      if (i !== lastIdx.value) { lastIdx.value = i; runOnJS(Haptics.selectionAsync)(); } // tick per point
    },
  );

  return (
    <CartesianChart data={data} xKey="x" yKey={['y']} chartPressState={state}>
      {({ points }) => (
        <>
          <Line points={points.y} color={tokens.accent} strokeWidth={2.5} curveType="natural" />
          {isActive && (
            <Circle cx={state.x.position} cy={state.y.y.position} r={6} color={tokens.accent} />
          )}
        </>
      )}
    </CartesianChart>
  );
}
```

Above the chart, render the **selected value + date** in the header (tabular figures), updating live as the finger moves — like Coinbase/Apple Stocks. The header is the tooltip; a floating bubble is optional and often noisier. Use `Haptics.selectionAsync()` (light tick), not `notification` (too heavy). On release, animate back to the latest value. See `motion-recipes.md` for the spring on the indicator and `gestures-and-haptics.md` for haptic intensity.

---

## 7. Donut / progress ring & candlesticks

**Donut (budget split):** keep it to ≤4 slices — beyond that it's unreadable; use a stacked bar instead. One accent for the focus slice, tinted neutrals for the rest, a gap between slices, and the **key number in the center hole** (e.g. `68%` or remaining balance) — the hole is prime real estate, never leave it empty.

```tsx
// Progress ring with react-native-svg — single arc, rounded cap
import Svg, { Circle } from 'react-native-svg';
const R = 54, C = 2 * Math.PI * R;
<Svg width={120} height={120}>
  <Circle cx={60} cy={60} r={R} stroke={tokens.trackNeutral} strokeWidth={10} fill="none" />
  <Circle cx={60} cy={60} r={R} stroke={tokens.accent} strokeWidth={10} fill="none"
    strokeDasharray={C} strokeDashoffset={C * (1 - pct)} strokeLinecap="round"
    transform="rotate(-90 60 60)" />
</Svg>
```
Animate `strokeDashoffset` from full→target with Reanimated for the fill-in. Center the percentage/value with tabular figures.

**Candlesticks (crypto):** no RN library renders these cleanly out of the box — draw with **raw Skia** (`@shopify/react-native-skia`). For each candle: a thin `Rect` (wick, high→low) plus a wider `Rect` (body, open→close); color **green up / red down** per `color-systems.md` functional colors (and add a shape/position cue so it's not color-alone — see §9). Batch all candles in one Skia canvas; never one `<View>` per candle. For heavy OHLC datasets, decimate to the visible window and downsample on zoom-out (§8).

---

## 8. Performance — 60fps, no per-frame React re-render

- **Renderer choice = the perf decision.** Skia (victory-native-xl, raw Skia) and native (Swift Charts, Vico) draw on the GPU/native side — smooth with hundreds–thousands of points. SVG (gifted-charts, react-native-svg) creates a DOM node per element; keep SVG charts to small point counts.
- **Drive animation on the UI thread.** Scrub indicators and draw-in use Reanimated shared values / Skia clocks — **never** `setState` per finger move or per frame (that re-renders the React tree 60×/sec and drops frames). See `performance-and-quality.md`.
- **Downsample large series** to roughly the pixel width before plotting (LTTB or simple bucketing) — 5,000 points on a 360px-wide chart is wasted work; ~360 visually identical points render free.
- **Memoize** the data array and chart config (`useMemo`) so an unrelated parent re-render doesn't rebuild paths. Keep the formatter functions stable.
- Avoid live-updating a chart on every websocket tick — coalesce to ~1–4 fps for the visual, or animate the last point only.

---

## 9. Accessibility — never color-alone, always a summary

Charts are invisible to screen readers unless you give them a text equivalent and don't lean on color alone (`accessibility-deep.md`, `ui-restraint-and-accessibility.md`).

- **Accessible summary:** wrap the chart in an `accessible` view with an `accessibilityLabel` that states the takeaway: *"Balance trend, last 30 days, up 12 percent, current 4 200 000 so'm."* The screen reader user gets the insight, not "chart".
- **Don't rely on color alone:** gain/loss also gets an arrow/sign (`+`/`−`, ▲/▼); series get direct labels or patterns, not just hue; selected bar differs in weight, not only color. Verify legibility for color-blind users.
- **Provide the numbers too:** pair the chart with the actual figure (current value, % change) in text — the chart is reinforcement, the number is the source of truth.
- Swift Charts exposes `.accessibilityLabel`/`.accessibilityValue` per mark and an audio-graph; fl_chart and Compose need an explicit `contentDescription`/`accessibilityLabel` on the container.
- Respect **reduce-motion**: skip or shorten the draw-in animation when the OS flag is set.

---

## Quality checklist (charts)

- [ ] Chartjunk removed — no plot box, no full gridline mesh, no 3D/shadows; at most 2–3 faint y references, no verticals.
- [ ] One accent + tinted neutrals; finance green/red = gain/loss only; no rainbow auto-palette.
- [ ] Axes label only what matters (first/last/min/max/now), abbreviated and locale-formatted (`25K so'm`), tabular figures everywhere.
- [ ] Sparkline gradient fades to transparent (~≤20% top), single hue; line 2–2.5px, smooth curve, rounded caps.
- [ ] Bars zero-baselined; trends may zoom the range; donut ≤4 slices with a value in the center hole.
- [ ] Empty / loading / error states built; same footprint across all; skeleton mimics the chart silhouette, not a spinner.
- [ ] Touch scrub with live value-in-header, selected indicator, selection-haptic per data point.
- [ ] Animate-in on first render; reduce-motion respected.
- [ ] Skia/native renderer for large or interactive datasets; animation on the UI thread, no per-frame `setState`; data/series downsampled and memoized.
- [ ] Accessible summary label states the takeaway; never color-alone (sign/arrow/label added); real numbers shown in text alongside.
