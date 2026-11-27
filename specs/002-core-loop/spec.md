# Feature Specification: Core Loop

**Feature Branch**: `002-core-loop`

**Created**: 2026-11-27

**Status**: Draft

**Input**: User description: "core-loop — daily training loop in Home: pet, class logging, water, tatami board, and character messages. Spec 001 delivers the user to Home."

**Sources** (behavior only; cite by section; do not treat paths as implementation):
- Product scope §3 (class, water, pet, stage, messages), §4 (rules), §5 (content rules), §6 (Home, Interactions, Board, Nickname, Board elements)
- Product mockups: Home screens and discipline sheet
- Message bank: all groups, triggers, and rotation
- Domain architecture: rules, messages, catalogs (behavior only)
- Stored values: board element rotation (behavior only; calculation detail deferred to plan)
- Art assets: pet states and stages

## Clarifications

### Session 2026-11-27

- Q: When Home opens and both the new-user message and the no-glasses-at-midday message are eligible, which one should the speech bubble show? → A: Priority: new-user → midday → state message
- Q: If the remembered last-used discipline is no longer among the user’s selected disciplines, which discipline should Home use for the chip and the next class log? → A: Fall back to the first selected discipline
- Q: When today’s water glasses exceed the user’s goal, how should the bottle on Home look? → A: Cap visual fill at full; fraction text may exceed goal (e.g. 10/8)
- Q: When the user taps History or Settings in the Home tab bar during this feature, what should happen? → A: Tabs navigate to placeholder shells (no History/Settings content yet)

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Log a class with one tap (Priority: P1)

A signed-in user on Home taps the main training button once. The character reacts, today’s board gains one cell with the day’s element, Undo is offered briefly, and multiple taps in the same day each add another class.

**Why this priority**: Class logging is the core habit action; without it there is no board progress, stage progress, or meaningful pet state.

**Independent Test**: On Home with at least one selected discipline, tap the main class button once and again; verify bounce, board cell, Undo window, and that each tap counts as a separate class for today.

**Acceptance Scenarios**:

1. **Given** the user is on Home with a remembered (or default) discipline, **When** they tap the main class button ("Fui a clase" / equivalent English), **Then** one class is registered for today with date and time, the character bounces (or fades under Reduce Motion), and the day’s element appears on a tatami cell.
2. **Given** a class was just registered, **When** fewer than 5 seconds have passed, **Then** Undo is available and tapping Undo removes that last class and updates the board and pet state accordingly.
3. **Given** a class was registered, **When** 5 seconds or more have passed, **Then** Undo is no longer available for that class from Home.
4. **Given** the user already logged one or more classes today, **When** they tap the main class button again, **Then** another class is added (several classes per day count) and another cell is filled.
5. **Given** the user has no network, **When** they tap the main class button, **Then** the class is registered immediately without waiting for the network and the board updates at once.
6. **Given** a class is registered for a local calendar day, **When** that day is viewed later or on another device for the same account, **Then** the same board element is associated with that class (element depends on the class’s local date and weekly rotation; changing the class’s discipline later does not change its element).

---

### User Story 2 - Pet state and stage follow the rules (Priority: P1)

The character’s state is always derived from recent classes and today’s water. Stage rises with accumulated classes and never goes down. A new user starts at normal. Meeting the water goal can raise normal to animated for that day only.

**Why this priority**: The pet is the emotional feedback loop of the product; state and stage must match the closed rules or the Home experience is wrong.

**Independent Test**: With controlled class and water histories, open Home and verify each state boundary, stage threshold, water-only animated day, and that stage does not decrease when classes are removed via Undo within the window.

**Acceptance Scenarios**:

