# Feature Specification: Settings and Account Management

**Feature Branch**: `004-settings-account`

**Created**: 2026-11-27

**Status**: Draft

**Input**: User description: "Settings tab and account management: profile and training preferences, Change Password, Delete Account, Sign Out, external links and the optional minimum-version lock screen. Spec 002 leaves a placeholder Settings shell; this spec replaces it."

## Clarifications

### Session 2026-11-27

- Q: Al borrar la cuenta con éxito, ¿qué datos del usuario deben eliminarse del servicio además de la credencial de autenticación? → A: User document + all classLogs + all waterLogs + auth account
- Q: ¿De dónde deben salir las URLs de Support, Terms y Privacy Policy que abre Settings? → A: Local constants / embedded app configuration
- Q: ¿En qué momentos debe evaluarse el bloqueo por versión mínima (si está incluido en el build)? → A: Cold start and every time the app enters foreground
- Q: Cuando Delete Account o Change Password no pueden completar por red, ¿cuánto debe esperar la app antes de mostrar el mensaje de sin red? → A: Wait up to ~10 seconds for the server, then show no-network
- Q: Antes de completar Sign Out, ¿cuánto debe intentar la app terminar el sync pendiente? → A: Attempt sync up to ~3 seconds, then clear local and sign out anyway (data architecture)

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Sign Out (Priority: P1)

A signed-in user opens Settings, taps Sign Out, and returns to Start without any confirmation alert. Local data on the device is cleared. Progress that already reached the cloud remains available after a later Sign In. Scheduled reminders are cancelled.

**Why this priority**: Account exit must work for every signed-in user and is required for App Store account control and for switching devices cleanly. It is independently valuable even before profile editing ships.

**Independent Test**: From a signed-in session with local progress, sign out and confirm Start appears, no local session remains, reminders are gone, and a later Sign In restores cloud progress.

**Acceptance Scenarios**:

1. **Given** a signed-in user on Settings, **When** they tap Sign Out, **Then** no confirmation alert appears, the app briefly attempts pending sync (up to 3 seconds), the session ends, local data is cleared, reminders are cancelled, and the app shows Start.
2. **Given** a user who signed out after progress had synced, **When** they Sign In again on the same or another device, **Then** their cloud progress is available.
3. **Given** a signed-in user offline with unsynced class logs, **When** they Sign Out, **Then** the app still signs out without an alert, returns to Start, clears local data, and those unsynced logs may be lost (accepted risk).

---

### User Story 2 - Delete Account (Priority: P1)

A signed-in user chooses Delete Account in Settings. A confirmation sheet asks for the current password. On success, the same sheet shows an account-deleted success state with a short progress bar (~2.5 s), then the app returns to Start with no session and no local data. The service deletes the user document, all of that user's class logs, all of that user's water logs, and the authentication account. If Reduce Motion is on, the success state appears without animation.

**Why this priority**: In-app account deletion is mandatory for store compliance and irreversible; it must be correct before other Settings polish.

**Independent Test**: Delete an account end to end with the correct password; verify Start, empty session, deleted progress, and cancelled reminders. Separately verify wrong password and offline failure without deletion.

**Acceptance Scenarios**:

1. **Given** a signed-in user on Settings, **When** they open Delete Account, **Then** a confirmation sheet appears over darkened Settings with a current-password field (eye toggle), a destructive confirm button inactive while the field is empty, and Cancel that closes the sheet leaving Settings unchanged.
2. **Given** the confirmation sheet with a wrong password, **When** they confirm, **Then** nothing is deleted, the field shows a wrong-password error, and the sheet stays open.
3. **Given** the confirmation sheet with the correct password and network available, **When** they confirm, **Then** the app reauthenticates first, deletes the user document plus all class logs and water logs for that user, deletes the authentication account, clears local data, cancels reminders, shows the account-deleted success content in the same sheet (~2.5 s bar), then opens Start with no session.
4. **Given** Reduce Motion is enabled, **When** deletion succeeds, **Then** the success content appears without animation and the app still returns to Start.
5. **Given** the device is offline, **When** the user confirms Delete Account, **Then** the app waits up to ~10 seconds for the server, shows a no-network message, and does not delete the account or progress.

---

### User Story 3 - Change Password (Priority: P1)

A signed-in user opens Change Password, enters current, new, and repeated new passwords, and updates successfully. A success sheet with animation appears over Settings, then closes so Settings remains. There is no password-recovery link on this screen. Reduce Motion skips the animation.

