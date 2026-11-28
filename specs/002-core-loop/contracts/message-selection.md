# Contract: Message selection and catalog keys

**Feature**: 002-core-loop  
**Layer**: Domain `MessageSelector` + `MessageRotationStore`; Presentation resolves text

## Domain output

- `MessageKey` = group + index (and optional format args such as stage display name).
- Domain never returns visible copy or the nickname.

## Groups and keys

Pattern: `message.<group>.<n>` (en base + es). Stage names: `stage.1.name`, `stage.2.name` (passed as `%1$@`).

| Group | Count | Trigger |
|-------|-------|---------|
| `message.state.animated.*` | 5 | State message when pet is animated by classes |
| `message.state.normal.*` | 5 | Normal; (T) indexes only if no class today |
| `message.state.down.*` | 5 | Down |
| `message.state.lazy.*` | 5 | Lazy |
| `message.event.returnAfterLazy.1` | 1 | First class leaving lazy |
| `message.event.stageChange.*` | 2 | Stage increased; arg = localized stage name |
| `message.ambient.newUser.*` | 3 | Once ever; no classes yet |
| `message.ambient.middayNoWater.1` | 1 | After 12:00 local, 0 glasses, once per day |
| `message.water.add.*` | 5 | Add glass (lowest priority among add) |
| `message.water.half.*` | 3 | Reached ceil(goal/2) |
| `message.water.goal.*` | 4 | Reached goal; prefer over animated-by-classes bank when animated-by-water |
| `message.water.remove.*` | 3 | Remove glass |
| `message.water.droppedBelowGoal.*` | 2 | Only if state fell animated→normal from water |

Total bank: **44** messages. Ship full `en` and `es` in this feature.

## Priority

Non-water message priority MUST be: **class-event (return or stage) > new-user > midday > state message**.  
**Water add**: goal → half → add.  
**Water remove**: dropped-below-goal (if applicable) → remove.

## Rotation storage (UserDefaults)

- Used indexes per state; same state message for the whole local day.
- Water: avoid only the previous index of the same group.
- Once flags: new-user; midday for calendar day; **once-flag per stage reached** (stage-change does not re-fire for that stage after Undo + re-register).
- `MessageRotationStore` exposes `clear()`. **Sign Out** and **Delete Account** (feature 004) call it; do **not** wire those call sites in 002.

## Non-message catalog

| Key | Use |
|-----|-----|
| `home.notice.alreadyLoggedThisHour` | Same-hour rejection toast (2.5 s, no action; not a character message) |
| Home chrome keys (`home.button.*`, stage line formats, tab titles, shell titles) | UI chrome |

## Tests

- Every `MessageKey` exists in the String Catalog for `en` and `es`.
- Nickname never appears in any `message.*` string.
- Half/goal fire on the correct glass for goals 4, 8, and 16.
- Stage-change once-flag per stage: after the message fires for a stage, Undo + re-register of the threshold class does not show it again.
- Midnight / local-day boundary resets midday and state-day bubble flags (covered next to MessageRotationStore / MessageSelector tests).
