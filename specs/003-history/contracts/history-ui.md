# Contract: History UI (screens 6 / 6b–6e)

**Feature**: 003-history  
**Package reuse**: `BrenzTabBar`, isometric grid, bottle (if shown), translucent sheet, chart bars (if not cut), `BrenzSuccessMark`, Reduce Motion modifier.  
**App reuse**: `DayBoard` in `Presentation/Shared/` (History large size).

---

## History screen

| Element | Contract |
|---------|----------|
| Week bar | Exactly Mon–Sun of current local week; no prev/next |
| Today cell | Lime border + bold day number; selectable |
| Past + classes | Checkmark in lime circle; selectable |
| Past + empty | Number only; selectable |
| Future | Dimmed; **not** tappable |
| Default selection | Today |
| On select past/today | Update day label (Presentation localizes `DayLabelKind` via `Localizable.xcstrings`; Domain never holds “Today”/date copy), large tatami (≥3 rows), Classes card, Water card |
| Empty day tatami | Outlined empty cells, min 3 rows; Classes total 0 |
| Classes card | Day total + optional six-block chart (cut 2 drops chart) |
| Water card | Day total + optional chart; **no** list, **no** +/− |
| Filled cell tap | Opens log sheet (6b); empty cell tap → no-op |
| Overflow `+N` | Not tappable |
| Tab bar | Remains visible behind sheets (`BrenzTabBar`) |
| Loading | None (cache-first) |
| Locale | Device Spanish → Spanish strings; else English |

---

## Log sheet — edit (6b)

| Element | Contract |
|---------|----------|
| Presentation | Bottom translucent sheet; darkened History; tab bar behind |
| Header | Close (X) + title “Class log” / “Registro de clase” |
| Row | Day element asset, discipline label, time (“Today · …” / localized) |
| Chips | Always the four fixed disciplines (`boxing`, `jiu_jitsu`, `muay_thai`, `kickboxing`); single selection; not filtered by `selectedDisciplines` |
| Save | Inactive when selection == stored; active otherwise |
| Delete record | Danger secondary style → switch to 6c |
| Editable fields | Discipline only (no time/date controls) |

---

## Log sheet — confirm delete (6c)

| Element | Contract |
|---------|----------|
| Copy | Discipline + time; notice that class leaves tatami and stage does not go down |
| Delete | Solid danger confirm |
| Cancel | Returns to 6b without deleting |

---

## Log sheet — class deleted (6d)

| Element | Contract |
|---------|----------|
| Duration | ≈ 2.5 s then auto-close (Reduce Motion: skip sequence, close after delete) |
| Visual | `BrenzSuccessMark` + emptying progress |
| Figure | Today → `{count} classes today` / `{count} clases hoy`; other day → `{count} classes that day` / `{count} clases ese día`; plurals 0/1/n in catalog |
| Count | Remaining classes of the **chosen** day after delete |

---

## After delete (6e)

History stays on the same chosen day; tatami/cards/week checkmark refresh from stream. No character speech bubble.

---

## Accessibility / motion

- All animations go through the package Reduce Motion modifier (001).
- With Reduce Motion on: no 6d checkmark/ring/bar animation; sheet still closes after successful delete.