**Why this priority**: Password update is a core account-security action available to every v1 email account and pairs with deletion for account lifecycle completeness.

**Independent Test**: Change password with valid inputs; verify success sheet then Settings, and that Sign In works with the new password. Separately verify validation and offline errors.

**Acceptance Scenarios**:

1. **Given** Change Password with any field empty, new password under 8 characters, or repeat that does not match, **When** the user views the screen, **Then** the update button stays inactive (mismatch may also show the repeat-field error when applicable).
2. **Given** all fields filled, new password 8+ characters, and matching repeat, **When** the current password is wrong, **Then** the current-password field shows the wrong-current error and the password is not changed.
3. **Given** valid filled fields with matching new/repeat and correct current password online, **When** the user updates, **Then** a success sheet appears over darkened Settings (~2.5 s bar), then closes and Settings remains; the new password works on the next Sign In.
4. **Given** Reduce Motion is enabled, **When** update succeeds, **Then** the success sheet appears without animation and still returns to Settings.
5. **Given** the device is offline, **When** the user attempts to update, **Then** the app waits up to ~10 seconds for the server and shows a no-network message without changing the password.
6. **Given** Change Password is open, **When** the user looks for recovery, **Then** no recovery link is present; recovery remains available only via Sign In after Sign Out.

---

### User Story 4 - Edit Profile and Training Preferences (Priority: P2)

A signed-in user edits nickname (max 10 characters, optional), selected disciplines (at least one), water goal (4 to 16 glasses), and optional reminder (days and time). Changes apply on Home without waiting for the server. Reminder permission is requested only when the reminder is turned on.

**Why this priority**: Personalization improves daily use but account exit and password security come first.

**Independent Test**: Change each preference while signed in and confirm Home reflects nickname, discipline selection, water goal, and reminder schedule; verify permission-denied reminder behavior.

**Acceptance Scenarios**:

1. **Given** Settings Character section, **When** the user sets a nickname of at most 10 characters (or clears it), **Then** the value is saved and Home shows or hides the nickname line accordingly; a 10-character nickname does not clip or break layout.
2. **Given** Training disciplines with at least one selected, **When** the user tries to deselect the last remaining discipline, **Then** at least one discipline remains selected.
3. **Given** the last used discipline is among the selected set, **When** the user removes that discipline from the selection, **Then** Home falls back to the first selected discipline for logging defaults.
4. **Given** water goal between 4 and 16, **When** the user changes the goal, **Then** Home uses the new goal for the bottle and water rules immediately.
5. **Given** today's glass count already meets or exceeds a newly lowered goal, **When** the goal is saved, **Then** Home recalculates today's water-related state with the new goal (glasses already logged stay; no error screen).
6. **Given** the reminder is off, **When** the user turns it on and picks days and time, **Then** the app requests notification permission only at that moment and schedules reminders if allowed.
7. **Given** notification permission is denied, **When** the user tries to enable the reminder, **Then** the reminder stays off, Settings shows a notice, and a way to open system Settings is available.
8. **Given** any of these preference edits while offline, **When** the user saves or toggles, **Then** changes apply immediately on Home (optimistic); server rejection is not shown to the user in v1.

---

### User Story 5 - External Links, Rate App, and Footer (Priority: P2)

From Settings, the user opens Support, Terms, and Privacy Policy as external pages, can rate the app via the system review dialog, and sees the app icon and version in the footer.

**Why this priority**: Legal and support surfaces are required for store readiness but do not block account lifecycle.

**Independent Test**: From Settings, open each external link destination and trigger Rate the app; confirm footer shows icon and version.

**Acceptance Scenarios**:

1. **Given** Settings More section, **When** the user taps Support, Terms, or Privacy Policy, **Then** the corresponding external page opens using URLs from local app constants (not Remote Config).
2. **Given** Settings More section, **When** the user taps Rate the app, **Then** the system review dialog is presented (subject to platform eligibility).
3. **Given** Settings, **When** the user views the footer, **Then** the app icon and installed version are visible along with Terms and Privacy Policy entry points.

---

### User Story 6 - Minimum-Version Lock Screen (Priority: P3)

When the installed app version is lower than the configured minimum version, a lock screen with an App Store button blocks the app. Evaluation runs on cold start and every time the app enters foreground. If the minimum version is missing or invalid, or the installed version is equal or higher, no lock is shown. This story is the first cut in the schedule if time runs short.

