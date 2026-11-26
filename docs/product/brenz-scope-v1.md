# Brenz — Scope and decision status (v1)

Date: 2026-11-26 (revision 21)
Status: logic, flow, messages and architecture closed. v1 in Spanish and English. Account with email only. Firestore as the only source. Mockups v1 aligned with email only. Message bank closed (`docs/product/brenz-message-bank.md`), English version pending. Schedule re-estimated: build 40–53 h against 47 h available. Spec Kit follows (Fri Nov 27, together with project creation). Art for 7 board elements missing (Sat Nov 28). No code.
Revision 21: document translated to English. File names, paths and token names are now English. Quoted UI copy is the `es` string (marked `(es)`). The `en` copy is drafted with the message bank (Tue Dec 1).

Legend: [DECIDED] closed by Bren. [PROPOSED] proposed by Claude, to be confirmed. [PENDING] not discussed. [REMOVED] left v1.
Screen details and flow: `docs/product/brenz-mockups-v1.md`. Character messages: `docs/product/brenz-message-bank.md`. Canvas: https://claude.ai/artifact/UpgmAerVfchKrzyHe88zHs
Architecture (Nov 26): `docs/architecture/brenz-architecture-base.md`, `-domain.md`, `-data.md` and `-presentation.md`. If the scope and the architecture differ, the architecture wins.

---

## 1. Product focus

- [DECIDED] Product narrowed to physical training. Not generic habits.
- [DECIDED] Reason: the generic habits market is saturated.
- [DECIDED] Pilot at Bren's gym, with iPhone users.
- [DECIDED] Disciplines: boxing, jiu-jitsu, muay thai, kickboxing.
- [DECIDED] v1 habits: attend class and drink water.
- [DECIDED] Nutrition enters in v2.
- [DECIDED] The product is not tied to a character or a clothing style. Rules, stages and messages hold for any character.
- [DECIDED] Vision: several characters, and each user picks the one they identify with (men, women, diverse professions). Out of v1.
- [DECIDED] v1 character: fox. The tiger was dropped (inconsistent stripes in AI, gym cliché, less identification with the user). Its final design is open (section 10).

## 2. Positioning

- Habits with a pet: Finch dominates. Plus costs $39.99 per year.
- Combat loggers: Momentum (Pro $59.99/year, $6.99/month), Marune, MA Journey, Jacks, Combat-Performance, Fitrack.
- Gap: pet + sarcastic tone + local gym.
- [PROPOSED] Do not compete on logging depth. No techniques or belts.
- [DECIDED Nov 26] v1 language: Spanish and English. iPhone language Spanish: Spanish. Any other language: English. Texts live in a String Catalog with `en` as the base language and `es`. Replaces "Mexican Spanish only".
- [PROPOSED] Store category: Health & Fitness.

## 3. v1 scope

