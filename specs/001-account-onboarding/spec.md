# Feature Specification: Account Onboarding

**Feature Branch**: `001-account-onboarding`

**Created**: 2026-11-27

**Status**: Draft

**Input**: User description: "account-onboarding — first-launch and session behavior: splash, start, intro, account creation and sign in, password reset, saved session, onboarding"

**Sources** (behavior only; cite by section; do not treat paths as implementation):
- Product scope §3 (account items), §6 (first launch, account routes, screens), §11 (flows closed)
- Product mockups: Splash, Start, Intro, Sign Up, Sign In, Reset Password, Onboarding
- Presentation architecture: root screen and navigation
- Data architecture: authentication

## Clarifications

### Session 2026-11-27

- Q: If a saved session exists but the profile cannot be loaded (no network or server error), what must the app show after splash? → A: Keep the session and show a retryable error state; do not open Home or Onboarding until the profile loads.
- Q: When requesting a password reset for a valid-format email that has no registered account, what must the user see? → A: Always show the same generic “link sent” confirmation (do not reveal whether the email is registered).
- Q: If session (and profile when applicable) takes longer than the splash animation (~1.5 s full or ~0.8 s short), what must the user see after that animation would normally end? → A: Keep showing the splash (logo) until session/profile resolves or fails; do not navigate to Start/Home/Onboarding early.
- Q: Must the user explicitly accept Terms and Privacy (checkbox or similar) before creating an account, or are links on Start/Sign Up enough? → A: Links only; no acceptance control required before Sign Up.
- Q: After showing the generic “link sent” confirmation on Reset Password, where must the user remain? → A: Stay on Reset Password with confirmation visible and a clear path back to Sign In.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Create account and complete onboarding (Priority: P1)

A person opens Brenz for the first time, chooses to start, watches the intro, creates an account with email and password, completes the three onboarding steps (optional nickname, disciplines, optional reminder), and reaches Home.

**Why this priority**: Without account creation and onboarding there is no usable product; the account is mandatory and there is no guest mode.

**Independent Test**: On a device with no saved session and first-launch flag unset, complete Start → Intro → Sign Up → Onboarding and verify Home opens with onboarding marked complete.

**Acceptance Scenarios**:

1. **Given** the app has never launched on this device and there is no saved session, **When** the user opens the app, **Then** the full animated splash appears (logo, brand name, slogan, copyright footer) and then Start.
2. **Given** the user is on Start, **When** they tap Start ("Empezar"), **Then** Intro screen 1 of 3 opens with a segmented progress bar and no skip control.
3. **Given** an Intro screen animation is playing, **When** the looping animation finishes its cycle, **Then** Continuar appears (last screen: Empezar) and the user can advance; they cannot skip earlier.
4. **Given** the user finished the last Intro screen, **When** they continue, **Then** Sign Up opens with email and password fields, legal links, and no close/dismiss control.
5. **Given** valid unused email and a password of at least 8 characters, **When** the user submits Sign Up, **Then** the account is created, a session is established, and Onboarding step 1 opens.
6. **Given** Onboarding step 1, **When** the user leaves the nickname empty or enters at most 10 characters and continues, **Then** Onboarding step 2 (disciplines) opens with progress dots reflecting step 2.
7. **Given** Onboarding step 2 with one discipline pre-selected by default and multi-select allowed, **When** the user keeps or changes the selection (at least one) and continues, **Then** Onboarding step 3 (reminder) opens.
8. **Given** Onboarding step 3 with reminder off, **When** the user finishes, **Then** onboarding is complete, Home opens, and no notification permission prompt is shown.
9. **Given** Onboarding step 3, **When** the user turns the reminder on (days and time) and finishes, **Then** notification permission is requested, onboarding completes, and Home opens.

---

### User Story 2 - Returning user with saved session (Priority: P1)

A person who already signed in and finished onboarding opens the app again. They see a short splash and land on Home without repeating Start, Intro, Sign Up, or Onboarding.

**Why this priority**: Session persistence is the everyday path after first launch; every later open must feel instant and skip the welcome flow.

**Independent Test**: With a saved session and onboarding completed, cold-start the app and verify short splash then Home only.

**Acceptance Scenarios**:

