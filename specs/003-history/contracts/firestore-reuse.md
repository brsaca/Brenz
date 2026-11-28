# Contract: Firestore and repository reuse (History)

**Feature**: 003-history  
**Rule**: Do **not** add a second rules file or duplicate repository protocols.

---

## Paths (unchanged)

```text
users/{uid}
users/{uid}/classLogs/{logId}
users/{uid}/waterLogs/{logId}
```

---

## classLogs mutations used by History

| Operation | Allowed fields | Source |
|-----------|----------------|--------|
| Update (ChangeClassDiscipline) | `discipline` only (one of `boxing`, `jiu_jitsu`, `muay_thai`, `kickboxing`) | Existing `firestore.rules` from 001/002 |
| Delete (DeleteClass) | Document delete | Same rules file |
| Create | Not from History | 002 RegisterClass |

**Forbidden updates**: `elementId`, `registeredAt`, any other field.

---

## TrainingRepository (existing protocol)

History-relevant methods (names may match 002 implementation; behavior is normative):

```text
observeClasses(uid:from:to:) -> AsyncStream<[ClassLog]>
observeWater(uid:from:to:) -> AsyncStream<[WaterLog]>
updateDiscipline(uid:logId:discipline:) -> …   // optimistic
deleteClass(uid:logId:) -> …                   // optimistic
```

Week range queries use `registeredAt` bounds for the current Monday–Sunday local week (device `Calendar`).

---

## ProfileRepository (existing protocol)

History **reads** profile when assembling pet/stage context after delete if needed via existing observe streams. History **must not** call profile update for `lastDiscipline` or `stageReached` as part of discipline change or delete.

---

## Emulator tests

Extend existing `firebase/` emulator suite only if a History-specific rule edge is missing. Prefer proving “update cannot change `elementId`” once in the shared rules tests rather than a parallel file.