In:
- [DECIDED] 4 fixed disciplines, with one as the default.
- [DECIDED] Manual class log, one tap, today only, with "Undo" for 5 s. Several classes per day count: each tap adds one (same dynamic as Forest).
- [DECIDED] The main Home button is "Fui a clase" (es) (Nov 26). No "+" button in the tab bar in v1. No long press in Home. If the pilot shows confusion, change the text to "Registrar clase" (es). The "+" is re-evaluated in v1.1, when there are two or more log actions.
- [DECIDED] Each log stores date and time. History groups the hours into 3-hour blocks.
- [DECIDED] Edit or delete a logged class (Nov 26): in History, tapping a tatami cell opens a translucent sheet from the bottom with two actions. Change the discipline and delete the log. Neither time nor date is edited. Delete asks for confirmation and shows a class-deleted animation. Mockups 6b to 6e.
- [DECIDED] Classes from previous days are not added.
- [DECIDED] No weekly goal. No pressure.
- [DECIDED] Water: glass counter. + and − buttons inside the board card in Home (Nov 26). There is no separate "+ Water" button. Each tap adds or removes one glass.
- [DECIDED Nov 26] Each glass is stored as an event with its time (`registeredAt`), to get statistics on drinking time. Only today's glasses are added. No limit on glasses or time.
- [DECIDED] The day is the local calendar day. Water resets at local midnight.
- [DECIDED] Sports bottle always visible in Home. It fills when adding and drops animated when removing.
- [DECIDED] Today's board in Home: class tatami + bottle (section 6).
- [DECIDED] One character with 4 states: animated, normal, down, lazy.
- [DECIDED] 2 stages tied to attendance consistency: "Intento" (es) and "Reincidente" (es) (Nov 26). Stage 3 ("Irremediable" (es)) in v1.1.
- [DECIDED] Character nickname, optional, customizable by the user (section 6).
- [DECIDED] Character messages (Nov 26): 44 messages in first person, no nickname. Detail, triggers and rotation in `docs/product/brenz-message-bank.md`. English version pending.
- [DECIDED] Optional local reminder (days and time).
- [DECIDED] History (Nov 26): a single screen with the current week (Monday to Sunday). The current day has a border. Past days with training have a checkmark. Below: the day's tatami and Classes and Water cards with a 3-hour block chart. No Day/Week tabs. No event list. No navigating to previous weeks.
- [DECIDED] Navigation: floating tab bar with Train (Home), History and Settings. It leaves room for future sections. Own component `BrenzTabBar`, inside the design system package (Nov 26). Cut: native `TabView`.
- [DECIDED] Free. No purchases. iPhone only. iOS 17 minimum (Nov 26).
- [DECIDED] No user profile (age, weight, height).
- [DECIDED] Firebase from v1. Reason: back up progress if the app is uninstalled and leave the core ready for a future Android.
- [DECIDED] Firestore is the only data source (Nov 26). It uses its offline persistence: the app works in the gym without network. No own local database or sync layer.
- [DECIDED] Firebase Remote Config in v1 to adjust the rules and set a minimum version (`min_version`) without publishing a release (Nov 26).
- [DECIDED] Dark mode only in v1 (Nov 26, replaces the cream light mode). Lime accent. Only the character keeps 90s aesthetics.
- [DECIDED] Mandatory account with email and password only (Nov 26, replaces guest mode and removes Apple from v1). Reference: Forest. Covers sign up, sign in, reset password, change password, sign out and delete account. No name or date of birth requested. There is no guest mode.
- [DECIDED] Sign in with Apple leaves v1 (Nov 26). Reason: linking Apple with email accounts is complex (Apple can hide the email). Guideline 4.8 does not require offering it when the app only uses its own login. It returns in v1.1 with provider linking.
- [DECIDED] The Account screen disappears (Nov 26): with a single method it would be a single button. The first launch goes from the intro straight to Sign Up, without an X.
- [DECIDED] Sign Up appears once, on the first launch, after the intro. On later launches the session stays saved and the app opens Home.
- [DECIDED] Settings exists only with a signed-in session. It shows Email, Change Password, Delete Account and Sign Out.
- [DECIDED] Sign Out has no alert: it signs out, deletes local data (it stays in the cloud) and returns to the first screen.
- [DECIDED] Delete Account (Nov 26): in Settings it opens a confirmation sheet with "Lamentamos verte ir" (es) and the "Contraseña actual" (es) field (7b). On confirming, the same sheet shows an account-deleted animation (7c) and the app returns to Start. Mockups 7b, 7c and 7e.
- [DECIDED] Change Password (Nov 26): screen with current password, new password and repeat (7d). On update, a success sheet with animation (7i) and back to Settings. Errors in 7g and 7h. It does not offer password recovery.
- [DECIDED] Splash, Start ("Empezar" (es) / "Ya tengo cuenta" (es)) and a 3-screen intro with animation, like Forest. No "Skip". The button appears when the animation ends.
- [DECIDED] Splash (Nov 26): the first time it is shown in full, with text and animation. Later times it shows only the logo, about 0.8 s.
- [DECIDED] Settings includes Support, "Rate the app" (es: "Calificar la app"), version, Terms and Privacy Policy. Support, Terms and Policy open an external web page.

Out of v1:
- Character picker. Nutrition, HealthKit, on-device AI, widgets, custom habits, social, Apple Watch, iPad, Android.
- Guest mode (use without an account).
- Sign in with Apple and provider linking (v1.1).
- "+" button in the tab bar (v1.1, if there is more than one log action).
- Store of visual elements and holiday elements (the catalog stays prepared, section 7).
- Other board themes, such as the ring (the catalog stays prepared, section 7).
- Board elements per discipline (v1.1).
- History of previous weeks, by Month and by Year.
- Light theme. Background and palette customizable per user (the theme stays prepared, section 7).
- Memory mini-game with combinations per discipline (v1.1; start with boxing only). Estimated 5–6 h plus art.
- Change email from the app. Mandatory email verification. Phone verification.
- Languages other than Spanish and English.
- Water drinking-time statistics (the data is already stored).

## 4. Brenz rules (closed)

State (recalculated with the logged classes):
- Lazy: 7 days or more without a class.
- Down: 4 to 6 days without a class.
- Animated: last class 3 days ago or less and 2 or more classes in the last 7 days.
- Normal: last class 3 days ago or less and fewer than 2 classes in the last 7 days.
- New user: starts at normal.
- Water: if the day's goal is met, normal rises to animated only that day. Water never lowers the state.
- [DECIDED] The animated state from water happens only on meeting the goal. Each glass adds only a short bounce, without changing the state.
- [DECIDED] If the user removes a glass and drops below the goal, today's state is recalculated. It returns to normal.

Stage (attendance consistency):
- [DECIDED] It rises with accumulated classes. It never goes down.
- [DECIDED] Deleting a class does not lower the stage reached. The stage reached is stored. The state is recalculated with the remaining classes.
- [DECIDED] Stage 1 to stage 2 at 10 accumulated classes.
- [DECIDED] Stage names (Nov 26): stage 1 "Intento" (es), stage 2 "Reincidente" (es), stage 3 (v1.1) "Irremediable" (es). Replaces Beginner, Steady and Veteran.
- [DECIDED] Stages have a numeric internal key (1, 2, 3). The visible name is separate text (key `stage.1.name` in the catalog) and can change without touching art or logic. Messages receive the stage name as an argument.
- [DECIDED] Each character defines its look per stage. The stage logic is the same for all.

Reset:
- There is no button. The next logged class returns the character to normal.

