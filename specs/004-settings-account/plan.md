# Implementation Plan: Settings and Account Management

**Branch**: `004-settings-account` | **Date**: 2026-11-27 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/004-settings-account/spec.md`

**Note**: This template is filled in by the `/speckit-plan` command; its definition describes the execution workflow.

## Summary

Replace the Settings placeholder shell from 002 with the full Settings surface: Character, Training, Account, More, and footer. Reuse onboarding preference components and Update* use cases from 001, the translucent sheet + animated checkmark from 003 (6d), BrenzTabBar, ProfileRepository, AuthRepository, and the single `firebase/firestore.rules`. Ship Sign Out (no alert), Delete Account (reauth → Firestore → Auth), Change Password (7d/7g/7h/7i, no recovery, no “new equals current” error), external links via 001 URL constants, Rate via system review, and an optional version-lock root surface that wins over every 001 surface through the reserved extension point. Cut order if short: 0) version lock UI, 4) Change Password, 5) success/deletion animations.

## Technical Context

**Language/Version**: Swift (Xcode 16 or later)

**Primary Dependencies**: SwiftUI, Observation, StoreKit (system review only), Firebase iOS SDK via SPM (FirebaseAuth, FirebaseFirestore, FirebaseRemoteConfig), local package BrenzDesignSystem. No other third-party.

**Storage**: Firestore with offline persistence. UserDefaults for MessageRotationStore (per user) and LaunchMarkerStore (`hasLaunchedBefore`, not user data). ReminderScheduler via UNUserNotificationCenter.

**Testing**: Swift Testing only under `BrenzTests/Domain`, `BrenzTests/Data`, and `BrenzTests/Presentation`. Fake repositories for Domain/ViewModels. Existing Firestore rules via emulator in `firebase/tests/` (extend cases if needed; do not create a second rules file). No UI tests.

**Target Platform**: iOS 17+, iPhone only.

**Project Type**: single iOS app with folders by layer + local package Packages/BrenzDesignSystem.

**Performance Goals**: Preference edits apply immediately (optimistic). Sensitive account actions wait **10 s** then map to noNetwork. Sign Out attempts pending sync **3 s**. Success sheets use a **2.5 s** auto-close bar. 60 fps UI; Reduce Motion skips success animations.

**Canonical durations** (reference from tasks; do not invent other values):

| Constant | Value | Used by |
|----------|-------|---------|
| Network wait | **10 s** | Change Password, Delete Account (timeout → `noNetwork`) |
| Sign Out pending sync | **3 s** | `waitForPendingWrites` then continue |
| Success sheet auto-close bar | **2.5 s** | Delete Account 7c, Change Password 7i |

**Constraints**: no animation libraries; design tokens only in views; en/es String Catalog; Reduce Motion via existing package modifier; dark mode only; no Combine; no duplicate protocols/components/rules; do not clear LaunchMarkerStore on Sign Out / Delete Account; never raise Remote Config `min_version` before Apple approves that version.

**Scale/Scope**: Settings tab content (replaces 002 shell); Change Password full screen; Delete Account sheet 7b→7c/7e; Sign Out; Support/Rate/Terms/Privacy; footer; optional VersionLock root surface (cut 0). Depends on 001 (auth, onboarding components/use cases, LegalURLs, root extension point) and 003 (translucent sheet, BrenzSuccessMark success pattern).

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- I. Layer Separation: PASS — Domain use cases + protocols; Firebase Auth/Firestore/Remote Config stay in Data/Firebase/; Presentation/ViewModels never import Firebase
- II. Platform and Stack: PASS — iOS 17+, SwiftUI, MVVM `@Observable`, AppContainer; Firebase SPM only; BrenzDesignSystem; StoreKit is system framework (not a third-party package)
- III. Language and Naming: PASS — English code/docs; Settings strings in Localizable.xcstrings (en base, es)
- IV. Pure Domain Rules: PASS — VersionComparator remains pure; water goal bounds validated in Domain; no Date()/Calendar.current inside rules
- V. Data Source and Stored Values: PASS — Firestore profile fields; optimistic preference writes; waterGoal 4–16; discipline catalog order for lastDiscipline fallback; single firestore.rules
- VI. Testing: PASS — Swift Testing in BrenzTests/{Domain,Data,Presentation}; rules emulator; no UI tests
- VII. Content Voice: PASS — no new character messages in this feature
- VIII. UI and Design System: PASS — dark only; reuse translucent sheet + BrenzSuccessMark + BrenzTabBar; Reduce Motion modifier on success motion
- IX. Scope Discipline: PASS — cut order documented; no change-email / Apple / recovery-from-Settings; no “new equals current” error in v1
- X. Release Safety: PASS — in-app Delete Account end-to-end; min_version default 0.0.0; never raise before approval; no Tamagotchi branding
- XI. Document Precedence: PASS — data Sign out / Delete account order and stored-values override product prose
- XII. Spec Kit Process: PASS — plan artifacts under specs/004-settings-account/

**Gate result**: PASS (no unjustified violations)

**Post-design re-check (Phase 1)**: PASS — `data-model.md` and `contracts/` reuse 001/003 surfaces and protocols without forking, keep Firebase in Data, wire `versionLock` through the 001 root precedence slot, document MessageRotationStore.clear vs LaunchMarkerStore retention, and confine tests to BrenzTests/{Domain,Data,Presentation}.

## Project Structure

### Documentation (this feature)

```text
specs/004-settings-account/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/           # Phase 1 output (/speckit-plan command)
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (repository root)