1. **Given** no classes ever logged, **When** Home is shown, **Then** the pet state is normal (new user) and stage is stage 1 with the visible stage name.
2. **Given** the last class was 3 days ago or less and there are fewer than 2 classes in the last 7 days, **When** state is calculated, **Then** the pet is normal.
3. **Given** the last class was 3 days ago or less and there are 2 or more classes in the last 7 days, **When** state is calculated, **Then** the pet is animated.
4. **Given** 4 to 6 days without a class, **When** state is calculated, **Then** the pet is down.
5. **Given** 7 or more days without a class, **When** state is calculated, **Then** the pet is lazy.
6. **Given** base state normal and today’s water glasses meet the user’s water goal, **When** state is calculated for today, **Then** the pet is animated for that day only; water never lowers state by itself.
7. **Given** the pet was animated only because of water and the user removes a glass so today’s count is below the goal, **When** state is recalculated, **Then** the pet returns to normal for today.
8. **Given** the user reaches 10 accumulated classes, **When** that class is registered, **Then** stage becomes stage 2, the stage line updates, and stage never decreases afterward even if a class is undone or later deleted elsewhere.
9. **Given** the pet is lazy, **When** the user logs the next class, **Then** state returns toward normal/animated per the class rules (no separate reset control).
10. **Given** a nickname is set, **When** Home shows the character, **Then** nickname and stage line appear under the character (stage 1 style: stage name · class N of 10; stage 2 style: stage name · N classes). **Given** nickname is empty, **When** Home shows the character, **Then** the nickname line is omitted.
11. **Given** Reduce Motion is on, **When** the user logs a class or changes state, **Then** bounce/spin-style motion is replaced by a simple fade or no motion, and the correct state and stage art still appear.

---

### User Story 3 - Add and remove water glasses on the board card (Priority: P1)

On Home’s board card the user adds and removes glasses with + and −. The bottle fills and drains. There is no separate water button. Each glass is an event with a time. Only today’s glasses are managed here; the counter resets at local midnight.

**Why this priority**: Water is the second daily habit on Home and can elevate the pet to animated for the day.

**Independent Test**: From Home, press + until the goal, then − below the goal; verify bottle level text, pet reaction, and offline immediacy.

**Acceptance Scenarios**:

1. **Given** the user is on Home, **When** they tap +, **Then** one glass is added for today with a timestamp, the bottle rises one step with a bounce (or fade under Reduce Motion), the character does a short bounce, and the fraction text updates (e.g. glasses/goal).
2. **Given** today’s glasses are at or above the goal and base state is normal, **When** the goal is reached, **Then** the pet becomes animated for that day.
3. **Given** today’s glasses are above zero, **When** they tap −, **Then** one glass is removed, the bottle drains animated (or fades under Reduce Motion), and the fraction text updates.
4. **Given** today’s glasses are zero, **When** they tap −, **Then** no glass is removed and the count stays at zero.
5. **Given** no network, **When** the user adds or removes a glass, **Then** the change applies immediately without waiting for the network.
6. **Given** local midnight passes (or the local calendar day changes), **When** Home shows today’s water, **Then** today’s count starts at zero; previous days’ glasses are not editable from Home.
7. **Given** any number of glasses already logged today, **When** the user taps +, **Then** another glass is added (no daily glass or time limit on Home).
8. **Given** today’s glasses are above the water goal, **When** Home shows the bottle, **Then** the bottle fill stays at visually full and the fraction text shows the true count over goal (e.g. 10/8).

---

### User Story 4 - Character messages in the speech bubble (Priority: P2)

The speech bubble shows state, class-event, water, and ambient messages according to the message bank rules. The user can close the bubble with X. Water messages are temporary (~4 s) and can reopen a closed bubble.

**Why this priority**: Messages give sarcastic, supportive feedback without blocking the core log/water loop; they depend on state and actions already working.

**Independent Test**: Exercise first Home with no classes, a class after lazy, stage change at 10 classes, water taps at half/goal/remove-below-goal, midday with zero glasses, and bubble close/reopen.

**Acceptance Scenarios**:

1. **Given** a pet state for the local day, **When** a state message is chosen, **Then** one message from that state’s bank is shown for the whole day for that state, without repeating until that state’s bank is exhausted across days, and messages never include the nickname.
2. **Given** normal state and no class logged today, **When** a normal state message is chosen, **Then** the two “today reminder” (T) messages may appear; **Given** a class already logged today, **When** a normal state message is chosen, **Then** (T) messages are not used.
3. **Given** the previous state was lazy, **When** the user logs the first class that leaves lazy, **Then** the return-after-lazy event message is shown once.
4. **Given** the user reaches the stage-change class count, **When** that class is registered, **Then** a stage-change message appears in the bubble (no modal), with the visible stage name supplied as an argument.
5. **Given** the user taps + or − for water, **When** a water message is selected, **Then** priority on add is goal reached, then half of goal (rounded up), then add; on remove is dropped-below-goal (only if state actually fell from water-animated to normal) else one glass less; the message shows ~4 seconds then the state message returns.
6. **Given** animated only because of water (base normal), **When** the goal is reached, **Then** the bubble shows a goal-reached water message, not the animated-by-classes bank.
7. **Given** no classes ever and new-user messages not yet shown, **When** Home appears, **Then** a new-user message shows once (ahead of midday and state); it does not show again after it has been shown or after classes exist.
8. **Given** local time is after 12:00, today’s glasses are zero, the midday message was not shown today, and new-user is not eligible (already shown or classes exist), **When** Home appears, **Then** the no-glasses-at-midday message shows once that day ahead of the state message (no notification).
9. **Given** new-user and midday are both eligible on the same Home open, **When** the bubble message is chosen, **Then** only new-user is shown for that open; midday may show on a later Home appear the same day once new-user is no longer eligible and midday’s once-per-day flag is still unset.
10. **Given** the bubble is visible, **When** the user taps X, **Then** the bubble hides until pet state or the local day changes; **Given** the bubble was closed, **When** a water message fires, **Then** the bubble opens anyway for that temporary message.
11. **Given** the app language is Spanish or English, **When** any message is shown, **Then** the copy matches that language and follows content rules (sarcasm at actions, no guilt/punishment/mockery, no work/office/clothing, pet does not die).

---

### User Story 5 - Change discipline from Home (Priority: P2)

The user changes the active discipline from the Home chip and a single-selection translucent sheet. The last used discipline is remembered for the next class. The chip is hidden when only one discipline is selected.

**Why this priority**: Multi-discipline users need the correct label on each log without leaving Home; single-discipline users should not see noise.

**Independent Test**: With two or more selected disciplines, open the sheet, pick another, log a class, and reopen Home later; with one discipline, verify the chip is hidden.

**Acceptance Scenarios**:

1. **Given** two or more selected disciplines, **When** Home is shown, **Then** a discipline chip shows the current (last used if still selected, otherwise first selected; or first selected if none remembered) discipline.
2. **Given** the chip is visible, **When** the user taps it, **Then** a floating translucent single-selection sheet lists the user’s selected disciplines.
3. **Given** the sheet is open, **When** the user selects another discipline, **Then** that discipline becomes current for the next class log and is remembered as last used.
4. **Given** exactly one selected discipline, **When** Home is shown, **Then** the discipline chip is hidden.
5. **Given** a remembered last-used discipline that is no longer in the user’s selected disciplines, **When** Home resolves the current discipline, **Then** it uses the first selected discipline for the chip (if shown) and for the next class log.
6. **Given** a previously logged class, **When** the user later changes only that class’s discipline (out of scope here, History), **Then** that class’s board element remains the one fixed at registration time (behavior constraint for this feature’s stored element).

---

### User Story 6 - Tatami grows with today’s classes (Priority: P3)

Today’s tatami on Home starts with one row, grows as classes fill cells up to a cap of 12 cells (4 rows of 3), and shows "+N" when today’s classes exceed the cap. Empty days show empty outlined cells. Layout size is derived from today’s class count, not stored.