1. **Given** a saved session and onboarding completed, **When** the user opens the app, **Then** only the logo splash shows for about 0.8 seconds (no slogan or footer) and then Home.
2. **Given** a saved session and onboarding completed, **When** the user opens the app, **Then** Start, Intro, Sign Up, and Onboarding do not appear.
3. **Given** the user reinstalled the app on the same device and the previous session still persists on the device, **When** they open the app, **Then** short splash leads to Home (or Onboarding if onboarding was never completed) and prior progress is restored from the cloud account.

---

### User Story 3 - Sign in and resume incomplete onboarding (Priority: P2)

An existing account holder chooses "I already have an account", signs in with email and password, and either reaches Home or resumes Onboarding if they abandoned it earlier.

**Why this priority**: Existing users must enter without creating a second account; abandoned onboarding must not strand them.

**Independent Test**: From Start, use Sign In with a known account; repeat with an account whose onboarding is incomplete.

**Acceptance Scenarios**:

1. **Given** the user is on Start, **When** they tap "Ya tengo cuenta", **Then** Sign In opens with email, password, forgot-password link, and a create-account link.
2. **Given** valid credentials and onboarding completed, **When** the user signs in, **Then** Home opens.
3. **Given** valid credentials and onboarding not completed (or profile missing), **When** the user signs in, **Then** Onboarding step 1 opens instead of Home.
4. **Given** the user is on Sign Up, **When** they tap the link to sign in, **Then** Sign In opens.
5. **Given** the user is on Sign In, **When** they tap the link to create an account, **Then** Sign Up opens.
6. **Given** wrong email or password, **When** the user submits Sign In, **Then** an invalid-credentials error is shown and they remain on Sign In.
7. **Given** no network, **When** the user submits Sign In, **Then** a no-network error is shown and they remain on Sign In.

---

### User Story 4 - Reset password by email (Priority: P2)

A user who forgot their password requests a reset link from Sign In without leaving the account-required model.

**Why this priority**: Password recovery unblocks returning users; Change Password inside Settings is out of scope for this feature.

**Independent Test**: From Sign In, open Reset Password, submit a valid email, and verify a reset-link confirmation path without signing the user in.

**Acceptance Scenarios**:

1. **Given** the user is on Sign In, **When** they tap "¿Olvidaste tu contraseña?", **Then** Reset Password opens.
2. **Given** a valid email format on Reset Password (whether or not an account exists for that address), **When** the user requests the reset, **Then** the same generic “link sent” confirmation is shown on Reset Password (user remains on that screen) with a clear path back to Sign In, without establishing a session and without revealing whether the email is registered.
3. **Given** an invalid email format, **When** the user submits Reset Password, **Then** an invalid-email error is shown and no link is sent.
4. **Given** no network, **When** the user submits Reset Password, **Then** a no-network error is shown.

---

### User Story 5 - First-launch intro without skip (Priority: P3)

A first-time user who taps Start sees three intro screens with looping animation, segmented progress, and no skip; Reduce Motion softens motion on splash and intro.

**Why this priority**: Intro sets product expectations like peer apps; it is first-launch-only along the Start path and must respect accessibility motion settings.

**Independent Test**: First launch → Start → walk all three Intro screens with Reduce Motion off and on; confirm no Omitir/Skip and button timing rules.

**Acceptance Scenarios**:

1. **Given** first launch after full splash, **When** the user is on Start, **Then** they see the animated character area, tagline, Start and I-have-an-account actions, and Terms and Privacy Policy links that open external pages.
2. **Given** Intro screens 1–3, **When** each screen plays, **Then** a ~9 s looping animation runs, there is no skip control, and Continuar (last: Empezar) appears only after the animation ends.
3. **Given** Reduce Motion is enabled, **When** splash or intro would animate, **Then** motion is reduced to a simple fade or no motion while the flow and copy remain usable.
4. **Given** the user already completed first launch (has launched before) and has no session, **When** they open the app (for example after sign-out from a later feature), **Then** splash is short (logo only) and Start appears without forcing the full first-launch splash or replaying Intro unless they choose Start again.

---

### Edge Cases

