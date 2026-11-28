# Research: History (003)

**Branch**: `003-history` | **Date**: 2026-11-27  
**Spec**: [spec.md](./spec.md) | **Plan**: [plan.md](./plan.md)

All Technical Context items are resolved. No `NEEDS CLARIFICATION` markers remain.

---

## R1. Ownership of ChangeClassDiscipline vs DeleteClass

**Decision**: Spec 002 delivers `DeleteClass` (and Home Undo). Spec 003 owns `ChangeClassDiscipline` (moved here from 002’s planned surface) and the full History log sheet that calls both.

**Rationale**: Product log sheet (6b–6e) lives only in History. Delete is required for P1 correction flows and already pairs with Home Undo hour-freeing. Discipline edit is History-only and is cut-order item 6, so it must not block 002.

**Alternatives considered**:
- Implement both writes in 002 → couples Home delivery to History UI.
- Defer DeleteClass to 003 → leaves no path to free a wrong hour from History if 003 slips.

---

## R2. Reuse of Design System and repositories

**Decision**: Reuse without rebuild: `BrenzTabBar`, isometric tatami grid, water bottle, translucent floating sheet, Reduce Motion modifier, and (if present) chart-bar component from `Packages/BrenzDesignSystem`. Reuse `TrainingRepository` / `ProfileRepository` protocols and implementations, ClassLog/WaterLog DTOs, and the single `firebase/firestore.rules`. Place `BrenzSuccessMark` in the design system package if 001/002 did not already add it—003 is the first consumer; 004 reuses it for account/password success.

**Rationale**: Constitution II (one local package) and design-system decision forbid duplicating tab bar/grid/bottle/sheet. Data architecture forbids a second rules file and duplicate protocols. Stored-values already constrain `classLogs` update to `discipline` only.

**Alternatives considered**:
- Rebuild History-specific grid/bottle → violates “do not rebuild” and drifts tokens.
- New `HistoryRepository` → duplicates TrainingRepository observe/update/delete.
- Separate `firestore.rules.history` → split source of truth for security.

---

## R3. Current week only; future days not tappable

**Decision**: `ObserveWeekHistory` (or the History ViewModel assembly) builds exactly seven local-calendar days from Monday 00:00 through Sunday end of the current week. No previous/next controls. Day cells: today (lime border, bold number), past-with-classes (checkmark), past-without (number only), future (dimmed, `isSelectable == false` / no tap handler). Default selected day: today. Canonical flag name: `isSelectable` (see [data-model.md](./data-model.md)).

**Rationale**: Scope §3/§6 and mockups History (6). Soft “proposed” past-day tap in scope is resolved by mockups and the feature spec Assumptions.

**Alternatives considered**:
- Native `TabView` week pager → out of v1; invites other weeks.
- Allow future tap as empty preview → contradicts FR-004.

---

## R4. Tatami size derivation (History)

**Decision**: `BoardLayout` receives the day’s ordered classes, `BoardTheme` (3 columns, maxCells 12), and History `minimumRows = 3`. Filled cells = first 12 by `registeredAt` ascending, then `id` ascending. Overflow count = `max(0, count - 12)` shown as `+N`. Row count = `max(minimumRows, ceil(min(count, 12) / columns))`. Size is never persisted.

**Rationale**: Domain `BoardLayout` AC; scope §6 Board; clarifications on >12 ordering. Home uses min 1 row via the same pure function with a different minimum.

**Alternatives considered**:
- Store row count on the day document → violates derived-size rule and stored-values.
- Different grid component for History → rebuild forbidden; DayBoard in Presentation/Shared wraps the package grid.

---

## R5. Class ordering and hour-block charts

**Decision**:
- Day class list sort: `registeredAt` ascending, then `id` ascending (stable tie-break for offline multi-device same-hour duplicates).
- Chart buckets (Classes and Water identical): half-open local windows `[6–9)`, `[9–12)`, `[12–15)`, `[15–18)`, `[18–21)`, `[21–24)`; labels 6a, 9a, 12p, 3p, 6p, 9p. Times `00:00–05:59` count in 6a. Pure function `HourBlockBucket` (name may vary) takes `Date` + `Calendar`, returns one of six indices. Boundary tests: 08:59 → 6a, 09:00 → 9a.

