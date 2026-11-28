# Contract: Version lock root surface

**Feature**: 004-settings-account  
**Date**: 2026-11-27  
**Cut**: 0 (optional UI; comparator + Remote Config field always present)

## Extension point

Feature 001 reserves root surface `versionLock` with highest precedence:

```text
versionLock > splash hold > profileLoadError > welcome > onboarding > home
```

004 **activates** that surface when the optional lock is included in the build.

## Evaluation moments

- Cold start (as soon as `RulesConfig` / installed version are known).
- Every time the app enters foreground.

## Comparison

| Input | Behavior |
|-------|----------|
| `min_version` missing / invalid | Treat as `"0.0.0"` → never lock |
| Installed ≥ min | No lock |
| Installed < valid min | Show lock screen (if UI not cut) |

Pure function: `VersionComparator` (Domain). `now` not required. Installed version from bundle; min from `RulesConfigRepository` (Remote Config key `min_version`, code default `"0.0.0"`).

## Lock screen UI (when not cut)

- Blocks all other app use (no dismiss into Home/Settings).
- Primary action: open App Store listing via local URL constant (App Store Connect app id).
- No mockup required; dark theme + DesignSystem tokens.
- Strings in catalog (`versionLock.*`).

## Release safety

- Default remote/code value remains `"0.0.0"` until a deliberate ops decision.
- **Never** raise `min_version` before Apple approves that app version (constitution X / data Remote Config).

## Cut behavior

If cut 0 removes the lock UI:
- Still fetch/validate `min_version`.
- Still run `VersionComparator` in Domain/tests.
- Root must not falsely lock; invalid/missing still means allow.
- Reactivating the UI later only wires the existing surface slot.
