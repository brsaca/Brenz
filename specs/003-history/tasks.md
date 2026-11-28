---

description: "Task list for History feature implementation"
---

# Tasks: History

**Input**: Design documents from `/specs/003-history/`

**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/, quickstart.md

**Tests**: Swift Testing for Domain and ViewModels with fake repositories. Firestore
rules reuse the existing `firebase/` emulator suite (no second rules file). No UI
tests. Tests are written in the same task as the code. All suites MUST pass before
a task is complete. Do not add XCTest. Tests live only under `BrenzTests/Domain`,
`BrenzTests/Data`, and `BrenzTests/Presentation`.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

**Dependencies**: Specs **001** (auth, profile, Reduce Motion modifier, translucent sheet) and **002** (`DeleteClass`, class/water observe, History shell, `DayBoard`, Design System grid/bottle/tab bar) must be far enough that History can replace the shell. This feature does not rebuild those pieces.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Domain**: `Brenz/Domain/` (Entities, Rules, UseCases, Repositories protocols only; no localized text)
- **Data**: `Brenz/Data/Firebase/` and `Brenz/Data/Local/` (DTOs, repository implementations)
- **Presentation**: `Brenz/Presentation/History/` and `Brenz/Presentation/Shared/`
- **Composition root**: `Brenz/App/AppContainer.swift`
- **Design system**: `Packages/BrenzDesignSystem/`
- **Strings**: `Brenz/Resources/Localizable.xcstrings` (en base + es)
- **Firestore rules and tests**: `firebase/` (reuse only)
- **Unit tests**: `BrenzTests/` mirroring the layer folders

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Confirm 001/002 prerequisites and History feature folders before Domain work

- [ ] T001 Confirm iOS 17 target, Firebase SPM (Auth/Firestore/RemoteConfig only), and no new third-party packages for History in the Xcode project
- [ ] T002 [P] Confirm `Brenz/Resources/Localizable.xcstrings` exists with en base + es locales
- [ ] T003 [P] Confirm Design System reuse from 001/002: `BrenzTabBar`, isometric grid, bottle, translucent floating sheet, Reduce Motion modifier under `Packages/BrenzDesignSystem/`; note whether `BrenzSuccessMark` and chart-bar components already exist
- [ ] T004 [P] Confirm Data reuse: `TrainingRepository` protocol exposes `observeClasses`, `observeWater`, `deleteClass`; add `updateDiscipline` to the protocol and `TrainingRepositoryImpl` if absent; `ProfileRepository` exists; single `firebase/firestore.rules` (do not create a second rules file); ClassLogDTO/WaterLogDTO present under `Brenz/Data/Firebase/`
- [ ] T005 Confirm 002 deliverables: `DeleteClass` in `Brenz/Domain/UseCases/DeleteClass.swift`, History shell under `Brenz/Presentation/History/`, and `DayBoard` in `Brenz/Presentation/Shared/DayBoard.swift`
- [ ] T006 Create History presentation folder scaffolding under `Brenz/Presentation/History/` (replace shell placeholders as later stories land)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Pure Domain week assembly, board/hour rules, and AppContainer wiring that ALL user stories share

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T007 [P] Add Domain types `WeekDayCell` (`kind`: today \| pastWithTraining \| pastWithoutTraining \| future; `isSelectable` false when future), `DayLabelKind` (`today` \| `date(Date)`—pure Domain value, no localized text), `DayViewSnapshot` (`dayLabel: DayLabelKind`, board, classTotal, waterTotal; blocks optional until US4), `HourBlock` (6a/9a/12p/3p/6p/9p), and `LogSheetSession` states (edit \| confirmDelete \| deletedSuccess \| closed) in `Brenz/Domain/Entities/` (or `Brenz/Domain/Models/` if that folder convention already exists from 001/002); Presentation/ViewModel localizes `DayLabelKind` via `Brenz/Resources/Localizable.xcstrings`
- [ ] T008 [P] Implement or extend pure `BoardLayout` in `Brenz/Domain/Rules/BoardLayout.swift` for History: `minimumRows = 3`, `BoardTheme` columns=3 `maxCells=12`, filled cells = first 12 of already-ordered classes, `overflowCount = max(0, count - 12)`, rowCount = `max(minimumRows, ceil(min(count, 12) / columns))`; size never persisted; include Domain tests for empty day (3 rows) and >12 overflow in `BrenzTests/Domain/BoardLayoutTests.swift`
- [ ] T009 [P] Implement pure `HourBlockBucket.bucket(for:calendar:)` in `Brenz/Domain/Rules/HourBlockBucket.swift` with half-open windows `[06:00–09:00)`, `[09:00–12:00)`, `[12:00–15:00)`, `[15:00–18:00)`, `[18:00–21:00)`, `[21:00–24:00)` and `00:00–05:59` → 6a; include Domain tests that 08:59 → 6a and 09:00 → 9a in `BrenzTests/Domain/HourBlockBucketTests.swift`
- [ ] T010 [P] Add day class ordering helper `sortedForDay` (`registeredAt` ascending, then `id` ascending) in `Brenz/Domain/Rules/` (or beside BoardLayout); include Domain tests for same-hour offline duplicates in `BrenzTests/Domain/ClassLogOrderingTests.swift`
- [ ] T011 Implement `ObserveWeekHistory.execute(uid:now:calendar:)` → `AsyncStream<WeekHistorySnapshot>` in `Brenz/Domain/UseCases/ObserveWeekHistory.swift`: current local week Monday 00:00…Sunday end only; seven `WeekDayCell`s; per-day class/water maps; class lists ordered via T010; snapshot carries **week data only** (no `selectedDay`); emit from cache then on class/water/profile changes; reuse `TrainingRepository` / `ProfileRepository` protocols only (no Firebase imports)
- [ ] T012 Wire `ObserveWeekHistory` (and later story use cases) in `Brenz/App/AppContainer.swift` injecting Domain protocols only; Data implementations stay composition-root-only
- [ ] T013 Confirm existing `firebase/` emulator suite covers `classLogs` update: `discipline` only (cannot change `elementId`/`registeredAt`); extend `firebase/tests/` only if that edge is missing—do not add a second rules file

