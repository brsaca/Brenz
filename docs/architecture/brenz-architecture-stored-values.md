# Brenz — Architecture: Stored values and board element rotation

Date: 2026-11-27
Status: closed by Bren. No code.
Repo location: `docs/architecture/brenz-architecture-stored-values.md`
Base: `brenz-architecture-domain.md`, `brenz-architecture-data.md`.
Precedence: this document wins over the domain, data and scope documents where they differ (scope section 6 "Board elements", section 7 pending items, section 12 item 20).

## Stored enum values
- [DECIDED] Values are lowercase snake_case text. Once published, they never change.
- `Discipline` stored values: `boxing`, `jiu_jitsu`, `muay_thai`, `kickboxing`.
  - Swift cases: `boxing`, `jiuJitsu`, `muayThai`, `kickboxing`. Each declares its raw value explicitly.
  - Unknown stored value: read as `.unknown`. Never written back.
  - Firestore rules validate against these 4 values (`classLogs`, `selectedDisciplines`, `lastDiscipline`).
- `PetState`: not stored. Always calculated.
- `stageReached`: integer. Values 1 or 2. Value 3 reserved for v1.1.
- `characterId`: text. v1 value: `fox`.
- `elementId`: integer from 1 to 7.
- Reminder days: integers from 1 to 7. Monday = 1, Sunday = 7.
- Weekday conversion: `Calendar` numbers Sunday as 1. Convert to Monday = 1 in one single Domain function. One test covers the 7 days.

## Last used discipline
- [DECIDED] Field `lastDiscipline` in `users/{uid}`. Optional text. Same 4 valid values.
- `RegisterClass` updates `lastDiscipline` in the same operation.
- If the field is missing, use the first discipline of `selectedDisciplines`.
- Profile security rules validate the 4 values.
- Rejected: `UserDefaults`. Sign out deletes it and it does not survive a device change.
- `UserProfile` entity gains `lastDiscipline` (optional).

## Board element rotation
- [DECIDED] The element of a class depends on its local date. It rotates every week.
- Base order (`BoardElement.baseOrder`, replaces `weekday`):
  1. boxing glove
  2. hand wraps
  3. heavy bag
  4. shin guards
  5. mouthguard
  6. jump rope
  7. pads
- Reference date: Monday 2026-01-05 (local calendar).
- `dayIndex`: Monday = 0 … Sunday = 6, from the class local date.
- `weekIndex`: whole weeks between the reference Monday and the Monday of the class week.
- Formula: `elementId = ((dayIndex + weekIndex) mod 7) + 1`.
- Use the reference date, not the ISO week. The ISO week number jumps at year change.
- It is a pure function in Domain. It receives `Date` and `Calendar`. It never reads the clock.
- `RegisterClass` calculates `elementId` once and stores it.
- Editing the discipline of a class does not change `elementId`. Past weeks never change.
- Firestore rules do not change (`elementId` from 1 to 7, not editable).
- Catalog lookup: (discipline, `elementId`), then (`elementId`). `BoardElement.discipline` stays optional (v1.1).
- No new art. Same 7 images.
- "Monday = glove" holds only in week 0.

## Constraints
- Do not use random values for the element.
- Do not read `Date()` or `Calendar.current` inside the element function.
- Do not store the week index.
- Do not add Remote Config keys for the rotation.
- Do not add values to `Discipline` beyond the 4 listed.

## Acceptance criteria
- A test covers 8 consecutive weeks and the year change.
- A test covers the Sunday-first to Monday-first conversion for the 7 weekdays.
- A class keeps its `elementId` after its discipline changes.
- Two devices show the same element for the same class date.
- Unknown discipline text reads as `.unknown` and is not written back.

## Cut order
- If time is short: fixed order (week index = 0 always). Same function, constant `weekIndex`. Add this as cut 0.5, after the `min_version` lock screen and before the intro animations.
