---

description: "Task list for Core Loop (002) feature implementation"
---

# Tasks: Core Loop

**Input**: Design documents from `/specs/002-core-loop/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Swift Testing for Domain and ViewModels with fake repositories. Firestore
rules tested with the Firebase emulator in `firebase/tests/`. No UI tests. Tests are
written in the same task as the code. All suites MUST pass before a task is complete.
Do not add XCTest.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

**Assumption**: Spec 001 delivers Auth + onboarding profile to Home, plus the `ProfileRepository` protocol, `UserProfileDTO`, and `Brenz/Data/Firebase/ProfileRepositoryFirestore.swift` (001 T012). This feature extends that Firestore implementation for `lastDiscipline` / `stageReached` and does **not** add `ProfileRepositoryImpl`. Where other 001 artifacts are not yet present, foundational tasks create the shared Domain/Data pieces this feature needs (`UserProfile` fields, weekday helper, `firebase/firestore.rules` base + extensions).

**Navigation note**: Train tab → Home screen.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Domain**: `Brenz/Domain/` (Entities, Rules, UseCases, Repositories protocols only; no localized text)
- **Data**: `Brenz/Data/Firebase/` and `Brenz/Data/Local/` (DTOs, repository implementations)
- **Presentation**: `Brenz/Presentation/<Feature>/` and `Brenz/Presentation/Shared/` (views and ViewModels)
- **Composition root**: `Brenz/App/AppContainer.swift`
- **Design system**: `Packages/BrenzDesignSystem/`
- **Strings**: `Brenz/Resources/Localizable.xcstrings` (en base + es)
- **Firestore rules and tests**: `firebase/`
- **Unit tests**: `BrenzTests/` mirroring the layer folders

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project folders, package scaffold, catalog, and Firebase rules home for core loop

- [ ] T001 Create layer folders per plan.md: `Brenz/App/`, `Brenz/Core/`, `Brenz/Domain/{Entities,Rules,UseCases,Repositories}/`, `Brenz/Data/{Firebase,Local}/`, `Brenz/Presentation/{Home,History,Settings,Shared}/`, `Brenz/Resources/`, `BrenzTests/{Domain,Data,Presentation}/`, `firebase/tests/`, `Packages/BrenzDesignSystem/`
- [ ] T002 Confirm iOS 17 deployment target; add Firebase SPM products `FirebaseAuth`, `FirebaseFirestore`, `FirebaseRemoteConfig` only (no other third-party packages) in `Brenz.xcodeproj`
- [ ] T003 [P] Create `Brenz/Resources/Localizable.xcstrings` with `en` base + `es` (empty groups ready for `home.*`, `message.*`, `stage.*`)
- [ ] T004 [P] Scaffold local package `Packages/BrenzDesignSystem` (tokens placeholder + package manifest) and link it to the Brenz app target
- [ ] T005 [P] Reuse/extend the existing `firebase/firestore.rules` from 001 (do **not** create a second rules file) and ensure `firebase/tests/` harness entry point exists

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared Domain types, repos, design-system Home building blocks, tab shells, and optimistic training storage that every user story needs

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T075 [P] Import 8 placeholder pet images into `Brenz/Resources/Assets.xcassets` named `{character}_{stage}_{state}` (`fox`, stages `s1`/`s2`, states `animated`/`normal`/`down`/`lazy`) — required before US2 catalog/UI (T039, T040)
- [ ] T006 Create `Discipline` enum with stored values `boxing`, `jiu_jitsu`, `muay_thai`, `kickboxing` (+ `.unknown` on read only) and catalog order boxing → jiu_jitsu → muay_thai → kickboxing in `Brenz/Domain/Entities/Discipline.swift`
- [ ] T007 [P] Create/extend `UserProfile` in `Brenz/Domain/Entities/UserProfile.swift` with `lastDiscipline: Discipline?`, `stageReached: Int` (1 or 2; never decreases), `waterGoal: Int` (4…16), `selectedDisciplines: [Discipline]`, `petNickname: String?` (max 10), `characterId`, `onboardingCompleted`
- [ ] T008 [P] Implement Monday-based weekday conversion (Calendar Sunday=1 → Monday=1) covering all 7 weekdays in `Brenz/Domain/Rules/WeekdayConversion.swift` with tests in `BrenzTests/Domain/WeekdayConversionTests.swift`
- [ ] T009 [P] Create `RulesConfig` value type with defaults (`animatedThreshold` 2, `windowDays` 7, `downDays` 4, `lazyDays` 7, `stage2Classes` 10, `waterGoalDefault` 8, `minVersion` `"0.0.0"`); document that the active pet band is derived (`daysSinceLastClass < downDays`), not a field — in `Brenz/Domain/Rules/RulesConfig.swift`
- [ ] T010 [P] Create `ClassLog` (`id`, `discipline`, `elementId` 1…7, `registeredAt`) and `WaterLog` (`id`, `registeredAt`) in `Brenz/Domain/Entities/ClassLog.swift` and `Brenz/Domain/Entities/WaterLog.swift`
- [ ] T011 [P] Create `BoardElement` catalog type with `baseOrder` 1…7 (glove, wraps, bag, shin guards, mouthguard, jump rope, pads) in `Brenz/Domain/Entities/BoardElement.swift`
- [ ] T012 Implement pure `BoardElementId.compute(date:calendar:)` — reference Monday `2026-01-05`, `elementId = ((dayIndex + weekIndex) mod 7) + 1`, no `Date()` / `Calendar.current`; expose a **single internal switch** for cut 0.5 (`weekIndex = 0` constant inside the same function; no separate cut task) — in `Brenz/Domain/Rules/BoardElementId.swift` with tests for 8 consecutive weeks + year change in `BrenzTests/Domain/BoardElementIdTests.swift`
- [ ] T013 [P] Implement `BoardLayout` (3 columns, start 1 row, grow by rows, cap 12 cells, `+N` overflow; derived from today’s class count, not stored) in `Brenz/Domain/Rules/BoardLayout.swift` with tests in `BrenzTests/Domain/BoardLayoutTests.swift`
- [ ] T014 [P] Define repository protocols in `Brenz/Domain/Repositories/TrainingRepositoryProtocol.swift`, `Brenz/Domain/Repositories/RulesConfigRepositoryProtocol.swift`, `Brenz/Domain/Repositories/MessageRotationStoreProtocol.swift` (include `clear()`; Sign Out / Delete Account in 004 call it — do **not** wire in 002), `Brenz/Domain/Repositories/CatalogProviderProtocol.swift` (reuse `ProfileRepository` protocol from 001 — do **not** redefine it or add `ProfileRepositoryImpl`)
- [ ] T015 [P] Implement `ClassLogDTO` / `WaterLogDTO` mapping (client-generated ids; `registeredAt` device time, never `@ServerTimestamp`; unknown discipline → `.unknown` on read, never written back) in `Brenz/Data/Firebase/ClassLogDTO.swift` and `Brenz/Data/Firebase/WaterLogDTO.swift` with mapping tests in `BrenzTests/Data/ClassWaterDTOTests.swift`
- [ ] T016 Implement optimistic `TrainingRepositoryImpl` (Firestore local cache; no await server ack) for `users/{uid}/classLogs` and `users/{uid}/waterLogs` in `Brenz/Data/Firebase/TrainingRepositoryImpl.swift`
- [ ] T017 [P] Extend existing `UserProfileDTO` from 001 for `lastDiscipline` and monotonic `stageReached` mapping in `Brenz/Data/Firebase/UserProfileDTO.swift`
- [ ] T078 Extend `ProfileRepositoryFirestore` (001 T012 at `Brenz/Data/Firebase/ProfileRepositoryFirestore.swift`) with the methods 002 needs: write `lastDiscipline`, read/write `stageReached` — do **not** add `ProfileRepositoryImpl`
- [ ] T018 Extend single `firebase/firestore.rules` for `classLogs` (create: discipline ∈ four values, `elementId` 1…7, `registeredAt` ≤ request.time + 1 day; update: only `discipline`; immutable `elementId`/`registeredAt`; delete owner) and `waterLogs` (create: only `registeredAt`; update deny; delete owner); profile `stageReached` never decreases; add emulator cases from `contracts/firestore-class-water.md` in `firebase/tests/`
- [ ] T019 [P] Build `BrenzTabBar` (items: title + icon + selection binding; translucent `Material`; no screen knowledge) in `Packages/BrenzDesignSystem/Sources/BrenzDesignSystem/Components/BrenzTabBar.swift`
- [ ] T020 [P] Build water bottle component (fill `0…1`, Reduce Motion aware) in `Packages/BrenzDesignSystem/Sources/BrenzDesignSystem/Components/WaterBottleView.swift`
- [ ] T021 [P] Build tatami/isometric grid component (rows, columns, filled cells, overflow affordance) in `Packages/BrenzDesignSystem/Sources/BrenzDesignSystem/Components/TatamiGridView.swift`
- [ ] T022 Confirm/reuse Reduce Motion modifier from 001 in `Packages/BrenzDesignSystem`; all Home motion must use it
- [ ] T023 Create History and Settings placeholder shells (title / empty body; each tab owns a `NavigationStack`) in `Brenz/Presentation/History/HistoryPlaceholderView.swift` and `Brenz/Presentation/Settings/SettingsPlaceholderView.swift`
- [ ] T024 Wire root tab host with `BrenzTabBar` destinations Train / History / Settings in `Brenz/Presentation/Shared/MainTabView.swift` and inject deps in `Brenz/App/AppContainer.swift`
- [ ] T025 [P] Add Home chrome String Catalog keys (`home.button.*`, tab titles, shell titles, stage line formats) en + es in `Brenz/Resources/Localizable.xcstrings`
- [ ] T026 Confirm the app builds and launches on the iOS 17 simulator (SweetPad or Xcode)

**Checkpoint**: Foundation ready — user story implementation can now begin

---

## Phase 3: User Story 1 - Log a class with one tap (Priority: P1) 🎯 MVP

**Goal**: One-tap class registration from Home with `elementId` stored once, Undo for 5 s, at most one class per local clock hour, optimistic offline board update

**Independent Test**: On Home with ≥1 selected discipline, tap main class button; verify bounce/fade, board cell with day’s element, Undo window; second tap same hour → system notice and no write; tap in a later hour → another class

### Implementation for User Story 1

- [ ] T027 [P] [US1] Implement `RegisterClass` use case (inputs: discipline, `now`, `Calendar`, today’s logs, profile, `RulesConfig`; compute `elementId`; same local hour → `alreadyLoggedThisHour` with **no writes**; success → create `ClassLog` + set `lastDiscipline` + maybe bump `stageReached`; order logs by `registeredAt` then `id`) in `Brenz/Domain/UseCases/RegisterClass.swift` with tests (14:59 + 15:00 both accepted; same hour rejects with unchanged write count; **`registeredAt` of a Home class is always today’s local date**) in `BrenzTests/Domain/RegisterClassTests.swift` (kickboxing+boxing catalog fallback lives only in `CurrentDisciplineResolverTests` / T057 — do **not** duplicate here)
- [ ] T028 [P] [US1] Implement `UndoClass` and `DeleteClass` (delete log; free hour; never lower `stageReached`; `DeleteClass` stays in 002 because Undo needs it) in `Brenz/Domain/UseCases/UndoClass.swift` and `Brenz/Domain/UseCases/DeleteClass.swift` with tests proving Undo/Delete free the hour in `BrenzTests/Domain/UndoDeleteClassTests.swift`
- [ ] T030 [US1] Implement `ObserveTodayBoard` (today’s class logs → layout + stored `elementId` assets) in `Brenz/Domain/UseCases/ObserveTodayBoard.swift` with Domain tests in `BrenzTests/Domain/ObserveTodayBoardTests.swift`
- [ ] T031 [US1] Create `DayBoard` composing package grid + element assets (code-drawn placeholders until board art arrives) in `Brenz/Presentation/Shared/DayBoard.swift`
- [ ] T032 [US1] Create `HomeViewModel` (`@Observable`): resolve current discipline; call `RegisterClass`; own 5 s Undo timer for last **successful** registration only; same-hour rejection shows toast key `home.notice.alreadyLoggedThisHour` (2.5 s, no action, no Undo) and must not cancel/hide open Undo; optimistic path does not await network — include Presentation test: tapping “Fui a clase” and +/− updates UI with a fake repository whose write **never completes** — in `Brenz/Presentation/Home/HomeViewModel.swift` with tests in `BrenzTests/Presentation/HomeViewModelClassTests.swift`
- [ ] T033 [US1] Create `HomeView` vertical layout (bubble placeholder OK, character placeholder OK, board card, main class button, floating tab bar; full tatami growth is US6) using BrenzDesignSystem tokens only in `Brenz/Presentation/Home/HomeView.swift`
- [ ] T034 [US1] Add `home.notice.alreadyLoggedThisHour` and main button copy en + es in `Brenz/Resources/Localizable.xcstrings` (system toast is **not** a `message.*` line; use this key everywhere)
- [ ] T035 [US1] Wire `RegisterClass` / `UndoClass` / `ObserveTodayBoard` into `Brenz/App/AppContainer.swift` and verify offline class log updates UI without waiting for network

**Checkpoint**: User Story 1 fully functional and independently testable (MVP)

---

## Phase 4: User Story 2 - Pet state and stage follow the rules (Priority: P1)

**Goal**: Derived pet state/stage on Home from classes + water rules; `stageReached` monotonic; nickname + stage line; Reduce Motion on state change

**Independent Test**: With controlled class/water histories, open Home and verify state boundaries (normal/animated/down/lazy), stage at 10 classes, water-only animated day, stage never decreases on Undo

### Implementation for User Story 2

- [ ] T036 [P] [US2] Implement pure `PetStateCalculator` (no classes → normal; lazy if `daysSinceLastClass >= lazyDays`; down if `downDays <= days < lazyDays`; **active** if `days < downDays` → animated when classes in `windowDays` ≥ `animatedThreshold` else normal; water elevates normal→animated for today only when glasses ≥ goal; never lowers except undoing that raise; no separate active constant) in `Brenz/Domain/Rules/PetStateCalculator.swift` with boundary tests including **`downDays = 3` and `downDays = 5`** (active band shifts) in `BrenzTests/Domain/PetStateCalculatorTests.swift`
- [ ] T037 [P] [US2] Implement pure `StageCalculator` (`stageReached := max(stored, computedFromTotalClasses)`; never decreases; stage 1→2 at `stage2Classes` default 10) in `Brenz/Domain/Rules/StageCalculator.swift` with tests in `BrenzTests/Domain/StageCalculatorTests.swift`
- [ ] T038 [US2] Implement `ObservePetStatus` use case returning derived `PetStatus` (state, stage, nickname passthrough) in `Brenz/Domain/UseCases/ObservePetStatus.swift`
- [ ] T039 [P] [US2] Add local catalog provider mapping stage×state → pet art asset names `{character}_{stage}_{state}` (`characterId` v1 `fox`, stages `s1`/`s2`) in `Brenz/Data/Local/LocalCatalogProvider.swift`
- [ ] T040 [US2] Update `HomeViewModel` / `HomeView` to show character art for current stage×state, optional nickname (omit when empty), stage line (`stage.1.name` · class N of 10; `stage.2.name` · N classes), Reduce Motion fade instead of bounce/spin in `Brenz/Presentation/Home/`
- [ ] T041 [US2] Add `stage.1.name`, `stage.2.name`, and stage-line format keys en + es in `Brenz/Resources/Localizable.xcstrings`
- [ ] T042 [US2] Ensure `RegisterClass` persists increased `stageReached` only when stage rises; Undo of the threshold class does not lower stage or re-fire stage-change once-flag per stage (Domain tests in `BrenzTests/Domain/StagePersistenceTests.swift`; aligns with MessageRotationStore once-flag / T051)

**Checkpoint**: User Stories 1 and 2 both work independently

---

## Phase 5: User Story 3 - Add and remove water glasses on the board card (Priority: P1)

**Goal**: Board-card +/− water events for today only; bottle fill/fraction; optimistic offline; water can elevate pet to animated

**Independent Test**: From Home, press + until goal then − below goal; verify bottle level text, pet reaction, offline immediacy; − at zero is no-op; count may exceed goal with visual fill capped

### Implementation for User Story 3

- [ ] T043 [P] [US3] Implement `AddWaterGlass` (create `WaterLog` with `registeredAt = now`; today only; no count/time limit) in `Brenz/Domain/UseCases/AddWaterGlass.swift` with tests in `BrenzTests/Domain/AddWaterGlassTests.swift`
- [ ] T044 [P] [US3] Implement `RemoveWaterGlass` (delete most recent today’s glass by `registeredAt` then `id`; no-op at zero) in `Brenz/Domain/UseCases/RemoveWaterGlass.swift` with tests in `BrenzTests/Domain/RemoveWaterGlassTests.swift`
- [ ] T045 [US3] Extend `TrainingRepositoryImpl` observers/queries for today’s water logs in `Brenz/Data/Firebase/TrainingRepositoryImpl.swift`
- [ ] T046 [US3] Update `HomeViewModel` for +/−, bottle fill `min(1, todayGlasses/goal)`, fraction text `todayGlasses/goal` (may exceed), goal from profile `waterGoal` (4…16) else `RulesConfig.default.waterGoalDefault`; wire Reduce Motion on bottle/character bounce in `Brenz/Presentation/Home/HomeViewModel.swift` with tests (including − at zero no-op; +/− UI updates when fake repo write never completes) in `BrenzTests/Presentation/HomeViewModelWaterTests.swift`
- [ ] T047 [US3] Compose bottle +/− inside Home board card in `Brenz/Presentation/Home/HomeView.swift` and `Brenz/Presentation/Shared/DayBoard.swift`
- [ ] T048 [US3] Verify local-midnight “today” reset for **water**, **board**, and **pet state** via Domain/Presentation tests using injected `Calendar`/`Date` in `BrenzTests/Domain/WaterDayBoundaryTests.swift` and `BrenzTests/Presentation/HomeViewModelMidnightTests.swift` (bubble day flags → US4 / T079)

**Checkpoint**: Class + pet + water daily loop works on Home without leaving the Train tab (Train tab → Home screen)

---

## Phase 6: User Story 4 - Character messages in the speech bubble (Priority: P2)

**Goal**: Speech bubble with state / class-event / water / ambient messages, rotation, priorities, close (X), `waterMessageDuration` = 4 s water messages; full 44-key bank en + es

**Independent Test**: Exercise new-user, midday (after 12:00, 0 glasses), state message, return-after-lazy, stage-change, water goal/half/remove/dropped-below-goal, bubble close/reopen

### Implementation for User Story 4

- [ ] T049 [P] [US4] Define `MessageKey` (group + index + optional format args; no visible copy; never nickname) in `Brenz/Domain/Entities/MessageKey.swift`
- [ ] T050 [US4] Implement pure `MessageSelector` with priorities — non-water: class-event (return or stage) > new-user > midday > state; water add: goal → half (`ceil(goal/2)`) → add; water remove: dropped-below-goal (only if animated→normal from water) → remove; (T) normal only if no class today; same state message for whole local day; stage-change respects once-flag per stage — in `Brenz/Domain/Rules/MessageSelector.swift` with tests (goals 4/8/16 half+goal; nickname absent; stage once-flag) in `BrenzTests/Domain/MessageSelectorTests.swift`
- [ ] T051 [US4] Implement `MessageRotationStore` in UserDefaults (per-user used indexes per state; last water index per group; once flags new-user + midday for local day; **once-flag per stage reached**; expose `clear()` — Sign Out / Delete Account in feature 004 call it; do **not** wire those call sites in 002) in `Brenz/Data/Local/MessageRotationStore.swift` with tests covering stage once-flag and `clear()` in `BrenzTests/Data/MessageRotationStoreTests.swift`
- [ ] T079 [US4] Add midnight / local-day boundary test for bubble day flags (midday once-flag and state-day message reset) next to T051 in `BrenzTests/Domain/MessageBubbleDayBoundaryTests.swift`
- [ ] T052 [P] [US4] Add all 44 `message.*` keys + groups from `contracts/message-selection.md` in en + es in `Brenz/Resources/Localizable.xcstrings` (sarcasm at actions; no guilt/punishment/death; no work/office/clothing; no nickname placeholders)
- [ ] T053 [US4] Implement ambient/class-event hooks from `RegisterClass` / Home open / water use cases returning optional `MessageKey` in `Brenz/Domain/UseCases/` as needed (`ObserveAmbientMessage` or equivalent)
- [ ] T054 [US4] Update `HomeViewModel` for bubble visibility, X hide-until state/day change, Presentation constant `waterMessageDuration = 4` seconds (reopen then return to state message), event precedence in `Brenz/Presentation/Home/HomeViewModel.swift` with tests in `BrenzTests/Presentation/HomeViewModelMessageTests.swift`
- [ ] T055 [US4] Implement speech bubble UI (text from catalog via `MessageKey`, close control) in `Brenz/Presentation/Home/SpeechBubbleView.swift` and integrate in `HomeView.swift`

**Checkpoint**: Messages work on top of the class/water/pet loop without blocking taps

---

## Phase 7: User Story 5 - Change discipline from Home (Priority: P2)

**Goal**: Discipline chip + translucent single-selection sheet; persist `lastDiscipline` immediately; hide chip when only one discipline selected

**Independent Test**: With two+ selected disciplines, open sheet, pick another, leave Home without logging; reopen and confirm remembered discipline; with one discipline, chip hidden

### Implementation for User Story 5

- [ ] T056 [P] [US5] Implement `UpdateLastDiscipline` (profile write only; zero class logs) in `Brenz/Domain/UseCases/UpdateLastDiscipline.swift` with tests in `BrenzTests/Domain/UpdateLastDisciplineTests.swift`
- [ ] T057 [US5] Add current-discipline resolution helper (remembered if ∈ `selectedDisciplines`, else first selected in catalog order) in `Brenz/Domain/Rules/CurrentDisciplineResolver.swift` with tests (missing/invalid → first selected; kickboxing then boxing → boxing) in `BrenzTests/Domain/CurrentDisciplineResolverTests.swift`
- [ ] T058 [US5] Build floating translucent single-selection discipline sheet listing `selectedDisciplines` in `Brenz/Presentation/Home/DisciplineSheet.swift`
- [ ] T059 [US5] Add discipline chip (hidden when exactly one selected) and wire sheet → `UpdateLastDiscipline` immediately on selection in `Brenz/Presentation/Home/HomeView.swift` / `HomeViewModel.swift` with Presentation tests in `BrenzTests/Presentation/HomeViewModelDisciplineTests.swift`
- [ ] T060 [US5] Add discipline chip/sheet catalog strings en + es in `Brenz/Resources/Localizable.xcstrings`

**Checkpoint**: Multi-discipline logging path complete; single-discipline UX uncluttered

---

## Phase 8: User Story 6 - Tatami grows with today’s classes (Priority: P3)

**Goal**: Home tatami starts at one row, grows by rows of 3 up to 12 cells, shows `+N` overflow, shrinks on Undo; empty day shows outlined cells

**Independent Test**: Log or simulate 0…13+ classes in a day; verify rows, filled cells with stored elements, `+N`, and shrink after Undo

### Implementation for User Story 6

- [ ] T061 [US6] Wire `BoardLayout` outputs (rows, filled cells, overflow `+N`) into `DayBoard` / tatami grid so empty day = one row of outlined cells; growth as classes fill; cap 12; overflow label in `Brenz/Presentation/Shared/DayBoard.swift`
- [ ] T062 [US6] Ensure Undo within 5 s shrinks rows to remaining today’s class count (layout not stored) — cover in `BrenzTests/Domain/BoardLayoutTests.swift` and `BrenzTests/Presentation/HomeViewModelClassTests.swift`
- [ ] T063 [US6] Add overflow `+N` / empty-cell accessibility and chrome strings en + es if needed in `Brenz/Resources/Localizable.xcstrings`
- [ ] T064 [US6] Verify element on each filled cell is the log’s stored `elementId` (discipline change does not alter element; ChangeClassDiscipline is 003) in `BrenzTests/Domain/BoardElementIdTests.swift` and `BrenzTests/Presentation/HomeViewModelClassTests.swift`

**Checkpoint**: Board visual growth matches FR-014 independently

---

## Phase 9: User Story 7 - Rule values from remote configuration with offline defaults (Priority: P3)

**Goal**: Remote Config drives rule thresholds and default water goal; invalid values fall back per key; offline uses defaults; no lock-screen UI; no RC keys for element rotation

**Independent Test**: Network blocked → defaults; harness overrides → behavior changes; invalid remote values → defaults; profile waterGoal 4…16 wins over default for UI/messages

### Implementation for User Story 7

- [ ] T065 [P] [US7] Implement `RulesConfigRepositoryImpl` mapping RC keys `rules_animated_threshold`, `rules_window_days`, `rules_down_days`, `rules_lazy_days`, `rules_stage2_classes`, `rules_water_goal_default`, `min_version` → `RulesConfig`; each invalid/out-of-range field independently replaced by in-app default; if `downDays < 1` or `downDays >= lazyDays` use in-app defaults for that pair; no RC keys for element rotation or active band in `Brenz/Data/Firebase/RulesConfigRepositoryImpl.swift` with validation tests in `BrenzTests/Data/RulesConfigRepositoryTests.swift`
- [ ] T066 [US7] Fetch/activate Remote Config on app open without blocking splash (debug fetch interval 0; production minimum 12 h) and expose current `RulesConfig` via `AppContainer` in `Brenz/App/AppContainer.swift`
- [ ] T067 [US7] Confirm `PetStateCalculator`, `StageCalculator`, water goal fallback, and `RegisterClass` stage threshold consume repository `RulesConfig` (new values on next calculation) with Domain tests using injected configs in `BrenzTests/Domain/RulesConfigConsumptionTests.swift`
- [ ] T068 [US7] Document that `min_version` is loaded but lock-screen UI is out of scope for 002 (no Presentation lock UI)

**Checkpoint**: All user stories independently functional with tunable offline-safe rules

---

## Phase 10: Polish & Cross-Cutting Concerns

**Purpose**: Quality bar across stories — catalog completeness, design tokens, art placeholders, quickstart validation

- [ ] T069 [P] Confirm every `MessageKey` and Home chrome key (including `home.notice.alreadyLoggedThisHour`) resolves in `en` + `es` in `Brenz/Resources/Localizable.xcstrings` (no raw keys on device)
- [ ] T070 Confirm all Home views use BrenzDesignSystem tokens only; dark mode only; Reduce Motion respected for class bounce, bottle, element appear, tatami growth, state crossfade
- [ ] T071 [P] Run full Swift Testing suites under `BrenzTests/Domain`, `BrenzTests/Data`, `BrenzTests/Presentation` and Firestore emulator tests under `firebase/tests/`
- [ ] T072 Run manual checklist in `specs/002-core-loop/quickstart.md` on simulator (same-hour toast, Undo, airplane mode, shells, languages)
- [ ] T073 Code cleanup: no Firebase imports outside `Brenz/Data/Firebase/` and composition root; no `Date()` / `Calendar.current` inside Domain rules/use cases; confirm a single `firebase/firestore.rules` file (001 extended, no second file)
- [ ] T074 Commit in English (Conventional Commits) after the completed feature slice when asked
- [ ] T076 [P] Until board art delivery (**Sat Nov 28**), use code-drawn placeholder shapes for the 7 board elements behind `LocalCatalogProvider` / `DayBoard`; when delivered, import the 7 element images into `Brenz/Resources/Assets.xcassets` and point the catalog at them (no Domain API change)
- [ ] T077 [P] Add Domain tests with different injected `TimeZone`/`Calendar` covering **today**, **midday** (after 12:00 local), and **water day** boundaries in `BrenzTests/Domain/LocalCalendarBoundaryTests.swift`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — start immediately
- **Foundational (Phase 2)**: Depends on Setup — **BLOCKS** all user stories; depends on 001 for `ProfileRepository` / `UserProfileDTO` / `ProfileRepositoryFirestore` (extend via T078; no `ProfileRepositoryImpl`); T075 pet placeholders first so US2 T039/T040 can resolve assets
- **User Stories (Phases 3–9)**: All depend on Foundational completion
  - Prefer priority order: US1 → US2 → US3 → US4 → US5 → US6 → US7
  - US2/US3 can proceed in parallel after US1 Domain class APIs exist if staffed
  - US4 depends on US2 state + US3 water triggers for full coverage
  - US5 can start after Foundational (integrates with US1 logging)
  - US6 refines board UI; Domain `BoardLayout` already in Foundational
  - US7 can stub earlier via `RulesConfig.default` (including `RulesConfig.default.waterGoalDefault`); RC wiring is this phase
- **Polish (Phase 10)**: Depends on all desired user stories being complete
- **Navigation**: Train tab → Home screen

### User Story Dependencies

- **User Story 1 (P1)**: After Foundational — MVP; no dependency on other stories
- **User Story 2 (P1)**: After Foundational; uses class (and later water) inputs — independently testable with fakes
- **User Story 3 (P1)**: After Foundational; elevates pet when US2 calculator present
- **User Story 4 (P2)**: Best after US2 + US3 for real triggers; selector testable with fakes alone
- **User Story 5 (P2)**: After Foundational; integrates with US1 `RegisterClass` discipline input
- **User Story 6 (P3)**: After Foundational `BoardLayout` + US1 `DayBoard` wiring
- **User Story 7 (P3)**: After Foundational `RulesConfig`; consumers from US2/US3

### Within Each User Story

- Tests pass before a task is complete (same task as code)
- Domain rules/use cases before Data wiring
- Data before Presentation
- Story complete before moving to next priority when solo

### Parallel Opportunities

- Phase 1: T003, T004, T005 in parallel after T001/T002
- Phase 2: T075 first (pet placeholders); then T007–T011, T014–T015, T017, T019–T021, T025 marked [P]; T078 after T017
- US1: T027–T028 in parallel
- US2: T036–T037, T039 in parallel (after T075)
- US3: T043–T044 in parallel
- US4: T049 and T052 in parallel with selector work; T079 with T051
- US5: T056 resolver/tests parallel with sheet UI once protocol exists
- US7: T065 parallelizable with polish prep
- Polish: T076–T077 art + calendar boundary tests in parallel with T069/T071
- Different developers can own US5 vs US6 after Foundational + US1 shell

---

## Parallel Example: User Story 1

```bash
# Domain use cases for User Story 1 together:
Task: "Implement RegisterClass in Brenz/Domain/UseCases/RegisterClass.swift"
Task: "Implement UndoClass and DeleteClass in Brenz/Domain/UseCases/"

