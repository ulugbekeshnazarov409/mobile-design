# Skia & Custom Graphics — signature visuals

Custom canvas graphics (gradient mesh, aurora glow, animated blobs, frosted blur, shaders) are the "wow" that separates top-studio apps from template apps — but only when used for **one signature moment per screen**, never everywhere. Skia is GPU-heavy and adds a native dependency; reach for it deliberately, fall back gracefully on low-end devices, and honor Reduce Motion. This file is the React Native / Expo Skia recipe layer; apply it from the premium-polish pass, not by default.

> If a `View` + `expo-linear-gradient` already gets you there, **stop there**. Skia is for visuals plain views and SVG physically cannot do.

---

## When Skia vs SVG vs plain views

| Need | Reach for | Why |
| --- | --- | --- |
| Static icon, logo, simple shape | `react-native-svg` or a `View` | Declarative, light, no native canvas, no dev build |
| Linear/radial gradient fill, rounded card, ring | `expo-linear-gradient` / SVG | Cheap, works in Expo Go |
| Animated vector path, morphing icon, stroke draw-on | `react-native-svg` + Reanimated | SVG handles modest animation fine |
| **Gradient mesh / aurora background** | **Skia** | Multi-stop blended gradients + blur, no SVG equivalent |
| **Animated blob / glow / liquid** | **Skia** | UI-thread canvas, per-frame paths |
| **Frosted card w/ backdrop blur of canvas content** | **Skia BackdropFilter** | True backdrop sampling |
| **Custom shader (noise, grain, mesh, ripple)** | **Skia RuntimeShader (SkSL)** | Only path to fragment shaders in RN |
| **Procedural / generative / lots of nodes at 60fps** | **Skia** | Retained-mode SVG chokes; Skia draws imperatively |

Decision rule: **static or low-count → SVG/View. Custom gradients, blur, shaders, or per-frame animation → Skia.** Don't pull in Skia to draw a circle.

> **Skia needs a dev build — it does NOT run in Expo Go.** Install `@shopify/react-native-skia`, add its config plugin, and run `eas build` / `expo run:ios|android`. It also bundles a native `.so`/framework → meaningfully larger app size. Budget for that before committing.

---

## Skia basics

```tsx
import { Canvas, RoundedRect, Path, Group, Paint, LinearGradient, vec } from '@shopify/react-native-skia';

<Canvas style={{ width, height }}>
  {/* Filled rounded card with a vertical gradient */}
  <RoundedRect x={0} y={0} width={width} height={160} r={24}>
    <LinearGradient
      start={vec(0, 0)} end={vec(0, 160)}
      colors={['#6D5DF6', '#4B3FD6']}
    />
  </RoundedRect>

  {/* A stroked path (e.g. custom shape / chart line) */}
  <Path path="M0,80 C100,20 200,140 320,80" style="stroke" strokeWidth={4} color="#fff" />
</Canvas>
```

- **Canvas** is the GPU surface. Everything inside it is drawn imperatively each frame.
- **Paint** = how a shape is filled/stroked (color, style, blend, opacity). Set via props (`color`, `style="stroke"`, `strokeWidth`, `opacity`) or a child `<Paint>`. A gradient/shadow/blur child *is* the paint for its parent shape.
- **Group** applies transforms (`transform`, `clip`, `layer`, `blendMode`, `opacity`) to all children — your main composition tool.
- **Gradients:** `LinearGradient`, `RadialGradient` (`c`, `r`), `SweepGradient` (`c`, conic — great for progress rings and glows). Use `positions={[0,0.6,1]}` to bias stops.
- **Rounded shapes / clip:** `RoundedRect`, or `Group clip={rrect(rect(...), rx, ry)}` to mask children to a rounded region.

---

## Premium uses

### Aurora / mesh gradient background
Stack a few large, blurred radial blobs and let them blend — the cheapest "expensive-looking" background.

