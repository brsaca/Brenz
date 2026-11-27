# Implementation Plan: Account Onboarding

**Branch**: `001-account-onboarding` | **Date**: 2026-11-27 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-account-onboarding/spec.md`

**Note**: This template is filled in by the `/speckit-plan` command; its definition describes the execution workflow.

## Summary

Ship the mandatory account path (splash → Start screen → Intro → Sign Up / Sign In / Reset Password → Onboarding → Home) with Firebase Auth + Firestore profile, no guest mode. Session decides the post-splash destination (session + completed → home; session + incomplete **or missing profile document** → onboarding; session + profile **fetch failure** → `profileLoadError`; no session → `welcome` / Start screen); `hasLaunchedBefore` only picks full vs short splash and whether the Start screen leads to Intro — a reinstall with a saved session never shows the Start screen. This feature also establishes the shared app base: `Packages/BrenzDesignSystem`, `AppContainer`, and Firebase SPM products (`FirebaseAuth`, `FirebaseFirestore`, `FirebaseRemoteConfig` only). Root state exposes one unused extension point for a future version-lock surface (feature 004 / cut 0); version lock wins over every other root surface when later wired. Nickname (`petNickname` stored; UI copy: nickname), disciplines, and reminder each get one shared Presentation component and one Domain use case so Settings (004) reuses them without duplication. Firestore rules live in the single existing file `firebase/firestore.rules` (extend; do not add a second rules file).

## Technical Context

**Language/Version**: Swift 5.9+ / current Xcode (26.x), iOS 17 minimum

**Primary Dependencies**: SwiftUI, Observation (`@Observable`), Firebase iOS SDK via SPM (`FirebaseAuth`, `FirebaseFirestore`, `FirebaseRemoteConfig`), local package `Packages/BrenzDesignSystem`. No Analytics, no Crashlytics, no other third-party packages. Splash/Intro: native SwiftUI with local assets. US1 baseline = static fade + fixed Intro timer; US5 (cut order 1) adds looping animation only.

**Storage**: Firestore (`users/{uid}` profile) with offline persistence. `hasLaunchedBefore` in `UserDefaults` (device-level, not user data; set once when the full splash finishes the first time). Session in Firebase Auth / Keychain. Post-splash destination is session-driven; `hasLaunchedBefore` only selects full vs short splash and whether the Start screen leads to Intro.

**Testing**: Swift Testing for Domain, Data, and Presentation (ViewModels) with fake repositories under `BrenzTests/{Domain,Data,Presentation}/` only. Firestore security rules tests with the Firebase emulator under `firebase/tests/` against existing `firebase/firestore.rules`. No UI tests in v1. Timing assertions use an **injected monotonic clock**.

**Target Platform**: iOS 17+, iPhone only, dark mode only

**Project Type**: single iOS app module with layer folders + local package `Packages/BrenzDesignSystem`

**Performance Goals**: Returning user cold start ≈ 2 s to Home including 0.8 s short splash minimum on stable network; auth/reset **timeout 10 s** then clear no-network error

**Timing (injected monotonic clock)**: Splash minimums **1.5 s** (full) and **0.8 s** (short); Intro cycle **9 s** (Continuar/Empezar timer; looping polish when US5 ships); network **timeout 10 s**. Splash and Intro values are **minimums**; the network wait is a **timeout** (exact).

**Constraints**: MVVM with `@Observable @MainActor`; Clean Architecture behind Domain protocols; no Combine; no animation libraries; design tokens only from BrenzDesignSystem; all user-facing strings in `Localizable.xcstrings` (`en` base, `es`); English identifiers only; BrenzDesignSystem Reduce Motion modifier on splash/Intro at creation (US1); History and Settings tab shells are out of scope (feature 002)

**Scale/Scope**: Shared base + Auth `welcome` flow (Start screen + Intro + auth) + 3-step Onboarding + minimal Home destination (Train only). Screens: Splash (full/short), Start screen, Intro ×3, Sign Up, Sign In, Reset Password, Onboarding ×3, retryable profile-load error, Home placeholder. Shared reuse surfaces for 004: nickname, disciplines, reminder.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- I. Layer Separation: PASS — Auth/profile behind Domain protocols; Firebase confined to `Data/Firebase/`; Presentation and ViewModels do not import Firebase
- II. Platform and Stack: PASS — iOS 17, SwiftUI, MVVM `@Observable`, `AppContainer`; Firebase SPM products only; BrenzDesignSystem as sole local package
- III. Language and Naming: PASS — English code/docs/identifiers; UI copy only in `Localizable.xcstrings` (`en` base, `es`)
- IV. Pure Domain Rules: PASS — `VersionComparator` and weekday conversion stay pure with injected `now`/`Calendar`; nickname/discipline/reminder validation in Domain without clock reads
- V. Data Source and Stored Values: PASS — Firestore profile + Auth; stored discipline strings and `lastDiscipline` follow stored-values; optimistic class/water writes not required in this feature
- VI. Testing: PASS — Swift Testing for Domain/ViewModels; rules emulator tests for profile create/update; no UI tests
- VII. Content Voice: PASS — no character message bank in this feature; sarcasm rules deferred to 002
- VIII. UI and Design System: PASS — dark-only; tokens/components from BrenzDesignSystem; Reduce Motion modifier on splash/Intro at creation (US1); animation-specific fallbacks in US5 only
- IX. Scope Discipline: PASS — no guest mode, no Apple sign-in, no Change Password / Sign Out / Delete Account UI, no History/Settings shells, version-lock UI cut 0
- X. Release Safety: PASS — version-lock surface is an extension point only; `min_version` field/comparator available via Remote Config wiring without raising the gate; no forbidden brand names
- XI. Document Precedence: PASS — stored discipline/`lastDiscipline`/`selectedDisciplines` order taken from stored-values over product prose
- XII. Spec Kit Process: PASS — plan artifacts under `specs/001-account-onboarding/`; decisions already live in `docs/`

**Gate result**: PASS (no unjustified violations)

**Post-design re-check (Phase 1)**: PASS — `data-model.md` and `contracts/` keep Firebase behind Data, reuse Domain use cases for Settings without importing Firebase into Presentation, leave `versionLock` as an inert extension point (cut 0 / release safety), and follow stored-values for discipline strings, catalog order, and `lastDiscipline`.

## Project Structure

### Documentation (this feature)

```text
specs/001-account-onboarding/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   ├── firestore-users.md
│   ├── auth-errors.md
│   └── root-surfaces.md
└── tasks.md              # NOT created by /speckit-plan
```

### Source Code (repository root)

```text
Brenz/
├── App/
│   ├── BrenzApp.swift
│   ├── AppContainer.swift
│   └── RootViewModel.swift          # root surfaces + version-lock extension point
├── Core/
│   ├── AuthError.swift              # or Domain-owned errors re-exported
│   ├── AppError.swift
│   ├── Logger+Brenz.swift
│   └── LegalURLs.swift              # Terms + Privacy provisional URLs (final Dec 10)
├── Domain/
│   ├── Entities/
│   │   ├── AuthUser.swift
│   │   ├── UserProfile.swift        # includes lastDiscipline
│   │   ├── ReminderSettings.swift
│   │   └── Discipline.swift
│   ├── Rules/
│   │   ├── VersionComparator.swift  # wired for future lock; UI cut 0
│   │   ├── DisciplineCatalog.swift  # catalog order helpers
│   │   └── ReminderWeekday.swift    # Calendar weekday → Mon=1…Sun=7
│   ├── UseCases/
│   │   ├── SignUp.swift
│   │   ├── SignIn.swift
│   │   ├── ResetPassword.swift
│   │   ├── ObserveSession.swift
│   │   ├── ObserveProfile.swift
│   │   ├── CompleteOnboarding.swift
│   │   ├── UpdateNickname.swift     # shared with Settings (004)
│   │   ├── UpdateSelectedDisciplines.swift  # shared with Settings (004)
│   │   └── UpdateReminder.swift     # shared with Settings (004)
│   └── Repositories/                # protocols only
│       ├── AuthRepository.swift
│       ├── ProfileRepository.swift
│       ├── RulesConfigRepository.swift
│       └── ReminderScheduler.swift
├── Data/
│   ├── Firebase/
│   │   ├── FirebaseBootstrap.swift
│   │   ├── DTOs/UserProfileDTO.swift
│   │   ├── AuthRepositoryFirebase.swift
│   │   ├── ProfileRepositoryFirestore.swift
│   │   ├── RulesConfigRepositoryRemote.swift
│   │   └── AuthErrorMapper.swift
│   └── Local/
│       ├── LaunchMarkerStore.swift  # hasLaunchedBefore
│       └── ReminderSchedulerLocal.swift
├── Presentation/
│   ├── Auth/
│   │   ├── Welcome/                 # Start, Intro
│   │   ├── SignUp/
│   │   ├── SignIn/
│   │   └── ResetPassword/
│   ├── Onboarding/
│   ├── Home/                        # Train destination only (no History/Settings shells)
│   └── Shared/
│       ├── NicknameField.swift      # ONE component for onboarding + Settings
│       ├── DisciplineMultiSelect.swift
│       ├── ReminderEditor.swift
│       └── ProfileLoadErrorView.swift
└── Resources/
    └── Localizable.xcstrings
Packages/
└── BrenzDesignSystem/               # tokens, theme, Reduce Motion, shared chrome
firebase/
├── firestore.rules                  # SINGLE rules file (extend existing; do not create a second)
├── firebase.json                    # points at firestore.rules (when present)
└── tests/                           # emulator tests for users/{uid}
docs/
├── architecture/
└── product/
```

**Structure Decision**: Single app module with the constitution folder layout. Feature 001 creates `App/`, `Core/`, Domain auth/profile slice, Firebase + local data adapters, `Presentation/Auth`, `Presentation/Onboarding`, minimal `Presentation/Home`, and the three Shared editors. `Packages/BrenzDesignSystem` is bootstrapped here as the visual base. `Presentation/History` and `Presentation/Settings` shells are deferred to feature 002. Root exposes a version-lock surface slot consumed later by 004 without 001 implementing the lock UI.

## Complexity Tracking

> No constitution violations requiring justification.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| — | — | — |
