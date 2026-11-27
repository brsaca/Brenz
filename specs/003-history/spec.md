# Feature Specification: History

**Feature Branch**: `003-history`

**Created**: 2026-11-27

**Status**: Draft

**Input**: User description: "History tab: current week, chosen day's tatami and cards, and log sheet to change discipline or delete a class. Replaces the History shell left by spec 002."

**Sources** (cited by section; not copied at length):
- `docs/product/brenz-scope-v1.md` — §3 (History; edit or delete), §4 (stage and state after delete), §6 (History; Log sheet 6b–6e; Board)
- `docs/product/brenz-mockups-v1.md` — History screens; screens 6a–6e; Log sheet rules
- `docs/architecture/brenz-architecture-domain.md` — ChangeClassDiscipline, DeleteClass, ObserveWeekHistory (behavior only)
- `docs/architecture/brenz-architecture-stored-values.md` — element stays fixed when discipline changes (behavior only)
- `docs/product/brenz-message-bank.md` — checked for delete reactions (none)

## Clarifications

### Session 2026-11-27

- Q: How should a class or water log's local time map onto the six chart labels (6a, 9a, 12p, 3p, 6p, 9p)? → A: Half-open windows [6–9), [9–12), [12–15), [15–18), [18–21), [21–24); times 00:00–05:59 count in 6a
- Q: When a day has more than 12 classes, which classes appear as the 12 filled tatami cells (and are therefore tappable)? → A: First 12 by local time ascending; overflow only in +N (not tappable)
- Q: Server rejection of an optimistic write? → A: Not shown to the user in v1; only logged (data architecture).
- Q: If two or more classes on the same day share the exact same local time, what order decides which enter the 12 filled cells? → A: At most one class per hour at registration (spec 002); if two devices create two classes in one hour while offline, order by registration timestamp ascending, then by log id ascending

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Browse the current week and a day's board (Priority: P1)

A signed-in user opens History and sees only the current week (Monday through Sunday). They choose a past day or today and see that day's label, large tatami, and the day's Classes and Water summary cards.

**Why this priority**: Without seeing the week and a day's board, edit and delete have nowhere to start. This replaces the placeholder History shell from the core loop.

**Independent Test**: With at least one class logged today and one on a past weekday of the same week, open History, confirm the week bar states, select each day, and verify the tatami and card totals match that day—without opening the log sheet.

**Acceptance Scenarios**:

1. **Given** the user is signed in and opens History, **When** the screen loads, **Then** they see the current week Monday to Sunday with no previous/next week controls, and today's cell has a lime border and a bold day number.
2. **Given** a past day in the current week has at least one class, **When** History is shown, **Then** that day shows a checkmark (lime circle) instead of only the number.
3. **Given** a past day in the current week has no classes, **When** History is shown, **Then** that day shows only its number (no checkmark).
4. **Given** a future day in the current week, **When** the user looks at the week bar, **Then** that day appears dimmed and does not respond to taps.
5. **Given** History is open, **When** the user taps a past day or today, **Then** the day label, large tatami (at least three rows), Classes card, and Water card update to that day.
6. **Given** the chosen day has no classes, **When** the day view is shown, **Then** the tatami shows empty outlined cells (minimum three rows) and the Classes total is zero.
7. **Given** the device language is Spanish, **When** History is shown, **Then** all History texts appear in Spanish; **Given** any other language, **Then** texts appear in English.

---

### User Story 2 - Delete a class from the tatami (Priority: P1)

The user taps a filled tatami cell, confirms deletion, and the class is removed. The board, class totals, and pet state refresh immediately. The stage already reached does not go down. Offline deletion applies at once.

**Why this priority**: Correcting a wrong log is essential in the gym; stage must never punish a correction.

**Independent Test**: Log two classes on a day, open History, delete one after confirmation, and verify the tatami cell count, Classes total, and (on Home) pet state update while stage reached stays the same—then repeat once with network off.

**Acceptance Scenarios**:

