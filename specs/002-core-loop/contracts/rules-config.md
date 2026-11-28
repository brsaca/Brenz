# Contract: RulesConfig / Remote Config

**Feature**: 002-core-loop  
**Layer**: Domain `RulesConfig` + Data `RulesConfigRepository` (Firebase Remote Config)

## Keys and defaults

| Remote Config key | Domain field | Default | Notes |
|-------------------|--------------|---------|--------|
| `rules_animated_threshold` | `animatedThreshold` | `2` | Classes in window for animated |
| `rules_window_days` | `windowDays` | `7` | Lookback window |
| `rules_down_days` | `downDays` | `4` | Start of down band (`daysSinceLastClass >= downDays`) |
| `rules_lazy_days` | `lazyDays` | `7` | Lazy when `daysSinceLastClass >= lazyDays` |
| `rules_stage2_classes` | `stage2Classes` | `10` | Classes to reach stage 2 |
| `rules_water_goal_default` | `waterGoalDefault` | `8` | Used when profile goal absent |
| `min_version` | `minVersion` | `"0.0.0"` | No lock-screen UI in 002 |

## Derived active band (not a Remote Config field)

There is **no** RC key for “active days.” Pet **active** = `daysSinceLastClass < downDays`. With defaults (`downDays` 4, `lazyDays` 7) active is **0…3** days. Then:

- active + classes in `windowDays` ≥ `animatedThreshold` → animated
- active + below threshold → normal
- `downDays <= days < lazyDays` → down
- `days >= lazyDays` → lazy

## Forbidden keys (this feature)

- No Remote Config keys for board element rotation / `weekIndex` / `baseOrder`.
- No Remote Config key for the active band (derived from `downDays` only).

## Validation

- `RulesConfigRepository` maps fetched values → `RulesConfig`.
- Each invalid or out-of-range value is replaced by that field’s in-app default independently.
- Pair rule: if `downDays < 1` or `downDays >= lazyDays`, use in-app defaults for `downDays` and `lazyDays`.
- Offline / failed fetch → last activated values or `RulesConfig.default`.
- Background fetch on app open; does not block splash; production minimum interval 12 h (0 in debug).
- New values apply on the next state calculation (no forced Home reload).

## Consumers

- `PetStateCalculator` (uses derived active band from `downDays` / `lazyDays`)
- `StageCalculator` / `RegisterClass` (stage bump)
- Water goal fallback for bottle fraction and water message thresholds (half = ceil(goal/2)): use profile `waterGoal` when set (4…16), else `RulesConfig.default.waterGoalDefault`
- Settings editing of the goal is out of scope