**Checkpoint**: Foundation ready — user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Browse the current week and a day's board (Priority: P1) 🎯 MVP

**Goal**: Replace the 002 History shell with the current-week bar, day selection (past/today), large tatami (min 3 rows / cap 12 / `+N`), and Classes/Water day totals (charts deferred to US4).

**Independent Test**: With ≥1 class today and ≥1 on a past weekday this week, open History, confirm week-bar states, select each selectable day, and verify tatami and card totals match that day—without opening the log sheet.

### Implementation for User Story 1

- [ ] T014 [P] [US1] Implement `HistoryViewModel` in `Brenz/Presentation/History/HistoryViewModel.swift`: ViewModel owns selected day (default today); subscribe to `ObserveWeekHistory` week-data stream; ignore selection when `WeekDayCell.isSelectable == false`; build chosen-day `DayViewSnapshot` from week logs with `BoardLayout` `minimumRows=3`; empty cell / overflow `+N` not actionable yet (hooks for US2)
- [ ] T015 [P] [US1] Add Presentation tests in `BrenzTests/Presentation/HistoryViewModelTests.swift`: future days not tappable / selection ignored; default selection is today; empty day yields min 3-row board and classTotal 0
- [ ] T016 [US1] Replace History shell with `HistoryView` in `Brenz/Presentation/History/HistoryView.swift`: Mon–Sun week bar (today lime border + bold number; past-with-training lime checkmark; past-without number only; future dimmed non-interactive); day label; large `DayBoard` from `Brenz/Presentation/Shared/DayBoard.swift`; Classes and Water summary cards showing day totals only (no chart UI yet; Water read-only—no list, no +/−); tab bar remains `BrenzTabBar`
- [ ] T017 [P] [US1] Add History browse strings (week/day labels, card titles, empty states) en+es to `Brenz/Resources/Localizable.xcstrings` (device Spanish → Spanish; otherwise English); no hardcoded UI copy
- [ ] T018 [US1] Verify on simulator (SweetPad): week has no prev/next; offline browse still updates from cache; Spanish locale shows Spanish History copy

