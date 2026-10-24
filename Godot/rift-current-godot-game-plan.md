<!--
Type: Phased game plan
Target: Godot 4.x, GDScript
Output: Complete Godot game project
Related: None
-->

# RIFT CURRENT — Godot Game Plan

> Original top-down arcade flight game inspired by *River Raid* (1982). Implementation target: Godot 4.x stable, GDScript, Windows/Linux, 60 FPS, Qwen 3.8 Flash Next IQ3 (64K context). This document is the immutable design specification; `TODO.md` will be created by the implementation agent as the live progress tracker.

## 1. Reference research and design boundaries

**Historical facts (Atari 2600 version).** *River Raid* was designed by Carol Shaw, developed and published by Activision, and released in December 1982. It is a vertically scrolling shoot-'em-up. A jet travels along a changing river, steers left/right, changes forward speed, and fires missiles. Banks, enemies, and fuel exhaustion destroy a jet. Flying over fuel depots refills fuel; destroying depots instead earns points. Bridges divide river sections, and a destroyed bridge governs the next section/restart position. Fuel drains at a constant rate independent of flight speed, refueling is more effective when flying slowly, and fuel below one quarter triggers an alert. The 2600 manual lists tanker 30, helicopter 60, fuel depot 80, jet 100, bridge 500 points; it also describes three reserve jets at the start and an extra reserve every 10,000 points. Difficulty switches affect straight versus steered missiles. Later platform versions add features, including attacking helicopters; these should **not** be mistaken for universal 2600 mechanics.

**What makes it work:** constant forward pressure; riverbank precision; fast but readable encounters; tactical fuel-versus-score choice; bridges as memorable goals; a progressively tighter route; one-button combat. The original is score-driven rather than a modern finite campaign.

**Our original design choices, not historical claims:** fixed-length sector chunks, deterministic seeded generation, six-sector campaign loop followed by endless escalation, one standard unguided bolt, three new enemy archetypes, and a rechargeable emergency burst introduced late. The exact speeds, scores, spawn intervals, and difficulty tables below are new balancing targets.

