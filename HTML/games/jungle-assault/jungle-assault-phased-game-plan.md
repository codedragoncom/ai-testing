<!--
Type: Phased implementation plan
Target: HTML Canvas, vanilla JavaScript
Output: dist/jungle-assault.html
Related: jungle-assault.md; jungle-assault-build-instructions.md
-->

# JUNGLE ASSAULT — Operation Ironstorm
## Phased Game Development Plan for Qwen 27B Q3

**Final deliverable:** `dist/jungle-assault.html`  
**Genre:** Original vertically scrolling, top-down arcade run-and-gun inspired by 1980s arcade games  
**Runtime:** HTML5 Canvas 2D, vanilla JavaScript, Web Audio API  
**Shipping constraints:** One offline-capable HTML file; all CSS, JS, art, and audio inline; no runtime dependencies  
**Development approach:** Modular source, automated tests, browser playtesting, final HTML bundling

## Agent operating instructions

You are the lead JavaScript game engineer. Read this entire document and inspect the repository before editing. Work on **only one phase at a time**. Preserve working systems. Do not silently remove requirements or declare untested functionality working. Favor reliability and readable code over minification. Make a Git commit after a passing phase when Git is available. Update the progress table and development log after each phase. Stop at the gate if criteria fail; repair before advancing.

**Definition of done for every phase:** code implemented, appropriate automated tests pass, game starts in a browser, console has no new errors, manual gameplay smoke test passes, no regression in earlier systems, and the phase report states exactly what was tested. If browser tools are unavailable, explicitly mark browser validation as unverified; do not claim PASS for that gate.

**Progress tracker**

| Phase | Status | Evidence / Notes |
|---|---|---|
| 01 Foundation | NOT STARTED | |
| 02 Player and camera | NOT STARTED | |
| 03 Combat | NOT STARTED | |
| 04 Enemy AI | NOT STARTED | |
| 05 Terrain and cover | NOT STARTED | |
| 06 Eight stages | NOT STARTED | |
| 07 Fortresses | NOT STARTED | |
| 08 Pickups and rescues | NOT STARTED | |
| 09 Graphics and VFX | NOT STARTED | |
| 10 Audio and UI | NOT STARTED | |
| 11 Mobile and performance | NOT STARTED | |
| 12 QA and export | NOT STARTED | |

## Nonnegotiable game design

- Player moves in eight directions at normalized speed; can move while shooting.
- Classic keyboard aiming follows facing/movement, with optional mouse aim; mobile uses twin sticks.
- Rifle fires continuously with cooldown; bullets are bounded, faction-tagged and collision-tested in world space.
- Grenades have visible trajectory/fuse, bounded area damage once per target, and limited inventory.
- Camera moves north (negative world Y) through eight handcrafted stages; screen coordinates are derived from camera transform, never used for gameplay collisions.
- Six enemy archetypes: patrol, rifleman, charger, trench guard, machine gunner, elite guard.
- Trees, rocks, rivers, barricades, destructible cover, supplies, prisoner rescues, finite fortress waves.
- Three starting lives, safe respawn/invulnerability, scoring, high score, pause, restart, game over, victory.
- Original procedural Canvas visuals and Web Audio sounds; no copied game assets.
- Responsive keyboard/mouse/touch; landscape-friendly controls; 60 FPS target on capable desktop hardware.
- Final file works from `file://`, even with network disabled.

## Engineering contracts

### Coordinates and time
- World X right, world Y down, north negative Y. `screenPosition = worldPosition - cameraPosition`.
- Logical viewport 640 × 800, scaled proportionally with capped DPR (1.5–2). No aspect distortion.
- Simulation uses fixed 1/60-second updates with bounded accumulator (max 4–5 catch-up steps per render); presentation uses `requestAnimationFrame`.
- Gameplay timers use simulation elapsed time and stop during pause, title, and transition states.

