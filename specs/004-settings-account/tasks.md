---

description: "Task list for Settings and Account Management implementation"
---

# Tasks: Settings and Account Management

**Input**: Design documents from `/specs/004-settings-account/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Swift Testing for Domain and ViewModels with fake repositories. Firestore
rules tested with the Firebase emulator in `firebase/tests/`. No UI tests. Tests are
written in the same task as the code. All suites MUST pass before a task is complete.
Do not add XCTest. Required coverage per research.md R11 is mapped into story phases.
Tests live **only** under `BrenzTests/Domain`, `BrenzTests/Data`, `BrenzTests/Presentation`.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

**Depends on**: Features 001 (auth, onboarding preference components/use cases, LegalURLs, root `versionLock` slot; **001 only schedules** reminders on enable), 002 (Settings tab shell / BrenzTabBar; `CurrentDisciplineResolverTests` gate for lastDiscipline fallback), 003 (translucent sheet + BrenzSuccessMark). This feature replaces the 002 Settings placeholder only. **004 owns** reminder reconcile-on-open (`ReconcileReminder`, FR-017).

**Canonical durations** (plan.md / research.md R9): network wait **10 s**; Sign Out pending sync **3 s**; success sheet auto-close bar **2.5 s**.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies). Do **not** mark `[P]` on tasks that write the same file.
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Domain**: `Brenz/Domain/` (Entities, Rules, UseCases, Repositories protocols only; no localized text)
- **Data**: `Brenz/Data/Firebase/` and `Brenz/Data/Local/` (DTOs, repository implementations)
- **Presentation**: `Brenz/Presentation/Settings/` and `Brenz/Presentation/Shared/` (views and ViewModels)
- **Composition root**: `Brenz/App/AppContainer.swift`
- **App entry**: `Brenz/App/BrenzApp.swift` (scenePhase → `ReconcileReminder`)
- **Design system**: `Packages/BrenzDesignSystem/`
- **Strings**: `Brenz/Resources/Localizable.xcstrings` (en base + es)
- **Firestore rules and tests**: `firebase/` (single `firestore.rules` — never a second rules file)
- **Unit tests**: `BrenzTests/` mirroring Domain / Data / Presentation only

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Confirm prerequisites from 001–003 and create the Settings feature folders this branch owns

- [ ] T001 Confirm 001–003 prerequisites exist: `NicknameField`, `DisciplineMultiSelect`, `ReminderEditor`, `UpdateNickname`, `UpdateSelectedDisciplines`, `UpdateReminder`, `LegalURLs`, `AuthRepository`, `ProfileRepository`, `RulesConfigRepository`, `VersionComparator`, `MessageRotationStore`, `LaunchMarkerStore`, `ReminderScheduler` (**001 only schedules** the reminder when enabled — reconcile-on-open is **004** / T043, not 001), translucent sheet + `BrenzSuccessMark`, Settings tab shell / `BrenzTabBar`, and root surface slot for `versionLock`. Cite feature 002 `CurrentDisciplineResolverTests` as the gate for lastDiscipline → first-selected catalog fallback (**do not duplicate** that suite in 004)
- [ ] T002 Create `Brenz/Presentation/Settings/` folder structure for Settings views and ViewModels (`SettingsView`, `SettingsViewModel`, Change Password, Delete Account) replacing the 002 placeholder body only
- [ ] T003 [P] Confirm `Brenz/Resources/Localizable.xcstrings` exists for en + es and reserve `settings.*` / `versionLock.*` key namespaces (no hardcoded UI copy)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared account-cleanup, repository extensions, Settings shell scaffolding, and DI — MUST complete before ANY user story

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T004 Extend `AuthRepository` protocol in `Brenz/Domain/Repositories/AuthRepository.swift` with `reauthenticate(password:)`, `changePassword(to:)`, `deleteUser()`, `signOut()`, and pending-writes wait (**3 s** for Sign Out; **10 s** timeout → `AuthError.noNetwork` for sensitive actions); implement in `Brenz/Data/Firebase/` without exposing Firebase types outside Data
- [ ] T005 [P] Extend `ProfileRepository` (and Training delete helpers as needed) in `Brenz/Domain/Repositories/` + `Brenz/Data/Firebase/` for optimistic preference updates including `waterGoal` (**4…16**) and batch delete of `users/{uid}` + that user's `classLogs` + `waterLogs` for Delete Account
- [ ] T006 Implement shared local session cleanup in `Brenz/Data/Local/` (and/or Firebase session teardown): Firestore `terminate` + `clearPersistence`, **`MessageRotationStore.clear()`** on Sign Out and Delete Account, `ReminderScheduler` cancel-all, rebuild repositories via `Brenz/App/AppContainer.swift` — **never** clear `LaunchMarkerStore` / `hasLaunchedBefore`
- [ ] T007 [P] Scaffold Settings shell UI in `Brenz/Presentation/Settings/SettingsView.swift` + `SettingsViewModel.swift`: Character, Training, Account, More, footer sections; one `NavigationStack`; email read-only from session; wire into 002 tab destination; inject Domain protocols only via `Brenz/App/AppContainer.swift`
- [ ] T008 [P] Add Settings chrome strings (section titles, rows, footer labels, no-network, reminder permission notice) en + es in `Brenz/Resources/Localizable.xcstrings` per `contracts/settings-ui.md`
- [ ] T009 Confirm `firebase/firestore.rules` already enforces `waterGoal` 4…16; extend `firebase/tests/` only if coverage is missing — do **not** create a second rules file

**Checkpoint**: Foundation ready — user story implementation can now begin in priority order (or in parallel if staffed)

---

## Phase 3: User Story 1 - Sign Out (Priority: P1) 🎯 MVP

**Goal**: Signed-in user taps Sign Out in Settings with no alert; pending sync ≤**3 s**; session ends; local data cleared except launch marker; `MessageRotationStore.clear()`; reminders cancelled; app shows Start; cloud-synced progress remains for later Sign In

**Independent Test**: From a signed-in session with local progress, sign out and confirm Start appears, no local session remains, reminders are gone, `hasLaunchedBefore` survives, and a later Sign In restores cloud progress

### Tests for User Story 1 (REQUIRED) ⚠️

> **NOTE: Write these tests in the same task as the code. All suites MUST pass before a task is complete.**

- [ ] T010 [P] [US1] Domain/Data test that `hasLaunchedBefore` survives Sign Out (LaunchMarkerStore untouched) in `BrenzTests/Data/SignOutCleanupTests.swift` (and/or Domain fake coverage)

### Implementation for User Story 1

- [ ] T011 [US1] Implement `SignOut` use case in `Brenz/Domain/UseCases/SignOut.swift`: wait pending writes ≤**3 s** → Auth sign out → shared local cleanup (T006, including `MessageRotationStore.clear()`) → root returns to Start; no confirmation alert; unsynced offline classes may be lost (accepted)
- [ ] T012 [US1] Wire Sign Out in `Brenz/Presentation/Settings/SettingsViewModel.swift` + More section row in `Brenz/Presentation/Settings/SettingsView.swift` (immediate tap, no alert); rebuild session via `Brenz/App/AppContainer.swift` / `RootViewModel`
- [ ] T013 [US1] Add Sign Out–related strings en + es in `Brenz/Resources/Localizable.xcstrings` if any remain beyond T008

**Checkpoint**: User Story 1 fully functional and independently testable (MVP account exit)

---

## Phase 4: User Story 2 - Delete Account (Priority: P1)

**Goal**: Delete Account sheet 7b (password + eye, confirm inactive when empty, Cancel); on success reauth → delete classLogs + waterLogs + user doc → delete Auth → local cleanup like Sign Out (`MessageRotationStore.clear()`; never clear `hasLaunchedBefore`) → 7c (**2.5 s** / Reduce Motion static) → Start; wrong password → 7e, nothing deleted; offline → wait **10 s** → noNetwork

**Independent Test**: Delete an account end to end with the correct password; verify Start, empty session, deleted progress, cancelled reminders, launch marker intact. Separately verify wrong password and offline failure without deletion

### Tests for User Story 2 (REQUIRED) ⚠️

- [ ] T014 [US2] Domain tests in `BrenzTests/Domain/DeleteAccountTests.swift` (single suite file — no parallel siblings on this path): wrong password deletes nothing; order is reauth → Firestore (logs then profile) → Auth → local cleanup (`MessageRotationStore.clear()`, LaunchMarker untouched); offline/hang maps to `AuthError.noNetwork` within **10 s** and deletes nothing
- [ ] T015b [P] [US2] Presentation test: DeleteAccountViewModel confirm enable (non-empty password), wrong password → 7e, noNetwork mapping, and success path signals 7c → Start in `BrenzTests/Presentation/DeleteAccountViewModelTests.swift`

### Implementation for User Story 2

- [ ] T016 [US2] Implement `DeleteAccount` use case in `Brenz/Domain/UseCases/DeleteAccount.swift` (reauth → batch Firestore delete → Auth delete → shared cleanup from T006 including `MessageRotationStore.clear()`; ≤**10 s** → `AuthError.noNetwork`; map wrong password to domain error for 7e; never clear `hasLaunchedBefore`)
- [ ] T017 [P] [US2] Add `DeleteAccountViewModel` in `Brenz/Presentation/Settings/DeleteAccountViewModel.swift` (confirm enable when password non-empty; wrong-password field error; noNetwork; success → 7c then Start)
- [ ] T018 [US2] Add Delete Account confirmation/success UI (sheet 7b → 7c/7e) in `Brenz/Presentation/Settings/` reusing translucent sheet + `BrenzSuccessMark` from 003/BrenzDesignSystem; **2.5 s** lime bar; Reduce Motion via package modifier; present from Settings Account section
- [ ] T019 [US2] Add Delete Account strings (7b/7c/7e, no-network) en + es in `Brenz/Resources/Localizable.xcstrings`

**Checkpoint**: User Stories 1 AND 2 both work independently (store-compliant account deletion)

---

## Phase 5: User Story 3 - Change Password (Priority: P1)

**Goal**: Full-screen Change Password 7d (current/new/repeat + eyes; 8+ notice; no recovery); enable iff all filled ∧ new.count ≥ 8 ∧ new == repeat; wrong current → 7g; mismatch → 7h; success → 7i over Settings then dismiss (**2.5 s** bar); offline → **10 s** → noNetwork; Reduce Motion skips animation; no “new equals current” check in v1

**Independent Test**: Change password with valid inputs; verify success sheet then Settings, and Sign In works with the new password. Separately verify validation and offline errors

### Tests for User Story 3 (REQUIRED) ⚠️

- [ ] T020 [US3] Domain tests in `BrenzTests/Domain/ChangePasswordTests.swift` (single suite file): enable rules (empty / new &lt; 8 / mismatch / valid); timeout/offline maps to noNetwork within **10 s** and does not change the password
- [ ] T021 [US3] Presentation tests in `BrenzTests/Presentation/ChangePasswordViewModelTests.swift` (single suite file): enable-rule UI state; wrong current password stays on 7d and surfaces 7g; noNetwork mapping

### Implementation for User Story 3

- [ ] T022 [US3] Implement `ChangePassword` use case in `Brenz/Domain/UseCases/ChangePassword.swift` (reauth + change password; ≤**10 s** → `AuthError.noNetwork`; no “new equals current” validation)
- [ ] T023 [P] [US3] Add `ChangePasswordViewModel` in `Brenz/Presentation/Settings/ChangePasswordViewModel.swift` (enable rules; 7g/7h/noNetwork mapping; success triggers 7i)
- [ ] T024 [US3] Add `ChangePasswordView` full screen 7d in `Brenz/Presentation/Settings/ChangePasswordView.swift` (no recovery link) + success sheet 7i over darkened Settings reusing 003 sheet/`BrenzSuccessMark`; **2.5 s** bar; Reduce Motion; route from Settings `NavigationStack`
- [ ] T025 [US3] Add Change Password strings (7d/7g/7h/7i, 8+ notice, no-network) en + es in `Brenz/Resources/Localizable.xcstrings`

**Checkpoint**: Account lifecycle (Sign Out, Delete, Change Password) complete and independently testable

---

## Phase 6: User Story 4 - Edit Profile and Training Preferences (Priority: P2)

**Goal**: Edit `petNickname` (UI copy: nickname; optional, max 10), selected disciplines (≥1; lastDiscipline fallback via 002 `CurrentDisciplineResolverTests`), water goal 4…16, and reminder (permission only when enabling; denied → off + notice + open system Settings; **001 schedules** on enable). Optimistic updates apply on Home immediately. **004** reconciles reminder on app open (T043).

**Independent Test**: Change each preference while signed in and confirm Home reflects nickname, discipline selection, water goal, and reminder schedule; verify permission-denied reminder behavior; verify reconcile-on-open

### Tests for User Story 4 (REQUIRED) ⚠️

- [ ] T026 [P] [US4] Domain test: `UpdateWaterGoal` rejects outside **4…16** and accepts bounds in `BrenzTests/Domain/UpdateWaterGoalTests.swift`

### Implementation for User Story 4

- [ ] T027 [P] [US4] Implement `UpdateWaterGoal` use case in `Brenz/Domain/UseCases/UpdateWaterGoal.swift` (Domain validation **4…16**; optimistic write via `ProfileRepository`; default remains 8 until edited)
- [ ] T028 [US4] Wire Character + Training sections in `Brenz/Presentation/Settings/SettingsView.swift` / `SettingsViewModel.swift`: reuse `NicknameField` + `UpdateNickname` (`petNickname`, max 10, empty hides nickname; UI copy: nickname); reuse `DisciplineMultiSelect` + `UpdateSelectedDisciplines` (≥1; lastDiscipline fallback owned/gated by 002 — do not reimplement tests); reuse `ReminderEditor` + `UpdateReminder` (permission only when turning on; **001 schedules** when enabled; denied → notice + open system Settings — reconcile is T043); add water goal control 4…16 → `UpdateWaterGoal`
- [ ] T029 [US4] Add preference-specific strings (water goal, reminder denied notice) en + es in `Brenz/Resources/Localizable.xcstrings` if not covered by T008; confirm nickname / `petNickname` is never translated
- [ ] T043 [US4] Implement `ReconcileReminder` use case in `Brenz/Domain/UseCases/ReconcileReminder.swift` and call it on app open from `Brenz/App/BrenzApp.swift` (or root `scenePhase`); Domain tests in `BrenzTests/Domain/ReconcileReminderTests.swift`: permission revoked → reminder is switched off; permission granted and reminder on → schedule is rebuilt from profile. Feature **004 owns** reconcile-on-open (FR-017); **001 only schedules**

**Checkpoint**: Preferences editable from Settings and reflected on Home without sync wait; reminder reconciles on open

---

## Phase 7: User Story 5 - External Links, Rate App, and Footer (Priority: P2)

**Goal**: Support / Terms / Privacy open external pages via 001 `LegalURLs` local constants; Rate uses system review dialog; footer shows app icon + installed version + Terms/Privacy entry points

**Independent Test**: From Settings, open each external link destination and trigger Rate the app; confirm footer shows icon and version

### Implementation for User Story 5

- [ ] T030 [US5] Wire link actions in `Brenz/Presentation/Settings/SettingsView.swift` / `SettingsViewModel.swift`: More → `LegalURLs.support`; footer → `LegalURLs.terms` and `LegalURLs.privacy` externally (not Remote Config)
- [ ] T031 [US5] Present system review dialog for Rate the app via StoreKit in `Brenz/Presentation/Settings/` (platform may no-op — still correct); keep Firebase out of Presentation. Sequential with T030 (same Settings presentation files — no `[P]`)
- [ ] T032 [US5] Implement Settings footer (app icon + installed bundle version string + Terms/Privacy) in `Brenz/Presentation/Settings/SettingsView.swift`; version string not localized as prose; strings for Rate/Support rows en + es in `Brenz/Resources/Localizable.xcstrings`

**Checkpoint**: Legal/support surfaces and footer ready for store checklist

---

## Phase 8: User Story 6 - Minimum-Version Lock Screen (Priority: P3) — cut 0 if short

**Goal**: When installed &lt; valid `min_version`, show lock screen with App Store button blocking the app; evaluate on cold start and every foreground entry; missing/invalid min → treat as `"0.0.0"` → never lock. Optional UI is first cut; comparator + field always remain. Never raise remote `min_version` before Apple approval

**Independent Test**: With min higher than installed, confirm lock and App Store action on cold start and foreground; with equal/higher installed or invalid/missing min, confirm no lock

### Implementation for User Story 6

- [ ] T033 [P] [US6] Confirm/extend `VersionComparator` + `RulesConfigRepository` (`min_version` default `"0.0.0"`) in `Brenz/Domain/Rules/VersionComparator.swift` and Data Remote Config; invalid/missing → `"0.0.0"`; add/keep Domain tests that never false-lock
- [ ] T034 [US6] Activate reserved `versionLock` root surface in `Brenz/App/RootViewModel.swift` (precedence: versionLock &gt; splash &gt; profileLoadError &gt; welcome &gt; onboarding &gt; home); evaluate on cold start and every foreground entry
- [ ] T035 [US6] Add `VersionLockView` in `Brenz/Presentation/` (or Shared/Root) using BrenzDesignSystem tokens; App Store CTA via local store-listing URL constant in `Brenz/Core/` (alongside `LegalURLs`); block all other use
- [ ] T036 [US6] Add `versionLock.*` strings en + es in `Brenz/Resources/Localizable.xcstrings`

**Checkpoint**: Optional lock works when included; if cut, leave T034–T036 undone but keep T033 comparator defaults non-locking

---

## Phase 9: Polish & Cross-Cutting Concerns

**Purpose**: Localization, design-system compliance, Reduce Motion, coverage gaps (G1/G3/G4/G5/G6), and quickstart validation across stories

- [ ] T037 [P] Confirm all new Settings/account/version-lock strings have en + es in `Brenz/Resources/Localizable.xcstrings` with no mixed untranslated keys (SC-009)
- [ ] T038 Confirm views use BrenzDesignSystem tokens only; dark mode only; Reduce Motion respected on 7c/7i via package modifier; no animation libraries; no literal colors/magic numbers; durations use **10 s** / **3 s** / **2.5 s** constants from plan
- [ ] T039 [P] Confirm reuse boundaries in `specs/004-settings-account/contracts/reuse.md` (no forked Nickname/Discipline/Reminder, no second rules file, no recovery link on Change Password, LaunchMarker / `hasLaunchedBefore` never cleared, `MessageRotationStore.clear()` on Sign Out and Delete Account)
- [ ] T040 Run required Swift Testing suites in `BrenzTests/Domain`, `BrenzTests/Data`, `BrenzTests/Presentation` and any extended `firebase/tests/`; all MUST pass
- [ ] T041 Validate manual scenarios A–F in `specs/004-settings-account/quickstart.md` on the simulator (Sign Out, Delete, Change Password, Preferences, Links/Rate/Footer, Version lock or cut-0 comparator safety); include SC-010 manual check that a 10-character nickname does not clip on Settings and Home
- [ ] T042 Commit in English after the completed work (Conventional Commits)
- [ ] T044 Verification: lowering the water goal below today’s glass count makes Home recalculate its water-related state; confirm Home reads `waterGoal` reactively from the profile stream (002). **Do not change Home** unless this check fails
- [ ] T045 Presentation tests in `BrenzTests/Presentation/SettingsViewModelTests.swift` (single suite file — covers G1/G3/G5/G6): (1) notification permission denied → reminder forced off + notice + open system Settings; (2) `petNickname` / nickname input clamped to 10 characters; (3) Sign Out has no confirmation and succeeds; (4) Support / Terms / Privacy / Rate link actions call the right handlers

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — can start immediately (assumes 001–003 landed or land first)
- **Foundational (Phase 2)**: Depends on Setup — **BLOCKS** all user stories
- **User Stories (Phase 3–8)**: All depend on Foundational completion
  - Prefer priority order: US1 → US2 → US3 → US4 → US5 → US6
  - US4 and US5 can proceed in parallel after Foundational
  - US6 is optional (cut 0); can be deferred entirely under schedule pressure
- **Polish (Phase 9)**: Depends on all desired user stories being complete (T044/T045 after Settings shell + preference wiring)

### User Story Dependencies

- **User Story 1 (P1) Sign Out**: After Foundational — uses shared cleanup (T006); no dependency on other stories — **MVP**
- **User Story 2 (P1) Delete Account**: After Foundational — reuses shared cleanup; independently testable from US1 UI but shares T006
- **User Story 3 (P1) Change Password**: After Foundational — independent of US1/US2 UI; uses Auth extensions (T004)
- **User Story 4 (P2) Preferences**: After Foundational — independent; needs Settings shell (T007); T043 after ReminderScheduler available
- **User Story 5 (P2) Links / Rate / Footer**: After Foundational — independent; needs Settings shell (T007); T030 then T031 (same files)
- **User Story 6 (P3) Version lock**: After Foundational — independent root surface; **cut first** if short (keep comparator)

### Within Each User Story

- Tests pass before a task is complete
- Use cases / Domain rules before ViewModels
- ViewModels before SwiftUI views
- Story complete before moving to next priority when solo

### Parallel Opportunities

- T003 with other Setup once T001–T002 are clear
- T005, T007, T008 in parallel within Foundational (after T004/T006 sequencing for cleanup consumers)
- Once Foundational completes: US3, US4, US5, US6 can start in parallel with US1/US2 if staffed
- T014 is a single DeleteAccount Domain suite (T015/T015a merged); T015b may run `[P]` (different file)
- T020 and T021 are one suite each (Domain vs Presentation); no shared-file `[P]` (T021a merged into them)
- T026/T027 in parallel (different files)
- T030 then T031 sequentially (shared Settings presentation files)
- T037/T039 in parallel within Polish

### Cut order (if schedule pressure)

0. US6 Version lock UI (T034–T036) — keep T033  
4. US3 Change Password (Phase 5) — email recovery via Sign In remains  
5. Success/deletion animations — keep 7c/7i confirmations and navigation; static success acceptable  

Do **not** cut Sign Out or Delete Account.

### Merged / retired task IDs (stable — do not reuse numbers)

- **T015**, **T015a**: merged into **T014** (`DeleteAccountTests.swift`)
- **T021a**: merged into **T020** / **T021** (Domain vs Presentation files)
- New IDs appended: **T043**, **T044**, **T045**

---

## Parallel Example: User Story 2

```bash
# Single Domain suite (not parallel siblings on the same file):
Task: "Domain DeleteAccountTests: wrong password / order / 10 s noNetwork in BrenzTests/Domain/DeleteAccountTests.swift"

