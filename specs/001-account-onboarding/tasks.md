---

description: "Task list for Account Onboarding implementation"
---

# Tasks: Account Onboarding

**Input**: Design documents from `/specs/001-account-onboarding/`

**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/, quickstart.md

**Tests**: Swift Testing for Domain and ViewModels with fake repositories. Firestore
rules tested with the Firebase emulator in `firebase/tests/`. No UI tests. Tests are
written in the same task as the code (or paired test tasks below). All suites MUST pass
before a task is complete. Do not add XCTest.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Domain**: `Brenz/Domain/` (Entities, Rules, UseCases, Repositories protocols only; no localized text)
- **Data**: `Brenz/Data/Firebase/` and `Brenz/Data/Local/` (DTOs, repository implementations)
- **Presentation**: `Brenz/Presentation/<Feature>/` and `Brenz/Presentation/Shared/` (views and ViewModels)
- **Composition root**: `Brenz/App/AppContainer.swift`
- **Design system**: `Packages/BrenzDesignSystem/`
- **Strings**: `Brenz/Resources/Localizable.xcstrings` (en base + es)
- **Firestore rules and tests**: `firebase/` — single rules file `firebase/firestore.rules` (extend existing; do not create a second)
- **Unit tests**: `BrenzTests/` mirroring allowed layers only: `Domain/`, `Data/`, `Presentation/` (no `BrenzTests/App/`)

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project folders, design system package, Firebase SPM products, localization catalog, placeholder assets

- [ ] T001 Create layer folders per plan.md under `Brenz/` (`App/`, `Core/`, `Domain/{Entities,Rules,UseCases,Repositories}/`, `Data/{Firebase/DTOs,Local}/`, `Presentation/{Auth/Welcome,Auth/SignUp,Auth/SignIn,Auth/ResetPassword,Onboarding,Home,Shared}/`, `Resources/`) and move `Brenz/BrenzApp.swift` into `Brenz/App/BrenzApp.swift`; remove or replace `Brenz/ContentView.swift` with root wiring later
- [ ] T002 [P] Bootstrap local package `Packages/BrenzDesignSystem/` with dark-only color/typography/spacing tokens, shared chrome stubs, and a Reduce Motion view modifier; link the package to the Brenz app target
- [ ] T003 [P] Confirm iOS 17 deployment target and add Firebase SPM products only (`FirebaseAuth`, `FirebaseFirestore`, `FirebaseRemoteConfig`) to the Brenz target; keep `Brenz/GoogleService-Info.plist` in the target
- [ ] T004 [P] Create `Brenz/Resources/Localizable.xcstrings` with `en` base + `es` and seed keys needed for splash/start placeholders (no hardcoded UI copy)
- [ ] T005 [P] Extend existing `firebase/firestore.rules` (deny-all baseline if not already present; do **not** create a second rules file) and add `firebase/tests/` harness stub for emulator rules tests (users/{uid} coverage lands with US1)
- [ ] T055 [P] Add placeholder local assets for splash/intro under `Brenz/Resources/` (or asset catalog) for native SwiftUI animation; final art may replace placeholders later