# After Domain + DayBoard:
Task: "Create HomeViewModel class logging + Undo timer in Brenz/Presentation/Home/HomeViewModel.swift"
Task: "Create HomeView in Brenz/Presentation/Home/HomeView.swift"
```

---

## Parallel Example: User Story 3

```bash
Task: "Implement AddWaterGlass in Brenz/Domain/UseCases/AddWaterGlass.swift"
Task: "Implement RemoveWaterGlass in Brenz/Domain/UseCases/RemoveWaterGlass.swift"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL — blocks all stories)
3. Complete Phase 3: User Story 1
4. **STOP and VALIDATE**: Independent Test for US1 + SC-001/SC-009
5. Demo optimistic class log + Undo + same-hour notice

### Incremental Delivery

1. Setup + Foundational → foundation ready
2. US1 → MVP class loop
3. US2 → pet feedback
4. US3 → water loop (full daily Home habits)
5. US4 → speech bubble personality
6. US5 → multi-discipline chip/sheet
7. US6 → tatami growth polish
8. US7 → Remote Config tuning
9. Polish → quickstart.md sign-off

### Parallel Team Strategy

1. Team completes Setup + Foundational together
2. After Foundational:
   - Dev A: US1 → US2
   - Dev B: US3 (integrate pet elevation with A)
   - Dev C: US5 shells/chip (then US6 board polish)
