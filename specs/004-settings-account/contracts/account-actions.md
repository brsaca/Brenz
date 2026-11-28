# Contract: Account actions (Sign Out, Delete Account, Change Password)

**Feature**: 004-settings-account  
**Date**: 2026-11-27  
**Sources**: data architecture Sign out / Delete account; spec FR-003–FR-013

## Shared constraints

| Action class | Network wait | Failure |
|--------------|--------------|---------|
| SignOut pending sync | ≤ **3 s** then continue | Unsynced offline classes may be lost (accepted); no alert |
| ChangePassword / DeleteAccount | ≤ **10 s** | `noNetwork`; action not completed |
| Preference Update* | Optimistic | Server rejection not shown (v1) |
| Success sheet bar (7c / 7i) | **2.5 s** auto-close | Reduce Motion → static; navigation unchanged |

Local cleanup **always** calls `MessageRotationStore.clear()` and **never** clears `LaunchMarkerStore` / `hasLaunchedBefore`.

---

## SignOut

**Trigger**: More → Sign Out (no confirmation).

**Order**:
1. `waitForPendingWrites` (≤ **3 s**)
2. Auth sign out
3. Firestore `terminate` + `clearPersistence`
4. Clear per-user UserDefaults / `MessageRotationStore.clear()`
5. Cancel reminders
6. Rebuild repositories → root shows Start

**Postcondition**: No session; cloud-synced progress remains for later Sign In; `hasLaunchedBefore` unchanged.

---

## DeleteAccount

### Sheet 7b
- Darkened Settings behind translucent sheet.
- Current password field (eye toggle).
- Destructive confirm **inactive** while field empty.
- Cancel → dismiss; Settings unchanged.

### Success path
1. Reauthenticate with password
2. Delete Firestore data in batches: `classLogs`, `waterLogs`, then `users/{uid}`
3. Delete Auth user
4. Local cleanup like Sign Out steps 3–6
5. Same sheet → 7c content (`BrenzSuccessMark`, **2.5 s** bar, “Volviendo al inicio”)
6. Root → Start

### Failure: wrong password (7e)
- Delete **nothing** (no Firestore, no Auth, no local clear)
- Field error; sheet stays open

### Failure: network
- After **10 s** → no-network message; nothing deleted

### Reduce Motion
- 7c static; still navigates to Start after presentation window / immediate dismiss policy consistent with package modifier

---

## ChangePassword

### Screen 7d
- Full screen (Sign Up style), back to Settings.
- Fields: current, new, repeat (each with eye).
- Notice: new password needs 8+ characters.
- **No** recovery link.
- Primary button enabled iff:
  - all three non-empty
  - `new.count >= 8`
  - `new == repeat`
- **No** “new equals current” validation in v1.

### Errors
| Case | UI |
|------|----|
| Repeat mismatch | 7h on repeat field; button stays disabled when mismatch |
| Wrong current password | 7g on current field; stay on 7d; password unchanged |
| Timeout / offline | noNetwork after **10 s**; password unchanged |

### Success 7i
- Translucent sheet over darkened Settings (reuse 6d/7c pattern).
- Auto-closes after **2.5 s** bar (or Reduce Motion static path).
- Settings remains; next Sign In uses new password.

---

## Domain / Data mapping (conceptual)

```text
ChangePassword.execute(current, new)
  → AuthRepository.reauthenticate + changePassword (wait ≤ 10 s)

DeleteAccount.execute(password)
  → AuthRepository.reauthenticate
  → delete all user Firestore docs (Training + Profile)
  → AuthRepository.deleteUser
  → local cleanup (MessageRotationStore.clear, reminders, cache)
  → (hasLaunchedBefore never cleared)

SignOut.execute()
  → pending writes ≤ 3 s → signOut → local cleanup (MessageRotationStore.clear)
  → (hasLaunchedBefore never cleared)

ReconcileReminder.execute()  // 004; on app open / scenePhase
  → permission revoked → reminder off + cancel
  → permission granted ∧ on → rebuild schedule from profile
```

Presentation maps `AuthError.wrongCurrentPassword` → 7g / 7e copy; `AuthError.noNetwork` → no-network copy.