### Interfaces
Maintain stable module boundaries and document any intentional API changes:
- `Game`: owns state machine and orchestration.
- `Input`: keyboard/mouse/touch action state with focus-loss reset.
- `Camera`: world-to-screen transform and scrolling clamps.
- `Collision`: circle/rectangle/swept-segment tests with faction filtering.
- `Entities`: update, bounds, render, lifecycle/death callbacks.
- `Spawning`: one-shot data-driven activation events; bounded active enemy counts.
- `Level`: geometry, hazards, spawns, pickups, objectives, fortress definition.
- `Renderer`: Canvas drawing without changing gameplay state.

### Reuse and performance
- Pool bullets, particles, and transient effects.
- No uncontrolled spawning or unbounded arrays.
- Cache static terrain tiles or offscreen layers.
- Avoid per-frame allocations where feasible.
- No external assets, fonts, network calls, runtime `import` or CDN access in final export.

### Project layout
```
jungle-assault/
  GAME_PLAN.md
  index.html
  package.json
  src/
    main.js
    core/{Game,GameLoop,Input,Camera,Collision}.js
    entities/{Player,Enemy,Bullet,Grenade,Pickup}.js
    systems/{Combat,EnemyAI,Spawning,Terrain,Missions,Audio,Effects}.js
    levels/{level01,...,level08}.js
    ui/{HUD,Menus}.js
  tests/
  tools/build.mjs
  dist/jungle-assault.html
```
The development server/build tools may be used locally; the final game must require none of them. Any bundler output must embed the application code, rather than leave unresolved file imports.

---

## Phase 01 — Foundation and project architecture
**Goal:** a running game shell before gameplay complexity.

**Implement**
1. Repository structure, package scripts, source modules and test runner.
2. Canvas creation, resolution scaling, resize handling, DPR cap.
3. Fixed-step game loop and game state machine: TITLE, HOW_TO_PLAY, PLAYING, PAUSED, PLAYER_DYING, RESPAWNING, FORTRESS_BATTLE, STAGE_COMPLETE, GAME_OVER, VICTORY.
4. Input abstraction and focus-loss protection.
5. Basic scene renderer with temporary ground, debug FPS and camera bounds.
6. Initial single-file export command and build verification.

**Tests:** timestep consistency at differing render FPS, pause freezes updates, resize maintains aspect ratio, input resets on blur, browser startup.

**Gate:** the title screen and a basic game scene render; pause/resume and resize work; export opens offline.

## Phase 02 — Player movement, world coordinates and camera
**Goal:** responsive controlled movement in a vertically scrolling world.

**Implement**
1. Eight-direction movement with normalized diagonal speed, acceleration/stop behavior.
2. Player bounds and solid-obstacle collision with wall sliding.
3. World-space player placement and northward camera follow/clamp.
4. Separate movement-facing and optional mouse aim direction.
5. Temporary collision-debug overlay toggled in development mode.

**Tests:** all eight directions equal max speed, no wall traversal, consistent screen/world transform, camera stops at limits, no input sticking.

**Gate:** player traverses a test map and camera tracks without jitter or coordinate drift.

## Phase 03 — Rifle, grenades and collision engine
**Goal:** reliable projectile combat.

**Implement**
1. Rifle with fixed cooldown, eight-direction shooting, muzzle origin and animation signal.
2. Pooled player/enemy bullet entities; bounded lifetime; bullet ownership and hit rules.
3. Swept segment-vs-circle/rectangle collision for fast projectiles; first-impact resolution.
4. Grenade inventory, aim/throw trajectory, fuse, detonation, area damage once per entity, explosion visuals.
5. Damage pipeline with health, death once, invulnerability/respawn support.

**Tests:** diagonal shots, high-speed tunneling, friendly fire rules, projectile cleanup, grenade one-hit rule, no negative grenades, simultaneous impacts.

**Gate:** moving and shooting works with no known collision tunneling; grenades damage target groups reliably.

## Phase 04 — Enemy AI and spawning
**Goal:** combat against believable finite threats.

**Implement**
1. Six enemy types: patrol soldier, rifleman, charging soldier, trench guard, machine gunner, elite guard.
2. Explicit finite state transitions: IDLE/PATROL/ALERT/CHASE/ATTACK/RETREAT/DEAD.
3. Bounded line of sight, detection, cover selection (simplified), rate-limited attacks.
4. Data-driven one-shot activation spawns, safe spawn locations, active enemy cap and cleanup.
5. Enemy death scoring, drop hooks and basic telemetry.