**Checkpoint**: Folders, design system, SPM, strings catalog, placeholder assets, and firebase rules harness exist

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared Domain types, repository protocols, composition root, Firebase bootstrap, root surface model, launch marker — MUST complete before ANY user story

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T006 [P] Create Domain entities `Brenz/Domain/Entities/AuthUser.swift`, `Brenz/Domain/Entities/UserProfile.swift` (fields: id, characterId, petNickname optional max 10, stageReached, waterGoal, onboardingCompleted, selectedDisciplines, lastDiscipline optional, reminder), `Brenz/Domain/Entities/ReminderSettings.swift` (isEnabled, days 1…7 Mon=1…Sun=7, timeHour, timeMinute), `Brenz/Domain/Entities/Discipline.swift` (stored raw: boxing, jiu_jitsu, muay_thai, kickboxing + unknown read-only; catalog order boxing → jiu_jitsu → muay_thai → kickboxing)
- [ ] T007 [P] Create Domain rules `Brenz/Domain/Rules/VersionComparator.swift` (pure; injected versions; no Date()), `Brenz/Domain/Rules/DisciplineCatalog.swift` (catalog order + sort selectedDisciplines; unknown never written), `Brenz/Domain/Rules/ReminderWeekday.swift` (Calendar weekday → Mon=1…Sun=7 with injected Calendar)
- [ ] T008 [P] Define repository protocols in `Brenz/Domain/Repositories/AuthRepository.swift`, `Brenz/Domain/Repositories/ProfileRepository.swift`, `Brenz/Domain/Repositories/RulesConfigRepository.swift`, `Brenz/Domain/Repositories/ReminderScheduler.swift` (protocols only; no Firebase)
- [ ] T009 [P] Create Core errors and logging in `Brenz/Core/AuthError.swift` (kinds: invalidCredentials, emailAlreadyInUse, weakPassword, invalidEmail, noNetwork, profileLoadFailed per contracts/auth-errors.md), `Brenz/Core/AppError.swift`, `Brenz/Core/Logger+Brenz.swift` (os.Logger; never log email or nickname)
- [ ] T010 Implement `Brenz/Data/Local/LaunchMarkerStore.swift` for device-scoped `hasLaunchedBefore` in UserDefaults (MUST NOT be cleared by Sign Out / Delete Account)
- [ ] T011 Implement Firebase bootstrap and DTO mapping in `Brenz/Data/Firebase/FirebaseBootstrap.swift` (Auth + Firestore offline persistence + Remote Config), `Brenz/Data/Firebase/DTOs/UserProfileDTO.swift` (Firestore shape per contracts/firestore-users.md including lastDiscipline null), `Brenz/Data/Firebase/AuthErrorMapper.swift` (Firebase codes → AuthError only inside Data)
- [ ] T012 Implement Data adapters `Brenz/Data/Firebase/AuthRepositoryFirebase.swift`, `Brenz/Data/Firebase/ProfileRepositoryFirestore.swift`, `Brenz/Data/Firebase/RulesConfigRepositoryRemote.swift` (expose min_version / defaults; no lock UI), `Brenz/Data/Local/ReminderSchedulerLocal.swift` (UNUserNotificationCenter)
- [ ] T013 Wire composition root `Brenz/App/AppContainer.swift` injecting Domain protocols with Data implementations; Environment-inject from `Brenz/App/BrenzApp.swift`
- [ ] T014 Create `Brenz/App/RootViewModel.swift` + root surface enum per contracts/root-surfaces.md (`splash(full|short)`, `welcome`, `onboarding`, `home`, `profileLoadError`, reserved `versionLock` unused cut 0; precedence: versionLock > splash hold > profileLoadError > welcome > onboarding > home). **Session decides destination** (session + completed → home; session + incomplete **or missing profile document** → onboarding; session + profile **fetch failure** → profileLoadError; no session → `welcome` / Start screen). `hasLaunchedBefore` selects **only** full vs short splash (and whether the Start screen leads to Intro). Reinstall with surviving session never shows the Start screen. Splash holds until session (and profile when session) resolves — no early navigate past 1.5 s / 0.8 s minimums (injected monotonic clock; see contracts/root-surfaces.md)
- [ ] T015 [P] Add Swift Testing coverage in `BrenzTests/Domain/DisciplineCatalogTests.swift` (unknown mapping; kickboxing-then-boxing sorts to [boxing, kickboxing]), `BrenzTests/Domain/VersionComparatorTests.swift`, `BrenzTests/Presentation/RootViewModelSurfaceTests.swift` (matrix stubs: no session → `welcome`; incomplete or missing profile document → onboarding; complete → home; profile fetch failure → profileLoadError; splash hold; reinstall-with-session never Start screen) with fakes; replace or keep `BrenzTests/Domain/SmokeTests.swift` as smoke only
- [ ] T016 Confirm the app builds and launches on the simulator (SweetPad or Xcode) showing splash/root shell wired to RootViewModel

**Checkpoint**: Foundation ready — user story implementation can now begin

---

## Phase 3: User Story 1 - Create account and complete onboarding (Priority: P1) 🎯 MVP

**Goal**: First-launch path Start → Intro → Sign Up → Onboarding (nickname, disciplines, reminder) → Home with session and `onboardingCompleted == true`