```tsx
import { Canvas, Fill, Circle, RadialGradient, Blur, Group, vec } from '@shopify/react-native-skia';

<Canvas style={StyleSheet.absoluteFill}>
  <Fill color="#0B0B12" />
  <Group blendMode="screen">
    <Blur blur={60} />
    <Circle c={vec(80, 120)} r={180}>
      <RadialGradient c={vec(80, 120)} r={180} colors={['#7C5CFF', 'transparent']} />
    </Circle>
    <Circle c={vec(300, 360)} r={200}>
      <RadialGradient c={vec(300, 360)} r={200} colors={['#FF5C9A', 'transparent']} />
    </Circle>
  </Group>
</Canvas>
```

### Custom progress ring with glow
`SweepGradient` for the conic sweep; a second blurred copy underneath = glow.

```tsx
import { Canvas, Path, Skia, SweepGradient, Blur, Group, vec } from '@shopify/react-native-skia';

const ring = Skia.Path.Make();
ring.addArc({ x: 12, y: 12, width: 120, height: 120 }, -90, 360 * progress); // progress 0..1

<Canvas style={{ width: 144, height: 144 }}>
  <Group>
    <Path path={ring} style="stroke" strokeWidth={12} strokeCap="round">
      <SweepGradient c={vec(72, 72)} colors={['#5EE7DF', '#7C5CFF', '#5EE7DF']} />
      <Blur blur={8} />{/* glow copy */}
    </Path>
    <Path path={ring} style="stroke" strokeWidth={12} strokeCap="round">
      <SweepGradient c={vec(72, 72)} colors={['#5EE7DF', '#7C5CFF', '#5EE7DF']} />
    </Path>
  </Group>
</Canvas>
```

### Frosted card over canvas content — BackdropFilter + Blur
Blurs whatever Skia drew behind it, inside a clipped region. (For blurring *RN view* content behind glass, use `expo-blur` instead — see glassmorphism-and-materials.md.)

```tsx
import { BackdropFilter, Blur, Fill, rrect, rect } from '@shopify/react-native-skia';

<Group clip={rrect(rect(24, 200, width - 48, 140), 24, 24)}>
  <BackdropFilter filter={<Blur blur={12} />} clip={rrect(rect(24, 200, width - 48, 140), 24, 24)}>
    <Fill color="rgba(255,255,255,0.10)" />{/* tint over the blur */}
  </BackdropFilter>
</Group>
```

### Noise / grain overlay & soft shadow
A faint grain over flat gradients kills banding and adds texture. Use a `RuntimeShader` noise (below) or a tiled image at ~4% opacity. For depth on a shape, add a `<Shadow dx dy blur color inner?>` child to a `RoundedRect`/`Path` (inner shadow = pressed/inset look).

---

## Animation — drive Skia from the UI thread

Skia reads Reanimated **shared values** and `useClock` directly, so animation runs on the UI thread at 60fps with no JS round-trip. Use `useDerivedValue` to compute paths/values per frame.

```tsx
import { Canvas, Path, Skia, useClock } from '@shopify/react-native-skia';
import { useDerivedValue } from 'react-native-reanimated';

const clock = useClock();
const wave = useDerivedValue(() => {
  const t = clock.value / 1000;
  const p = Skia.Path.Make();
  p.moveTo(0, 80);
  for (let x = 0; x <= 320; x += 8) {
    p.lineTo(x, 80 + Math.sin(x * 0.04 + t * 2) * 14);
  }
  return p;
}, [clock]);

<Canvas style={{ width: 320, height: 160 }}>
  <Path path={wave} style="stroke" strokeWidth={3} color="#7C5CFF" />
</Canvas>
```

- Animated **blob:** same pattern — perturb a closed path's control points with `sin(clock + offset)`.
- Coordinate with gesture/spring motion via Reanimated; see motion-recipes.md for the timing/easing tokens to feed in.
- Prefer `useClock` only while the effect is on screen; pause it when off-screen to save battery.

---

## Shaders (SkSL RuntimeShader) — brief