**Why this priority**: Useful for forced updates, but optional and cuttable; comparator behavior still exists with a safe default that never locks incorrectly.

**Independent Test**: With a minimum version higher than the installed build, confirm lock and App Store action; with equal/higher installed version or invalid/missing minimum, confirm no lock.

**Acceptance Scenarios**:

1. **Given** a valid minimum version higher than the installed version, **When** the app evaluates version on cold start or on entering foreground, **Then** a lock screen with an App Store button blocks use of the app.
2. **Given** installed version equal to or higher than the minimum, **When** the app evaluates version on cold start or on entering foreground, **Then** no lock screen is shown.
3. **Given** a missing or invalid minimum-version value, **When** the app evaluates version on cold start or on entering foreground, **Then** the value is treated as no minimum (equivalent to allowing all installs) and no lock screen is shown.

---

### Edge Cases

- **Delete account fails midway and is retried**: The user document, class logs, and water logs are deleted before the authentication account is removed. If the flow fails after data deletion but before full local cleanup, the user can retry; the design avoids leaving orphaned progress tied to a live account.
- **Wrong password on delete**: Nothing is deleted; the confirmation sheet stays open with the wrong-password field error (mockup 7e).
- **Session not recent**: The confirmation sheet requires the current password and reauthenticates before deletion so a recent session is established in the same step; no separate “session expired” screen is invented in v1.
- **Offline delete**: Waits up to ~10 seconds for the server; shows no-network; does not delete.
- **Offline change password**: Waits up to ~10 seconds for the server; shows no-network; does not change the password.
- **Sign out offline with unsynced classes**: Attempts pending sync up to 3 seconds, then proceeds without alert; unsynced classes may be lost (accepted risk). Reminders are still cancelled and local data cleared.
- **Reminder permission denied**: Reminder remains off; notice plus path to system Settings; no silent “on” state.
- **Removing the last used discipline**: Selection keeps at least one discipline; Home uses the first selected discipline when the previous last-used value is no longer selected.
- **Water goal lowered below today's glass count**: Glasses remain; Home recalculates against the new goal (goal may already be met); no error or rollback UI.
- **Nickname at 10 characters**: Accepted maximum; UI does not clip or break; empty nickname hides the nickname line.
- **Cancel on delete confirmation**: Closes the sheet; Settings unchanged.
- **Reduce Motion** on delete or password success: Success content without animation; navigation outcome unchanged.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Settings MUST be available only with a signed-in session. This feature replaces the Settings placeholder from the core-loop shell. *(Source: scope §3 Settings, §6 Settings, §11 item 2; mockups Flow / Settings)*
- **FR-002**: Settings MUST present sections Character (nickname), Training (disciplines, reminder, water goal), Account (email, Change Password, Delete Account), and More (Support, Rate the app, Sign Out), plus a footer with app icon, version, Terms, and Privacy Policy. *(Source: scope §6 Settings; mockups Settings (7))*
- **FR-003**: Sign Out MUST NOT show a confirmation alert. It MUST end the session, clear local data, cancel reminders, and return to Start. Cloud progress that already synced MUST remain. *(Source: scope §3 Sign Out, §11 items 5–6; data Sign out; domain SignOut)*
- **FR-004**: Sign Out while offline with unsynced classes MAY lose those classes. This is an accepted risk. No alert. *(Source: scope §3 / §7; data Sign out)*
- **FR-005**: Before finishing Sign Out, the app MUST attempt to finish pending sync for up to 3 seconds, then proceed with local cleanup even if sync cannot complete. *(Source: data Sign out — behavior only; Clarifications 2026-11-27)*
- **FR-006**: Delete Account MUST open confirmation sheet 7b (password field with eye, destructive confirm inactive when empty, Cancel leaves Settings unchanged). *(Source: scope §6 Delete Account 7b; mockups 7b / Delete Account rules)*
- **FR-007**: On Delete Account confirm, the app MUST reauthenticate with the current password first; only then delete the user document, all class logs and water logs for that user, then delete the authentication account; then clear local state like Sign Out and show success 7c. *(Source: scope §6 7b–7c; mockups Delete Account rules; data Delete account; domain DeleteAccount; Clarifications 2026-11-27)*
- **FR-008**: Wrong password on Delete Account MUST leave data intact and show error 7e on the field. *(Source: scope §6 7e; mockups 7e)*
- **FR-009**: Delete Account success (7c) MUST show account-deleted content with a lime progress bar that completes in about 2.5 seconds, then open Start with no session or local data. Reduce Motion MUST skip the animation. *(Source: scope §6 7c; mockups 7c — product bar duration from scope; mockup “4 s loop” describes the prototype only)*
- **FR-010**: Change Password MUST be a full screen (7d) with current, new, and repeat fields (each with eye), notice that new password needs 8+ characters, update button enabled only when all fields are filled, new password has 8+ characters, and repeat matches; no recovery link. *(Source: scope §6 7d; mockups 7d / Change Password rules; domain ChangePassword)*
- **FR-011**: Change Password MUST show error 7g for wrong current password and 7h for mismatched repeat. *(Source: scope §6 7g–7h; mockups Change Password rules)*
- **FR-012**: Change Password success MUST show sheet 7i over darkened Settings with the same success pattern as account deleted (~2.5 s bar), then close so Settings remains. Reduce Motion MUST skip the animation. *(Source: scope §6 7i; mockups 7i)*
- **FR-013**: Delete Account, Change Password, and Sign In MUST wait for the server (up to ~10 seconds for Delete Account and Change Password network failure); when they cannot complete they MUST show a no-network message and MUST NOT complete the sensitive action. Other Settings preference edits MUST apply immediately (optimistic). Server rejection of optimistic writes MUST NOT be shown to the user in v1. *(Source: scope §7; data Offline writes; Clarifications 2026-11-27)*
- **FR-014**: Nickname MUST be optional, maximum 10 characters, editable in Character; empty hides the nickname line; it is never translated; character messages MUST NOT use it. *(Source: scope §6 Nickname; domain UserProfile / UpdateProfile)*
- **FR-015**: Selected disciplines MUST keep at least one. If the last used discipline is removed from the selection, Home MUST fall back to the first selected discipline. *(Source: stored-values Last used discipline; domain UpdateProfile; scope §6 Training)*
- **FR-016**: Water goal MUST be editable from 4 to 16 glasses (default 8 from product rules). Changes MUST apply on Home. *(Source: scope §4 water goal, §6 Settings; domain UserProfile)*
- **FR-017**: Reminder (days and time) MUST request notification permission only when turned on. If denied, reminder stays off with a notice and a way to open system Settings. Reminders MUST be cancelled on Sign Out and Delete Account. On app open, reminder schedule MUST reconcile with the profile. *(Source: scope §3 reminder, §6 onboarding/Settings; data ReminderScheduler / Sign out / Delete account; domain ReminderScheduler)*
- **FR-018**: Support, Terms, and Privacy Policy MUST open as external pages whose URLs come from local constants / embedded app configuration (not Remote Config). Rate the app MUST use the system review dialog. Footer MUST show icon and version. *(Source: scope §3 Settings links, §6 Settings, §9 Store checklist; mockups Settings (7); Clarifications 2026-11-27)*
- **FR-019**: If installed version is lower than a valid minimum version, the app MUST show an optional lock screen with an App Store button that blocks the app. The app MUST evaluate this on cold start and every time it enters foreground. Missing or invalid minimum MUST mean no lock. Equal or higher installed version MUST never lock. This lock screen is the first scheduled cut. *(Source: scope §4 min_version, §6 version lock, §8 cut order; data Remote Config min_version; domain VersionComparator; presentation Version lock screen; Clarifications 2026-11-27)*
- **FR-020**: All Settings and account texts MUST be available in Spanish and English. Reduce Motion MUST be respected on success animations and related motion. *(Source: scope §6 / content; constitution Language / UI)*
- **FR-021**: Out of scope for this feature: change email, Sign in with Apple, password recovery from Settings, languages other than Spanish and English, History/Home behavior beyond applying preference side effects, store listing copy, review notes, and demo account. *(Source: scope §3 / §9 / §11; user feature bounds)*

