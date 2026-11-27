# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]

**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`

**Note**: This template is filled in by the `/speckit-plan` command; its definition describes the execution workflow.

## Summary

[Extract from feature spec: primary requirement + technical approach from research]

## Technical Context

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.
-->

**Language/Version**: Swift (Xcode 16 or later)

**Primary Dependencies**: SwiftUI, Observation, Firebase iOS SDK via SPM (FirebaseAuth, FirebaseFirestore, FirebaseRemoteConfig), local package BrenzDesignSystem. No other third-party.

**Storage**: Firestore with offline persistence. UserDefaults only for message rotation.

**Testing**: Swift Testing for Domain and ViewModels with fake repositories. Firestore rules with the Firebase emulator in firebase/tests/. No UI tests.

**Target Platform**: iOS 17+, iPhone only.

**Project Type**: single iOS app with folders by layer + local package Packages/BrenzDesignSystem.

**Performance Goals**: works offline with default values; 60 fps UI.

**Constraints**: no animation libraries; design tokens only in views; en/es String Catalog; Reduce Motion respected; dark mode only; no Combine.

**Scale/Scope**: [domain-specific, e.g., screens/features in scope or NEEDS CLARIFICATION]

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- I. Layer Separation: Domain/Data/Presentation only; Domain pure; Firebase stays in Data/Firebase/
- II. Platform and Stack: iOS 17+, SwiftUI, MVVM `@Observable`, AppContainer; Firebase SPM only; BrenzDesignSystem local package
- III. Language and Naming: English for code/docs; user-facing text only in Localizable.xcstrings (en base, es)
- IV. Pure Domain Rules: state/stage/board/element/version are pure; `now` and `Calendar` as parameters
- V. Data Source and Stored Values: Firestore + offline; optimistic class/water writes; follow stored-values.md
- VI. Testing: Swift Testing for Domain and ViewModels; Firestore rules via emulator; no UI tests in v1
- VII. Content Voice: sarcasm on actions only; no guilt/punishment/death; no work/office/clothing
- VIII. UI and Design System: dark mode only; BrenzDesignSystem tokens; SF Pro; Reduce Motion respected
- IX. Scope Discipline: v1 scope only; out-of-v1 stays out; new features need a documented decision
- X. Release Safety: in-app account deletion; do not raise min_version early; no brand names like Tamagotchi
- XI. Document Precedence: stored-values > architecture > scope > project instructions/rules
- XII. Spec Kit Process: constitution → specify → plan → tasks → analyze (max 2) → implement; decisions in docs/

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/           # Phase 1 output (/speckit-plan command)
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Expand with feature-specific paths under this layout.
  Do not introduce web/backend option trees; constitution mandates this structure.
-->

```text
Brenz/
├── App/
├── Core/
├── Domain/
│   ├── Entities/
│   ├── Rules/
│   ├── UseCases/
│   └── Repositories/
├── Data/
│   ├── Firebase/
│   └── Local/
├── Presentation/
│   ├── Auth/
│   ├── Onboarding/
│   ├── Home/
│   ├── History/
│   ├── Settings/
│   └── Shared/
└── Resources/
    └── Localizable.xcstrings
Packages/
└── BrenzDesignSystem/
firebase/                    # Firestore rules and emulator tests
docs/
├── architecture/
└── product/
```

**Structure Decision**: [Document feature paths under Brenz/ (App, Core, Domain, Data, Presentation, Resources), Packages/BrenzDesignSystem, firebase/, docs/architecture/, docs/product/]

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |
