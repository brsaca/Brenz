# Quickstart: Account Onboarding

**Feature**: 001-account-onboarding  
**Date**: 2026-11-27  
**Purpose**: Validate the feature end-to-end after implementation. Not an implementation guide.

## Prerequisites

- Xcode 16+ with iOS 17 simulator
- Firebase project configured for the app (`GoogleService-Info.plist` present)
- SPM products linked: `FirebaseAuth`, `FirebaseFirestore`, `FirebaseRemoteConfig` only
- Local package `Packages/BrenzDesignSystem` resolvable by the app target
- Firestore emulator available for rules tests under `firebase/`
- Device/simulator language switchable between Spanish and English

## Setup

1. Open `Brenz.xcodeproj` (or workspace that includes the design-system package).
2. Select an iPhone simulator, iOS 17+.
3. Build and run the Brenz scheme.
4. For rules tests: start the Firestore emulator and run tests in `firebase/tests/` (see repo firebase README when present).

## Automated checks

```bash
# Domain + ViewModel unit tests (Swift Testing)
xcodebuild test -scheme Brenz -destination 'platform=iOS Simulator,name=iPhone 16' \
  -only-testing:BrenzTests
```

Expected highlights:
- Discipline unknown mapping and catalog-order persistence helpers
- Nickname max 10 / optional
- Root surface matrix (no session, incomplete onboarding, complete, profile error)
- SignIn/SignUp timeout → `noNetwork` with fake repos
- `hasLaunchedBefore` not cleared by simulated sign-out / delete cleanup helpers when those land

## Manual validation scenarios

Contracts: [firestore-users.md](./contracts/firestore-users.md), [auth-errors.md](./contracts/auth-errors.md), [root-surfaces.md](./contracts/root-surfaces.md). Model: [data-model.md](./data-model.md).

### V1 — First launch create + onboarding (P1)

1. Reset simulator (or clear app data) so `hasLaunchedBefore` is false and no session exists.
2. Launch → full splash (logo, brand, slogan, copyright) → Start.
3. Tap Start → Intro 1…3 (no skip; Continuar/Empezar after animation).
4. Sign Up with new email + password ≥ 8 (no X on Sign Up).
5. Onboarding: optional nickname ≤ 10; disciplines with boxing preselected; reminder off → Finish.
6. **Expect**: Home (Train). Profile `onboardingCompleted == true`; `selectedDisciplines` in catalog order.

### V2 — Returning session (P1)

1. Force-quit with completed onboarding session.
2. Launch → short logo splash (~0.8 s, longer if resolving) → Home.
3. **Expect**: no Start / Intro / Sign Up / Onboarding.

### V3 — Sign in + incomplete onboarding (P2)

1. Sign Out from a later build or use an account with `onboardingCompleted == false` (or delete profile doc in emulator while keeping Auth).
2. From Start → “I already have an account” → Sign In.
3. **Expect**: Onboarding step 1, not Home.

### V4 — Offline Sign In wait (P2)

1. Enable airplane mode on Sign In with valid-looking credentials.
2. Submit.
3. **Expect**: after ~10 s, no-network error; remain on Sign In; no session.

### V5 — Reset Password privacy (P2)

1. From Sign In open Reset Password.
2. Submit a valid-format email that is not registered.
3. **Expect**: same generic “link sent” UI as for a registered email; stay on Reset Password; path back to Sign In; no session.

### V6 — Launch marker survival

1. Complete first launch once (`hasLaunchedBefore == true`).
2. After Sign Out / Delete Account are available (004), perform them.
3. Cold start with no session.
4. **Expect**: short splash → Start; not the full first-launch splash; Intro only if user taps Start again.

### V7 — Localization + Reduce Motion

1. Set device language to Spanish → every 001 screen shows Spanish catalog copy.
2. Set to English (or non-Spanish) → English copy.
3. Enable Reduce Motion → splash/intro without looping/bounce; flow still reachable to Sign Up.

### V8 — Profile load failure

1. With a valid session, simulate profile read failure (offline without cache, or rules deny temporarily).
2. **Expect**: after splash, retryable error; not Home/Onboarding; session preserved; retry succeeds when profile loads.

## Non-goals for this quickstart

- History / Settings tab shells (002)
- Version lock UI (004; root slot only)
- Change Password / Sign Out / Delete Account UI (004)
- Class/water registration and character messages (002+)