**Why this priority**: Visual growth reinforces logging; the cap and overflow keep the Home card usable.

**Independent Test**: Log from 0 through 13+ classes in a day (or simulate counts) and verify rows, filled cells, and "+N".

**Acceptance Scenarios**:

1. **Given** zero classes today, **When** Home’s board is shown, **Then** empty outlined cells appear for the Home starting layout (one row).
2. **Given** classes are logged today and a free cell exists, **When** a class is added, **Then** that cell is marked and the day’s element stands on it.
3. **Given** all current cells are filled, **When** another class is logged and the cap is not reached, **Then** a new row is added (3 columns) and the new class fills a cell.
4. **Given** 12 cells are filled, **When** a 13th (or further) class is logged today, **Then** the board keeps the cap and shows "+N" for the overflow count.
5. **Given** classes are undone within the Undo window so fewer cells are needed, **When** the board refreshes, **Then** rows shrink back to what today’s remaining classes require (layout not stored).

---

### User Story 7 - Rule values from remote configuration with offline defaults (Priority: P3)

Thresholds that drive state, stage, and the default water goal come from remote configuration, with built-in defaults that work with no network. Invalid remote values fall back to defaults. Editing the personal water goal in Settings is out of scope; Home uses the profile’s goal when set, otherwise the default.

**Why this priority**: Tunable rules without a store release; offline gym use must still behave correctly.

**Independent Test**: With network blocked, verify default thresholds; with remote values overridden (test harness), verify state/stage/water goal behavior changes; with invalid remote values, verify defaults apply.

**Acceptance Scenarios**:

1. **Given** no network and no cached remote overrides, **When** Home calculates state and stage, **Then** defaults apply: animated threshold 2 classes, window 7 days, down at 4 days, lazy at 7 days, stage change at 10 classes, default water goal 8 glasses.
2. **Given** valid remote overrides for those values, **When** Home recalculates, **Then** the overridden values are used.
3. **Given** invalid remote values, **When** rules are applied, **Then** each invalid value is replaced by its default.
4. **Given** the user’s profile has a water goal between 4 and 16, **When** water UI and messages run, **Then** that goal is used for fraction text, half, and goal-reached (Settings editing of the goal is out of scope for this feature).

---

### Edge Cases