- **Abandoned onboarding**: User creates an account or signs in, leaves during Onboarding (app kill or background forever). Next open with that session shows Onboarding step 1, not Home and not Sign Up.
- **No network on sign up**: User submits Sign Up offline. Account is not created; a no-network error is shown; user remains on Sign Up.
- **No network on sign in / reset**: Same class of no-network error; no session change.
- **Email already in use**: Sign Up with an existing email shows email-already-in-use; user remains on Sign Up.
- **Weak password**: Password shorter than 8 characters is rejected before the request is sent; weak-password feedback is shown.
- **Invalid email format**: Sign Up, Sign In, or Reset Password with a malformed email shows invalid-email feedback without sending a request when validation fails client-side.
- **Reinstall with saved session**: After reinstall on the same phone, if the system still holds the session, short splash → Home (or Onboarding if incomplete); cloud progress restores. If the session is gone, welcome flow (Start) applies.
- **Reminder permission denied**: If the user enables reminder and denies notification permission, the reminder stays off with a notice and a way to open system Settings (presentation reminder rules); onboarding can still complete.
- **Empty nickname**: Allowed; nickname is optional and omitted later on Home when empty (Home display itself is out of scope beyond reaching Home).
- **Nickname over 10 characters**: Input does not accept more than 10 characters (or equivalent blocking validation).
- **Submitting while busy**: While Sign Up (or Sign In) is submitting, the primary action stays blocked so the user cannot double-submit.
- **Sign Up has no dismiss**: User cannot close Sign Up with an X; they may use back from Sign Up when it was reached from Intro, or navigate to Sign In via the text link.
- **Session present, profile load fails**: After splash, if a session exists but the profile cannot be loaded (no network or server error), the app keeps the session and shows a retryable error; it does not open Home or Onboarding until the profile loads successfully.
- **Reset for unknown email**: Valid email format with no registered account still shows the same generic “link sent” confirmation as a known email; the UI MUST NOT indicate that the address is unregistered.
- **After reset confirmation**: User remains on Reset Password with the confirmation visible and a clear way to return to Sign In (no auto-redirect that hides the message; no session established).
- **Slow session resolution**: If session/profile is still unknown after the minimum splash duration (~1.5 s full / ~0.8 s short), the splash remains (logo) until resolution succeeds or fails; the app does not flash Start, Home, or Onboarding early.

## Requirements *(mandatory)*

### Functional Requirements

Source tags use document section labels only (no file paths in requirement text).

