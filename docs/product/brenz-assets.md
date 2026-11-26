# Brenz — Art asset register

Date: 2026-11-26 (revision 4)
Use: status of each character image and the recipe to normalize them. Final PNGs do not live in this document. They go in the repo in `Assets.xcassets`.
Revision 4: translated to English. Asset names are English (`animated`, `down`, `lazy`, `icon`, `elem_<n>_<name>`). The delivered ZIP keeps its original file name.
Changes in revision 3: no art changes. Paths updated to the scope (`docs/product/brenz-scope-v1.md`) and a note that the day's element also appears in the History log sheet (34 pt).

## Version

- Current art: fox with office clothing. 8 states + icon, all approved. It is a placeholder to program against. Delivered in `brenz_assets_v1_oficina.zip` (original file name).
- The product does not depend on this design. The character is interchangeable and there will be several in the future (see `docs/product/brenz-scope-v1.md`, sections 1, 7 and 10).
- Redesign direction: casual everyday clothing; gloves, shorts and sportswear when training. Pending, no work in progress.
- The clothing descriptions below document only the current art. They are not product rules.
- The normalization recipe is reused unchanged with any character. Only the color seeds change.
- The interface is now dark. The character's dark line gets lost on black, so in the mockups it sits on a cream card #EFE3C8. If the character is redesigned, consider a light line and remove the card.

## File names

- Current: `brenz_s1_normal`, etc.
- Proposed for code: `{character}_{stage}_{state}`, for example `fox_s1_normal`. Stages 1 and 2 (3 in v1.1). States: normal, animated, down, lazy. Renamed on integration.

## Final format

- PNG with transparent background. 2048×2048.
- Figure at 80% of the master height, centered. Feet line at y = 1884 px.
- Same transform for all states and stages: scale 0.8938 and offset (246, 154) applied to the original 2048×2048 Miora canvas. The original stage 1 master has its box at x 231–1509, y 103–1936.
- Steps per image, in order:
  1. Remove loose specks (components under 5000 px).
  2. Align against the stage 1 master by the right paw of the image. It is measured as the last run of columns with pixels in the bottom 70 rows (base and center x). Do not use the full box: the tail can touch the floor. Do not use a fixed x cut: in lazy the left paw crosses the cut and skews the measurement.
  3. Do not rescale. The platform keeps the scale between states. The paw width measurement varies up to 5% by pose and is not reliable.
  4. Match colors to the stage 1 master by groups (fur, clothing, cream, accessories) with soft correction. In stage 2, the color of the new garment is compared with the normal image of that stage. Without that reference, the adjustment turned the garment gray.
  5. Unify the line color to #211A13.
  6. Apply the common transform.

## Asset list (current art)

| Asset | Status | Notes |
|---|---|---|
| `brenz_s1_normal` | Approved | Master P1c. Stage 1 reference. |
| `brenz_s1_animated` | Approved | Big eyes, smile, fist up (right side of the image), raised tail. More cream on the tail than the master (accepted). |
| `brenz_s1_down` | Approved | Third attempt. Head down, drooping ears, arms down, tail on the floor. |
| `brenz_s1_lazy` | Approved | Closed eyes, yawn, object in hand, long tail on the floor. Body shifted ~32 px to the left by the pose. |
| `brenz_s2_normal` | Approved | Stage 2 reference (extra garment over the shirt). |
| `brenz_s2_animated` | Approved | Fist raised on the left side of the image (in stage 1 it is on the right). Accepted: it changes by stage, not by state. |
| `brenz_s2_down` | Approved | Head down, drooping ears, arms down, tail on the floor. Ear notch as an open cut with a faint smudge; not visible at app size (accepted). |
| `brenz_s2_lazy` | Approved | Closed eyes, yawn, object with a light interior in hand, drooping tail. Body shifted ~90 px to the left when aligning by the right paw. |
| `brenz_icon` | Approved (to review) | 1024×1024 RGB, no alpha. Head with sweatband and towel around the neck on #3F7F7A. The guard-up fists option was dropped (garment over fur, cut-off fists). Miora watermark removed by cropping the bottom 6.5%. The mockups use a provisional lime icon. Decide whether to redo it for the dark interface. |

## Board elements (pending)

Decision: option B. 7 elements, one per day of the week, shared by all disciplines and characters. Each class of that day repeats the element. Usage detail in `docs/product/brenz-scope-v1.md`, section 6.

| Day | Proposed asset | Element | Status |
|---|---|---|---|
| Monday | `elem_1_glove` | Boxing glove | Pending |
| Tuesday | `elem_2_wraps` | Hand wraps | Pending |
| Wednesday | `elem_3_heavy_bag` | Heavy bag | Pending |
| Thursday | `elem_4_shin_guards` | Shin guards | Pending |
| Friday | `elem_5_mouthguard` | Mouthguard | Pending |
| Saturday | `elem_6_jump_rope` | Jump rope | Pending |
| Sunday | `elem_7_pads` | Pads | Pending |

- ISO numbering: 1 = Monday, 7 = Sunday.
- The round bell left the 7 elements (Nov 26).
- [PROPOSED] Format: transparent PNG, 1024×1024, object centered at 80% of the canvas. No pose alignment recipe: they are loose objects.
- [PROPOSED] Same style as the character: flat 2D, uniform dark line, flat colors, a single flat shadow. Base palette. No visible brands.
- [PROPOSED] All 7 must read at 38 pt (Home with 4 rows). Clear silhouette, no fine detail. Sizes in `docs/product/brenz-assets-table.md`. They are also seen at 34 pt in the History log sheet.
- Future (v1.1): elements per discipline with the name `elem_{discipline}_{n}_{name}`. The catalog already resolves them before the shared ones.
- While there is no art, the mockups and the first build use simple shapes as placeholders.
- The sports bottle is not an asset. It is drawn in SwiftUI.

## Review criteria per image

- Truly transparent background and no specks.
- The character's own traits present and consistent (on the fox: round glasses, notch in the left ear of the image, wraps on both paws).
- Clothing colors equal to those of the normal image of its stage.
- Pose coherent with the state. A raised fist does not fit in down or lazy.
- Same scale and same feet line as the master.
- Line thickness similar to the master. Reference: ~24–26% of the opaque pixels are line.
- No Miora watermark.

## Generation lessons

- Always use the stage's normal master as the reference. Using another state spreads poses (the fist of animated moved to down).
- Explicitly ask for the position of both arms in each state.
- For stage 2, the reference is the normal image of stage 2, not that of stage 1.
- Ask for hand objects to read as empty (light interior).
- An icon must communicate the app category (training), not the character's clothing.
- Miora images with a solid background carry a watermark at the bottom right. Crop or regenerate.
