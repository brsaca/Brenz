# Data Model: Core Loop (002)

**Branch**: `002-core-loop` | **Date**: 2026-11-27  
**Authority**: `docs/architecture/brenz-architecture-stored-values.md` &gt; domain/data docs.

## Entities

### ClassLog

| Field | Type | Notes |
|-------|------|--------|
| `id` | `String` | Non-optional in Domain; client-generated document id on create |
| `discipline` | `Discipline` | Stored text: `boxing`, `jiu_jitsu`, `muay_thai`, `kickboxing`; unknown → `.unknown` on read, never written back |
| `elementId` | `Int` | 1…7; computed once at registration; immutable afterward |
| `registeredAt` | `Date` | Device local time; never `@ServerTimestamp` |

**Relationships**: Belongs to `users/{uid}`. Counts toward today’s board, pet state, and stage.

**Validation**:
- At most one class per local clock hour of its local day (enforced in `RegisterClass`, not rules).
- Ordered for a day: `registeredAt` ascending, then `id` ascending.

**State / outcomes** (`RegisterClass`):
- Success → create log + update `lastDiscipline` (+ optional stage bump persistence).
- `alreadyLoggedThisHour` → no writes; Presentation shows system notice; no Undo.

### WaterLog

| Field | Type | Notes |
|-------|------|--------|
| `id` | `String` | Client-generated |
| `registeredAt` | `Date` | Device time |

**Relationships**: One document = one glass under `users/{uid}/waterLogs/{id}`.

**Validation**:
- Home adds/removes only glasses whose local calendar day is today.
- No count or time-of-day limit.
- Remove deletes the most recent glass of today (by `registeredAt`, then `id`).

### UserProfile (fields used / extended by 002)

| Field | Type | Notes |
|-------|------|--------|
| `lastDiscipline` | `Discipline?` | Optional; Firestore text; same 4 valid values |
| `stageReached` | `Int` | 1 or 2; never decreases |
| `waterGoal` | `Int` | 4…16; Home uses profile when set, else `RulesConfig.default.waterGoalDefault` |
| `selectedDisciplines` | `[Discipline]` | Stored in catalog order |
| `petNickname` | `String?` | Max 10; display only; never in messages |
| `characterId` | `String` | v1: `fox` |
| `onboardingCompleted` | `Bool` | Assumed true when Home is shown |
| `reminder` | (from 001) | Untouched by core loop |

**Resolution — current discipline**:
1. If `lastDiscipline` ∈ `selectedDisciplines` → use it.
2. Else → first of `selectedDisciplines` in catalog order: boxing, jiu_jitsu, muay_thai, kickboxing.

### PetStatus (derived, not stored)

| Field | Type | Notes |
|-------|------|--------|
| `state` | `PetState` | `animated` \| `normal` \| `down` \| `lazy` |
| `stage` | `Int` | max(calculated from class count, `stageReached`) |
| `nickname` | `String?` | Passthrough for display |

**State rules** (thresholds from `RulesConfig`; **active band is derived**, not a constant or RC field):
- No classes ever → `normal` (new user).
- `daysSinceLastClass >= lazyDays` → `lazy`.
- `downDays <= daysSinceLastClass < lazyDays` → `down`.
- **Active** when `daysSinceLastClass < downDays`:
  - classes in `windowDays` ≥ `animatedThreshold` → `animated`
  - else → `normal`
- With defaults (`downDays` 4, `lazyDays` 7) active is **0…3** days.
- Water: if base is `normal` and today’s glasses ≥ goal → elevate to `animated` for today only.

### Today’s board (derived)

| Piece | Notes |
|-------|--------|
| Tatami layout | `BoardLayout(todayClasses, BoardTheme.tatami)` — 3 columns, start 1 row, cap 12, `+N` overflow; not stored |
| Cells | Each filled cell shows the log’s stored `elementId` asset |
| Bottle | Fill = min(1, todayGlasses / goal); fraction text = `todayGlasses/goal` (may exceed) |

### BoardElement (catalog)

| Field | Type | Notes |
|-------|------|--------|
| `id` | `String` | Catalog id |
| `assetName` | `String` | Art asset |
| `baseOrder` | `Int` | 1…7 (replaces older `weekday` naming) |
| `discipline` | `Discipline?` | Optional; v1.1 per-discipline art |

Base order: 1 glove, 2 wraps, 3 bag, 4 shin guards, 5 mouthguard, 6 jump rope, 7 pads.

### RulesConfig

| Field | Default | Remote Config key |
|-------|---------|------------------------------|
| `animatedThreshold` | 2 | `rules_animated_threshold` |
| `windowDays` | 7 | `rules_window_days` |
| `downDays` | 4 | `rules_down_days` |
| `lazyDays` | 7 | `rules_lazy_days` |
| `stage2Classes` | 10 | `rules_stage2_classes` |
| `waterGoalDefault` | 8 | `rules_water_goal_default` |
| `minVersion` | `"0.0.0"` | `min_version` (no lock UI in 002) |

No field for “active days” — active band = `daysSinceLastClass < downDays`.

Invalid remote values → replace that field with its in-app default. If `downDays < 1` or `downDays >= lazyDays`, use in-app defaults for that pair. No RC keys for element rotation.

### MessageRotationStore (local)

Per-user UserDefaults:
- Used indexes per pet state (no-repeat until bank exhausted).
- Last index per water message group.
- Once flags: new-user shown; midday shown for local day; **once-flag per stage reached** (stage-change message does not re-fire for that stage).

Exposes `clear()`. **Sign Out** and **Delete Account** (feature 004) call it. Do **not** wire those call sites in 002.

## Element id calculation

```text
referenceMonday = 2026-01-05 (local calendar interpretation)
dayIndex        = mondayBasedWeekday(date) - 1   // Mon=0 … Sun=6
weekIndex       = wholeWeeks(referenceMonday → mondayOf(date's week))
elementId       = ((dayIndex + weekIndex) mod 7) + 1
```

Cut 0.5 (time short): same function; set internal constant `weekIndex = 0` (single switch inside `BoardElementId.compute` — no separate API or task).

## Transitions

### Class registration

```text
[idle] --tap Register--> check same local hour?
  --yes--> alreadyLoggedThisHour (notice; no Undo; prior Undo untouched)
  --no--> write ClassLog + lastDiscipline
         optionally raise stageReached
         start 5 s Undo window
           --Undo within 5 s--> delete that log (hour free); recalculate state
           --timeout--> Undo unavailable (delete only via History / 003)
```

### Water

```text
[today count n] --+--> create WaterLog(now); n+1; maybe water message / animated-by-water
[today count n>0] --- --> delete latest today WaterLog; n-1; maybe dropped-below-goal
[today count 0] --- --> no-op
[local midnight] --> today's editable set empties (prior days remain in Firestore)
```

### Stage

```text
stageReached := max(stageReached, computedFromTotalClasses)
never decreases on Undo or DeleteClass
stage-change message once per stage reached (once-flag per stage in MessageRotationStore; Undo + re-register does not clear it)
```

## Discipline catalog order

1. `boxing`  
2. `jiu_jitsu`  
3. `muay_thai`  
4. `kickboxing`  

Selecting kickboxing then boxing still yields **boxing** as the first selected when resolving fallbacks.
