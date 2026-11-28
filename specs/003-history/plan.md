# Implementation Plan: History

**Branch**: `003-history` | **Date**: 2026-11-27 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/003-history/spec.md`

**Note**: This template is filled in by the `/speckit-plan` command; its definition describes the execution workflow.

## Summary

Replace the History shell left by spec 002 with the full current-week History experience: Monday–Sunday week bar, chosen-day tatami (large, min 3 rows / cap 12 / `+N`), Classes and Water summary cards with 3-hour block charts, and the log sheet (6b–6e) to change discipline or delete a class.

Technical approach: Domain owns week assembly (`ObserveWeekHistory`), day ordering, `BoardLayout` (History minimum rows), hour-block bucketing, `ChangeClassDiscipline` (moved here from 002), and delete side-effects via existing `DeleteClass` + `PetStateCalculator` / `StageCalculator`. Presentation replaces the shell under `Presentation/History/` and reuses `DayBoard`, `BrenzTabBar`, the isometric grid, bottle, translucent sheet, `BrenzSuccessMark`, and the Reduce Motion modifier from `Packages/BrenzDesignSystem` (built in 001/002). Data reuses `TrainingRepository`, `ProfileRepository`, DTOs, and the single `firebase/` rules file—no second rules file, no duplicate protocols. Optimistic offline writes already defined in 001/002.

## Technical Context

**Language/Version**: Swift (Xcode 16 or later)

**Primary Dependencies**: SwiftUI, Observation, Firebase iOS SDK via SPM (FirebaseAuth, FirebaseFirestore, FirebaseRemoteConfig), local package BrenzDesignSystem. No other third-party.

**Storage**: Firestore with offline persistence. UserDefaults only for message rotation (out of this feature). Class/water logs and profile via existing repositories from 001/002.

**Testing**: Swift Testing for Domain and ViewModels with fake repositories. Tests live only in `BrenzTests/Domain`, `BrenzTests/Data`, and `BrenzTests/Presentation`. Firestore rules already covered by 001/002 emulator tests (reuse; do not add a second rules file). No UI tests.

**Target Platform**: iOS 17+, iPhone only.

**Project Type**: single iOS app with folders by layer + local package Packages/BrenzDesignSystem.

**Performance Goals**: works offline with local cache; History and log-sheet actions refresh within ~2 s of confirm (excluding optional ~2.5 s 6d sequence); 60 fps UI.

**Constraints**: no animation libraries; design tokens only in views; en/es String Catalog; Reduce Motion respected via package modifier from 001; dark mode only; no Combine; do not rebuild BrenzTabBar / tatami grid / water bottle; do not duplicate repository protocols or firestore.rules; size of tatami is derived never stored; `ChangeClassDiscipline` does not mutate `elementId`, `registeredAt`, or `lastDiscipline`.

**Scale/Scope**: One History screen + log sheet states (6b–6e). Replaces 002 History shell. Dependencies: 001 (auth, profile, Reduce Motion modifier, translucent sheet foundation), 002 (Home logging, DeleteClass, Design System grid/bottle/tab bar, DayBoard shell). Out of scope: other weeks, event lists, edit time/date, add past classes, Settings (004), Home undo/register (002).

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### Pre-research (Phase 0 gate)

- I. Layer Separation: PASS — Domain use cases/rules pure; Firestore only in `Data/Firebase/`; History ViewModels and views import no Firebase.
- II. Platform and Stack: PASS — iOS 17+, SwiftUI, MVVM `@Observable`, AppContainer; Firebase SPM only; BrenzDesignSystem local package; no new third-party libs.
- III. Language and Naming: PASS — code/docs English; all History/log-sheet copy in `Localizable.xcstrings` (en base, es), including 6d plural figure strings.
- IV. Pure Domain Rules: PASS — `BoardLayout`, hour-block bucketing, day ordering, `PetStateCalculator` / `StageCalculator` take `now`/`Calendar` as parameters; no `Date()` inside rules.
- V. Data Source and Stored Values: PASS — reuse Firestore + offline optimistic writes; `ChangeClassDiscipline` updates only `discipline`; `elementId`/`registeredAt`/`lastDiscipline` unchanged per stored-values.
- VI. Testing: PASS — Swift Testing in Domain/Presentation (and Data if mapping needed); no UI tests; rules reuse existing emulator suite.
- VII. Content Voice: PASS — no character message on delete; no guilt/punishment copy in log sheet (stage-does-not-go-down notice is factual).
- VIII. UI and Design System: PASS — dark only; tokens from package; reuse translucent sheet + `BrenzSuccessMark`; Reduce Motion skips 6d animation.
- IX. Scope Discipline: PASS — current week only; water read-only on History; cut order documented if time is short.
- X. Release Safety: PASS — no account deletion or `min_version` changes in this feature; no forbidden brand names.
- XI. Document Precedence: PASS — element/`lastDiscipline`/ordering follow stored-values over product prose.
- XII. Spec Kit Process: PASS — plan artifacts under `specs/003-history/`; decisions already in architecture/product docs.

### Post-design (Phase 1 re-check)

Re-evaluated after `research.md`, `data-model.md`, `contracts/`, and `quickstart.md`: **PASS** (no new violations). Design keeps Domain pure, reuses a single rules file and existing protocols, places UI copy in the String Catalog (including 6d plurals), derives board size, and confines tests to `BrenzTests/{Domain,Data,Presentation}`.

## Project Structure

### Documentation (this feature)

```text
specs/003-history/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/           # Phase 1 output (/speckit-plan command)
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (repository root)

