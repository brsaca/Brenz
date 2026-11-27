# Brenz — Architecture: Stored values and board element rotation

Date: 2026-11-27
Status: closed by Bren. No code.
Repo location: `docs/architecture/brenz-architecture-stored-values.md`
Base: `brenz-architecture-domain.md`, `brenz-architecture-data.md`.
Precedence: this document wins over the domain, data and scope documents where they differ (scope section 3 "Manual class log" on one class per tap, section 6 "Board elements", section 7 pending items, section 12 item 20).

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
- Selecting a discipline in the Home discipline sheet also writes `lastDiscipline` (one profile write, no class registered).
- Changing the discipline of an existing class (History) does NOT write `lastDiscipline`.
- If the field is missing, use the first discipline of `selectedDisciplines`.
- If `lastDiscipline` is not among `selectedDisciplines`, use the first discipline of `selectedDisciplines`.
- "First discipline of `selectedDisciplines`" means the first in catalog order (boxing, jiu_jitsu, muay_thai, kickboxing), not the first selected by time.
- `selectedDisciplines` is stored in catalog order.
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

## Class registration limit
- [DECIDED Nov 27] A day has at most one class per local clock hour (for example 14:00 to 14:59). Several classes per day still count, each in a different hour.
- The hour is the local clock hour of `registeredAt`, on the local day of the class.
- `RegisterClass` checks the day's classes before writing. If one exists in the same local clock hour, it writes nothing and returns a distinct outcome (`alreadyLoggedThisHour`).
- Home shows a short system notice (catalog key, `en` and `es`) when the outcome is returned. It is not a character message. No Undo is offered.
- Undo (Home) and delete (History) free the hour.
- Changing the discipline of a class does not change its hour.
- The check runs on the device with `Calendar`. Firestore rules do not enforce it.
- Two devices can still create two classes in the same hour while offline. History orders the day's classes by `registeredAt` ascending, then by log id ascending.

## Constraints
- Do not use a rolling 60-minute window for the class limit.
- Do not add a character message for a rejected class.
- Do not enforce the class limit in Firestore rules.
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
- A test covers a second class in the same local clock hour: rejected, nothing written.
- A test covers classes at 14:59 and 15:00: both accepted.
- A test covers Undo and delete: the hour is free again.
- A test covers two classes with the same hour and different ids: ordered by `registeredAt`, then by id.
- A test covers selecting a discipline in the Home sheet: persists `lastDiscipline` without creating a class log.
- A test covers editing a class discipline: leaves `lastDiscipline` unchanged.
- A test covers selecting kickboxing then boxing: the first discipline is boxing.

## Cut order
- If time is short: fixed order (week index = 0 always). Same function, constant `weekIndex`. Add this as cut 0.5, after the `min_version` lock screen and before the intro animations.