**Independent Test**: Device with no session and `hasLaunchedBefore` unset → full splash → Start → Intro → Sign Up → Onboarding → Home; profile shows onboarding complete

### Tests for User Story 1 (REQUIRED) ⚠️

> Write/run these with the matching implementation tasks. All suites MUST pass before the story checkpoint.

- [ ] T017 [P] [US1] Domain/use-case tests for SignUp (password < 8 → weakPassword before network; ~10 s timeout → noNetwork by design; success seeds profile onboardingCompleted false, selectedDisciplines `[boxing]`, lastDiscipline nil) and CompleteOnboarding in `BrenzTests/Domain/SignUpTests.swift` and `BrenzTests/Domain/CompleteOnboardingTests.swift` with fake repos
- [ ] T018 [P] [US1] ViewModel tests for SignUp/Onboarding `isSubmitting` blocking in `BrenzTests/Presentation/SignUpViewModelTests.swift` and `BrenzTests/Presentation/OnboardingViewModelTests.swift`
- [ ] T019 [P] [US1] Firestore emulator rules tests in `firebase/tests/` against existing `firebase/firestore.rules` for `users/{uid}`: authenticated owner create/update allowed; unauthenticated denied; foreign uid denied; invalid discipline string rejected; petNickname length ≤ 10; waterGoal 4…16; stageReached must not decrease

### Implementation for User Story 1

- [ ] T020 [P] [US1] Implement use cases `Brenz/Domain/UseCases/SignUp.swift` (client email format + password ≥ 8 before request; 10 s timeout → noNetwork by design; create Auth + profile stub: characterId `"fox"`, stageReached 1, waterGoal default 8, onboardingCompleted false, `selectedDisciplines = [boxing]`, `lastDiscipline = nil`; at-least-one discipline has no exception; **no email verification step** — FR-003), `Brenz/Domain/UseCases/CompleteOnboarding.swift`, `Brenz/Domain/UseCases/ObserveSession.swift`, `Brenz/Domain/UseCases/ObserveProfile.swift`
- [ ] T021 [P] [US1] Implement shared Domain use cases for Settings reuse: `Brenz/Domain/UseCases/UpdateNickname.swift` (optional; length 0…10; stored `petNickname`), `Brenz/Domain/UseCases/UpdateSelectedDisciplines.swift` (≥ 1 no exception; persist catalog order; default boxing), `Brenz/Domain/UseCases/UpdateReminder.swift` (optional; if enabled require ≥ 1 day + time; permission only when enabling)
- [ ] T022 [P] [US1] Build shared Presentation editors `Brenz/Presentation/Shared/NicknameField.swift`, `Brenz/Presentation/Shared/DisciplineMultiSelect.swift` (four disciplines; boxing preselected; multi-select; ≥ 1), `Brenz/Presentation/Shared/ReminderEditor.swift` using BrenzDesignSystem tokens only (`ProfileLoadErrorView` is owned by T030 / US2)
- [ ] T023 [US1] Implement Auth Welcome + Sign Up UI: `Brenz/Presentation/Auth/Welcome/` (Start screen + **non-cuttable Intro minimum**: 3 screens, segmented progress, **no skip control**, fixed **9 s timer** per screen then Continuar / **Empezar on screen 3**, **static fade** baseline no loop; apply BrenzDesignSystem **Reduce Motion modifier at creation** — looping animation polish only is T045), `NavigationStack` Intro → Sign Up with **back button and no X**, `Brenz/Presentation/Auth/SignUp/SignUpView.swift` + `SignUpViewModel.swift` (email + password only; Terms/Privacy links via provisional constants in `Brenz/Core/LegalURLs.swift` — final URLs Dec 10; no acceptance checkbox; no X/dismiss; link to Sign In; primary disabled while submitting)
- [ ] T024 [US1] Implement Onboarding UI `Brenz/Presentation/Onboarding/` (3 steps with progress dots: (1) NicknameField, (2) DisciplineMultiSelect, (3) ReminderEditor; reminder off → no permission prompt; reminder on → request permission; denied → force off + notice + open system Settings; finish → CompleteOnboarding → Home)
- [ ] T025 [US1] Implement minimal Home destination `Brenz/Presentation/Home/` (Train-only placeholder; no History/Settings shells) and wire root: session decides destination (SignUp success → onboarding; CompleteOnboarding → home; no session → `welcome`; missing profile document → onboarding; profile fetch failure → profileLoadError); full splash (1.5 s branded minimum) when !hasLaunchedBefore else short (0.8 s); apply BrenzDesignSystem **Reduce Motion modifier to splash at creation**; set `hasLaunchedBefore` **once when the full splash finishes the first time**; reinstall with session never Start screen
- [ ] T026 [US1] Add all US1 user-facing strings (splash, start, intro, sign up, onboarding, home, auth errors, reminder permission notice) en+es in `Brenz/Resources/Localizable.xcstrings`
- [ ] T027 [US1] Extend existing `firebase/firestore.rules` for users/{uid} owner read/write + field constraints from contracts/firestore-users.md (do not create a second rules file); verify emulator tests T019 pass