**Tests:** no infinite spawn, correct enemy ownership of shots, line of sight blocked by cover, enemies stop shooting upon death, bounded counts.

**Gate:** a 2-minute combat sandbox remains stable and feels fair.

## Phase 05 — Terrain, jungle and destructible cover
**Goal:** coherent battlefields that match their collision geometry.

**Implement**
1. Procedurally drawn reusable grass, trails, jungle trees, rocks, mud, river and bridge tiles.
2. Separate decoration, solid obstacles, destructible cover, slow terrain, hazard and supply placement data.
3. Destructible crates/barrels/barricades/guard posts with finite health, damage states, debris and collision removal.
4. Cache static layers and render only visible world regions.

**Tests:** terrain collider matches visible shape, destructible obstacles are removed after destruction, no clipping at camera boundaries, movement modifiers recover after leaving terrain.

**Gate:** a visually coherent test stage supports cover, hazards and scenery with no world/screen mismatch.

## Phase 06 — Eight handcrafted scrolling stages
**Goal:** a playable beginning-to-end level structure.

Create these eight **distinct data-defined stages**, each 3–5 screen heights, with start/end, terrain, enemy trigger groups, obstacles, supplies, optional rescues, and encounter boundary:
1. Jungle Landing — movement/shooting tutorial.
2. River Crossing — water, bridges and banks.
3. Enemy Encampment — barricades, camp and prisoner.
4. The Canyon — narrow rocky ambush routes.
5. Swamp Territory — slow mud, limited lanes.
6. Mountain Outpost — rocky fortifications.
7. The War Zone — layered crossfire and supplies.
8. Ironstorm Fortress — elite defenders and final command center.

**Implement:** consistent level schema, loading/unloading/reset, stage progress, safe spawn, enemy trigger IDs, end markers, transition state.

**Tests:** every level loads and scrolls end-to-end, spawns once, all routes are passable, resets have no lingering bullets/enemies.

**Gate:** player can traverse all eight stage maps through test mode; normal stage objectives are defined and reachable.

## Phase 07 — Fortress waves and stage completion
**Goal:** each stage ends in a finite, readable combat set-piece.

**Implement**
1. Fortress activation line, scroll stop, warning, finite waves, defensive entities and completion conditions.
2. Unique per-stage fortress compositions using guard towers, trenches, machine guns, barricades.
3. Stage 8 elite commander with telegraphed, bounded patterns; never an endless invulnerability phase.
4. Completion bonus, results transition, next-stage routing and final victory.

**Tests:** every wave ends after all mandatory units are defeated, no duplicate activation, no impossible progression, boss patterns have safe responses.

**Gate:** each of eight stages can be completed in a playable route; victory happens only after final encounter.

## Phase 08 — Pickups, rescues, progression and balance
**Goal:** meaningful arcade rewards and full life cycle.

**Implement**
1. Grenade crates, score pickups, bonus life thresholds and inventory caps.
2. Prisoner-rescue locations, guard prerequisites, safe rescue interaction and scoring.
3. Three starting lives, death animation, one-life deduction, safe respawn, invulnerability, game over and retry.
4. Persistent high score with safe localStorage failure handling.
5. Difficulty progression, enemy density and pickup balancing.

**Tests:** grenade caps, rescue triggers once, life lost only once, safe spawn, score bookkeeping, final run progression.

**Gate:** the scoring/lives/rescue loop works across stage changes and restarts.

## Phase 09 — Original pixel art and VFX
**Goal:** readable retro presentation without breaking gameplay.

**Implement**
1. Original eight-direction player sprite, run/idle/firing/death frames using Canvas drawing or pre-rendered offscreen canvases.
2. Distinct enemies and silhouettes, muzzle flashes and projectile appearance.
3. Explosion, impact, dust, debris and rescue particles with bounded pools.
4. Jungle layering, consistent palette, UI-safe contrast and tasteful shake applied only to rendering.
5. Optional accessibility settings to reduce flashes/shake.

**Tests:** effects do not affect hitboxes, sprite orientation tracks aim, bounded particles, hazards/bullets remain easy to read.