References: [Wikipedia: River Raid](https://en.wikipedia.org/wiki/River_Raid), [AtariAge transcription of Activision's Atari 2600 manual](https://www.atariage.com/manual_html_page.php?SoftwareLabelID=409), [Activision manual scan](https://www.ataricompendium.com/archives/manuals/vcs/river_raid.pdf). Accessed 2026-10-22.

## 2. Original game identity

**Title:** RIFT CURRENT

**Pitch:** Pilot a nimble salvage interceptor through a shifting storm-cut canyon. Thread narrow waterways, clear hostile drones, decide whether to harvest volatile charge pods or shoot them for points, and break shield gates before the current swallows you.

**Tone:** luminous teal water, slate canyon walls, amber warning lights, chunky original vector/pixel-inspired silhouettes, punchy synthesizer percussion. Fictional drone craft, not historical aircraft. Original hand-drawn/procedural sprites, UI, names, code, sound and music only. No River Raid art, trademark, map, sound or level recreation.

**Player fantasy:** survive one more sector through route reading and risk management. No story scenes, shops, inventory, online leaderboards, multiplayer, or bosses in v1. These omissions protect the small-model implementation budget.

## 3. Fixed product contract

| Item | Decision |
|---|---|
| Engine | Current installed Godot 4.x stable; record exact binary version in `TODO.md`; validate version-sensitive APIs with official docs |
| Renderer | Compatibility, 2D; no external plugins |
| Window | Logical 480×800 portrait gameplay, desktop window 720×900; aspect-preserving canvas scaling with dark sidebars and a fixed 480×720 playfield plus 80px HUD |
| Coordinates | +Y downward; player near y=625; terrain moves downward as camera advances; world distance increases upward along the route |
| Gameplay | 60 FPS target; physics at 60 Hz; delta-based movement |
| Inputs | Left/A, Right/D steer; Up/W accelerate; Down/S brake; Space/J fire; Escape pause; Enter confirm; R restart at game-over; M mute |
| Optional controller | Left stick steer/throttle; south face button fire; Start pause, implemented only after keyboard passes |
| Persistence | `user://rift_current_save.json` for high score and settings only; atomic-ish temp-write-then-rename strategy or guarded FileAccess write with backup; corrupt file defaults safely |
| Randomness | `RandomNumberGenerator` with integer `run_seed`, logged to debug overlay; sector N determined by seed + N |
| Play sessions | Arcade run until all lives lost, with sectors cycling and scaling indefinitely; no victory screen |

### 3.1 Gameplay tuning constants (single `GameBalance.gd`)

| Parameter | v1 value |
|---|---:|
| Steering speed | 260 px/s, instant response |
| Base scroll speed | 170 px/s |
| Brake scroll speed | 110 px/s |
| Accelerate scroll speed | 255 px/s |
| Speed interpolation | 5 per second |
| Fire interval | 0.18 s; one projectile per interval while held |
| Projectile speed | 580 px/s relative to screen |
| Fuel capacity | 100 |
| Fuel drain | 5 units/s while active, independent of speed |
| Refuel | 30 units/s during body overlap with an intact pod; no instant refill |
| Warning | 25 units or less; beep no more than once per 1.2 s |
| Starting lives | 3 total craft (not three extra reserves) |
| Extra life | at 10,000, 20,000, etc.; max 9 total lives |
| Respawn protection | 1.5 s flashing and collision immunity |
| Sector length | 2400 world pixels; gate near sector end |
| Bank safe margin | player collision radius 13px plus 4px |
| World cleanup | despawn objects >100px below gameplay area; cap active enemy count at 16 and projectiles at 32 |

Balance values are design targets, not references to the original. All numerical parameters belong in `GameBalance.gd`; do not duplicate magic numbers.

## 4. Rules and system specification

### 4.1 Game loop and states

`BOOT → MENU → PLAYING ↔ PAUSED → DYING → RESPAWNING → PLAYING`, or `DYING → GAME_OVER → MENU/PLAYING`. The sector gate marks a checkpoint once destroyed and passed. Pause freezes simulation timers, enemies, scrolling, fuel drain and shooting, but UI remains responsive. The `RunController` owns state changes and score/lives/fuel events; it never performs movement or terrain drawing.

**Start:** choose Play; create seed, set score=0/fuel=100/lives=3, spawn player, generate first two chunks. **Frame:** read input, move sideways, alter scroll speed, advance world distance, move spawned objects, emit shots, resolve hits, fuel, score and gate progression, recycle offscreen entities. **Failure:** wall contact, enemy collision, projectile hit or fuel reaching zero consumes exactly one life. If lives remain, restore fuel=60 and spawn at a safe corridor position associated with the latest reached gate, not at the exact crash coordinates. If none remain, submit high score and display Game Over.

### 4.2 Player and collision

Use `CharacterBody2D` with `CollisionShape2D` CircleShape2D radius 13, a `Polygon2D` placeholder graphic, optional `Area2D` fuel detector, and `Timer` fire cooldown. Horizontal movement is direct and clamped to the gameplay bounds; the *river boundaries* are checked by `RiverCourse.is_safe_x_at_y(player.global_position.x, player.global_position.y, radius)` against interpolated banks at the player world row. Never let screen clamp stand in for bank collision. Up/down adjust world scroll speed only; craft does not travel freely around the entire screen. One visible damage trigger per death through `RunController.request_player_death(reason)`, guarded by state and invulnerability flag. Input ignored in DYING, RESPAWNING and GAME_OVER.

### 4.3 World and procedural river

Use a fixed-width 480px playfield, bank samples every 80 world pixels; interpolate linearly between adjacent samples. Begin with river center x=240 and half-width=145. For each subsequent sample: center changes by bounded random ±35, half-width by ±18; clamp half-width to 82–174 and center so banks remain ≥12px from screen sides. Consecutive shifts must be smoothed by interpolation. At sectors 3+, occasional short islands are drawn, but **island collision is deferred until Phase 7**. Bank render draws water polygon and opposing canyon bands from the same bank data used for collision. Do not independently draw and collide different curves.

Generate next chunk before its leading edge enters the viewport; retain one trailing chunk until all related objects leave the screen. Sector/chunk uses monotonically increasing `world_start_y` equivalent distance values; `distance` increases as player advances. For screen Y, compute `screen_y = player_anchor_y - (object_world_distance - run_distance)` (objects ahead have smaller screen Y). Maintain one coordinate convention everywhere. Chunks and spawn plans are determined by a local RNG seeded with a stable hash of `(run_seed, sector_index)`; never use global random calls for world generation. Enforce a continuous passable route: at least 120px opening around one navigable corridor at each sample and avoid placing blocking enemies/fuel all in the same cross-section. If generated layout fails validation, reroll up to 5 times then fall back to a wide safe segment. Use deterministic manually defined starter chunk for the first 800 distance units.

**Gate:** one breakable shield gate near the final 200px of each 2400px sector. It spans 70% of water width with one visible gap of at least 85px; shooting its central generator (3 HP) disables the gate and gives score, while maneuvering through the gap allows survival but no gate bonus. Passing a disabled or intact gate advances the sector; only destruction establishes the latest checkpoint. If a gate is intact, collision with its solid arms destroys player. Avoid hard-locking progression behind a missed gate.

### 4.4 Enemies, hazards, combat

| Type | Behavior | HP | Collision | Score | First phase |
|---|---|---:|---|---:|---|
| Drift skiff | Traverses slowly from one river-side safe waypoint toward another, reverses at waypoint; no attacks | 1 | Fatal | 40 | 4 |
| Patrol rotor | Moves laterally with sinusoidal offset of 40px, period 2.2s; no projectile initially | 1 | Fatal | 75 | 7 |
| Sentry buoy | Stationary within river, flashes 0.5s then fires one downward bolt, 2.5s cooldown | 2 | Fatal | 110 | 8 |
| Charge pod | Stationary pickup: continuous fuel while overlapping; can be destroyed | 1 | Harmless to touch | 50 if destroyed | 5 |
| Shield gate | Two solid arms plus 3-HP center generator; solid while enabled | 3 | Fatal on arms | 400 if disabled | 6 |

Player bolt travels upward and damages first overlapped hostile target; no piercing. Enemy bolt travels down at 220 px/s. Player shots never hurt player; enemy shots never hurt enemies. `Area2D` monitoring and explicit collision masks; queued deletion via `call_deferred`/`queue_free` only after one-shot `destroyed` guard. All entities emit `destroyed(score_value, world_position)`; the run controller credits score exactly once. Spawn each entity at least 130px away from another blocking object in the same vertical band; preserve a safe lane. Do not spawn on player current position. Difficulty only changes spawn counts and pattern speed, not unfair unavoidable geometry.

### 4.5 Difficulty and progression

| Sector | River min half-width | Max hostiles per 2400px | Fuel pods | Scroll adjustment | New element |
|---|---:|---:|---:|---:|---|
| 1 | 125 | 5 | 4 | +0 | Starter skiffs |
| 2 | 112 | 7 | 4 | +5 | More lateral movement |
| 3 | 102 | 8 | 3 | +10 | Rotors; islands |
| 4 | 94 | 9 | 3 | +15 | Sentry buoys |
| 5 | 88 | 10 | 3 | +20 | Mixed formations |
| 6+ | 82 | min(14, 10 + floor((sector-6)/2)) | max(2, 3-floor((sector-6)/5)) | min(35, 20+2*(sector-6)) | Endless escalating cycle |

Never generate fewer than two reachable fuel pods in a sector, and ensure a fuel pod within the first 700px of every sector after sector 1. Check placement against banks and gate geometry. Score bonuses: skiff 40, rotor 75, buoy 110, pod destruction 50, gate 400; +250 for reaching every third sector. Fuel strategy is the main tradeoff: shooting a pod removes its future refill potential.

### 4.6 HUD, menus, presentation, accessibility

HUD: score, best score, sector, three-to-nine life pips, fuel bar with numeric %, low-fuel warning, pause button hint. Main menu: Play, Controls, Settings, Quit. Settings: master volume, music volume, sound volume, screen shake toggle, flash reduction toggle; save immediately on changes. Pause: Resume, Restart Run, Settings, Menu. Game Over: final score, best, sector, seed, Retry, Menu. Keyboard navigable buttons, legible 18px+ text at logical resolution; warning is conveyed by text and audio, not color alone. Avoid rapid flashing; reduced-flash disables blink effects. Decorative water particles and short explosions are added only after functional MVP.

Audio should use original synthesized tones or independently produced clips: fire tick, impact burst, refuel chirp, low-fuel two-tone, gate crack, menu confirm and looping restrained synth backing. Muting disables both music and effects; no externally sourced copyrighted assets.

## 5. Scene architecture and ownership

```text
rift_current/
  project.godot
  rift-current-godot-game-plan.md
  TODO.md                         # agent creates at implementation start
  scenes/
    main.tscn                      # Root + menus + run world switch
    world.tscn                     # RunController, RiverCourse, EntityRoot, player, HUD
    player.tscn
    entities/skiff.tscn rotor.tscn buoy.tscn charge_pod.tscn gate.tscn
    entities/player_bolt.tscn enemy_bolt.tscn
    ui/main_menu.tscn pause_menu.tscn game_over.tscn hud.tscn settings.tscn
  scripts/
    game_balance.gd                # constants only
    main.gd                        # scene switching/menu delegation
    run_controller.gd              # state, life, score, sector, signals
    river_course.gd                # bank samples, render and world-distance queries
    sector_generator.gd            # deterministic chunk plans + safety validator
    player.gd                      # input, movement, cooldown
    entity_base.gd                 # shared HP/destroy-once rules; no deep hierarchy
    enemy_skiff.gd enemy_rotor.gd enemy_buoy.gd
    charge_pod.gd gate.gd projectile.gd
    hud.gd settings_store.gd       # save/settings only
  resources/                       # optional .tres tuning resources, only if needed
  assets/graphics/                 # original assets after gameplay works
  assets/audio/                    # original sound/music
  tests/                            # smoke scenes and small GDScript checks
  docs/                             # only if later needed
```

**Ownership:** `Main` instantiates/frees `World`. `World` owns `RunController`, `RiverCourse`, player and `EntityRoot`. `RiverCourse` is responsible for terrain data, display and spawn plans; `RunController` tracks progress and commissions chunks but does not touch low-level collision shapes. Entities own their own movement/hit logic. `HUD` reads signals and cannot mutate gameplay. `SettingsStore` reads/writes only settings/high score. No autoload required until settings is added; then a single `SettingsStore` autoload is permitted.

**Signals:** Player `shot_requested(origin)` and `death_requested(reason)`; Entity `destroyed(points, pos)`; ChargePod `fuel_contact(delta_amount)`; Gate `disabled(sector_index)`; RunController `score_changed`, `fuel_changed`, `lives_changed`, `sector_changed`, `state_changed`, `run_finished`. Prefer parent wiring after instantiation; no child finding siblings by absolute node path. Reject duplicate subscriptions.

**Collision layers (bit values):** 1 player body, 2 terrain/gates, 4 enemies, 8 player projectiles, 16 enemy projectiles, 32 pickups. Player mask 2|4|16; enemies 1|8; projectile masks player=4|2 and enemy=1; pickup Area mask=1. Bank collision may be queried from `RiverCourse` rather than Godot physics geometry. If using walls as collision objects, do not also trigger bank death via the sampled bank function in the same frame. Gate uses dedicated collision shapes, not terrain polygon. Collision outcomes are all routed through guarded methods.

**Lifecycle:** generator produces sector data; world instantiates entities as they approach screen; entity signals connect once; run controller handles score; objects offscreen are released; on restart the entire `World` is replaced after disconnecting UI. Do not keep raw references to freed nodes.

## 6. Phased build: one approved phase at a time

**Universal phase gate, required for EVERY phase:**

- [ ] Required features and tasks implemented as specified
- [ ] Each task individually checked by real observation or test
- [ ] `godot --headless --path . --editor --quit` imports without errors
- [ ] `godot --headless --path . --quit-after 120` launches without parse/runtime errors (adapt flags to installed Godot version)
- [ ] Scene opens and gameplay-specific manual test performed where applicable
- [ ] No regression in prior accepted behavior
- [ ] Bugs and exact commands/results recorded in `TODO.md`
- [ ] `TODO.md` updated, status READY FOR REVIEW
- [ ] User explicitly approves; only then status ACCEPTED

When a phase has no game world yet, manual testing means verify a launched window/menu rather than gameplay. Headless checks never substitute for actual interactive play. If a gate fails, keep status IN PROGRESS or BLOCKED, fix only the failure, rerun it and prior critical smoke checks; do not proceed.

### Phase 0 — Bootable project shell

**Objective:** a clean Godot project that always starts. **Dependencies:** Godot installed. **Files:** `project.godot`, `scenes/main.tscn`, `scripts/main.gd`, `scripts/game_balance.gd`, `TODO.md`, `tests/README.md` (only if useful).

**Tasks:** choose exact stable Godot 4 version and record it; create directories as used; set renderer, stretch/aspect, display, main scene, and input actions; draw placeholder menu with Play disabled or a message; create TODO from template; add run and headless test instructions. No player/world generation yet.

**Integration:** `Main` is the only startup scene; avoid autoloads. **Acceptance:** opens correct-sized window, input actions listed in Input Map, ESC exits test only when not playing, no missing resource errors. **Test:** import, headless launch, GUI launch, inspect scaling at 720×900 and a larger monitor. **Common failures:** invalid project setting key, missing scene path, deprecated API. **Done:** universal gate plus user approval.

### Phase 1 — Player steering and scroll prototype

**Objective:** responsive readable flight over a static wide practice river. **Dependencies:** P0 ACCEPTED. **Files:** `scenes/world.tscn`, `scenes/player.tscn`, `scripts/player.gd`, `scripts/run_controller.gd`, `scripts/game_balance.gd`, `scripts/main.gd`.

**Tasks:** Main Play enters World; create player graphic/collider; left/right steering; up/down change scrolling speed, display current speed/debug distance; bind pause but do not build pause menu; update distance using physics delta; clamp craft to visible playfield. Use a static blue rectangle and simple banks as placeholder. **Integration:** Player emits death_request but deaths not processed yet; RunController owns distance. **Acceptance:** consistent steering and speed over 60 seconds; no drift when keys released; debug speed changes between 110–255. **Test:** hold each key, release, test simultaneous inputs, resize. **Common failures:** speed tied to FPS, double input processing, player leaves viewport. **Done:** universal gate.

### Phase 2 — Continuous safe canyon

**Objective:** deterministic vertical scrolling river whose bank collision matches rendering. **Dependencies:** P1 ACCEPTED. **Files:** `scripts/river_course.gd`, `scripts/sector_generator.gd`, `scenes/world.tscn`, `scripts/run_controller.gd`.

**Tasks:** implement world-distance samples, smooth banks, stable seed and first 800px safe approach; sample queries for both render and collision; generate before viewport entry; prune behind; validate 120px viable corridor; fixed debug seed. For this phase touching banks only logs `BANK HIT`/resets position (real life loss P5). **Integration:** same distance drives course and future entities. **Acceptance:** 10 minutes scrolling without seams, blank rows, impossible paths or unbounded chunk growth; same seed reproduces same samples; player at bank edge reports hit accurately. **Test:** seeded long run, brake/accelerate, compare sample coordinates. **Common failures:** coordinate sign, seams, different visual and collision interpolation, repeated random generation. **Done:** universal gate.

### Phase 3 — Firing and projectiles

**Objective:** dependable one-button shooting. **Dependencies:** P2 ACCEPTED. **Files:** `scenes/entities/player_bolt.tscn`, `scripts/projectile.gd`, `scripts/player.gd`, `scenes/world.tscn`.

**Tasks:** Space/J hold-to-fire at 0.18s, visual bolt moves upward, max 32, expires on collision or bounds; instrument debug target dummy (temporary or test-only). **Integration:** Player requests bolt; World owns spawn and connects hit event, later entity health. **Acceptance:** ≤6 bolts/s, one shot per cooldown, no accumulated offscreen bolts, hits dummy only once. **Test:** tap/hold/release, shoot during speed change, inspect remote scene tree after 2 minutes. **Common failures:** physics signal fires twice, creation deferred incorrectly, deletion leaks. **Done:** universal gate.

### Phase 4 — First hostile and scoring

**Objective:** first actual dodge/shoot challenge. **Dependencies:** P3 ACCEPTED. **Files:** `scenes/entities/skiff.tscn`, `scripts/entity_base.gd`, `scripts/enemy_skiff.gd`, `scripts/sector_generator.gd`, `scripts/run_controller.gd`, `scenes/ui/hud.tscn`, `scripts/hud.gd`.

**Tasks:** deterministic skiff waypoints and spawn plans, HP=1, hit once, +40 score, simple HUD, offscreen cleanup, no life loss yet (contact shows debug collision). **Integration:** `destroyed` credited once by RunController; world can spawn/cull entities. **Acceptance:** ≥5 skiffs over initial sector, shootable, no double awards, no impossible blocking placements. **Test:** kill/contact/miss skiffs, scroll 3 sectors, check count bounded, replay same seed. **Common failures:** spawns below player, collision layer mismatch, duplicate score, riverbank overlap. **Done:** universal gate.

### Phase 5 — Fuel, lives and fail/restart

**Objective:** complete survival tension. **Dependencies:** P4 ACCEPTED. **Files:** `scenes/entities/charge_pod.tscn`, `scripts/charge_pod.gd`, `scripts/run_controller.gd`, `scripts/player.gd`, HUD, `scenes/ui/game_over.tscn`, `scripts/main.gd`.

**Tasks:** fuel drains 5/s, touch pod refuels 30/s while overlapping, shot pod grants 50 but eliminates refill, low-fuel sound/text placeholder, collision and exhaustion each lose exactly one life, 1.5s protected respawn, three starting lives, game-over restart. Trigger respawn at a validated safe river x, clear nearby hostiles/bolts. **Integration:** centralized death guard; fuel pod emits overlap supply to RunController; game-over screen calls Main to restart World. **Acceptance:** fuel never <0/>100; pad refill and destruction mutually exclusive; one collision costs exactly one life; restart resets state and deterministic seed can be reproduced. **Test:** drain fuel, refuel slowly/quickly, intentionally crash, chain-contact during protection, lose all lives, retry 5 times. **Common failures:** zero-fuel duplicate death, refueling after pod destroyed, unsafe respawn, stale HUD. **Done:** universal gate.

### Phase 6 — Gates, sectors, menus: MVP

**Objective:** fun complete arcade loop. **Dependencies:** P5 ACCEPTED. **Files:** `scenes/entities/gate.tscn`, `scripts/gate.gd`, `scripts/sector_generator.gd`, `scripts/run_controller.gd`, `scenes/ui/main_menu.tscn`, `scenes/ui/pause_menu.tscn`, `scripts/main.gd`, HUD.

**Tasks:** gates every 2400px with 85px survivable gap; central generator 3 HP, +400 and checkpoint if disabled; sector advances on passing gate; respawn uses last cleared gate; main/pause/game over/retry menus; score extra life each 10K once, cap 9; sector-clear bonus every third sector; difficulty values for sectors 1–2; ensure Pause truly freezes world. **Integration:** Gate emits disabled once, RunController tracks checkpoint; menus only send commands. **Acceptance:** full loop from main menu to surviving sectors 1–3, deaths, checkpoints, high-score-on-run session (nonpersistent yet), pause and retry works. **Test:** 15-minute interactive session, destroy and bypass gates, die both sides of checkpoint, gain bonus life, pause while bullet exists. **Common failures:** gate traps entire channel, checkpoint set before gate destruction, menu restart duplicates world. **Done:** universal gate + explicit **MVP milestone**. Do not move to P7 until user accepts playable MVP.

### Phase 7 — Rotor enemies and islands

**Objective:** add variety without losing readability. **Dependencies:** P6 ACCEPTED. **Files:** `scenes/entities/rotor.tscn`, `scripts/enemy_rotor.gd`, `scripts/river_course.gd`, `scripts/sector_generator.gd`.

**Tasks:** rotor sinusoidal ±40px path with safe bank clamp, 1 HP/75 points; islands from sector 3 with collision using same sample geometry and a minimum alternate corridor of 120px; deterministic spawn clearance and fairness validator. **Integration:** shared entity death/score, river query now considers islands. **Acceptance:** island navigation fair, enemies remain within corridor, no geometry mismatch or duplicated death; seeded replay stable. **Test:** deliberately skirt each island, 10-minute sector 3+ run, replay seeds, 120px corridor assertions. **Common failures:** island visually passable but collision blocked, rotor crosses bank. **Done:** universal gate.

### Phase 8 — Sentry buoys and escalation

**Objective:** richer pressure over long runs. **Dependencies:** P7 ACCEPTED. **Files:** `scenes/entities/buoy.tscn`, `scenes/entities/enemy_bolt.tscn`, `scripts/enemy_buoy.gd`, `scripts/projectile.gd`, `scripts/sector_generator.gd`, `scripts/game_balance.gd`.

**Tasks:** buoy warning 0.5s, single downward shot every 2.5s while visible, 2 HP/110 pts; apply exact sector difficulty table and enforce safe fuel placement, projectile cap, enemy cap, spawn separation; optional HUD sector banner. **Integration:** enemy bolt layer 16 collides only player; death guard unchanged. **Acceptance:** warnings visible, projectiles predictable, ten sectors playable without impossible wall+enemy lock, object counts bounded. **Test:** stand in shot line/dodge, rapid restarts, sector 6+ soak and seed reproduction. **Common failures:** bolts spawn inside player, too many enemies, warning tied to framerate. **Done:** universal gate.

### Phase 9 — Persistence, settings, accessibility

**Objective:** polished user controls and lasting best score. **Dependencies:** P8 ACCEPTED. **Files:** `scripts/settings_store.gd`, `scenes/ui/settings.tscn`, HUD, menus, `project.godot`.

**Tasks:** versioned JSON `{schema:1, high_score:int, settings:{master,music,sfx,shake,reduce_flash}}`; validate types/clamp, recover from malformed/missing save, protect high score, add keyboard navigable settings and mute; add controller inputs only if straightforward and tested. **Integration:** one SettingsStore autoload, RunController calls record_high_score, UI subscribes to changed settings. **Acceptance:** save persists across restarts, invalid JSON safely resets without crash, volume settings immediately apply, reduce_flash honored. **Test:** fresh run, score record, restart program, corrupt test save copy, keyboard-only menus, unplug/replug controller if supported. **Common failures:** file path confusion, overwriting high score, pause menu input leak. **Done:** universal gate.

### Phase 10 — Original art/audio and release

**Objective:** shippable desktop build without feature creep. **Dependencies:** P9 ACCEPTED. **Files:** original `assets/graphics/*`, `assets/audio/*`, scene presentation resources, `export_presets.cfg` (do not store signing secrets), optional small test scripts, README.

**Tasks:** replace primitives with original consistent sprites/shapes while retaining collision sizes; water movement, explosions, optional slight shake, UI refinements, original SFX/music; profile, cap counts, test 720×900/1080p/1440p; export Windows and Linux, include credits and control guide. Avoid new game modes. **Integration:** art is skin over tested collision/gameplay logic. **Acceptance:** 60 FPS average on modest desktop, no sustained spikes >33ms in normal play, no runaway nodes/memory over 30 minutes, no high-severity bug, both exports boot and play. **Test:** 30-minute soak; 10 repeated restarts; input and sound toggles; exports run on target OS if available, otherwise record untested target honestly. **Common failures:** sprite pivots differ from collider, missing export templates, missing assets in exported package. **Done:** universal gate plus final user release approval.

## 7. Testing discipline

Run from project root; adapt executable path (`godot` or `godot4`) and CLI flags after `godot --help` and official 4.x docs. Never interpret a zero exit code from import as proof the game is playable.

```bash
godot --version
godot --headless --path . --editor --quit
godot --headless --path . --quit-after 120
godot --path .
```

**Automated smoke tests to add gradually:** data seed determinism; bank interpolation and passability; collision flags/layer masks; projectile only one hit; award score once; no score on no-kill; fuel bounds; threshold extra-life awards once; respawn immunity; checkpoint behavior; save parse and clamp; high score never decreases. GDScript test runner can be plain `SceneTree` script launched with `--script`, not a third-party framework. Record commands, Godot version, exit codes, error counts, and screenshots/observations if GUI-tested.

**Manual verification matrix:** keyboard diagonal combinations, fire rate, wall collision at both edges, fuel overlap at all scroll speeds, pod shot versus refuel choice, hostile hit, enemy bolt, gate destroy versus bypass, bank/island collision, death during invulnerability, dying with 0 fuel, pause while moving, game over, replay, resize, settings persistence, 30-minute memory soak, exported application launch. If impossible to test a target platform, mark NOT TESTED; never pass it silently.

**Regression rule:** after any change, rerun headless import, launch, and prior phase's critical play scenarios. If broken, restore functionality with minimal scoped patch, reopen affected TODO task, and rerun. Never rewrite whole scripts/architecture merely to repair a local issue.

## 8. Risk register and mitigation

| Risk | Protection |
|---|---|
| River drawing differs from collision | One sampled bank representation for both; edge tests in P2 |
| Scroll direction reversed | One world-to-screen formula and seeded debug markers |
| Impossible procedural layouts | Width/corridor validator, spawn clearance, fallback wide chunk |
| Ghost collisions and duplicate death | Run state guard and respawn immunity; single death entry point |
| Duplicate score from Area2D callbacks | Per-entity `destroyed` latch, queued freeing |
| Infinite entities/projectiles | Visible-distance culling and hard caps |
| Fuel pod exploited or instant killed | Overlap refuel and shot destruction mutually exclusive |
| Gate/checkpoint soft-lock | Always passable gap; checkpoint only on destroyed generator |
| Changing Godot API | Pin/record installed version; check official docs, import each phase |
| Model forgets context | Plan immutable, TODO live, exact phase gates, short task scope |
| Scope creep | No stores, bosses, narrative, multiplayer, external add-ons |
| Copyright | New title, shapes, tunes, sounds, enemies and route algorithm |

## 9. TODO.md — complete initial template (create only at Phase 0)

The agent must copy/adapt the following template into root `TODO.md` upon beginning work. **Do not create `TODO.md` while generating this plan.** The plan remains the design authority; TODO is the only progress authority. There must be no `IMPLEMENTATION_STATUS.md` or parallel tracker. Every unchecked task remains open until verified. An accepted phase remains accepted unless regression requires reopening it and documenting why.

```markdown
# RIFT CURRENT — Development TODO

Specification: rift-current-godot-game-plan.md
Godot version: NOT RECORDED
Run seed for latest reproducible test: NONE
Current phase: 0
Last accepted phase: NONE
Phase status: IN PROGRESS
Build status: NOT TESTED
Next action: Set up Phase 0 Godot project and baseline validation.

## Status vocabulary
- IN PROGRESS: unfinished or repairing defects
- BLOCKED: dependency or failing test prevents further work
- READY FOR REVIEW: all gates tested; waiting for explicit user approval
- ACCEPTED: user explicitly approved this phase; record date and approval note

## Phase checklists
### P0 — Bootable project shell [IN PROGRESS]
- [ ] Exact Godot version recorded
- [ ] Main scene/project settings and input actions configured
- [ ] Placeholder start UI works; README/test invocation documented
- [ ] Completion gate passed; user approval recorded
### P1 — Player and scrolling [NOT STARTED]
- [ ] Player controls and speed tested
- [ ] Distance updates using delta; viewport boundary tested
- [ ] Completion gate passed; user approval recorded
### P2 — Continuous canyon [NOT STARTED]
- [ ] Sampled bank render/collision unified
- [ ] Seeded generation, safe corridor, cleanup verified
- [ ] Completion gate passed; user approval recorded
### P3 — Firing [NOT STARTED]
- [ ] Cooldown, projectile motion, hit-once and cap verified
- [ ] Completion gate passed; user approval recorded
### P4 — Skiffs and score [NOT STARTED]
- [ ] Deterministic enemies, collision and one-time score tested
- [ ] HUD and cleanup verified
- [ ] Completion gate passed; user approval recorded
### P5 — Fuel and lives [NOT STARTED]
- [ ] Fuel drain/refill/pod destruction tested
- [ ] Lives, guarded death, respawn and game over tested
- [ ] Completion gate passed; user approval recorded
### P6 — Gates and MVP [NOT STARTED]
- [ ] Gates, sector advance and checkpoints tested
- [ ] Menus, pause/retry, extra life and MVP playtest passed
- [ ] Completion gate passed; user approval recorded
### P7 — Rotors and islands [NOT STARTED]
- [ ] Rotor and island behavior/fairness tested
- [ ] Completion gate passed; user approval recorded
### P8 — Buoys and difficulty [NOT STARTED]
- [ ] Enemy fire and warnings tested
- [ ] Full scaling and spawn caps tested
- [ ] Completion gate passed; user approval recorded
### P9 — Save/settings [NOT STARTED]
- [ ] Versioned high score/settings save-load tested
- [ ] Keyboard/accessibility and corruption recovery tested
- [ ] Completion gate passed; user approval recorded
### P10 — Release polish [NOT STARTED]
- [ ] Original visual/audio replacements complete
- [ ] Soak/performance/export verification complete
- [ ] Completion gate passed; final user approval recorded

## Reusable phase completion gate — copy under active phase
- [ ] Required features implemented
- [ ] Every individual task verified
- [ ] Headless import clean
- [ ] Script parse and headless startup clean
- [ ] GUI/gameplay verified
- [ ] Critical regression tests passed
- [ ] Known issues and limits recorded
- [ ] Test commands/results recorded
- [ ] TODO updated and status READY FOR REVIEW
- [ ] User reviewed and explicitly approved; status ACCEPTED

## Latest test results
| Date/time | Phase/task | Command or manual steps | Result PASS/FAIL/NOT TESTED | Evidence/errors |
|---|---|---|---|---|

## Known issues and blockers
| ID | Severity | Reproduction | Owner/next action | Status |
|---|---|---|---|---|

## Technical decisions
| Date | Decision | Why | Impacted files |
|---|---|---|---|

## Session history
| Date/time | Phase | Work performed | Tests | User approval | Next action |
|---|---|---|---|---|---|

## Accepted phase history
| Phase | Date | Explicit user approval quote/summary | Verification reference |
|---|---|---|---|
```

Before every implementation session read plan + TODO + relevant project files; inspect git diff/status if repository exists, and preserve unrelated edits. At end: task checkmarks, actual verification evidence, exact next action, and only claim READY FOR REVIEW if all applicable checks pass. Never infer approval from silence.

## 10. Coding-agent execution protocol

1. Read this file in full once, TODO in full every session, then read only code relevant to the *current unchecked task*.
2. If TODO missing, current task is **P0 TODO creation**. If TODO disagrees with code, inspect/tests decide reality; document reconciliation.
3. Implement the smallest unchecked task in the current user-approved phase, typically 1–3 files per iteration. Respect existing interfaces.
4. Run version, import and headless launch commands after every task; GUI test mechanics when possible. If GUI not available, report NOT TESTED and do not claim completion gate passed.
5. Keep scripts focused (prefer <250 lines; split only when a clear responsibility emerges). Use typed GDScript where practical, no speculative abstraction.
6. Fix local failures and regression before moving on. Never overwrite completed code because a context window reset.
7. Update TODO with verified progress and failures. Provide concise report: phase/task, changed files, behavior, exact commands/results, manual tests, blockers, next action.
8. When phase is complete, set READY FOR REVIEW and **STOP**. User must approve advancing to next phase.

### Master handoff prompt for Qwen 3.8 Flash Next IQ3

```text
You are the implementation engineer for RIFT CURRENT, a Godot 4.x GDScript desktop arcade game.

Read rift-current-godot-game-plan.md COMPLETELY before coding. Read TODO.md if present. Inspect project.godot, existing scenes/scripts and git status. Do not assume this is a new project. If TODO.md is absent, create it from section 9 of the plan as the FIRST Phase 0 task. Never create IMPLEMENTATION_STATUS.md or a second tracker.

Treat the plan as fixed requirements and TODO.md as the source of implementation status. Identify the current phase, last explicitly accepted phase, next unchecked task and blocking defects. Work ONLY on one unchecked task in the currently authorized phase unless I explicitly authorize a whole phase. Do not begin the next phase even if you finish early.

Implement small, native Godot 4.x changes in focused scenes/scripts. Keep functional existing systems intact. Do not import assets, plugins, engines or external dependencies without approval. When Godot APIs may vary, check official documentation for the installed exact version and test them. Never fabricate successful tests.

After coding, run the appropriate Godot headless import and startup checks, task-specific tests and regression checks; perform GUI gameplay tests when available. Fix failures before calling the task done. Mark only VERIFIED items checked. Update TODO.md with actual command, exit code, result, known issues, decisions, session notes and next action. If GUI verification is unavailable, note NOT TESTED; do not claim the phase complete.

Return:
1. Phase and task performed
2. Files added/modified
3. Observed behavior
4. Exact tests and PASS/FAIL/NOT TESTED outcomes
5. Bugs/blockers and corrections
6. TODO.md status and next action

At a phase completion gate, set READY FOR REVIEW, request explicit approval, and STOP. Only mark ACCEPTED after I approve it. Do not work ahead.
```

### Recovery prompt

```text
STOP feature work. You have drifted or lost context. Re-read rift-current-godot-game-plan.md and TODO.md, inspect current files and git diff, and report (a) last ACCEPTED phase, (b) active phase/task, (c) verified versus unverified changes, (d) any regression, and (e) the smallest safe recovery step. Do not rewrite working code or implement future-phase features. Repair only demonstrated errors, rerun relevant Godot checks, update TODO.md with real results, then STOP for my direction if the phase gate is reached.
```

## 11. Shipping checklist

- [ ] All P0–P10 accepted, no critical bug, all high risks either mitigated or disclosed
- [ ] Main menu→run→pause→death→retry→game over reliably works
- [ ] 30-minute soak and multi-restart test logged; stable 60 FPS target on test machine
- [ ] High score/settings survive relaunch, corrupt file safely handled
- [ ] Keyboard controls and accessible warning presentation verified
- [ ] Only original/unencumbered art and audio shipped, credits included
- [ ] Windows and Linux exports launch and play; missing platform test clearly identified
- [ ] README includes engine version, launch/export steps, inputs, asset provenance
- [ ] TODO records final build version, tests, accepted approval and known limits

**Minimum viable product is Phase 6.** The game should already be replayable and enjoyable by then; Phases 7–10 add variety, persistence and polish, not a missing foundation.