**Checkpoint**: User Story 1 fully functional and testable independently (MVP browse)

---

## Phase 4: User Story 2 - Delete a class from the tatami (Priority: P1)

**Goal**: Tap a filled tatami cell → log sheet (6b) → confirm delete (6c) → remove class via reused `DeleteClass`; refresh board/cards/week checkmark; stage reached unchanged; works offline; no character message.

**Independent Test**: Log two classes on a day, open History, delete one after confirmation, verify tatami count, Classes total, and (on Home) pet state update while stage reached stays the same—then repeat once offline.

### Implementation for User Story 2

- [ ] T019 [P] [US2] Implement `LogSheetViewModel` in `Brenz/Presentation/History/LogSheetViewModel.swift` for states `edit` (6b) and `confirmDelete` (6c): show element/discipline/time; Cancel returns to edit; Confirm calls reused `DeleteClass` with `now`/`calendar`; stay on same chosen day; do not change `lastDiscipline`; do not emit message-bank keys
- [ ] T020 [P] [US2] Domain tests in `BrenzTests/Domain/DeleteClassHistoryEffectsTests.swift` (or extend existing DeleteClass tests): after delete, `stageReached` unchanged; pet state matches `PetStateCalculator` on remaining classes; hour freed; `lastDiscipline` unchanged
- [ ] T021 [US2] Add `LogSheetView` in `Brenz/Presentation/History/LogSheetView.swift` for 6b/6c using translucent floating sheet over darkened History (tab bar behind): close (X), title, element + discipline + time row, discipline chips (display only for delete path until US3), Delete record → confirmation copy (class leaves tatami; stage does not go down), solid danger Delete + Cancel
- [ ] T022 [US2] Wire filled-cell tap in `HistoryView` / `HistoryViewModel` to open log sheet for that `ClassLog`; empty cells and `+N` overflow must be no-ops (`Brenz/Presentation/History/HistoryView.swift`, `HistoryViewModel.swift`); after delete, refresh from `ObserveWeekHistory` stream (board shrinks to needed rows min 3; Classes −1; past-day checkmark clears if last class)
- [ ] T023 [P] [US2] Add log-sheet 6b/6c strings (titles, Delete record, confirmation notice, Cancel, Delete) en+es to `Brenz/Resources/Localizable.xcstrings`
- [ ] T024 [US2] Verify on simulator: Cancel does not delete; offline confirm Delete refreshes immediately; Reduce Motion path closes sheet after delete without 6d (full 6d timing is US5); after delete, reopen Home and confirm pet state is recalculated and `stageReached` is unchanged; no character speech bubble on delete

**Checkpoint**: Browse (US1) + delete (US2) work independently; MVP correction flow shippable

---

## Phase 5: User Story 3 - Change a class discipline (Priority: P2)

**Goal**: From log sheet 6b, select a different discipline and Save; only `ClassLog.discipline` updates; time/date/`elementId`/`lastDiscipline` unchanged; Save inactive when selection equals stored.

**Independent Test**: Open a class log sheet, select another discipline, save, confirm same element and time while discipline label updates; reopening with same discipline leaves Save inactive.

### Implementation for User Story 3