Configurable values (Remote Config, with default values inside the app):
- Threshold of classes for animated: 2.
- Window: 7 days.
- Days for down: 4.
- Days for lazy: 7.
- Classes for stage change: 10.
- Default water goal: 8 glasses (editable from 4 to 16 in Settings).
- [DECIDED Nov 26] `min_version`: minimum app version. Default `0.0.0`. If the installed version is lower, a (optional) lock screen asks to update. Never raise it before Apple approves that version.
- The app must work with the default values offline.

## 5. Content rules (from the project)

- Sarcasm aims at actions. Never at the person.
- Failure message, in order: acknowledge the fact, observe the habit, propose a small step.
- No guilt, punishment or mockery.
- The pet does not die. If the user fails, the character is down or lazy.
- The user can always start over.
- Messages do not mention work, office or clothing. They talk about classes, consistency and training.
- [DECIDED] Message bank closed (Nov 26): `docs/product/brenz-message-bank.md`. Covers states, return after lazy, new user, stage change and water. Includes rotation and display rules. English version: Claude draft and tone review by Bren (Tue Dec 1).

## 6. Screens and flow v1

First launch flow:
1. Full splash, with animation (~1.5 s).
2. Start: "Empezar" (es) or "Ya tengo cuenta" (es).
3. Intro (3 screens). The last button says "Empezar" (es).
4. Sign Up (mandatory, email only). No X. [DECIDED] Replaces the Account screen.
5. Onboarding (3 steps): welcome and nickname, disciplines, reminder.
6. Home.

Later launches (with saved session): short splash (logo only, ~0.8 s) → Home.

Account routes:
- [DECIDED] Sign Up → Onboarding 1.
- [DECIDED] Start → "Ya tengo cuenta" (es) → Sign In (email) → Home.
- [DECIDED Nov 26] If on sign in `onboardingCompleted` is false or there is no profile (the user abandoned onboarding), Onboarding 1 opens.
- [DECIDED] Sign Up offers "¿Ya tienes cuenta? Inicia sesión" (es). Sign In offers "¿Olvidaste tu contraseña?" (es) and "¿No tienes cuenta? Crea una" (es). Reset sends a link by email.
- [DECIDED] Settings → Delete Account → confirmation with password (7b) → animation (7c) → Start.
- [DECIDED] Settings → Change Password (7d) → success sheet (7i) → Settings.

Screens:
- Splash: logo (tatami with glove), BRENZ, slogan "Entrena, crece tu experiencia." (es), footer "© 2026 Brenda Saavedra. Todos los derechos reservados." (es). That content is first-time only. After that, only the logo.
- Start: animated character and tatami, "Entrena. Yo opino." (es), Start button, I-have-an-account button, links to Terms and Privacy Policy.
- Intro (3): log classes, add water, fill your tatami. Segmented bar and a 9 s looping animation.
- Sign Up, Sign In, Reset Password.
- Onboarding (3): character welcome with nickname; disciplines with one default (multiple selection); optional reminder (notification permission only if turned on). Each step carries progress dots.
- Home (first screen on later launches; the character is always visible).
- Sheet: change discipline (floating, translucent, single selection).
- History.
- History: log sheet (6b), confirm delete (6c), class deleted with animation (6d) and History after delete (6e).
- Settings: Character (nickname), Training (disciplines, reminder, water goal), Account (Email, Change Password, Delete Account), More (Support, Rate the app, Sign Out). Footer: icon, version, Terms and conditions, Privacy Policy.
- Settings: confirm delete account (7b), account deleted with animation (7c) and wrong password (7e).
- Settings: change password (7d), errors (7g and 7h) and password updated with animation (7i).
- [REMOVED in v1] Apple variant of delete account (7f) and of Settings (7j). They return with Apple in v1.1.
- [OPTIONAL, Nov 26] Version lock screen: if the installed version is lower than `min_version`, it shows a button to the App Store. First cut of the schedule. No mockup.
- No evolution modal. No failure screen (the message goes in Home).

Home, top to bottom:
1. Discipline chip (remembers the last one; hidden if there is only one).
2. Character speech bubble, with an X to close it.
3. Large character. Below: nickname and stage in small text (e.g. "Intento · clase 4 de 10" (es); in stage 2: "Reincidente · 14 clases" (es)).
4. Today's board card: class tatami + bottle with + and − buttons.
5. Main button "Fui a clase" (es).
6. Floating tab bar: Train, History, Settings.

Interactions:
- "Fui a clase" (es): the character bounces and the day's element appears on a tatami tile. "Undo" 5 s.
- + (water): the bottle rises one glass with a bounce. The character bounces briefly. On reaching the goal, the character becomes animated that day.
- − (water): the bottle drops animated.
- [DECIDED] Water messages (Nov 26): each tap of + or − shows a temporary message (~4 s) in the bubble and then the state message returns. If the user closed the bubble with the X, these messages open it anyway. Priority per tap: goal reached, half, add. Detail in `docs/product/brenz-message-bank.md`.
- [PROPOSED] Bubble X: it hides it until the character state or the day changes. Water messages are the exception (above).

