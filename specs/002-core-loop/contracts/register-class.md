# Contract: RegisterClass / Undo / hour limit

**Feature**: 002-core-loop  
**Layer**: Domain use case (+ Presentation notice mapping)

## Inputs

| Input | Source |
|-------|--------|
| `discipline` | Resolved current discipline (lastDiscipline or first selected in catalog order) |
| `now` | Injected `Date` |
| `calendar` | Injected `Calendar` |
| Today’s existing `ClassLog`s | `TrainingRepository` |
| Profile | `ProfileRepository` |
| `RulesConfig` | For stage threshold when persisting `stageReached` |

## Outcomes

| Outcome | Writes | Undo | UI |
|---------|--------|------|-----|
| `registered(ClassLog, eventMessage?)` | Create `classLogs` doc; set `lastDiscipline`; maybe bump `stageReached` | Yes, 5 s | Bounce / board / optional class-event message |
| `alreadyLoggedThisHour` | **Nothing** | **No** (and must not dismiss an open Undo for a prior success) | Toast key `home.notice.alreadyLoggedThisHour` (2.5 s, no action) — not a character message |

## Stored fields on success

```text
classLogs/{id}:
  discipline: <valid stored value>
  elementId: BoardElementId.compute(date: now, calendar:)  // 1…7
  registeredAt: now

users/{uid}:
  lastDiscipline: <same discipline>
  stageReached: max(existing, computed)   // only if increased
```

## Element function

```text
BoardElementId.compute(date:calendar:) -> Int
  reference Monday: 2026-01-05
  elementId = ((dayIndex + weekIndex) mod 7) + 1
  no Date() / Calendar.current inside
  weekIndex not stored
  cut 0.5: same function, internal weekIndex = 0 (single switch; no separate task)
```

On success from Home, `registeredAt` MUST fall on the device’s **today** local calendar date.

## Hour rule

- Same local calendar day + same `calendar.component(.hour, from:)` → reject.
- Distinct hours (e.g. 14:59 and 15:00) → both accepted.
- After `UndoClass` or `DeleteClass` removes the log for that hour → hour is free.

## Related use cases

| Use case | Contract note |
|----------|----------------|
| `UndoClass` | Deletes last registered log within ViewModel 5 s window; frees hour; does not lower `stageReached` |
| `DeleteClass` | In 002 because Undo needs it; frees hour; does not lower `stageReached` |
| `ChangeClassDiscipline` | **Deferred to 003** (updates `discipline` only; does not change `elementId` or `lastDiscipline`) |
| `UpdateLastDiscipline` | Home sheet selection: profile write only |

## Ordering

When comparing/sorting a day’s logs (including collision display in History later):  
`registeredAt` ascending, then `id` ascending.
