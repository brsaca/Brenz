---

description: "Task list template for feature implementation"
---

# Tasks: [FEATURE NAME]

**Input**: Design documents from `/specs/[###-feature-name]/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Swift Testing for Domain and ViewModels with fake repositories. Firestore
rules tested with the Firebase emulator in `firebase/tests/`. No UI tests. Tests are
written in the same task as the code. All suites MUST pass before a task is complete.
Do not add XCTest.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

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
- Paths below are samples — replace with real paths from plan.md

<!--
  ============================================================================
  IMPORTANT: The tasks below are SAMPLE TASKS for illustration purposes only.

  The /speckit-tasks command MUST replace these with actual tasks based on:
  - User stories from spec.md (with their priorities P1, P2, P3...)
  - Feature requirements from plan.md
  - Entities from data-model.md
  - Endpoints from contracts/

  Tasks MUST be organized by user story so each story can be:
  - Implemented independently
  - Tested independently
  - Delivered as an MVP increment

  DO NOT keep these sample tasks in the generated tasks.md file.
  ============================================================================
-->

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create project structure per plan.md (`Brenz/App`, `Core`, `Domain`, `Data`, `Presentation`, `Resources`; `Packages/BrenzDesignSystem`; `firebase/`)
- [ ] T002 Confirm iOS 17 target; Firebase SDK via SPM with FirebaseAuth, FirebaseFirestore, FirebaseRemoteConfig only; no other package
- [ ] T003 [P] Confirm `Brenz/Resources/Localizable.xcstrings` exists for en + es

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

Examples of foundational tasks (adjust based on your project):

- [ ] T004 Create shared Domain entities/value types all stories depend on
- [ ] T005 [P] Define shared repository protocols in `Brenz/Domain/Repositories/`
- [ ] T006 [P] Wire composition root in `Brenz/App/AppContainer.swift` (inject Domain protocols; Data only here)
- [ ] T007 Configure Firebase and Firestore offline persistence in `Brenz/Data/Firebase/`
- [ ] T008 Configure Domain error cases/keys (no localized strings in Domain)
- [ ] T009 Confirm the app runs on the simulator with SweetPad

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - [Title] (Priority: P1) 🎯 MVP

**Goal**: [Brief description of what this story delivers]

**Independent Test**: [How to verify this story works on its own]

### Tests for User Story 1 (REQUIRED for new behavior) ⚠️

> **NOTE: Write these tests in the same task as the code. All suites MUST pass before a task is complete.**

- [ ] T010 [P] [US1] Domain test for [behavior] in `BrenzTests/Domain/[Name]Tests.swift`
- [ ] T011 [P] [US1] Firestore rules test in `firebase/tests/`

### Implementation for User Story 1

- [ ] T012 [P] [US1] Create [Entity] in `Brenz/Domain/Entities/[Entity].swift`
- [ ] T013 [P] [US1] Define repository protocol in `Brenz/Domain/Repositories/[Repo]Protocol.swift`
- [ ] T014 [US1] Implement repository in `Brenz/Data/Firebase/[Repo].swift` (depends on T012, T013)
- [ ] T015 [US1] Add view model in `Brenz/Presentation/[Feature]/[Feature]ViewModel.swift` (inject protocol)
- [ ] T016 [US1] Add SwiftUI view in `Brenz/Presentation/[Feature]/[Feature]View.swift`
- [ ] T017 [US1] Add en+es strings to `Brenz/Resources/Localizable.xcstrings` (no hardcoded UI copy)

**Checkpoint**: At this point, User Story 1 should be fully functional and testable independently

---

## Phase 4: User Story 2 - [Title] (Priority: P2)

**Goal**: [Brief description of what this story delivers]

**Independent Test**: [How to verify this story works on its own]

### Tests for User Story 2 (REQUIRED for new behavior) ⚠️

- [ ] T018 [P] [US2] Domain test for [behavior] in `BrenzTests/Domain/[Name]Tests.swift`
- [ ] T019 [P] [US2] Firestore rules test in `firebase/tests/`

### Implementation for User Story 2

- [ ] T020 [P] [US2] Create [Entity] in `Brenz/Domain/Entities/[Entity].swift`
- [ ] T021 [US2] Implement repository in `Brenz/Data/Firebase/[Repo].swift`
- [ ] T022 [US2] Add view model + SwiftUI view in `Brenz/Presentation/[Feature]/`
- [ ] T023 [US2] Add en+es strings to `Brenz/Resources/Localizable.xcstrings`; integrate with US1 if needed

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently

---

## Phase 5: User Story 3 - [Title] (Priority: P3)

**Goal**: [Brief description of what this story delivers]

**Independent Test**: [How to verify this story works on its own]

### Tests for User Story 3 (REQUIRED for new behavior) ⚠️

- [ ] T024 [P] [US3] Domain test for [behavior] in `BrenzTests/Domain/[Name]Tests.swift`
- [ ] T025 [P] [US3] Firestore rules test in `firebase/tests/`

### Implementation for User Story 3

- [ ] T026 [P] [US3] Create [Entity] in `Brenz/Domain/Entities/[Entity].swift`
- [ ] T027 [US3] Implement repository in `Brenz/Data/Firebase/[Repo].swift`
- [ ] T028 [US3] Add view + en/es strings; verify on the simulator with SweetPad

**Checkpoint**: All user stories should now be independently functional

---

[Add more user story phases as needed, following the same pattern]

---

## Phase N: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] TXXX [P] Documentation updates in docs/
- [ ] TXXX Code cleanup and refactoring
- [ ] TXXX Confirm all new strings have en + es in `Brenz/Resources/Localizable.xcstrings`
- [ ] TXXX Confirm views use BrenzDesignSystem tokens only; dark mode only; Reduce Motion respected
- [ ] TXXX [P] Additional Swift Testing coverage in `BrenzTests/`
- [ ] TXXX Confirm all test suites pass; verify on the simulator with SweetPad
- [ ] TXXX Run quickstart.md validation
- [ ] TXXX Commit in English after the completed task

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3+)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2 → P3)
- **Polish (Final Phase)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - May integrate with US1 but should be independently testable
- **User Story 3 (P3)**: Can start after Foundational (Phase 2) - May integrate with US1/US2 but should be independently testable

### Within Each User Story

- Tests pass before a task is complete
- Models before services
- Services before endpoints
- Core implementation before integration
- Story complete before moving to next priority

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational tasks marked [P] can run in parallel (within Phase 2)
- Once Foundational phase completes, all user stories can start in parallel (if team capacity allows)
- All tests for a user story marked [P] can run in parallel
- Models within a story marked [P] can run in parallel
- Different user stories can be worked on in parallel by different team members

---

## Parallel Example: User Story 1

```bash
# Launch all tests for User Story 1 together:
Task: "Domain test for [behavior] in BrenzTests/Domain/[Name]Tests.swift"
Task: "Firestore rules test in firebase/tests/"

# Launch Domain types for User Story 1 together:
Task: "Create [Entity] in Brenz/Domain/Entities/[Entity].swift"
Task: "Define repository protocol in Brenz/Domain/Repositories/[Repo]Protocol.swift"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1
4. **STOP and VALIDATE**: Test User Story 1 independently
5. Deploy/demo if ready

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 → Test independently → Deploy/Demo (MVP!)
3. Add User Story 2 → Test independently → Deploy/Demo
4. Add User Story 3 → Test independently → Deploy/Demo
5. Each story adds value without breaking previous stories

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together
2. Once Foundational is done:
   - Developer A: User Story 1
   - Developer B: User Story 2
   - Developer C: User Story 3
3. Stories complete and integrate independently

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Tests pass before a task is complete (Swift Testing; no XCTest)
- Commit in English after the completed task
- Stop at any checkpoint to validate story independently
- Avoid: vague tasks, third-party packages, hardcoded UI strings
- Avoid: literal colors/magic numbers in views, animation libraries, motion without Reduce Motion fallback
- Avoid: same file conflicts, cross-story dependencies that break independence