- **Midnight rollover**: At local midnight, “today” advances; water count for the new day starts at 0; yesterday’s classes remain dated to the previous local day; state/stage messages re-evaluate for the new day; a closed bubble may reopen if state or day changed.
- **Device time zone change**: Day boundaries and “days without class” use the device’s current local calendar; after a time zone change, “today,” midday (after 12:00 local), and water reset follow the new local calendar without requiring network.
- **Class logged offline**: Registration, board update, Undo window, and pet recalculation happen immediately; sync happens later when connectivity returns without blocking the tap.
- **Undo after 5 seconds**: Undo control is gone; the class remains; Home does not offer delete/edit of that class (History is out of scope).
- **Water removed below the goal**: If animated was only from water, state returns to normal and the dropped-below-goal message may show; if animated came from classes, state stays animated and a ordinary remove message is used.
- **13th class in a day**: Twelfth cell fills the cap; further classes show "+N" and still count for state and stage.
- **Glasses above water goal**: Bottle fill stays visually full; fraction text shows true count over goal; + remains allowed.
- **State change after a class is registered**: Pet art, stage line, and eligible messages update immediately after the log (including return-after-lazy and stage-change events when they apply).
- **First launch with no classes**: Normal state, empty tatami (outlined cells), new-user message eligible once, water at 0/goal, main button ready; no weekly goal pressure.
- **Undo last class that caused stage up**: Stage reached does not go down; state recalculates from remaining classes.
- **Only one selected discipline**: Discipline chip hidden; logging still uses that discipline.
- **Last-used discipline no longer selected**: Resolve to the first selected discipline for chip and next class log.
- **Bubble closed then state changes**: Bubble may show again for the new state/day rules; water taps always can reopen it for ~4 s.
- **New-user and midday both eligible**: Show new-user only for that Home open; midday may appear on a later open the same day after new-user is no longer eligible.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Home MUST show, top to bottom: discipline chip (when more than one discipline is selected), character speech bubble with close control, large character with optional nickname and stage line, today’s board card (tatami + bottle with +/−), main class button, and the floating tab destinations Train / History / Settings. Tapping History or Settings MUST navigate to placeholder shells for those destinations; full History and Settings content remain out of scope. Sources: scope §6 Home; mockups Home.
- **FR-002**: The main class button MUST register exactly one class for the local today per tap, storing discipline, local date/time, and a board element fixed at registration. Sources: scope §3, §6 Interactions; domain RegisterClass behavior; stored values Board element rotation.
- **FR-003**: After each successful class registration from Home, the character MUST bounce (respecting Reduce Motion), the day’s element MUST appear on a tatami cell, and Undo MUST be available for 5 seconds for that last registration only. Sources: scope §3, §6 Interactions; domain UndoClass behavior.
- **FR-004**: Several classes on the same local day MUST each count for board, state, and stage. Sources: scope §3.
- **FR-005**: Classes for previous days MUST NOT be addable from Home. Sources: scope §3.
- **FR-006**: Pet state MUST be recalculated (never stored as source of truth) using: lazy (≥7 days without class), down (4–6 days without class), animated (last class ≤3 days ago and ≥2 classes in last 7 days), normal (last class ≤3 days ago and <2 classes in last 7 days); new user with no classes starts normal. Sources: scope §4; domain PetStateCalculator behavior.
- **FR-007**: Meeting today’s water goal MUST raise normal to animated for that day only; water MUST NEVER lower state except by undoing that water-based raise when glasses fall below the goal; each glass alone MAY cause a short bounce without changing state. Sources: scope §4.
- **FR-008**: Stage MUST rise with accumulated classes (stage 1→2 at 10 classes by default), MUST never go down, and MUST keep the highest stage reached even if classes are undone. Visible names: stage 1 and stage 2 as defined by the message bank / scope (stage 3 out of v1). Sources: scope §4; message bank Stages; domain StageCalculator behavior.
- **FR-009**: Character appearance MUST reflect current stage and state (four states × two stages in v1). Sources: assets pet states and stages; domain catalogs.
- **FR-010**: Home water controls MUST be + and − inside the board card only (no separate water action); each glass MUST be an individual timed event; only today’s glasses are added or removed; water resets at local midnight; no glass count or time-of-day limit. Sources: scope §3, §6 Interactions.
- **FR-011**: The bottle MUST always be visible on Home, fill when adding, drain when removing, and show today’s count over goal; when today’s glasses exceed the goal, visual fill MUST stay at full while the fraction text MAY exceed the goal (e.g. 10/8). Sources: scope §3, §6 Board.
- **FR-012**: Day for classes, water, messages, and “today” MUST be the device’s local calendar day. Sources: scope §3, §4; message bank Logic rules.
- **FR-013**: Board element for a class MUST depend on that class’s local date, rotate on a weekly cycle so the same local date yields the same element on every device, and MUST NOT change when the class’s discipline changes. Sources: stored values Board element rotation (wins over scope §6 fixed weekday mapping and assets day table); scope §6 Board elements (discipline change does not change element).
- **FR-014**: Tatami on Home MUST use a 3-column grid, start at 1 row, grow by rows as needed, cap at 12 cells, show "+N" beyond the cap, show empty outlined cells when empty, and derive layout from today’s class count (not stored). Sources: scope §6 Board; domain BoardLayout behavior.
- **FR-015**: Speech bubble MUST support state messages (once per state and local day, rotation without repeat until bank exhausted; (T) normal messages only with no class today), class events (return after lazy; stage change), water messages (~4 s then return to state message; add priority goal → half → add; remove priority dropped-below-goal → remove), and ambient messages (new user once; no glasses at midday after 12:00 local once per day). On Home open, when more than one non-water message is eligible, priority MUST be new-user → midday → state (class-event messages still take precedence when their trigger just fired). Sources: message bank Logic rules and groups; scope §6 Interactions; domain Messages behavior.
- **FR-016**: Closing the bubble with X MUST hide it until pet state or local day changes, except water messages which MUST reopen it. Sources: scope §6 Interactions [PROPOSED] Bubble X; message bank Logic rules.
- **FR-017**: Messages MUST be first person, MUST NOT use the nickname, MUST exist in Spanish and English, and MUST follow content rules (sarcasm at actions; no guilt/punishment/mockery; no work/office/clothing; pet does not die). Sources: scope §5; message bank Voice rules.
- **FR-018**: Nickname (optional, max 10 characters, not translated) and stage line MUST show under the character when a nickname exists; omit the nickname line when empty. Sources: scope §6 Nickname, Home.
- **FR-019**: Discipline chip and sheet MUST remember last used discipline for logging; if none is remembered, or the remembered value is not among the user’s selected disciplines, Home MUST use the first selected discipline; chip hidden when only one discipline is selected; sheet is floating, translucent, single selection. Sources: scope §6 Home, Sheet; stored values Last used discipline; mockups Discipline sheet.
- **FR-020**: Class logging and water add/remove MUST work offline and MUST NOT wait for the network to update Home. Sources: scope §3.
- **FR-021**: Rule numeric values (animated threshold, window days, down days, lazy days, classes for stage change, default water goal) MUST be remotely configurable with the defaults in FR-007/FR-008/User Story 7, MUST work offline with defaults, and invalid remote values MUST fall back to defaults. Sources: scope §4 Configurable values; domain RulesConfig behavior.
- **FR-022**: All Home motion for bounce, bottle, element appearance, tatami growth, and state crossfade MUST respect Reduce Motion (fade or no motion). Sources: scope §6 animation note / art direction; presentation Reduce Motion behavior; mockups Reduce Motion notes.
- **FR-023**: Visible Home and message texts MUST ship in Spanish and English. Sources: scope §2 / §3 languages; message bank.

