# Research: Account Onboarding

**Feature**: 001-account-onboarding  
**Date**: 2026-11-27  
**Status**: Complete — all Technical Context unknowns resolved from architecture docs + plan notes

## R1. Shared base bootstrap in feature 001

**Decision**: Feature 001 establishes the production base: local package `Packages/BrenzDesignSystem`, `AppContainer` (Environment-injected from `BrenzApp`), and Firebase SPM products `FirebaseAuth`, `FirebaseFirestore`, `FirebaseRemoteConfig` only.

**Rationale**: Constitution II and base architecture require this stack before any screen can ship. Auth/onboarding cannot run without Auth + Firestore; Remote Config must be wired so `RulesConfig` / `min_version` exist for the version-lock extension point without implementing the lock UI.

**Alternatives considered**:
- Defer BrenzDesignSystem to a later feature → rejected; all screens need tokens and Reduce Motion from day one.
- Add Analytics/Crashlytics “just for pilots” → rejected; constitution forbids them.
- Extra DI libraries (Factory/Swinject) → rejected; own `AppContainer` only.

## R2. Root surfaces and version-lock extension point (cut 0)

**Decision**: `RootViewModel` publishes a single root surface enum covering `splash`, `welcome`, `onboarding`, `home`, `profileLoadError`, and a reserved `versionLock` case (or equivalent slot). Enum case is `welcome`; the first screen in that stack is the **Start screen** in prose. When a future feature marks the app as below `min_version`, `versionLock` wins over every other surface. Feature 001 never activates that case (cut 0); it only leaves the extension point and keeps `VersionComparator` + Remote Config `min_version` available.

**Rationale**: Presentation architecture already lists the version lock as optional/first cut. Plan notes require one extension point so 004 can plug in without reshaping root navigation. Precedence “lock wins” prevents Start screen/Home flashing under a forced update.

**Alternatives considered**:
- Implement full lock UI in 001 → rejected; cut 0 and out of this feature’s scope.
- Hard-code only today’s surfaces with no slot → rejected; forces a breaking root change in 004.
- Separate root coordinator package → rejected; one app module, folders by layer.

## R3. Shared nickname / disciplines / reminder (001 + 004)

**Decision**: Define exactly one Presentation component and one Domain use case per rule set in `Presentation/Shared/` and `Domain/UseCases/`:

| Rule set | Component | Use case |
|----------|-----------|----------|
| Nickname (optional, max 10, never translated) | `NicknameField` | `UpdateNickname` |
| Disciplines (multi-select, catalog order) | `DisciplineMultiSelect` | `UpdateSelectedDisciplines` |
| Reminder (optional days/time, permission only when on) | `ReminderEditor` | `UpdateReminder` |

`CompleteOnboarding` composes validation + persistence (and may call the same repository paths / shared validators). Settings (004) reuses the three components and three use cases; it does not fork them.

**Rationale**: Plan notes mandate one component and one use case per rule set so onboarding and Settings stay identical. Architecture already states nickname is requested in onboarding and edited in Settings; reminder rules are shared.

**Alternatives considered**:
- Single `UpdateProfile` only, with screen-local UI duplication → rejected; UI and validation would drift between onboarding and Settings.
- Put editors in BrenzDesignSystem → rejected; they understand domain concepts (Discipline, ReminderSettings); package must stay entity-free.
- Separate “onboarding-only” use cases plus later Settings use cases → rejected; violates reuse requirement.

## R4. Discipline storage, default, and `lastDiscipline`

**Decision**:
- Stored raw values: `boxing`, `jiu_jitsu`, `muay_thai`, `kickboxing`.
- Unknown Firestore text maps to `.unknown` on read; `.unknown` is never shown in pickers and never written back.
- Catalog order: boxing → jiu_jitsu → muay_thai → kickboxing.
- Onboarding step 2 preselects boxing.
- SignUp seeds `selectedDisciplines = [boxing]` and `lastDiscipline = nil`. The “at least 1 discipline” rule has **no exception** (including the SignUp stub profile).
- `selectedDisciplines` is always persisted in catalog order (not selection-time order). Covered by a Domain test (kickboxing then boxing → stored `[boxing, kickboxing]` conceptually; first = boxing).
- `lastDiscipline` is an optional field on `UserProfile` / `users/{uid}` from day one. Fallback when missing or not in selection: first of `selectedDisciplines` in catalog order. Writers that update `lastDiscipline` (RegisterClass, Home sheet) belong to later features; 001 only ensures the field exists on the entity/DTO and survives onboarding profile writes (leave nil until a later write sets it).

**Rationale**: stored-values.md is document precedence winner; plan notes restate the same contracts.

**Alternatives considered**:
- Default to first alphabetically or last tapped → rejected; product/catalog order names boxing first.
- Store selection-time order → rejected; stored-values and the kickboxing→boxing test require catalog order.
- Omit `lastDiscipline` until training feature → rejected; entity/DTO contract must match Firestore early to avoid migration churn.

## R5. Launch marker vs session

**Decision**: `hasLaunchedBefore` lives in `UserDefaults` at device scope. It is set **once**, when the **full splash finishes the first time**. Sign Out and Delete Account (004) MUST NOT clear it. Full splash (logo + brand + slogan + copyright, **1.5 s** minimum branded motion) only when the marker is false; later launches use the **0.8 s** logo-only splash. Durations measured with an **injected monotonic clock**; splash values are minimums. The marker selects **only** full vs short splash and whether the Start screen leads to Intro; it does **not** choose Home / Onboarding / `welcome`. Destination after splash is decided by **session + onboarding state** (session + completed → home; session + incomplete **or missing profile document** → onboarding; session + profile **fetch failure** → `profileLoadError`; no session → `welcome` / Start screen). Intro is only entered from the Start screen after the user taps Start; after marker is true, cold start with no session goes short splash → Start screen without replaying full splash or auto-playing Intro. A reinstall with a surviving session never shows the Start screen.