**Checkpoint**: US1 independently testable — first launch create account + onboarding reaches Home

---

## Phase 4: User Story 2 - Returning user with saved session (Priority: P1)

**Goal**: Cold start with saved session and completed onboarding shows short splash then Home only (no welcome/onboarding)

**Independent Test**: Saved session + onboardingCompleted → cold start → short logo splash (~0.8 s, longer if resolving) → Home; Start/Intro/Sign Up/Onboarding never appear

### Tests for User Story 2 (REQUIRED) ⚠️

- [ ] T028 [P] [US2] RootViewModel tests for completed-session matrix in `BrenzTests/Presentation/RootViewModelReturningUserTests.swift` (short splash → home; splash holds past 0.8 s until profile known; reinstall-with-session lands home or onboarding from cloud flag and **never Start**; hasLaunchedBefore does not override session destination)

### Implementation for User Story 2

- [ ] T029 [US2] Implement short splash presentation (logo only, ~0.8 s minimum) in root/splash views under `Brenz/Presentation/Auth/Welcome/` or `Brenz/App/` when `hasLaunchedBefore == true`; keep splash until ObserveSession + ObserveProfile resolve or fail
- [ ] T030 [US2] Build `Brenz/Presentation/Shared/ProfileLoadErrorView.swift` and wire RootViewModel path: session + onboardingCompleted → home; session + incomplete **or missing profile document** after successful load path → onboarding; session + profile **fetch failure** → profileLoadError (retry; keep session; never Home/Onboarding until load succeeds); reinstall with session never Start screen
- [ ] T031 [US2] Confirm session persistence via Firebase Auth Keychain across force-quit; add/adjust strings for profile-load error retry in `Brenz/Resources/Localizable.xcstrings` if missing

**Checkpoint**: US1 and US2 both work — returning users skip welcome flow

---

## Phase 5: User Story 3 - Sign in and resume incomplete onboarding (Priority: P2)

**Goal**: From Start screen, Sign In with email/password; completed → Home; incomplete or missing profile document → Onboarding 1; profile fetch failure → profileLoadError; cross-links Sign Up ↔ Sign In; errors stay on Sign In

**Independent Test**: Start screen → “Ya tengo cuenta” → Sign In with known account (complete, incomplete, and fetch-failure cases)

### Tests for User Story 3 (REQUIRED) ⚠️

- [ ] T032 [P] [US3] Domain SignIn tests in `BrenzTests/Domain/SignInTests.swift` (invalid credentials; any client-side format failure including password < 8 or malformed email → `invalidCredentials` before network — MUST NOT return `weakPassword`; ~10 s timeout → noNetwork by design; session unchanged on failure) with fake repos
- [ ] T033 [P] [US3] SignInViewModel tests in `BrenzTests/Presentation/SignInViewModelTests.swift` (`isSubmitting` blocks double submit; error kinds surface; no weakPassword)

### Implementation for User Story 3

