# Research: Core Loop (002)

**Branch**: `002-core-loop` | **Date**: 2026-11-27  
**Status**: Complete — all plan unknowns resolved from stored-values, architecture, spec clarifications, and plan notes.

## R1. Board element rotation formula

**Decision**: Compute once in `RegisterClass` and store `elementId` (1…7):

`elementId = ((dayIndex + weekIndex) mod 7) + 1`

- `dayIndex`: Monday = 0 … Sunday = 6 from the class local date (via the shared weekday-conversion helper).
- `weekIndex`: whole weeks from reference Monday **2026-01-05** (local calendar) to the Monday of the class week.
- Pure Domain function: `(Date, Calendar) -> Int`. Never calls `Date()` or `Calendar.current`.
- Do **not** store `weekIndex`. Do **not** add Remote Config keys for rotation.
- Cut 0.5: same `BoardElementId.compute` with internal constant `weekIndex = 0` (single switch in T012; no separate task).
- Catalog field is `baseOrder` (1…7), not `weekday` (stored-values precedence over domain doc wording).
- Lookup: `(discipline, elementId)` then `elementId`. Changing discipline later does not change stored `elementId`.

**Rationale**: Same local date → same element on every device; ISO week numbers jump at year boundaries; stored-values is highest precedence.

**Alternatives considered**:
- Fixed weekday→element map (scope §6 / assets table) — rejected; only true in week 0.
- ISO week number — rejected; fails year-change stability.
- Recompute on read — rejected; discipline edits and historical consistency require a stored snapshot.
- Random / Remote Config rotation — rejected by stored-values constraints.

## R2. One class per local clock hour

**Decision**: `RegisterClass` loads today’s class logs, compares local clock hour of `registeredAt` with `now` (both via injected `Calendar`). If a log already occupies that hour, write nothing and return outcome `alreadyLoggedThisHour`. Home shows toast key `home.notice.alreadyLoggedThisHour` (auto-dismiss 2.5 s, no action, no Undo for that rejection; does not hide prior Undo). Undo and `DeleteClass` free the hour. Check runs on device only; **no Firestore rule**. A successful Home registration’s `registeredAt` is always today’s local calendar date.

**Rationale**: Gym offline use; rules cannot see “local hour” portably without storing timezone; product notice is system UI, not pet voice.

**Alternatives considered**:
- Rolling 60-minute window — rejected by stored-values.
- Firestore unique constraint / rule — rejected; device Calendar is source of truth; dual offline devices can still collide (order by `registeredAt` then `id`).
- Character message for rejection — rejected; voice rules / FR-024.

## R3. Weekday conversion (Calendar Sunday=1 → Monday=1)

**Decision**: One Domain function covering all seven weekdays, tested for Mon…Sun. Prefer reusing the helper introduced in 001 for reminder day numbering (e.g. `ReminderWeekday` / Monday-based conversion). Same function feeds reminder days, `dayIndex` (Monday=0 via `mondayBasedWeekday - 1`), and any other Monday-based indexing.

**Rationale**: Single conversion avoids Sunday/Monday drift between reminders and board rotation.

**Alternatives considered**: Duplicate conversions per feature — rejected. Using ISO weekday APIs inconsistently across call sites — rejected.

## R4. `lastDiscipline` writes

**Decision**:
- `RegisterClass` updates `lastDiscipline` on the profile in the **same** operation as creating the class log.
- Selecting a discipline in the Home sheet writes **only** `lastDiscipline` (profile update; no class log).
- `ChangeClassDiscipline` is **moved entirely to feature 003** (not implemented in 002). `DeleteClass` stays in 002 because Undo needs it.
- Resolve current discipline: remembered if still in `selectedDisciplines`; else **first selected in catalog order** (boxing → jiu_jitsu → muay_thai → kickboxing). Kickboxing then boxing → first is boxing.

**Rationale**: Stored-values contract; survives device change (Firestore, not UserDefaults).

**Alternatives considered**: UserDefaults for last discipline — rejected (sign-out / multi-device). First-by-selection-time — rejected (catalog order).

## R5. Optimistic writes and Undo window

**Decision**: Class and water repository methods write to the Firestore local cache and return without awaiting server acknowledgment. HomeViewModel owns the 5 s Undo timer for the last successful registration only. Same-hour rejection must not cancel/hide an open Undo for a prior successful log.

**Rationale**: Constitution V + SC-001/SC-002; gym offline.

**Alternatives considered**: Wait for server then show Undo — rejected. Longer Undo — rejected (product: 5 s).

## R6. Water model

**Decision**: Each glass is `users/{uid}/waterLogs/{id}` with `registeredAt` (device time, never `@ServerTimestamp`). Home +/− only add/remove **today’s** glasses; − deletes the most recent glass of the local day. No daily counter document. No glass-count or time-of-day limit. Bottle visual fill capped at 1.0 when count &gt; goal; fraction text may exceed goal.

**Rationale**: Architecture data + FR-010/FR-011; future History stats need event times.

**Alternatives considered**: Daily aggregate document — rejected. Enforcing “today only” in rules — rejected (device Calendar; rules do not enforce).