```text
Brenz/
├── App/                 # RootViewModel versionLock; BrenzApp / scenePhase → ReconcileReminder on open
├── Core/                # LegalURLs / AppStoreURL constants (reuse 001 LegalURLs; add store listing if needed)
├── Domain/
│   ├── Entities/        # UserProfile.waterGoal edits; AuthUser email display
│   ├── Rules/           # VersionComparator (existing); water goal bounds helpers if needed
│   ├── UseCases/        # Reuse UpdateNickname, UpdateSelectedDisciplines, UpdateReminder
│   │                    # New: UpdateWaterGoal, ChangePassword, DeleteAccount, SignOut,
│   │                    # ReconcileReminder (004 owns reconcile-on-open; 001 only schedules)
│   └── Repositories/    # Reuse AuthRepository, ProfileRepository, RulesConfigRepository,
│                        # MessageRotationStore, ReminderScheduler — no duplicate protocols
├── Data/
│   ├── Firebase/        # Auth changePassword/reauth/deleteUser; Profile update waterGoal;
│   │                    # batch delete user + classLogs + waterLogs; Remote Config min_version
│   └── Local/           # MessageRotationStore.clear() on Sign Out / Delete;
│                        # LaunchMarkerStore / hasLaunchedBefore never cleared;
│                        # ReminderScheduler.cancelAll on Sign Out / Delete
├── Presentation/
│   ├── Settings/        # SettingsView (+ sections), ChangePasswordView, DeleteAccount sheet,
│   │                    # SettingsViewModel, ChangePasswordViewModel, DeleteAccountViewModel
│   ├── Shared/          # Reuse NicknameField, DisciplineMultiSelect, ReminderEditor from 001
│   └── …                # Root version-lock view (optional cut 0)
├── Resources/
│   └── Localizable.xcstrings
Packages/
└── BrenzDesignSystem/   # Reuse translucent sheet, BrenzSuccessMark, BrenzTabBar, Reduce Motion
BrenzTests/              # Tests ONLY under Domain, Data, Presentation (no other test roots)
├── Domain/              # ChangePassword enable rules; DeleteAccount order; waterGoal bounds;
│                        # ReconcileReminder; hasLaunchedBefore survival (via LaunchMarker fake)
├── Data/                # SignOut / DeleteAccount local cleanup order; timeout → noNetwork
└── Presentation/        # ViewModel enable/error/sheet/Sign Out/links/reminder denied/nickname clamp
firebase/                # Existing firestore.rules + emulator tests (extend, do not fork)
docs/
├── architecture/
└── product/
```

**Structure Decision**: Replace `Presentation/Settings/` placeholder from 002 only. Shared preference UI and Update* use cases stay where 001 put them (Shared + Domain/UseCases); 004 consumes them. Account lifecycle use cases live in Domain/UseCases and call existing Auth/Profile/Training repositories. **004 owns** `ReconcileReminder` on app open; **001 only schedules** when the reminder is enabled. Version lock is a RootViewModel surface, not a Settings tab screen. No second rules file, no forked design components. Home already reads `waterGoal` from the profile stream (002); 004 verifies reactivity without changing Home unless that check fails.

## Complexity Tracking

> No constitution violations requiring justification.
