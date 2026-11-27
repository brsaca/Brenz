# Brenz Constitution

## Core Principles

### I. Layer Separation
Domain, Data, and Presentation are the only layers. Dependencies MUST point
inward. Domain MUST NOT import Firebase, SwiftUI, or UIKit. Firebase types
MUST NOT leave `Data/Firebase/`. Presentation and ViewModels MUST NOT import
Firebase.
Rationale: keeps Domain pure and Firebase replaceable behind protocols.
Source: `docs/architecture/brenz-architecture-base.md`,
`docs/architecture/brenz-architecture-domain.md`,
`docs/architecture/brenz-architecture-data.md`.

### II. Platform and Stack
Minimum iOS is 17. UI MUST be SwiftUI. Presentation MUST use MVVM with
`@Observable` and an own `AppContainer`. Third-party libraries MUST NOT be
added except the Firebase SDK via SPM (`FirebaseAuth`, `FirebaseFirestore`,
`FirebaseRemoteConfig`). Analytics, Crashlytics, and Combine MUST NOT be used.
There MUST be one app module with folders by layer. The only local package
MUST be `Packages/BrenzDesignSystem`.
Rationale: locks a small, testable stack without extra dependencies.
Source: `docs/architecture/brenz-architecture-base.md`,
`docs/product/brenz-design-system-decision.md`.

### III. Language and Naming
Code, identifiers, Firestore fields, String Catalog keys, docs, comments, and
commits MUST be English. Spanish MUST appear only as user-facing text in
`Localizable.xcstrings` (`es`). Visible text MUST NOT appear in code. `en` MUST
be the catalog base language.
Rationale: one engineering language; localization stays in the catalog.
Source: `docs/architecture/brenz-architecture-base.md`,
`.cursor/rules/brenz-rule-language-naming.mdc`.

### IV. Pure Domain Rules
State, stage, board layout, element rotation, and version comparison MUST be
pure functions. `now` and `Calendar` MUST enter as parameters. Rules MUST NOT
read the clock (`Date()` or `Calendar.current`).
Rationale: same inputs yield the same result on every device and in tests.
Source: `docs/architecture/brenz-architecture-domain.md`,
`docs/architecture/brenz-architecture-stored-values.md`.

### V. Data Source and Stored Values
Firestore MUST be the only data source, with offline persistence. Class and
water writes MUST be optimistic. Stored enum values, `lastDiscipline`, reminder
day numbering, and board element rotation MUST follow stored-values.md.
Rationale: gym offline use and stable cross-device stored contracts.
Source: `docs/architecture/brenz-architecture-data.md`,
`docs/architecture/brenz-architecture-stored-values.md`.

### VI. Testing
Domain and ViewModels MUST use Swift Testing. Firestore security rules MUST be
tested with the emulator. UI tests MUST NOT ship in v1. Domain tests MUST run
without network.
Rationale: fast unit coverage of rules and ViewModels; rules proven on emulator.
Source: `docs/architecture/brenz-architecture-base.md`.

### VII. Content Voice
Sarcasm MUST target actions, never the person. Guilt, punishment, and mockery
MUST NOT appear. The pet MUST NOT die. Messages MUST NOT mention work, office,
or clothing.
Rationale: keeps the sarcastic tone supportive and training-focused.
Source: `docs/product/brenz-scope-v1.md` (sections 4–5),
`.cursor/rules/brenz-rule-messages.mdc`.

### VIII. UI and Design System
The app MUST ship dark mode only. Colors, typography, spacing, and shared
components MUST come from `BrenzDesignSystem` tokens. Typography MUST use
SF Pro. Standard SwiftUI controls MUST be used except `BrenzTabBar`. Reduce
Motion MUST be respected via the package modifier.
Rationale: one dark visual system with accessible motion.
Source: `docs/product/brenz-design-system-decision.md`,
`docs/architecture/brenz-architecture-presentation.md`,
`.cursor/rules/brenz-rule-design-system.mdc`.

### IX. Scope Discipline
Only v1 scope MUST be built. Items marked out of v1 MUST stay out. New features
MUST have a documented decision before work starts.
Rationale: protects the pilot timeline against scope creep.
Source: `docs/product/brenz-scope-v1.md`.

### X. Release Safety
In-app account deletion MUST work end to end. `min_version` MUST NOT be raised
before Apple approves that version. Brand names, including "Tamagotchi", MUST
NOT appear in the app or store listing.
Rationale: App Store compliance and safe forced-update rollout.
Source: `docs/product/brenz-scope-v1.md` (section 9),
`docs/architecture/brenz-architecture-data.md`.

### XI. Document Precedence
When sources conflict, precedence MUST be:
`docs/architecture/brenz-architecture-stored-values.md` >
architecture documents >
`docs/product/brenz-scope-v1.md` >
project instructions and Cursor rules.
Rationale: stored contracts and architecture override product prose and prompts.
Source: `docs/architecture/brenz-architecture-stored-values.md`,
`docs/architecture/brenz-architecture-base.md`,
`docs/product/brenz-scope-v1.md`.

### XII. Spec Kit Process
Work MUST follow Spec Kit order: constitution → specify → plan → tasks →
analyze (maximum 2 passes) → implement. Each important decision MUST be saved
as a Markdown file under `docs/`.
Rationale: decisions stay reviewable and implementation stays plan-driven.
Source: `docs/product/brenz-scope-v1.md` (section 8).

## Governance

This constitution governs Brenz engineering. Amendments MUST update this file,
bump `CONSTITUTION_VERSION` (MAJOR for incompatible principle changes, MINOR
for new or expanded principles, PATCH for clarifications), and set
`LAST_AMENDED_DATE` to the amendment day. Reviews and Spec Kit artifacts MUST
verify compliance with these principles. Runtime detail lives in the cited
`docs/` paths and `.cursor/rules/brenz-rule-*.mdc`; those files MUST NOT
contradict this constitution.

**Version**: 1.0.0 | **Ratified**: 2026-11-27 | **Last Amended**: 2026-11-27