1. **Given** a chosen day with at least one class on the tatami, **When** the user taps a filled cell, **Then** the log sheet opens from the bottom showing that class's day element, discipline, and time, with actions to change discipline or delete the record.
2. **Given** the log sheet is open, **When** the user chooses delete record, **Then** the same sheet switches to a confirmation that shows the discipline and time, states that the class is removed from the tatami and the stage does not go down, and offers Delete and Cancel.
3. **Given** the confirmation is shown, **When** the user chooses Cancel, **Then** the sheet returns to the log sheet without deleting the class.
4. **Given** the confirmation is shown, **When** the user confirms Delete, **Then** the class is removed, History refreshes without that log, the tatami returns to the number of rows needed (minimum three), the Classes card total drops by one, the pet state is recalculated from remaining classes, and the stage reached is unchanged.
5. **Given** the device is offline, **When** the user confirms Delete, **Then** the same immediate refresh occurs without requiring network.
6. **Given** Reduce Motion is on, **When** the user confirms Delete, **Then** no class-deleted animation plays and the sheet still closes after the delete completes.
7. **Given** a day has a class in a given local clock hour, **When** the user deletes that class, **Then** that hour is free again for a new registration elsewhere; no History action registers a class.

---

### User Story 3 - Change a class discipline (Priority: P2)

The user opens the log sheet for a class, picks a different discipline, and saves. Time, date, and the board element stay the same. Save stays inactive until the discipline actually changes.

**Why this priority**: Fixes a mis-tagged discipline without rewriting the day's board element or time.

**Independent Test**: Open a class log sheet, select another discipline, save, and confirm the cell still shows the same element and time while the discipline label updates; reopening with the same discipline leaves Save inactive.

**Acceptance Scenarios**:

1. **Given** the log sheet is open for a class, **When** the user has not changed the discipline, **Then** Save is inactive.
2. **Given** the log sheet is open, **When** the user selects a different discipline and taps Save, **Then** the sheet closes, History shows the updated discipline for that cell, and the time, date, and element remain unchanged.
3. **Given** the log sheet is open, **When** the user selects the same discipline already stored and looks at Save, **Then** Save stays inactive.
4. **Given** the device is offline, **When** the user saves a discipline change, **Then** the update applies immediately without requiring network.

---

### User Story 4 - Day summary cards with 3-hour charts (Priority: P2)

For the chosen day, the user sees Classes and Water cards, each with a total and a chart of six 3-hour blocks (6a, 9a, 12p, 3p, 6p, 9p). Water is read-only: no glass list and no add/remove controls.

**Why this priority**: Completes the day view promised by the product screens; totals alone are not enough to understand when training and water happened.

**Independent Test**: On a day with classes and glasses at known times, select that day and verify each card's total and which blocks show activity—without any water edit controls.

**Acceptance Scenarios**:

1. **Given** a chosen day with classes logged at different hours, **When** the Classes card is shown, **Then** it shows the day's class total and a 3-hour block chart covering 6a, 9a, 12p, 3p, 6p, and 9p, with each log counted in exactly one half-open block (`[6–9)`, `[9–12)`, `[12–15)`, `[15–18)`, `[18–21)`, `[21–24)`; times `00:00–05:59` in 6a).
2. **Given** a chosen day with water glasses logged at different hours, **When** the Water card is shown, **Then** it shows the day's glass total and the same 3-hour block chart structure and mapping rules as Classes.
3. **Given** History's Water card for any day, **When** the user views it, **Then** there is no event list and no control to add or remove glasses.
4. **Given** a log at a local time before 06:00, **When** the day chart is shown, **Then** that log contributes to the 6a block (and still to the day total).

---

### User Story 5 - Class-deleted animation before the sheet closes (Priority: P3)

After confirming delete, the sheet shows a short class-deleted success sequence (~2.5 s) and then closes. With Reduce Motion, that sequence is skipped.

**Why this priority**: Confirms the destructive action visually; the delete itself (P1) already works without it.

**Independent Test**: Confirm a delete with Reduce Motion off and observe the success sequence for about 2.5 seconds before the sheet closes; repeat with Reduce Motion on and confirm an immediate close without the sequence.

