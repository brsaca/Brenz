# Brenz — Architecture: Data

Date: 2026-11-26
Status: closed by Bren. No code.
Repo location: `docs/architecture/brenz-architecture-data.md`
Base: `brenz-architecture-base.md`. Contracts: `brenz-architecture-domain.md`.

## Principles
- [DECIDED] Firestore is the only data source. With offline persistence.
- [DECIDED] Firebase is a replaceable implementation behind the Domain protocols.
- [DECIDED] No Firebase type leaves `Data/`.

## Firestore structure
```
users/{uid}                     profile
users/{uid}/classLogs/{logId}   one class
users/{uid}/waterLogs/{logId}   one glass
```
- `users/{uid}`: `characterId`, `petNickname`, `stageReached`, `waterGoal`, `onboardingCompleted`, `selectedDisciplines`, `reminder`.
- `classLogs`: `discipline`, `elementId`, `registeredAt`.
- `waterLogs`: `registeredAt`.
- The day's water total is the count of glasses of the local day. "−" deletes the most recent glass of the day.
- Week reads use range queries on `registeredAt`.
- [DECIDED] Water only for today. The app does not add glasses for previous days. The rules do not enforce it.
- [DECIDED] Water has no limit on time or number of glasses. The time of each glass is stored for future statistics.

## DTO and mapping
- [DECIDED] One DTO per document, in `Data/Firebase/`, with native Firestore `Codable`.
- [DECIDED] Each DTO has `toEntity()` and `init(entity)`.
- [DECIDED] `@DocumentID var id: String?` on read. `toEntity()` throws an error if the `id` is nil. The repository discards that document and logs it. No fallback `UUID` is invented.
- [DECIDED] On create, the id is generated on the client with `document().documentID`.
- [DECIDED] Enums are stored as stable text (example: `"boxing"`). The DTO declares them as `String`. `toEntity()` converts with a fallback (`.unknown`).
- [DECIDED] Dates are plain `Date`. No manual `Timestamp`.
- [DECIDED] `registeredAt` uses the device time. Never `@ServerTimestamp`: it is nil while offline. `@ServerTimestamp` stays for audit fields. V1 does not use them.
- [DECIDED] The local day is calculated on the device with `Calendar`.

## Firestore security rules
- [DECIDED] Everything denied by default. Only the owner with a session has access (`request.auth.uid == uid`).
- Profile: `petNickname` maximum 10 characters; `waterGoal` from 4 to 16; `stageReached` only goes up.
- `classLogs` on create: `discipline` among the 4 valid; `elementId` from 1 to 7; `registeredAt` not greater than server time plus 1 day.
- `classLogs` on edit: only `discipline` changes. Delete allowed.
- `waterLogs` on create: only the `registeredAt` field, with the same 1-day cap. No edit. Delete allowed. No limit on glasses or time.
- No reads across users or across collections.
- Rules file and emulator tests in `firebase/`.

## Remote Config
- [DECIDED] One key per value. Proposed names: `rules_animated_threshold`, `rules_window_days`, `rules_down_days`, `rules_lazy_days`, `rules_stage2_classes`, `rules_water_goal_default`, `min_version`.
- [DECIDED] Default values in code (`RulesConfig.default`). The app works offline.
- [DECIDED] Background fetch on app open. It does not block the splash.
- [DECIDED] Minimum interval: 12 h in production, 0 in debug.
- [DECIDED] New values apply on the next state calculation. No forced reload.
- [DECIDED] If the fetch fails, the last activated value or the default is used.
- [DECIDED] `RulesConfigRepository` validates. An out-of-range value is replaced by the default.
- `min_version`: text with the minimum version. Default `"0.0.0"`. Invalid: `"0.0.0"`.
  - It can take up to 12 h to apply. Offline it cannot be forced.
  - Never raise `min_version` before Apple approves that version.
  - The lock screen with a link to the App Store is optional (first cut).

## Authentication
- [DECIDED] Firebase Auth, email with password only. No guest mode. No Apple in v1.
- [DECIDED] `AuthRepository` exposes the session as `AsyncStream<AuthUser?>`.
- [DECIDED] Minimum password of 8 characters, validated in Domain before calling Firebase.
- [DECIDED] The session lives in the iOS keychain. It survives uninstalling the app. After reinstalling on the same iPhone, the user lands on Home and Firestore restores the progress.
- [DECIDED] Firebase error codes are translated to `AuthError` inside Data.

## Offline writes
- [DECIDED] Optimistic writes. The repository writes to the local cache and returns without waiting for the server.
- [DECIDED] Reads emit from the cache instantly.
- [DECIDED] If the server rejects a write, the error is logged. The user is not notified in v1.
- [DECIDED] `DeleteAccount`, `ChangePassword` and `SignIn` do wait for the server. Offline they show "no network".

## Sign out
Order:
1. Wait for pending writes up to 3 s (`waitForPendingWrites`).
2. Sign out of Auth.
3. Clear the Firestore cache (`terminate` and `clearPersistence`).
4. Delete the user's `UserDefaults`, including message rotation.
5. Cancel the reminders.
6. Rebuild the repositories and go back to Start.
- [ACCEPTED] If there is no network and classes remain unsynced, those classes are lost. It is rare. No alert, per the scope.

## Delete account
Order:
1. Reauthenticate with the current password.
2. Delete the Firestore data (`classLogs`, `waterLogs`, profile). Firestore does not delete subcollections on its own: delete in batches.
3. Delete the Auth user.
4. Same steps 3 to 6 as sign out.
- If it fails midway, the user retries. Data is deleted first so no orphans are left.

## Local (`Data/Local/`)
- `MessageRotationStore`: `UserDefaults`, per user. Stores used indexes per state, last index per water group and "once" flags (new user, midday of the day).
- `LocalCatalogProvider`: data written in code. No JSON or Firestore.
- `ReminderScheduler`: `UNUserNotificationCenter`.
  - Each chosen day is a repeating weekly notification with its own identifier.
  - Permission is requested only when the reminder is turned on.
  - One fixed text from the catalog.
  - On app open, it cancels and reschedules according to the profile.
  - With permission denied, Settings shows the switch off, a notice and a shortcut to system Settings.
- `hasLaunchedBefore` in `UserDefaults`, unrelated to the user. It decides full or short splash.

## Constraints
- Do not import Firebase outside `Data/Firebase/`.
- Do not use `@ServerTimestamp` in `registeredAt`.
- Do not wait for the server on class or water writes.
- Do not store message rotation in Firestore.
- Do not add Analytics or Crashlytics.
- Do not invent an `id` when it is missing.

## Acceptance criteria
- With the iPhone in airplane mode, registering a class and glasses updates the tatami and the bottle instantly.
- Emulator tests: no session denies everything; another user denies; editing `registeredAt` denies; lowering `stageReached` denies; creating a class with an invalid `discipline` denies.
- Deleting an account with email works end to end and leaves Firestore and Auth without the user's data.
- Signing out leaves the app on Start with no local data or notifications.
- An out-of-range Remote Config value does not change the behavior.
