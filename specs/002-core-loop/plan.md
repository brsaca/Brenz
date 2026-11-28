# Implementation Plan: Core Loop

**Branch**: `002-core-loop` | **Date**: 2026-11-27 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/002-core-loop/spec.md`

**Note**: This template is filled in by the `/speckit-plan` command; its definition describes the execution workflow.

## Summary

Deliver the daily Home training loop after onboarding (001): one-tap class logging with Undo (5 s), water +/− as individual events, calculated pet state/stage, weekly board-element rotation stored on each class, character messages with local rotation, and the floating tab bar with History/Settings **placeholder shells** (full screens land in 003/004). **Train tab → Home screen.**

**Dependency on 001**: Feature 001 delivers Auth, the completed onboarding profile path to Home, the `ProfileRepository` protocol, `UserProfileDTO`, and `Brenz/Data/Firebase/ProfileRepositoryFirestore.swift` (001 T012). This feature **extends** that Firestore implementation for `lastDiscipline` and `stageReached`; it does **not** add `ProfileRepositoryImpl` or redefine the protocol.

Technical approach: pure Domain calculators and use cases (`RegisterClass`, `UndoClass`, `DeleteClass`, `AddWaterGlass`, `RemoveWaterGlass`, observers, `MessageSelector`); Firestore `classLogs` / `waterLogs` with optimistic writes; Remote Config for rule thresholds with in-app defaults (active pet band derived from `downDays`, no RC field); `MessageRotationStore` in UserDefaults; shared UI pieces (`BrenzTabBar`, bottle, tatami grid) in `Packages/BrenzDesignSystem`. Extend the single existing `firebase/firestore.rules` from 001—do not add a second rules file. `ChangeClassDiscipline` is deferred to feature 003.

## Technical Context

**Language/Version**: Swift (Xcode 16 or later)

**Primary Dependencies**: SwiftUI, Observation, Firebase iOS SDK via SPM (FirebaseAuth, FirebaseFirestore, FirebaseRemoteConfig), local package BrenzDesignSystem. No other third-party.

**Storage**: Firestore with offline persistence (`users/{uid}/classLogs`, `users/{uid}/waterLogs`, profile fields including `lastDiscipline` and `stageReached`). UserDefaults only for message rotation (and existing non-user flags from 001).

**Testing**: Swift Testing under `BrenzTests/Domain`, `BrenzTests/Data`, and `BrenzTests/Presentation` only. Fake repositories for use cases and ViewModels. Firestore rules via Firebase emulator under `firebase/tests/`. No UI tests.

**Target Platform**: iOS 17+, iPhone only.

**Project Type**: single iOS app with folders by layer + local package Packages/BrenzDesignSystem.

**Performance Goals**: optimistic class/water updates without waiting for the server (SC-001/SC-002 perceived &lt; 2 s); offline with defaults.

**Constraints**: no animation libraries; design tokens only in views; en/es String Catalog; Reduce Motion via existing package modifier from 001; dark mode only; no Combine; Domain never reads `Date()` / `Calendar.current`; no Remote Config keys for element rotation; one-class-per-hour enforced on device only (not in Firestore rules).

**Scale/Scope**: Home (Train) full loop; History and Settings as empty shells with tab navigation; `DeleteClass` Domain use case tested in 002 (Undo needs it); `ChangeClassDiscipline` deferred to 003; Remote Config rule keys (not `min_version` lock screen UI). Pet/board art depends on external assets (see Art dependency below).

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- I. Layer Separation: Domain/Data/Presentation only; Domain pure; Firebase stays in Data/Firebase/ — **PASS**
- II. Platform and Stack: iOS 17+, SwiftUI, MVVM `@Observable`, AppContainer; Firebase SPM only; BrenzDesignSystem local package — **PASS** (`BrenzTabBar`, bottle, tatami grid land in the package)
- III. Language and Naming: English for code/docs; user-facing text only in Localizable.xcstrings (en base, es) — **PASS** (system notice + 44 messages as catalog keys)
- IV. Pure Domain Rules: state/stage/board/element/version are pure; `now` and `Calendar` as parameters — **PASS** (`elementId` function takes `Date` + `Calendar`)
- V. Data Source and Stored Values: Firestore + offline; optimistic class/water writes; follow stored-values.md — **PASS**
- VI. Testing: Swift Testing for Domain and ViewModels; Firestore rules via emulator; no UI tests in v1 — **PASS** (tests only under Domain/Data/Presentation folders)
- VII. Content Voice: sarcasm on actions only; no guilt/punishment/death; no work/office/clothing; messages never use nickname — **PASS**
- VIII. UI and Design System: dark mode only; BrenzDesignSystem tokens; SF Pro; Reduce Motion respected — **PASS** (reuse 001 Reduce Motion modifier)
- IX. Scope Discipline: v1 scope only; shells for History/Settings; lock screen out of scope — **PASS**
- X. Release Safety: account deletion remains 001/004; do not raise `min_version` early; no Tamagotchi branding — **PASS**
- XI. Document Precedence: stored-values > architecture > scope — **PASS** (`baseOrder` / weekly rotation over fixed weekday map; device hour limit)
- XII. Spec Kit Process: plan artifacts under `specs/002-core-loop/`; decisions already in `docs/` — **PASS**

**Gate result (pre-research)**: PASS — no unjustified violations. Complexity Tracking left empty.

### Post-design re-check (after Phase 1)

| Principle | Design artifact check | Result |
|-----------|----------------------|--------|
| I–II | Domain use cases + Data Firebase DTOs; package components only; no Firebase in Presentation | PASS |
| III | Catalog keys for Home, notices, 44 messages (`en`/`es`) in contracts | PASS |
| IV | `BoardElementId.compute(date:calendar:)` documented; no clock reads | PASS |
| V | Optimistic writes; stored-values formula/`lastDiscipline`/hour limit in data-model + contracts | PASS |
| VI | Tests only under Domain/Data/Presentation; emulator cases listed | PASS |
| VII–VIII | Message voice + Reduce Motion + BrenzTabBar/bottle/grid in package | PASS |
| IX | History/Settings shells; lock screen cut 0; no second rules file | PASS |
| X–XII | No early `min_version` raise; decisions cited from `docs/` | PASS |

**Gate result (post-design)**: PASS.

## Project Structure

### Documentation (this feature)

```text
specs/002-core-loop/
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
├── App/                          # AppContainer wires training/message/rules deps
├── Core/
├── Domain/
│   ├── Entities/                 # ClassLog, WaterLog, UserProfile(+lastDiscipline), catalogs
│   ├── Rules/                    # PetStateCalculator, StageCalculator, BoardLayout,
│   │                             # BoardElementId, weekday conversion, MessageSelector, RulesConfig
│   ├── UseCases/                 # RegisterClass, UndoClass, DeleteClass (002),
│   │                             # AddWaterGlass, RemoveWaterGlass, UpdateLastDiscipline,
│   │                             # ObservePetStatus, ObserveTodayBoard, ambient messages
│   │                             # (ChangeClassDiscipline → 003)
│   └── Repositories/             # TrainingRepository, RulesConfigRepository,
│                                 # MessageRotationStore, CatalogProvider protocols
│                                 # (ProfileRepository protocol reused from 001)
├── Data/
│   ├── Firebase/                 # ClassLogDTO, WaterLogDTO, TrainingRepositoryImpl,
│   │                             # RulesConfigRepositoryImpl (Remote Config);
│   │                             # extend UserProfileDTO + ProfileRepositoryFirestore (001)
│   └── Local/                    # MessageRotationStore (UserDefaults; clear() for 004), LocalCatalogProvider
├── Presentation/
│   ├── Home/                     # Train tab → Home screen; HomeView, HomeViewModel, discipline sheet;
│   │                             # waterMessageDuration = 4 s; same-hour toast 2.5 s
│   ├── History/                  # Placeholder shell only (003 replaces content)
│   ├── Settings/                 # Placeholder shell only (004 replaces content)
│   └── Shared/                   # DayBoard (composes package grid + entity assets)
└── Resources/
    ├── Assets.xcassets           # pet {character}_{stage}_{state}; board elements when delivered
    └── Localizable.xcstrings     # home.*, message.*, stage.*, home.notice.alreadyLoggedThisHour