# Different file — may run in parallel with T014:
Task: "DeleteAccountViewModelTests in BrenzTests/Presentation/DeleteAccountViewModelTests.swift"

# After use case lands, ViewModel can proceed while strings are drafted:
Task: "Add DeleteAccountViewModel in Brenz/Presentation/Settings/DeleteAccountViewModel.swift"
Task: "Add Delete Account strings en + es in Brenz/Resources/Localizable.xcstrings"
```

## Parallel Example: User Story 3

```bash
# One Domain file + one Presentation file (not multiple [P] writers on the same path):
Task: "ChangePasswordTests enable rules + 10 s noNetwork in BrenzTests/Domain/ChangePasswordTests.swift"
Task: "ChangePasswordViewModelTests enable UI + 7g + noNetwork in BrenzTests/Presentation/ChangePasswordViewModelTests.swift"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup  
2. Complete Phase 2: Foundational (CRITICAL — blocks all stories)  
3. Complete Phase 3: User Story 1 (Sign Out)  
4. **STOP and VALIDATE**: Independent Sign Out test (quickstart A)  
5. Demo account exit if ready  

### Incremental Delivery

1. Setup + Foundational → foundation ready  
2. US1 Sign Out → validate → MVP account exit  
3. US2 Delete Account → validate → store compliance  
4. US3 Change Password → validate (or cut 4 if needed)  
5. US4 Preferences + ReconcileReminder (T043) → validate Home side effects  
6. US5 Links / Rate / Footer → store checklist  
7. US6 Version lock → optional (cut 0)  
8. Polish → T044/T045 + quickstart A–F  

### Parallel Team Strategy

1. Team completes Setup + Foundational together  
2. Once Foundational is done:  
   - Developer A: US1 + US2 (shared cleanup consumers)  
   - Developer B: US3 Change Password  
   - Developer C: US4 + US5 preferences/links  
   - Developer D (optional): US6 version lock  
3. Stories integrate through Settings shell + AppContainer without forking shared components  

---

## Notes

- [P] tasks = different files, no dependencies on incomplete sibling work
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Tests pass before a task is complete (Swift Testing; no XCTest)
- Commit in English after the completed task (Conventional Commits)
- Stop at any checkpoint to validate story independently
- Avoid: vague tasks, third-party packages, hardcoded UI strings
- Avoid: literal colors/magic numbers in views, animation libraries, motion without Reduce Motion fallback
- Avoid: forking 001 preference components/use cases; second `firestore.rules`; clearing LaunchMarkerStore / `hasLaunchedBefore`
- Always call `MessageRotationStore.clear()` on Sign Out and Delete Account
- Avoid: change email, Sign in with Apple, password recovery from Settings, “new equals current” error
- Never raise Remote Config `min_version` before Apple approves that version
- Durations: network wait **10 s**, Sign Out pending sync **3 s**, success bar **2.5 s**
