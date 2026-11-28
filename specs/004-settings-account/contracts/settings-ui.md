# Contract: Settings UI

**Feature**: 004-settings-account  
**Date**: 2026-11-27  
**Mockups**: Settings (7), 7b–7e, 7g–7i  
**Catalog keys**: `settings.*` (en base + es)

## Availability

- Visible only with signed-in session and completed onboarding (Home tab shell from 002).
- Replaces 002 Settings placeholder content; tab destination unchanged.

## Sections (top → bottom)

### Character
- `petNickname` via reused `NicknameField` (max 10, optional; **UI copy: nickname**).
- Saves through `UpdateNickname` (optimistic); input clamped to 10 characters.

### Training
- Disciplines via reused `DisciplineMultiSelect` (≥1) → `UpdateSelectedDisciplines` (lastDiscipline fallback gated by 002 `CurrentDisciplineResolverTests`).
- Reminder via reused `ReminderEditor` → `UpdateReminder` (permission only when turning on; **001 schedules** on enable; denied → off + notice + open system Settings). Reconcile-on-open is **004** (`ReconcileReminder` / `scenePhase`).
- Water goal control 4…16 → `UpdateWaterGoal` (optimistic; Home reads `waterGoal` reactively from the profile stream).

### Account
- Email: read-only from session.
- Change Password → pushes full screen 7d (see account-actions).
- Delete Account → presents sheet 7b (see account-actions).

### More
- Support → open `LegalURLs.support` externally.
- Rate the app → system review dialog (StoreKit); eligibility may no-op.
- Sign Out → immediate action, **no alert** (see account-actions).

### Footer
- App icon + installed version string.
- Terms → `LegalURLs.terms`.
- Privacy Policy → `LegalURLs.privacy`.

## Navigation

- One `NavigationStack` for Settings tab.
- Routes: at least `changePassword`.
- Sheets: delete confirmation / success; password success 7i over Settings.
- Tab bar stays behind floating sheets (same as 003).

## Motion

- 7c / 7i use translucent sheet + `BrenzSuccessMark` + **2.5 s** lime bar.
- Reduce Motion: no animation; outcomes unchanged.
- Prefer package Reduce Motion modifier; no per-view ad-hoc branching.

## Localization

- All chrome strings via String Catalog (`settings.section.character`, `settings.row.changePassword`, error keys for 7e/7g/7h, success copy for 7c/7i, etc.).
- Nickname never translated.
- Version string is the bundle version (not localized as prose).
