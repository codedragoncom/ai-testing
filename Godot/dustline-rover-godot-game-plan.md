<!--
Type: Phased game plan
Target: Godot 4.x, GDScript
Output: Complete Godot game project
Related: None
-->

# DUSTLINE ROVER — Godot Game Plan

**Status:** Specification v1.0 · **Engine:** Godot 4.x stable · **Language:** GDScript · **Target:** Windows/Linux desktop, 60 FPS · **Coding agent:** Qwen 3.8 Flash Next IQ3 (64K context) · **Delivery:** Original 2D arcade game

> This file is the permanent design contract. The implementation agent creates and maintains `TODO.md` as the sole live progress tracker. **Build exactly one approved phase at a time.** Never mark a phase accepted without the user's explicit approval.

## 1. Research and design lineage

**Reference:** [Moon Patrol](https://en.wikipedia.org/wiki/Moon_Patrol), a 1982 Irem side-scrolling shooter designed by Takashi Nishiyama. The arcade game moves a lunar rover continuously rightward, asks players to jump/shoot ground hazards, and fires forward and upward together to deal with airborne threats. It uses multiple scrolling background layers, alphabetic checkpoints, timed stages, two courses, score bonuses, lives, and escalating challenges.

| Reference feature | Documented behavior | Dustline Rover adaptation |
|---|---|---|
| Movement | Rover advances through scrolling lunar ground | Fixed screen-space rover; world distance advances at fixed speed |
| Jump | Avoid holes and ground obstacles | One-button jump with fixed gravity, buffered input, and visible shadow |
| Attack | Horizontal and vertical shots emitted together | One button launches a forward pulse and upward ion bolt |
| Threats | Ground craters/mines/rocks and airborne UFO variants | Fractures, spires, mines, three original drone types |
| Progression | Two courses of 26 letter checkpoints; major E/J/O/T/Z transitions | One 5-sector campaign, five major gates, optional harder loop |
| HUD | Stage time, course progress, threat indicators | Distance bar, score, lives, warning icons, sector clock |
| Bonus | Faster-than-average stages yield points | Clear-sector time bonus versus published target |
| Visual signature | Three depth layers of parallax scenery | Four original desert silhouettes at distinct speeds |
| Lives | Limited lives, extra lives at score thresholds | Three initial lives, one extra at 10k and 30k, max five |

**Fact versus design choice:** The reference column reflects the linked article. Exact speeds, dimensions, enemy hit points, spawn positions, color themes, difficulty numbers, and all Dustline Rover features below are new design decisions—not historical claims. No original sprites, sound, names, artwork, melodies, or level layouts are reused.

## 2. Original game identity

**Title:** DUSTLINE ROVER  
**Tagline:** Keep moving. Mind the gap. Watch the sky.

**Pitch:** Pilot a nimble survey rover across the wind-cut flats of a distant frontier moon. Leap broken ground, blast crystal spires, intercept raiders overhead, and race through five dangerous survey sectors for time bonuses.

**Design pillars:** (1) readable reflex choices, (2) satisfying jump-and-double-shot rhythm, (3) deterministic retryable stage chunks, (4) layered scenery with a strong sense of motion. Target 10–15 minutes for one successful campaign. Single player, no network, no monetization.

**Art:** 320×180 base canvas scaled 4× to 1280×720; crisp original chunky pixel/vector hybrid shapes, violet skies, rust-gold ground, cyan weapon effects. Near-field terrain dark and high contrast. Everything must be recognizable as custom work. No generated assets are required: draw geometry with `Polygon2D`, `Line2D`, `ColorRect`, and small original PNGs only after mechanics work.

## 3. Non-negotiable technical contract

- Godot 4.x stable, GDScript only, Godot-native nodes/resources, no addons, external packages, shaders, C#, or editor plugins required.
- Window viewport 320×180; default window 1280×720; `viewport` stretch with `keep` aspect, nearest filtering. In gameplay, logical coordinates are pixels; positive X right, positive Y down.
- Fixed simulation via `_physics_process(delta)` at 60 Hz; use `delta` for motion. A deterministic level seed is stored for a run.
- Rover visually stays at **x=72** and ground contact baseline **y=145**. Obstacles move left as **world_distance** increases. Gameplay is suspended in pause, death recovery, menus, and results screens.
- Avoid runtime `TileMap` editing for gaps. Use compact authored **segment data** (`distance`, `kind`, `variant`) and a static flat floor with explicit collidable solid intervals. A gap is a missing solid interval; rover airborne movement uses a simple vertical integrator, not `CharacterBody2D` floor snapping across holes.
- Separate numerical gameplay coordinates from rendering. Collision checks run in world-distance space so parallax/scaling do not affect hits.
- One active campaign of five sectors. Stage data are authored arrays, not open-ended procedural generation; fixed hazards ensure regression tests are repeatable.
- All entities have explicit ownership: the World creates/removes threats; entities signal hits/events; RunController owns score, lives and state; UI only observes.
- Inputs: `jump` Space/W/Up or gamepad South; `fire` J/X/Ctrl or gamepad East; `pause` Escape/P. Holding fire repeats at cooldown; jump requires a new press, with 0.10-second buffer and 0.08-second coyote tolerance. Disable jump while airborne except buffered landing.
- Pin the exact installed stable Godot version in `docs/ENVIRONMENT.md` at Phase 0; verify any version-specific API against that version's official Godot docs before coding.

## 4. Gameplay design: exact baseline values

| System | Value or rule |
|---|---|
| Base speed | 85 world pixels/s (Sector 1), rises by +8 each sector, capped at 117 |
| Sector length | 1,600 world pixels each; campaign length 8,000 |
| Rover | 18×12 collision box; center x=72, on-ground bottom y=145 |
| Jump | vertical speed -190 px/s, gravity 470 px/s²; approximate max rise 38px and airtime 0.81s |
| Ground danger | collide with obstacle/gap while vulnerable → lose one life; explosive mine also destroys itself |
| Damage cooldown | 1.5s visual blink, no second hit during cooldown |
| Recovery | 0.65s freeze; restore rover 100px before most recent passed safe marker, clear nearby threats, grant invulnerability; never spawn inside hazard |
| Fire | repeat each 0.22s when held; both pulse types fired simultaneously; maximum 8 active pulses per type |
| Forward pulse | 190 px/s relative to screen, 1 hit, range 180px; targets spires, mines, low drones |
| Upward bolt | 225 px/s upward, 1 hit, disappears at y=20; targets airborne drones |
| Score | spire 100, mine 150, scout 200, bomber 300, skimmer 250; cleared sector 500 + 25 × whole seconds under par, max bonus 1,500 |
| Lives | start 3, cap 5; award one each upon crossing 10,000 and 30,000 total score once/run |
| Par times | sectors 1–5: 20, 19, 18, 18, 17 seconds (use elapsed moving gameplay time only) |
| Continue | no mid-run paid/score-retaining continue; game over → title or fresh run |
| Pause | freezes all gameplay clocks, motion, projectile timers and spawns |

**Important geometry rule:** The ground is an interval at y=145, not an endlessly collidable rectangle. A rover is supported only when its bottom reaches y=145 **and** its world-position footprint overlaps a solid interval. A fall below y=185 causes damage. Ground-obstacle hits use AABB and continuous-enough movement: at max speed 117px/s and 60Hz, 1.95 px/frame. Projectiles swept by previous/current segment intersection or a small shape cast to prevent tunneling.

### Hazards and enemies

| Type | Visual tell | Behavior | HP / damage | First sector |
|---|---|---|---|---|
| Fracture | black chasm, amber edge | Ground gap 22–34px wide; jump it; never shootable | instant hit on fall | 1 |
| Spire | glowing angular crystal | Stationary, forward-shot destructible; can jump over | 1 HP / 1 life | 1 |
| Scout drone | small cyan triangle | Traverses from x=340 to x=-30 at y=62–95, no firing | 1 / collision | 2 |
| Proximity mine | red pulsing orb | Ground obstacle, forward-shot destructible | 1 / collision | 3 |
| Bomber drone | amber diamond | Enters at y=45–60; drops one bomb when x≈210; bomb falls 95px/s and creates a **temporary** 26px fracture for 2.5s upon hitting solid ground | 2 / collision or bomb | 4 |
| Skimmer | twin-wing silhouette | Travels left at y=110–125; vulnerable to forward shots | 1 / collision | 5 |

**AI specifications:** No complex behavior trees. Each airborne threat has a `FLYING → ATTACKING (bomber only) → EXITING → FREED` lifecycle. Scouts and skimmers have constant horizontal speeds (-65 and -90 px/s). Bomber moves -55 px/s, drops at most once, then exits. Bomb detonation may not overwrite an existing chasm; instead it produces a harmless visual flash there. Enemy spawn windows are authored to avoid unavoidable triple threats. Destroyed targets award points once (guard boolean). Expired entities never award points.

### Five sectors

1. **Dust Approach:** fractures and crystal spires; one threat at a time, generous spacing.
2. **Blue Shale:** scouts arrive overhead; alternate sky and ground pressure.
3. **Red Basin:** mines join, with paired jump-vs-shoot choices.
4. **Storm Shelf:** bombers create temporary ground hazards; warning lamp gives ≥0.75s lead-in.
5. **Night Traverse:** skimmers and mixed patterns; tight but fair final section.

Each sector is 1,600px long, assembled from authored event offsets starting at 150px and ending before 1,480px. Keep at least 120px safe lead-in/out. Never put two ground gaps within 110px; leave ≥100px between ground threats in sectors 1–2; enforce ≥65px in later sectors. At least one test run must be able to clear every sector without taking damage. Sector background palette changes at boundaries. Finishing Sector 5 yields a victory results screen and offers a **Challenge Loop** with speed +10px/s, max 127, and the same authored events. The normal release needs one complete successful campaign; challenge loop is Phase 9.

### Scoring and checkpoints

Track `world_distance` continuously and `sector_index = clamp(floor(world_distance / 1600), 0, 4)`. Checkpoints are sector boundaries at distances 0, 1600, 3200, 4800, 6400, 8000. Maintain a safe recovery marker every 200px only if marker's ±65px area has no authored ground hazard or active temporary fracture. At sector completion: freeze the sector clock, award and display the bonus, then after a 1.2s overlay resume. **No time bonus after a death in that sector** (simple and transparent). Increment score through one RunController method to prevent duplicate awards. Persist only local best score/settings, not a mid-campaign save.

### UI and accessibility

HUD: score top-left, sector/time top-center, lives top-right, progress strip directly under the top edge. Icons light for ground/air/bomber threat arriving within 1.1s. Main menu: Start, Controls, Settings, Quit. Pause: Resume, Restart (confirmation), Settings, Title (confirmation). Results: score, sector times, victory/game-over message, Restart, Title. Settings: SFX/music volume, reduced flash, high-contrast hazards, keybind display; no rebinding UI in v1. All interactions keyboard-accessible and legible at 720p. Flash effects ≤3Hz in reduced-flash mode; no essential information only by color (shape warning icons too).

### Original audio

Use generated simple synth bleeps or self-created recordings: jump chirp, dual-shot zap, impact thud, drone burst, score tick, sector-clear chime. A sparse original percussion/bass loop is optional, but may be muted. Audio buses `Master`, `Music`, `SFX`; set defaults to -8/-14/-10 dB. No commercial/game-original samples or copied tunes.

## 5. Scene ownership and architecture

```text
res://
  project.godot
  scenes/
    main.tscn               # Main root, swaps title/gameplay/results
    gameplay.tscn           # RunController, World, Rover, HUD, Effects
    rover.tscn              # Rover presentation and input/physics
    projectile.tscn         # Reused Area2D-style hit volume; 2 modes
    enemy.tscn              # Configured enemy per data kind
    hazard.tscn             # Spire/mine/temporary chasm visuals and hit zones
    ui/title.tscn
    ui/pause.tscn
    ui/results.tscn
    ui/settings.tscn
  scripts/
    main.gd                 # Screen navigation; no score logic
    gameplay/run_controller.gd # State machine, time, lives, scoring, sector gates
    gameplay/world.gd       # Authored event delivery, solid ground intervals, despawn
    gameplay/sector_catalog.gd # Static typed event arrays and validation
    gameplay/rover.gd       # Jump, grounded state, hit request and shot emission
    gameplay/projectile.gd  # Motion and target hit rules
    gameplay/enemy.gd       # Enum state and simple motion/attack
    gameplay/hazard.gd      # Type-specific contacts/destruction
    gameplay/parallax.gd    # Four drawing layers keyed to world distance
    gameplay/hud.gd         # UI observer
    ui/menu_screen.gd       # Focus, buttons, routing
    systems/save_store.gd   # Local JSON best score/settings, versioned
    systems/audio_router.gd # Plays original SFX and bus settings
  assets/graphics/          # Original art introduced in Phase 10
  assets/audio/             # Original sounds introduced in Phase 10
  tests/                    # Headless smoke scripts and scenario notes
  docs/ENVIRONMENT.md       # Engine version, commands, export setup
  docs/TEST_LOG.md          # Optional raw test logs, not progress authority
  TODO.md                   # CREATED BY IMPLEMENTATION AGENT, not now
```

**Avoid premature files:** Add each proposed file only in its owning phase; an empty directory or placeholder class is not progress.

**Scene tree:** `Main(Node)` hosts `ScreenHost(Control/Node)` and one current screen. `Gameplay(Node2D)` hosts `World(Node2D)`, `Rover(Node2D)`, `Projectiles(Node2D)`, `Enemies(Node2D)`, `FX(Node2D)`, `RunController(Node)`, `HUD(CanvasLayer)` and `PauseLayer(CanvasLayer)`. World owns spawned obstacle and enemy nodes. UI never searches for enemies. World exposes `is_ground_solid_at(distance)` and `get_safe_recovery_distance(distance)`, and emits `event_spawned`. Rover exposes `request_damage(cause)` and emits `fired(mode, origin)`, `hit_requested(cause)`. Threats emit `destroyed(kind, points, instance_id)` or `contact_requested(cause)`. RunController alone accepts/ignores damage, issues `score_changed`, `lives_changed`, `sector_changed`, `state_changed` and `run_finished`. Connect signals once in `_ready()`, avoid bidirectional scene references.

**State machine:** `TITLE → READY → RUNNING ↔ PAUSED → RECOVERING → RUNNING → SECTOR_CLEAR → RUNNING → VICTORY`, or `RUNNING/RECOVERING → GAME_OVER`. Inputs and timers have an explicit allowlist per state. `READY` starts on confirm; transitions happen only in RunController. Pause should use gameplay-specific processing flags or `get_tree().paused` with PauseLayer configured to process while paused. Never let menu callbacks process twice.

**Collision groups:** Bit 1 Rover, 2 solid hazards, 3 enemies, 4 player projectiles, 5 enemy bombs. Projectiles scan 2/3 only; rover checks 2/3/5. Ground fractures are evaluated by world interval query, not Area2D. Use collision masks consistently and rely on precise manual checks for surface support. Disable defeated hitboxes immediately before any death animation.

**Entity lifecycle:** World owns active arrays, despawns anything x<-48 or older than prescribed TTL, and clears all on restart/recovery according to explicit rules. Every new object is registered once; every freed object removed or pruned via `is_instance_valid`. Cap enemies 12, hazards 24 and each projectile mode 8. No indefinite timers attached to freed owners.

**Save:** `user://dustline_save.json` schema `{"version":1,"best_score":0,"settings":{"music_db":-14,"sfx_db":-10,"reduced_flash":false,"high_contrast":false}}`. Gracefully fall back to defaults on absent, unreadable, invalid or future schema files. Write atomically via temp path and rename where supported; do not corrupt previous valid save. Save only after a new record or settings change.

## 6. Build discipline for a 64K local model

1. At every session start read this entire plan, `TODO.md`, then inspect only the files for the current phase plus direct dependencies. Do not trust previous chat as implementation state.
2. Use a small context budget: quote current requirement IDs, relevant source signatures, failing output, and last tests; omit massive log dumps.
3. Before code, announce current phase/task and which files will change. Do not edit unrelated files.
4. Implement one task or a small logically coupled set; keep scripts preferably under 250 lines and refactor only when needed.
5. Run import/parse and phase-specific tests. Record **real** commands, exit codes, observable runtime results, and skipped manual checks.
6. If a test fails, stop, determine first reproducible error, make minimal fix, rerun previous passing regression checks; reopen tasks as needed.
7. Update `TODO.md` after evidence is collected. Distinguish **IN PROGRESS**, **BLOCKED**, **READY FOR REVIEW**, **ACCEPTED**; only user can authorize the last transition.
8. Give a concise handoff: changed paths, behaviors, commands/results, known issues, next action. Stop for approval before moving to the next phase.

## 7. Phased implementation — 11 gates

Every phase uses the following standard completion gate, in addition to phase-specific acceptance criteria:

```markdown
### Phase Completion Gate
- [ ] All required features implemented
- [ ] All tasks individually verified
- [ ] Godot imports without errors
- [ ] No GDScript parsing errors
- [ ] Gameplay tested where applicable (manual or scripted evidence noted)
- [ ] No critical regressions
- [ ] Known issues documented
- [ ] TODO.md updated
- [ ] Phase reviewed and explicitly approved by user
```

**A gate can be READY FOR REVIEW with the last checkbox unchecked.** It becomes ACCEPTED only after approval. Run smoke regressions after every phase, including Phase 0's boot check.

### Phase 0 — Skeleton and verifiable boot

- **Objective/dependencies:** Initialize clean project; none.
- **Tasks:** [ ] Create `project.godot`, `main.tscn`, `main.gd`, initial directories when needed. [ ] Configure size, stretch, nearest filtering, 60Hz physics, inputs. [ ] Create `docs/ENVIRONMENT.md` documenting installed stable version and run commands. [ ] Make boot screen show title and `Enter to start` placeholder. [ ] Add first `tests/smoke.gd` that checks root scene exists and exits nonzero on missing root.
- **Files:** `project.godot`, `scenes/main.tscn`, `scripts/main.gd`, `docs/ENVIRONMENT.md`, `tests/smoke.gd`, `TODO.md`.
- **Integration:** Main is the only configured boot scene; no gameplay objects yet.
- **Acceptance:** `godot --headless --path . --import` exits 0; `godot --headless --path . --quit-after 60` runs without script errors; windowed launch shows boot screen, keyboard focus works. Use actual executable name (`godot`, `godot4`, or absolute path).
- **Failure modes:** Missing `run/main_scene`, invalid project settings, focus swallowed, incorrect Godot version.
- **Done:** Standard gate plus recorded screen-launch observation.

### Phase 1 — Rover, jump, and deterministic world distance

- **Objective/dependencies:** A controllable rover on scrolling ground; Phase 0.
- **Tasks:** [ ] Add gameplay scene, run controller minimal `RUNNING`, rover and world scripts. [ ] Implement fixed-x rover, jump physics and safe landing. [ ] Track `world_distance += speed*delta` only when running. [ ] Draw scrolling ground tick marks and a debug distance label. [ ] Add jump state test (initial grounded, rise, return to grounded).
- **Files:** `scenes/gameplay.tscn`, `scenes/rover.tscn`, `scripts/gameplay/{run_controller,world,rover}.gd`, `tests/rover_smoke.gd`.
- **Integration:** Main starts gameplay; World supplies ground query to Rover; RunController gates motion.
- **Acceptance:** Continuous apparent travel, x≈72, jumping at least 30px, landing at y=145, no double jump, distance stops when paused through debug toggle. 5 consecutive jumps succeed.
- **Test:** Headless import; scripted physics step assertions; windowed Space key 5×; restart and compare distance reset.
- **Failure modes:** Wrong sign of gravity, time-scaled motion twice, `is_on_floor()` assumption with no collision floor, divergent coordinates.
- **Done:** Standard gate and verified jump arc.

### Phase 2 — Authored terrain fractures and collisions

- **Objective/dependencies:** Jumpable holes and recoverable life loss; Phase 1.
- **Tasks:** [ ] Add sector catalog with safe, separated fractures in Sector 1. [ ] Ground as solid intervals with missing gap intervals and visible edges. [ ] Detect falling below y=185, decrement one life, freeze 0.65s and respawn at safe marker with 1.5s invulnerability. [ ] Add debug key to jump to a chosen segment **in debug builds only**. [ ] Test no damage over flat land, a fall costs one life, no repeat damage on invulnerability.
- **Files:** `scripts/gameplay/sector_catalog.gd`, `world.gd`, `rover.gd`, `run_controller.gd`, `tests/terrain_smoke.gd`.
- **Integration:** One RunController damage function; world reset clears pending hazards.
- **Acceptance:** Cleanly visible chasms, successful jumps, life exactly -1 per failed gap, same checkpoint restores safe ground; restarting resets all state.
- **Test:** Script two predetermined traversals (jump/no jump); play first three gaps manually; log life and safe marker positions.
- **Failure modes:** One-frame landing across gap, respawning in hole, decrementing more than once.
- **Done:** Standard gate and evidence for safe recovery.

### Phase 3 — Dual shots and destructible ground targets

- **Objective/dependencies:** Core offensive rhythm; Phase 2.
- **Tasks:** [ ] Add projectile scene/script and spire hazard; [ ] spawn both shots on single fire action; repeat at 0.22s when held; [ ] forward hit spires, upward shot flies offscreen; [ ] despawn on impact/range; [ ] award 100 once per spire, no friendly fire; [ ] cap projectiles; [ ] document layer/mask setup.
- **Files:** `scenes/{projectile,hazard}.tscn`, `scripts/gameplay/{projectile,hazard,rover,world,run_controller}.gd`, `tests/combat_smoke.gd`.
- **Integration:** Projectile hit calls hazard, hazard emits destroyed event, controller scores once.
- **Acceptance:** Both shots visibly spawn in same frame, spire clears with forward pulse, score +100 once, no leaked projectiles during 5-minute idle fire.
- **Test:** Repeated hold-fire test; direct-hit and near-miss; headless entity-count bound.
- **Failure modes:** Projectile misses due to masks, scores twice, projectiles queue_free while iterating.
- **Done:** Standard gate and stable fire loop.

### Phase 4 — Air threats, damage and spawns

- **Objective/dependencies:** Mixed ground/sky choices; Phase 3.
- **Tasks:** [ ] Add scout enemy scene/script, fixed left movement and spawn data in Sector 2. [ ] Add upward-shot collisions; [ ] contact damage via RunController; [ ] cleanup offscreen enemies; [ ] threat warning icon debug indicator; [ ] test destroyed and escaped enemy accounting.
- **Files:** `scenes/enemy.tscn`, `scripts/gameplay/{enemy,world,projectile,run_controller}.gd`, `tests/enemy_smoke.gd`.
- **Integration:** World executes sector spawn offsets exactly once per run. RunController only scores destruction signals.
- **Acceptance:** Scout appears at authored positions, upward bolt destroys it for +200, contact costs one life, escaped enemy grants zero, max entities holds during loops.
- **Test:** Script expected spawn count and exact-once event; visually play three scout events.
- **Failure modes:** Duplicate spawn at frame boundary, shot intersecting wrong enemy, dangling signal connections.
- **Done:** Standard gate and reproducible mixed threats.

### Phase 5 — Five-sector data and HUD

- **Objective/dependencies:** Structured full run; Phase 4.
- **Tasks:** [ ] Author full event lists for five sectors and validate spacing. [ ] Add mines and skimmers, and bomber basics **without** crater creation until Phase 8. [ ] Implement sector gates/time bonuses and sector transition overlay. [ ] Add full HUD, score/lives/timer/progress and warning icons. [ ] Enforce three-life game over and fresh restart.
- **Files:** `scripts/gameplay/{sector_catalog,world,enemy,hazard,run_controller,hud}.gd`, `scenes/gameplay.tscn`, `tests/progression_smoke.gd`.
- **Integration:** World consults sector catalog; RunController owns clock/transition; HUD listens to signals only.
- **Acceptance:** Five sector transitions at exact distances; score and life numbers match events; par-time bonuses only if no death that sector; game over stops movement; replay resets spawn cursor and timers.
- **Test:** Headless scripted sector crossing and scoring; manual run with debug fast-forward, then a normal-speed spot check.
- **Failure modes:** Boundary event repeated, end screen reached twice, bonus after death, timer includes pause.
- **Done:** Standard gate, complete progression smoke evidence.

### Phase 6 — Menus, pause, game flow: **MVP milestone**

- **Objective/dependencies:** Enjoyable start-to-finish playable build; Phase 5.
- **Tasks:** [ ] Create title, pause, results and minimal settings screens. [ ] Wire Start/Resume/Restart/Title/Quit; keyboard focus; confirmations for destructive navigation. [ ] Support 3-life game over and victory on fifth sector. [ ] Add simple distinct placeholder shapes for every hazard and enemy and basic debug-free HUD. [ ] Complete a no-cheat play session.
- **Files:** `scenes/ui/{title,pause,results,settings}.tscn`, `scripts/ui/menu_screen.gd`, `scripts/main.gd`, `scripts/gameplay/run_controller.gd`, `scenes/main.tscn`, `tests/flow_smoke.gd`.
- **Integration:** Main owns screen lifecycle; game controller emits finish reason; UI only requests transitions.
- **Acceptance:** Start → play → pause/resume → death/game over → restart and start → finish → victory all work. Mouse optional; keyboard mandatory. Full campaign has no impossible obstacle pattern and is fun without art polish.
- **Test:** Headless state transition checks and at least one documented full manual session on Windows or Linux; game can be exited without force quit.
- **Failure modes:** Pause tree deadlock, duplicate gameplay scene, state persists into new run, focused buttons invisible.
- **Done:** Standard gate plus **explicit user MVP acceptance**. Freeze core mechanics before expanding scope.

### Phase 7 — Fairness, feel and balancing

- **Objective/dependencies:** Tight reliable controls; accepted MVP.
- **Tasks:** [ ] Add input buffer/coyote window and jump shadow; [ ] tune obstacle spacing and hitboxes; [ ] show impending warning icons ≥1.1s before encounters; [ ] adjust sector speed values and inspect par times; [ ] add reduced-flash setting to combat feedback.
- **Files:** `rover.gd`, `world.gd`, `sector_catalog.gd`, `hud.gd`, `settings.tscn`, `tests/fairness_smoke.gd`.
- **Integration:** No new enemy categories or architecture changes.
- **Acceptance:** At least three complete trial runs reviewed; no forced hits from authored combinations; buffered jump triggers within 0.10s, invulnerability visible and no repeat damage.
- **Test:** Boundary jump tests, timing calculations at 60/120 rendering FPS, manual two-speed trials.
- **Failure modes:** Coyote jump from mid-chasm, changes invalidate safe markers, overlay blocks fire.
- **Done:** Standard gate and balance notes.

### Phase 8 — Bombers and dynamic terrain

- **Objective/dependencies:** Signature advanced aerial danger; Phase 7.
- **Tasks:** [ ] Bomber drops a single bomb near x=210; [ ] bomb falls at 95px/s; [ ] on solid-ground contact creates 26px temporary fracture lasting 2.5 seconds; [ ] temporary fracture warning ring before formation; [ ] reject overlapping existing gaps; [ ] ensure respawn clears transient holes; [ ] preserve fair approach time.
- **Files:** `enemy.gd`, `world.gd`, `hazard.gd`, `sector_catalog.gd`, `tests/bomber_smoke.gd`.
- **Integration:** World solid-interval query subtracts active temporary fractures without mutating base segment data.
- **Acceptance:** One bomb per bomber, timed hole appears/expires, no permanent world corruption, one damage max during invulnerability; spawned hole never covers rover instantly without warning.
- **Test:** Run deterministic bomber case; test pause during hole TTL; restart wipes holes; stress multiple bombers.
- **Failure modes:** TTL counts while paused, hole at player feet unavoidable, base terrain permanently altered.
- **Done:** Standard gate and bomber-specific video/manual evidence.

### Phase 9 — High scores, challenge loop, settings

- **Objective/dependencies:** Replay value and persisted preferences; Phase 8.
- **Tasks:** [ ] Implement SaveStore schema v1, best score, settings; [ ] new challenge loop after victory, speed+10 cap127 and reused authored events; [ ] audio bus controls even if music not yet present; [ ] validate corrupt save fallback; [ ] clarify retained records vs reset score.
- **Files:** `scripts/systems/save_store.gd`, `scripts/systems/audio_router.gd`, `scripts/main.gd`, `scripts/gameplay/run_controller.gd`, `scenes/ui/settings.tscn`, `tests/save_smoke.gd`.
- **Integration:** Persistence accessed only by Main/SaveStore; gameplay reports final score; no autosave during run.
- **Acceptance:** Best score survives restart, invalid JSON does not crash, selected volume options survive reload, challenge loop raises speed without duplicating events or endless objects.
- **Test:** Save/load in isolated `user://` test context or backed-up test profile, truncate test save, reload; run one loop boundary.
- **Failure modes:** Bad save overwrites valid record, file permissions, repeated extra-life thresholds after loop.
- **Done:** Standard gate and persistence test evidence.

### Phase 10 — Original presentation, QA and exports

- **Objective/dependencies:** Release candidate; Phase 9.
- **Tasks:** [ ] Replace placeholders with original four-layer parallax art and coherent custom rover/drone shapes; [ ] add original self-made audio cues, particles and transitions, high-contrast hazards; [ ] verify accessibility toggles; [ ] profile 10-minute stress run; [ ] create export presets and Windows/Linux test builds; [ ] fill credits/help and test checklist; [ ] fix critical regressions.
- **Files:** `scripts/gameplay/parallax.gd`, graphics/audio assets, `audio_router.gd`, UI scenes, `export_presets.cfg`, `docs/ENVIRONMENT.md`, `tests/release_checklist.md`.
- **Integration:** Presentation observes game state; no gameplay-dependent logic hidden in art/animation callbacks.
- **Acceptance:** Stable average 60 FPS at 1280×720 on ordinary desktop; no >50ms repeatable gameplay stutter; bounded live entities; audio defaults and mute work; no external unlicensed assets; Windows and Linux exports boot, play, pause, score, finish, restart, quit.
- **Test:** Headless import and automated smoke suites; 10-minute runtime monitoring; real manual export smoke on both OSes (mark untested if unavailable); visually check at 720p and one non-16:9 display.
- **Failure modes:** Missing export templates, wrong resources in package, scaled pixel filtering, leaking audio players, unknown licenses.
- **Done:** Standard gate and signed-off release checklist.

## 8. TODO.md — mandatory template (agent copies at first implementation session)

The following is a complete **template embedded here**. Do **not** create a separate `TODO.md` during planning. At implementation kickoff, instantiate it at the Godot project root; unpack checklists using each phase's Tasks above, keeping every task unchecked until tested. `TODO.md` is authoritative for current status; this plan is authoritative for required behavior.

```markdown
# DUSTLINE ROVER — Implementation TODO

Plan: dustline-rover-godot-game-plan.md
Current phase: 0 — Skeleton and verifiable boot
Phase status: IN PROGRESS
Last accepted phase: None
Build status: NOT BUILT
Last verified commit/revision: None
Next action: Set up Godot project and verify boot

## Phase checklist
### Phase 0 — Skeleton and verifiable boot
- [ ] Create project.godot and main scene
- [ ] Configure viewport, rendering, inputs and physics
- [ ] Add title placeholder and keyboard start focus
- [ ] Record installed Godot version and environment
- [ ] Add headless smoke test
- [ ] Run import, script and windowed boot tests
#### Phase Completion Gate
- [ ] All required features implemented
- [ ] All tasks individually verified
- [ ] Godot imports without errors
- [ ] No GDScript parsing errors
- [ ] Gameplay tested where applicable
- [ ] No critical regressions
- [ ] Known issues documented
- [ ] TODO.md updated
- [ ] Phase reviewed and approved

### Phase 1 — Rover, jump and deterministic distance
- [ ] Gameplay/Rover/World scenes and scripts
- [ ] Jump, grounding, scrolling distance and visual ticks
- [ ] Jump physics and restart tests
- [ ] Nine-item Phase Completion Gate from plan Section 7

### Phase 2 — Fractures and collisions
- [ ] Author Sector 1 fracture events and solid-ground intervals
- [ ] Gap fall, life loss, recovery and invulnerability
- [ ] Safe marker and no-duplicate-damage tests
- [ ] Nine-item Phase Completion Gate from plan Section 7

### Phase 3 — Dual shots and ground targets
- [ ] Forward and upward projectiles, cooldown and caps
- [ ] Spire hazards, collision masks and exactly-once score
- [ ] Combat, cleanup and sustained-fire tests
- [ ] Nine-item Phase Completion Gate from plan Section 7

### Phase 4 — Air threats and spawns
- [ ] Scout scene, movement and authored spawns
- [ ] Upward-shot destruction, enemy contact, despawn
- [ ] Once-only spawn, hit and escape tests
- [ ] Nine-item Phase Completion Gate from plan Section 7

### Phase 5 — Five sectors and HUD
- [ ] All five event lists and spacing validation
- [ ] Mines, skimmers and pre-crater bomber movement
- [ ] Sector timer, transitions, bonuses and warning indicators
- [ ] Lives, game over, replay and sector smoke tests
- [ ] Nine-item Phase Completion Gate from plan Section 7

### Phase 6 — Menus and MVP
- [ ] Title, pause, results and minimal settings screens
- [ ] Menu navigation, confirmation and lifecycle
- [ ] Full manual campaign and state-transition tests
- [ ] MVP user review
- [ ] Nine-item Phase Completion Gate from plan Section 7

### Phase 7 — Feel and fairness
- [ ] Jump input buffer, coyote time and shadow
- [ ] Fair collision bounds, warning timings and balance
- [ ] Reduced-flash feedback and multi-run tests
- [ ] Nine-item Phase Completion Gate from plan Section 7

### Phase 8 — Dynamic bomber hazards
- [ ] One bomb per bomber and downward motion
- [ ] Temporary ground fracture and expiration
- [ ] Overlap rejection, fair warnings and recovery cleanup
- [ ] Bomb timer, pause and restart tests
- [ ] Nine-item Phase Completion Gate from plan Section 7

### Phase 9 — Persistence and challenge
- [ ] Versioned SaveStore and graceful bad-save fallback
- [ ] Challenge loop speed and event reset
- [ ] Audio settings and restart persistence tests
- [ ] Nine-item Phase Completion Gate from plan Section 7

### Phase 10 — Presentation and release
- [ ] Original parallax graphics and effects
- [ ] Original SFX/music and accessibility verification
- [ ] Profiling, credits, full regression suite
- [ ] Windows/Linux exports and real platform smoke tests
- [ ] Nine-item Phase Completion Gate from plan Section 7

For each phase, expand its last checklist item into these nine distinct
unchecked tasks before starting that phase: all features implemented; all
tasks verified; clean Godot import; no GDScript parse errors; gameplay tested;
no critical regressions; issues documented; TODO updated; user approved.
Never check the approval item on the agent's own authority.

## Phase status register
| Phase | Status | Review date / approval reference |
|---|---|---|
| 0 | IN PROGRESS | — |
| 1 | IN PROGRESS (NOT STARTED) | — |
| 2 | IN PROGRESS (NOT STARTED) | — |
| 3 | IN PROGRESS (NOT STARTED) | — |
| 4 | IN PROGRESS (NOT STARTED) | — |
| 5 | IN PROGRESS (NOT STARTED) | — |
| 6 | IN PROGRESS (NOT STARTED) | — |
| 7 | IN PROGRESS (NOT STARTED) | — |
| 8 | IN PROGRESS (NOT STARTED) | — |
| 9 | IN PROGRESS (NOT STARTED) | — |
| 10 | IN PROGRESS (NOT STARTED) | — |

Status vocabulary: IN PROGRESS, BLOCKED, READY FOR REVIEW, ACCEPTED.
For a not-started phase, annotate IN PROGRESS (NOT STARTED) until accepted
prior phase authorizes its activation; it is not permission to work ahead.

## Testing results (append, never fabricate)
| UTC timestamp | Phase/task | Command or manual scenario | Exit / actual result | Pass/Fail/Not run |
|---|---|---|---|---|

## Known issues and blockers
| ID | Severity | Reproduction | Impact | Owner/next step | Status |
|---|---|---|---|---|---|

## Technical decisions
| Date | Decision | Reason | Files impacted |
|---|---|---|---|

## Session history (append only)
| Date | Work completed | Changed files | Tests performed | Next task |
|---|---|---|---|---|

## Approval log (explicit user approval only)
| Phase | Approved by | Date | Exact approval note |
|---|---|---|---|
```

**Status transition rules:** `IN PROGRESS → BLOCKED` for unresolved blocker; `BLOCKED → IN PROGRESS` after mitigation; `IN PROGRESS → READY FOR REVIEW` only after passing required tests and non-approval gate items; `READY FOR REVIEW → ACCEPTED` **only** when user explicitly approves. Regressions reopen affected task checkboxes, change phase to `IN PROGRESS`, and record why. An agent may never infer acceptance from silence or code generation.

## 9. Full test strategy and release checklist

**Commands** (run from project root, substitute real Godot executable):

```bash
godot --version
godot --headless --path . --import
godot --headless --path . --quit-after 60
godot --headless --path . --script res://tests/smoke.gd
# As each test file exists:
godot --headless --path . --script res://tests/rover_smoke.gd
# Later repeat with terrain_smoke, combat_smoke, enemy_smoke,
# progression_smoke, flow_smoke, fairness_smoke, bomber_smoke, save_smoke.
# After Phase 10 configured presets:
godot --headless --path . --export-release "Windows Desktop" build/DustlineRover.exe
godot --headless --path . --export-release "Linux/X11" build/DustlineRover.x86_64
```

The scripted test files should extend `SceneTree`, instantiate the relevant root scene or test subject, advance physics frames, assert expected values, log one-line PASS/FAIL, and call `quit(0)` or `quit(1)`. If the selected Godot version does not support a command as written, consult its official command-line docs, record the tested alternative, and update `ENVIRONMENT.md` (do not silently skip). Headless import catches parser/resource errors but **does not prove manual inputs, art, audio, fairness or exported builds**.

**Minimum tests:** project import/load and parse every phase; start/restart state after Phase 1; jump apex/landing and held-button no-double-jump after Phase 1; gap and damage invulnerability after Phase 2; both projectiles and once-only scoring after Phase 3; spawn/despawn/masks after Phase 4; sector boundaries/bonus/game-over after Phase 5; pause/UI/complete run after Phase 6; frame rate independence and buffered jump after Phase 7; bomber TTL/overlap after Phase 8; corrupt save round-trip after Phase 9; memory/FPS and both desktop exports after Phase 10.

**Release blockers:** any parser/runtime crash, missing input, soft lock, repeat scoring exploit, unavoidable authored obstacle, restart corruption, missing export resources, missing attribution for third-party assets, or systematic <60 FPS on stated target. Non-blocking cosmetic issues must be listed honestly.

## 10. Major implementation risks and mitigations

| Risk | Prevention | Specific regression |
|---|---|---|
| World-space/screen-space mismatch | Fixed rover x, single scalar world distance and transformed hazard rendering | Jump into known chasm at distance 400 |
| Holes bridged by floor collider | Explicit ground interval query, no universal floor collider | No-jump gap causes fall every run |
| Multiple hits per frame | RunController damage cooldown and `destroyed` guard | One collision takes exactly one life |
| Spawn duplication | Monotonic authored event cursor per run, reset on restart | Same event spawns once at 60/120 render FPS |
| Impossible combos | Spacing limits, safe lead-in/out and complete manual run | Zero-hit viable path per sector |
| Temporary gap invalidates respawn | Clear dynamic fractures and choose validated safe marker | Death never respawns in hole |
| Entity leaks | Hard caps, range despawn, freed-node pruning | Stable counts in 10-minute soak |
| Pause clock drift | State-gated delta and pausable TTL | Pause 10 seconds, sector timer unchanged |
| Model context truncation | Current phase only, TODO source of truth, narrow changes | Fresh-chat resume succeeds without repeats |
| Scope growth | 11 approval gates, MVP at 6, no bosses/upgrades in v1 | Any proposed feature traces to phase/spec |

## 11. IP and shipping rules

- Use the original title **DUSTLINE ROVER** and entirely new character, terrain, enemy, UI, icon, soundtrack and SFX designs. No original Moon Patrol art, sprites, audio, logos, ROM assets, trademarks in the game title, or ripped maps.
- Mechanics such as jumping, moving terrain, scoring and parallax are general gameplay ideas; implementation and expression must be original.
- Record source/license for every externally acquired font/audio/graphic in `docs/ASSET_CREDITS.md` if external assets are introduced; avoid external assets by default.
- Export templates must match the installed Godot release. Test on Windows and Linux before stating both platforms are verified.
- **Definition of shipped:** all phases ACCEPTED; all release-blocker tests passed on actual builds; no critical issues; original asset provenance documented; exported archives launch outside the development project.

## 12. Master Qwen implementation handoff prompt

Copy this prompt into Qwen with this plan available in the project root:

```text
You are the implementation engineer for DUSTLINE ROVER in Godot 4.x/GDScript.
Use dustline-rover-godot-game-plan.md as the binding specification.
Read the ENTIRE plan before edits. Then read TODO.md if it exists; if absent,
create it from the full embedded template, expanding all 11 phase checklists.
Inspect the existing project and Git status; do not overwrite user changes.

Determine the last explicitly ACCEPTED phase and the current incomplete task.
Work ONLY on that task or the current phase if the user specifically authorizes
the whole phase. Do not begin an unaccepted next phase. Before modifying,
state the current phase, the task IDs, the file list and proposed test(s).

Implement Godot-native scenes and focused scripts with no unnecessary packages.
Preserve working code and project settings. No speculative features, refactors,
renames or extra mechanics outside the current phase. Verify version-sensitive
Godot APIs with official docs. Use original placeholder art initially.

Run the real Godot headless import and relevant script/scenario tests, then
manually check graphical behavior when a GUI is available. Do not claim manual
or export checks passed if they were not run. Fix errors and rerun regressions.
Report exact commands and exit codes, modified files, behaviors verified,
remaining issues and next task. Update TODO.md only based on evidence.

After all non-approval gate items pass, set phase READY FOR REVIEW, not
ACCEPTED. STOP and wait for my explicit approval before next phase.
If blocked, mark BLOCKED with repro steps and ask a specific question.
```

## 13. Short recovery prompt

```text
Stop feature development. Read dustline-rover-godot-game-plan.md and TODO.md.
Inspect current files and Git diff. State the last user-ACCEPTED phase, current
phase/task, actual build state and first failing check. Reconcile TODO.md with
tested evidence; do not mark generated-but-untested work complete. Restore the
last working behavior through the smallest targeted repair (preserve unrelated
edits; never hard-reset). Run import and regression tests, record results, then
continue only the active approved task. Stop for phase review when ready.
```

## 14. Research and API references

- [Moon Patrol — Wikipedia](https://en.wikipedia.org/wiki/Moon_Patrol): history, original gameplay, checkpoints, scoring, scrolling and enemies (accessed 2026-10-23).
- [Godot stable — Multiple resolutions](https://docs.godotengine.org/en/stable/tutorials/rendering/multiple_resolutions.html): viewport scaling and aspect behavior.
- [Godot stable — Using Area2D](https://docs.godotengine.org/en/stable/tutorials/physics/using_area_2d.html): collision signals for projectiles/hit areas.
- [Godot stable — Command line tutorial](https://docs.godotengine.org/en/stable/tutorials/editor/command_line_tutorial.html): headless import, scripted tests and exports.
- [Godot stable — ProjectSettings](https://docs.godotengine.org/en/stable/classes/class_projectsettings.html): display/physics/input settings. Verify property spellings against the exact installed stable release.

**First approved action:** Phase 0 only. Create `TODO.md`, initialize the project, run import, confirm windowed boot, report results and request Phase 0 acceptance.
