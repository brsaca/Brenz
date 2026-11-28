# Research: Settings and Account Management

**Feature**: 004-settings-account  
**Date**: 2026-11-27  
**Status**: Complete — all Technical Context unknowns resolved from architecture, 004 spec clarifications, and plan notes

## R1. Reuse vs fork for preference UI and Update* use cases

**Decision**: Reuse `NicknameField`, `DisciplineMultiSelect`, `ReminderEditor` and use cases `UpdateNickname`, `UpdateSelectedDisciplines`, `UpdateReminder` from feature 001 without duplicating or forking. Add `UpdateWaterGoal` in 004 for the Training water-goal control (4…16). Prefer these granular use cases over a single `UpdateProfile` wrapper for Settings edits.

**Rationale**: Plan notes and constitution IX require shared surfaces; onboarding and Settings edit the same profile fields. Domain “one use case per action” fits granular updates. Architecture’s `UpdateProfile` name is treated as the profile-edit family; Settings ships the named Update* actions already established for 001 plus water goal.

**Alternatives considered**:
- Monolithic `UpdateProfile` for all fields — rejected: diverges from 001 component/use-case pairing and complicates partial optimistic saves.
- Copy-paste Settings-specific fields — rejected: double maintenance and Spanglish risk.

## R2. Reuse sheet / success mark / tab bar / repositories / rules

**Decision**: Reuse the translucent floating sheet and `BrenzSuccessMark` success pattern from 003 (mockup 6d) for Delete Account 7b→7c and Change Password 7i. Keep `BrenzTabBar` as the shell from 002. Call existing `ProfileRepository` and `AuthRepository` protocols; extend Data implementations only. Keep a single `firebase/firestore.rules` (extend tests if needed). Do not create parallel protocols or a second rules file.

**Rationale**: Scope and mockups explicitly reuse 6d components for 7c/7i. Constitution I/V and data architecture require one rules file and Firebase behind existing protocols.

**Alternatives considered**:
- New confirmation components for account flows — rejected by product (“No new components”).
- Separate `AccountRepository` — rejected: Auth + Profile + Training already cover the delete order.

## R3. Sign Out order and local stores

**Decision**: Sign Out follows `docs/architecture/brenz-architecture-data.md`: (1) wait for pending writes up to **3 s**, (2) Auth sign out, (3) terminate + clear Firestore persistence, (4) clear per-user UserDefaults including `MessageRotationStore.clear()`, (5) cancel reminders, (6) rebuild repositories and return to Start. **Do not** clear `LaunchMarkerStore` / `hasLaunchedBefore`. No confirmation alert. Offline unsynced classes may be lost (accepted).

**Rationale**: Spec FR-003–FR-005 and data architecture; 001 FR-007 already requires the launch marker to survive Sign Out / Delete Account.

**Alternatives considered**:
- Alert before Sign Out — rejected by scope.
- Clear all UserDefaults including launch marker — rejected: marker is not user data; short splash / Start-without-Intro behavior would regress.

## R4. Delete Account order and wrong-password behavior

**Decision**: Order: reauthenticate with current password → delete Firestore data in batches (classLogs, waterLogs, then user document) → delete Auth user → same local cleanup as Sign Out steps 3–6 (including `MessageRotationStore.clear()`, excluding LaunchMarkerStore) → show 7c (**2.5 s** bar) → Start. Wrong password: delete nothing, keep sheet open, show 7e. Confirm button inactive while password empty. Network wait **10 s** → `noNetwork`. Reduce Motion: 7c without animation, still navigates to Start.

**Rationale**: Spec FR-006–FR-009, clarifications, data Delete account, constitution X (in-app deletion).

**Alternatives considered**:
- Delete Auth before Firestore — rejected: orphans progress under a live account.
- Separate “session expired” screen — rejected: reauth in 7b establishes recent session.

## R5. Change Password enable rules and errors

**Decision**: Button enabled iff all three fields non-empty, new password length ≥ 8, and repeat matches. Errors: wrong current → 7g; repeat mismatch → 7h (also keeps button disabled when mismatch). **No** “new password equals current” validation or error in v1 (overrides mockup prose that sketched that style). No recovery link on 7d. Wait **10 s** → `noNetwork`. Success 7i over darkened Settings (**2.5 s** bar), auto-closes; Settings remains. Reduce Motion skips animation.

**Rationale**: Spec FR-010–FR-013 and explicit plan notes. Product mockup mentioned “new equals current” as a style note; 004 clarifications / plan notes remove it from v1 scope.

**Alternatives considered**:
- Ship “new equals current” error — rejected for v1 by plan notes.
- Recovery link on Change Password — rejected: recovery stays on Sign In after Sign Out.

## R6. Version lock root surface (cut 0)

**Decision**: Activate the reserved `versionLock` root surface from 001’s extension point. Precedence: `versionLock` wins over splash targets, welcome, onboarding, home, and profile-load error. Evaluate on cold start and every foreground entry using `RulesConfig.minVersion` (Remote Config key `min_version`, default `"0.0.0"`) via `VersionComparator`. Invalid/missing → treat as `"0.0.0"` → never lock incorrectly. Optional UI is **cut 0** if schedule pressure; comparator + field always remain. Never raise remote `min_version` before Apple approves that version. App Store button uses a local store-listing URL constant (App Store Connect app id).

