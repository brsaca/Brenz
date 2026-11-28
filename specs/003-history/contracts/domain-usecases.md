# Contract: Domain use cases (History)

**Feature**: 003-history  
**Consumers**: `HistoryViewModel`, `LogSheetViewModel` via `AppContainer`  
**Dependencies**: Existing `TrainingRepository`, `ProfileRepository` protocols (no new protocols). `ChangeClassDiscipline` validates with the `Discipline` enum only—no `RulesConfigRepository` or `CatalogProvider`.

---

## ObserveWeekHistory

```text
execute(uid: String, now: Date, calendar: Calendar) -> AsyncStream<WeekHistorySnapshot>
```

**Behavior**:
- Emits immediately from local cache, then on class/water/profile changes for the current local week.
- Week = Monday 00:00 … Sunday end (device calendar). No adjacent weeks.
- Each emission is a `WeekHistorySnapshot` with **week data only**: seven `WeekDayCell` values (`isSelectable` false for future) and per-day ordered class/water logs. No `selectedDay` / `DayViewSnapshot` in the stream.
- Class lists per day are ordered by `registeredAt` ascending, then `id` ascending.

**Out of contract**: Loading UI; the ViewModel owns the selected day (default today), builds `DayViewSnapshot` (including `DayLabelKind`) from week data, and localizes `DayLabelKind` via `Localizable.xcstrings`.

---

## ChangeClassDiscipline

```text
execute(uid: String, logId: String, newDiscipline: Discipline) async throws
```

**Error style** (match 001/002 write use cases / architecture): Domain throws `DataError` (e.g. `.notFound`, `.unknown`) for missing log or invalid discipline; Presentation publishes an optional `AppError` on the ViewModel. No `Result` wrapper.

**Preconditions**:
- `newDiscipline` is one of the four valid `Discipline` cases (`boxing`, `jiu_jitsu`, `muay_thai`, `kickboxing`; never `.unknown`). Validation uses the `Discipline` enum only.
- Target `ClassLog` exists.

**Effects**:
- Writes **only** `discipline` on `users/{uid}/classLogs/{logId}`.
- Does **not** modify `elementId`, `registeredAt`.
- Does **not** read or write `UserProfile.lastDiscipline`.
- Optimistic: returns after local persistence; server rejection logged only in Data (v1)—not thrown as a user-facing error.

**Non-effects**: No pet-state message; no stage change; no hour change.

---

## DeleteClass (reuse from 002)

```text
execute(uid: String, logId: String, now: Date, calendar: Calendar) -> DeleteClassResult
```

**Effects** (History-relevant):
- Removes the class log (optimistic).
- Pet state is recalculated from remaining classes (callers may observe via `ObservePetStatus` / week stream).
- `stageReached` is never decreased.
- The local clock hour of the deleted log becomes free for a future `RegisterClass`.
- Does **not** emit a character `MessageKey` for delete.
- Does **not** change `lastDiscipline`.

**Presentation follow-up (not Domain)**: 6d success UI / Reduce Motion skip; stay on chosen day.

---

## Pure rules used by History

### BoardLayout

```text
layout(classes: [ClassLog], theme: BoardTheme, minimumRows: Int) -> BoardLayoutResult
```

- History calls with `minimumRows = 3`.
- Cap filled cells at `theme.maxCells` (12); `overflowCount = max(0, classes.count - 12)`.
- Filled subset = first `maxCells` of the already-ordered `classes` array.

### HourBlockBucket

```text
bucket(for date: Date, calendar: Calendar) -> HourBlock  // six cases / indices
```

- Half-open windows per [data-model.md](../data-model.md); `00:00–05:59` → 6a.
- Same function for classes and water.

### Ordering helper (if extracted)

```text
sortedForDay(_ classes: [ClassLog]) -> [ClassLog]
```

- `registeredAt` ↑, then `id` ↑.
