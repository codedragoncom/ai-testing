<!--
Type: Phased game plan
Target: Godot 4.x, GDScript
Output: Complete Godot game project
Related: None
-->

# SHADOW INTERCEPT — Godot Game Implementation Plan

**Version:** 1.0  
**Target:** Godot 4.x, GDScript, desktop keyboard; designed for Qwen 3.8 Flash Next IQ3 as implementation model  
**Genre:** Retro top-down vehicular combat / endless arcade score attack  
**Reference:** [Spy Hunter (1983)](https://en.wikipedia.org/wiki/Spy_Hunter)  
**Project title:** **Shadow Intercept** (working title; original branding and content)

> **Core mandate:** Build a playable, polished, *original* spiritual successor. Re-create the ideas of a high-speed vertical road chase, weaponized car, tactical enemy encounters, resupply truck, forks, and brief water segments. Do **not** copy the original title, logos, art, sounds, theme tune, maps, or exact vehicle designs.

---

## 1. How to use this document with Qwen

This is **one plan document, not a one-file game**. A normal multi-file Godot project is safer and easier for a 27B-class quantized coding model to build, debug, and extend. Work in the numbered phases below, **one phase per coding session**. Have the model read this plan plus only the relevant files. Small, complete edits beat a request to generate the entire game at once.

**Operating rules for the implementation agent:**

1. Use **GDScript**, Godot 4.x APIs, typed variables and signals where useful. Check the installed engine version first and keep APIs consistent with it.
2. Work in the current repository. Inspect existing files before changes; never recreate the project or erase working features.
3. Implement **one phase only**. Respect its scope and acceptance tests. No premature work on later phases.
4. Maintain a running `TODO.md` as the **single source of truth for implementation progress**. It records task checkboxes, current phase, acceptance gates, actual test results, blockers, decisions and the next action. Keep this plan unchanged unless the user approves a specification change.
5. Prefer small reusable scenes/scripts over huge manager classes. Each gameplay system has a clear owner; avoid circular references and excessive autoloads.
6. Use programmatically drawn placeholder shapes first. Replace art and sound only after mechanics feel good. Do not fetch copyrighted game assets.
7. Never say a test passed unless it actually ran. If GUI testing is unavailable, run headless import / script checks where possible and mark visual or playability checks **not verified**.
8. After a phase, stop and report changes, tests, remaining issues, and a short proposal for the next phase. Do not continue without approval.
9. When an API is uncertain, consult the installed Godot class reference or official docs; do not invent methods or nodes.
10. Keep the game playable with keyboard only. Mouse may be used in menus; controller support is optional polish.

### Recommended session sequence

1. Start a new or continued coding session with the **master handoff prompt** in §15.
2. Supply this file, `TODO.md` if it exists, and the current codebase. Ask for **Phase 0** only if no phase is started; otherwise resume the first incomplete task from `TODO.md`.
3. Review its report, run the game, fix blockers, then accept the phase.
4. Repeat for Phases 1–10. Split a phase in two if the model begins to lose context or produce speculative code. Mark each checkbox only after verification, and mark the phase complete only when its completion gate passes.

### Mandatory progress tracking: `TODO.md`

The **game plan specifies what to build**; `TODO.md` records **what has actually been done**. Do not maintain a separate `IMPLEMENTATION_STATUS.md` or duplicate checklists in `README.md`.

At the beginning of each coding session:

1. Read this plan and `TODO.md`. If `TODO.md` is missing, create it from the phase tasks and acceptance criteria in §5; leave all tasks unchecked until verified against the project.
2. Inspect the repository and last test results. Check that the current phase and completed items reflect actual code; never trust checkmarks without evidence when they conflict with the project.
3. Select the first incomplete, unblocked task in the active phase. State the task and the verification required before editing.

While coding and at the end of each session:

1. Update `TODO.md` after **each verified task**, not only at the end of the phase. Record files changed, tests actually run, results, known issues and decisions.
2. Use `[x]` only for implemented **and verified** work; keep unverified or partially working tasks `[ ]` and explain the gap. A successful code generation or headless import alone does not verify game feel.
3. Preserve completed task history and unresolved bugs. If an earlier task regresses, reopen its checkbox and record why.
4. Never skip an incomplete task without recording the dependency/blocker. Do not silently change requirements to make a task appear complete.
5. A phase is complete only when **all task checkboxes and its completion gate** are checked, with evidence. Stop for user approval before beginning the next phase.
6. End every session by writing a concise next-action line so a fresh Qwen session can resume without guessing.

**Required `TODO.md` skeleton** (the agent expands each phase with the exact tasks and acceptance checks from §5):

```markdown
# Shadow Intercept — Development Tracker

## Current Status
- Current phase: 0 — Project scaffold
- Last accepted phase: None
- Build status: Not verified
- Godot version: Not checked
- Next action: Verify engine version and inspect existing project

## Phase 0 — Project scaffold and test harness
- [ ] Verify installed Godot version and initialize project
- [ ] Configure viewport and input actions
- [ ] Create title and gameplay placeholder scenes
- [ ] Write README with launch commands and controls
- [ ] Initialize debug seed and logging

### Phase 0 Completion Gate
- [ ] All phase tasks implemented and individually verified
- [ ] Headless import completed without errors (record command/result)
- [ ] Script/scene loading checked without errors
- [ ] Manual menu/start/exit playtest completed
- [ ] Existing features checked for regressions
- [ ] No open critical blockers
- [ ] TODO.md updated with evidence and next action

Phase status: IN PROGRESS

## Phases 1–10
<!-- Expand the exact tasks and acceptance criteria from Section 5.
     Keep all checkboxes unchecked until verified. -->

## Known Issues and Blockers
- None documented yet

## Decisions
- Godot 4.x + GDScript; procedural original visuals first

## Session History
### Session 1 — YYYY-MM-DD
- Work: Not started
- Files changed: None
- Tests run: None
- Results: Not verified
- Next action: Start Phase 0
```

For every later phase, create a matching **Phase N Completion Gate** including all acceptance criteria for that phase, a no-critical-regression check, test evidence, and a final status of `IN PROGRESS`, `BLOCKED`, `READY FOR REVIEW`, or `ACCEPTED`. Only user-approved phases are `ACCEPTED`. Do not pre-check future tasks.

---

## 2. Vision, player experience and scope

**One-sentence pitch:** Pilot a prototype pursuit car through a continuously scrolling highway, evade traffic, destroy hostile interceptors, meet an allied supply rig, and survive escalating territory changes for the highest score.

**Player fantasy:** Fast reflex driving with deliberate choices: stay on the road to score, avoid civilians, shoot threats, decide when to use rare special weapons, and maneuver into a supply truck without crashing.

**Game loop:** Drive → read traffic → steer/accelerate → identify threat → evade or attack → earn score → collect resupply → survive tougher waves → repeat until lives run out.

**Core design principles:**

- Readable at speed: traffic class and attack telegraphs should be unmistakable.
- Arcade feel over realistic car physics: responsive controls and fair collisions.
- Risk/reward: staying on-road and driving faster pays, but traffic and enemies get harder.
- Short recoveries: death, respawn, and returning to play should be quick.
- Deterministic testing: fixed RNG seeds make encounter bugs reproducible.

**MVP (Phases 0–5):** playable highway, player handling, civilian and enemy traffic, gunfire, collision/death, score/lives, title/game-over/restart, stable difficulty ramp.

**Full v1 (Phases 6–10):** special weapon loadouts, supply rig, forks/regions, optional water transformation, audio/VFX, balancing, saved best score and release build.

**Explicit non-goals for v1:** multiplayer, open world, real-world cars, realistic drivetrain physics, elaborate cutscenes, in-game microtransactions, online leaderboards, 3D terrain, advanced traffic navigation.

---

## 3. Game specification

### 3.1 View and display

- 2D overhead view with vertical forward motion visually represented by the world scrolling **down**.
- Logical canvas: **480 × 720**; stretch with `canvas_items` to fit modern screens; preserve aspect ratio, use pillarboxing as needed.
- Nominal 60 FPS physics; delta-independent logic. Player stays mainly in the lower 55% of the view; road features and vehicles move relative to the player.
- Default road: 3 lanes, roughly 300 logical pixels wide, centered at x=240, with shoulders on both sides. Keep widths/data configurable.
- Art direction: original near-future midnight-highway palette; bright silhouettes, contrasting hazard indicators, restrained synth/retro styling.

### 3.2 Controls

| Action | Keyboard default | Meaning |
|---|---|---|
| Steer | Left / Right or A / D | Change lateral position |
| Accelerate | Up or W | Increase forward speed |
| Brake | Down or S | Decrease forward speed |
| Main gun | Space | Hold for repeating fire |
| Special weapon | Shift | Use currently selected special |
| Cycle special | Q / E | Previous / next available special |
| Call supply rig | C | Request when signal is available |
| Pause / resume | Esc | Toggle pause overlay |
| Confirm / restart | Enter | Menu action |

**Initial handling values (tune, don't treat as final):** speed in 0–900 world-units/s, starting at 420; acceleration 300/s²; braking 520/s²; passive deceleration 120/s²; steering 300 px/s at normal speed; extra responsive steering at low speed; gun interval 0.15 s. Expose exported tuning properties or one config resource. Define whether world scroll is zero at zero speed (yes).

### 3.3 Player state

`ACTIVE`, `INVULNERABLE`, `DESTROYED`, `RESPAWNING`, `TRANSFORMING`, `GAME_OVER`. Implement transitions in one owner, not scattered `if` flags. Player cannot fire while destroyed or transforming. After a lost life, respawn on clear road with about 2 seconds of flashing invulnerability. No automatic loss of life during the initial learning grace period (suggest 20 seconds), but do **not** make players permanently invulnerable.

Starting lives: **3**. Earn one extra life on score threshold (suggest every 20,000 points), with a cap of 5 held lives. If lives are interpreted as remaining respawns rather than total vehicles, document that choice and test it consistently.

### 3.4 Traffic and enemy archetypes

| Unit | Faction | Movement | Interaction | Phase |
|---|---|---|---|---|
| Civilian sedan | Neutral | Follows lane; occasional safe lane change | Must not be shot or rammed | 3 |
| Cargo truck | Neutral | Slow, large collision footprint | Obstacle, higher avoidance challenge | 3 |
| Pursuer | Enemy | Tracks player lane with limited turning rate | Can be shot or forced off-road | 4 |
| Blocker | Enemy | Tries to occupy player's future lane | Forces evasive maneuver | 4 |
| Armed raider | Enemy | Keeps distance, shoots back | Projectile hazard | 5 |
| Attack drone | Enemy | Crosses from upper screen, telegraphs target | Drops a timed danger marker | 8 optional |

**Fairness:** prevent impossible enemy-and-civilian wall patterns. A spawn director must keep at least one plausible path open for the player at the moment a group enters. Never spawn directly overlapping the player or inside the respawn safe zone.

### 3.5 Weapons and combat

- **Twin forward blasters:** unlimited ammunition, rate-limited shots, simple straight-line projectiles, modest recoil/VFX. Default weapon.
- **EMP mist:** leaves a short trail; temporarily slows/disables nearby hostile ground vehicles without harming civilians. Limited charges.
- **Road slick:** creates a patch that makes pursuing hostiles skid. Limited charges; must not cause an unavoidable collision for civilians.
- **Skybreaker missile:** targets the airborne threat only; unavailable without a valid target. Limited charges.
- **Water mode replacement:** a short-range forward flame jet replaces the road slick during water segments (original presentation/behavior).

First deliver only the gun. Add specials after core combat is reliable. Use deterministic collision layers/masks. Projectiles should be freed when off-screen, on impact, or on timeout. Damage should be applied once per collision event, never once per frame.

### 3.6 Scoring and penalties

Proposed values for first balance pass:

| Event | Points / effect |
|---|---|
| Forward movement while on valid roadway | 10 points/s at baseline speed, scaled by speed |
| Destroy pursuer | +150 |
| Destroy blocker | +200 |
| Destroy armed raider | +300 |
| Destroy aerial threat | +500 |
| Damage/destroy civilian | No points; lock positive scoring for 5 seconds |
| Off-road | No distance points while off-road |
| Survive a complete region | +250 |

Display score, best score, lives, speed, active special + charges, and scoring penalty indicator. Never reward civilian destruction. Clamp score to a safe integer range as appropriate; avoid duplicate kill credit.

### 3.7 World progression

- Start with straight highway and low traffic density.
- Add slight road curvature with a **segment-based road centerline** that drifts horizontally as the world advances. The lane layout follows the road centerline rather than staying at one fixed x.
- Later add occasional forks and a clear **route choice gate**. Branch chosen based on player lateral position when crossing a checkpoint; do not create two complex simultaneous maps.
- Regions: **Neon Bypass** (starter), **Canyon Run** (narrow shoulders), **Stormline** (rain/low visibility but readable), **Floodway** (water route).
- Water is a bespoke phase after road transitions are solid. Enter via a marked conversion bay and switch steering, hazards, and relevant special weapon. Return to road with another conversion marker.

For MVP, road generation can be procedural geometry with lane markings, edge strips, and scenery sprites. Use a deterministic world-distance coordinate, segment pooling, and re-use objects instead of accumulating endless nodes.

### 3.8 Supply rig

A distinct friendly armored support truck enters on-screen after a cooldown and available-call signal. It matches the player's speed and exposes a clear rear docking zone. Player steers into the zone at a compatible relative speed, maintains contact for roughly 0.7 s, then receives one randomly or predictably selected limited special refill. Display the granted weapon. Failed docking must not destroy the player unfairly. No refills while transforming or already docking.

### 3.9 Difficulty

Difficulty grows from **elapsed survival / distance**, not solely score. Adjust spawn interval, enemy probability, allowed simultaneous hostiles, and road variation with clamped curves. Do not simply raise every speed indefinitely. Add fairness caps: maximum hostile count on-screen, minimum spawn gap, spawn-lane reservation, and temporary grace after respawn.

---

## 4. Technical architecture

### 4.1 Engine choices

- **Godot 4.x stable installed locally**; pin the actual version in `TODO.md` at Phase 0. Avoid version-specific APIs unless supported by that installed engine.
- **GDScript only** for v1. One conventional Godot project with `.tscn` scenes, `.gd` scripts, `.tres` resources, assets, tests.
- `CharacterBody2D` for player and other moving vehicles if collision-based movement is needed; `Area2D` for projectiles/pickups/hazards and overlap triggers; `CanvasLayer` for HUD/menus.
- `Node2D` procedural road rendering via `_draw()` in initial prototype; if tile art becomes useful later, consider `TileMapLayer`, not a prematurely complex map editor.
- Prefer signals for entity events (`destroyed`, `civilian_hit`, `collected`, `request_respawn`), and typed method calls within clearly owned modules.
- Avoid a full ECS, giant inheritance tree, or physics-heavy realistic wheel simulation.

Official reference: [Godot Area2D](https://docs.godotengine.org/en/stable/classes/class_area2d.html), [CharacterBody2D](https://docs.godotengine.org/en/stable/classes/class_characterbody2d.html), [TileMapLayer](https://docs.godotengine.org/en/stable/classes/class_tilemaplayer.html).

### 4.2 Proposed repository structure

```text
shadow-intercept/
  project.godot
  shadow-intercept-godot-game-plan.md
  TODO.md
  README.md
  scenes/
    main.tscn
    gameplay.tscn
    player.tscn
    traffic_vehicle.tscn
    enemy_vehicle.tscn
    projectile.tscn
    hazard.tscn
    supply_rig.tscn
    hud.tscn
    menus/
      title_menu.tscn
      pause_menu.tscn
      game_over.tscn
  scripts/
    game_controller.gd
    player_controller.gd
    road_controller.gd
    spawn_director.gd
    vehicle_base.gd
    traffic_vehicle.gd
    enemy_vehicle.gd
    weapon_controller.gd
    projectile.gd
    score_controller.gd
    supply_rig.gd
    region_controller.gd
    hud.gd
  data/
    balance.tres
  assets/
    art/
    audio/
  tests/
    test_notes.md
```

This is a **target layout**, not permission to create every empty file on Phase 0. Add files when their phase needs them. Avoid autoloads unless shared across scene changes (e.g., `RunState` for best score). Prefer one top-level `GameController` to own the run and connect signal sources.

### 4.3 Data ownership and update order

| Owner | Responsibilities | Must not own |
|---|---|---|
| `GameController` | Run lifecycle, pause, lives, phase states, coordination | Vehicle-specific steering math |
| `RoadController` | World distance, centerline, boundaries, drawing, road surface queries | Enemy AI and score awards |
| `PlayerController` | Input, velocity, steering, temporary invulnerability | Global enemy spawn policy |
| `SpawnDirector` | Spawn timing, fairness, pooling/despawning | Damage calculation |
| `WeaponController` | Fire cadence, charges, projectile creation | Global score |
| `ScoreController` | Scoring, penalty windows, extra life thresholds | Physics collisions |
| `RegionController` | Environment/forks/transformation schedule | HUD text formatting |
| `HUD` | Presentation; subscribe to signals | Game rules |

**Coordinate contract:** Player/enemies/road rendered in viewport coordinates; road content moves based on forward world distance. Pick **one** convention for relative traffic speed, e.g. `screen_y += (player_forward_speed - traffic_forward_speed) * delta`, and apply it consistently. For projectiles, use screen-space motion with clearly stated inheritance of player motion. Do not also move a parent container by world speed unless compensating exactly once; double-scrolling is a common bug.

**Physics contract:** Use `_physics_process(delta)` for movement/spawn scheduling; emit damage events, with i-frames and hit-once guards. `_process` only for presentation/VFX. Collision layers and masks documented in `README.md`.

**Randomness contract:** one initialized `RandomNumberGenerator` per run/seed. Debug key/CLI configuration can fix seed; production run may randomize. Log seed on run start.

---

## 5. Phased implementation roadmap

Each phase ends with **a runnable, testable build**. If tests fail, stay on the phase and fix the issues before moving forward.

### Phase 0 — Project scaffold and test harness

**Goal:** Empty but runnable Godot project with predictable configuration.

**Implement**

1. Verify Godot version and create `project.godot`, default entry scene, portrait-like logical viewport and stretch settings.
2. Define input actions exactly as §3.2.
3. Add minimal main/title scene with **Start**, **Quit** and readable title.
4. Create `README.md` with Godot launch/import commands, controls and scope.
5. Add `TODO.md` from the template in §1, expanded to cover all phases; note environment, tests, and limitations without checking unfinished work.
6. Prepare debug logging toggle and run seed, without building gameplay features.

**Acceptance:** project imports with no errors; launches to menu; Start opens a simple gameplay placeholder; Esc or exit safely returns/quits; no missing assets or broken node paths. Run a headless editor import if the Godot binary is present, e.g. `godot --headless --path . --editor --quit`, then `godot --headless --path . --quit-after 5`. If the binary uses `godot4`, use that instead. Report both exit status and log warnings.

### Phase 1 — Drivable player and scrolling straight highway

**Goal:** Driving feels responsive before enemies exist.

**Implement**

1. Road boundaries, stripes, shoulders and visually seamless downward scrolling using world distance.
2. Player vehicle with readable original geometric sprite and collision shape.
3. Acceleration, brake, passive deceleration, lateral steering, road-edge/off-road drag; speed clamping.
4. Camera/framing fixed to scrolling viewport. Keep player visible and recoverable near shoulders.
5. Display speed in a minimal HUD; expose handling tuning values.

**Acceptance:** 2 continuous minutes without a seam/jump; steering/braking work at different frame rates; vehicle cannot wander permanently outside world bounds; no movement at zero speed; on/off-road behavior obvious.

### Phase 2 — Run lifecycle, scoring, lives, and HUD

**Goal:** The game has meaningful stakes before hostile AI.

**Implement**

1. Game states: Title → Playing ↔ Paused → Game Over → Restart.
2. Score accrues only when forward movement and on-road conditions are true.
3. Initial lives, death animation placeholder, respawn invulnerability; extra life thresholds.
4. HUD: score, lives, speed, active region, penalty indicator placeholder.
5. Save best score locally using `FileAccess` in `user://`; handle absent/corrupt save safely.
6. Add a temporary debug-only damage action so death/restart can be tested; remove or gate for release.

**Acceptance:** pause freezes gameplay, timers and score; game over triggers at correct life count; restart clears the previous run's state; high score persists on reopening; one damage event cannot consume multiple lives.

### Phase 3 — Neutral traffic and collision rules

**Goal:** Traffic weaving is playable and fair.

**Implement**

1. Neutral sedan and large cargo vehicle, each with visible neutral colors and valid hitboxes.
2. Spawn director with lane selection, rate limit, safe initial distances, despawn and simple pool or reuse policy.
3. At least one lane-change behavior with warning/turn rate rather than teleporting.
4. Relative motion formula and collisions with player.
5. Civilian damage triggers 5-second scoring lock; HUD shows countdown.

**Acceptance:** 3-minute run without accumulating off-screen vehicles; no overlapping spawn stacks or unavoidable initial barriers; penalty starts and expires once; neutral units never award kill points.

### Phase 4 — Hostile traffic and main blasters

**Goal:** Complete first combat loop.

**Implement**

1. Pursuer and blocker AI with bounded lane changes and individual states (approach, align, attack/obstruct, recover).
2. Infinite-ammo forward blaster, shot cooldown, pooling/despawn, contact detection.
3. Enemy health, destruction event, simple explosion VFX and score award.
4. Enemy-vs-player collisions; distinguish ramming from bullet hits.
5. Limit on-screen enemy count; spawn path fairness.

**Acceptance:** shots reliably hit and cannot damage the shooter; one enemy yields points exactly once; destroyed enemies despawn; no collision multi-hit life drain; 5 minutes of play without runaway node count.

### Phase 5 — MVP completion and balance pass

**Goal:** Small finished arcade game before feature expansion.

**Implement**

1. Armed raider with readable projectile attack and telegraph; reasonable attack cooldown.
2. Progressive encounter difficulty with spawn and AI caps, respawn grace.
3. Title/pause/game-over polish, short help/controls screen, clear color coding.
4. Baseline sound placeholder hooks (silent if assets unavailable), screen shake toggle, basic explosions.
5. Debug toggles (seed, hitboxes, spawn rate) restricted to development build.
6. Document balancing values, known limitations and a release-worthy MVP checklist.

**Acceptance:** complete 10-minute playtest; fair enough to survive at least 2 minutes with careful play; no softlock/restart failure; 60 FPS target on a standard desktop; no unbounded timers/entities; no warnings/errors during normal play. **This is the stop point for an MVP release.**

### Phase 6 — Special weapons and inventory

**Goal:** Strategic limited-use abilities.

**Implement**

1. Special selection, ammunition counters, input cooldown and invalid-use feedback.
2. EMP mist and road slick with independent lifetimes, effects and entity filters.
3. Shared effect interface and VFX; avoid piling one-off checks into the player.
4. Clear charges on player death (or document a different design choice); preserve normal gun.

**Acceptance:** specials affect hostiles as specified; civilians are not deliberately targeted; charges decrement once; no negative ammo; pooled effects clean up; switching cannot fire invalid weapons.

### Phase 7 — Supply rig and resupply event

**Goal:** Risky but reliable vehicle docking.

**Implement**

1. Cooldown/availability signal, C-key request and automatic introduction after first region.
2. Supply rig approach, rear docking overlap zone, compatible relative-speed condition and progress cue.
3. Grant refill on successful dock, short invulnerability/clear separation as needed.
4. Cancellation/failure handling and rig despawn.

**Acceptance:** can consistently dock with correct alignment; cannot refill repeatedly in same visit; pressing C while unavailable gives feedback; no truck stuck at screen edge or collision-loop death.

### Phase 8 — Region system, road drift, forks and aerial threat

**Goal:** The world changes while remaining deterministic and readable.

**Implement**

1. Road centerline curvature over world-distance segments; lane placement follows current road width/center.
2. Territory palette/scenery modifiers and paced transitions.
3. Fork approach telegraph, choice gate, route selection, seamless merge into chosen region.
4. Aerial drone, marked impact target, avoidable bomb hazard, and Skybreaker anti-air missile.
5. Store region definitions as lightweight data, not hardcoded nested conditionals.

**Acceptance:** curve never snaps road under player; fork selection deterministic from input/position; no impossible lane barriers; drone bomb has clear warning; missile only consumes ammo on valid launch.

### Phase 9 — Water conversion segment

**Goal:** Distinct short mode, not a second giant game.

**Implement**

1. Entry bay, short transformation animation/state lock, boat sprite/collider and water boundaries.
2. Adjust handling and drag for water; add 1–2 simple enemy boat types.
3. Replace road slick with forward flame jet; preserve gun/other available specials as appropriate.
4. Exit bay, transformation back to car, state cleanup.

**Acceptance:** cannot enter two transformations at once; no carry-over road effects into water; player returns to car with correct controls/collider; all death/respawn paths work in either mode.

### Phase 10 — Final polish, QA and distribution

**Goal:** A stable and clearly original v1 build.

**Implement**

1. Original cohesive vector/pixel-style sprites, road decals, HUD and menus; scale safely at target resolution.
2. Original sound effects and original/cleared music; volume sliders and mute.
3. Gameplay tuning: readable spawns, speed curves, scoring, collision forgiveness, specials.
4. Accessibility: flashing reduced toggle, screen-shake off, remappable inputs where practical, sufficient contrast.
5. Save best score and settings safely; include credits and controls.
6. Export desktop release for primary OS, then secondary OS if time allows. Include licensing notices for third-party assets/tools if any.

**Acceptance:** 20–30 minute soak test without memory creep/crash; replay from game-over works repeatedly; pause/resume while specials or transforms active works; test at 16:9 and narrower screens; no copyrighted original assets; exported build starts cleanly without editor.

---

## 6. Road and spawning algorithms (implementation notes)

### Scrolling and positioning

Maintain `world_distance += player_speed * delta`. For a road segment or scenery object at fixed world-distance `feature_distance`, render its screen y as `reference_screen_y + (world_distance - feature_distance)` (with one consistent origin/sign convention). A feature ahead moves downward as the player advances. For traffic with `forward_speed`, its own world distance advances every physics tick; derive relative screen y from world positions. This single shared world-coordinate design avoids contradictory movement math.

Compute road center at `distance` via piecewise smooth interpolated segment anchors. Near the player, query `road_center_at(player_world_distance)` and `road_half_width_at(...)` for off-road penalty, shoulder drag and traffic lane centers. Keep road drift gentle enough to steer through at average speed. Start with straight segments; add curvature only in Phase 8.

### Fair spawn director

For each proposed wave: (1) choose eligible archetypes for current difficulty, (2) reserve a set of road lanes, (3) reject lane sets that block all player paths, (4) enforce distance/time gap from existing vehicles, (5) spawn outside the upper visible bound, (6) track active count and cleanup. Use fixed-seed replay to reproduce failures. Pool only after correctness is established.

### Collision layers (proposal)

| Layer | Objects | Typical masks |
|---|---|---|
| 1 | Player | Traffic, enemies, enemy shots, hazards |
| 2 | Neutral traffic | Player, effects as explicitly allowed |
| 3 | Enemies | Player, player shots, effects |
| 4 | Player projectiles | Enemies, optionally neutral traffic for penalties |
| 5 | Enemy projectiles | Player |
| 6 | Trigger zones | Player |
| 7 | Hazards/effects | Player or enemies based on ability |

Define exact masks in Godot, not just comments. Collision filtering must make friendly-fire/civilian penalties deliberate. `Area2D` overlap detection and direct `body_entered`/`area_entered` wiring require careful choice of body vs area types.

---

## 7. Enemy behavior: bounded state machines

No heavyweight pathfinding is needed. Each unit gets a compact state and transition timer.

**Pursuer:** `CRUISE → TARGET_LANE → CLOSE → DISENGAGE`. Target only when within a sensible vertical range; limit horizontal speed; re-evaluate lane target every 0.5–1.0 s, not every frame.

**Blocker:** `ENTER → SIGNAL_LANE_CHANGE → BLOCK → EXIT`. Make the lane change predictable by visible indicators. Do not force impossible wall formations.

**Raider:** `APPROACH → AIM_WARNING → SHOOT → COOLDOWN → EXIT`. Shots should be dodgeable with a reaction window, with aim direction locked shortly before fire.

**Drone:** `ENTER → MARK_GROUND → DROP → LEAVE`. Skybreaker missile can destroy before drop. Target marker remains legible in poor weather.

All state transitions are owned by the individual AI script. A global director controls **when and how many** enemies spawn, not per-frame steering.

---

## 8. UX and audio target

- Player vehicle: cool white/cyan and recognizable silhouette.
- Hostiles: red/amber accents; neutral traffic: varied non-hostile colors and no targeting reticle.
- Road edges and water boundaries: high contrast; clear distinction even without decorative terrain.
- Gun: brief muzzle flash, fast visible streak, sharp sound, modest hit spark.
- Hit: short screen feedback with reduced-flash option. Death: readable explosion, 1-second pause, then clear respawn.
- Supply availability: HUD signal; docking: ring/progress indicator; refill: large one-line notice.
- Score lock: explicit `CIVILIAN HIT — SCORING PAUSED` and countdown, never a silent penalty.
- Audio must be original/cleared. **Do not reproduce the original Spy Hunter soundtrack or licensed theme.** A new understated instrumental synth track may evoke 1980s action without copying melody.

---

## 9. QA and validation matrix

Use headless import/lint checks whenever possible **and** interactive playchecks for feel. Headless success is not proof of gameplay quality.

| Test | Procedure | Expected |
|---|---|---|
| Fresh import | Open project in clean Godot session | No parse or scene-load errors |
| Start/exit | Enter gameplay, pause, resume, quit | Correct states, no softlock |
| Long road | Drive 3+ minutes | No seam/teleport or increasing node count |
| Acceleration | Compare 30/60/120 FPS conditions | Similar travel distance within tolerance |
| Civilian impact | Damage neutral vehicle | No kill credit; 5s scoring lock |
| Enemy kill | Destroy single hostile | Exactly one score event |
| Multi-collision | Overlap several hitboxes in one tick | Max one life consumed due to i-frames |
| Spawn fairness | Fixed seeds at high difficulty | No unavoidable lane blockade |
| Supply dock | Align, match speed, wait | One refill only; rig safely exits |
| Special ammo | Fire final charge / cycle empty weapon | Charges never negative |
| Fork | Choose left/right across repeated seed | Stable correct region transition |
| Water roundtrip | Convert both directions, die mid-route | Correct controls, collision and cleanup |
| Persistence | Close/reopen game | Best score and settings retained |
| Export | Launch on target desktop without editor | No missing resource errors |

**Completion evidence:** For each phase record commands, exit codes, visible playtest result, approximate average FPS, active node count after a long run, and remaining defects. If an automated test framework is not configured, provide a manual test script/checklist and say which checks were not run.

---

## 10. Common failure modes to guard against

1. **Trying the entire game in one prompt:** causes duplicated logic and broken scene references. Enforce one-phase commits and acceptance gates.
2. **Viewport vs world confusion:** creates enemies moving backward or road sliding under cars. Document one world-distance coordinate contract.
3. **Double damage:** Godot signal plus overlap poll both triggering. Centralize damage, guard on invulnerability.
4. **Unbounded spawns:** bullets and cars off-screen never freed. Add cleanup and soak-test counts.
5. **Spawn impossibility:** all lanes blocked at once. Reserve escape corridor and difficulty caps.
6. **Car collision over-simulation:** realistic physics makes arcade controls frustrating. Start with deliberate kinematic motion.
7. **Feature creep:** water and branching road before core traffic/combat is fun. Freeze MVP at Phase 5.
8. **Old Godot APIs:** `TileMap` versus newer `TileMapLayer`, changed properties. Follow locally installed version docs.
9. **Invented verification:** successful script generation is not a tested build. Require actual import and observed gameplay results.
10. **Unauthorized asset reuse:** do not download or recreate branded car art, sound effects or the original's licensed music.

---

## 11. Performance budget

- Initial desktop target: **60 FPS**, low-end integrated GPU acceptable if using simple 2D assets.
- Typical live entities budget: ≤25 vehicles, ≤80 projectiles, ≤30 hazards/effects, configurable.
- No per-frame filesystem or network operations.
- Reuse road geometry/segments; recycle pooled instances only after proving correct reset behavior.
- Avoid nested scans across all entities each physics frame where signals or groups suffice.
- Add a debug overlay for FPS, active vehicles/projectiles, current region, world distance and RNG seed.

---

## 12. Deliverables and release criteria

**MVP deliverables after Phase 5:** importable Godot project; player/traffic/enemies/gun; menus; high score; README; `TODO.md` with phase gates and test evidence. It must be entertaining for a short run without special mechanics.

**v1 deliverables after Phase 10:** same plus supply mechanic, specials, region forks, water, original art/audio, settings, credits and desktop export.

**Definition of Done:** feature meets acceptance tests; no script parse errors; no known critical gameplay regressions; resources import on clean project; code has clear ownership; new files documented; `TODO.md` accurate and phase gate verified; player can complete multiple runs without restarting editor.

**Source control:** initialize git at Phase 0 if allowed. Make one logical commit per accepted phase with descriptive commit message. Never overwrite uncommitted user work. Keep imported caches (`.godot/`) and temporary exports out of git as appropriate.

---

## 13. Suggested build cadence for a smaller local coding model

| Step | Context supplied to model | Work request | Expected output |
|---|---|---|---|
| Plan | This plan + `TODO.md` | Explain only current incomplete task's file changes | Short execution outline |
| Implement | Current scene/script files | Write current phase only | Small compilable edits |
| Validate | Error logs + changed files | Fix exact failing diagnostics | Patch and test report |
| Review | Diff + acceptance checklist | Find regressions and missing tests | Findings, prioritized |
| Handoff | `TODO.md` + next phase section | Update checked tasks, gates, blockers and next action | Clean stopping point |

**Context limit strategy:** At 64K context, do not fill the window with the whole repository. Supply this plan, `TODO.md`, current scene scripts, relevant game controller APIs, and latest errors. Start a fresh session at phase boundaries if responses begin to repeat, ignore constraints, or invent APIs.

**Model settings suggestion:** low-to-moderate temperature for coding; allow enough output tokens for complete files; have Qwen produce patches rather than retranscribing the project. There is no proven universal sampler configuration for this specific IQ3 quant; tune to your local deployment.

---

## 14. Originality checklist

- [ ] Different title, vehicle silhouettes, scenery, color language, and UI.
- [ ] Original faction/enemy names and weapons, not exact copyrighted designs.
- [ ] New level geometry and procedural fork patterns.
- [ ] No ripped sprites, sound, screenshots, fonts, logos, or music from Spy Hunter.
- [ ] Credits/licenses for any third-party assets included.
- [ ] No claim that this is an official sequel or remake.

Mechanics are inspiration; expressive audiovisual content must be our own.

---

## 15. Master handoff prompt — copy into Qwen

```text
You are the lead Godot 4.x gameplay engineer for SHADOW INTERCEPT,
an original top-down vehicular combat arcade game.

Read shadow-intercept-godot-game-plan.md in full.
Read TODO.md if present; otherwise create it from the plan with unchecked tasks.
Inspect the existing Godot project and verify current progress against TODO.md.

TASK: Resume the first incomplete task of PHASE [NUMBER] ONLY,
including its acceptance tests. For a new project, begin Phase 0.

CONSTRAINTS:
- Godot 4.x (inspect installed version), GDScript, real .tscn scenes.
- Keep existing working features. Do not rewrite unrelated files.
- Respect architecture, coordinate system, collision ownership and scope in plan.
- Use simple procedural original placeholder visuals unless phase calls for polish.
- No external copyrighted assets, no copying Spy Hunter art, sound or branding.
- Small, understandable code files; avoid speculative APIs.
- Do not implement future phases or claim unrun tests passed.
- When uncertain, examine existing code and official Godot docs.
- This plan is the specification; TODO.md is the progress record.
- Never create a second status file or silently change this plan.
- Do not skip unfinished tasks or mark an unverified task complete.
- Reopen completed tasks if regression evidence appears.

WORKFLOW:
1. Read TODO.md, inspect existing code, select the next unchecked unblocked task,
   and give a compact list of files to change and verification required.
2. Implement this task within the active phase only.
3. Run Godot headless import / project launch checks if Godot is available.
4. Fix failures and report every test actually run (command + result).
5. Update TODO.md after each verified task. Preserve all checkboxes and
   history; record phase, files, tests, unresolved problems and next action.
6. For phase completion, verify every Phase N Completion Gate checkbox.
   Set READY FOR REVIEW, not ACCEPTED, until I approve the phase.
7. Stop and wait for my approval before the next phase.

OUTPUT: Brief summary, changed file list, test results, known issues,
and one suggested next step, matching TODO.md. If a GUI playtest was not performed,
state it explicitly.
```

### Quick correction prompt if implementation wanders

```text
Stop feature development. Compare current work against the acceptance
criteria for the active phase in shadow-intercept-godot-game-plan.md.
Fix only concrete compile/runtime errors and missing phase requirements.
Do not add later-phase features. Run available tests, update status, and stop.
```

---

## 16. Research basis

The 1983 arcade game's defining ideas include a vertically scrolling armed-car pursuit, civilian penalties, continuously increasing score while driving on-road, limited specials obtained from a supply van, road forks, and water/boat sequences. We use those high-level gameplay concepts with original design and assets. Sources:

- [Spy Hunter — gameplay and mechanics](https://en.wikipedia.org/wiki/Spy_Hunter)
- [Godot 4 official documentation](https://docs.godotengine.org/en/stable/)

**Recommended starting point:** Complete Phases 0–5 first. If that core loop is not fun, later specials, forks and boats will not fix it.
