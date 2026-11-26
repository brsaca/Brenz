# Brenz — iPhone Mockups v1 (summary for the repo)

Source: https://claude.ai/artifact/UpgmAerVfchKrzyHe88zHs
Date: 2026-11-26 (revision 16)
If this file and `docs/product/brenz-scope-v1.md` differ, the scope wins.
Language: the mockups are in Spanish. Every quoted UI string in this file is the `es` copy. V1 ships in Spanish and English (scope, section 2). There are no English mockups. English strings are resolved in the String Catalog. Store screenshots are taken from the app in each language.
Revision 16: translated to English. Screen names follow the code names: Start, Intro, Sign Up, Sign In, Reset Password, Change Password, Delete Account, Train, History, Settings.
Images: export PNGs from the canvas and save them in the repo. There are no images in this project.

## Flow
1. Splash (first launch): full and animated. Splash (later launches): logo only (~0.8 s).
2. Start: "Empezar" or "Ya tengo cuenta".
3. "Ya tengo cuenta" → Sign In (email) → Home.
4. "Empezar" → Intro (3 screens) → Sign Up.
5. Sign Up (mandatory, no X, email only) → Onboarding 1.
6. Onboarding: welcome and nickname → disciplines → reminder.
7. Home, with tab bar: Train, History, Settings.
8. History → tap a tatami cell → log sheet (6b).
9. Log sheet → "Eliminar registro" → confirmation (6c) → "Eliminar" → class deleted with animation (6d) → History without that log (6e).
10. Log sheet → choose another discipline → "Guardar" → back to History.
11. Settings → "Borrar cuenta" → confirmation (7b, with current password) → "Eliminar cuenta" → account deleted with animation (7c) → Start. Wrong password: 7e.
12. Settings → "Cambiar contraseña" (7d) → "Actualizar contraseña" → success sheet with animation (7i) → Settings. Errors: 7g and 7h.

Later launches: Splash (later launches) → Home (the session stays saved).
There is no guest mode. Every user has an account. Settings always shows Email, Change Password and Delete Account.
Sign Out has no alert: it signs out and returns to the first screen (Splash → Start).
Home does not open the log sheet. No "+" button in the tab bar.

## Canvas status
Aligned with the scope (rev. 18): email only. No Apple in any text, button or arrow. The Account, 7f and 7j screens are removed. The numbering is not shifted. The canvas includes the "Access flow" diagram and the two splashes: "Splash (first launch)" and "Splash (later launches)".
No mockup of the version lock screen (`min_version`, optional).

## Status of each route
- Closed: Sign Up, Sign In with email, Reset Password, edit or delete a log from History (6b to 6e), Settings (7), Sign Out without alert, Delete Account with password, error and animation (7b, 7c, 7e), Change Password with errors and success animation (7d, 7g, 7h, 7i).
- [REMOVED in v1] Apple. Returns in v1.1.
- If on sign in `onboardingCompleted` is false or there is no profile, Onboarding 1 opens (architecture, presentation).

## Screens
- Splash (first launch): logo (tatami with glove), BRENZ, "Entrena, crece tu experiencia.", footer "© 2026 Brenda Saavedra. Todos los derechos reservados."
- Splash (later launches): only the logo centered on #0B0B0C. No text.
- Start: animated character and tatami, "Entrena. Yo opino.", buttons Empezar and Ya tengo cuenta, links to Terms and Privacy Policy.
- Intro (3): segmented bar, 9 s looping animation. No "Omitir". The "Continuar" button appears when the animation ends. The last one says "Empezar".
- Sign Up (follows the last Intro screen, with a back button), Reset Password: dark forms with a lime button. Sign Up carries legal links and "¿Ya tienes cuenta? Inicia sesión".
- Sign In: email, password, "¿Olvidaste tu contraseña?", lime button "Iniciar sesión", "¿No tienes cuenta? Crea una".
- Onboarding (3 steps): character speech bubble with X, optional nickname, disciplines (multiple selection), reminder (days, time).
- Home (4 states: animated, normal, down, lazy): discipline selector, speech bubble with X, character, tatami and water card with + and −, button "Fui a clase".
- Discipline sheet: floating, translucent, single selection.
- History (6): current week, current day with lime border, checkmark on past days with training, day board of 361 × 340 pt, Classes and Water cards with a 3-hour block chart.
- History: log sheet (6b): opens from the bottom on tapping a tatami cell. Darkened background and tab bar behind. Header with X and title "Registro de clase". Row with the day's element in a cream circle, the discipline and the time ("Hoy · 6:12 a. m."). Section "Cambiar disciplina" with 4 single-selection chips (the chosen one in white). Lime button "Guardar" and "Eliminar registro" button in the danger color.
- History: confirm delete (6c): same translucent sheet. Trash icon in a danger circle, "¿Eliminar esta clase?", "Box · hoy, 6:12 a. m. Se quita de tu tatami. Tu etapa no baja.", solid danger button "Eliminar" and translucent button "Cancelar".
- History: class deleted (6d): the same sheet changes content, without closing. Title "Clase eliminada", circle with a lime checkmark that draws itself and a dotted ring that spins, large figure "0 clases hoy", text "Se quitó Box · 6:12 a. m. del tatami." and a lime bar that empties (≈ 2.5 s) and closes the sheet. 4 s looping animation in the mockup. With "Reduce Motion" there is no animation.
- History: after delete (6e): History refreshed without the log. Empty tatami (3 rows with outline), Classes card at "0 clases hoy" and a chart without bars.
- Settings (7): Character, Training, Account (Email, Change Password, Delete Account), More (Support, Rate the app, Sign Out). Footer: icon, version, Terms, Privacy Policy. Support, Terms and Policy open an external web page.
- Settings: confirm delete account (7b): same sheet as 6c over Settings. User icon with a minus sign in a danger circle, "¿Eliminar tu cuenta?", "Lamentamos verte ir. Se borran tu progreso y tus clases. No se puede deshacer.", "Contraseña actual" field with an eye button (placeholder "Confirma con tu contraseña"), solid danger button "Eliminar cuenta" and "Cancelar" button.
- Settings: account deleted (7c): the same sheet changes content. Title "Cuenta eliminada", animated checkmark, "Hasta pronto", "Borramos tu progreso. Siempre puedes empezar de nuevo.", a lime bar that empties and the text "Volviendo al inicio". When the bar finishes, the app opens Start (screen 0b). 4 s looping animation in the mockup.
- Settings: change password (7d): full screen, same style as Sign Up. Back button. Title "Cambia tu contraseña" and text "Escribe la actual y elige una nueva." Three fields with an eye button: "Contraseña actual", "Contraseña nueva" (notice "Usa 8 caracteres o más.") and "Repetir contraseña nueva". Lime button "Actualizar contraseña". No recovery link.
- 7e, delete account with a wrong password: the 7b sheet with the field in a danger border, the password typed and the message "Contraseña incorrecta. Inténtalo de nuevo."
- 7g, change password with a wrong current one: "Contraseña actual" field in a danger border and the message "La contraseña actual no es correcta."
- 7h, change password with a different repeat: "Repetir contraseña nueva" field in a danger border and the message "Las contraseñas no coinciden."
- 7i, password updated: translucent sheet over darkened Settings, same as 7c. Title "Contraseña actualizada", lime checkmark that draws itself with a dotted ring, "Listo", "Úsala la próxima vez que inicies sesión.", a lime bar that empties and the text "Volviendo a Ajustes". When the bar finishes, the sheet closes and Settings remains. 4 s looping animation in the mockup. With "Reduce Motion" there is no animation.