3. US4 after state + water APIs exist
4. US7 last or overlapping with polish

---

## Notes

- [P] tasks = different files, no dependencies on incomplete tasks
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Tests pass before a task is complete (Swift Testing; no XCTest)
- All unit/ViewModel tests live only under `BrenzTests/Domain`, `BrenzTests/Data`, `BrenzTests/Presentation`
- Commit in English after the completed task when asked
- Stop at any checkpoint to validate story independently
- Avoid: third-party packages beyond Firebase SPM, hardcoded UI strings, literal colors in views
- Avoid: animation libraries; motion without Reduce Motion fallback
- Avoid: second `firestore.rules` file; RC keys for element rotation or active band; lowering `stageReached`
- Avoid: History/Settings real content (shells only); `min_version` lock screen UI

## Moved / Unmapped Tasks

| ID | Disposition | Notes |
|----|-------------|--------|
| **T029** | **Moved to 003** | `ChangeClassDiscipline` (+ its Domain tests) belongs to History (003). Do **not** implement in 002. ID not reused. |
| **T028** | Kept in 002 | `DeleteClass` remains because Undo needs it. |
| **T075** | Kept in 002; **moved to Foundational start** | Pet placeholders before T039/T040. |
| **T078** | Appended; Foundational | Extend `ProfileRepositoryFirestore` (001); no `ProfileRepositoryImpl`. |
| **T079** | Appended; US4 next to T051 | Midnight bubble day-flag tests (split from T048). |

No other unmapped tasks: every remaining task maps to a US or foundational/polish concern. Stable IDs **T001–T077** retained; new IDs **T078–T079** appended.
