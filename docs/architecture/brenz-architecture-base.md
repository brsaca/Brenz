# Brenz — Architecture: Base

Date: 2026-11-26
Status: closed by Bren. No code.
Repo location: `docs/architecture/brenz-architecture-base.md`
Sibling documents: `brenz-architecture-domain.md`, `brenz-architecture-data.md`, `brenz-architecture-presentation.md`.
Product, screens and messages: `docs/product/` (scope, mockups, message bank, design system, assets).
Language: all documentation, file names, code and identifiers are in English. See `.cursor/rules/brenz-rule-language-naming.mdc`.

## Overrides in `brenz-scope-v1.md`
If the scope and these documents differ, the architecture wins on these points:
- Language (sections 2 and 3): v1 ships in Spanish and English. iPhone language Spanish: Spanish. Any other language: English.
- Water (sections 3 and 7): one document per glass (event). No daily document with a counter.
- Message rotation (section 7): lives on the device. It does not go in Firestore.
- Remote Config (section 4): adds `min_version`.
- Tab bar (section 6): custom `BrenzTabBar` component. Not native `TabView`.
- Cut order (section 8): see "Cut order" below.
- Schedule (section 8): English adds 4–6 h. Claude's assumption. Re-estimate.

## Decisions

### Platform
- [DECIDED] Minimum iOS: 17.
- [DECIDED] iPhone only.

### Presentation pattern
- [DECIDED] MVVM with `@Observable`.
- [DECIDED] Each ViewModel is an `@Observable @MainActor final class`.
- [DECIDED] The ViewModel calls use cases. It never calls Firebase.
- [DECIDED] No TCA. No Combine.

### Dependency injection
- [DECIDED] Own container `AppContainer`. No libraries (Factory, Swinject).
- [DECIDED] `AppContainer` creates repositories and use cases.
- [DECIDED] It is injected through `Environment` from `BrenzApp`.
- [DECIDED] ViewModels receive use cases through the initializer.
- [DECIDED] Tests pass fake implementations of the protocols.

### Folder structure
- [DECIDED] A single app. Folders by layer. No SPM modules per layer.
- [DECIDED] Exception: `BrenzDesignSystem` is a local Swift Package (see `brenz-design-system-decision.md`).

```
Brenz/
  App/               BrenzApp, AppContainer, RootViewModel
  Core/              extensions, logger, shared errors (no theme)
  Domain/
    Entities/
    Rules/           calculators, message selector, version comparator
    UseCases/
    Repositories/    protocols only
  Data/
    Firebase/        DTOs, repositories, error mapping
    Local/           MessageRotationStore, LocalCatalogProvider, ReminderScheduler
  Presentation/
    Auth/  Onboarding/  Home/  History/  Settings/
    Shared/          DayBoard and views that use entities
  Resources/         Localizable.xcstrings, Assets
Packages/BrenzDesignSystem/
firebase/            Firestore rules and emulator tests
docs/architecture/   these documents
docs/product/        scope, mockups, messages, assets
```

### Packages
- [DECIDED] Firebase iOS SDK through Swift Package Manager, three products: `FirebaseAuth`, `FirebaseFirestore`, `FirebaseRemoteConfig`.
- [DECIDED] No Analytics. No Crashlytics. TestFlight delivers the crash reports of the pilot.
- [DECIDED] No other third-party dependencies. The local `BrenzDesignSystem` package does not count.

### Errors and logging
- [DECIDED] Domain defines `AuthError` and `DataError`.
- [DECIDED] `AuthError`: invalid credentials, email in use, weak password, invalid email, wrong current password, session not recent, no network, unknown.
- [DECIDED] `DataError`: no network, permission denied, not found, unknown.
- [DECIDED] Data translates Firebase errors into those types. No Firebase type leaves Data.
- [DECIDED] Errors from optimistic writes do not reach the user. They are logged with `os.Logger`.
- [DECIDED] One `Logger` per layer. Subsystem: bundle id.
- [DECIDED] Logs never include email or nickname.

### Tests
- [DECIDED] Swift Testing (Xcode 16 or later).
- [DECIDED] Domain: calculators, `BoardLayout`, `VersionComparator`, `MessageSelector` and use cases with fake repositories.
- [DECIDED] Home, Auth and Settings ViewModels with the same fakes.
- [DECIDED] Firestore rules with the Firebase emulator, in `firebase/tests/`. Cases: no session, another user, editing `registeredAt`, lowering `stageReached`.
- [DECIDED] No UI tests in v1. Views use `#Preview`. Manual checklist before TestFlight: delete account, change password, both languages.

### Language
- [DECIDED] v1 in Spanish (`es`) and English (`en`). Strings for both languages are filled in completely.
- [DECIDED] `CFBundleDevelopmentRegion` and the catalog base language: `en`. Reason: iOS uses that region when the iPhone language matches none.
- [DECIDED] `es` covers Mexico, Spain and Latin America.
- [DECIDED] Store listing, screenshots and external pages (Support, Terms, Privacy Policy) in both languages.
- [DECIDED 26 Nov] Documentation, file names, code, identifiers and comments are written in English. No Spanglish in code. User-facing Spanish lives only in the String Catalog (`es`).

## Cut order (proposed, confirm when re-estimating)
0. Lock screen for `min_version` (the field and the comparator stay).
1. Intro animations.
2. History hourly charts.
3. Tatami growth.
4. Change password inside the app.
5. Animations for class deleted, account deleted and password updated.
6. Change the discipline of a log.
7. `BrenzTabBar` replaced by native `TabView` (saves 2–3 h).
8. Last resort: ship in Spanish only. The catalog stays ready.

## Constraints
- Do not add third-party dependencies.
- Do not import Firebase in Domain, Presentation or `BrenzDesignSystem`.
- Do not use `Combine`.
- Do not create SPM modules per layer.
- Do not write visible text in code. Everything goes in the catalog.

## Acceptance criteria
- Domain compiles without importing Firebase or SwiftUI.
- No ViewModel imports Firebase.
- `Package.resolved` lists only the Firebase SDK and its dependencies.
- Domain tests run without network or emulator.
- The app opens without errors with the iPhone in Spanish, in English and in another language (falls back to English).

## Suggested implementation order
1. Domain and its tests.
2. Data and its rules tests.
3. `BrenzDesignSystem`.
4. Presentation, screen by screen.

## Pending (close in Spec Kit)
- Exact values of the stored enums (example: `boxing`).
- Where the last used discipline is stored (proposal: `lastDiscipline` field in `UserProfile`).
