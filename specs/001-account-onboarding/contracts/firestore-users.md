# Contract: Firestore `users/{uid}` (onboarding subset)

**Feature**: 001-account-onboarding  
**Date**: 2026-11-27  
**Path**: `users/{uid}`  
**Rules file**: `firebase/firestore.rules` (single file; extend existing — do not create a second rules file)

## Document shape

```json
{
  "characterId": "fox",
  "petNickname": "Bren",
  "stageReached": 1,
  "waterGoal": 8,
  "onboardingCompleted": false,
  "selectedDisciplines": ["boxing", "jiu_jitsu"],
  "lastDiscipline": null,
  "reminder": {
    "isEnabled": false,
    "days": [],
    "timeHour": 0,
    "timeMinute": 0
  }
}
```

Nullable / omit rules:
- `petNickname`: omit or empty string when unused; max 10 characters when present.
- `lastDiscipline`: optional; omit/null until a later feature writes it. Valid values when set: `boxing` | `jiu_jitsu` | `muay_thai` | `kickboxing`.
- `selectedDisciplines`: non-empty array; only the four valid strings; **must be stored in catalog order** (`boxing`, `jiu_jitsu`, `muay_thai`, `kickboxing`).

## Write producers in 001

| Action | Effect |
|--------|--------|
| SignUp | Creates profile with `onboardingCompleted: false`, default `characterId` (`"fox"`), `stageReached: 1`, default `waterGoal`, empty/omitted `petNickname`, `selectedDisciplines: ["boxing"]`, `lastDiscipline: null`. At-least-one discipline has **no exception**. CompleteOnboarding may later replace the selection. |
| CompleteOnboarding | Sets `petNickname` (optional; UI copy: nickname), `selectedDisciplines` (catalog-ordered, ≥ 1), reminder, `onboardingCompleted: true` |
| UpdateNickname | Updates `petNickname` only (shared with Settings) |
| UpdateSelectedDisciplines | Replaces `selectedDisciplines` in catalog order (shared with Settings) |
| UpdateReminder | Replaces `reminder` object (shared with Settings) |

## Read mapping

| Stored | Domain |
|--------|--------|
| known discipline string | matching `Discipline` case |
| any other discipline string | `.unknown` (never written back) |
| missing `lastDiscipline` | `nil`; consumers fall back to first of `selectedDisciplines` in catalog order |

## Security rules (expectations for emulator tests)

- Deny all by default.
- Allow read/write only when `request.auth.uid == uid`.
- `petNickname` length ≤ 10 when present.
- `waterGoal` in 4…16.
- `stageReached` must not decrease on update.
- `selectedDisciplines` elements ⊆ {`boxing`,`jiu_jitsu`,`muay_thai`,`kickboxing`}.
- `lastDiscipline` when present must be one of the four valid values.

## Non-goals

- `classLogs` / `waterLogs` subcollections are not written by this feature.
- Cross-user reads/writes remain denied.