**Rationale**: Spec FR-006 / FR-007 / User Story 5 and data architecture: launch marker is not user data; session owns routing.

**Alternatives considered**:
- Tie marker to Auth uid → rejected; sign-out would incorrectly restore full first-launch experience.
- Always full splash → rejected; product requires short splash on later launches.
- Use marker to skip Start screen even without session → rejected; no-session always lands `welcome` / Start screen after splash.

## R6. Auth wait, routing, and profile-load failure

**Decision**:
- `SignUp`, `SignIn`, and `ResetPassword` wait for the server up to a **10 s timeout** (injected monotonic clock); on timeout map to `AuthError.noNetwork` **by design** and leave session unchanged. No email verification step after Sign Up (FR-003).
- After successful auth: if **missing profile document** or `onboardingCompleted == false` → Onboarding step 1; if completed → Home; if profile **fetch fails** → `profileLoadError`.
- If session exists but profile fetch fails (network/server): keep session; show retryable `profileLoadError`; do not open Home or Onboarding until profile loads.
- Splash remains until session (and profile when applicable) resolves or fails, even past the minimum splash duration (hold rule lives in FR-006 only; FR-007/FR-008 define visual content and minimums).
- Root destination after splash is session-driven (see R5); `hasLaunchedBefore` never overrides a surviving session into the Start screen.

**Rationale**: Spec clarifications + FR-005/FR-006; data architecture auth wait rule shared with 004 Change Password / Delete Account. A hung request after the 10 s timeout is treated as connectivity failure for a clear, stable user message.

**Alternatives considered**:
- Fail fast offline without waiting → rejected; explicit 10 s timeout is the product rule.
- Distinct “timeout” error kind → rejected; product maps timeout to `noNetwork` by design.
- Drop session on profile failure → rejected; clarification requires keep + retry.
- Navigate to Start screen when profile is slow → rejected; no early navigation off splash.
- Treat missing profile document as `profileLoadError` → rejected; missing document → Onboarding 1; fetch failure → `profileLoadError`.

## R7. Flow shape, Home destination, and splash/intro animation

**Decision**: Canonical happy path: Splash → Start screen → Intro (3, no skip) → Sign Up (email/password only, no X dismiss, no email verification) → Onboarding (3) → Home. No guest mode. No Apple sign-in. Feature 001 delivers a minimal Home (Train destination) so root routing is testable; History and Settings tab shells are created by feature 002, not here. `BrenzTabBar` may be absent or show Train-only until 002. Splash and Intro use **native SwiftUI with local assets** (no third-party animation libraries). **US1 non-cuttable Intro minimum** (T023): no skip; fixed **9 s** timer then Continuar (Empezar on screen 3); **static fade** baseline; BrenzDesignSystem Reduce Motion modifier at creation (T023/T025). **US5 (cut order 1)** adds **looping animation only**; cutting US5 removes only the looping animation. Timing (injected monotonic clock): splash minimums **1.5 s** full / **0.8 s** short; Intro **9 s**; network timeout **10 s**.

**Rationale**: Spec flow + plan notes; mockups require Home as success destination without pulling History/Settings into this cut. Constitution forbids animation libraries; native SwiftUI matches Reduce Motion rules. Analyze pass 2 ownership: hard Intro rules in US1; US5 is visual polish only.

**Alternatives considered**:
- Stub full three-tab shell in 001 → rejected; plan notes assign shells to 002.
- Guest mode / X on Sign Up as App Review Plan B → rejected for v1 implementation; stays risk note only in scope.
- Lottie / third-party animation packages → rejected; constitution and stack allow native SwiftUI only.
- Put 9 s timer / no-skip only in US5 → rejected; cutting US5 must not remove the usable Intro minimum.

## R8. Password reset privacy

**Decision**: For any valid email format, Reset Password always shows the same generic “link sent” confirmation, keeps the user on that screen with a clear path back to Sign In, establishes no session, and never reveals whether the address is registered.

**Rationale**: Spec clarification session 2026-11-27.

**Alternatives considered**: Firebase “user not found” surfaced to UI → rejected; account enumeration risk and contradicts clarification.

## R9. Testing strategy for this feature

**Decision**: Swift Testing covering: discipline catalog order + unknown mapping; nickname max 10 / optional; CompleteOnboarding; SignUp/SignIn/ResetPassword use cases with fake repos (including ~timeout → noNetwork by design; SignIn client format failure → invalidCredentials, never weakPassword); RootViewModel surface selection matrix under `BrenzTests/Presentation/` (no session, incomplete onboarding, complete, profile error, splash hold; reinstall-with-session never Start); ViewModels for Sign Up / Sign In / Onboarding / Reset Password with `isSubmitting`. Emulator tests against `firebase/firestore.rules`: authenticated owner can create/update own `users/{uid}`; unauthenticated denied; foreign uid denied; invalid discipline string rejected on write where rules enforce it. Test folders allowed: `BrenzTests/{Domain,Data,Presentation}/` only.

**Rationale**: Constitution VI; base architecture test list.

**Alternatives considered**: UI tests for the flow → rejected; no UI tests in v1.
