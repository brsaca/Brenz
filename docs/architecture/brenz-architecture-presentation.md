# Brenz — Architecture: Presentation

Date: 2026-11-26
Status: closed by Bren. No code.
Repo location: `docs/architecture/brenz-architecture-presentation.md`
Base: `brenz-architecture-base.md`. Design: `docs/product/brenz-design-system-decision.md`. Screens: `docs/product/brenz-mockups-v1.md`.

## Root screen and navigation
- [DECIDED] `RootViewModel` decides what to show based on the session and the profile:
  1. Splash until the first session and the first profile arrive. Full if `hasLaunchedBefore` is false. Short if it is true.
  2. No session: welcome flow (Start, Intro, Sign Up, Sign In, Reset Password).
  3. With session and `onboardingCompleted` false or profile missing: Onboarding.
  4. With session and `onboardingCompleted` true: Home with the tab bar.
- [DECIDED] Each tab has its own `NavigationStack`.
- [DECIDED] Routes are an `enum` per feature.
- [DECIDED] Sheets (discipline, log, delete account) use `.sheet`.
- [DECIDED] Custom tab bar: `BrenzTabBar`, in `BrenzDesignSystem`. Cut: native `TabView`.

## ViewModels
- [DECIDED] `@Observable @MainActor final class`. Read-only properties for the view. Actions are methods.
- [DECIDED] The view starts reading with `.task`. SwiftUI cancels the `AsyncStream` on exit.
- [DECIDED] Errors are published as an optional `AuthError` or `AppError`. The view shows them in the field or in an alert.
- [DECIDED] Forms (Sign Up, Change Password) keep `isSubmitting`. It blocks the button while waiting.
- [DECIDED] Home and History have no loading state. The local cache answers instantly.
- [DECIDED] The "Undo" timer (5 s) lives in the Home ViewModel.
- [DECIDED] `HomeViewModel` only displays messages. It does not choose messages. The water message lasts about 4 s and the base message returns.

## Text and String Catalog
- [DECIDED] A single `Localizable.xcstrings` in the app, with `en` (base) and `es`.
- [DECIDED] Keys by feature and element: `home.button.registerClass`, `settings.row.changePassword`.
- [DECIDED] Character message keys: `message.state.animated.1`. Dynamic keys are added to the catalog with state "manual". Xcode does not extract interpolated keys.
- [DECIDED] Resolved with `String(localized: String.LocalizationValue(key))`. With arguments, `String(format:)`.
- [DECIDED] Positional specifiers (`%1$@`). No `{stage}`. The stage name is a key (`stage.1.name`). It is resolved first and enters as an argument.
- [DECIDED] A unit test verifies that every `MessageKey` exists in the catalog, in both languages.
- [DECIDED] `AuthError` texts are translated in Presentation with the same pattern.
- [DECIDED] `BrenzDesignSystem` receives `LocalizedStringResource` for fixed text (buttons, titles, labels). It receives `String` for dynamic text (character message, nickname, figures). The nickname is never translated.
- [DECIDED] The package contains no text literals of its own.
- Dates, days and numbers use the device `Locale`.

## Animations
- [DECIDED] Native SwiftUI. No Lottie or libraries.
- Bounce, element appearance, bottle fill and state crossfade: transitions and `.animation(value:)`.
- Intro and splash loop: `phaseAnimator` or `TimelineView`.
- Motion constants (spring, duration) live in the package as tokens.
- A package modifier reads `accessibilityReduceMotion`. With "Reduce Motion" on, it applies a simple fade or no motion.
- No view decides on its own whether to animate.
- Clarification to the design system: the tokens and the modifier do not count as "own animations".

## Components
- [DECIDED] Theme, tokens, buttons, translucent sheet, speech bubble, grid, bottle, bars and `BrenzSuccessMark`: see `brenz-design-system-decision.md`.
- [DECIDED] `DayBoard` lives in `Presentation/Shared/`. It is used in Home and History. It does not go in the package.
- [DECIDED] The flow to delete a log lives in the History feature.
- [DECIDED] Any view that uses `Character`, `BoardElement` or `BoardTheme` stays outside the package.

## Reminder
- Onboarding and Settings turn the reminder on. Permission is requested only then.
- Permission denied: switch off, notice and shortcut to system Settings.

## Version lock screen (optional)
- If the installed version is lower than `minVersion`, it shows a lock screen with a button to the App Store.
- First cut of the schedule. The field and the comparator are not cut.
- The App Store link is missing. It comes from the app ID in App Store Connect.

## Constraints
- Do not import Firebase in Presentation.
- Do not use fixed colors or literal color names. Everything comes from `BrenzTheme`.
- Do not write visible text in code.
- Do not choose messages inside the ViewModels.
- Do not use `Combine`.
- Do not add animation libraries.

## Acceptance criteria
- No ViewModel imports Firebase.
- With "Reduce Motion" on, there is no bounce or spin. There is a fade or nothing.
- With the iPhone in English, no screen shows Spanish text or untranslated keys.
- With a 10-character nickname, no screen clips or breaks the layout.
- Home and History open without a loading screen using the local cache.
- With a saved session, launch goes from short splash to Home.
