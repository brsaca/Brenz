# Contract: Firestore classLogs and waterLogs

**Feature**: 002-core-loop  
**File**: extend existing `firebase/firestore.rules` from 001 (do **not** add a second rules file).

## Paths

```text
users/{uid}/classLogs/{logId}
users/{uid}/waterLogs/{logId}
```

## Shared access

- Default deny.
- Read/write only when `request.auth != null && request.auth.uid == uid`.
- No cross-user or cross-collection queries beyond owner paths.

## classLogs

### Create

Allowed fields only: `discipline`, `elementId`, `registeredAt`.

| Field | Rule |
|-------|------|
| `discipline` | Must be one of `boxing`, `jiu_jitsu`, `muay_thai`, `kickboxing` |
| `elementId` | Integer 1…7 |
| `registeredAt` | Timestamp/Date; must not be greater than `request.time` + 1 day |

**Not enforced in rules**: one-class-per-local-hour (device `RegisterClass` only).

### Update

- Only `discipline` may change.
- `elementId` and `registeredAt` must remain identical to the existing resource.
- New `discipline` must be one of the four valid values.

### Delete

Allowed for the owner.

## waterLogs

### Create

Allowed fields only: `registeredAt`.

| Field | Rule |
|-------|------|
| `registeredAt` | Must not be greater than `request.time` + 1 day |

**Not enforced**: “today only”, glass count, or time-of-day limits.

### Update

Denied.

### Delete

Allowed for the owner.

## Profile fields touched by this feature

Extend 001 `users/{uid}` rules as needed:

| Field | Rule |
|-------|------|
| `lastDiscipline` | Optional; if present, one of the four valid discipline strings |
| `stageReached` | Integer; on update must be ≥ existing value (never decreases); values 1 or 2 in v1 |

## Emulator tests (add to `firebase/tests/`)

- Unauthenticated: deny classLogs/waterLogs read/write.
- Other user uid: deny.
- Create classLog with invalid `discipline`: deny.
- Create classLog with `elementId` 0 or 8: deny.
- Update classLog `registeredAt` or `elementId`: deny.
- Create waterLog with extra fields / update waterLog: deny.
- Lower `stageReached` on profile: deny.
- Valid create/delete for owner: allow.
