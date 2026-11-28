# Contract: Reuse boundaries (001 / 002 / 003 / firebase)

**Feature**: 004-settings-account  
**Date**: 2026-11-27

## Must reuse (do not fork)

| Asset | From | 004 usage |
|-------|------|-----------|
| `NicknameField` | 001 Shared / Onboarding | Character section |
| `DisciplineMultiSelect` | 001 | Training section |
| `ReminderEditor` | 001 | Training section |
| `UpdateNickname` | 001 Domain | Character saves |
| `UpdateSelectedDisciplines` | 001 Domain | Training disciplines |
| `UpdateReminder` | 001 Domain | Training reminder — **001 schedules** when turned on |
| `LegalURLs` (Support, Terms, Privacy constants) | 001 Core | More → Support; footer → Terms + Privacy |
| Settings tab shell / `BrenzTabBar` | 002 + DesignSystem | Navigation entry only; replace placeholder body |
| `CurrentDisciplineResolverTests` | 002 Domain tests | Gate for lastDiscipline → first selected catalog fallback; **do not duplicate in 004** |
| Translucent floating sheet | 003 / BrenzDesignSystem | 7b / 7c / 7i |
| `BrenzSuccessMark` (+ **2.5 s** progress bar pattern) | 003 / BrenzDesignSystem | 7c / 7i |
| Reduce Motion modifier | BrenzDesignSystem | Success motion |
| `AuthRepository` protocol + impl | 001 Data | Change password, reauth, delete user, sign out, session email |
| `ProfileRepository` protocol + impl | 001 Data | Observe/update profile fields (`petNickname`, `waterGoal`, …) |
| `RulesConfigRepository` + `VersionComparator` | 001 / Domain | `min_version` evaluation |
| `MessageRotationStore` | Data/Local | `.clear()` on Sign Out / Delete Account |
| `LaunchMarkerStore` / `hasLaunchedBefore` | 001 Data/Local | **Read-only survival** — never clear here |
| `ReminderScheduler` | Data/Local | Cancel on Sign Out / Delete; scheduling API used by 001 enable + 004 reconcile |
| `firebase/firestore.rules` | firebase/ | Single file; profile waterGoal 4…16 already required — **do not create a second rules file** |

## May extend (same types)

- `AuthRepository`: ensure `reauthenticate`, `changePassword`, `deleteUser`, `signOut`, `wait`/timeout mapping exist (**10 s** network wait; **3 s** Sign Out pending sync).
- `ProfileRepository`: `updateWaterGoal` (or generic update used by `UpdateWaterGoal`).
- Training repository delete-all for uid during DeleteAccount batch cleanup (implementation detail behind DeleteAccount use case).
- Root surface enum: activate reserved `versionLock` from 001.
- **004 new**: `ReconcileReminder` use case + call from `BrenzApp` / `scenePhase` on app open (FR-017).

## Must not create

- Second `firestore.rules` / parallel rules package
- Duplicate Nickname / Discipline / Reminder components or Update* use cases (including duplicating 002 `CurrentDisciplineResolverTests`)
- Second MessageRotationStore or LaunchMarker API
- New success-mark / translucent sheet components
- Password recovery UI inside Settings
- Test suites outside `BrenzTests/Domain`, `BrenzTests/Data`, `BrenzTests/Presentation`
