# Data Model: Account Onboarding

**Feature**: 001-account-onboarding  
**Date**: 2026-11-27  
**Sources**: [spec.md](./spec.md), `docs/architecture/brenz-architecture-stored-values.md`, domain/data architecture

## Entities

### AuthUser

| Field | Type | Notes |
|-------|------|-------|
| id | String | Firebase Auth uid; non-optional |
| email | String | Account email |

Represents the active session. Absence of `AuthUser` means the `welcome` surface (Start screen first).

### UserProfile

| Field | Type | Notes |
|-------|------|-------|
| id | String | Same as Auth uid |
| characterId | String | v1: `"fox"` |
| petNickname | String? | Optional; max 10 characters; never translated; empty/nil omitted on Home later; UI copy: nickname |
| stageReached | Int | 1 or 2; SignUp seeds `1` |
| waterGoal | Int | 4…16; SignUp seeds default (8 from RulesConfig.default) |
| onboardingCompleted | Bool | `false` after SignUp; `true` after CompleteOnboarding |
| selectedDisciplines | [Discipline] | At least one (**no exception**); stored/persisted in catalog order; SignUp seeds `[boxing]` |
| lastDiscipline | Discipline? | Optional; SignUp seeds `nil`; not set by onboarding; fallback = first of `selectedDisciplines` in catalog order |
| reminder | ReminderSettings | Disabled/empty when reminder is off |

**Relationships**: 1:1 with `AuthUser` via `id`. **Missing profile document** after a successful load path with session → treat as onboarding incomplete (open Onboarding step 1). **Profile fetch failure** (network/server) → `profileLoadError` (retryable), session kept — not Onboarding, not Home.

### ReminderSettings

| Field | Type | Notes |
|-------|------|-------|
| isEnabled | Bool | Permission requested only when turning on |
| days | [Int] | 1…7, Monday = 1 … Sunday = 7 |
| timeHour | Int | Local hour when enabled |
| timeMinute | Int | Local minute when enabled |

When disabled: `isEnabled == false` and days may be empty; scheduler cancels pending notifications.

### Discipline

| Case | Stored raw value | In pickers |
|------|------------------|------------|
| boxing | `boxing` | yes (default preselected) |
| jiuJitsu | `jiu_jitsu` | yes |
| muayThai | `muay_thai` | yes |
| kickboxing | `kickboxing` | yes |
| unknown | — (read-only mapping) | never |

- `allCases` / catalog order for UI and storage sort: boxing, jiu_jitsu, muay_thai, kickboxing.
- Unknown stored text → `.unknown` on read; never written back.
- Multi-select requires ≥ 1 valid discipline to continue.

### LaunchMarker

| Field | Storage | Notes |
|-------|---------|-------|
| hasLaunchedBefore | UserDefaults (device) | Not user data; set once when the full splash finishes the first time; survives Sign Out and Delete Account; selects splash kind / whether Start screen leads to Intro only (not post-splash destination) |

### AuthFailure (domain error kinds used by this feature)

| Kind | When |
|------|------|
| invalidCredentials | Sign In wrong email/password |
| emailAlreadyInUse | Sign Up existing email |
| weakPassword | Sign Up only — password policy from server / client min length |
| invalidEmail | Malformed email on Sign Up / Reset Password |
| noNetwork | Offline or **10 s** timeout on SignUp / SignIn / ResetPassword (timeout maps to noNetwork by design; injected monotonic clock) |
| profileLoadFailed | Session present; profile **fetch/observe failed** (retryable; not an Auth sign-out). Distinct from a missing profile document, which routes to Onboarding step 1. |

Client-side: Sign Up password length < 8 → `weakPassword` before network. Sign In MUST NOT return `weakPassword`; any client-side format failure on Sign In (including password length < 8 or malformed email) → `invalidCredentials`.

## Validation rules

1. **Nickname**: optional; length 0…10; truncate or block input beyond 10; no localization.
2. **Password**: minimum 8 characters before network call.
3. **Email**: non-empty valid format before network call on Sign Up / Sign In / Reset Password.
4. **Disciplines**: ≥ 1 selected (**no exception**, including SignUp seed `[boxing]`); persist sorted by catalog order; default selection = boxing.
5. **Reminder**: optional; if enabled, require at least one day and a time; request notification permission only on enable; permission denied → force off + notice + link to system Settings; onboarding may still complete.
6. **onboardingCompleted**: only Domain `CompleteOnboarding` sets `true` after steps 1–3 succeed.
7. **lastDiscipline**: optional; if nil or not ∈ selectedDisciplines, consumers use first selected in catalog order.

## State transitions

### Root surface (session × profile × launch × future lock)

```text
[App launch]
    → splash (full if !hasLaunchedBefore, else short)  // marker = splash kind only
    → wait until session known (and profile if session)
         │
         ├─ (future) versionLock active → versionLock  // wins over all; unused in 001
         ├─ no session → welcome (Start screen → Intro → SignUp | SignIn → ResetPassword)
         ├─ session + profile fetch failed → profileLoadError (retry; keep session)
         ├─ session + (missing profile document | !onboardingCompleted) → onboarding step 1
         └─ session + onboardingCompleted → home
```

Session decides the destination. `hasLaunchedBefore` never routes a surviving session to the Start screen (including reinstall). Splash does not advance early if resolution exceeds **1.5 s** / **0.8 s** minimums (injected monotonic clock; hold rule = FR-006 only).

### Session / onboarding

```text
SignedOut ──SignUp success──► SignedIn(onboardingCompleted=false) ──CompleteOnboarding──► SignedIn(true)
SignedOut ──SignIn success──► SignedIn(flag from profile) ──► Onboarding or Home
ResetPassword ──never──► SignedIn
```

### Abandoned onboarding

Kill/background mid-onboarding with session → next cold start: short or full splash per marker → Onboarding step 1 (not Home, not Sign Up).

### Launch marker

```text
hasLaunchedBefore=false ──full splash finishes (first time)──► true
Sign Out / Delete Account ──do not──► clear marker
```

## Persistence map

| Concept | Location |
|---------|----------|
| Session | Firebase Auth (Keychain) |
| Profile | Firestore `users/{uid}` |
| Launch marker | `UserDefaults` device key |
| Reminder schedules | `UNUserNotificationCenter` via `ReminderScheduler` |
| Message rotation | out of scope (002+) |

## Out of model for this feature

- `ClassLog`, `WaterLog`, board/pet calculators (later features)
- Settings account actions (Change Password, Sign Out, Delete Account) — 004 consumes shared nickname/discipline/reminder pieces only
- History / Settings navigation shells — 002
