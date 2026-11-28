# Data Model: Settings and Account Management

**Feature**: 004-settings-account  
**Date**: 2026-11-27  
**Sources**: [spec.md](./spec.md), [research.md](./research.md), `docs/architecture/brenz-architecture-stored-values.md`, domain/data architecture

## Entities (reuse + Settings focus)

### AuthUser (session)

| Field | Type | Notes |
|-------|------|-------|
| id | String | Auth uid |
| email | String | Display-only in Settings Account section |

Session required for Settings. Ended by Sign Out or Delete Account → root returns to Start (welcome).

### UserProfile (preferences editable in Settings)

| Field | Type | Validation / Notes |
|-------|------|--------------------|
| id | String | Same as Auth uid |
| characterId | String | Not edited in Settings v1 |
| petNickname | String? | Optional; max 10; empty hides nickname line; never translated; **UI copy: nickname** |
| stageReached | Int | Not lowered by Settings |
| waterGoal | Int | **4…16**; default 8 until user edits |
| onboardingCompleted | Bool | Must be true to reach Settings via Home tabs |
| selectedDisciplines | [Discipline] | ≥1; removing last used falls back to first in fixed catalog order |
| lastDiscipline | Discipline? | May become nil / invalid relative to selection → Home uses first selected |
| reminder | ReminderSettings | Off → empty schedule; on → days (Mon=1…Sun=7) + time; permission only when turning on |

### ReminderSettings

| Field | Type | Notes |
|-------|------|-------|
| isEnabled | Bool | False when permission denied or user off |
| days | [Int] | 1…7 when enabled |
| timeHour | Int | Local hour |
| timeMinute | Int | Local minute |

### MinimumVersionPolicy

| Field | Type | Notes |
|-------|------|-------|
| minVersion | String | From RulesConfig / Remote Config `min_version`; default `"0.0.0"` |
| installedVersion | String | Bundle short version |
| isLocked | Bool | Pure: `VersionComparator` says installed < valid min |

Invalid/missing min → treat as `"0.0.0"` → not locked.

### LocalDeviceState

| Store | Cleared on Sign Out / Delete? | Notes |
|-------|-------------------------------|-------|
| MessageRotationStore (per user) | **Yes** — `clear()` | Required by plan notes |
| Firestore local cache | Yes | terminate + clearPersistence |
| Reminder schedules | Yes — cancel all | |
| LaunchMarkerStore (`hasLaunchedBefore`) | **No** | Not user data; short splash / Start path |

## Relationships

```text
AuthUser 1 ── observes ──► UserProfile
UserProfile ── selectedDisciplines ──► Discipline (1..4)
UserProfile ── waterGoal ──► Home water UI (002)
DeleteAccount ── deletes ──► users/{uid} + classLogs + waterLogs + Auth user
SignOut / DeleteAccount ── clear ──► MessageRotationStore (not LaunchMarkerStore)
RulesConfig.minVersion + installedVersion ──► versionLock root surface
```

## Validation rules

| Rule | Enforcement |
|------|-------------|
| Nickname ≤ 10 | Field + UpdateNickname |
| ≥1 discipline | DisciplineMultiSelect + UpdateSelectedDisciplines |
| Water goal 4…16 | UpdateWaterGoal + existing Firestore rules |
| Change Password enable | All fields filled ∧ new.count ≥ 8 ∧ new == repeat |
| Delete confirm enable | Password field non-empty |
| No “new equals current” check | Explicitly out of v1 |

## State transitions

### Sign Out

```text
signedIn(Home/Settings)
  → attempt pending sync (≤ **3 s**)
  → Auth signed out
  → clear Firestore cache + MessageRotationStore + reminders
  → rebuild repos
  → Start (welcome)
  LaunchMarkerStore unchanged
```

### Delete Account

```text
Settings → sheet 7b (password)
  → wrong password → stay 7b + error 7e (no deletes)
  → correct password:
       reauth → delete Firestore (logs then profile) → delete Auth user
       → local cleanup like Sign Out
       → sheet 7c (**2.5 s** or Reduce Motion static)
       → Start
```

### Change Password

```text
Settings → 7d
  → invalid enable rules → button inactive (± 7h on mismatch)
  → wrong current → stay 7d + 7g
  → success → 7i (**2.5 s**) → dismiss → Settings remains
```

### Reminder reconcile (004)

```text
app open / scenePhase active
  → ReconcileReminder
  → permission revoked → reminder off + cancel schedules
  → permission granted ∧ reminder on → cancel + reschedule from profile
```

### Version lock

```text
any root surface
  → cold start or foreground
  → if installed < valid minVersion → versionLock (blocks app)
  → else continue normal 001 precedence
```

## Use cases (004 ownership)

| Use case | Origin | Notes |
|----------|--------|-------|
| UpdateNickname | 001 reuse | Optimistic; field `petNickname` (UI copy: nickname) |
| UpdateSelectedDisciplines | 001 reuse | Keeps ≥1; lastDiscipline fallback gated by 002 `CurrentDisciplineResolverTests` |
| UpdateReminder | 001 reuse | Permission only when enabling; **001 schedules** on enable |
| ReconcileReminder | **004 new** | On app open (`BrenzApp` / `scenePhase`); revoke → off; granted+on → rebuild schedule |
| UpdateWaterGoal | **004 new** | Bounds 4…16; optimistic; Home reads `waterGoal` from profile stream |
| ChangePassword | 004 (Domain Account) | Wait **10 s**; no recovery |
| DeleteAccount | 004 (Domain Account) | Reauth → Firestore → Auth; wait **10 s** |
| SignOut | 004 (Domain Account) | **3 s** pending writes; no alert; `MessageRotationStore.clear()` |

## Out of model (v1)

- Change email; Sign in with Apple; password recovery from Settings
- “New password equals current” error entity
- Second Firestore rules document or forked preference components
