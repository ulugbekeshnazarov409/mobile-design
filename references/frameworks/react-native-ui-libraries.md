# React Native UI Libraries — detect, lock in, master

RN has no single UI standard, so the skill must (1) **detect which UI/styling library the project already uses**, (2) **commit to it for the entire task** — never mix two systems in one screen — and (3) know that library's theming, components, and idioms well enough to produce 1:1 production screens.

> Golden rule: **one styling system per app, used to the end.** If the repo styles with NativeWind, every new line is `className`. If it's Paper, every surface is a Paper component themed by the Paper theme. Don't drop a raw `StyleSheet` into a Tamagui app, or a Paper `<Button>` into a NativeWind screen, just because it's convenient.

---

## Detection — read these signals first

Check `package.json` deps + config + an existing screen. First match wins; if several coexist, follow what the *existing screens* actually use.

| Dependency / signal | Library | Styling model |
| --- | --- | --- |
| `nativewind` + `tailwind.config.js` + `global.css` | **NativeWind** (Tailwind) | utility `className` |
| `tamagui`, `@tamagui/*`, `tamagui.config.ts` | **Tamagui** | `styled()` + tokens + `$` props |
| `react-native-paper` | **React Native Paper** | Material 3, `PaperProvider` theme |
| `@rneui/themed` / `react-native-elements` | **RN Elements** | `ThemeProvider` + `makeStyles` |
| `@ui-kitten/components` + `@eva-design/eva` | **UI Kitten** | Eva Design tokens, `ApplicationProvider` |
| `@gluestack-ui/themed` / `@gluestack-ui/*` | **gluestack-ui** | utility props + `config` tokens |
| `@shopify/restyle` | **Restyle** | typed theme, `createBox/createText` |
| `dripsy` | **Dripsy** | theme-ui style `sx` |
| `react-native-unistyles` | **Unistyles** | `StyleSheet.create` + theme/runtime |
| `native-base` | **NativeBase** (deprecated → suggest gluestack) | utility props |
| none of the above | **Vanilla StyleSheet** | `StyleSheet.create` + token module |

Companion libs (orthogonal — add on top of any of the above): `@gorhom/bottom-sheet` (sheets), `@shopify/flash-list` (lists), `react-native-reanimated` + `moti` (animation), `react-native-gesture-handler` (gestures), `expo-image` (images), `@expo/vector-icons`/`lucide-react-native` (icons), `react-native-svg`.

---

## Per-library cheat-sheets

Only the essentials needed to build correctly and stay consistent. Pull deeper docs if the task is heavy.

### NativeWind (Tailwind for RN)
- **Theme:** `tailwind.config.js` (colors, spacing, radius, fontFamily) + CSS vars in `global.css`; dark via `dark:` + `useColorScheme()` from `nativewind`.
- **Use:** `className` on everything; variants `active:`, `disabled:`, `ios:`, `android:`, `web:`. No `StyleSheet`.
- **Components:** core RN primitives styled by className; for prebuilt, pairs with shadcn-style ports. Follow the `nativewind-ui` skill if present.

### Tamagui
- **Theme:** `tamagui.config.ts` tokens (`$color`, `$space`, `$radius`), themes light/dark; `<TamaguiProvider>`.
- **Use:** `<YStack>/<XStack>/<Stack>`, `<Text>/<SizableText>`, `$`-prefixed token props (`padding="$4"`), `styled()` for variants. Optional `@tamagui/*` component packages (Button, Sheet, Dialog).
- Compiler optimizes styles; keep to tokens.

### React Native Paper (Material 3)
- **Theme:** `MD3LightTheme`/`MD3DarkTheme` (or `useMaterial3`), wrap app in `<PaperProvider theme={…}>`; colors via `theme.colors.primary` etc.
- **Use:** `Button`, `TextInput`, `Card`, `List.Item`, `Appbar`, `FAB`, `Chip`, `Dialog`, `Snackbar`, `SegmentedButtons`. Don't hand-roll Material — use Paper's.
- Best when the app should look Material on both platforms.

### RN Elements (`@rneui/themed`)
- **Theme:** `createTheme()` + `<ThemeProvider>`; `makeStyles(theme => …)` for themed styles; `useTheme()`.
- **Use:** `Button`, `Input`, `Card`, `ListItem`, `Avatar`, `Tab`, `Overlay`, `Badge`, `Slider`. Cross-platform, fairly neutral look — theme it to brand.

### UI Kitten (Eva Design)
- **Theme:** `@eva-design/eva` (+ `eva.dark`) → `<ApplicationProvider {...eva} theme={...}>`; customize via JSON mapping/custom theme.
- **Use:** `Button`, `Input`, `Layout`, `Text` (with `category`), `Card`, `List`, `TopNavigation`, `BottomNavigation`. Strong token system; respect `status`/`appearance` props.

### gluestack-ui
- **Theme:** `config` tokens via `GluestackUIProvider`; utility-style props + `sx`.
- **Use:** `Box`, `VStack/HStack`, `Button`, `Input`, `Actionsheet`, `Modal`, `Toast`. Successor to NativeBase — prefer for new utility-prop apps.

### Restyle (`@shopify/restyle`)
- **Theme:** strongly-typed `theme.ts` (colors, spacing, textVariants, cardVariants); `ThemeProvider`.
- **Use:** `createBox<Theme>()`, `createText<Theme>()`, `useTheme`, `createVariant`. Props are theme keys (`bg="primary"`, `p="m"`). Great type-safety; everything goes through the theme.

### Dripsy
- **Theme:** theme-ui-shaped `theme` object + `DripsyProvider`; responsive arrays.
- **Use:** `View`/`Text`/`Pressable` from dripsy with `sx={{ … }}` referencing theme scales.

### Unistyles
- **Theme:** `UnistylesRegistry` themes/breakpoints; `createStyleSheet` + `useStyles`.
- **Use:** like `StyleSheet` but theme-aware + runtime (orientation, breakpoints). Good performance.

### Vanilla StyleSheet (fallback)
- Create one **token module** (`theme/tokens.ts`) with light+dark maps; every component imports tokens. No inline hex/magic numbers. `Platform.select` for OS diffs.

---

## Workflow when building an RN screen

1. **Detect** the library (table above) + companion libs + nav (Expo Router vs React Navigation).
2. **Lock in.** Declare in the plan: "Styling: <lib>." Use only that for the whole task.
3. **Reuse the theme.** Map design-foundations tokens onto the library's theme/tokens — don't invent a parallel palette.
4. **Reuse existing components** in `components/`/`ui/` before making new ones; match their prop API.
5. **Compose companions correctly:** sheets via `@gorhom/bottom-sheet`, long lists via FlashList, animation via reanimated/moti — these layer on top of any styling lib.
6. **Build, then run the restraint + accessibility pass** (`ui-restraint-and-accessibility.md`).

## If the project has NO UI library and one is warranted

Suggest (don't force) a fit, with reasoning + install:
- Want Material look, fast → **React Native Paper**.
- Want Tailwind/utility DX → **NativeWind**.
- Want max performance + design-system control → **Tamagui** or **Restyle**.
- Want type-safe minimal theme → **Restyle**.

Install the Expo way: `npx expo install <pkg>` and note provider wiring (e.g. wrap root in the library's Provider) + any babel/metro config. Then lock in and proceed.
