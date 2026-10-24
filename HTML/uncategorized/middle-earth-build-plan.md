<!--
Type: Phased implementation plan
Target: Three.js, OrbitControls, vanilla JavaScript
Output: middle-earth-journey.html
Related: middle-earth-journey.md
-->

# Middle-earth — A Journey Through Legend

**Agent execution playbook | Qwen 27B Q3 | 10 gated passes**

**Final artifact:** `dist/middle-earth-journey.html`  
**Cinematic duration:** exactly 227 seconds (3:47)  
**Nominal capture rate:** 60 FPS; frames `0..13619` represent times `frame / 60`, with `seekTime(227)` available as the endpoint.

## How to use this file

This document is the single source of truth for a coding agent working in this repository.

1. Read this document and inspect the repository before editing.
2. Find the first pass whose status is not `PASS` in the [progress tracker](#progress-tracker).
3. Complete **only that pass**. Do not jump ahead or silently skip requirements.
4. Run the pass's acceptance checks and the [quality gate](#quality-gate-after-every-pass).
5. Record a concise result in [execution log](#execution-log), updating pass status to `PASS` only if verified.
6. Commit the accepted pass to Git if Git is available; then proceed to the next pass when executing this playbook end-to-end. If blocked, record the blocker and stop rather than claiming success.
7. After Pass 10, provide the final HTML file and a factual test report.

**Autonomous operation:** Execute all ten passes sequentially, unless the user or agent host explicitly requires approval between stages. A failing gate must be repaired and rerun before advancing. Do not claim tests, screenshots, or performance figures that were not actually obtained.

## Non-negotiable project requirements

- Build a handcrafted miniature, procedurally generated Middle-earth-inspired diorama using **Three.js** and vanilla JavaScript.
- All 24 listed landmarks must exist and be **recognizably distinct**, not merely 24 labels or generic towers.
- Produce coherent geography, biomes, landmarks, animated water, volcano and magical effects, and a cinematic day/night cycle.
- Implement a scripted **227-second camera journey**, free-camera mode, and deterministic `seekTime(seconds)` and `seekFrame(frameIndex)`.
- All world generation uses an explicit seed. All animation derives from authoritative cinematic time; no free-running clocks or uncontrolled randomness.
- Develop with modules, but export **one HTML document** embedding all authored CSS and JavaScript. The only permitted runtime network dependencies are **pinned, mutually compatible Three.js and OrbitControls**; no external models, images, fonts, textures, or audio.
- Support responsive resizing, keyboard-accessible controls, appropriate sRGB output, and clear error states.
- Target approximately 60 FPS on capable desktop hardware; prioritize correctness and clear profiling over unverified performance claims.
- No placeholder geometry for completed landmarks, TODO implementations, omitted functions, fake tests, or fabricated screenshots.

## Recommended project layout

```text
middle-earth-journey/
├── middle-earth-build-plan.md   # this file
├── index.html                   # development entry
├── src/
│   ├── main.js
│   ├── world.js
│   ├── terrain.js
│   ├── landmarks.js
│   ├── architecture.js
│   ├── vegetation.js
│   ├── effects.js
│   ├── cinematic.js
│   └── ui.js
├── tests/
├── scripts/
└── dist/
    └── middle-earth-journey.html
```

The final single-file export may be produced using a **development-time bundler** (such as esbuild). Properly bundle local ES modules; do not concatenate files with unresolved local imports. The final HTML may load pinned Three.js modules via CDN; it must not rely on other source files or runtime build steps. If offline execution is explicitly required later, vendor Three.js within the bundle instead.

## World conventions and architecture contracts

- **Coordinates:** X increases west → east (`-100..100`); Z increases north → south (`-100..100`); Y is height.
- **Terrain:** `getTerrainHeight(x, z)` is canonical for terrain generation and landmark placement. All geometry must share the same coordinate frame.
- **Registry:** Exactly 24 unique entries with `id`, `name`, `region`, `x`, `z`, `scale`, `archetype`, and `focusHeight`.
- **Randomness:** Central seeded PRNG and deterministic noise. Avoid unseeded `Math.random()` in both initialization and rendering.
- **Animation:** A single `cinematicTime` drives `updateSceneAtTime(t)` and all environmental effects. Seeking must reconstruct state without replaying prior frames.
- **Camera:** Cinematic control and OrbitControls must never write to the camera in the same update. Free-camera mode is separate.
- **Performance:** Share geometry/materials, instance repeated vegetation, bound particle counts, limit shadow casters, and avoid per-frame geometry rebuilding and avoidable allocations.
- **Three.js compatibility:** Pin one Three.js release for core and OrbitControls; use its actual color management and shadow APIs (e.g. `outputColorSpace` in modern releases, rather than deprecated `outputEncoding`).

## Geographic reference

| Region | Expected features |
|---|---|
| Northwest | Grey Havens, Shire/Hobbiton, Bree, Weathertop; gentle plains and coastal terrain |
| North-central | Rivendell, Misty Mountains, Moria; steep mountains and passes |
| Central | Lothlórien, Fangorn, Isengard, Argonath; woods and the Great River |
| West-central / South | Amon Hen, Edoras, Helm's Deep; Rohan grassland and mountain gorges |
| Southern / Southeastern | Gondor, Minas Tirith, Osgiliath, Minas Morgul, and Mordor approaches |
| East / Southeast | Black Gate, Mount Doom, Barad-dûr, Dead Marshes, Cirith Ungol |
| Northeast | Mirkwood/Dol Guldur, Erebor, Dale, and eastern ridges |

Geography is compressed for a museum miniature, but recognizable relative placement must be preserved. The River Anduin must connect relevant central and southern scenes coherently.

## Required landmark registry

| # | Landmark | Essential recognizable geometry |
|---:|---|---|
| 1 | The Grey Havens | Pale stone coast, docks, arches, harbor buildings |
| 2 | Hobbiton | Grassy hillocks, round hobbit doors, chimneys, gardens |
| 3 | Bree & The Prancing Pony | Timber-framed village, pitched roofs, large inn |
| 4 | Weathertop | Rocky hill, broken circular ruins and columns |
| 5 | Rivendell | Pale elegant terraces, pavilions, waterfalls, autumn trees |
| 6 | Moria West Gate | Sheer cliff, dark arched portal, carved stonework |
| 7 | Lothlórien | Silver trees, gold foliage, elevated talan platforms |
| 8 | The Argonath | Twin monumental humanoid statues flanking a river passage |
| 9 | Amon Hen | Summit dais and ruined Seat of Seeing |
| 10 | Fangorn Forest | Dense dark old-growth trees, twisted trunks and canopy |
| 11 | Isengard | Ring wall and four-spined black Orthanc tower |
| 12 | Helm's Deep | Mountain gorge, curved defensive wall, Hornburg |
| 13 | Edoras | Green hill, golden-roofed Meduseld, small village |
| 14 | Minas Tirith | **Seven** tiered white ramparts, citadel, rocky prow |
| 15 | Osgiliath | Ruined river city and shattered bridges |
| 16 | Minas Morgul | Pale fortress, high spires, spectral green crown |
| 17 | The Black Gate | Dark gate, twin towers, iron teeth |
| 18 | Mount Doom | Volcanic cone, open molten crater and lava |
| 19 | Barad-dûr | Jagged black citadel topped by fiery Eye |
| 20 | The Dead Marshes | Low marsh pools, dead plants, wandering lights and mist |
| 21 | Erebor | Lone towering mountain and carved dwarven entrance |
| 22 | Dale | Compact walled city, towers and colorful roofs |
| 23 | Mirkwood / Dol Guldur | Dense dark canopy encircling ruined hill fortress |
| 24 | Cirith Ungol | High narrow watchtower in a jagged mountain pass |

## Cinematic script and timing

| Time | Sequence |
|---|---|
| 00:00–00:25 | Shire at dawn; low sweep past Hobbiton |
| 00:25–00:50 | Bree, Weathertop and Rivendell |
| 00:50–01:15 | Misty Mountains, Moria and golden Lothlórien |
| 01:15–01:40 | Argonath, Fangorn, Isengard, Edoras and Helm's Deep |
| 01:40–02:05 | Minas Tirith and Osgiliath at dusk |
| 02:05–02:30 | Mirkwood, Dol Guldur, Erebor and Dale |
| 02:30–03:20 | Dead Marshes, Black Gate, Minas Morgul, Cirith Ungol, Mount Doom and Barad-dûr at night |
| 03:20–03:47 | Rising, wide-angle grand overview of the diorama |

Use smooth position and look-at curves with scene-aware terrain clearance. The timeline endpoint `t=227` is separately seekable. At 60 FPS, `227 * 60 = 13,620` nominal frame intervals; output frames are indexed `0..13619`, with final nominal frame at `226.9833...` seconds.

## Progress tracker

Update each state from `PENDING` → `IN PROGRESS` → `PASS` (or `BLOCKED`).

| Pass | Workstream | Status |
|---:|---|---|
| 1 | Application foundation | PENDING |
| 2 | Geography and terrain | PENDING |
| 3 | Landmark framework | PENDING |
| 4 | Western landmarks (1–8) | PENDING |
| 5 | Central and southern landmarks (9–16) | PENDING |
| 6 | Eastern landmarks (17–24) | PENDING |
| 7 | Atmosphere and VFX | PENDING |
| 8 | Cinematic timeline | PENDING |
| 9 | UI and free camera | PENDING |
| 10 | Integration, validation and export | PENDING |

---

## Pass 1 — Application foundation

**Objective:** A stable rendering baseline and project structure.

**Implement:**
- Create source modules and development entry HTML.
- Initialize Three.js using a pinned release and matching OrbitControls.
- Set up WebGLRenderer (antialiasing, responsive sizing, capped DPR, tone mapping, sRGB output, sensible shadows), perspective camera, scene and simple lighting.
- Add a basic ground/grid and orbit interaction as a temporary visual baseline.
- Add a meaningful WebGL/import failure overlay.
- Add development start and single-file export commands. Bundle authored JS/CSS into `dist/middle-earth-journey.html`.
- Add simple startup/build validation.

**Acceptance checks:**
- Browser renders a visible scene with no console errors.
- Mouse/touch orbit and resizing work.
- Export command produces HTML that loads in a suitable browser environment.
- No terrain or landmark work is started here.

## Pass 2 — Geography and terrain

**Objective:** One coherent miniature world.

**Implement:**
- Replace baseline ground with continuous seeded terrain (roughly 200 × 200 units; 160–200 subdivisions per axis).
- Implement canonical `getTerrainHeight(x,z)` and regional masks for the Shire/Eriador, Misty Mountains, Rohan, Gondor, Mordor, Mirkwood and Erebor.
- Build Mordor's encircling ridges and central volcanic region; avoid hard seams.
- Color terrain by biome, slope and elevation with blended boundaries.
- Create western ocean, spline-defined Anduin river and beveled diorama base.
- Add a developer-friendly region view if useful.

**Acceptance checks:**
- Heights are finite and continuous; landmark coordinate reservations are within bounds.
- Shire, mountain ridge, Mordor basin, Northeast and river are visually distinguishable.
- Geography remains reproducible for the same seed; no visible seams or broken normals.

## Pass 3 — Landmark generation framework

**Objective:** Reusable architecture and a validated registry.

**Implement:**
- Create all 24 registry definitions with stable IDs, placement, scale, archetype and camera focus.
- Implement `createTower`, `createCastle`, `createFortress`, `createVillage`, `createMountain`, `createRock`, `createTree`, `createBridge`, `createWall`, `createRuins`, `createWaterfall`, and relevant helpers where used.
- Create shared material/geometry caching and consistent scale conventions.
- Add terrain-height placement and foundations to avoid buried or floating structures.
- Add a development camera selector for registry positions.
- Mark unbuilt entries explicitly as **not implemented**; never count them as completed.

**Acceptance checks:**
- Registry has exactly 24 unique valid entries.
- Shared builders produce nonempty valid geometry.
- Camera can focus on all reserved sites.
- No fake generic marker is counted as a completed landmark.

## Pass 4 — Western landmarks (1–8)

**Objective:** Eight recognizable western and central-north sites.

**Implement all:** Grey Havens, Hobbiton, Bree, Weathertop, Rivendell, Moria West Gate, Lothlórien and Argonath.

**Emphasis:** harbor arches, round doors, timber-framed gables, ruined rings, elven terraces and waterfalls, recessed mountain gate, golden talan platforms, twin statues with raised arms. Place vegetation intentionally and use shared geometry.

**Acceptance checks:**
- Every site has actual distinctive geometry, positioned above terrain and recognizable from its focus camera.
- All eight sites receive a screenshot review if browser screenshot tools are available.
- Previously working terrain and camera controls are preserved.

## Pass 5 — Central and southern landmarks (9–16)

**Objective:** Eight further geographically coherent structures.

**Implement all:** Amon Hen, Fangorn, Isengard, Helm's Deep, Edoras, Minas Tirith, Osgiliath, and Minas Morgul.

**Emphasis:** Orthanc's four spines; Helm's Deep's curved wall and gorge; Meduseld's golden hall; **seven** readable Minas Tirith tiers; Osgiliath's ruins/bridges; Minas Morgul's pale spectral silhouette. Instance dense Fangorn forest.

**Acceptance checks:**
- Sixteen total landmarks now have complete geometry.
- Distinctive forms are visible; structures do not float or disappear inside slopes.
- Test at least a front/hero camera shot for each of the eight new sites.

## Pass 6 — Eastern and dark-region landmarks (17–24)

**Objective:** Complete the landmark inventory.

**Implement all:** Black Gate, Mount Doom, Barad-dûr, Dead Marshes, Erebor, Dale, Mirkwood/Dol Guldur and Cirith Ungol.

**Emphasis:** towering gate and iron teeth; volcanic cone and crater; fiery Eye citadel; low marsh pools; a solitary mountain with carved gate; populated walled city; dark forest and ruins; isolated fortress in a mountain pass. Build geometry and static emissive materials; advanced effects come in Pass 7.

**Acceptance checks:**
- All **24/24** registry entries produce nonempty, visibly distinct landmark groups.
- Automated coverage checks enumerate every registry ID and constructed group.
- No major z-fighting, clipping, or obvious missing landmark.

## Pass 7 — Atmosphere, water, lava and VFX

**Objective:** Bring the world to life without losing determinism.

**Implement:**
- Procedurally animated sea and Anduin water surfaces.
- Waterfalls and mist at Rivendell.
- Mount Doom lava ribbons, emissive crater, dark ash and rising embers.
- Pulsing Barad-dûr fiery Eye; Minas Morgul spectral green light; Dead Marshes will-o'-wisps and drifting mist.
- Dawn, midday, dusk and night lighting palettes, interpolated smoothly with sky/fog and local illumination.
- A central `updateSceneAtTime(t)` that computes every time-varying state from absolute time.
- Analytical seeded particle trajectories or indexed noise; bounded pools and no per-frame runaway allocations.

**Acceptance checks:**
- Each specified effect is visibly present and geographically attached to the correct place.
- Seeking to time A, then B, then A reproduces the time-A scene state.
- No free-running shader clocks, elapsed-frame accumulators or uncontrolled randomness.

## Pass 8 — Exact 227-second cinematic and deterministic seeking

**Objective:** A scripted camera journey that can be rendered in any order.

**Implement:**
- Camera and look-at paths covering all chapters in the [cinematic script](#cinematic-script-and-timing).
- Smooth Catmull–Rom or compatible spline interpolation, safe camera framing and clearance.
- `seekTime(t)` updates camera, environment, particles, water, emissive effects, region metadata and render state.
- `seekFrame(i)` evaluates `seekTime(i / 60)` for valid output indices `0..13619`.
- Separate endpoint support for `seekTime(227)`.
- Real-time playback that advances one authoritative `cinematicTime` (may pause or loop by user setting).

**Acceptance checks:**
- No abrupt camera cuts, upside-down views, or major terrain/structure collisions.
- All chapter boundary timestamps and the endpoint can be sought directly.
- Out-of-order repeated seeking returns the same scene at the same time.
- Duration and frame calculations are correct.

## Pass 9 — HUD, controls and free camera

**Objective:** Complete accessible playback and exploration controls.

**Implement:**
- Minimal editorial masthead: **MIDDLE-EARTH — A Journey Through Legend**.
- HUD: current region/landmark, `MM:SS / 03:47`, frame readout, measured FPS and playback status.
- Play/Pause, Restart, timeline slider, Previous/Next Landmark, Cinematic/Free Camera and optional lighting preview.
- OrbitControls for free cam with mouse/touch; clean takeover and return to cinematic camera.
- Responsive controls with labels, focus states and restrained contrast.

**Acceptance checks:**
- Every control works and reflects actual state.
- Scrubbing changes camera and all effects consistently.
- Free-camera logic does not overwrite scripted transforms; switching back restores the correct cinematic pose.
- Keyboard and pointer interaction do not conflict with the HUD.

## Pass 10 — Integration, browser tests, optimization and export

**Objective:** Produce and verify the final deliverable.

**Implement:**
- Audit all preceding acceptance checks and fix regressions.
- Validate all 24 landmark IDs, locations, constructed geometry, visibility and identity.
- Inspect representative timestamps including `0, 25, 50, 75, 100, 125, 150, 180, 200, 226.9833, 227` seconds.
- Verify time-A → time-B → time-A seeking for identical inputs and same rendering environment.
- Test viewport resizing, scrubbing, camera modes, pause/resume, scene errors and exported HTML startup.
- Profile frame rate and render calls at representative camera positions; optimize geometry reuse, shadow casters and particle updates without dropping features.
- Export `dist/middle-earth-journey.html` and verify that authored CSS/JS are embedded and CDN versions are pinned.

**Acceptance checks:**
- Browser loading and console checks pass where browser tooling is available.
- All 24 landmarks pass coverage and visual inspection.
- Full 227-second timeline, endpoint and deterministic seeking work.
- UI controls work; output is a single deliverable HTML file.
- Report actual measured performance and any limitations, rather than claiming universal 60 FPS.

---

## Quality gate after every pass

Run this review **before** recording `PASS`:

1. **Scope:** Did the agent implement only the current pass and required interfaces?
2. **Completeness:** Are the features functional, with no placeholders presented as complete?
3. **Compatibility:** Do newly edited modules preserve prior passes and shared architecture?
4. **Automated checks:** Run syntax/build/unit or smoke tests that are available. Record the commands and observed results.
5. **Browser review:** If a browser/testing tool is available, load the application, inspect console errors and capture an appropriate screenshot. Record exactly what was observed.
6. **Visual correctness:** Inspect landmark silhouette, geographic placement, camera framing and unexpected occlusion when relevant.
7. **Determinism:** Verify repeatable seed and time-based state for affected features.
8. **Performance:** Identify major draw-call, memory and frame-time regressions without inventing performance numbers.
9. **Repair:** Fix current-pass problems and rerun failed checks.
10. **Decision:** Record `PASS` only if acceptance criteria are satisfied. Otherwise record `BLOCKED`, state what failed, and stop or continue repairs within the same pass.

### Report template

```markdown
### Pass N — [name]
- Status: PASS | BLOCKED
- Files changed:
- Implemented:
- Tests executed and results:
- Screenshots / browser checks (if available):
- Known issues:
- Performance observations (measured only):
- Git commit (if any):
- Next action:
```

## Deterministic animation examples

**Incorrect (frame-history dependent):**

```js
lava.rotation.y += 0.01;
particle.position.y += speed * dt;
```

**Correct (absolute-time derived):**

```js
lava.rotation.y = time * 0.2;
particle.position.y = baseY + ((time * speed + phase) % travelHeight);
```

Use deterministic initial seeds, particle-specific phases, and explicit time parameters for shader uniforms. Avoid per-frame random sampling. Seeking should update the same render-visible state regardless of the order of previous seeks.

## Suggested local model and agent settings

| Parameter | Recommendation |
|---|---|
| Model | Qwen 27B Q3 |
| Temperature | 0.2–0.3 |
| Top P | 0.9 |
| Context | 64K–128K |
| Response/output budget | 8K–16K tokens per pass to start, increase when necessary |
| Thinking | On, when supported |
| Execution | Repository-editing coding agent with terminal and browser access |
| Versioning | Git commit after each accepted pass |
| Testing | Browser automation such as Playwright plus automated registry/time checks |

Keep this document on disk, update progress in it, and commit it with the source. The repository and this plan are the source of truth; do not depend on conversational memory for implementation state.

## Execution log

Append one completed [report template](#report-template) entry here after each pass. Do not pre-fill test results.

---

## Final handoff checklist

- [ ] `dist/middle-earth-journey.html` exists and opens in a supported browser environment.
- [ ] All authored CSS and JavaScript are embedded; only pinned Three.js/OrbitControls CDN modules are external.
- [ ] No external art assets, fonts, models, or runtime build steps.
- [ ] 24 unique landmark registry entries produce 24 distinct visible landmarks.
- [ ] Regions and terrain share coherent geography and height functions.
- [ ] Day/night cycle and VFX are driven by one authoritative cinematic time.
- [ ] Scripted camera runs for exactly 227 seconds and includes all scheduled chapters.
- [ ] `seekTime(227)` is valid; output frame indices `0..13619` are valid at 60 FPS.
- [ ] Repeated out-of-order seeking is deterministic in a fixed rendering environment.
- [ ] Play/Pause, scrub, restart, landmark navigation and free-camera controls work.
- [ ] Responsive resizing, errors and browser fallback are handled.
- [ ] Performance observations and tests are truthfully documented.

**Deliver:** final HTML, updated plan with pass results, and concise implementation/test summary.