- [ ] T025 [P] [US3] Implement `ChangeClassDiscipline.execute(uid:logId:newDiscipline:) async throws` in `Brenz/Domain/UseCases/ChangeClassDiscipline.swift` (AppError style of 001/002: Domain throws `DataError`; Presentation publishes optional `AppError`): ensure `TrainingRepository.updateDiscipline` exists on protocol + impl if absent (do not create a second `firestore.rules`); write only `discipline` via that method; validate with `Discipline` enum only; never mutate `elementId`, `registeredAt`, or profile `lastDiscipline`; reject `.unknown`; optimistic write (server rejection logged only in Data)
- [ ] T026 [P] [US3] Domain tests in `BrenzTests/Domain/ChangeClassDisciplineTests.swift`: after execute, `elementId` and `registeredAt` unchanged; `UserProfile.lastDiscipline` unchanged; invalid discipline rejected
- [ ] T027 [US3] Extend `LogSheetViewModel` / `LogSheetView` so Save is enabled iff `selectedDiscipline != storedDiscipline`; Save calls `ChangeClassDiscipline` then closes sheet; dismiss without Save discards selection; always show the four fixed discipline chips (`boxing`, `jiu_jitsu`, `muay_thai`, `kickboxing`) single-selection as in the mockups—not filtered by `selectedDisciplines` (`Brenz/Presentation/History/LogSheetViewModel.swift`, `LogSheetView.swift`)
- [ ] T028 [P] [US3] Add Save / discipline-chip related strings en+es to `Brenz/Resources/Localizable.xcstrings` if missing
- [ ] T029 [US3] Wire `ChangeClassDiscipline` in `Brenz/App/AppContainer.swift`; verify offline save applies locally and Home next-class discipline (`lastDiscipline`) is unchanged

**Checkpoint**: User Stories 1–3 independently functional (cut-order item 6 can defer this phase if time-short)

---

## Phase 6: User Story 4 - Day summary cards with 3-hour charts (Priority: P2)

**Goal**: Classes and Water cards each show day total plus six-block 3-hour charts using `HourBlockBucket`; Water remains read-only.

**Independent Test**: On a day with classes and glasses at known times, select that day and verify each card’s total and which blocks show activity—without water edit controls.

### Implementation for User Story 4

- [ ] T030 [P] [US4] Extend `DayViewSnapshot` assembly (in `ObserveWeekHistory` and/or `HistoryViewModel`) to populate `classBlocks` and `waterBlocks` via `HourBlockBucket` so each log counts in exactly one bucket and in the day total (`Brenz/Domain/UseCases/ObserveWeekHistory.swift` and/or `Brenz/Presentation/History/HistoryViewModel.swift`)
- [ ] T031 [P] [US4] Domain tests in `BrenzTests/Domain/HourBlockBucketTests.swift` (extend): pre-06:00 → 6a and still in day total; Classes and Water use identical mapping
- [ ] T032 [US4] Add or reuse chart-bar component in `Packages/BrenzDesignSystem/` if missing; render six labeled blocks (6a, 9a, 12p, 3p, 6p, 9p) on Classes and Water cards in `Brenz/Presentation/History/HistoryView.swift`—Water card still has no event list and no add/remove
- [ ] T033 [P] [US4] Add chart axis/label strings en+es to `Brenz/Resources/Localizable.xcstrings` if needed
- [ ] T034 [US4] Verify on simulator: logs at 08:59 and 09:00 land in 6a and 9a; Water has no +/−

**Checkpoint**: Day view matches product charts (cut-order item 2 can defer this phase if time-short)

---

## Phase 7: User Story 5 - Class-deleted animation before the sheet closes (Priority: P3)

**Goal**: After confirm Delete, show ~2.5 s class-deleted success state (6d) then close; with Reduce Motion, skip sequence and close after delete.

**Independent Test**: Confirm delete with Reduce Motion off → success sequence ~2.5 s then close; with Reduce Motion on → immediate close without sequence.

### Implementation for User Story 5

- [ ] T035 [P] [US5] Add `BrenzSuccessMark` to `Packages/BrenzDesignSystem/` if 001/002 did not ship it (checkmark/ring + emptying progress using package motion tokens)
- [ ] T036 [US5] Extend `LogSheetView` / `LogSheetViewModel` with `deletedSuccess` (6d): after successful `DeleteClass`, show success mark + remaining-count figure for the chosen day for ~2.5 s then auto-dismiss; if Reduce Motion (package modifier from 001) is on, skip animation and close after delete (`Brenz/Presentation/History/LogSheetView.swift`, `LogSheetViewModel.swift`)
- [ ] T037 [P] [US5] Add plural figure strings for 0/1/n in en+es to `Brenz/Resources/Localizable.xcstrings`: today → `history.deleted.classesToday` (`{count} classes today` / `{count} clases hoy`); other day → `history.deleted.classesThatDay` (`{count} classes that day` / `{count} clases ese día`)
- [ ] T038 [P] [US5] Presentation tests in `BrenzTests/Presentation/LogSheetViewModelTests.swift`: Reduce Motion path skips 6d delay; remaining count is for chosen day after delete
- [ ] T039 [US5] Verify on simulator: ~2.5 s (±0.5 s) visible with motion on; immediate close with Reduce Motion on