**Acceptance Scenarios**:

1. **Given** Reduce Motion is off and the user confirms Delete, **When** the delete succeeds, **Then** the same sheet switches to the class-deleted state (checkmark, status figure for the count of remaining classes of the chosen day, emptying progress) for about 2.5 seconds and then closes.
2. **Given** Reduce Motion is on and the user confirms Delete, **When** the delete succeeds, **Then** the sheet closes without playing the class-deleted animation.

---

### Edge Cases

- **Only class of the day**: Deleting the last class leaves an empty tatami with the minimum three outlined rows, Classes total `0`, and a chart with no bars; if that day is in the past, the week-bar checkmark for that day disappears.
- **Delete that lowers pet state**: After delete, pet state is recalculated from remaining classes (for example animated → normal, or toward down/lazy by attendance rules). Stage reached never decreases.
- **Delete offline**: Removal and History refresh happen immediately; sync when network returns does not undo the local result.
- **More than 12 classes on a day**: The tatami shows the first 12 classes by local time ascending as filled cells; classes after those 12 appear only in the `+N` overflow indicator and are not tappable until a delete frees a filled slot; tapping a filled cell opens that class's log sheet; deleting reduces overflow or filled cells accordingly (the next earliest overflow class, if any, becomes a filled cell).
- **One class per hour**: A day has at most one class log per local clock hour; the rule is enforced when a class is registered (spec 002). Two devices can create two classes in one hour while offline; History MUST then order that day's classes by registration timestamp ascending, then by log id ascending. Deleting a class frees its hour.
- **Week boundary (Monday)**: The week bar always starts on Monday of the current local week; Sunday is the last cell; no spill into the previous or next week.
- **Device time zone change**: "Today", past, future, and week membership follow the device's current local calendar; after a time zone change, the week bar and day labels re-evaluate against the new local day without rewriting stored log times.
- **Change discipline to the same value**: Save remains inactive; no write occurs.
- **Tap empty tatami cell**: No log sheet opens; nothing is edited or deleted.
- **Log before 06:00**: A class or water log with local time `00:00–05:59` still counts in the day total and appears in the 6a chart block.
- **Close log sheet without saving**: Dismissing with the close control discards an unsaved discipline selection and leaves the class unchanged.
- **Delete while viewing another day**: After delete, the user remains on the same chosen day with refreshed board and cards.
- **No character message on delete**: Deleting a class does not trigger a message from the message bank (no delete-specific message exists).

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: History MUST show only the current local week, Monday through Sunday, with no controls to move to other weeks. *(Source: scope §3 History; scope §6 History; mockups History (6))*
- **FR-002**: The current day MUST use a lime border and a bold day number. *(Source: scope §6 History; mockups History (6))*
- **FR-003**: A past day with at least one class MUST show a checkmark in a lime circle; a past day without classes MUST show only the number. *(Source: scope §6 History; mockups History (6))*
- **FR-004**: Future days MUST appear dimmed and MUST NOT be tappable. *(Source: scope §6 History)*
- **FR-005**: Tapping today or a past day MUST show that day's label, large tatami, Classes card, and Water card. *(Source: scope §6 History; mockups History (6))*
- **FR-006**: The History tatami MUST use the shared board for the chosen day, show at least three rows, grow when needed up to twelve cells, show `+N` above twelve classes, show empty outlined cells when empty, and shrink back to the rows needed after deletes. When more than twelve classes exist for the day, the twelve filled cells MUST be the earliest by local time ascending; overflow classes MUST appear only via `+N` and MUST NOT be tappable until a delete promotes them into a filled cell. *(Source: scope §6 Board; scope §6 History after delete 6e; mockups 6e; Clarifications 2026-11-27)*
- **FR-007**: Classes and Water cards MUST each show a day total and a six-block 3-hour chart (6a, 9a, 12p, 3p, 6p, 9p). Each log MUST count in exactly one half-open local-time window: `[06:00–09:00)`, `[09:00–12:00)`, `[12:00–15:00)`, `[15:00–18:00)`, `[18:00–21:00)`, `[21:00–24:00)`; times `00:00–05:59` MUST count in 6a. Classes and Water MUST use the same mapping. *(Source: scope §3 History; scope §6 History; mockups History (6); Clarifications 2026-11-27)*
- **FR-008**: Water on History MUST be read-only: totals and chart only—no glass event list and no add/remove. *(Source: scope §3 History; scope §6 History; user scope for this feature)*
- **FR-009**: Tapping a filled tatami cell MUST open the log sheet from the bottom over a darkened History, with the tab bar remaining behind. *(Source: scope §3 edit or delete; scope §6 Log sheet 6b; mockups 6b; Log sheet rules)*
- **FR-010**: The log sheet MUST show the day's element, the class discipline and time, single-selection discipline chips, Save, and Delete record; time and date MUST NOT be editable. *(Source: scope §6 Log sheet 6b; mockups 6b; Log sheet rules)*
- **FR-011**: Save MUST stay inactive when the selected discipline equals the stored discipline, and MUST become active only after a different discipline is selected. *(Source: scope §6 Log sheet; mockups Log sheet rules)*
- **FR-012**: Saving a new discipline MUST update only the discipline; time, date, and board element MUST stay unchanged. *(Source: scope §3 edit or delete; stored-values element behavior; domain ChangeClassDiscipline behavior; mockups flow 10)*
- **FR-013**: Delete record MUST switch the same sheet to a confirmation that shows discipline and time, the notice that the class leaves the tatami and the stage does not go down, Delete, and Cancel (back to the log sheet). *(Source: scope §6 Log sheet 6c; mockups 6c; Log sheet rules)*
- **FR-014**: Confirming Delete MUST remove the class, recalculate pet state from remaining classes, leave stage reached unchanged, refresh the tatami and Classes card, and leave the user on History for that day. *(Source: scope §4 stage/state after delete; scope §6 Log sheet 6e; domain DeleteClass / StageCalculator behavior; mockups 6e; Log sheet rules)*
- **FR-015**: After confirm Delete, with Reduce Motion off, the sheet MUST show the class-deleted sequence (including the count of remaining classes of the chosen day) for about 2.5 seconds and then close; with Reduce Motion on, the animation MUST be skipped and the sheet MUST still close after the delete. *(Source: scope §6 Log sheet 6d; mockups 6d)*
- **FR-016**: History browse, discipline change, and delete MUST work offline and apply immediately. *(Source: scope §3 offline use; domain week-history observe and class write behaviors)*
- **FR-017**: All user-visible History and log-sheet texts MUST exist in Spanish and English (device Spanish → Spanish; otherwise English). *(Source: scope §3 v1 language)*
- **FR-018**: Tapping an empty tatami cell MUST NOT open the log sheet. *(Source: implied by scope §6 Log sheet "tapping a tatami cell" for a log; edge case mandated for this feature)*
- **FR-019**: Closing the log sheet without saving MUST discard unsaved discipline selection. *(Source: mockups 6b close control; Log sheet rules—only Save updates discipline)*
- **FR-020**: Deleting a class MUST NOT show a character message (no delete reaction in the message bank). *(Source: message-bank review; Home messages are out of this feature)*
- **FR-021**: A day has at most one class per local clock hour; the rule is enforced when a class is registered (spec 002). Classes of a day MUST be ordered by registration timestamp ascending, then by log id ascending, because two devices can create two classes in one hour while offline. *(Source: stored-values Class registration limit; Clarifications 2026-11-27; spec 002)*