**Rationale**: Spec FR-019, presentation Version lock, data Remote Config, constitution X, plan notes.

**Alternatives considered**:
- Lock only inside Settings — rejected: must block the whole app.
- Force-update via blocking splash forever without App Store CTA — rejected: needs store button when included.

## R7. External links, Rate, footer

**Decision**: Support, Terms, and Privacy open external pages using the **same local URL constants** introduced in 001 (`LegalURLs` / equivalent). Rate the app presents the system review dialog (StoreKit `requestReview`); platform may no-op — still correct. Footer shows app icon, installed version string, Terms, Privacy.

**Rationale**: Spec FR-018 and clarifications (local constants, not Remote Config).

**Alternatives considered**:
- Remote Config for legal URLs — rejected by clarification.
- Custom in-app web views — rejected: open external pages.

## R8. Water goal bounds and Home side effects

**Decision**: `UpdateWaterGoal` accepts only 4…16 (Domain validation; Firestore rules already constrain). Default remains 8 until edited. Optimistic write; Home (002) reads `waterGoal` reactively from the profile stream and recalculates water fraction / messages immediately; glasses already logged stay even if count ≥ new goal. Feature 004 adds a verification task only; do **not** change Home unless that check fails.

**Rationale**: Spec FR-016 / US4 AS5, stored/profile rules, 002 already observes profile.

**Alternatives considered**: Cap UI without Domain check — rejected: Domain must enforce before repository write. Fork Home water logic in 004 — rejected unless verification fails.

## R9. Timeouts and error mapping

**Decision**: Canonical durations (same as plan.md):
- **Network wait = 10 s** for ChangePassword and DeleteAccount; timeout maps to `AuthError.noNetwork` without completing the action.
- **Sign Out pending sync = 3 s** then continue cleanup.
- **Success sheet auto-close bar = 2.5 s** for 7c / 7i.
Preference Update* calls are optimistic and do not surface server rejection in v1.

**Rationale**: Spec FR-013 / FR-005 / FR-009 / FR-012, data Offline writes, base AuthError set.

**Alternatives considered**: Infinite wait or immediate failure — rejected by clarifications. Approximate “~Ns” without fixed constants — rejected by analyze remediation A1.

## R10. Reduce Motion

**Decision**: Apply the existing BrenzDesignSystem Reduce Motion modifier to 7c and 7i (and any related success motion). When enabled: no checkmark draw / bar animation; success content still appears and navigation outcomes unchanged.

**Rationale**: Spec FR-020, constitution VIII, presentation Animations.

**Alternatives considered**: Custom per-screen Reduce Motion branching — rejected: package modifier is the single path.

## R11. Test placement and required cases

**Decision**: Tests only under `BrenzTests/Domain`, `BrenzTests/Data`, `BrenzTests/Presentation`. Required coverage:
1. Change Password enable rules (empty / short / mismatch / valid) + timeout/offline → noNetwork within **10 s**.
2. Wrong current password keeps Change Password screen and surfaces 7g.
3. Delete Account: wrong password deletes nothing; order reauth → Firestore → Auth → cleanup; offline/hang → noNetwork within **10 s**.
4. `hasLaunchedBefore` survives Sign Out and Delete Account (never cleared).
5. Water goal bounds 4…16 (reject out of range).
6. `ReconcileReminder`: permission revoked → reminder switched off; permission granted and reminder on → schedule rebuilt.
7. Notification permission denied → reminder forced off + notice + open system Settings.
8. SettingsViewModel: Sign Out has no confirmation and succeeds; link actions call the right handlers; `petNickname` clamped to 10 characters.
9. lastDiscipline fallback: gate on feature 002 `CurrentDisciplineResolverTests` — do **not** duplicate in 004.

**Rationale**: Plan notes + constitution VI + analyze remediation. No UI test target in v1. One test suite file per parallel task (no shared-file `[P]`).

**Alternatives considered**: UI tests for sheets — rejected for v1.

## R12. Cut order under schedule pressure

**Decision**: If time is short, cut in this order (confirmations / recovery remain):
0. Version lock UI (comparator + `min_version` field stay; invalid still non-locking).
4. Change Password in-app (email recovery via Sign In remains).
5. Success and deletion animations (7c/7i confirmations and navigation outcomes remain; static success content acceptable).

**Rationale**: Spec Assumptions + scope §8 + plan notes (aligned with base cut order items 0, 4, 5).

**Alternatives considered**: Cutting Sign Out or Delete Account — rejected: store compliance and account lifecycle require them.

## R13. Reminder schedule vs reconcile ownership

**Decision**: Feature **001** schedules reminders when the user turns the reminder on (`UpdateReminder` / `ReminderScheduler`). Feature **004** owns **reconcile-on-open** (FR-017): `ReconcileReminder` use case invoked from `BrenzApp` / `scenePhase` on app open. Domain tests: permission revoked → reminder switched off in profile/schedule; permission granted and reminder on → schedule rebuilt from profile.

**Rationale**: Spec FR-017; data architecture “On app open, it cancels and reschedules according to the profile”; analyze C1 (004 ownership, not a 001 confirm-only gap).

**Alternatives considered**: Leave reconcile as a silent 001 prerequisite — rejected: underspecified and untested in 004. Duplicate scheduling logic in Settings only — rejected: must reconcile on every open.