## R7. Pet state and stage

**Decision**: `PetState` is always calculated (`PetStateCalculator`); never stored. Bands from `daysSinceLastClass` vs `RulesConfig`:
- **lazy** if `days >= lazyDays`
- **down** if `downDays <= days < lazyDays`
- **active** (derived, not a constant/RC field) if `days < downDays` → animated when classes in `windowDays` ≥ `animatedThreshold`, else normal
- Defaults: animated threshold **2**, window **7**, `downDays` **4**, `lazyDays` **7** → active **0…3** days; stage change **10**; water goal default **8**
- Invalid RC per key → default; if `downDays < 1` or `downDays >= lazyDays` → in-app defaults for that pair
- `stageReached` stored as 1 or 2 and **never decreases**

**Rationale**: Scope §4 + FR-006–FR-008 + FR-021; no new RC field for active.

**Alternatives considered**: Persist pet state — rejected. Separate “activeDays” RC key — rejected (derived from `downDays`). Lower stage on Undo/delete — rejected.

## R8. Messages and rotation

**Decision**: Domain `MessageSelector` returns `MessageKey` only. Rotation and once-flags live in `MessageRotationStore` (UserDefaults, per user). Messages never include nickname. Water messages use Presentation constant `waterMessageDuration = 4` seconds then return to state message; can reopen a closed bubble. Home-open priority among non-water: new-user → midday → state. Class-event messages still win when their trigger just fired.

**Rationale**: Message bank + domain Messages section; constitution VII.

**Alternatives considered**: Store rotation in Firestore — rejected. Choose messages in ViewModel — rejected.

## R9. Design-system components in this feature

**Decision**: Build `BrenzTabBar`, water bottle shape (0…1 fill), and isometric/tatami tile grid in `Packages/BrenzDesignSystem`. Reuse the Reduce Motion modifier from 001. `DayBoard` stays in `Presentation/Shared/` (composes grid + element assets from catalogs).

**Rationale**: Design-system decision + presentation architecture; Home cannot ship without these pieces.

**Alternatives considered**: Native `TabView` — cut #7 only if time-short later; default is custom tab bar. Putting `DayBoard` in the package — rejected (entity-aware).

## R9b. External art assets

**Decision**: Import **8** placeholder pet images into `Assets.xcassets` named `{character}_{stage}_{state}` (`fox`, `s1`/`s2`, `animated`/`normal`/`down`/`lazy`). Board’s **7** element images are external art expected **Sat Nov 28**; until delivery, use **code-drawn placeholder shapes** behind `LocalCatalogProvider`. Documented in plan.md Art dependency.

**Rationale**: Unblocks UI/catalog wiring without waiting on final board art.

**Alternatives considered**: Block Home on final art — rejected. Invent permanent drawn pets — rejected (placeholders then real assets).

## R10. History / Settings shells

**Decision**: Tab destinations navigate to placeholder shells (title / empty body). Features 003 and 004 replace shell content. No History log sheet, charts, or Settings rows in 002.

**Rationale**: Spec clarification + FR-001; keeps tab IA stable early.

**Alternatives considered**: Hide History/Settings tabs until 003/004 — rejected (mockups show three tabs on Home).

## R11. Firestore rules file

**Decision**: Reuse the existing `firebase/firestore.rules` from 001. Extend with `classLogs` and `waterLogs` match blocks. Do not create a second rules file. Emulator tests add cases for the new collections.

**Rationale**: Single source of security rules; plan notes.

**Alternatives considered**: Per-feature rules files — rejected.

## R12. Test layout and required cases

**Decision**: All unit/ViewModel tests live only under `BrenzTests/Domain`, `BrenzTests/Data`, `BrenzTests/Presentation`. Required coverage includes:

| Case | Layer |
|------|--------|
| 8 consecutive weeks + year change for `elementId` | Domain |
| 14:59 and 15:00 both accepted | Domain |
| Same hour → `alreadyLoggedThisHour`, nothing written | Domain |
| Home class `registeredAt` is always today’s local date | Domain |
| Undo and delete free the hour | Domain |
| Same-hour logs ordered by `registeredAt` then `id` | Domain |
| Home sheet writes `lastDiscipline` without a log | Domain + Presentation |
| Kickboxing then boxing → first selected is boxing | Domain |
| Pet active band with `downDays` = 3 and `downDays` = 5 | Domain |
| Injected `TimeZone`/`Calendar` for today, midday, water day | Domain |
| Midnight rollover: water, board, bubble day flags, pet state | Domain (+ Presentation as needed) |
| Optimistic class/water UI when fake repo write never completes | Presentation |

Also cover weekday conversion for 7 days; pet/stage/water/message boundaries per architecture acceptance criteria. All unit/ViewModel tests stay under `BrenzTests/Domain`, `BrenzTests/Data`, `BrenzTests/Presentation` only.

**Rationale**: Constitution VI + stored-values acceptance + plan notes + analyze remediation.

**Alternatives considered**: UI tests — out of v1. Scattering tests outside the three folders — rejected.