**Checkpoint**: All user stories independently functional

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Overflow edge cases, localization completeness, design-system compliance, quickstart validation

- [ ] T040 [P] Domain/Presentation coverage for >12 classes: first 12 by `registeredAt`↑ then `id`↑ are filled/tappable; `+N` not tappable; after delete, next earliest overflow promotes in `BrenzTests/Domain/BoardLayoutTests.swift` and/or `BrenzTests/Presentation/HistoryViewModelTests.swift`
- [ ] T041 [P] Confirm all new History/log-sheet strings have en + es (including 6d plurals) in `Brenz/Resources/Localizable.xcstrings`
- [ ] T042 Confirm views use BrenzDesignSystem tokens only; dark mode only; Reduce Motion respected; no rebuilt tab bar/grid/bottle/sheet; no Firebase imports in Presentation/Domain
- [ ] T043 Confirm no second `firestore.rules` and no duplicate repository protocols; `ChangeClassDiscipline` does not write `elementId`/`registeredAt`/`lastDiscipline`
- [ ] T044 Run automated checks from `specs/003-history/quickstart.md` (Domain cases 1–4 + Presentation case 5); confirm all `BrenzTests` suites pass
- [ ] T045 Manual quickstart scenarios A–E on simulator with SweetPad (week/day, delete, discipline, charts, overflow); keep **SC-006** as a manual smoke only (offline browse / save discipline / delete refresh ≤ 2 s, excluding optional ~2.5 s 6d); document any cut-order skips (charts → tatami growth polish → 6d → change discipline)
- [ ] T046 [P] Update feature docs only if implementation names diverge from `specs/003-history/contracts/` (keep English)
- [ ] T047 [P] Domain/Presentation tests for device calendar/time-zone change: with a different injected `calendar`/`now`, week bar membership and “today” (`WeekDayCell.kind` / `DayLabelKind`) recompute; stored `ClassLog.registeredAt` values remain untouched (`BrenzTests/Domain/` and/or `BrenzTests/Presentation/HistoryViewModelTests.swift`)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — can start immediately (requires 001/002 prerequisites available)
- **Foundational (Phase 2)**: Depends on Setup — **BLOCKS** all user stories
- **User Story 1 (Phase 3)**: Depends on Foundational — MVP browse
- **User Story 2 (Phase 4)**: Depends on Foundational + US1 day board (needs filled cells + History refresh)
- **User Story 3 (Phase 5)**: Depends on Foundational + US2 log sheet 6b shell
- **User Story 4 (Phase 6)**: Depends on Foundational + US1 cards (can parallel with US2/US3 after US1)
- **User Story 5 (Phase 7)**: Depends on US2 delete success path
- **Polish (Phase 8)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: After Foundational — no dependency on other stories — 🎯 MVP
- **User Story 2 (P1)**: After US1 (needs tatami cells) — independently testable delete flow
- **User Story 3 (P2)**: After US2 log sheet — independently testable Save/discipline
- **User Story 4 (P2)**: After US1 — can run parallel with US2/US3; independently testable charts
- **User Story 5 (P3)**: After US2 — independently testable 6d / Reduce Motion

### Within Each User Story

- Tests pass before a task is complete (written in same task or paired test task)
- Domain/rules before ViewModels before Views before simulator verification
- Story complete before moving to next priority when staffing is sequential

### Parallel Opportunities

- T002–T004 (Setup checks) in parallel
- T007–T010 (Domain types/rules/tests) in parallel after Setup
- After Foundational: US4 can proceed in parallel with US2 once US1 is done
- Within US2: T019/T020/T023 in parallel before view wiring
- Within US3: T025/T026/T028 in parallel
- Within US5: T035/T037/T038 in parallel before wiring 6d into the sheet
- Polish: T040/T041/T046/T047 in parallel where files differ

