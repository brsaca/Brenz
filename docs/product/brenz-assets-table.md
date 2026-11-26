# Brenz — Assets table: board elements

Date: 2026-11-26 (revision 5)
Source: mockups v1 (canvas "Brenz v1 — Mockups iPhone"). Use: assets day, Sat Nov 28, in Miora.
Revision 5: translated to English. Asset names are English.
Changes in revision 4: decisions on pads, color, format and canvas (below). No changes to the 7 elements or the sizes.

## Sizes in pt (side of the element inside the cell)

| Day | Asset | Home 1–2 rows (1–6 classes) | Home 3 rows (7–9) | Home 4 rows (10–12) | History (large tatami) | Log sheet | Min. readable | Result |
|---|---|---|---|---|---|---|---|---|
| Monday | `elem_1_glove` | 53 | 44 | 38 | 54 | 34 | 20 | Reads at all sizes |
| Tuesday | `elem_2_wraps` | 53 | 44 | 38 | 54 | 34 | 24 | Reads at all sizes |
| Wednesday | `elem_3_heavy_bag` | 53 | 44 | 38 | 54 | 34 | 20 | Reads at all sizes |
| Thursday | `elem_4_shin_guards` | 53 | 44 | 38 | 54 | 34 | 22 | Reads at all sizes |
| Friday | `elem_5_mouthguard` | 53 | 44 | 38 | 54 | 34 | 24 | Reads at all sizes |
| Saturday | `elem_6_jump_rope` | 53 | 44 | 38 | 54 | 34 | 26 | Reads at all sizes |
| Sunday | `elem_7_pads` | 53 | 44 | 38 | 54 | 34 | 22 | Reads at all sizes |

## Decisions (Nov 26)

- [DECIDED] Pads: a single pad. Simple round silhouette, with a white circle.
- [DECIDED] Color: no dominant mustard. The tatami slab is #D9A93B and the element would get lost. Use orange #C8642D, teal #3F7F7A, cream #EFE3C8, white and red. Mustard only in small details.
- [DECIDED] Format: transparent PNG 1024×1024, object at 80 % of the canvas, thick and uniform line. At 3x, 54 pt needs 162 px.
- [DECIDED] "States" canvas: Bren fixes the two outdated sections (History 361×292 pt and thumbnail).
- [PENDING] Mapping of day to `elementId`: Monday = 1 … Sunday = 7 (ISO style). Swift counts Sunday as 1. Close in architecture (Fri Nov 27).

## Rules

- Sizes: they come from the isometric tatami cell calculation in the mockup. Home: card 361×236 pt. History: card 361×340 pt.
- History uses 54 pt in all rows (the cell fits larger).
- The log sheet (6b) shows the class day's element on a cream circle #EFE3C8 of 56 pt. The element measures 34 pt. The element depends on the day, not on the chosen discipline.
- There is no longer a 7-day thumbnail. The 26 pt size was dropped.
- Minimum readability: estimated on placeholder shapes (recognizable silhouette, stroke ≥ 1.2 pt). Confirm with the real art.
- The element stands on its cell. The base of the object sits at the center of the cell.
- Jump rope: thicken the line or simplify the loop.
- Test all 7 on the #D9A93B slab and on the #F0C95A marked cell on Saturday the 28th.
