# Contract: Home surfaces and tab shells

**Feature**: 002-core-loop  
**Layer**: Presentation (+ BrenzDesignSystem components)

## Tab destinations

| Tab | Destination in 002 | Later |
|-----|--------------------|--------|
| Train | Full Home loop (Train tab → Home screen) | — |
| History | Placeholder shell (title / empty) | 003 replaces content |
| Settings | Placeholder shell (title / empty) | 004 replaces content |

- Custom `BrenzTabBar` from `BrenzDesignSystem` (items: title + icon + selection; no screen knowledge).
- Each tab owns its `NavigationStack`.

## Home vertical order (Train)

1. Discipline chip (hidden when exactly one selected discipline).
2. Speech bubble + close (X).
3. Character art (stage × state) + optional nickname + stage line.
4. Board card: tatami (`DayBoard` → package grid) + water bottle with +/−.
5. Main class button (“Fui a clase” / EN catalog equivalent).
6. Floating tab bar.

## Interactions

| Action | Result |
|--------|--------|
| Main class button | `RegisterClass`; on success: bounce (or fade under Reduce Motion), board cell, 5 s Undo; on `alreadyLoggedThisHour`: system toast |
| Undo (≤ 5 s) | `UndoClass` for last success |
| Water + / − | `AddWaterGlass` / `RemoveWaterGlass` for **today** only; optimistic |
| Discipline chip | Translucent single-selection sheet of `selectedDisciplines` |
| Sheet selection | Persist `lastDiscipline` immediately; no class log |
| Bubble X | Hide until pet state or local day changes; water messages may reopen for `waterMessageDuration` |
| History / Settings tabs | Navigate to shells only |

## System notice (same hour)

- Catalog key: `home.notice.alreadyLoggedThisHour` (use this key everywhere; never a shorter alias)
- Presentation: **toast**, auto-dismiss after **2.5 s**, **no action** button, **no Undo** for that rejection
- Must not cancel or hide an open Undo for a prior successful class
- Languages: `en` and `es`
- Not a `message.*` character line

## Water message duration

- Presentation constant: `waterMessageDuration = 4` seconds (then return to state message; may reopen a closed bubble)

## Design-system components built in 002

| Component | Package API shape (conceptual) |
|-----------|--------------------------------|
| `BrenzTabBar` | Items + selection binding |
| Water bottle | Fill `0…1` (+ motion tokens / Reduce Motion) |
| Tatami / isometric grid | Rows, columns, filled cells, cap / overflow affordance |

Reduce Motion modifier: reuse from 001; all Home motion must respect it.

## Out of surface scope

- History log sheet, charts, delete/edit UI (003).
- Settings rows, water-goal editor, account actions, lock screen (004 / other).