- [ ] T034 [P] [US3] Implement `Brenz/Domain/UseCases/SignIn.swift` (client email format + password ≥ 8; any client-side format failure → `invalidCredentials` — MUST NOT return `weakPassword`; ~10 s wait → noNetwork by design; map AuthError; success does not skip Root routing)
- [ ] T035 [US3] Implement `Brenz/Presentation/Auth/SignIn/SignInView.swift` + `SignInViewModel.swift` (email, password, forgot-password link, create-account link; primary disabled while submitting; remain on screen on invalidCredentials / noNetwork)
- [ ] T036 [US3] Wire `welcome` navigation: Start screen “I already have an account” → Sign In; Sign Up ↔ Sign In text links; on success RootViewModel opens Home, Onboarding step 1 (incomplete or missing profile document), or profileLoadError (fetch failure) per FR-006 (session decides); add en+es strings in `Brenz/Resources/Localizable.xcstrings`

**Checkpoint**: Sign In works for complete and incomplete onboarding accounts

---

## Phase 6: User Story 4 - Reset password by email (Priority: P2)

**Goal**: From Sign In, request reset for any valid-format email; always same generic “link sent” confirmation; stay on Reset Password; no session; no account enumeration

**Independent Test**: Sign In → Reset Password → submit valid email (registered or not) → confirmation on same screen + path back to Sign In; no session

### Tests for User Story 4 (REQUIRED) ⚠️

- [ ] T037 [P] [US4] Domain ResetPassword tests in `BrenzTests/Domain/ResetPasswordTests.swift` (invalid email blocked client-side; valid format always success confirmation path regardless of registration; ~10 s → noNetwork by design; never establishes session) with fake repos
- [ ] T038 [P] [US4] ResetPasswordViewModel tests in `BrenzTests/Presentation/ResetPasswordViewModelTests.swift` (confirmation stays on screen; isSubmitting blocks primary action per FR-026)

### Implementation for User Story 4

- [ ] T039 [P] [US4] Implement `Brenz/Domain/UseCases/ResetPassword.swift` (~10 s wait; map noNetwork by design; Data layer MUST NOT surface user-not-found as distinct UI — always generic success for valid format)
- [ ] T040 [US4] Implement `Brenz/Presentation/Auth/ResetPassword/ResetPasswordView.swift` + `ResetPasswordViewModel.swift` (invalid-email error; generic link-sent confirmation; remain on screen; clear path back to Sign In; no auto-redirect that hides confirmation; primary disabled while submitting)
- [ ] T041 [US4] Wire Sign In “forgot password” → Reset Password; add en+es strings in `Brenz/Resources/Localizable.xcstrings`

**Checkpoint**: Password reset privacy contract satisfied; US3+US4 auth recovery path complete

---

## Phase 7: User Story 5 - First-launch intro visual polish (Priority: P3) — cut candidate (cut order 1)

**Goal**: Start-screen + Intro **visual polish only** on top of US1: add looping Intro animation; richer Start-screen character motion; animation-specific Reduce Motion fallbacks. Hard Intro rules (no skip, 9 s timer, Continuar/Empezar, static fade, Reduce Motion modifier) already owned by T023/T025. Cutting US5 removes **only the looping animation** (and related Start polish). Launch marker survives future Sign Out/Delete Account; later no-session opens use short splash → Start screen.

**Independent Test**: First launch → Start screen → all three Intro screens with Reduce Motion off and on; looping polish vs US1 static fade; after marker true + no session → short splash → Start screen without forced full splash/auto Intro

### Tests for User Story 5 (REQUIRED) ⚠️

- [ ] T042 [P] [US5] LaunchMarkerStore tests in `BrenzTests/Data/LaunchMarkerStoreTests.swift` (set once when full splash finishes first time; simulated sign-out/delete helpers MUST NOT clear hasLaunchedBefore)
- [ ] T043 [P] [US5] Root/Welcome tests in `BrenzTests/Presentation/RootViewModelLaunchMarkerTests.swift` (marker false → full splash; marker true + no session → short splash → `welcome` / Start screen; Intro only after Start action; session present → never Start screen regardless of marker; hasLaunchedBefore does not choose destination)

### Implementation for User Story 5