---

## Parallel Example: User Story 1

```bash
# Domain foundation already done (BoardLayout, ObserveWeekHistory).
# Launch ViewModel + strings together:
Task: "Implement HistoryViewModel in Brenz/Presentation/History/HistoryViewModel.swift"
Task: "Add Presentation tests in BrenzTests/Presentation/HistoryViewModelTests.swift"
Task: "Add History browse strings en+es to Brenz/Resources/Localizable.xcstrings"

# Then UI + verify:
Task: "Replace History shell with HistoryView in Brenz/Presentation/History/HistoryView.swift"
Task: "Verify on simulator (SweetPad)"
```

## Parallel Example: User Story 2

```bash
Task: "Implement LogSheetViewModel in Brenz/Presentation/History/LogSheetViewModel.swift"
Task: "Domain tests for DeleteClass History effects in BrenzTests/Domain/DeleteClassHistoryEffectsTests.swift"
Task: "Add log-sheet 6b/6c strings en+es to Brenz/Resources/Localizable.xcstrings"
```

## Parallel Example: User Story 4 (after US1)

```bash
Task: "Extend DayViewSnapshot assembly with classBlocks/waterBlocks"
Task: "Domain tests for pre-06:00 → 6a in BrenzTests/Domain/HourBlockBucketTests.swift"
Task: "Add chart label strings en+es to Brenz/Resources/Localizable.xcstrings"
```

---

## Implementation Strategy

### MVP First (User Stories 1 + 2)

1. Complete Phase 1: Setup (confirm 001/002 reuse)
2. Complete Phase 2: Foundational (CRITICAL — blocks all stories)
3. Complete Phase 3: User Story 1 (week + day board + totals)
4. Complete Phase 4: User Story 2 (delete via log sheet)
5. **STOP and VALIDATE**: Independent tests + quickstart A/B
6. Demo if ready (browse + correct wrong logs)

### Incremental Delivery

1. Setup + Foundational → foundation ready
2. US1 → browse MVP
3. US2 → delete correction (still MVP)
4. US3 → change discipline (or cut 6 if time-short)
5. US4 → charts (or cut 2 if time-short)
6. US5 → 6d animation (or cut 5 if time-short)
7. Polish → overflow, catalog, quickstart

### Cut Order (if time-short)

Per plan.md (local 1–4 ↔ product scope §8 ids **2, 3, 5, 6**) — do **not** cut week bar, day selection, large tatami filled cells, delete with stage preservation, offline apply:

1. Cut charts — product **2** (skip/defer Phase 6 / US4)
2. Cut tatami growth polish — product **3** (keep min 3 / cap 12 / `+N` if cheap)
3. Cut 6d animation — product **5** (skip/defer Phase 7 / US5; Reduce Motion still = no animation)
4. Cut change discipline — product **6** (skip/defer Phase 5 / US3; ship delete-only log sheet)

### Parallel Team Strategy

1. Team completes Setup + Foundational together
2. After Foundational + US1:
   - Dev A: US2 (+ US5)
   - Dev B: US3
   - Dev C: US4
3. Stories integrate via shared `HistoryViewModel` / week stream without breaking prior stories

---

## Notes

- [P] tasks = different files, no dependencies on incomplete work
- [Story] labels map tasks to US1–US5 for traceability
- Each user story is independently completable and testable
- Tests pass before a task is complete (Swift Testing; no XCTest)
- Commit in English after the completed task (Conventional Commits)
- Stop at any checkpoint to validate story independently
- Avoid: new third-party packages, hardcoded UI strings, second `firestore.rules`, duplicate repository protocols
- Avoid: literal colors/magic numbers in views; motion without Reduce Motion fallback
- Avoid: rebuilding BrenzTabBar / tatami grid / water bottle / translucent sheet
- `ChangeClassDiscipline` MUST NOT mutate `elementId`, `registeredAt`, or `lastDiscipline`
- Server rejection of optimistic writes: log only, no user-facing error in v1
