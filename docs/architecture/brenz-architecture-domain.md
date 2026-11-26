# Brenz — Architecture: Domain

Date: 2026-11-26
Status: closed by Bren. No code.
Repo location: `docs/architecture/brenz-architecture-domain.md`
Base: `brenz-architecture-base.md`. Rule values: `docs/product/brenz-scope-v1.md`, section 4. Messages: `docs/product/brenz-message-bank.md`.

## Principles
- [DECIDED] Domain does not import Firebase or SwiftUI.
- [DECIDED] All entities are immutable `struct`s.
- [DECIDED] Every `id` is a non-optional `String`.
- [DECIDED] Domain knows no text. It returns keys, not text.

## Entities
- `ClassLog`: `id`, `discipline`, `elementId` (1 to 7), `registeredAt`.
- `WaterLog`: `id`, `registeredAt`. One glass is one event.
- `UserProfile`: `characterId`, `petNickname` (optional, maximum 10), `stageReached`, `waterGoal` (4 to 16), `onboardingCompleted`, `selectedDisciplines`, `reminder` (days and time; empty if off).
- `Character`: `id`, `name`, `assetPrefix`.
- `BoardElement`: `id`, `assetName`, `weekday`, `discipline` (optional).
- `BoardTheme`: `id`, `gridColumns`, `maxCells`.
- `AuthUser`: `id`, `email`.

## Enumerations
- `Discipline`: four valid cases (boxing, jiu-jitsu, muay thai, kickboxing) plus `.unknown`.
  - `allCases` returns only the four valid ones.
  - `.unknown` is never shown in pickers and never written back to Firestore.
  - A class with `.unknown` counts for state and stage.
- `PetState`: `animated`, `normal`, `down`, `lazy`.
- `Stage`: numeric key 1 and 2. Key 3 is reserved for v1.1.

## Rules (pure functions)
- `RulesConfig`: struct with the animated threshold, window in days, days for down, days for lazy, classes per stage, default water goal and `minVersion`. Default values live in `RulesConfig.default`.
- `PetStateCalculator`: receives classes, today's water, `RulesConfig`, `now` and `Calendar`. Returns `PetState`. Applies the scope rules, section 4.
- `StageCalculator`: receives accumulated classes, `stageReached` and `RulesConfig`. Returns the greater of the calculated stage and `stageReached`. The stage never goes down.
- `BoardLayout`: receives the day's classes and `BoardTheme`. Returns rows, filled cells and "+N".
- `VersionComparator`: compares two versions. If `minVersion` is invalid, it uses `"0.0.0"`.
- `now` and `Calendar` always enter as parameters. The rules never read the clock.

## Use cases
One use case per action. Each one is a `struct` with an `execute` method.
- Classes and water: `RegisterClass`, `UndoClass`, `ChangeClassDiscipline`, `DeleteClass`, `AddWaterGlass`, `RemoveWaterGlass`.
- Reading: `ObservePetStatus`, `ObserveTodayBoard`, `ObserveWeekHistory`. They return `AsyncStream`.
- Account: `SignUp`, `SignIn`, `SignOut`, `ResetPassword`, `ChangePassword`, `DeleteAccount`.
- Profile: `CompleteOnboarding`, `UpdateProfile`.
- Ambient messages: one use case that Home calls on appear.
- `UndoClass` deletes only the last registered class, within 5 s. The ViewModel controls the timer.
- `SignUp` creates the profile with `onboardingCompleted = false`.
- `DeleteAccount` coordinates the three repositories in the order of `brenz-architecture-data.md`.

## Repository protocols
- `AuthRepository`: current session as `AsyncStream<AuthUser?>`, sign up, sign in, sign out, reset, change password, reauthenticate, delete user.
- `TrainingRepository`: add, update discipline, delete and observe classes; add and delete glasses; observe glasses by day or range.
- `ProfileRepository`: observe and update profile; save `stageReached`.
- `RulesConfigRepository`: delivers `RulesConfig`. Validates the values and replaces invalid ones with those of `default`.
- `MessageRotationStore`: stores used indexes per state, last index per water group and "once" flags.
- `CatalogProvider`: delivers `Character`, `BoardElement` and `BoardTheme`.
- `ReminderScheduler`: schedules, cancels and reconciles reminders.
- Catalogs have no data repository.

## Messages
- [DECIDED] Domain returns a `MessageKey` (group and index). Presentation converts it to text.
- [DECIDED] `MessageSelector` is pure. It uses `MessageRotationStore`. Four sources:
  1. State message, in `ObservePetStatus`.
     - Chosen once per state and local day. Stored. Lasts the whole day even if more emissions arrive.
     - No repeats until the 5 of the state are exhausted. Used indexes are stored.
     - (T) messages enter the group only if there is no class today.
  2. Class events, in `RegisterClass`. Returns an optional event: return after lazy (previous state was lazy) or stage change (the stage went up, with the stage name as argument).
  3. Water messages, in `AddWaterGlass` and `RemoveWaterGlass`. They return the key. Priority when adding: goal, half, add. When removing: dropped below the goal, or one glass less. They only avoid repeating the previous one of the same group.
  4. Ambient messages, on opening Home. New user: only if there are no classes and it was not shown before. No glasses at midday: after 12:00 local, with 0 glasses, once per day.
- `HomeViewModel` only displays. The water message lasts about 4 s and the base message returns.

## Catalogs
- [DECIDED] V1: one character (`fox`), seven elements (one per day), one theme (`tatami`: 3 columns, cap 12).
- [DECIDED] Asset name: `{character}_{stage}_{state}`. Example: `fox_s1_normal`.
- [DECIDED] Element of the day: look up (discipline, day). If it does not exist, use (day).
- New characters, holidays or themes are added as rows. Domain does not change.

## Constraints
- Do not import Firebase, SwiftUI or UIKit.
- Do not read `Date()` or `Calendar.current` inside the rules.
- Do not return visible text from Domain.
- Do not add use cases outside the list without deciding it.
- Do not store the character state. It is always calculated.

## Acceptance criteria
- `PetStateCalculator`: tests for each state, the limits of 3, 4, 6 and 7 days, and water that raises to animated and returns to normal.
- `StageCalculator`: deleting a class does not lower the stage.
- `BoardLayout`: 1 row in Home, minimum 3 in History, cap 12 and "+N".
- `MessageSelector`: the same message all day; no repeats until exhausted; (T) absent with a class today.
- With the water goal at 4 and at 16, "half" and "goal reached" fire on the correct glass.
- `VersionComparator`: invalid text returns `"0.0.0"`.
