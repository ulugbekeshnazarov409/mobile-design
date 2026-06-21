# Map UI Patterns — chrome over maps, sheets, and padding

Maps are a full-bleed canvas you place UI *on top of*. The recurring failures are all spatial: controls hidden behind a bottom sheet, the user's location pin trapped under a panel, markers clipped by the status bar, the camera not accounting for overlaid UI. The fix is one idea repeated everywhere — **the visible map ≠ the map view's frame; tell the map about its insets**.

Applies to Yandex MapKit, Google Maps (`google_maps_flutter`, native SDK), `react-native-maps`, and Mapbox. The padding API differs; the discipline doesn't.

---

## Layout anatomy

```
┌─────────────────────────┐
│ status bar (safe)       │
│  ◐ search / origin pill │  ← floating top control (over map, inside safe area)
│                         │
│         M A P           │  ← full-bleed; markers, route, user dot
│                  ┌────┐ │
│                  │ ⌖  │ │  ← my-location FAB (above the sheet's peek height)
│                  └────┘ │
│ ┏━━━━━━━━━━━━━━━━━━━━━┓ │
│ ┃ ▔▔ grabber          ┃ │  ← bottom sheet (peek → expanded)
│ ┃ place / list / route┃ │
│ ┗━━━━━━━━━━━━━━━━━━━━━┛ │
│ home indicator (safe)   │
└─────────────────────────┘
```

Rules:
- **Floating controls live inside safe areas**, not flush to the screen edge. Top pill below the status bar; FAB above the sheet peek.
- **One primary action.** Usually the bottom sheet's CTA ("Order", "Go", "Confirm pickup") or a single FAB — not a cluster of equal buttons.
- **My-location button** sits bottom-trailing, always above the sheet's current top edge — move it up when the sheet expands.
- Markers/callouts must stay clear of top and bottom chrome → that's what map padding is for.

---

## The padding rule (the #1 map bug)

When UI overlays the map (top pill, bottom sheet), the *geometric* center of the map view is no longer the *visible* center. Set the map's content inset/padding to the overlay heights so:
- "Center on user" centers in the **visible** area, not behind the sheet.
- `fitBounds`/route framing leaves markers visible, not under panels.
- Native attribution/logo & compass move to legal, visible positions.

**Keep map padding in sync with the bottom sheet height** — update it as the sheet drags/snaps.

### react-native-maps
```tsx
<MapView
  mapPadding={{ top: insets.top + 56, left: 0, right: 0, bottom: sheetHeight }}  // sheetHeight from sheet's animated position
  showsMyLocationButton={false}                 // draw your own FAB for placement control
/>
// recenter into the visible region:
mapRef.current?.animateCamera({ center: userCoord });   // with padding set, this lands above the sheet
```

### google_maps_flutter
```dart
GoogleMap(
  padding: EdgeInsets.only(top: topInset + 56, bottom: sheetHeight),  // rebuild on sheet change
  myLocationButtonEnabled: false,
  // ...
)
```

### Google Maps native
- iOS: `mapView.padding = UIEdgeInsets(top:…, left:0, bottom: sheetHeight, right:0)`
- Android: `googleMap.setPadding(0, topPx, 0, sheetPx)`

### Yandex MapKit
```kotlin
// Android — focusRect defines the visible viewport for camera fitting
mapView.mapWindow.focusRect = ScreenRect(
    ScreenPoint(0f, topInsetPx),
    ScreenPoint(width, height - sheetPx)
)
// when moving camera to user / route, geometry is framed inside focusRect
```
```swift
// iOS — focusRect on the map window, same idea (CGRect of the visible area)
mapView.mapWindow.focusRect = YMKScreenRect(...)
```
Yandex frames camera geometry into `focusRect`; treat it exactly like padding — shrink it by the sheet height.

---

## Bottom sheet + map (Uber / Yandex Go pattern)

- Sheet has **snap points**: peek (just the grabber + summary), mid (list), expanded (full). `@gorhom/bottom-sheet` (RN), `showModalBottomSheet`/`DraggableScrollableSheet` (Flutter), `.presentationDetents` (SwiftUI), `BottomSheetScaffold` (Compose M3).
- On each snap/drag, **push the map padding and the FAB up** by the sheet's top offset (animate together).
- Sheet must not cover the active marker — when a place is selected, recenter with padding so the pin sits in the gap above the sheet.
- Backdrop dim only at the expanded detent; peek/mid keep the map fully interactive.

---

## Markers, callouts, clusters

- **Marker = a real component, not the default red pin** for branded apps: rounded badge with icon/price, a tail, selected state (scale up + accent + elevation). Keep tap target ≥ 44/48 even if the visual is small.
- **Cluster** when markers crowd: show a count bubble; tap → zoom in / expand. Libraries: `react-native-map-clustering`, Google Maps utils, Yandex `ClusterizedPlacemarkCollection`.
- **Callout/info**: prefer driving the **bottom sheet** with the selected place over a floating bubble — easier to make premium and accessible than native callouts.
- Animate selection: marker scales, map eases to center it (with padding), haptic on select.

---

## Real-world & a11y

- **Permissions:** location is a flow, not a given — handle denied / "while using" / precise-vs-approximate; show a non-blocking prompt and a manual-search fallback. Never dead-end when permission is off.
- **Performance:** debounce camera-idle handlers; don't re-render the marker list on every frame; cap visible markers (cluster the rest). Tile/network failures need an offline/error state.
- **Dark mode:** ship a dark map style (Google JSON style / Yandex night / Mapbox dark) — a light map under a dark UI is an instant AI tell.
- **Accessibility:** map controls (FAB, recenter, layer toggle) need labels; provide a non-map fallback (list of results) for screen-reader users.
- **RTL/locale:** controls mirror in RTL; place names honor the app language where the SDK supports it.

---

## Quality checklist (maps)

- [ ] Map padding / `focusRect` set to top + bottom overlay heights, and **updated as the sheet moves**.
- [ ] Recenter & route-fit land in the **visible** area, never behind the sheet.
- [ ] Floating controls inside safe areas; my-location FAB rides above the sheet peek.
- [ ] Markers are branded components with a selected state; crowding handled by clustering.
- [ ] Selection drives the bottom sheet + recenter + haptic, not just a native callout.
- [ ] Location permission flow covers denied/approximate; manual fallback exists.
- [ ] Dark map style shipped; tiles/error/offline state handled; marker render perf bounded.