### Key Entities

- **Class log**: One training registration; local date/time; discipline; fixed board element identity for that log; counts toward today’s board, state, and stage.
- **Water glass**: One timed drink event for a local day; today’s set drives bottle UI and water-based animated state.
- **Pet status**: Derived state (animated, normal, down, lazy) plus stage reached (1 or 2 in v1) and optional nickname for display.
- **Today’s board**: Tatami layout for today’s class logs plus bottle for today’s glasses; layout computed from counts.
- **Board element**: One of seven training objects assigned to a class from its local date via weekly rotation; shared across disciplines in v1.
- **Message selection**: Chosen message group and index for bubble display (state, event, water, ambient) with rotation and once-per-day/once-ever flags.
- **Rules configuration**: Tunable thresholds and default water goal, with offline defaults.
- **Last discipline**: Remembered discipline for the next Home class log among the user’s selected disciplines; if missing or not selected, resolve to the first selected discipline.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A user can register a class from Home in under 2 seconds from tap to visible board update and Undo availability (including offline).
- **SC-002**: A user can add or remove a water glass from Home in under 2 seconds from tap to updated bottle fraction (including offline).
- **SC-003**: In scripted reviews of the four pet states and both stages, observers correctly identify the intended state and stage in at least 95% of cases from Home alone.
- **SC-004**: For the same account and class local dates, two devices show the same board element on each corresponding class cell in 100% of sampled days across at least two consecutive weekly cycles.
- **SC-005**: Water goal, half-goal, and dropped-below-goal message triggers fire on the correct glass for goals 4, 8, and 16 in 100% of manual test runs.
- **SC-006**: With Reduce Motion enabled, no bounce or spin-style motion runs on class log, water change, or state change; feedback remains understandable via fade or static update in 100% of checked interactions.
- **SC-007**: After local midnight, today’s water shows 0 and yesterday’s classes do not appear as “today” on Home in 100% of overnight test runs.
- **SC-008**: Pilot users can complete the daily loop (optional water + at least one class) without leaving Home, with task success on first attempt for at least 90% of moderated pilot sessions.