Board (tatami):
- [DECIDED] It is a single component. It is used in Home (today) and in History (chosen day, large). The 7-day thumbnail no longer exists.
- [DECIDED] The board is a catalog theme. V1 ships a single theme: tatami. Others (ring, holidays, store) are added later.
- [DECIDED] Isometric tatami, like Forest's tree board: slab with a 3-column grid. Drawn in SwiftUI. No new art.
- [DECIDED] Empty cells are visible. Each class marks a cell and the element stands on top.
- [DECIDED] Home: starts with 1 row. History: shows at least 3 rows. If a class is logged and there is no free cell, a row is added. Cap: 12 cells (4 rows). More than 12 classes shows "+N".
- [DECIDED] The size is calculated from the day's classes. It is not stored. If classes are deleted, the tatami goes back to the rows needed.
- Empty day: empty cells with an outline (equivalent to Forest's sprout sign).
- Colors: slab #D9A93B, side faces #A5791C and #86610F, marked cell #F0C95A.
- Home: fixed-height card. The slab shrinks as the tatami grows. The bottle stays on the right. The card has no border.
- Bottle: sports shape, drawn in SwiftUI with a light stroke. Water in #5CC8E8. Text "5/8" below.

History:
- Current week, Monday to Sunday, no arrows. Current day: lime border and bold number. Past day with training: checkmark in a lime circle. Past day without training: only the number. Future day: dimmed.
- Below: day label, large tatami and two cards (Classes and Water). Each card carries the total and a 3-hour block chart (6a, 9a, 12p, 3p, 6p, 9p).
- [PROPOSED] Tapping a past day shows its tatami and its cards. Future days are not tappable.

Log sheet (6b to 6e) [DECIDED, Nov 26]:
- 6b. Opens from the bottom on tapping a tatami cell in History. Same style as the discipline sheet: translucent, radius 40, blur, darkened background. The tab bar stays behind.
- Content of 6b: close X, title "Registro de clase" (es), the day's element in a cream circle, discipline and time of the log, discipline chips (single selection), lime "Guardar" (es) button and "Eliminar registro" (es) button.
- Only two actions: change the discipline and delete. Time and date are not edited.
- "Guardar" (es) stays inactive if the discipline did not change.
- 6c, confirm delete: the same sheet changes to "¿Eliminar esta clase?" (es) with the discipline and time, the notice "Se quita de tu tatami. Tu etapa no baja." (es), solid danger button "Eliminar" (es) and "Cancelar" (es) button (back to 6b).
- 6d, class deleted: on confirming, the same sheet shows an animation without closing. A lime checkmark that draws itself, a dotted ring that spins, the figure "0 clases hoy" (es) and a bar that empties (≈ 2.5 s) before closing the sheet. With "Reduce Motion" there is no animation.
- 6e, History after delete: it refreshes without the log. The tatami goes back to empty cells, the Classes card lowers the total and the character state is recalculated. The stage reached does not go down (section 4).
- Danger color: #FF6B5E. In 6b, text over the same color at 16 %. In 6c and 7b, solid button with text #0B0B0C. Border and text of a form error also in danger. Only for destructive actions and errors.
- [DECIDED] Home does not open this sheet (no long press). History only.

Delete Account (7b, 7c, 7e) [DECIDED, Nov 26]:
- 7b, confirm. Opens from Settings → Delete Account. Same sheet and same pattern as 6c, with Settings darkened behind. User icon with a minus sign in a danger circle, title "¿Eliminar tu cuenta?" (es), text "Lamentamos verte ir. Se borran tu progreso y tus clases. No se puede deshacer." (es), "Contraseña actual" (es) field with an eye, solid danger button "Eliminar cuenta" (es) and "Cancelar" (es) button. "Cancelar" (es) closes the sheet and leaves Settings unchanged.
- Firebase requires a recent session to delete. That is why the current password is requested in the same sheet. "Eliminar cuenta" (es) reauthenticates first. If correct, it deletes the Firestore data and then the Auth user, and moves to 7c. If not, it does not delete and shows the error in the field (7e: "Contraseña incorrecta. Inténtalo de nuevo." (es)). The button stays inactive with the field empty.
- [REMOVED in v1] 7f, Apple variant.
- 7c, account deleted. On confirming, the same sheet changes content without closing. Title "Cuenta eliminada" (es), a lime checkmark that draws itself with a dotted ring, figure "Hasta pronto" (es), text "Borramos tu progreso. Siempre puedes empezar de nuevo." (es), a lime bar that empties (≈ 2.5 s) and the caption "Volviendo al inicio" (es). With "Reduce Motion" there is no animation.
- When the bar finishes, the app opens Start. No session or local data remain.
- It reuses the translucent sheet and the animated checkmark component of 6d. No new components.

Change Password (7d, 7g, 7h, 7i) [DECIDED, Nov 26]:
- Available for every v1 account (all are email accounts).
- 7d, full screen in the Sign Up style. Title "Cambia tu contraseña" (es). Three fields with an eye: "Contraseña actual" (es), "Contraseña nueva" (es) (notice "Usa 8 caracteres o más." (es)) and "Repetir contraseña nueva" (es). Lime button "Actualizar contraseña" (es). No recovery link: whoever forgot it signs out and uses the Sign In recovery.
- The button enables when the three fields are filled, the new one has 8 characters or more and the repeat matches.
- 7g, wrong current: field with danger border and "La contraseña actual no es correcta." (es). 7h, repeat differs: "Las contraseñas no coinciden." (es)
- 7i, success: translucent sheet over darkened Settings, same as 7c. "Contraseña actualizada" (es), animated checkmark, "Listo" (es), "Úsala la próxima vez que inicies sesión." (es), a bar that empties and "Volviendo a Ajustes" (es). When it finishes, the sheet closes and Settings remains. With "Reduce Motion" there is no animation.

References (Mobbin):
- Forest Home (a single button, no board): https://mobbin.com/screens/fb6274de-c7ca-4981-b33a-d024d3945180
- Forest Day Summary: https://mobbin.com/screens/5df398b9-50b4-4beb-b813-ca09ab5e1d72
- Forest sign in (Apple or email, sign up, My account): https://mobbin.com/flows/b93b5509-4d3b-46aa-ace6-7710428adee0?tab=screens
- Waterllama Home (water as a gauge): https://mobbin.com/screens/e652b413-cdf9-4201-a1c4-c5dbc91e2cb8
- Ladder (Mobbin, dark style, buttons and bottom sheet): style reference, no saved link.
- Confirmation sheet with animation (Bren's screenshot, Nov 25): reference for 6d, 7c and 7i.
- Design the mockups without clothing or office scenery. The character is treated as an interchangeable image.

Nickname:
- [DECIDED] Character nickname. Maximum 10 characters. Optional. If empty, the nickname line is not shown.
- [DECIDED] User preference, independent of the chosen character. Only its owner sees it.
- [DECIDED] It is requested on the first onboarding screen. It is edited in Settings.
- [DECIDED] Character messages do not use the nickname (Nov 26). The nickname is never translated.

Board elements (option B):
- [DECIDED] 7 elements, one per day of the week, shared by the 4 disciplines.
- [DECIDED] Monday boxing glove. Tuesday hand wraps. Wednesday heavy bag. Thursday shin guards. Friday mouthguard. Saturday jump rope. Sunday pads (Nov 26: the round bell is out).
- [DECIDED] The day is taken from the local date of the class.
- The sports bottle is not a board element. It is the water indicator.
- Changing the discipline of a log does not change its element: the element depends on the day.

Interface style (Nov 26, replaces the retro cream style):
- [DECIDED] Dark mode only. Reference: Ladder (Mobbin).
- [DECIDED] Tokens: `background` #0B0B0C, `surface` #1B1B1E, `surfaceHigh` #26262A, `accent` lime #D7F500 (text on lime #0B0B0C), `water` #5CC8E8, `tatami` #D9A93B, `textPrimary` #FFFFFF, `textSecondary` white 68 %, `border` white 14 %, `characterCard` cream #EFE3C8, `danger` #FF6B5E.
- [DECIDED] Pill buttons, radius 28. Large titles in uppercase.
- [DECIDED] Font: San Francisco (SF Pro). Large title 32 pt Black. Section title (History, Settings) 17 pt, centered. Body 17 pt weight 500.
- [DECIDED] Floating translucent tab bar and sheet, with blur. Boards and speech bubbles without border.
- [DECIDED] The character sits on a cream card, because its dark line gets lost on black. If the final art has a light line, the card is removed.
- [DECIDED] Standard SwiftUI components when they exist. Exception: `BrenzTabBar`.

## 7. Architecture (closed Nov 26)

[DECIDED] 28 decisions, layer by layer, in four documents. Detail, constraints and acceptance criteria there:
- `docs/architecture/brenz-architecture-base.md`: platform, pattern, dependency injection, folders, packages, errors, tests, language and cut order.
- `docs/architecture/brenz-architecture-domain.md`: entities, pure rules, use cases, protocols, messages and catalogs.
- `docs/architecture/brenz-architecture-data.md`: Firestore, DTOs, security rules, Remote Config, authentication, offline writes, sign out, delete account, reminder and local rotation.
- `docs/architecture/brenz-architecture-presentation.md`: root screen, navigation, ViewModels, text in two languages, animations and components.
- Design system: `docs/product/brenz-design-system-decision.md`.
- Rules for Cursor: `.cursor/rules/brenz-rule-architecture-*.mdc`, `brenz-rule-messages.mdc`, `brenz-rule-design-system.mdc`, `brenz-rule-run-tools.mdc` and `brenz-rule-language-naming.mdc`.

Summary:
- [DECIDED] iOS 17. MVVM with `@Observable`. Own `AppContainer`, no libraries. A single app with folders by layer. Exception: `Packages/BrenzDesignSystem/`.
- [DECIDED] Packages: Firebase through SPM with `FirebaseAuth`, `FirebaseFirestore` and `FirebaseRemoteConfig`. No Analytics or Crashlytics.
- [DECIDED] Firestore: `users/{uid}`, `users/{uid}/classLogs/{id}` and `users/{uid}/waterLogs/{id}`. One document per class and one document per glass.
- [DECIDED] Security rules: everything denied by default. Only the owner accesses, with a session. Validations for discipline, element, time, nickname, water goal and stage.
- [DECIDED] Optimistic writes: the app does not wait for the server on classes and water. `DeleteAccount`, `ChangePassword` and `SignIn` do wait.
- [DECIDED] Authentication: email with password only (Firebase Auth), behind a protocol. The session survives reinstalling the app.
- [DECIDED] Sign out and delete account: order defined in `brenz-architecture-data.md`. Sign out can lose unsynced classes if there is no network. Accepted risk.
- [DECIDED] Message rotation: on the device (`UserDefaults`), not in Firestore. "Once" flags included.
- [DECIDED] Local reminder with `UNUserNotificationCenter`. Reconciled on app open.
- [DECIDED] Catalogs (`Character`, `BoardElement`, `BoardTheme`) as fixed data in code. V1: one character (`fox`), seven elements, one theme (`tatami`).
- [DECIDED] Asset name: `{character}_{stage}_{state}`, for example `fox_s1_normal`. No name mentions office.
- [DECIDED] Texts in `Localizable.xcstrings` (`en` base, `es`). Message keys added to the catalog with state "manual".
- [DECIDED] Tests with Swift Testing in Domain and ViewModels. Firestore rules with the emulator. No UI tests in v1.
- [DECIDED Nov 26] Documentation, file names, code and identifiers in English. No Spanglish.
- [REMOVED] Daily water document with counter. Rotation in Firestore. Native `TabView` (stays as a cut). "V1 `es` only".
- [PENDING, close in Spec Kit] Exact names of the stored enum values. Where the last used discipline is stored (proposal: `lastDiscipline` in `UserProfile`).
- Each layer behind a protocol (Clean Architecture). Firebase is a replaceable implementation.

## 8. Schedule (availability: 8–10 h per day; December Fridays minimum 5 h)

Build method: [DECIDED] Spec Kit in Cursor, as in the project "Aprendizaje IA para iOS con Cursor". The decision `.md` files are added to the context of that project. Cursor prompts are used only to define architecture, if needed.

[DECIDED Nov 26] Updated schedule. The architecture finished on Thursday the 26th, so Spec Kit (constitution and specify) moves to Friday the 27th. One build day is gained:

| Day | Work | Hours |
|---|---|---|
| Wed Nov 25 | Close logic. Placeholder character art finished | — |
| Thu Nov 26 | Mockups. Assets table. Message bank. Architecture (28 decisions), documents and `.mdc` rules. Re-estimate and schedule | 9+ |
| Fri Nov 27 | Create the project (Xcode, Firebase, Cursor) and copy the documents to the repo (1 h). Spec Kit: constitution and specify (7–8 h). If it overflows, 1 h moves to Saturday | 8–9 |
| Sat Nov 28 | Assets day: 7 elements in Miora, normalize, test on the tatami | 7–8 |
| Mon Nov 30 | Spec Kit: plan, tasks, analyze (max 2 passes) and adversarial review (~4 h). Implement day 1 (~5 h) | 9 |
| Tue Dec 1 | Review the English message bank (1 h). Implement day 2 | 9 |
| Wed Dec 2 | Implement day 3 | 9 |
| Thu Dec 3 | Implement day 4 | 9 |
| Fri Dec 4 | Implement day 5 (minimum 5 h) | 6 |
| Mon Dec 7 | Implement day 6. Upload the build to external TestFlight. Last day to decide cuts | 9 |
| Tue Dec 8 | Testing and polish (manual checklist, both languages) | 8 |
| Wed Dec 9 | TestFlight at the gym | — |
| Thu Dec 10 | Pilot adjustments. Store listing texts in both languages | 8 |
| Fri Dec 11 | Screenshots in both languages, privacy labels, URLs and final build | 6 |
| Sat Dec 12 | Submit for review (target) | 1–2 |
| Mon Dec 14 – Tue Dec 15 | Margin. The 15th is the last day | — |

Build re-estimate (Nov 26, Claude's assumptions):

| Block | Hours |
|---|---|
| Base, navigation and root screen | 2–3 |
| Domain and tests | 4–5 |
| Firestore, repositories and rules with tests | 4–5 |
| Auth (session, account, delete, cleanup) | 3–4 |
| Remote Config and `min_version` | 1 |
| Design system (tokens, components, `BrenzTabBar`, bottle, grid) | 5–7 |
| Auth, Start, intro, splash and Onboarding | 5–7 |
| Home | 5–6 |
| History and log sheet | 4–5 |
| Settings and reminder | 3–4 |
| English (catalog, keys, translation and review) | 4–6 |
| **Total** | **40–53** |

- Available from Nov 30 to Dec 7: 47 h (Mon 30: 5, Tue 1: 9, Wed 2: 9, Thu 3: 9, Fri 4: 6, Mon 7: 9). The range covers 40–53 h. The upper bound does not fit. That is why the cut order stays in force.
- If Saturday Nov 28 is not worked: the assets move to Monday the 30th and implementation day 1 disappears. 42 h remain available.
- External TestFlight: the first build needs Apple's beta review. If it is delayed, the pilot moves to Thursday the 10th.
- Cut order (proposed, see `brenz-architecture-base.md`): 0) lock screen for `min_version`. 1) Intro animations (a static fade version remains). 2) History hourly charts (totals remain). 3) Tatami growth (fixed at 2 rows). 4) Change password inside the app (only email recovery remains). 5) Animations for class deleted, account deleted and password updated (confirmations and refresh remain). 6) Change the discipline of a log (only delete remains). 7) `BrenzTabBar` replaced by native `TabView` (saves 2–3 h). 8) Last resort: ship in Spanish only (the catalog stays ready).
- [DECIDED] Last day to submit for review: Dec 15. If it does not arrive, scope is cut. The date does not move.
- [DECIDED] Publication: as soon as Apple approves, in mid-December.
- [PROPOSED] Configure the release to go live automatically on approval.
- Review: 90% in under 24 h according to Apple. In recent years Apple announced delays from Dec 20 to 26.
- [DECIDED] The developer account has published apps before. The app already exists in App Store Connect.
- Pilot metric: how many users come back on day 7. Comprehension test: ask 3 people what "Fui a clase" (es) does, without explaining it.
- Assumption: 8–10 h Monday to Friday.