Packages/
└── BrenzDesignSystem/
    └── …/Components/             # BrenzTabBar, water bottle, tatami/isometric grid
                                  # (Reduce Motion modifier already from 001)
BrenzTests/
├── Domain/                       # calculators, elementId, weekday, use cases, MessageSelector
├── Data/                         # DTO mapping, RulesConfig validation (no live Firebase in unit tests)
└── Presentation/                 # HomeViewModel (Undo timer, sheet → lastDiscipline, same-hour notice)
firebase/
├── firestore.rules               # SAME file as 001; extend classLogs + waterLogs (no second file)
└── tests/                        # emulator cases for new collections
docs/
├── architecture/                 # stored-values, domain, data, presentation (authority)
└── product/                      # scope, mockups, message bank, design system
```

**Structure Decision**: Follow constitution folder layout. Feature code lives under Home + Shared + Domain/Data pieces above. History/Settings folders get navigable shells only. Design-system components required by Home (tab bar, bottle, grid) are built in this feature inside `Packages/BrenzDesignSystem`. Reuse the single existing `firebase/firestore.rules` from 001—never create a second rules file.

### Art dependency (external)

| Asset set | Count / naming | Status |
|-----------|----------------|--------|
| Pet placeholders | 8 images: `{character}_{stage}_{state}` with `fox`, stages `s1`/`s2`, states `animated`/`normal`/`down`/`lazy` in `Brenz/Resources/Assets.xcassets` | Import placeholders in 002 so `LocalCatalogProvider` resolves names |
| Board elements | 7 training-object images (glove…pads) | Expected delivery **Sat Nov 28**. Until then, Home uses **code-drawn placeholder shapes** behind `LocalCatalogProvider`; swap to real assets when delivered without Domain API changes |

Implementation must not block on final board art: placeholders ship first.

## Complexity Tracking

> No constitution violations requiring justification.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| — | — | — |

## Cut Order (if time is short)

Aligned with architecture base + stored-values + user plan notes:

0. Lock screen for `min_version` — **out of this feature** (field/comparator may exist; UI deferred).
0.5. Fixed element order: **same** `BoardElementId.compute` function with an internal constant `weekIndex = 0` (still pure; still stored once). Prefer shipping real `weekIndex` first. Task **T012** exposes that single switch inside the function—**no separate cut task**.
Then apply scope section 8 / base cut list (intro animations, History charts, tatami growth, etc.) as later features allow.

Do **not** cut: one-class-per-hour, optimistic writes, calculated pet state, `stageReached` monotonicity, message voice/keys, shells for History/Settings, or English catalog entries (last resort only at cut 8 of the global list).
