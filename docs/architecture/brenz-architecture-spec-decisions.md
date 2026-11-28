# Brenz — Architecture: Spec decisions (features 001–004)

Date: 2026-11-27
Status: closed by Bren. No code.
Repo location: `docs/architecture/brenz-architecture-spec-decisions.md`
Base: `brenz-architecture-domain.md`, `brenz-architecture-data.md`, `brenz-architecture-presentation.md`.
Precedence: this document wins over `brenz-architecture-domain.md`, `brenz-architecture-data.md`, `brenz-architecture-presentation.md` and the scope where they differ. It does not override `brenz-architecture-stored-values.md`.

## Routing and splash
- [DECIDED] The session decides the destination: session + `onboardingCompleted` → home; session + incomplete or missing profile document → onboarding; no session → welcome/Start; profile fetch failure → `profileLoadError`. `hasLaunchedBefore` only picks full vs short splash and whether Start leads to Intro. A reinstall with a saved session never shows Start.
- [DECIDED] `hasLaunchedBefore` is set once, when the full splash finishes the first time. It survives Sign Out and Delete Account (`LaunchMarkerStore` is never cleared).

## Sign Up and onboarding seed
- [DECIDED] Sign Up seeds `selectedDisciplines = [boxing]` and `lastDiscipline = nil`. Onboarding preselects boxing. The rule "at least 1 discipline" has no exception.

## Pet state bands
- [DECIDED] Pet state bands are derived from Remote Config: active if `daysSinceLastClass < downDays`; down if `downDays <= days < lazyDays`; lazy if `days >= lazyDays`. With defaults (4, 7) active is 0 to 3 days. No new Remote Config field. Invalid values (`downDays < 1` or `downDays >= lazyDays`) fall back to in-app defaults.

## Timing constants
- [DECIDED] Network wait 10 s maps to `noNetwork` by design; splash minimums 1.5 s (full) and 0.8 s (short); Intro cycle 9 s; pending sync before Sign Out 3 s; success sheet bar 2.5 s; water message 4 s; system notice toast 2.5 s auto-dismiss. All measured with an injected monotonic clock.

## Intro and Reduce Motion
- [DECIDED] Intro minimum is not cuttable: no skip, "Continuar" after the fixed 9 s timer, "Empezar" on screen 3, static fade. Cutting the Intro animation removes only the loop. Reduce Motion is applied from the first version of splash and Intro.

## Auth errors
- [DECIDED] Sign In: any client-side format failure returns `invalidCredentials`. `weakPassword` exists only for Sign Up.
- [DECIDED] Change Password has no "new equals current" error in v1.

## Class log ordering and use case ownership
- [DECIDED] Class log ordering: `registeredAt` ascending, then log id ascending. The hour limit is enforced in 002. History ordering and overflow belong to 003.
- [DECIDED] Use case ownership: `RegisterClass`, `DeleteClass` (Undo) in 002; `ChangeClassDiscipline` in 003; `ReconcileReminder` (on app open) in 004; `ProfileRepository` is extended, never duplicated.

## Messages
- [DECIDED] Message selection: once-flag per stage reached. Precedence: class-event (return or stage) > new-user > midday > state message.
- [DECIDED] `MessageRotationStore.clear()` runs on Sign Out and Delete Account.

## Localization boundary
- [DECIDED] Domain holds no localized text. Day labels use `DayLabelKind` (`today`, or `date(Date)`); Presentation localizes.
- [DECIDED] Figure 6d copy: today → "{count} clases hoy" (en "{count} classes today"); another day → "{count} clases ese día" (en "{count} classes that day"). Plural variations for 0, 1, n.

## Constraints
- Do not add Remote Config keys.
- Do not store derived values.
- Do not create a second `firestore.rules`.

## Acceptance criteria
- A test covers session + `onboardingCompleted` → home; session + incomplete/missing profile → onboarding; no session → Start; profile fetch failure → `profileLoadError`.
- A test covers `hasLaunchedBefore` set after the first full splash and still set after Sign Out and Delete Account.
- A test covers Sign Up seed: `selectedDisciplines = [boxing]`, `lastDiscipline = nil`; Onboarding rejects zero disciplines.
- A test covers pet bands with defaults (4, 7): days 0–3 active, 4–6 down, ≥ 7 lazy; invalid Remote Config falls back to defaults.
- A test covers each timing constant with an injected monotonic clock (network 10 s → `noNetwork`, splash 1.5/0.8 s, Intro 9 s, Sign Out sync 3 s, success bar 2.5 s, water message 4 s, toast 2.5 s).
- A test covers Intro: no skip; "Continuar" only after 9 s; "Empezar" on screen 3.
- A test covers Sign In client-side format failure → `invalidCredentials`; `weakPassword` only on Sign Up.
- A test covers Change Password accepting a new password equal to the current one (no dedicated error).
- A test covers class logs ordered by `registeredAt` ascending, then log id ascending.
- A test covers use case ownership: `RegisterClass`/`DeleteClass` in 002, `ChangeClassDiscipline` in 003, `ReconcileReminder` on app open in 004; no duplicate `ProfileRepository`.
- A test covers message precedence: class-event > new-user > midday > state; once-flag per stage reached.
- A test covers `MessageRotationStore.clear()` on Sign Out and Delete Account.
- A test covers Domain returning `DayLabelKind` without localized strings; Presentation maps today vs other-day figure 6d copy with plurals for 0, 1, n.
