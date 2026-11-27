# Contract: Root surfaces and navigation

**Feature**: 001-account-onboarding  
**Date**: 2026-11-27

## Root surface enum (conceptual)

```text
splash(kind: full | short)
welcome          // Start screen, Intro, SignUp, SignIn, ResetPassword stack
onboarding       // steps 1…3
home             // Train destination only in 001
profileLoadError // session kept; retry (fetch failure only)
versionLock      // EXTENSION POINT — unused in 001 (cut 0)
```

Terminology: enum case is **`welcome`**; the first screen in that stack is the **Start screen** in prose.

## Who decides what

| Concern | Owner |
|---------|--------|
| Post-splash destination | **Session + profile**: session + `onboardingCompleted` → home; session + incomplete **or missing profile document** → onboarding; session + profile **fetch failure** → `profileLoadError`; no session → `welcome` (Start screen) |
| Full vs short splash | **`hasLaunchedBefore` only** |
| Whether Start screen leads to Intro | **`hasLaunchedBefore` / first-launch Start path** (Intro only after Start action) |

A reinstall with a surviving session **never** shows the Start screen: short splash → Home or Onboarding from cloud profile. `hasLaunchedBefore` MUST NOT override session into `welcome`.

`hasLaunchedBefore` is set **once**, when the **full splash finishes the first time**.

## Precedence (highest first)

1. **`versionLock`** — when a future feature activates it (app version < `min_version`), it wins over every other surface including splash completion targets, `welcome`, onboarding, home, and profile error.
2. **`splash`** — until session is known, and until profile is known when a session exists (or profile fetch has failed).
3. **`profileLoadError`** — session present and profile **fetch/observe failed** (network or server). Not used for a missing document after a successful load path.
4. **`welcome`** — no session (presents Start screen first).
5. **`onboarding`** — session and (`onboardingCompleted == false` OR **missing profile document** after a successful load path).
6. **`home`** — session and `onboardingCompleted == true`.

Feature 001 never sets `versionLock` active. The case/slot must exist so 004 can bind Remote Config `min_version` + `VersionComparator` without reshaping root.

## Timing (injected monotonic clock)

All durations below are measured with an **injected monotonic clock**. Splash and Intro values are **minimums**; the network wait is a **timeout** (exact).

| Concern | Value | Kind |
|---------|-------|------|
| Full splash branded presentation | **1.5 s** | minimum |
| Short splash logo presentation | **0.8 s** | minimum |
| Intro Continuar/Empezar timer (per screen) | **9 s** | minimum (US1); looping polish (US5) aligns to the same cycle |
| SignUp / SignIn / ResetPassword server wait | **10 s** | timeout → `noNetwork` |

| Marker | Minimum branded presentation | After minimum if still resolving |
|--------|------------------------------|----------------------------------|
| `hasLaunchedBefore == false` | Full splash 1.5 s (logo, BRENZ, slogan, copyright) | Keep splash (logo) until resolve/fail (hold rule = FR-006) |
| `hasLaunchedBefore == true` | Short splash 0.8 s (logo only) | Keep splash (logo) until resolve/fail (hold rule = FR-006) |

Do not navigate to the Start screen, Home, or Onboarding before resolution succeeds or fails.

## Welcome flow routes (`welcome` surface)

```text
Start screen
  ├─ Start action → Intro(1) → Intro(2) → Intro(3) → SignUp
  └─ I already have an account → SignIn
SignUp ↔ SignIn (text links)
SignIn → ResetPassword (forgot password)
ResetPassword → SignIn (after confirmation path)
SignUp success → Onboarding(1)
SignIn success → Onboarding(1) | Home | profileLoadError per profile outcome
```

Constraints:
- Intro (US1 non-cuttable minimum): 3 screens, segmented progress, **no skip**; fixed **9 s** timer then Continuar (Empezar on screen 3); **static fade** baseline (no loop); BrenzDesignSystem Reduce Motion modifier at creation. `NavigationStack` Intro → Sign Up with back button; **no X**.
- Intro looping animation is US5 polish only; cutting US5 removes **only the looping animation**.
- Sign Up: email + password only; Terms/Privacy links; no acceptance checkbox; **no X / dismiss**; **no email verification step**.
- No guest mode.
- Splash/Intro motion: native SwiftUI + local assets.

## Shared editors (onboarding ↔ Settings 004)

| Surface | Component | Use case |
|---------|-----------|----------|
| Nickname (UI copy) / `petNickname` (stored) | `NicknameField` | `UpdateNickname` |
| Disciplines | `DisciplineMultiSelect` | `UpdateSelectedDisciplines` |
| Reminder | `ReminderEditor` | `UpdateReminder` |

Onboarding hosts these inside its three steps; Settings (004) embeds the same components and calls the same use cases.

## Out of scope shells

- History tab shell → feature 002
- Settings tab shell → feature 002
- Version lock UI / App Store button → feature 004 (uses this extension point)