Worth it for effects no amount of layered shapes can do cheaply: **mesh gradients, animated noise/grain, ripple, dissolve, metaball.** Write SkSL (GLSL-like), pass uniforms (time, resolution) per frame.

```tsx
import { Canvas, Fill, Skia, Shader } from '@shopify/react-native-skia';
import { useClock, useDerivedValue } from 'react-native-reanimated';

const source = Skia.RuntimeEffect.Make(`
uniform float u_time;
uniform float2 u_res;
half4 main(float2 xy) {
  float2 p = xy / u_res;
  float n = sin(p.x * 8.0 + u_time) * 0.5 + 0.5;     // animated band
  return half4(mix(half3(0.42,0.33,1.0), half3(1.0,0.36,0.6), n * p.y), 1.0);
}`)!;

const clock = useClock();
const uniforms = useDerivedValue(() => ({ u_time: clock.value / 1000, u_res: [width, height] }), [clock]);

<Canvas style={{ width, height }}>
  <Fill><Shader source={source} uniforms={uniforms} /></Fill>
</Canvas>
```

Keep SkSL short — it runs per pixel per frame. Don't ship a complex shader to low-end Androids without a static fallback.

---

## Performance & restraint

- **One signature canvas per screen.** A Skia canvas is an offscreen GPU surface; several full-screen ones = dropped frames and battery drain.
- **Watch overdraw.** Big blurred/blended layers stacked full-screen are the #1 cost. Shrink the canvas to the region that needs it; don't wrap the whole tree.
- **Offscreen cost.** `BackdropFilter`, `Blur`, and `layer` Groups force extra render passes — use one, not five.
- **Low-end fallback.** Gate the fancy version behind a capability/perf check; render a flat `expo-linear-gradient` background otherwise. Banding-free flat gradient > janky shader.
- **Honor Reduce Motion.** `AccessibilityInfo.isReduceMotionEnabled()` → freeze the clock (render a static frame), don't animate. See ui-restraint-and-accessibility.md.
- For glass specifically, prefer native `expo-blur` (glassmorphism-and-materials.md) — only use Skia `BackdropFilter` when blurring *Skia-drawn* content. See performance-and-quality.md for frame budgets and profiling.

---

## Cross-framework note (brief)

| Framework | Canvas | Gradients/Brush | Shaders |
| --- | --- | --- | --- |
| **Compose** | `Canvas {}` / `Modifier.drawBehind` | `Brush.linearGradient/radialGradient/sweepGradient` | `RuntimeShader` (AGSL, API 33+) via `Modifier.graphicsLayer` |
| **SwiftUI** | `Canvas { ctx, size in ... }` / `Shape` | `LinearGradient` / `AngularGradient` / `RadialGradient` | `colorEffect` / `distortionEffect` / `layerEffect` (Metal `[[ stitchable ]]`) |
| **Flutter** | `CustomPaint` + `CustomPainter` | `ui.Gradient` / `Shader.linear/radial` | `FragmentShader` (load `.frag`, set uniforms) |

Same discipline everywhere: one hero effect, GPU-aware, animate on the render thread, provide a static fallback.

---

## Quality checklist (graphics)

- [ ] Skia chosen only because View/SVG/`expo-linear-gradient` genuinely can't do it.
- [ ] One signature canvas per screen — not a canvas behind every card.
- [ ] Dev build configured (Skia config plugin); confirmed it is NOT expected to run in Expo Go; app-size hit accepted.
- [ ] Animation driven by Reanimated shared values / `useClock` on the UI thread (no per-frame JS setState).
- [ ] Blur / BackdropFilter / layer passes minimized; canvas sized to the region, not the whole tree.
- [ ] Reduce Motion → static frame; clock paused off-screen.
- [ ] Low-end / older-Android fallback (flat gradient) wired and tested on a real device.
- [ ] Text/content over canvas still passes contrast in light **and** dark (see glassmorphism-and-materials.md).