## 9. Store checklist

- Verify the name "Brenz" in App Store Connect (the app is already registered).
- Do not use "Tamagotchi" anywhere in the listing. It is a registered trademark.
- Do not cite Zootopia, Sing, Daria or Clumsy Ninja in the listing.
- [DECIDED Nov 26] Listing in Spanish and English: name, subtitle, description, release notes and screenshots. Screenshots are taken with the simulator in each language.
- Prepare the privacy policy URL, terms URL and support URL, in Spanish and English. They are external web pages. Links to Terms and Policy go in Start, Sign Up and the Settings footer.
- Privacy labels: with Firebase there is data linked to the user (identifier, email, training logs with time, water logs with time, nickname). Declare them accurately.
- In-app account deletion (guideline 5.1.1(v)). Covered by 7b, 7c and 7e. Test the full flow before submitting.
- Sign in with Apple: out of v1. Guideline 4.8 does not apply while the app only uses its own login. If v1.1 adds Apple, enable the capability then.
- [RISK] Mandatory account. Guideline 5.1.1(v) asks that apps without account-based features can be used without signing up. Brenz justifies it with progress backup, changing iPhones and the gym pilot, but Apple may question it. Prepare a review note that explains it. Plan B: reactivate guest mode (X in Sign Up and Settings without account fields).
- [RISK] `min_version`: never raise it before Apple approves that version. Otherwise even the reviewer is locked out.
- "Rate the app": use the system review dialog.
- Demo account for review: create one with email and password and put it in the review notes. With a mandatory account it is essential. With it the reviewer can test Change Password and Delete Account.
- Age questionnaire.
- Rejection risk: minimal functionality (guideline 4.2). Pet, board, history, backup and reminders cover it.
- No HealthKit in v1.