**Gate:** game looks polished and combat information remains legible.

## Phase 10 — Audio, menus and HUD
**Goal:** complete arcade presentation and controls.

**Implement**
1. Single lazily created Web Audio context; procedural rifle, grenade, explosion, pickup, rescue, hit and level-complete sounds; optional synthesized music.
2. Main menu, instructions, pause, game over, victory, restart and mute.
3. HUD: score, high score, lives, grenades, stage, rescued prisoners and progress.
4. Keyboard accessibility, focus indication and responsive overlay layout.

**Tests:** no AudioContext before user gesture, mute works, menu transitions don't create duplicate listeners, pause freezes simulation.

**Gate:** full game can be started and navigated with keyboard/mouse, menus stable.

## Phase 11 — Mobile input and performance
**Goal:** practical phone/tablet controls and sustained play.

**Implement**
1. Left touch movement stick, right aim/fire stick, grenade and pause buttons; Pointer Events with pointer capture and per-pointer state.
2. Landscape-first responsive sizing, no page scroll while interacting, touch safety zones.
3. CPU profiling, pooling improvements, static background caching, bounded AI and event processing.
4. Quality options if needed for visual particles and DPR.

**Tests:** simultaneous two-stick plus grenade input; pointer cancellation; blur; rotated screen; long 5-minute soak at representative stage; check counts/memory/frame times.

**Gate:** usable controls on mobile and no obvious progressive slowdown on desktop.

## Phase 12 — Full QA, balancing and final single-file export
**Goal:** ship a finished independent HTML game.

**Implement**
1. Automated tests for coordinates, collisions, input, AI lifecycle, stage data, fortress completion and state transitions.
2. Browser smoke tests: start, movement/fire, grenade, death/retry, stage transitions, eight-stage playthrough, victory, restart, pause, touch where available.
3. Repair crashes, collision errors, broken routes, duplicated spawns and visual regressions.
4. Tune controls, enemy telegraphing, grenade availability, stage timing and lives.
5. Build `dist/jungle-assault.html`; inline all application code/styles/assets, remove runtime imports and network dependencies.
6. Test final file via `file://` with network disabled; capture proof (screenshots, console output, test summary).

**Release gate:** all eight stages are playable, boss/waves are finite, critical combat and scrolling tests pass, menus and touch work, and offline HTML opens with no network activity. Record any measured FPS limitations accurately.

---

## Required phase report template

```markdown
### Phase NN — [Name]
- **Status:** PASS / FAIL / BLOCKED
- **Files changed:** ...
- **Features implemented:** ...
- **Tests run and results:** ...
- **Browser checks and screenshots:** ...
- **Regressions checked:** ...
- **Open issues:** ...
- **Next phase:** ...
```

## Agent launch prompt

> Read `GAME_PLAN.md` completely and treat it as authoritative. Inspect the repository. Start with Phase 01 only. Implement all Phase 01 requirements, run tests and browser validation when tools are available, update the progress tracker and phase report, and stop. Do not begin Phase 02 without an explicit instruction.

## Continue prompt

> Read `GAME_PLAN.md` and the most recent phase report. Inspect the current codebase. Execute the next unfinished phase only. Preserve working functionality, test acceptance criteria and regressions, update the report and progress tracker, and stop before beginning the following phase.

## Final shipping checklist

- [ ] Eight-direction player movement and firing behave correctly.
- [ ] Bullets collide in world space and do not tunnel through cover/enemies.
- [ ] Grenades detonate once and respect inventory limits.
- [ ] All six enemy types function; active entity counts remain bounded.
- [ ] All eight stages are distinct and completable.
- [ ] Fortress waves and final commander can be defeated.
- [ ] Pickups, rescues, score, lives, game over and victory work.
- [ ] Player respawns safely; invulnerability prevents multi-life loss.
- [ ] UI, synthesized audio, keyboard, mouse and touch work.
- [ ] Scrolling never desynchronizes rendering from collision.
- [ ] No network calls/runtime imports remain in final HTML.
- [ ] `dist/jungle-assault.html` runs from `file://` with network disabled.
- [ ] Browser smoke tests and limitations are documented.