**Rationale**: Spec clarifications 2026-11-27; FR-007; stored-values ordering.

**Alternatives considered**:
- Closed intervals or center-of-block → ambiguous double-counting.
- Exclude pre-06:00 from charts but keep totals → contradicts clarification and FR-007.

---

## R6. ChangeClassDiscipline semantics

**Decision**: Use case updates only `ClassLog.discipline` via `TrainingRepository.updateDiscipline`. Must not write `elementId`, `registeredAt`, or profile `lastDiscipline`. Presentation: Save enabled only when selected discipline ≠ stored. Dismiss without Save discards selection. Offline optimistic write (existing repository behavior).

**Rationale**: Stored-values “editing discipline does not change elementId / does not write lastDiscipline”; FR-011–FR-012; firestore.rules already: edit only `discipline`.

**Alternatives considered**:
- Also refresh `lastDiscipline` for Home consistency → rejected by stored-values and FR-012 scenario 5.
- Allow time edit in same sheet → out of scope.

---

## R7. DeleteClass side effects

**Decision**: Reuse `DeleteClass` from 002. After delete: remove log; recalculate pet state from remaining classes (and water rules as defined elsewhere); leave `stageReached` unchanged (`StageCalculator` max with stored); free that local clock hour for a future `RegisterClass`; refresh History board/cards on the same chosen day; do not emit a character message. Confirmation copy states stage does not go down.

**Rationale**: Scope §4; domain StageCalculator; FR-014/FR-020; message bank has no delete reaction.

**Alternatives considered**:
- Lower stageReached when class count drops → forbidden.
- Show Home speech bubble on delete → no bank entry; out of feature.

---

## R8. Class-deleted success UI (6d) and Reduce Motion

**Decision**: Same translucent sheet switches content (~2.5 s) then closes. Figure copy:
- Chosen day is today → `"{count} clases hoy"` / `"{count} classes today"`
- Chosen day is another day → `"{count} clases ese día"` / `"{count} classes that day"`
Plural variations in `Localizable.xcstrings` for 0, 1, and n (en + es). Use `BrenzSuccessMark` for checkmark/ring; emptying progress bar from package motion tokens. If `accessibilityReduceMotion` (package modifier from 001): skip animation, close after delete completes.

**Rationale**: Spec assumption on figure copy; mockups 6d; design-system `BrenzSuccessMark`; FR-015. 004 will reuse the success mark for 7c/7i.

**Alternatives considered**:
- Hard-code “0 clases hoy” → wrong for other days and non-zero counts.
- Ship 4 s looping mockup duration → product timing is ≈2.5 s.

---

## R9. Water card on History

**Decision**: Read-only day total + same hour-block chart. No glass list, no +/−. Observe glasses for the chosen local day via existing TrainingRepository range/day observe (from 002).

**Rationale**: FR-008; scope §3 History water is summary-only.

**Alternatives considered**:
- Full water editor on History → out of scope; Home owns add/remove.

---

## R10. Testing placement and required cases

**Decision**: All new tests under `BrenzTests/Domain`, `BrenzTests/Data`, `BrenzTests/Presentation` only. Required coverage:
1. Discipline edit keeps `elementId` and `lastDiscipline`.
2. Delete keeps `stageReached` and recalculates pet state.
3. Same-hour logs ordered by `registeredAt` then `id`.
4. Hour-block bucketing at 08:59 (6a) and 09:00 (9a).
5. Future days not tappable (ViewModel or presentation unit).

**Rationale**: User plan notes; constitution VI; no UI test target in v1.

**Alternatives considered**:
- Snapshot/UI tests for week bar → forbidden in v1.
- New `BrenzTests/History` folder → violates “tests live only in Domain/Data/Presentation.”

---

## R11. Server rejection UX

**Decision**: Optimistic write; if server rejects, log only—no user-facing error in v1 (aligned with data architecture and spec clarification).

**Rationale**: Spec Clarifications 2026-11-27; data offline writes section.

**Alternatives considered**:
- Toast on rejection → deferred beyond v1.
