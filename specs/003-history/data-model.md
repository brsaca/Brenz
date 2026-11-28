# Data Model: History (003)

**Branch**: `003-history` | **Date**: 2026-11-27  
**Spec**: [spec.md](./spec.md) | **Research**: [research.md](./research.md)

This feature does **not** introduce new Firestore collections or fields. It consumes and updates existing entities from 001/002 under the stored-values contracts.

---

## Persisted entities (reuse)

### ClassLog

| Field | Type | Notes |
|-------|------|--------|
| `id` | `String` | Document id; non-optional in Domain |
| `discipline` | `Discipline` | Only field editable from History (`ChangeClassDiscipline`) |
| `elementId` | `Int` (1…7) | Fixed at registration; **never** changed by discipline edit |
| `registeredAt` | `Date` | Device local instant; **never** edited from History |

**Validation / rules**:
- At most one class per local clock hour at registration (002); History still orders by `registeredAt` ↑ then `id` ↑ when offline duplicates exist.
- Firestore update: only `discipline` among the four valid stored values.
- Delete allowed; frees the clock hour for a later `RegisterClass`.

### WaterLog

| Field | Type | Notes |
|-------|------|--------|
| `id` | `String` | |
| `registeredAt` | `Date` | Used for day total + hour-block chart only |

**History behavior**: read-only aggregate for the chosen local day. No create/update/delete from History.

### UserProfile (relevant fields)

| Field | Type | History interaction |
|-------|------|---------------------|
| `stageReached` | `Int` (1…2) | **Unchanged** by delete |
| `lastDiscipline` | `Discipline?` | **Unchanged** by `ChangeClassDiscipline` |

Pet **state** is not stored; recalculated after delete from remaining classes (+ water rules elsewhere).

**Log sheet disciplines**: The sheet always shows the four fixed disciplines (`boxing`, `jiu_jitsu`, `muay_thai`, `kickboxing`), as in the mockups—not filtered by profile `selectedDisciplines`.

---

## Derived / Domain models (not stored)

Domain types MUST NOT hold localized text. Presentation/ViewModel maps pure values to `Localizable.xcstrings`.

### WeekHistorySnapshot

Return value of `ObserveWeekHistory`. Assembled for the current local Monday–Sunday week. Carries **week data only**—no selected day. The ViewModel owns the selected day (default today) and builds `DayViewSnapshot` from this snapshot.

| Field | Description |
|-------|-------------|
| `days` | Exactly seven `WeekDayCell` values (flags via `kind` / `isSelectable`) |
| `classLogsByDay` | Per-day ordered `[ClassLog]` (keys = local calendar days; order = `registeredAt` ↑ then `id` ↑) |
| `waterLogsByDay` | Per-day `[WaterLog]` for totals and hour-block charts |

### WeekDayCell

One of seven days in the current Monday–Sunday local week.

| Field | Description |
|-------|-------------|
| `date` | Local calendar day |
| `dayNumber` | Display number |
| `kind` | `today` \| `pastWithTraining` \| `pastWithoutTraining` \| `future` |
| `isSelectable` | Canonical flag: `false` when `future`; else `true` |

### DayLabelKind

Pure Domain value for the chosen-day heading. **Not** a localized string.

| Case | Meaning |
|------|---------|
| `today` | Chosen day is the current local calendar day |
| `date(Date)` | Chosen day is another local calendar day (Presentation formats via catalog / date style) |

Presentation/ViewModel localizes `DayLabelKind` through `Localizable.xcstrings` (e.g. “Today” / “Hoy”, or a weekday-date string). Domain never returns “Today”/“Hoy” text.

### DayViewSnapshot

Chosen day’s assembled UI state.

| Field | Description |
|-------|-------------|
| `dayLabel` | `DayLabelKind` (`today` or `date(Date)`); localized only in Presentation |
| `board` | `BoardLayout` result (rows, filled cells ≤12, overflow `+N`) |
| `classTotal` | Count of class logs that day |
| `waterTotal` | Count of water logs that day |
| `classBlocks` | Six hour-block counts |
| `waterBlocks` | Six hour-block counts |

### BoardLayout (pure)

**Inputs**: ordered `[ClassLog]` for the day, `BoardTheme` (columns=3, maxCells=12), `minimumRows` (History=3).  
**Outputs**: `rowCount`, `filledCells` (first 12 by order), `overflowCount`.  
**Invariant**: size/rowCount derived every emission; never written to Firestore.

### HourBlock (pure)

Six buckets labeled `6a, 9a, 12p, 3p, 6p, 9p`.

| Local time | Bucket |
|------------|--------|
| `00:00`–`05:59` | 6a |
| `[06:00`–`09:00)` | 6a |
| `[09:00`–`12:00)` | 9a |
| `[12:00`–`15:00)` | 12p |
| `[15:00`–`18:00)` | 3p |
| `[18:00`–`21:00)` | 6p |
| `[21:00`–`24:00)` | 9p |

Each log contributes to exactly one bucket and to the day total.

### LogSheetSession (transient UI)

| State | Meaning |
|-------|---------|
| `edit` (6b) | Show element, discipline chips, time, Save, Delete record |
| `confirmDelete` (6c) | Confirm destructive delete |
| `deletedSuccess` (6d) | Success mark + remaining-count figure (~2.5 s) then dismiss |
| `closed` | Sheet dismissed; History shows refreshed day (6e) |

Save enabled iff `selectedDiscipline != storedDiscipline`.

---

## Relationships

```text
UserProfile 1──* ClassLog
UserProfile 1──* WaterLog
ObserveWeekHistory ──► WeekHistorySnapshot
  ├── days: [WeekDayCell × 7]
  ├── classLogsByDay
  └── waterLogsByDay
HistoryViewModel (owns selected day, default today) ──► DayViewSnapshot (dayLabel: DayLabelKind)
DayViewSnapshot.board.filledCells ──tap──► LogSheetSession(edit)
ChangeClassDiscipline ──► ClassLog.discipline only
DeleteClass ──► remove ClassLog; PetState recalculated; stageReached unchanged
Presentation ──localizes──► DayLabelKind via Localizable.xcstrings
```

---

## State transitions

### Discipline change

```text
[edit, Save inactive] --select different discipline--> [edit, Save active]
[edit, Save active] --Save--> write discipline --> close sheet --> DayViewSnapshot refresh
[edit] --select same as stored--> Save inactive (no write)
[edit] --dismiss--> discard selection (no write)
```

**Guarantees**: `elementId`, `registeredAt`, `lastDiscipline` identical before/after.

### Delete

```text
[edit] --Delete record--> [confirmDelete]
[confirmDelete] --Cancel--> [edit]
[confirmDelete] --Confirm--> delete log --> [deletedSuccess] (or skip if Reduce Motion) --> closed
```

**Guarantees**: `stageReached` unchanged; pet state matches calculators on remaining logs; hour free; no message-bank emission; user remains on same chosen day.

---

## Ordering rules

For any day’s class list (board cells, overflow promotion, same-hour ties):

1. `registeredAt` ascending  
2. `id` ascending  

Filled tatami cells = first 12 of that ordered list. Overflow = remainder (`+N`), not tappable until a delete promotes them.

---

## Localization keys (figure 6d)

| Situation | `es` | `en` |
|-----------|------|------|
| Today, count | `{count} clases hoy` | `{count} classes today` |
| Other day, count | `{count} clases ese día` | `{count} classes that day` |

String Catalog must define plural variants for **0**, **1**, and **n** in both languages. Keys are English identifiers (e.g. `history.deleted.classesToday`, `history.deleted.classesThatDay`); exact key names finalized at implement/tasks time.
