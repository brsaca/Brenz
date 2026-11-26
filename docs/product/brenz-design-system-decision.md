# Brenz — Decision: Design System as a Swift Package

Date: 2026-11-26 (revision 5)
Source of truth: `docs/product/brenz-scope-v1.md`. If this document and the scope differ, the scope wins. On architecture (folders, text, animations), the `docs/architecture/brenz-architecture-*.md` documents win.
Changes in revision 5: translated to English. Token names are now English: `background`, `surface`, `surfaceHigh`, `accent`, `water`, `tatami`, `textPrimary`, `textSecondary`, `border`, `characterCard`, `danger`. The art-only token is `line`.
Changes in revision 4 (alignment with the architecture): `DayBoard` lives in `Presentation/Shared/`. Motion tokens and the "Reduce Motion" modifier go in the package. Fixed texts enter as `LocalizedStringResource`. The package has no text literals.
Changes in revision 3: added the `danger` token and the confirmation sheet components (delete log) with the class-deleted animation.

## Decision

[DECIDED] The design system is a local Swift Package: `BrenzDesignSystem`.
[DECIDED] A single target. No separate modules (Tokens, Components, Icons).
[DECIDED] The package has no domain or data logic.
[DECIDED] The package receives simple data. It does not receive domain entities.
[DECIDED] The app imports the package. The package imports nothing from the app.

## Reason

- The package prevents importing Firebase and domain at compile time.
- Only what is `public` leaves the package. Access stays controlled.
- Changing the design does not touch the screens.

## Location

```
Packages/BrenzDesignSystem/
├─ Package.swift
└─ Sources/BrenzDesignSystem/
   ├─ Tokens/       colors, spacing, radii, typography, motion
   ├─ Theme/        BrenzTheme, EnvironmentKey
   ├─ Styles/       ButtonStyle, LabelStyle, ViewModifier
   ├─ Components/   components without logic
   └─ Resources/    Assets.xcassets
```

## Tokens

[DECIDED] Semantic colors: `background`, `surface`, `surfaceHigh`, `accent`, `water`, `tatami`, `textPrimary`, `textSecondary`, `border`, `characterCard`, `danger`.
[DECIDED] v1 values (scope section 6):
- background #0B0B0C.
- surface #1B1B1E.
- surfaceHigh #26262A (chips, chart columns, icon boxes).
- accent #D7F500 (lime). Text on accent uses #0B0B0C.
- water #5CC8E8.
- tatami #D9A93B. Side faces #A5791C and #86610F. Marked cell #F0C95A.
- textPrimary #FFFFFF.
- textSecondary white at 68 %.
- border white at 14 %.
- characterCard #EFE3C8 (cream, character background).
- danger #FF6B5E (Nov 26). Only for destructive actions (delete log, delete account). Uses: text on danger at 16 % opacity (secondary destructive button) and solid fill with text #0B0B0C (confirmation button).
[DECIDED] The `line` token no longer exists in the interface. The dark line #2B2420 stays only in the character and element art.
[DECIDED] Dark mode only in v1.
[DECIDED] Colors live in the package Asset Catalog. They are loaded with `bundle: .module`.
[DECIDED] No view uses fixed colors. No view uses literal color names (`lime`, `cream`, `red`).
[DECIDED] Motion tokens (spring, duration) live in the package. A modifier reads "Reduce Motion" and applies a simple fade or no motion.

## Typography and shapes

[DECIDED] San Francisco with Dynamic Type styles.
[DECIDED] Large title: 32 pt Black, uppercase (Start, intro, account).
[DECIDED] Section title: 17 pt, medium weight, centered (History, Settings).
[DECIDED] Primary button: 16 pt Heavy, uppercase, pill with radius 28.
[DECIDED] Body: 17 pt, weight 500.
[DECIDED] Radii: boards and cards 24, Settings lists 20, speech bubble 28, sheet 40, tab bar 32.
[DECIDED] Screen margin: 16 pt.
[DECIDED] Boards and speech bubbles have no border.

## Theme

[DECIDED] `BrenzTheme` is a struct with the tokens.
[DECIDED] The default theme reads from the Asset Catalog.
[DECIDED] Views read the theme with `@Environment(\.theme)`.
[PROPOSED, out of v1] `themeId` in the profile chooses the theme per user.
[DECIDED] V1 has no theme picker. Only the `Environment` is ready.

## Text

[DECIDED] The package contains no text literals of its own.
[DECIDED] Fixed texts (buttons, titles, labels) enter as `LocalizedStringResource`.
[DECIDED] Dynamic texts (character message, nickname, figures) enter as `String`.

## How the design changes

- Value change (color, radius): edit the token or the Asset Catalog.
- Shape change (button, chip): edit the `ButtonStyle` or `ViewModifier`.
- Feature views do not change in either case.

## What goes inside the package

- Buttons, chips, cards. Includes the destructive button (`danger` fill) and the secondary destructive button (`danger` text on `danger` at 16 %).
- Floating tab bar (`BrenzTabBar`). It receives a list of items (title, icon) and the active item. It does not know the screens. Translucent background with `Material`.
- Floating translucent sheet (container without logic). The same sheet serves to change discipline, edit a log, confirm and show the result.
- Speech bubble with tail and close button (receives text and a close action).
- Generic isometric tile grid (receives rows, columns, filled cells and cap).
- Bottle as a shape (receives a value from 0 to 1).
- Chart bars per block (receives values and labels).
- Styles and modifiers.
- Motion tokens and the "Reduce Motion" modifier.
- Result view with animated checkmark (`BrenzSuccessMark`): circle, dotted ring and a checkmark that draws itself (`trim`). It receives an activation `Bool`. It respects "Reduce Motion" (no animation, full checkmark).

## What stays outside the package

- `DayBoard`. It calculates tiles from the day's classes. It is feature logic.
- Location of `DayBoard`: `Presentation/Shared/`. It is used in Home and in History.
- Any view that uses `Character`, `BoardElement`, `BoardTheme` or Firebase entities.
- The calculation of 3-hour blocks for History.
- The flow to delete a log (log sheet, confirmation, result and automatic close). It lives in the History feature.

## Constraints

- Do not add external dependencies to the package.
- Do not add more than one target in v1.
- Do not add light mode in v1.
- Do not add own animations in the package except the result checkmark. Motion tokens and the "Reduce Motion" modifier do go in the package. Respect "Reduce Motion".
- Do not move `DayBoard` into the package.
- Do not use `danger` outside destructive actions.
- Do not write text literals inside the package.

## Acceptance criteria

- `Package.swift` declares a single target with resources.
- No package file imports `FirebaseCore`, `FirebaseFirestore` or app modules.
- All colors come from `BrenzTheme`.
- Changing `accent` in the Asset Catalog changes the color across the whole app without editing views.
- Changing `danger` in the Asset Catalog changes all destructive buttons.
- Every public component has a `#Preview`.
- Components respect Dynamic Type.
- The tab bar lets the content behind show through.
- With "Reduce Motion" on, the result checkmark appears complete and without ring spin.
- No package file contains a visible text literal.

## Pending

- Final names of the spacing tokens.
- Contrast of 11 pt texts at 60 % white (close to the limit).
- Contrast of the `danger` text (#FF6B5E) on its 16 % background. Verify at implementation.
- `.mdc` rule in `.cursor/rules/` (see `brenz-rule-design-system.mdc`).