- **FR-001**: The product MUST require an account; guest mode MUST NOT exist. *(scope §3, §11.2; mockups Flow; data authentication)*
- **FR-002**: Account creation and sign-in MUST use email and password only; Apple sign-in MUST NOT be offered in this feature. *(scope §3, §11.1–2; mockups Canvas status; data authentication)*
- **FR-003**: Email verification MUST NOT be required to complete sign up or reach onboarding/Home. *(scope §3 Out of v1; user feature constraints)*
- **FR-004**: Password MUST be at least 8 characters and MUST be validated before the create/sign-in request is sent. *(data authentication; architecture base authentication errors; scope Change Password notice pattern)*
- **FR-005**: The app MUST surface these authentication failures to the user: invalid credentials, email already in use, weak password, invalid email, and no network. *(architecture base authentication errors; data authentication; offline wait for Sign In)*
- **FR-006**: Root navigation MUST choose the surface from session and profile state: (1) splash until session (and profile when applicable) is known—full splash if the device has never launched before, short splash otherwise; if resolution takes longer than the branded/minimum splash duration, the splash (logo) MUST remain until resolution succeeds or fails—MUST NOT navigate to Start, Home, or Onboarding early; (2) no session → welcome flow (Start, Intro, Sign Up, Sign In, Reset Password); (3) session with onboarding incomplete or profile missing → Onboarding; (4) session with onboarding complete → Home; (5) session present but profile cannot be loaded (network or server failure) → keep the session and show a retryable error surface—MUST NOT open Home or Onboarding until the profile loads successfully. *(presentation root screen and navigation; scope §6; §11.3)*
- **FR-007**: First launch MUST show the full splash (logo, BRENZ, slogan, copyright footer) with animation; later launches with prior launch MUST show logo-only splash for at least about 0.8 s, and longer if session/profile is still resolving (FR-006). *(scope §3, §6, §11.7; mockups Splash screens)*
- **FR-008**: First-launch full splash presents about 1.5 s of branded motion as a minimum while session resolution proceeds per FR-006; if resolution is still pending after that motion, the splash MUST continue (logo visible) until session/profile is known or load fails. *(scope §6 First launch flow)*
- **FR-009**: Start MUST offer Start and I-have-an-account actions, plus Terms and Privacy Policy links that open external pages; an acceptance checkbox or similar control MUST NOT be required on Start. *(scope §6 Screens; §11.9; mockups Start)*
- **FR-010**: Choosing Start MUST open the three-screen Intro; choosing I-have-an-account MUST open Sign In. *(scope §6; mockups Flow)*
- **FR-011**: Intro MUST have three screens, a segmented progress bar, ~9 s looping animation per screen, no skip, Continuar after animation ends, and Empezar on the last screen. *(scope §3, §6; mockups Intro)*
- **FR-012**: Completing Intro MUST open Sign Up. *(scope §6; mockups Flow)*
- **FR-013**: Sign Up MUST collect email and password only (no name or date of birth), MUST NOT show a close/X control, MUST include Terms and Privacy Policy links to external pages, MUST NOT require an acceptance checkbox or similar control before submit, and MUST offer a link to Sign In. *(scope §3, §6; mockups Sign Up; §11.2)*
- **FR-014**: Successful Sign Up MUST establish a session and open Onboarding step 1. *(scope §6 account routes; §11.3; mockups Flow)*
- **FR-015**: Sign In MUST collect email and password, offer forgot-password and create-account links, and on success open Home or Onboarding per FR-006. *(scope §6; mockups Sign In; presentation root)*
- **FR-016**: Reset Password MUST, for any valid email format, show the same generic “link sent” confirmation whether or not an account exists for that address; MUST keep the user on Reset Password with that confirmation visible and a clear path back to Sign In; MUST NOT reveal whether the email is registered; and MUST NOT leave the user signed in by that action alone. *(scope §6 account routes; mockups Reset Password)*
- **FR-017**: Onboarding MUST have exactly three steps with progress dots: (1) welcome and optional nickname, (2) disciplines, (3) optional reminder. *(scope §6; mockups Onboarding)*
- **FR-018**: Nickname MUST be optional, maximum 10 characters, never translated, and requested on onboarding step 1. *(scope §6 Nickname)*
- **FR-019**: Disciplines MUST present the four fixed disciplines with multiple selection and exactly one pre-selected by default; the user MUST keep at least one selected to continue. *(scope §3, §6 Onboarding)*
- **FR-020**: Reminder MUST be optional (days and time). Notification permission MUST be requested only if the user turns the reminder on. *(scope §6; presentation Reminder; mockups Onboarding)*
- **FR-021**: Completing Onboarding MUST set onboarding completed and open Home. *(scope §6; presentation root)*
- **FR-022**: With a saved session and onboarding completed, launch MUST go short splash → Home. *(scope §6 Later launches; presentation acceptance; data authentication session persistence)*
- **FR-023**: The session MUST persist across app restarts; after reinstall on the same device, a surviving session MUST land on Home (or Onboarding if incomplete) with progress restored from the cloud account. *(data authentication; scope §3 progress backup)*
- **FR-024**: All user-visible strings in this feature MUST be available in Spanish and English; device language Spanish → Spanish; any other language → English. *(scope §2; presentation text catalog)*
- **FR-025**: Splash and Intro MUST respect Reduce Motion (fade or no motion instead of looping/bounce motion). *(scope art/animation rules; presentation Animations)*
- **FR-026**: Sign Up and Sign In primary actions MUST stay disabled/blocked while a submission is in progress. *(presentation form submitting behavior)*

### Key Entities

