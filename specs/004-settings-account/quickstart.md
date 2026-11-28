# Quickstart: Settings and Account Management

**Feature**: 004-settings-account  
**Date**: 2026-11-27  
**Purpose**: Validate the feature end-to-end after implementation. Not an implementation guide.

## Prerequisites

- Features 001–003 implemented enough to reach Home with tab bar, Settings shell, shared Nickname/Discipline/Reminder components, translucent sheet + `BrenzSuccessMark`, Auth session, and Firestore profile.
- Feature 002 `CurrentDisciplineResolverTests` exist as the gate for lastDiscipline fallback (004 does not duplicate them).
- Xcode 16+, iOS 17 simulator, Brenz scheme.
- `GoogleService-Info.plist` present; Firebase Auth + Firestore + Remote Config linked.
- `LegalURLs` constants reachable for Support / Terms / Privacy.
- Device language switchable es / en.
- Firestore emulator available if extending rules tests under `firebase/`.

## Setup

1. Open the Brenz Xcode project/workspace.
2. Build and run on an iPhone simulator (iOS 17+).
3. Sign in (or Sign Up + complete onboarding) so Home tabs appear.
4. Open the Settings tab (replaces 002 placeholder).

## Canonical durations

| Constant | Value |
|----------|-------|
| Network wait (Change Password / Delete Account) | **10 s** |
| Sign Out pending sync | **3 s** |
| Success sheet auto-close bar (7c / 7i) | **2.5 s** |

## Automated checks (required)

Run Swift Testing suites (**only** `BrenzTests/Domain`, `BrenzTests/Data`, `BrenzTests/Presentation`):

```bash
# From Xcode: Test BrenzTests, or xcodebuild test on the Brenz scheme
```

Required assertions (see [research.md](./research.md) R11 and [contracts/account-actions.md](./contracts/account-actions.md)):

| # | Case | Layer |
|---|------|-------|
| 1 | Change Password enable rules (empty / &lt;8 / mismatch / valid) + offline → noNetwork within **10 s** | Domain (`ChangePasswordTests.swift`) |
| 2 | Wrong current password → stay on 7d + 7g; noNetwork mapping | Presentation (`ChangePasswordViewModelTests.swift`) |
| 3 | Delete Account wrong password → deletes nothing; order reauth → Firestore → Auth; offline → noNetwork within **10 s** | Domain (`DeleteAccountTests.swift`) |
| 4 | Delete Account ViewModel confirm / 7e / noNetwork / 7c → Start | Presentation |
| 5 | `hasLaunchedBefore` survives Sign Out and Delete Account (never cleared) | Data / Domain |
| 6 | Water goal rejects outside 4…16; accepts bounds | Domain |
| 7 | `ReconcileReminder`: permission revoked → reminder off; granted + on → schedule rebuilt | Domain |
| 8 | Permission denied → reminder off + notice + open system Settings; Sign Out no confirmation; link handlers; nickname clamped to 10 | Presentation (`SettingsViewModelTests.swift`) |
| 9 | lastDiscipline fallback | **002** `CurrentDisciplineResolverTests` (do not re-test in 004) |

## Manual scenarios

### A. Sign Out (P1)

1. From Settings, tap Sign Out.
2. Expect: **no alert**; pending sync up to **3 s**; Start appears; reminders gone; local session cleared; `MessageRotationStore` cleared.
3. Confirm short splash / Start path still respects prior `hasLaunchedBefore` (never cleared by Sign Out).
4. Sign In again → cloud progress available if previously synced.
5. Optional airplane mode with unsynced class: still signs out; unsynced loss accepted.

### B. Delete Account (P1)

1. Settings → Delete Account → 7b.
2. Empty password → confirm inactive; Cancel leaves Settings.
3. Wrong password → 7e; data intact.
4. Correct password online → reauth, data+Auth deleted, 7c (**2.5 s**), Start, no session; `hasLaunchedBefore` intact; `MessageRotationStore.clear()` ran.
5. Reduce Motion on → 7c without animation, still Start.
6. Airplane mode → wait **10 s** → no-network; nothing deleted.

### C. Change Password (P1; cut 4 if needed)

1. Open 7d: no recovery link.
2. Verify button enable rules; mismatch → 7h.
3. Wrong current → 7g; stay on screen.
4. Valid online → 7i (**2.5 s**) auto-closes → Settings; Sign In works with new password.
5. Offline → **10 s** → no-network.
6. Reduce Motion → 7i without animation.

### D. Preferences (P2)

1. Edit nickname / `petNickname` (≤10 / empty), disciplines (≥1), water goal 4…16, reminder on/off.
2. Return to Home: preferences apply without sync wait.
3. Reminder denied permission → stays off + notice + path to system Settings.
4. Lower water goal below today’s glasses → glasses remain; Home recalculates from profile `waterGoal` stream (do not change Home unless this fails).
5. 10-character nickname: no clipping on Settings and Home (SC-010).
6. Background/foreground or cold open with reminder on: schedule reconciles (`ReconcileReminder`).

### E. Links / Rate / Footer (P2)

1. Support (More) opens external page via 001 URL constants.
2. Rate triggers system review (may no-op).
3. Footer shows icon + version; Terms/Privacy from footer open external pages via 001 URL constants.

### F. Version lock (P3; cut 0)

1. With valid min &gt; installed (debug override / Remote Config) → lock on cold start and on foreground; App Store button works.
2. Equal/higher installed or invalid/missing min → no lock.
3. If cut: confirm comparator defaults still never false-lock.

## Expected outcomes

- Settings matches [contracts/settings-ui.md](./contracts/settings-ui.md).
- Account flows match [contracts/account-actions.md](./contracts/account-actions.md).
- Reuse boundaries honored ([contracts/reuse.md](./contracts/reuse.md)): no second `firestore.rules`; `MessageRotationStore.clear()` on Sign Out / Delete; `hasLaunchedBefore` never cleared.
- Version lock matches [contracts/version-lock.md](./contracts/version-lock.md) when included.
- es and en show complete Settings copy with no mixed keys.

## Out of this quickstart

- Implementation code, migrations, full test source.
- Raising production `min_version` (ops only after Apple approval).
- Change email / Apple Sign In / recovery from Settings.
