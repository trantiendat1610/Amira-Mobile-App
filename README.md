# Mobile App Design Tokens

Design tokens for the **Mobile App** design system, exported from Figma and generated for Web, iOS and Android from a single source.

| Category   | Tokens | Notes |
|------------|--------|-------|
| Color      | 123 palette + 13 semantic | Light & Dark modes (12 hues, Neutral 1–13, success / warning / info / error / link) |
| Typography | 25 text styles | Google Sans Flex · Base, SM, LG, XL (Normal, Strong, Underline, Delete, Italic) · Heading 1–5 |
| Size       | 9 | 4 → 56 |
| Space      | 8 margin + 13 padding | aliases of Size |
| Radius     | 5 | 4 / 8 / 12 / 16 / 24 |
| Control    | 4 heights + 4 line widths | |
| Shadow     | 3 | multi-layer drop shadows |

Preview all tokens: open [`docs/index.html`](docs/index.html), or enable GitHub Pages on the `/docs` folder.

## Repository layout

```
tokens/                 ← source of truth (W3C Design Tokens / DTCG JSON)
  core.json             typography, size, space, radius, shadow
  color/light.json      color palette + semantic colors, light mode
  color/dark.json       color palette + semantic colors, dark mode
tokens.config.json      name prefixes and Kotlin package
build.js                zero-dependency generator (Node ≥ 18)
dist/                   generated — do not edit
  web/                  tokens.css, typography.css, tokens.js, tokens.d.ts, tokens.json
  ios/                  DesignTokens.swift (SwiftUI)
  android/res/          values/ds_colors.xml, values-night/ds_colors.xml, values/ds_dimens.xml
  android/compose/      DesignTokens.kt (Jetpack Compose)
docs/index.html         generated preview page
```

## Updating tokens

1. Edit the JSON in `tokens/`. References use `{group.token}`, e.g. `"{size.sm}"`.
2. Run `npm run build` (or `node build.js`).
3. Commit both `tokens/` and the regenerated `dist/` + `docs/`.

CI runs `node build.js --check` on every push and pull request and fails if the generated files are stale.

## Usage

### Web (CSS)

```html
<link rel="stylesheet" href="dist/web/tokens.css">
<link rel="stylesheet" href="dist/web/typography.css">
```

```css
.card {
  padding: var(--ds-padding-lg);
  border-radius: var(--ds-radius-base);
  background: var(--ds-color-neutral-1);
  box-shadow: var(--ds-shadow-secondary);
  font: var(--ds-typography-base-normal);
}
```

Dark mode follows the OS automatically. Force a mode with `<html data-theme="dark">` or `data-theme="light"`.
Each text style also has a utility class: `.ds-text-base-strong`, `.ds-text-heading1`, …

### JavaScript / TypeScript / React Native

```ts
import { color, padding, radius, typography } from 'mobile-design-tokens';

const styles = {
  backgroundColor: color.light.blue[6],   // or color.dark.blue[6]
  padding: padding.lg,                     // 24 (numbers are px/dp/pt)
  borderRadius: radius.base,               // 16
  ...typography.base.strong,               // { fontFamily, fontSize, lineHeight, fontWeight, … }
};
```

### iOS (SwiftUI, iOS 16+)

Add the package with Swift Package Manager (`File ▸ Add Package Dependencies…` → this repo URL), or copy `dist/ios/DesignTokens.swift` into your project.

```swift
import DesignTokens

Text("Hello")
    .dsTextStyle(DSTypography.baseStrong)
    .foregroundStyle(DSColor.neutral12)
    .padding(DSPadding.lg)
    .background(DSColor.blue1)
    .clipShape(RoundedRectangle(cornerRadius: DSRadius.base))
    .dsShadow(DSShadow.secondary)
```

Colors switch between light and dark automatically. Add the Google Sans Flex font files to your app and list them under `UIAppFonts` in `Info.plist`.

### Android (Jetpack Compose)

Copy `dist/android/compose/DesignTokens.kt` into your app and change the `package` line (or set `kotlinPackage` in `tokens.config.json` and rebuild).

```kotlin
DSTypography.fontFamily = FontFamily(Font(R.font.google_sans_flex)) // once, at startup

DSTheme {
    Text(
        "Hello",
        style = DSTypography.baseStrong,
        color = DS.colors.neutral12,
        modifier = Modifier
            .background(DS.colors.blue1, RoundedCornerShape(DSRadius.base))
            .padding(DSPadding.lg)
    )
}
```

### Android (XML views)

Copy `dist/android/res/` into `app/src/main/res/`. Colors in `values-night` are used automatically in dark mode.

```xml
<TextView
    android:textColor="@color/ds_neutral_12"
    android:padding="@dimen/ds_padding_lg"
    android:textSize="@dimen/ds_font_size_base" />
```

## Naming

| Figma variable | CSS | Swift | Kotlin | Android XML |
|---|---|---|---|---|
| `Color/Base/Blue/6` | `--ds-color-blue-6` | `DSColor.blue6` | `DS.colors.blue6` | `@color/ds_blue_6` |
| `Color/Brand/Success/colorSuccessBg` | `--ds-color-success-bg` | `DSColor.successBg` | `DS.colors.successBg` | `@color/ds_success_bg` |
| `Space/Padding/paddingContentHorizontalLG` | `--ds-padding-content-horizontal-lg` | `DSPadding.contentHorizontalLG` | `DSPadding.contentHorizontalLG` | `@dimen/ds_padding_content_horizontal_lg` |
| `Border Radius/BorderRadius` | `--ds-radius-base` | `DSRadius.base` | `DSRadius.base` | `@dimen/ds_radius_base` |
| Text style `Base/Strong` | `.ds-text-base-strong` | `DSTypography.baseStrong` | `DSTypography.baseStrong` | — |
| Effect style `boxShadowSecondary` | `--ds-shadow-secondary` | `DSShadow.secondary` | `DSShadow.secondary` | — |

Base-size tokens with no suffix in Figma (`size`, `padding`, `margin`, `BorderRadius`, `controlHeight`, `lineWidth`, `FontSize`) are named `base` here.

## Platform notes

- Shadows: SwiftUI and Compose have no shadow spread, so `spread` is kept as data only. SwiftUI's shadow radius is set to half the Figma blur.
- Line height on iOS is approximated with `lineSpacing` + vertical padding, since SwiftUI has no exact line-height API.

## License

[MIT](LICENSE)