- **Session**: Whether a signed-in account is active on the device; drives root navigation with onboarding state.
- **Account credentials**: Email and password used to create or restore a session; password minimum length 8.
- **User profile (onboarding subset)**: Optional nickname (≤10 characters), selected disciplines (multi-select, one default), optional reminder (days, time, enabled flag), and onboarding-completed flag.
- **Launch marker**: Whether the device has launched the app before; selects full vs short splash.
- **Auth failure**: User-facing failure kinds for this feature: invalid credentials, email already in use, weak password, invalid email, no network, and profile-load failure (session kept; retryable).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A new user can go from first open through Sign Up and all three onboarding steps to Home in under 5 minutes on a stable connection. *(time target under Assumptions)*
- **SC-002**: A returning user with saved session and completed onboarding reaches Home within about 2 seconds of cold start on a stable connection, including the ~0.8 s logo splash; on slow resolution the splash may remain longer until session/profile is known (FR-006). *(latency target under Assumptions; splash duration from sources)*
- **SC-003**: 100% of first-launch paths that complete Intro land on Sign Up with no skip control present on any Intro screen.
- **SC-004**: 100% of attempts to create an account with password length under 8 characters are blocked before a create request is sent.
- **SC-005**: With device language Spanish, every screen in this feature shows Spanish copy; with English (or non-Spanish), every screen shows English copy—no mixed untranslated keys on Start, Intro, Sign Up, Sign In, Reset Password, or Onboarding.
- **SC-006**: With Reduce Motion on, splash and intro do not play looping or bouncing motion; the user can still complete Intro and reach Sign Up.
- **SC-007**: In manual trials of abandoned onboarding (kill app mid-onboarding), the next cold start with the same session opens Onboarding, not Home, in 100% of trials.
- **SC-008**: Offline Sign Up and Sign In attempts show a no-network error and do not change session state in 100% of trials.
- **SC-009**: After successful Sign Up + onboarding, or Sign In with completed onboarding, Home is the next screen in 100% of successful trials (no guest bypass).

## Assumptions

- Invented timing for SC-001: under 5 minutes end-to-end for a careful first-time user; sources do not define a completion-time KPI.
- Invented timing for SC-002: about 2 seconds to Home including the sourced ~0.8 s short splash on a stable connection; sources define splash duration, not total cold-start budget. Slow resolution may keep splash longer (FR-006); SC-002 applies to the stable-connection path.
- Full first-launch splash presents ~1.5 s of branded animation (scope §6) as a minimum while root navigation waits until session state is known (presentation root). If session resolves sooner, the branded splash still completes its first-launch presentation before Start. If session/profile takes longer than the minimum splash, the splash (logo) stays until resolution succeeds or fails—no early navigation to Start/Home/Onboarding.
- The single default selected discipline is one of the four fixed disciplines; sources require a default but do not name which—product lists boxing first among disciplines, treated here as the default selection unless a later product decision names another.
- Sign Up without network behaves like Sign In: request waits on connectivity and surfaces no-network (Sign In offline behavior is explicit in data authentication; Sign Up offline is required by this feature’s edge cases).
- Terms, Privacy Policy, and support URLs exist as external pages by store readiness; this feature only requires that Start and Sign Up open those external pages when tapped. Explicit legal acceptance (checkbox) is not required for Sign Up in this feature.
- Change Password, Delete Account, and Sign Out are specified elsewhere and are out of scope here; after those actions, returning to Start is assumed by product scope but not tested in this feature.
- Home content, tab bar, training, history, and settings are out of scope except as the success destination after auth/onboarding.

## Out of Scope

- Change Password, Delete Account, Sign Out (separate account-settings feature).
- Home and everything after Home (training, water, history, settings), except reaching Home.
- New-user character message on Home (spec 002).
- Guest mode, Apple sign-in, mandatory email verification, phone verification, change email.
- Languages other than Spanish and English.

## Traceability Summary

| Area | Primary sources |
|------|-----------------|
| Mandatory email account, no guest/Apple | scope §3, §11; data authentication; mockups |
| Splash full vs short | scope §3, §6, §11.7; mockups Splash; presentation root |
| Start, Intro, Sign Up, Sign In, Reset, Onboarding | scope §6; mockups screens; §11 |
| Abandoned onboarding → Onboarding | scope §6, §11.3; presentation root; mockups route status |
| Password min 8, auth error kinds | data authentication; architecture base errors |
| Session survives reinstall | data authentication |
| Languages es/en | scope §2; presentation catalog |
| Reduce Motion | presentation Animations; scope animation rules |
| Reminder permission only if on | scope §6; presentation Reminder |