## Log sheet: rules
- It allows only two actions: change the discipline and delete. Time and date are not edited.
- "Guardar" updates the discipline. Without a discipline change, "Guardar" stays inactive.
- "Eliminar registro" opens the confirmation (6c). "Eliminar" removes the tatami cell, recalculates the character state and shows the animation (6d). It does not lower the stage reached.
- "Cancelar" in 6c goes back to the log sheet.
- It applies to logs of today and of past days that are visible in History.
- Danger color: #FF6B5E. Text on a background of the same color at 16 % in sheet 6b. Solid button with text #0B0B0C in confirmations 6c and 7b. Border and text of a form error in danger. It is used only in destructive actions and errors.

## Delete Account: rules
- The confirmation (7b) uses the same sheet and the same pattern as 6c. "Cancelar" closes the sheet and leaves Settings as it was.
- Firebase requires a recent session to delete. That is why 7b asks for the current password in the same sheet, without a separate screen.
- "Eliminar cuenta" first reauthenticates with that password. If it is correct, it deletes the Firestore data, then the Auth user, and moves to 7c. If it is wrong, it deletes nothing and shows the error in the field (7e).
- "Eliminar cuenta" stays inactive while the field is empty.
- When the animation (7c) ends, the app returns to Start. There is no session or local data.

## Change Password: rules
- Available for every v1 account (all are email accounts).
- Firebase asks to reauthenticate with the current password and then updates the new one.
- "Actualizar contraseña" is enabled when the three fields have text, the new one has 8 characters or more and the repeat matches.
- Errors: wrong current password (7g) and different repeat (7h). The "new equals current" error uses the same style, without a drawn mockup.
- The screen does not offer password recovery. Whoever forgot it signs out and uses "¿Olvidaste tu contraseña?" in Sign In.
- On update, the success sheet (7i) covers Settings with the same animation as 7c and 6d, and closes by itself.

## Visual system
- Background #0B0B0C. Surface #1B1B1E. Surface high #26262A. Lime accent #D7F500 (text on lime #0B0B0C). Water #5CC8E8. Tatami #D9A93B (faces #A5791C and #86610F, marked cell #F0C95A). Character cream card #EFE3C8. Danger #FF6B5E.
- SF Pro. Large title 32 pt Black uppercase. Section title 17 pt medium weight, centered. Primary button 16 pt Heavy uppercase. Body 17 pt weight 500.
- Radii: pill 28, boards and cards 24, Settings lists 20, speech bubble 28, sheet 40, tab bar 32. Screen margin 16 pt.
- Tab bar: floating translucent capsule with blur (own component `BrenzTabBar`). Active tab in white, no color. Content passes behind it.
- Boards and speech bubbles without border.
- Only the character carries 90s aesthetics.
- Token detail in `docs/product/brenz-design-system-decision.md`.

## Tatami elements (one per day)
Monday glove, Tuesday wraps, Wednesday heavy bag, Thursday shin guards, Friday mouthguard, Saturday jump rope, Sunday pads.

## Pending
- Final app icon (provisional today).
- Final character art (PNG). Today there are placeholders.
- Mockup of the version lock screen, if it is not cut.