## 10. Art direction

- [DECIDED] Character with an AI tool: Miora (https://miora.design).
- [DECIDED] The character is interchangeable. The product does not depend on its clothing or style.
- [DECIDED] Design direction: casual everyday clothing. When training, gloves, shorts and sportswear. Reason: so martial arts read clearly and more users identify.
- [PENDING] Final design of the v1 character (keep the current fox with other clothing, or redesign). Pending, not worked in this chat.
- [DECIDED] Current art (8 states + icon): placeholder to program against. Record and recipe in `docs/product/brenz-assets.md`. Delivered as a ZIP. It is replaced without touching code if the design changes.
- [DECIDED] Own traits of the current fox: round black glasses, V-shaped notch in the left ear, white boxing wraps on both paws. They are kept in a redesign unless Bren decides otherwise.
- [DECIDED] Bren's references: clumsy personality in the style of Clumsy Ninja + retro 90s cartoon aesthetic.
- Art style (character and elements): flat 2D, uniform dark line #2B2420 (token `line`, changeable), flat colors without gradients, a single flat shadow. The placeholder PNGs use #211A13. They are matched when the character is redesigned. The interface is modern and dark (section 6). Only the art carries 90s aesthetics.
- Base art palette: fox orange #C8642D, teal #3F7F7A, mustard #D9A93B, cream #EFE3C8. Clothing colors are defined with the new design.
- [DECIDED] States (without specific objects or clothing): animated (open eyes, crooked smile, fist up), normal (serious look), down (drooping shoulders, head lowered), lazy (leaning back, yawn).
- [DECIDED] Per character: 8 images (2 stages × 4 states) + app icon.
- [DECIDED] Board elements: 7 images shared by all characters and disciplines. Produced on the assets day (Sat Nov 28). Detail in `docs/product/brenz-assets.md`.
- [DECIDED] Sports bottle and tatami: drawn in code. They need no asset.
- [PENDING] App icon: the one in the mockups is provisional (lime square with tatami). The `brenz_icon` art uses the #3F7F7A background of the previous style. Decide whether to redo it.
- Character format: transparent PNG, 2048×2048, figure at 80% of the height, same canvas, same base pose and same feet line in all states. Needed for crossfades.
- Animation in code (native SwiftUI): breathing, bounce when logging a class or glass, element appearance on the tile, tatami growth, bottle fill and drain, crossfade between states, flash on stage change, class-deleted animation, account-deleted animation and password-updated animation (a single reusable success sheet view), looping intro and splash animation. Respect "Reduce Motion" with the package modifier. No generated animation.
- Prompts without brand or character names.
- Asset name: see `docs/architecture/brenz-architecture-domain.md`. The current files `brenz_s1_normal`, etc. are renamed when integrated.
- Edit prompt: KEEP line with the character traits, clothing, colors, line, canvas and position; CHANGE line with only the state change.

## 11. Flows to close

Reviewed on Nov 26. All closed.

1. [REMOVED in v1] Sign in with Apple. Returns in v1.1.
2. [DECIDED Nov 26] Mandatory account with email only. No guest mode. Settings exists only with a session. Sign Up replaces the Account screen. Review risk in section 9.
3. [DECIDED Nov 26] New versus existing user: with email only there is no ambiguity. Sign Up leads to onboarding. Sign In leads to Home. If `onboardingCompleted` is false on sign in, onboarding opens (`brenz-architecture-presentation.md`).
4. [DECIDED Nov 26] Edit or delete a logged class: log sheet from History only (6b to 6e). Home does not open it. No "+" in the tab bar.
5. [DECIDED Nov 26] Sign Out deletes local data. No data conflict between accounts.
6. [DECIDED Nov 26] Sign Out without alert: it signs out and returns to the first screen.
7. [DECIDED Nov 26] Splash: full and animated the first time. Only the logo (~0.8 s) later times. Start, intro and Sign Up only on the first launch. After signing out or deleting the account it returns to Start.
8. [PROPOSED] The exact time of a log is shown in the log sheet (6b). The chart bars stay in 3-hour blocks.
9. [DECIDED Nov 26] Support, Terms and Policy open an external web page.
10. [DECIDED Nov 26] Delete Account: confirmation with "Lamentamos verte ir" (es) (7b) and account-deleted animation (7c). When it ends, it returns to Start (section 6).
11. [DECIDED Nov 26] Reauthentication on delete account: the current password is requested in the same 7b sheet, with the error in 7e.
12. [DECIDED Nov 26] Change Password: screen 7d with errors 7g and 7h and success sheet 7i.

## 12. Pending items (in order)

1. [DONE Nov 26] Message bank: `docs/product/brenz-message-bank.md` (44 messages, triggers and rotation).
2. [DONE Nov 26] Flows of section 11. Sign Up replaces the Account screen (confirmed).
3. [DONE Nov 26] Architecture: 28 decisions in four documents (section 7).
4. [DONE Nov 26] Decision documents and `.mdc` rules generated in Claude's project. Still to copy them to the repo (`docs/architecture/`, `docs/product/`, `.cursor/rules/`) and to the context of the project "Aprendizaje IA para iOS con Cursor" (Fri Nov 27).
5. [DONE Nov 26] Re-estimate the build: 40–53 h against 47 h available (section 8).
6. Assets day (Sat Nov 28): 7 board elements in Miora, per `docs/product/brenz-assets-table.md`.
7. Spec Kit in the project "Aprendizaje IA para iOS con Cursor": constitution and specify (Fri Nov 27); plan, tasks, analyze (max 2 passes) and adversarial review (Mon Nov 30).
8. Prepare privacy policy, terms and support URLs (in Spanish and English), a review note about the mandatory account and a demo account (Thu Dec 10).
9. Final design of the v1 character (section 10) and of the icon. If it changes, redo 8 images + icon.
10. [DONE Nov 26] Stage names: "Intento", "Reincidente", "Irremediable" (es) (v1.1).
11. Clarify the name: "Brenz" today is the name of the fox and of the app. With several characters, decide whether "Brenz" stays as the app name and each character has its own.
12. [DONE Nov 26] Project instructions updated. The obsolete mockups brief was removed from the project.
13. Replace the fox placeholder PNGs in the mockups when the ZIP arrives.
14. [DONE Nov 26] `danger` token #FF6B5E in the design system and in the `.mdc` rule. Delete Account and Change Password screens done (7b to 7j).
15. [DONE Nov 26] Mockups adjusted to email only: no Apple, 7f and 7j removed, Sign Up replaces Account, short splash added.
16. Export the canvas PNGs to the repo (Spec Kit input).
17. Review `docs/product/brenz-assets-table.md` before Saturday the 28th.
18. Create the project: Xcode (iOS 17), Firebase project (Auth with email, Firestore, Remote Config), `GoogleService-Info.plist`, Cursor (Fri Nov 27).
19. Message bank and app texts in English: Claude draft and tone review by Bren (Tue Dec 1).
20. Close in Spec Kit: exact names of the stored enums and where the last used discipline lives (section 7).
21. [DONE Nov 26] Translate the remaining product documents to English and rename them: `brenz-mockups-v1.md`, `brenz-message-bank.md`, `brenz-assets.md` and `brenz-assets-table.md`.

Project instructions: updated on Nov 26 with the English file names and the language rule (all documentation, file names, code and identifiers in English, no Spanglish).

## Sources

- Finch Plus $39.99/year: https://habi.app/insights/finch-alternatives/
- Momentum (BJJ and MMA): https://apps.apple.com/app/id6763167542
- Marune: https://apps.apple.com/us/app/marune/id1405069801
- Review times 2026: https://www.choicely.com/tutorials/how-long-does-app-review-take
- First app waiting 8 days: https://developer.apple.com/forums/thread/842081
- Review during the holidays (2024): https://www.macrumors.com/2024/12/02/app-store-review-times-holidays/
- Forest Home (Mobbin): https://mobbin.com/flows/8f3128bb-79e0-4e0e-8b66-2d9ac61bae35?tab=screens
- Forest sign in (Mobbin): https://mobbin.com/flows/b93b5509-4d3b-46aa-ace6-7710428adee0?tab=screens
- San Francisco, system font (WWDC26): https://developer.apple.com/videos/play/wwdc2026/251/
