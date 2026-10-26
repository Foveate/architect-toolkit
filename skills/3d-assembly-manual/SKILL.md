---
name: 3d-assembly-manual
description: Build a phone-first interactive 3D step-by-step guide (LEGO-manual style) in the 3D diagram look (white model with ink edges, one red for what each step changes, paper and grid, Archivo / Instrument Sans / JetBrains Mono chrome) as a single self-contained Three.js HTML file. Use when the user asks for a 3D build guide, install sequence, assembly/instruction manual, or "step-by-step 3D" for a physical task (garden, house, furniture, repair).
---

# 3D assembly manual

Proven on a garden-bed build, then generalised after kitchen and sauna test builds. Restyled 2026-10 after the Shanghai Grand Opera House diagram: the same white-clay drawing as the 3D walkthroughs, with that diagram's chrome. **Copy `starter.html`**: a project-agnostic engine with a four-step shelf example that renders on its own. Edit only the `CONFIG` block and the `PROJECT SCENE` block; leave the engine alone.

## CONFIG keys (top of the script)
| Key | What |
|---|---|
| `title`, `subtitle`, `date` | Header text; `date` prefixes the page counter. |
| `storageKey` | localStorage key for the last page; unique per manual. |
| `accent` / `fill` / `fillInk` | The one colour. `accent` (#d7261e) is flat red for arrows, the plan frame, the outline flash and the big step numbers. `fill` (#e2352c) shades this step's parts and `fillInk` (#8e1510) inks them. Keep the defaults unless the user asks for another hue. The ink must stay darker than the fill. |
| `view` | Default view direction vector. |
| `plan` | Plan inset `{c:[x,z], r}`; `null` derives it from the scene's Box3 (the shadow frame and the paper ground always do). |
| `figure` | Off (`null`) by default; opt in only when asked, with `[x, z, rotY?]`: a flat cut-out that faces the view unless `rotY` is given. Frame the overview preset to take in its raised hand, 7.4 ft up. |
| `CAM` | Presets `{c, r, v?, walls?, cut?}`. |
| `NAMES`, `INVENTORY` | Part labels keyed like `P`; inventory groups `[[group, [[part, qty]]]]`. |
| `STEPS` | Pages. Copy keys: `title text acts meta time tip parts dims notes inventory calendar`, plus `word`, one word for the big stage caption ("Base", "Shelves"). `meta` is `'a · b · c'`, and its first item is set in dark red. Other keys: `detail` `{p, r, hide?:[regKeys], view?}` (the detail card's own pass: hide parts in front, look from another direction), `view` (direction), `walls` `{h, sides?}` (walls on those sides drop to `h` ft; omit sides = all), `cut:[regKeys]` (fade to 0.12 for a cutaway), `onEnter(t)` (special animation; `t` has `lastLand, objs, tween, ease, easeIO, setOpacity, setColor, paint, T, still`; return a new end time to delay marks). Step keys override the preset. |

PROJECT SCENE block: context geometry (auto-painted white), `wall(side, x0, x1, y0, y1, z0, z1)` for rooms, `P.*` part builders, optional `THUMB_ONLY[key]` builders (a thicker thumbnail-only variant for thin parts: bands, wires), `reg()` calls. Groups passed to `reg()` are re-centred automatically, so parts can be built in absolute coordinates. Build with the helpers (`box`, `cyl`, `sphere`, `tube`, `lines`), which dress every mesh in the look. Options are `{edge: false | 'soft', glass, opacity, noShadow}`: `'soft'` is grey ink for context, and `glass` is the glazing material. Don't model a floor: the engine lays the paper ground and grid under the finished assembly.

## Look (the 3D diagram)
| | Value |
|---|---|
| Paper | `#f7f7f5`: background, ground and fog |
| Model | `#ffffff` `MeshLambertMaterial`, double-sided, `polygonOffset` 1/1 so the ink wins |
| Ink | `#242527`, opaque, `EdgesGeometry` at 20°; soft ink `#8d8e8b` for context and walls |
| Glass | `#dfe4e6` at 0.5 |
| This step | `fill` shaded, edges in `fillInk`; arrows flat `accent`, unlit |
| Light | ambient 0.7 + hemisphere `#ffffff` / `#e6e6e0` 0.2 + sun 0.42 from (−150, 260, 190) |
| Shadow | one PCF-soft map, 2048², bias −0.0004, fitted to the finished assembly at any turn |
| Ground | paper Lambert, 20 radii wide, just under the lowest point, taking the shadow |
| Grid | `GridHelper`, 40 cells of 1/2/5 × 10ⁿ ≥ r/6, `#e6e6e0` |
| Fog | paper, from d + 0.75 r to d + 3.5 r, where d is the distance from the camera to the centre; it follows the camera, so the model itself never greys out |

- three r128 draws these values as written (legacy light units, linear output, no colour management). A face toward the sun clips to white, a face turned away reads 227, the ground near the model is white and fades into the paper.
- Already-built parts and all context are plain white with ink. Only what this step adds or removes turns red. The parts and done pages stay white, except `only` parts that belong to them. Part thumbnails are drawn in red.
- Round parts are faceted and inked on every facet, like a drawn model. `cyl` defaults to 12 sides and `sphere` is an icosahedron; `tube` has no ink.
- Walls take shadow but cast none: a room's walls would lay big wedges of shade across the drawing.
- No people by default: every figure tried so far (cylinder people, then a flat Modulor cut-out) read as weird. Scale comes from the dimension lines and the live scale bar. `modulorFigure()` stays in the starter only for an explicit request.
- Light only (`color-scheme: light`), like the diagram.

## Chrome (the diagram's)
- **Type:** Instrument Sans for text, Archivo 800 for the big red step numbers and the uppercase titles, and JetBrains Mono, uppercase and tracked, for labels, counters, meta, dimensions and the timer.
- **Tokens:** paper `#f7f7f5`, card `#ffffff`, ink `#151617`, ink-2 `#46484b`, muted `#8a8c8f`, rule `#e3e3df`, accent and accent-ink from CONFIG.
- **Layout:**
  - Phone: stage on top (60%, at least 300 px), panel below.
  - From 960 px: stage left, a 420 px panel right.
- **Stage:**
  - The header sits over the drawing: title, subtitle, a mono "date · n of N", and a rule under it.
  - Parts callout card top-left; plan card and Replay chip top-right.
  - The caption bottom-left: big red number, `word`, and a mono meta line (hidden on phones).
  - Bracket scale bar, a mono "Drag to turn" hint, and a 2 px red progress bar along the bottom.
- **Panel:**
  - Step dots (ink done, red current, tap to jump), then a mono tab line ("Step 2 of 4 · ≈5 min").
  - Red number with the uppercase title and the lede; acts with ink number discs.
  - The mono meta line, and the tip on paper with a 2 px red rule.
  - Inventory as square cards; calendar rows on rules.
  - Footer: an ink timer (time since the first step), Back, and an ink primary Next.
- **Shapes:** square corners and 1 px rules everywhere. Only number discs and the chip's dot are round. Focus is a 2 px accent outline.

## Engine (in starter.html)
- `reg(key, obj, {add, remove, only:[pages], from, lag, each, span, part, grow, noGhost, noArrow})`; a page's scene is the cumulative state; `only` parts fade when their page turns. `tint(obj, on)` paints a part red or white, ink included.
- Orthographic camera; preset `{c, r, v}` tweened 900 ms ease-in-out, walls and cutaways tweened with it. Every tween's clock starts on its first rendered frame, so a slow first frame can't eat the opening motion. Landings 700 ms ease-out along an arrow, ghost outline at destination, accelerating stagger `span*sqrt(i/(n-1))`.
- Per step:
  - Parts callout with 3D thumbnails from a second renderer (`toDataURL`). Below 760 px it is one scrolling row of 32 px thumbs, with ellipsised names and a right-edge fade when it overflows.
  - Numbered action marks with screen offsets, clamped below the callout; the leader is redrawn from the clamped position to its point.
  - Detail card: a square card like the plan. It scales in from its centre, the GL pass draws into its view box (`glRect`), and a leader runs from its edge to a ring on the point. A circular mask can't work here, because the ground and grid show through the corners of the mask.
  - Dimension lines, the live scale bar, the plan inset, the step number, the time and one tip.
- Tap a part = blink + outline flash. Swipe the panel (pointer events, `touch-action:pan-y`). `#stepN` deep-links, `#stepN&still` freezes for captures.

## Process
1. Measure or estimate the real space from photos (mark "verify"); list parts grouped by where they come from.
2. Write STEPS copy: plain instructions, 2-4 numbered acts, a one-word `word`, quantities in the meta line.
3. Build, then verify at 375 px and at 960 px or wider in a browser over a localhost server (e.g. `python3 -m http.server`; keep the viewport meta). Capture with headless Chrome through a 390x844 iframe harness at `#stepN&still`.
4. Deliver one `.html` file. Three.js r128 loads from cdnjs and the fonts from Google Fonts; everything else is inline, so it opens by double-click or on any static host. Until you have a capture from the browser the reader will use, call the render unverified.

## Gotchas
- The look depends on r128's legacy light units. On three r155 or later, multiply every intensity by π, turn colour management off and output linear, or the drawing comes out dark and grey.
- A pale `fill` loses the step: the sunlit top clips to white. Keep the fill saturated and `fillInk` darker than it, so the ink still reads on red.
- Without `polygonOffset` on surfaces, the ink z-fights its own faces.
- Keep the pulse base colour under its own userData key (opacity code owns `userData.base`).
- Emissive glow is invisible on white or red parts; blink opacity instead.
- Touch-only handlers can't be tested with mouse-driven automation; use pointer events.
- Rooms: work on opposite walls (sink run vs nook) needs two presets with different `v`, each cutting the walls between camera and work (learned on a kitchen build).
- Numbers derived from another file (plank rows, rips): port its calculation functions instead of retyping results.
- Captures: run headless Chrome one at a time, each under `perl -e 'alarm 50; exec @ARGV' <chrome --headless=new --user-data-dir=<scratch>/st-N --disable-gpu --use-angle=swiftshader --enable-unsafe-swiftshader --window-size=520,880 --timeout=15000 --screenshot=...>` (macOS has no `timeout`; Chrome does not exit on its own), then `pkill -f user-data-dir=<scratch>/st-N`. Parallel runs give black canvases.

## Sources
- Video with no transcript (music-only): read the burned-in captions from frames. Draw the paused video to a canvas at intervals, lay the frames out as a contact sheet on the page, screenshot it. Seeking in a background tab renders black.