- [ ] T044 [US5] Polish Start screen in `Brenz/Presentation/Auth/Welcome/` (animated character area via native SwiftUI + local/placeholder assets, tagline, Start + I-have-an-account, Terms/Privacy external links via `Brenz/Core/LegalURLs.swift`; no acceptance checkbox) using BrenzDesignSystem + Localizable strings
- [ ] T045 [US5] Polish Intro in `Brenz/Presentation/Auth/Welcome/`: **add looping animation only** on top of T023 minimum (native SwiftUI + local assets; 9 s cycle aligned with existing timer); keep T023 no-skip / Continuar / Empezar rules unchanged; provide **animation-specific Reduce Motion fallback** for the loop (fade or no motion) — base Reduce Motion modifier already applied in T023
- [ ] T046 [US5] Provide **animation-specific Reduce Motion / asset fallback** for splash/intro looping polish when looping assets are unavailable or Reduce Motion is on; native SwiftUI only; T023 static fade + Reduce Motion modifier remain the baseline; usable through Sign Up
- [ ] T047 [US5] Confirm `hasLaunchedBefore` is set **once when the full splash finishes the first time** and document/guard that future Sign Out / Delete Account (004) MUST NOT clear `LaunchMarkerStore`; verify short splash → Start screen when marker true and no session (session present never Start screen)

**Checkpoint**: All user stories independently functional; intro accessibility and launch-marker rules met

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Localization completeness, design-system compliance, quickstart validation, cleanup

- [ ] T048 [P] Confirm every 001 screen string has en + es in `Brenz/Resources/Localizable.xcstrings` (Start, Intro, Sign Up, Sign In, Reset Password, Onboarding, splash, auth errors, profile-load error) — no mixed untranslated keys
- [ ] T049 Confirm all new views use BrenzDesignSystem tokens only (no literal colors/magic numbers); dark mode only; Reduce Motion modifier present on splash/Intro from T023/T025; US5 animation-specific fallbacks only where looping polish exists
- [ ] T050 [P] Confirm versionLock root case remains inert (cut 0); RulesConfigRepositoryRemote exposes min_version for future 004 without lock UI
- [ ] T051 Confirm Presentation/ViewModels do not import Firebase; Domain has no SwiftUI/Firebase; AuthErrorMapper stays in `Brenz/Data/Firebase/`
- [ ] T052 Run all BrenzTests + `firebase/tests/` emulator suites; fix failures
- [ ] T053 Run manual validation from `specs/001-account-onboarding/quickstart.md` (V1–V8) on simulator; verify SC-001…SC-009 acceptance targets where measurable
- [ ] T054 Code cleanup: remove leftover `ContentView` if unused; English identifiers only; commit in English after completed work when requested

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — start immediately
- **Foundational (Phase 2)**: Depends on Setup — **BLOCKS all user stories**
- **US1 (Phase 3)**: Depends on Foundational — MVP path
- **US2 (Phase 4)**: Depends on Foundational + US1 ObserveSession/ObserveProfile/Home wiring (routing matrix)
- **US3 (Phase 5)**: Depends on Foundational; integrates with US1 welcome stack and root routing
- **US4 (Phase 6)**: Depends on Foundational + US3 Sign In entry point
- **US5 (Phase 7)**: Depends on Foundational + US1 Welcome/Intro shells; adds looping polish and launch-marker confirmation only
- **Polish (Phase 8)**: Depends on all desired user stories

### User Story Dependencies

- **User Story 1 (P1)**: After Foundational — no dependency on other stories — **MVP** (owns non-cuttable Intro minimum + Reduce Motion modifier)
- **User Story 2 (P1)**: After Foundational; practically after US1 root/Home exists — independently testable with completed session fixture
- **User Story 3 (P2)**: After Foundational; shares `welcome` navigation with US1 — independently testable via Sign In
- **User Story 4 (P2)**: After US3 Sign In link exists — independently testable via Reset Password
- **User Story 5 (P3)**: After US1 Intro/Start shells — independently testable for looping polish / launch marker; cutting US5 removes only looping animation

### Within Each User Story

- Tests pass before a task is complete
- Domain use cases before ViewModels/views
- Shared editors before Onboarding hosting
- Core implementation before root integration
- Story complete before moving to next priority when sequential

### Parallel Opportunities

