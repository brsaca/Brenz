# Quickstart: Core Loop (002)

**Branch**: `002-core-loop` | **Date**: 2026-11-27  
**Purpose**: Runnable validation guide after implementation. Details live in [data-model.md](./data-model.md) and [contracts/](./contracts/).

## Prerequisites

- Spec **001** complete enough to reach Home with `onboardingCompleted == true`, selected disciplines, and the shared weekday-conversion helper available (reuse if present).
- Firebase project with Auth + Firestore + Remote Config; emulator available for rules tests.
- Single rules file: `firebase/firestore.rules` (extended, not duplicated).
- Xcode 16+, iOS 17 simulator, Swift Testing.

## Setup

```bash
# From repo root
open Brenz.xcodeproj   # or workspace if generated
# Ensure Packages/BrenzDesignSystem resolves
# Start Firestore emulator when running rules tests (see firebase/)
```

Wire in `AppContainer`: `TrainingRepository`, `RulesConfigRepository`, `MessageRotationStore`, catalogs, and Home / shell destinations.

## Automated validation

### Domain (`BrenzTests/Domain`)

```bash
# Run Domain test target / filter as configured in the Xcode scheme
```

Must include:

1. `elementId` across **8 consecutive weeks** and a **year change** (reference Monday 2026-01-05); cut 0.5 uses same function with `weekIndex = 0`.
2. Weekday conversion **7 days** (Calendar Sunday=1 → Monday=1).
3. Classes at **14:59** and **15:00** both accepted.
4. Second class in the **same hour** → `alreadyLoggedThisHour`, repository write count unchanged.
5. Home class `registeredAt` is always **today’s local date**.
6. **Undo** and **DeleteClass** free the hour for a new registration (`ChangeClassDiscipline` is 003).
7. Same-hour collision ordering: `registeredAt` then `id`.
8. Sheet / `UpdateLastDiscipline`: persists `lastDiscipline` with **zero** class logs created.
9. Selected kickboxing then boxing → fallback **first selected in catalog order** is **boxing**.
10. Pet state: active = `days < downDays`; boundaries with defaults and with **`downDays = 3` / `downDays = 5`**; water→animated; `stageReached` never decreases; MessageSelector priorities.
11. Injected `TimeZone`/`Calendar` for today, midday, and water day (`BrenzTests/Domain/LocalCalendarBoundaryTests.swift`).
12. Midnight rollover covers water, board, and pet state (US3); bubble day-flag midnight coverage lives with US4 MessageRotationStore / MessageSelector tests.

### Data (`BrenzTests/Data`)

- DTO ↔ entity mapping for `ClassLog` / `WaterLog` / `lastDiscipline`.
- `RulesConfig` invalid remote values → defaults; `downDays < 1` or `downDays >= lazyDays` → in-app pair defaults.
- Emulator: cases in [contracts/firestore-class-water.md](./contracts/firestore-class-water.md) against the **single** `firebase/firestore.rules` from 001.

### Presentation (`BrenzTests/Presentation`)

- `HomeViewModel`: 5 s Undo window; toast `home.notice.alreadyLoggedThisHour` (2.5 s, no action) without clearing prior Undo; tapping “Fui a clase” and +/− updates UI when fake repo write **never completes**; water − at zero is no-op; `waterMessageDuration` = 4 s.

## Manual simulator checklist

1. Sign in (001) → Home shows Train tab with empty tatami outlines, water `0/goal`, character normal.
2. Tap main class button → cell fills with day’s element, Undo appears; wait 5 s → Undo gone.
3. Tap again in the **same** clock hour → no new cell; toast `home.notice.alreadyLoggedThisHour` (~2.5 s); no character message; no new Undo.
4. After the next hour (or mocked clock in tests) → second class accepted.
5. Undo within 5 s → cell removed; tap again same hour → accepted.
6. Water + to goal with base normal → pet animated; − below goal → normal + dropped-below message when applicable; bottle visual stays full if count &gt; goal while text shows e.g. `10/8`.
7. Airplane mode: class and water update UI immediately (manual acceptance of SC-001 / SC-002 perceived immediacy; no latency instrumentation).
8. Two+ disciplines: chip + sheet changes `lastDiscipline` without logging; one discipline: chip hidden.
9. History / Settings tabs open **placeholder shells** only.
10. Reduce Motion on: fades/static updates only (no bounce/spin).
11. Language `es` and `en`: Home chrome + a sample of `message.*` keys resolve (no raw keys).

## Expected outcomes

| Check | Pass criteria |
|-------|----------------|
| SC-001 / SC-002 | **Manual acceptance** in this checklist (items 2–7) plus the Presentation optimistic-path test (fake repo write never completes → UI still updates). No latency instrumentation. |
| SC-004 | Same local class dates → same stored `elementId` |
| SC-009 | Same-hour second tap writes nothing |
| Shells | History/Settings reachable, empty of 003/004 content |
| Rules | One `firestore.rules` file covers users + classLogs + waterLogs |

## Out of this quickstart

- Full History edit/delete UI (exercise Domain `DeleteClass` in unit tests; `ChangeClassDiscipline` is 003).
- Settings water-goal editor / account flows.
- `min_version` lock screen.
- Final board element art if not yet delivered (code-drawn placeholders are acceptable until Sat Nov 28).