### Key Entities

- **Week day cell**: One day in the current Monday–Sunday week; states: current, past-with-training, past-without-training, future.
- **Day view**: Chosen day's label, tatami occupancy, Classes summary, Water summary.
- **Class log**: A single training entry with local date, time, discipline, and fixed board element for that date; at most one per local clock hour on a given day; editable discipline only; deletable from History when shown as a filled tatami cell (earliest twelve by local time; later ones overflow behind `+N`).
- **Water day summary**: Aggregate glasses for the chosen day, charted by the same half-open 3-hour blocks as Classes (including pre-06:00 in 6a); not editable from History.
- **Hour block**: One of six day chart buckets labeled 6a / 9a / 12p / 3p / 6p / 9p; assignment uses local wall time and the half-open windows in FR-007.
- **Pet status**: Derived state from remaining class attendance (and water rules elsewhere); refreshed after delete; stage reached is stored and never lowered by delete.
- **Log sheet session**: Transient UI over a single class log for discipline change or delete confirmation / success.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In a usability check with five signed-in pilot users who already have at least one class this week, at least four correctly identify today's cell and open a past training day on the first try. *(Assumed sample size; see Assumptions)*
- **SC-002**: From a filled tatami cell to confirmed delete, a user completes the flow in under 15 seconds when Reduce Motion is on (no animation wait). *(Assumed target; see Assumptions)*
- **SC-003**: After deleting a class that was the only one that day, 100% of test runs show zero Classes total, an empty minimum three-row tatami, and no week-bar checkmark for that past day.
- **SC-004**: After any delete that would mathematically lower stage by remaining class count, stage reached remains at the pre-delete value in 100% of test runs, while pet state matches the recalculated attendance rules.
- **SC-005**: Changing discipline leaves time, date, and element identical to pre-save values in 100% of test runs; Save is inactive whenever the selection equals the stored discipline.
- **SC-006**: With network disabled, browse, save discipline, and delete each complete with on-screen refresh within 2 seconds of the confirming action (excluding the optional ~2.5 s success animation). *(Assumed latency bound; see Assumptions)*
- **SC-007**: With Reduce Motion off, the class-deleted sequence is visible for approximately 2.5 seconds (±0.5 s) before the sheet closes; with Reduce Motion on, the sheet closes without that sequence in 100% of test runs.
- **SC-008**: On a day with more than twelve classes, every tester sees a `+N` indicator for the count beyond twelve, can open only the twelve earliest (by local time) filled cells, cannot open overflow classes via `+N`, and after deletes sees overflow classes promote into filled cells in local-time order until twelve or fewer remain.
- **SC-009**: Fixture days place at most one class per local clock hour, except the tie-break test.