### Key Entities

- **User profile (preferences)**: Nickname (optional, max 10), selected disciplines (one or more of the four training disciplines), last used discipline (optional; falls back to first selected), water goal (4–16), reminder (days Monday=1…Sunday=7 and time; empty when off), email (display only in Account).
- **Account session**: Signed-in email/password session required for Settings; ended by Sign Out or Delete Account.
- **Minimum version policy**: Remote minimum app version compared to installed version on cold start and every foreground entry; invalid/missing treated as allowing all installs.
- **Local device state**: Cached progress, preference cache, message-rotation flags, and scheduled reminders cleared on Sign Out and Delete Account.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of Sign Out attempts from Settings reach Start with no confirmation alert and no remaining local session on the device.
- **SC-002**: 100% of successful Delete Account flows (correct password, online) remove the user document, all class logs and water logs for that user, and the authentication account; cancel reminders; and land on Start with no local session within the success presentation window (~2.5 s, or immediately with Reduce Motion).
- **SC-003**: 100% of wrong-password Delete Account attempts leave progress and account intact and show the inline wrong-password error.
- **SC-004**: Users can complete Change Password with valid inputs online and return to Settings after the success presentation; the new password works on the next Sign In.
- **SC-005**: Offline Delete Account and Change Password show no-network within ~10 seconds and do not complete the action in 100% of airplane-mode trials.
- **SC-006**: After editing nickname, disciplines, water goal, or reminder online or offline, Home reflects the new preferences without a user-visible sync wait (target: perceived as immediate; invented target under Assumptions).
- **SC-007**: When notification permission is denied, 100% of reminder enable attempts leave the reminder off and show a notice with a path to system Settings.
- **SC-008**: With installed version ≥ minimum, or with missing/invalid minimum, lock screen appearance rate is 0% on cold start and on foreground entry. With installed version < valid minimum, lock screen appearance rate is 100% on those same evaluation moments when the optional lock is included in the build.
- **SC-009**: On a device set to English and one set to Spanish, Settings and account flows show the matching language with no mixed untranslated keys in spot checks of all FR-002 sections.
- **SC-010**: A 10-character nickname saves and displays without clipping on Settings and Home in both languages.