```text
Brenz/
├── App/                          # AppContainer wires ObserveWeekHistory, ChangeClassDiscipline, DeleteClass
├── Core/
├── Domain/
│   ├── Entities/                 # ClassLog, WaterLog, UserProfile (reuse)
│   ├── Rules/
│   │   ├── BoardLayout.swift     # History minimumRows=3, cap 12, +N; size derived
│   │   ├── HourBlockBucket.swift # 3-hour half-open windows (FR-007)
│   │   ├── PetStateCalculator.swift  # reuse; recalculate after delete
│   │   └── StageCalculator.swift     # reuse; stageReached never lowered
│   ├── UseCases/
│   │   ├── ObserveWeekHistory.swift
│   │   ├── ChangeClassDiscipline.swift  # owned by 003 (moved from 002)
│   │   └── DeleteClass.swift            # reuse from 002
│   └── Repositories/             # TrainingRepository, ProfileRepository protocols (reuse)
├── Data/
│   └── Firebase/                 # reuse ClassLogDTO, TrainingRepositoryImpl, firestore.rules path
├── Presentation/
│   ├── History/
│   │   ├── HistoryView.swift           # replaces 002 shell
│   │   ├── HistoryViewModel.swift
│   │   ├── LogSheetView.swift          # 6b–6e states
│   │   └── LogSheetViewModel.swift
│   ├── Shared/
│   │   └── DayBoard.swift              # reuse from 002; History size/minimumRows
│   └── …Home/Settings…
└── Resources/
    └── Localizable.xcstrings           # History + 6d plural keys (0/1/n, today vs other day)
Packages/
└── BrenzDesignSystem/
    ├── Components/                     # BrenzTabBar, grid, bottle, translucent sheet, BrenzSuccessMark, chart bars
    └── …Reduce Motion modifier (001)…
BrenzTests/
├── Domain/           # BoardLayout, HourBlockBucket, ChangeClassDiscipline, DeleteClass effects, ordering
├── Data/             # only if DTO/update-discipline mapping needs coverage
└── Presentation/     # HistoryViewModel: future days not tappable; Save inactive; 6d Reduce Motion
firebase/             # existing firestore.rules + emulator tests (reuse; classLogs update: discipline only)
docs/
├── architecture/
└── product/
```

**Structure Decision**: Feature code lives under `Presentation/History/` (replace shell), Domain use cases/rules listed above, and catalog keys in `Resources/Localizable.xcstrings`. Shared board stays in `Presentation/Shared/DayBoard`. Design-system components and Reduce Motion stay in `Packages/BrenzDesignSystem`—add `BrenzSuccessMark` / translucent sheet there only if 001/002 did not already ship them. No new Data protocols, no second `firestore.rules`, no new test root folders.

## Complexity Tracking

> No Constitution Check violations. Table left empty.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| — | — | — |

## Cut Order (time-short)

If delivery time is short, cut in this order (product scope §8 numbering; History feature remains shippable with earlier cuts):

| Local cut # | Product scope §8 id | What is cut |
|-------------|---------------------|-------------|
| 1 | **2** | Hourly charts — keep Classes/Water day totals; omit 3-hour block charts |
| 2 | **3** | Tatami growth — History tatami can stay fixed at rows needed for a simpler layout (still honor min 3 / cap 12 / `+N` if already cheap; otherwise prefer fixed rows over animated growth) |
| 3 | **5** | 6d animation — after delete, close the sheet immediately (confirmation + refresh remain); still honor Reduce Motion path as “no animation” |
| 4 | **6** | Change discipline — ship delete-only log sheet; omit `ChangeClassDiscipline` UI/use case until time allows |

Do not cut: current-week bar, day selection (past/today), large tatami filled cells, delete with stage preservation, offline apply.

## Phase Gates

- **Phase 0**: `research.md` — all technical unknowns resolved (none marked NEEDS CLARIFICATION).
- **Phase 1**: `data-model.md`, `contracts/`, `quickstart.md` — design complete; Constitution Check re-evaluated PASS.
- **Phase 2** (not this command): `/speckit-tasks` generates `tasks.md`.
