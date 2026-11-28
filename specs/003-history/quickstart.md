# Quickstart: History (003)

**Branch**: `003-history`  
**Goal**: Validate the History week, day board, log sheet, and required Domain/Presentation tests without implementing the full task list yet.

Contracts: [domain-usecases.md](./contracts/domain-usecases.md), [history-ui.md](./contracts/history-ui.md), [firestore-reuse.md](./contracts/firestore-reuse.md).  
Model: [data-model.md](./data-model.md).

---

## Prerequisites

- Specs **001** and **002** implemented far enough that:
  - Auth + profile exist
  - `DeleteClass` and class/water observe work offline
  - History **shell** tab is reachable via `BrenzTabBar`
  - `Packages/BrenzDesignSystem` exposes tab bar, grid, bottle, translucent sheet, Reduce Motion modifier (`BrenzSuccessMark` present or added by this feature)
- Xcode 16+, iOS 17 simulator
- Firebase project / emulator available only if exercising real Firestore (unit tests use fakes)

---

## Setup

```bash
# From repo root
open Brenz.xcodeproj   # or workspace used by the project
# Select scheme Brenz, destination iPhone simulator
```

Wire in `AppContainer` (when implementing): `ObserveWeekHistory`, `ChangeClassDiscipline`, reused `DeleteClass`.

---

## Automated checks (required)

Run Swift Testing targets; tests must live only under:

- `BrenzTests/Domain`
- `BrenzTests/Data`
- `BrenzTests/Presentation`

| # | Case | Layer | Expected |
|---|------|-------|----------|
| 1 | Discipline edit keeps `elementId` and `lastDiscipline` | Domain | After `ChangeClassDiscipline`, log.elementId unchanged; profile.lastDiscipline unchanged |
| 2 | Delete keeps `stageReached` and recalculates state | Domain | stageReached same; PetState matches calculator on remaining classes |
| 3 | Same-hour logs ordered by `registeredAt` then `id` | Domain | Sort/BoardLayout filled order matches |
| 4 | Bucketing at 08:59 and 09:00 | Domain | 08:59 → 6a; 09:00 → 9a |
| 5 | Future days not tappable | Presentation | Future `WeekDayCell.isSelectable == false` / selection ignored |

### Manual smoke: SC-006 (offline ≤ 2 s)

**SC-006** (browse / save discipline / delete refresh within 2 seconds offline, excluding optional ~2.5 s 6d) is a **manual smoke only**. It is **not unit-testable** (wall-clock UX latency depends on device/simulator timing). Verify during scenarios A–C below with Airplane mode; do not add an automated timing assertion.

Example invoke (adjust scheme/test plan names to the repo):

```bash
xcodebuild test -scheme Brenz -destination 'platform=iOS Simulator,name=iPhone 16' \
  -only-testing:BrenzTests
```

---

## Manual validation scenarios

### A. Week and day board (P1)

1. Sign in; ensure ≥1 class today and ≥1 on a past weekday this week.
2. Open History.
3. Confirm Mon–Sun only; no week navigation; today lime+bold; past with class shows checkmark; future dimmed and ignores taps.
4. Tap a past day → label, tatami (≥3 rows), Classes/Water totals match that day.
5. Airplane mode: browse still updates from cache.

### B. Delete (P1)

1. Tap a filled cell → 6b opens.
2. Delete record → 6c → Cancel returns to 6b.
3. Confirm Delete → (6d if motion on) → History without that log; Classes −1; Home pet state may change; stage unchanged.
4. Re-register in the freed hour succeeds (002 RegisterClass).
5. Reduce Motion on → no 6d animation; sheet closes after delete.

### C. Change discipline (P2; skip if cut 6)

1. Open 6b; Save inactive.
2. Select other discipline → Save → sheet closes; cell shows new discipline; same element and time.
3. Profile `lastDiscipline` / Home next-class discipline unchanged.
4. Offline save still applies locally.

### D. Charts (P2; skip if cut 2)

1. Logs at 08:59 and 09:00 appear in 6a and 9a respectively.
2. Pre-06:00 log counts in 6a and in the day total.
3. Water card has no add/remove controls.

### E. Overflow

1. Fixture with >12 classes (ordered): 12 filled cells + `+N`; `+N` not tappable; after deletes, next earliest overflow fills a cell.

---

## Expected outcomes summary

- History shell from 002 is fully replaced.
- No second `firestore.rules`; no duplicate repository protocols.
- Design-system components reused, not rebuilt.
- Cut order if short: charts → tatami growth polish → 6d animation → change discipline (delete remains).

---

## Out of this quickstart

- Previous weeks, event lists, edit time/date, Settings (004), Home register/Undo (002).
- Full `tasks.md` generation (`/speckit-tasks`).
