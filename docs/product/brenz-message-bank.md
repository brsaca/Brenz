# Brenz — Message bank v1

Date: 2026-11-26 (revision 3)
Status: closed by Bren in Spanish. English version pending (Claude draft, tone review by Bren, Tue Dec 1). No code.
Language: the messages below are the `es` copy. Everything else in this document is English. Each message gets an English (`en`) string with the same key.
Replaces in `docs/product/brenz-scope-v1.md`:
- Section 4: stage names (see below).
- Section 12: pending items 1, 2, 10 and 15 are closed.
If this file and the scope differ on messages or stages, this file wins.
Revision 3: translated to English. The stage placeholder is now `{stage}` (was the Spanish placeholder). Message groups use the English state names: animated, normal, down, lazy.
Changes in revision 2 (architecture, Nov 26): v1 ships in Spanish and English. Rotation lives on the device. The `{stage}` marker is written `%1$@` in the catalog. See `docs/architecture/brenz-architecture-domain.md` (Messages section) and `.cursor/rules/brenz-rule-messages.mdc`.

## Stages (visible name)
- [DECIDED] Stage 1: "Intento" (es). Stage 2: "Reincidente" (es). Stage 3 (v1.1): "Irremediable" (es).
- Numeric internal key (1, 2, 3). The visible name is separate text (key `stage.1.name`).
- Messages use `{stage}` in this document. In the catalog it is `%1$@`. The stage name enters as an argument. It is not written by hand.

## Voice rules
- First person. No nickname. No nickname placeholder.
- Sarcasm at the action, not at the person.
- No work, office or clothing.
- All texts go in the String Catalog, in `es` and `en`. Keys: `message.<group>.<n>`.

## Logic rules
- State message: chosen by the character's current state.
- State rotation: no repeats until the state bank is exhausted. Used indexes are stored, on the device. The same message lasts the whole day.
- Water rotation: only avoids repeating the previous message of the same group.
- Priority per water tap: goal reached, then half, then add one glass.
- Animated only because of water (base state normal): the bubble shows the goal-reached message. It does not show the animated-by-classes messages.
- Normal messages marked (T): they appear only if no class is logged today. With a class today, the other 3 are used.
- Half of the goal: calculated with the user's goal (4 to 16 glasses). Rounded up.
- "Dropped below the goal": it appears only if the state really went down (animated by water returns to normal). If animated comes from classes, "One glass less" appears.
- Water messages: temporary, about 4 s, in the bubble. Then the state message returns.
- Bubble closed with X: water messages open it anyway. They are a direct response to a tap.
- "No glasses at midday": on opening the app after 12:00 local, with 0 glasses. Once per day. No notification.
- "New user": only with no logged classes and only once.

## State messages (5 per state)

### Animated
1. Otra clase. Qué sorpresa. Mi escepticismo está en crisis.
2. Entrenas seguido. Me quedé sin quejas. Qué incómodo.
3. Constancia. Qué palabra tan rara. Te queda bien.
4. El tatami empieza a llenarse. Las estadísticas lo notaron.
5. Sigue así. Lo digo sin sarcasmo. Casi.

### Normal
1. Todo en orden. Aquí eso casi es una celebración.
2. Hoy hay clase o no hay. Yo solo observo.
3. Un día normal. Con suerte, con guantes.
4. (T) La clase no se registra sola. Solo lo menciono.
5. (T) Miro el tatami vacío con mucho interés.

### Down
1. Van días sin clase. El tatami ya lo notó. Con una clase corta basta.
2. Pasaron varios días sin entrenar. Pasa. Elige una clase fácil y llega.
3. El costal lleva días sin visitas. Se hizo a la idea. Una clase lo contradice.
4. No hubo clase esta semana. Lo anoto sin drama. Mañana, solo una.
5. Hace días que no te veo. Mi cara seria aguanta. Una clase y seguimos.

### Lazy
1. Más de una semana sin clase. Yo también me rendí al sillón. Hoy, una sola.
2. Ya pasó una semana. Soy decoración. Una clase me despierta.
3. Bostezo por los dos. Una clase cambia eso.
4. El costal sigue colgado y yo sigo bostezando. Una clase nos mueve a los dos.
5. Pasó una semana. Nadie lleva la cuenta. Bueno, yo sí. Una clase y la borro.

## Event messages

### Return after lazy
Shown once: the first class logged with the state at lazy.
1. Volviste. No pregunto nada. Anotado: una clase.

### New user (first Home)
1. Hola. Soy tu mascota. Hoy el tatami está vacío.
2. Empezamos de cero. Registra tu primera clase cuando quieras.
3. Aquí estoy. Observo. Cuando entrenes, lo anoto.

### Stage change (class 10)
No modal. Message in the Home bubble.
1. Diez clases. Ahora eres {stage}. Es un elogio.
2. Etapa nueva: {stage}. Volviste tantas veces que ya es costumbre.

## Water messages

### Add a glass (5)
1. Un vaso más. Qué manera tan discreta de ser constante.
2. Agua. Sin drama. Así me gusta.
3. Otro vaso. No me hagas decirlo dos veces.
4. Hidratación registrada. Mi emoción es real, aunque no lo parezca.
5. Un vaso. Paso pequeño. Cuenta igual.

### Half of the goal (3)
1. Mitad de la meta. La botella ya parece un plan.
2. Medio camino. El resto es repetir lo mismo.
3. Vas por la mitad. Sigo sin quejas.

### Goal reached (4)
The character becomes animated that day only if its base state is normal.
1. Meta cumplida. Hoy me permito un poco de entusiasmo.
2. Todos los vasos. Hasta la botella se siente vista.
3. Meta cumplida. Qué logro tan líquido.
4. Lo lograste. Sin aplausos, pero con respeto.

### Remove a glass (3)
1. Un vaso menos. Corrección anotada. Sin juicios.
2. Botón equivocado. Pasa. Ya lo quité.
3. Un vaso menos. La botella lo entiende.

### Remove and drop below the goal (2)
Only if today's state goes back from animated to normal.
1. Bajaste de la meta. Vuelvo a normal. Un vaso lo arregla.
2. Meta deshecha. No pasa nada. Un vaso y regresa.

### No glasses at midday (1)
1. Cero vasos hasta ahora. La botella se aburre. Empieza con uno.

## Total
- State: 20. Return: 1. New user: 3. Stage change: 2. Water: 18.
- Total: 44 messages.

## English version (pending)
- Claude draft of the 44 messages and of the app texts. Tone review by Bren (1 h, Tue Dec 1).
- Same tone: dry, observant, affection disguised as sarcasm. The sarcasm aims at the action.
- Same logic rules and the same keys. Only the text changes.
- `{stage}` is also `%1$@` in English.

## Acceptance criteria
- No message mentions work, office or clothing.
- No stage text writes the name by hand: it uses `{stage}` (`%1$@` in the catalog).
- Every message exists in `es` and `en`, with the same key.
- With the water goal at 4 or at 16, "half" and "goal reached" fire on the correct glass.
- With a class logged today, the (T) messages do not appear.
- With animated by classes, removing below the goal does not show "Vuelvo a normal".