## Assumptions

- Spec 002's Settings tab shell remains the navigation entry; this feature replaces its placeholder content only.
- Product success bar duration for 7c/7i follows scope (~2.5 s). Mockups' “4 s looping animation” refers to the design prototype loop, not the shipping duration.
- “Immediate” preference updates means the UI updates without waiting for server acknowledgement (optimistic), typically within one interaction frame from the user's perspective (invented perception target for SC-006).
- System review dialog eligibility is controlled by the platform; Rate the app may no-op when the platform suppresses the dialog—still counts as correct behavior.
- App Store destination for the version lock uses the app's store listing identity already registered for Brenz.
- Default water goal remains 8 until the user changes it.
- No new error screens beyond 7e, 7g, 7h, no-network for waiting actions, and reminder permission notice.
- Server rejection of optimistic preference or training writes is logged only and not shown (v1).
- Cut order: if schedule pressure requires it, the minimum-version lock screen UI is cut first while still treating invalid/missing minimum as non-locking.
- Support, Terms, and Privacy Policy destination URLs are shipped as local constants in the app; changing them requires an app update.
- Delete Account and Change Password treat network failure after waiting up to ~10 seconds as no-network; no separate retry UI in v1 beyond the user confirming again.
- Sign Out's “brief” pending-sync attempt is capped at up to 3 seconds; cleanup and return to Start always continue afterward.

## Source Traceability

| Area | Primary sources |
|------|-----------------|
| Settings structure, links, Sign Out | scope §3, §6; mockups Settings (7) |
| Delete Account 7b/7c/7e | scope §6, §9; mockups Delete Account rules |
| Change Password 7d/7g/7h/7i | scope §6; mockups Change Password rules |
| Offline waiting vs optimistic | scope §7; data Offline writes |
| Sign out / delete order, reminders | data Sign out, Delete account, Local reminder |
| Profile, password, delete, sign out, version compare | domain UpdateProfile, ChangePassword, DeleteAccount, SignOut, VersionComparator |
| Last used / selected disciplines | stored-values Last used discipline |
| min_version lock | scope §4, §6, §8; data Remote Config; presentation Version lock |
| Nickname / water goal bounds | scope §4, §6 Nickname |

## Out of Scope

- Change email; Sign in with Apple; password recovery from Settings (recovery stays on Sign In after Sign Out).
- Languages other than Spanish and English.
- History and Home feature work belonging to specs 002 and 003, except preference side effects described here.
- Store listing, review notes, demo account.
- Implementation details: service vendor names, UI framework, file paths, and formulas.