## Assumptions

- Spec 002 leaves a navigable History shell that this feature replaces with the full week and day experience; Train (Home) logging and Undo remain owned by spec 002.
- Default chosen day when opening History is today.
- At most one class per local clock hour per day. Enforcement is defined in spec 002 and the stored-values document. History orders classes of a day by registration timestamp ascending, then by log id ascending when the offline multi-device exception applies.
- Discipline chips on the log sheet are the four fixed v1 disciplines; single selection only.
- The ~2.5 s class-deleted duration is the product timing; a longer looping clip in the mockups is for design review only, not the shipped wait.
- The copy of the figure must read correctly for today and for other days; the plan defines it.
- Scope §6 marks "tapping a past day" as proposed; mockups and the overall History decision treat it as in-scope—this spec follows the mockups and treats past-day selection as required.
- Invented measurable targets (SC-001 sample, SC-002 15 s, SC-006 2 s offline refresh, SC-007 ±0.5 s) are pilot heuristics, not prior product SLAs.
- No message-bank entry reacts to delete; History does not invent one.
- Server rejections of optimistic writes are not shown to the user in v1 (data architecture).
- If week data is unavailable, History shows the empty week shell with today selected and zero totals, without an error.
- Settings, account actions, previous weeks, month/year views, event lists, editing time/date, and adding classes for past days stay out of this feature.

## Out of Scope

- Previous weeks; month and year views.
- Event list for classes or water.
- Editing time or date of a log.
- Adding classes for past days.
- Settings and account actions (spec 004).
- Home logging and Undo (spec 002).
- Character speech-bubble messaging driven by History actions.

## Source contradictions

None. Scope, mockups, domain behavior notes, stored-values element rule, and the message bank were compared; no conflicting decisions required a `[NEEDS CLARIFICATION]` marker. The only soft gap (past-day tap marked proposed in scope §6 while mockups treat it as part of History) is resolved under Assumptions in favor of the mockups.