## Assumptions

- Spec 001 has already delivered the user to Home with a completed onboarding profile (selected disciplines, optional nickname, water goal in range or default).
- Personal water goal editing lives in Settings (out of scope); this feature reads the profile goal when present and otherwise uses the default of 8.
- Invented timing targets: SC-001 and SC-002 use a 2-second perceived update budget; sources require immediacy/offline optimism but do not define a numeric latency KPI.
- Invented quality targets: SC-003 95%, SC-008 90%; sources do not define pilot recognition or task-success percentages.
- Bubble X behavior marked [PROPOSED] in scope §6 is treated as decided for this feature, consistent with the message bank (water may reopen a closed bubble).
- English copies of all 44 messages ship with this feature even though the message bank notes an English tone review pending; keys and logic are identical across languages.
- Minimum-version lock screen and remote `min_version` enforcement are out of scope (separate feature); rule remote config here covers training thresholds and default water goal only.
- History, edit/delete of past logs, reminders configuration, account actions, and Settings screens are out of scope except as tab destinations on Home that navigate to placeholder shells (no real History/Settings content in this feature).
- **Source conflict resolved (no clarification needed)**: Scope §6 Board elements / assets day table describe a fixed weekday→element map (e.g. Monday = glove). Stored values Board element rotation defines weekly rotation from a reference Monday so that fixed map holds only in week 0 of the cycle. Document precedence (constitution / stored values over scope) → this spec adopts weekly rotation and same-result-on-every-device behavior (FR-013). The element formula is deferred to the plan.
- **Source conflict resolved**: Message bank wins over scope where message/stage naming differs; stage visible names and message triggers follow the message bank.

## Out of Scope

- History screen content, log sheet, editing or deleting classes outside the 5-second Undo (spec 003). Placeholder History shell reachable from the tab bar is in scope for this feature.
- Settings content, reminder configuration, water goal editing UI, account actions, minimum-version lock screen (spec 004). Placeholder Settings shell reachable from the tab bar is in scope for this feature.
- Account, splash, intro, and onboarding (spec 001).
- Stage 3, store of elements, board themes beyond tatami, elements per discipline, weekly training goal, guest mode.
- Drinking-time statistics UI (events are stored; charts are History).

## Traceability Summary

| Area | Primary sources |
|------|-----------------|
| One-tap class, Undo 5 s, several/day | scope §3, §6 Interactions |
| Pet states, water→animated, stage | scope §4; domain rules |
| Water +/−, events, local day reset | scope §3, §6 |
| Home layout, chip, bubble, board card | scope §6 Home; mockups Home |
| Tatami growth, cap 12, "+N" | scope §6 Board; domain BoardLayout |
| Element weekly rotation, stable per log | stored values Board element rotation (over scope §6 / assets day table) |
| Messages, rotation, triggers | message bank; domain Messages; scope §5–§6 |
| Nickname + stage line | scope §6 Nickname, Home |
| Last discipline | stored values Last used discipline; scope §6 |
| Remote rule defaults offline | scope §4; domain RulesConfig |
| Pet art by stage×state | assets; domain catalogs |
| Reduce Motion | scope animation rules; presentation; mockups |
| Offline optimistic class/water | scope §3 |
| Languages es/en | scope; message bank |