- Phase 1: T002, T003, T004, T005, T055 in parallel after T001 folder layout
- Phase 2: T006–T009 in parallel; then T010–T012; T015 parallel with T014 wiring
- US1: T017–T019 parallel; T020–T022 parallel; then T023→T027 sequential-ish
- US3/US4 domain tasks can proceed in parallel once Foundational is done if Sign In/Reset UI staffed separately
- US5 polish can overlap late US2/US3 if Welcome files are coordinated

---

## Parallel Example: User Story 1

```bash
# Tests + Domain use cases in parallel (different files):
Task: "SignUp/CompleteOnboarding tests in BrenzTests/Domain/"
Task: "Firestore users/{uid} emulator tests in firebase/tests/"
Task: "SignUp + CompleteOnboarding + Observe* use cases in Brenz/Domain/UseCases/"
Task: "UpdateNickname/Disciplines/Reminder use cases in Brenz/Domain/UseCases/"
Task: "NicknameField + DisciplineMultiSelect + ReminderEditor in Brenz/Presentation/Shared/"

# Then UI integration (depends on above):
Task: "Welcome + SignUp views in Brenz/Presentation/Auth/"
Task: "Onboarding + Home + root wiring"
```

---

## Parallel Example: User Story 3 + 4 (after Foundational)

```bash
Task: "SignIn use case + tests in Brenz/Domain/UseCases/SignIn.swift and BrenzTests/Domain/SignInTests.swift"
Task: "ResetPassword use case + tests in Brenz/Domain/UseCases/ResetPassword.swift and BrenzTests/Domain/ResetPasswordTests.swift"
# Then wire SignIn UI, then ResetPassword UI (Reset depends on Sign In navigation)
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL — blocks all stories)
3. Complete Phase 3: User Story 1
4. **STOP and VALIDATE**: Independent Test for US1 + quickstart V1
5. Demo if ready

### Incremental Delivery

1. Setup + Foundational → foundation ready
2. US1 → first-launch create + onboarding → MVP
3. US2 → returning session cold start
4. US3 → Sign In + incomplete onboarding resume
5. US4 → Reset Password privacy path
6. US5 → Intro looping / Start-screen visual polish + launch-marker confirmation (cut candidate; cut removes looping only)
7. Polish → localization, rules, quickstart V1–V8

---

## Notes

- [P] tasks = different files, no dependencies on incomplete tasks
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Tests pass before a task is complete (Swift Testing; no XCTest)
- Commit in English after the completed task when asked
- Stop at any checkpoint to validate story independently
- Avoid: guest mode, Apple sign-in, Change Password / Sign Out / Delete Account UI, History/Settings shells, version-lock UI, Analytics/Crashlytics, Combine, XCTest
- Avoid: hardcoded UI strings, literal colors, animation libraries without Reduce Motion fallback
- Shared editors + Update* use cases are the Settings (004) reuse contract — do not fork them
- `versionLock` is an inert extension point only (cut 0)
- **Cut candidate (cut order 1)**: User Story 5 (T044–T047) — looping Intro animation + Start-screen visual polish + animation-specific Reduce Motion fallbacks + launch-marker confirmation. MVP (US1) ships the non-cuttable Intro via T023 (no skip, 9 s timer, Continuar/Empezar, static fade, Reduce Motion modifier) and splash Reduce Motion via T025. **Cutting US5 removes only the looping animation** (and related Start polish); T023/T025 baseline remains.
- Session decides post-splash destination; `hasLaunchedBefore` only selects splash kind / Intro-from-Start; enum case is `welcome`, prose screen name is Start screen
- `BrenzTests/` allows only `Domain/`, `Data/`, `Presentation/` (RootViewModel tests live under Presentation)
- Single Firestore rules file: `firebase/firestore.rules` (extend existing)
- SignIn MUST NOT return `weakPassword`; client format failures → `invalidCredentials`
- Terms/Privacy provisional URL constants: `Brenz/Core/LegalURLs.swift` (final URLs Dec 10)
- Durations (injected monotonic clock): splash minimums 1.5 s full / 0.8 s short; Intro cycle 9 s; network timeout 10 s (timeout is exact; splash/Intro are minimums)
