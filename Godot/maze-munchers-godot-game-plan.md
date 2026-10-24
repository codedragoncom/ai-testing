<!--
Type: Phased game plan
Target: Godot 4.x, GDScript
Output: Complete Godot game project
Related: None
-->

# MAZE MUNCHERS — Godot Game Implementation Plan

**Specification version:** 1.0  
**Reference:** [Pac-Man (1980)](https://en.wikipedia.org/wiki/Pac-Man)  
**Target implementation agent:** Qwen 3.8 Flash Next IQ3 (64K context)  
**Engine:** Godot 4.x stable, GDScript, Windows/Linux, 2D, 60 FPS  
**Status:** DESIGN ONLY — no game code or `TODO.md` exists yet

## 1. Reference Research and Design Boundaries

Pac-Man is a maze-chase arcade game created by Namco, designed by Toru Iwatani, and released in Japan on May 22, 1980. A player navigates a fixed maze, eats dots to clear each level, avoids four ghosts, and collects larger energizers that temporarily make ghosts vulnerable. Ghosts have contrasting pursuit behavior; a captured player loses a life, while eating successive vulnerable ghosts yields increasing points. Fruit bonuses, horizontal warp tunnels, ghost regeneration, and faster later rounds add depth. See [Wikipedia: Pac-Man](https://en.wikipedia.org/wiki/Pac-Man), especially **Gameplay**.

| Reference mechanic | Our interpretation | Source status |
|---|---|---|
| Four-direction maze traversal | Center-to-center grid navigation with queued turns | Documented inspiration; implementation chosen here |
| Clear all small dots | Collect all **crumbs** to finish a maze | Documented inspiration |
| Four ghosts with personalities | Four original **Wardens** with separate target rules | Documented inspiration; algorithms below are new |
| Power pellets | Collect **spark cores** to reverse predator/prey roles | Documented inspiration |
| Sequential ghost-eating bonuses | Chain reward 200/400/800/1600 | Documented inspiration; selected values |
| Warp passages | Two linked horizontal portals | Documented inspiration |
| Bonus fruit | Timed **relic** collectible | Documented inspiration; original art |
| Rising difficulty | Speed/time table per round | Documented inspiration; values chosen here |
| 256th-round bug | Not reproduced | Historical defect, explicitly excluded |

**Not a clone:** No Pac-Man name, maze map, ghost likenesses, character silhouettes, sprite sheets, sounds, logos, commercial fonts, or exact level layouts. Only general game rules inform the design. The finished work must be independently drawn, sounded, and named. If commercially released, perform a separate IP/trademark review.

## 2. Original Game: Maze Munchers

**Pitch:** Guide **Nib**, a glowing six-legged salvage bot, through neon circuit labyrinths, collecting energy crumbs while four security Wardens hunt it. Grab a spark core and turn the hunt around. Clear the board, race the next circuit, and chase a local high score.

**Visual identity:** Top-down, dark navy background, thick cyan/indigo circuit walls, amber crumbs, electric lime cores. Nib is a rounded hexagonal bot with animated antennae—never a yellow circular mouth. Wardens are distinct geometric drone shapes with colored cores: red diamond **Vector**, violet triangle **Oracle**, teal cross **Relay**, orange square **Drifter**. During vulnerability their outlines pulse white/blue; while returning home they are small hollow symbols.

**Audio identity:** Original synthesizer clicks for pickups, pulsing low note for danger, rising arpeggio for chain captures, short round fanfare; avoid melodies and sampled effects from the reference game.

### Scope

- Single-player offline arcade game; keyboard mandatory, controller optional.
- One handcrafted **original** 25-column × 27-row symmetrical-but-not-identical circuit maze, mirrored visually but with deliberate asymmetrical routes. Three color themes cycling by round; no random level generator.
- Four Wardens, four spark cores, one relic spawn point, two warp portals, three starting lives, score/high score, menus, pause, game over, settings.
- No bosses, weapons, online services, multiplayer, upgrades, procedural generation, or persistent mid-round saves.
- A level counts as cleared when all regular crumbs are consumed; spark cores also count toward clearance if still present. Relic is optional.

## 3. Complete Gameplay Rules

### 3.1 Timing, coordinates, layout

- Logical game window: **960 × 720**, stretch mode `canvas_items`, aspect `keep`; viewport configurable for resizing. Gameplay board occupies a 600 × 648 logical region using **24-pixel tiles**, with HUD in remaining margins. Camera is stationary.
- Grid coordinates are `Vector2i(x,y)` with origin upper-left. Tile center = `board_origin + Vector2(x + 0.5, y + 0.5) * 24`.
- Map dimensions: 25 × 27, exactly 675 cells. Define a deterministic array of 27 ASCII strings, each exactly 25 characters. Tile symbols: `#` wall, space traversable, `.` crumb, `o` spark core, `P` Nib start, `A/B/C/D` Warden starts, `H` home/repair tile, `R` relic spawn, `T` portal. Validate unknown characters, row lengths, and duplicate/missing spawn symbols before play.
- Portal cells: exactly two `T` on the same row at opposite horizontal ends, linked both ways. The game must not count portals as crumbs. Warden home is a small walkable chamber near map center with one shared legal exit. No entity may enter a wall.
- **Map authoring requirement:** Original corridors with loops, four corner regions, multiple routes between quadrants and a reachable home entrance; do not trace the Pac-Man board. Minimum 120 crumb tiles, exactly 4 cores, all collectible cells reachable from P. An automated flood-fill must validate reachability through portals.
- Board is loaded once per round, with collectible states reset between rounds. HUD and game state are not children of board tiles.

### 3.2 Player control

- Four discrete directions, no diagonal movement. Input: arrows and WASD; `Esc` pause; `Enter` confirm; `R` restart at game over (with menu confirmation or clear labeling).
- Player speed: **112 pixels/second** initially. Move along cardinal directions between tile centers. At each center: try last queued input, otherwise maintain current direction if legal, otherwise stop. Buffered direction persists until used or replaced.
- Prevent reverse-direction exploits and diagonal drifting: only choose turns within **2 pixels** of a tile center, snap exactly to center before direction change. The simulation uses `_physics_process(delta)`; do not multiply velocity by delta twice.
- At a portal center, teleport to its linked center once per entrance, preserve direction, and enforce a one-tile exit lock to prevent repeated teleporting. Collision and pickup checks happen after teleport and must not double-trigger.
- Nib has no health bar or weapon. Touching a hostile Warden costs one life; set a **1.5-second respawn grace period** after reset. During grace, collision cannot cost additional lives. Nib and Wardens reset to fixed start cells without replenishing eaten crumbs after a lost life.
- Default lives: 3 total attempts. When losing the last, transition to game over; on a remaining life, run a 1-second READY countdown before movement resumes.
- Visual animation: alternate antenna tilt and leg positions while moving, idle breathing when stopped, brief flash during grace period. Direction faces current movement.

### 3.3 Collectibles and score

| Event | Points | Behavior |
|---|---:|---|
| Crumb | 10 | Removed once; decrements remaining counter |
| Spark core | 50 | Removed once; activates vulnerability for active Wardens; decrements remaining counter |
| First vulnerable Warden | 200 | Chain index 1 |
| Second | 400 | Chain index 2 |
| Third | 800 | Chain index 3 |
| Fourth | 1600 | Chain index 4 |
| Relic | 100–500 | 100 rounds 1–2, 300 rounds 3–5, 500 rounds 6+ |

- Spark core timer: **6.0s** rounds 1–2; **5.0s** rounds 3–5; **4.0s** rounds 6–9; **3.0s** rounds 10+. Picking another core refreshes duration and resets chain index to zero. While vulnerable, Wardens can be captured; their returning forms are harmless and cannot be captured again until home regeneration.
- Vulnerable Wardens flash at **2Hz** during the final 1.5 seconds. All timeouts use game simulation time, and freeze on pause.
- Relic appears once when approximately 35% of original crumbs have been cleared and again at 70%. Spawn at `R` for 8 seconds each, unless collected earlier. Its timer pauses with game. It is never required to clear the round.
- Award one bonus life upon first crossing **10,000 points** per run; cap total lives at 5. Never repeatedly award when score remains above threshold.
- High score is a single offline integer in `user://save.json`; write atomically via temporary file + replace/rename where practical; malformed files fall back to 0.

### 3.4 Warden AI — deterministic, limited complexity

Each Warden follows `PEN`, `SCATTER`, `CHASE`, `FRIGHTENED`, `RETURNING`, or `RESPAWN`. They move one tile center to the next; choose one of legal neighbors at each center. Never choose a wall or block all movement. They should avoid reversing unless entering frightened mode, transitioning scatter/chase, reaching a dead end, or recovering from an invalid target. No full-maze A* needed for every frame.

| Warden | Visual | CHASE target | SCATTER target |
|---|---|---|---|
| Vector | Red diamond | Nib's current tile | Upper-right open corner tile |
| Oracle | Violet triangle | 3 walkable cells ahead of Nib; if blocked, nearest valid traversable cell | Upper-left open corner |
| Relay | Teal cross | Tile formed by `Nib + (Nib - Vector)`, clamped to nearest reachable tile | Lower-right open corner |
| Drifter | Orange square | Nib when Manhattan distance > 7; otherwise lower-left corner | Lower-left open corner |

**Route rule:** At an intersection, evaluate each legal neighboring tile by its squared Euclidean distance to the current target; choose minimum, ties broken in order UP, LEFT, DOWN, RIGHT. If the target is blocked, find nearest walkable tile with deterministic tie-breaking. If the greedy choice would loop on a dead-end cycle, allow a legal reverse after two repeated visits. No pathfinding plugin. `FRIGHTENED` choices use a seeded `RandomNumberGenerator` (seed at start of run for test reproducibility). `RETURNING` uses BFS from Warden's current grid position to `H` at tile centers, and must reach home within a reasonable bound.

**Modes:** After individual release, each Warden starts in SCATTER. Global schedule based on active gameplay elapsed time: SCATTER 7s → CHASE 20s → SCATTER 7s → CHASE 20s → SCATTER 5s → CHASE indefinitely. This schedule pauses during frightened mode, countdowns, or pause. `FRIGHTENED` outranks chase/scatter. After core timeout, return to the current global mode.

**Release schedule:** Vector leaves at round start; Oracle after 4s; Relay after 8s; Drifter after 12s. Later rounds shorten waits by 0.5s per round to floor 1s. Start timers on ACTIVE play, not while READY. If a Warden is captured, enter RETURNING, go to H, then remain RESPAWN for 1 second and re-enter using regular chase/scatter mode.

**Speed:** Round 1 Warden speed 88 px/s, +3 per completed round, cap 118; frightened speed 70% of normal; returning speed 150% of normal. Portal passage speed: 75% of current mode speed, only while in a portal corridor cell. Speeds are constants in a single balance resource.

**Collision resolution:** Use grid-aware proximity overlap based on center distance at or below **12 pixels** between Nib and a Warden, including tile-center crossings. On contact: if Warden FRIGHTENED → capture it; if RETURNING/RESPAWN or Nib is in grace → no effect; otherwise one lost life, halt interactions immediately, and transition DYING. A collision handler may resolve only one death or capture per Warden per physics tick. Check both current proximity and segment-crossing swaps so entities cannot pass through one another unnoticed.

### 3.5 Rounds and state machine

`BOOT → MENU → READY → ACTIVE → {DYING → READY | ROUND_CLEAR → READY | GAME_OVER}`. `ACTIVE ↔ PAUSED`. Settings accessible from MENU and PAUSED. READY duration 1s. ROUND_CLEAR duration 1.5s and adds +1 to round; reset map collectibles and spawns. DYING duration 0.8s before decrement/reset decision. GAME_OVER shows score, best, round reached, and restart/menu.

If the last collectible is consumed on the same tick as a hostile collision, **death takes precedence**. Pause is only available while ACTIVE, READY or DYING; never allow movement or score during pause. All scheduled tasks must respect state transitions; no orphan timers acting after round reset.

## 4. Technical Architecture

**Principle:** One `GameController` owns game flow, `Board` owns static topology and collectibles, `ActorMotor` owns reusable grid motion, `WardenBrain` owns AI decisions, and UI listens to signals. Avoid putting scoring, menus, tiles and AI in a single script.

```text
res://
  project.godot
  scenes/
    main.tscn              # Root, GameController, Board, Actors, HUD, UI
    board.tscn             # Tile presentation/collectibles
    nib.tscn               # Nib visual and actor movement
    warden.tscn            # Parametrized original drone
    ui/main_menu.tscn
    ui/hud.tscn
    ui/pause_menu.tscn
    ui/game_over.tscn
  scripts/
    game_controller.gd     # State transitions, round reset, score/lives
    board.gd               # ASCII parse, grid queries, pickups, portal maps
    actor_motor.gd         # Cardinal center-to-center movement
    nib.gd                 # Input, direction queue, player visual events
    warden.gd              # Warden mode, release, collision representation
    warden_brain.gd        # Target selection, deterministic turn choice/BFS return
    balance.gd             # Central constants, speed/time formulas
    save_store.gd          # High score and settings persistence
    ui/hud.gd
    ui/menus.gd
  data/maze_01.txt         # 27 original 25-character rows
  assets/graphics/         # Original generated or hand-drawn sprites later
  assets/audio/            # Original short FX/music later
  tests/test_board.gd
  tests/test_motor.gd
  tests/test_rules.gd
  docs/                    # Optional testing notes, NOT competing status file
  TODO.md                  # Created by implementation agent, not in this deliverable
```

**Scene ownership:** `main.tscn` contains the controller with references to child Board, Nib, `Wardens` node, and UI layer. Board creates wall/crumb visuals and exposes immutable map topology plus mutable collectible state. GameController instantiates four Warden instances. Nib and Wardens query Board through injected references, not globally looked-up nodes. GameController owns score and lives, resets actors, and handles collisions. Use `queue_free()` for temporary relic/popup nodes; ensure old nodes/timers do not survive reset.

**Signal contracts:** `Board.collectible_taken(kind, cell)`, `Board.remaining_changed(count)`, `Nib.cell_entered(cell)`, `Warden.mode_changed(id, mode)`, `GameController.score_changed(score)`, `lives_changed(lives)`, `round_changed(round)`, `state_changed(state)`, `high_score_changed(best)`. Events carry values, not scene references; connect signals once in `main` and disconnect for despawned instances.

**Collision policy:** Since movement is grid-authoritative, use direct board legality checks and proximity/segment-crossing for actor collisions. Collision layer 1 walls (render/optional StaticBody2D), 2 player, 3 Warden, 4 pickup areas. If actual `Area2D` detectors are added for FX, their masks are 2/3 as appropriate, but **do not use two independent collision systems to award pickups or damage**. Core rules should live in controller/board.

**Data authority:** `Balance` constants for speeds/timers; `Board` for tiles and pickups; `GameController` for score, lives, round, active state; `SaveStore` for persisted high score, controls/settings. Do not duplicate counters in HUD. Persist only after high score changes/game over or settings change, not every frame.

**No external dependencies.** Use Godot CanvasItem drawing (e.g. `_draw`) and Label/Control for placeholder art. Version-sensitive APIs must be checked against the installed Godot 4.x official docs: [Godot documentation](https://docs.godotengine.org/en/stable/).

## 5. Phased Implementation — Stop at Every Gate

**Global rule:** Only work on the current approved phase. Every phase must leave a runnable build with no parse errors. A passing headless import is NOT proof of gameplay. At completion update TODO and mark READY FOR REVIEW, then **stop for explicit user approval** before marking ACCEPTED or starting next phase.

### Phase 0 — Bootstrap and testing foundation

- **Objective:** Godot project launches to a placeholder title scene.
- **Dependencies:** None.
- **Files:** `project.godot`, `scenes/main.tscn`, `scripts/game_controller.gd`, `scripts/balance.gd`, `TODO.md`, empty test scripts as needed.
- **Implement:** 960×720 window; stretch keep; 60 FPS target; inputs `move_up/down/left/right`, `pause`, `confirm`, `restart`; root scene, startup state MENU, visible version label, documented Godot version; add minimal test runner using Godot `--headless --script` script pattern that exits nonzero on assertion failure.
- **Integration:** `main` set as main scene; no autoload until justified.
- **Acceptance:** Editor opens; game launches to title; no parse/import errors; input actions visible.
- **Test:** Run Godot headless editor import and a normal GUI session; record exact executable/version and commands.
- **Common failures:** Bad scene UIDs, invalid Godot API, missing input action. Fix, reimport, retest.
- **Done:** All gate items pass and user approves.

### Phase 1 — Maze parser and visible board

- **Objective:** Valid original maze shown on screen.
- **Dependencies:** Accepted Phase 0.
- **Files:** `data/maze_01.txt`, `scenes/board.tscn`, `scripts/board.gd`, `tests/test_board.gd`, `scenes/main.tscn`.
- **Implement:** Exact 25×27 parsing, symbols validation, at least 120 crumbs, four cores, all spawn symbols, two portals, flood-fill reachability, tile-to-screen conversion, wall/path debug drawing. Document chosen map in file, no borrowed map. Provide `is_walkable`, `neighbors`, `tile_to_world`, `world_to_tile`.
- **Integration:** GameController creates/locates Board and refuses ACTIVE if map invalid.
- **Acceptance:** No broken rows; every collectible reachable; wall graphics align; valid spawn/portal positions.
- **Test:** Unit test good map and three invalid fixtures (bad width, missing portal, unreachable crumb); visual inspect all board edges.
- **Common failures:** Spaces lost by trimming, off-by-one grid center, disconnected chamber. Preserve raw spaces, validate before drawing.
- **Done:** Gate passes and approval.

### Phase 2 — Nib movement and portals

- **Objective:** Nib navigates every corridor precisely.
- **Dependencies:** Phase 1.
- **Files:** `scenes/nib.tscn`, `scripts/nib.gd`, `scripts/actor_motor.gd`, `tests/test_motor.gd`, `scenes/main.tscn`.
- **Implement:** Cardinal buffered input, tile center snapping, wall blocks, stop/reverse/turn, portals with latch, simple procedural bot drawing. Use speed from Balance.
- **Integration:** Player emits cell entry; Board is sole walkability authority.
- **Acceptance:** No clipping/drift after 100 turns; reversal works; portal traverses exactly once; blocked direction does not cancel ongoing valid direction.
- **Test:** Keyboard live test, collision boundary test, 1000 fixed-step deterministic movement steps, portal bidirectional test.
- **Common failures:** Stutter at center, delta misuse, portal loops. Log tile and queued direction in debug mode.
- **Done:** Gate passes and approval.

### Phase 3 — Pickups, scoring and HUD

- **Objective:** Maze has a complete solo collection loop.
- **Dependencies:** Phase 2.
- **Files:** `scripts/board.gd`, `scripts/game_controller.gd`, `scenes/ui/hud.tscn`, `scripts/ui/hud.gd`, `tests/test_rules.gd`.
- **Implement:** Crumbs, cores as 50-point pickups (power effect deferred to Phase 6), remaining count, score, round counter display, sound placeholders, last-crumb round-complete event stub that displays completion then resets board.
- **Integration:** Board emits once per collectible; Controller awards score; HUD subscribes to scores/remaining.
- **Acceptance:** Pickup cannot score twice; count decrements once; all items cleared completes round; HUD consistent.
- **Test:** Collect same tile twice, last-item edge case, independent reload of board.
- **Common failures:** Double signal and stale HUD; disconnect duplicate subscriptions.
- **Done:** Gate passes and approval.

### Phase 4 — One basic Warden and lives

- **Objective:** Nib has a real opponent and lose/retry loop.
- **Dependencies:** Phase 3.
- **Files:** `scenes/warden.tscn`, `scripts/warden.gd`, `scripts/warden_brain.gd`, `scripts/game_controller.gd`, `tests/test_rules.gd`.
- **Implement:** Vector pursuer only, target current Nib tile, legal center-to-center choices, simple SCATTER/CHASE, hostile contact, DYING/READY transitions, three lives, grace period, game over placeholder.
- **Integration:** Controller resolves proximity and crossing collision; reset leaves pickups consumed.
- **Acceptance:** Enemy cannot pass walls; contacts remove exactly one life; zero lives ends game; remaining life resets positions.
- **Test:** Simulate overlapping and crossing actors, contact during grace, chase against unreachable direct vector.
- **Common failures:** Infinite death callback, cornering through walls, ghost stuck. Add state guard and deterministic direction tie rules.
- **Done:** Gate passes and approval.

### Phase 5 — Four Wardens, pursuit personalities — **MVP**

- **Objective:** Complete enjoyable maze chase with different opponent behaviors.
- **Dependencies:** Phase 4.
- **Files:** `scripts/warden_brain.gd`, `scripts/warden.gd`, `scripts/game_controller.gd`, `scripts/balance.gd`, AI tests.
- **Implement:** Four distinct original Wardens; delayed releases; scatter/chase schedule, tie-breaking, Drifter distance switch, respawn positioning, round difficulty speed scaling; pause/restart keyboard support at minimum. Ensure global state runs only during active gameplay.
- **Integration:** Four separate Warden instances share Board but have independent mode and position. Replace any temporary single-Warden hardcoding.
- **Acceptance:** Each Warden demonstrably chooses a distinct target; no impossible turns or indefinite stuck states; player can clear, die, restart, and play multiple rounds.
- **Test:** Deterministic 5-minute hands-on round, forced corner cases, pause/resume timer checks, AI target fixture tests, headless import.
- **Common failures:** All four behave identically, speed jumps on transitions, AI deadlocks. Instrument target cells and turn choices.
- **Done:** **MVP gate** passes, game is playable, and user approves before Phase 6.

### Phase 6 — Spark core combat reversal

- **Objective:** Finish signature risk/reward mechanic.
- **Dependencies:** Phase 5.
- **Files:** `scripts/game_controller.gd`, `scripts/warden.gd`, `scripts/warden_brain.gd`, `scripts/balance.gd`, rules tests.
- **Implement:** Frightened timer, flashing, seeded random legal turns, 200/400/800/1600 chain, returning BFS to H, 1-second regeneration, refresh/reset chain when another core consumed, temporary speed changes.
- **Integration:** Captures and hostile hits resolved in one place; returning Wardens cannot kill or be recaptured.
- **Acceptance:** Timer pauses/resumes; chain scoring correct; all captured Wardens reliably return; normal chase resumes.
- **Test:** Two cores in succession; capture all four; timer expires during overlap; pause with 1s left; test BFS from dead ends.
- **Common failures:** Repeated capture score, returning trapped, conflicting mode restoration. Single source of mode truth with explicit priority.
- **Done:** Gate passes and approval.

### Phase 7 — Bonus relic and full round progression

- **Objective:** Scoring variety and long-session replayability.
- **Dependencies:** Phase 6.
- **Files:** `scripts/board.gd`, `scripts/game_controller.gd`, `scripts/balance.gd`, rules tests.
- **Implement:** Relic spawn at 35/70% cleared, 8-second TTL, score bracket; round clear/death precedence; speed caps, power duration steps, bonus life once at 10k (cap 5), full round reset without leaked timers.
- **Integration:** Board owns relic occupancy, Controller owns score/threshold and round state.
- **Acceptance:** Relic never blocks clearance; 10k bonus fires once; timing and difficulty follow specification; round transitions clean.
- **Test:** Force 35/70% thresholds, high scores, dying on last crumb, ten sequential quick round resets.
- **Common failures:** Float threshold double-trigger, relic persisting across rounds, timer after free. Use integer threshold gates and generation token for resets.
- **Done:** Gate passes and approval.

### Phase 8 — Menus, settings and persistence

- **Objective:** Complete game front end and stable stored high score.
- **Dependencies:** Phase 7.
- **Files:** `scenes/ui/main_menu.tscn`, `pause_menu.tscn`, `game_over.tscn`, `scripts/ui/menus.gd`, `scripts/save_store.gd`, `scripts/game_controller.gd`.
- **Implement:** Start, pause/resume, restart, quit, score summary, high score, full-screen toggle, master audio slider, optional color-assist outlines, save/load validation. Use only `user://` for saves. Settings changes persisted at menu transitions.
- **Integration:** UI driven by state_changed and data signals; game input ignored while menu consumes focus.
- **Acceptance:** All menu paths work by keyboard; fresh install and corrupt save do not crash; high score survives restart.
- **Test:** Manual keyboard focus matrix, save/reopen, corrupt JSON fallback, pause while frightened/relic timers active.
- **Common failures:** Input reaches player under overlay, multiple save writes, focus lost on gamepad. Gate inputs by GameState.
- **Done:** Gate passes and approval.

### Phase 9 — Original presentation and accessibility

- **Objective:** Finished cohesive arcade feel, no reference-game assets.
- **Dependencies:** Phase 8.
- **Files:** `assets/graphics/*`, `assets/audio/*`, Nib/Warden/Board drawing scripts, HUD scenes.
- **Implement:** Animated bot/drone shapes; original circuit-wall treatments; wall glow with cheap 2D visuals, core pulse, score popups, distinct warning flashes, original 1–3-second synthesized FX and optional looping track. Add reduced-flash setting, separate audio volume, high contrast/shape indicators for each Warden. No strobing above 3Hz. Check HUD readability at 720p and 1080p.
- **Integration:** Presentation cannot change grid movement, hitboxes, timings, or scoring; test with all effects disabled.
- **Acceptance:** All art/audio original; Wardens recognizable without color; motion is readable; optional effects off do not affect rules.
- **Test:** Visual playthrough, color-blind/small-window review, asset audit, score parity with FX off.
- **Common failures:** Animated sprite origin shifts, huge particle nodes, licensed sound sample. Draw/anchor to grid center and audit sources.
- **Done:** Gate passes and approval.

### Phase 10 — QA, performance and desktop release

- **Objective:** Stable distributable build for Windows/Linux.
- **Dependencies:** Phase 9.
- **Files:** `project.godot`, `export_presets.cfg` (without secrets), tests, `README.md`, `CHANGELOG.md`, `TODO.md`.
- **Implement:** All regression tests, controller optional mapping if stable, export preset setup with matching Godot export templates, clean first-run experience, credits/controls, reproducible release checklist.
- **Integration:** Test exported games, not only editor F6 scene. No telemetry or network dependencies.
- **Acceptance:** 60 FPS target at 960×720 on ordinary desktop hardware; no sustained frame-time spikes >33ms under normal play; memory stable for 30 min; no critical bugs; Windows and Linux packages launch from clean directories; local save survives restart; project reopens cleanly.
- **Test:** Headless import, automated logic suite, 30-minute GUI soak, three full clear/death/restart cycles, fresh saves, each desktop export smoke test. Record hardware, Godot version, actual commands/results.
- **Common failures:** Export templates missing, path case mismatch Linux vs Windows, game pauses on focus loss incorrectly, stale resources. Fix before final sign-off.
- **Done:** Final gate passes and explicit approval. Tag v1.0 only after accepted.

### Standard phase completion gate — copy for EACH phase into TODO.md

```markdown
### Phase Completion Gate
- [ ] All required features implemented
- [ ] All tasks individually verified
- [ ] Godot imports without errors
- [ ] No GDScript parsing errors
- [ ] Gameplay tested in GUI (or a documented reason why unavailable)
- [ ] No critical regressions
- [ ] Known issues documented
- [ ] TODO.md updated with actual tests and files
- [ ] Phase reviewed and explicitly approved by user
```

**Status meanings:** IN PROGRESS = actively coding/testing; BLOCKED = needs missing tool/decision or reproducible failure; READY FOR REVIEW = checks passed but approval pending; ACCEPTED = user explicitly approved. Do not equate READY FOR REVIEW with ACCEPTED.

## 6. TODO.md — Single Source of Progress Truth

The plan is permanent specification. `TODO.md` is the only living status tracker; **do not create `IMPLEMENTATION_STATUS.md`**. When work starts, Qwen creates TODO.md from the template below and expands every phase checklist with the implementation tasks in Section 5. Keep verified history, don't erase previous decisions or failed test results. A regression reopens the affected task. Only mark checkboxes after actual verification.

```markdown
# Maze Munchers — TODO
Plan: maze-munchers-godot-game-plan.md
Current phase: 0 — Bootstrap
Current status: IN PROGRESS
Last accepted phase: None
Build status: NOT TESTED
Next actionable task: Create project.godot and main.tscn
Godot version: NOT VERIFIED

## Phase 0 — Bootstrap [IN PROGRESS]
- [ ] Set up Godot project, main scene and input actions
- [ ] Create baseline scripts and test runner
- [ ] Run import and GUI launch; record outputs
### Phase Completion Gate
- [ ] All required features implemented
- [ ] All tasks individually verified
- [ ] Godot imports without errors
- [ ] No GDScript parsing errors
- [ ] Gameplay tested in GUI (or limitation documented)
- [ ] No critical regressions
- [ ] Known issues documented
- [ ] TODO.md updated
- [ ] Phase reviewed and explicitly approved

## Phase 1 — Board [NOT STARTED]
- [ ] Implement original map, parser, reachability and board visuals
- [ ] Run unit and GUI tests
- [ ] Complete standard phase gate (copy all nine items above)
## Phase 2 — Nib movement [NOT STARTED]
- [ ] Implement grid motor, buffered input and portals
- [ ] Run unit and GUI tests
- [ ] Complete standard phase gate
## Phase 3 — Collectibles and HUD [NOT STARTED]
- [ ] Implement pickups, score, HUD and clear stub
- [ ] Run unit and GUI tests
- [ ] Complete standard phase gate
## Phase 4 — First Warden [NOT STARTED]
- [ ] Implement Vector, chase, lives, death and grace
- [ ] Run unit and GUI tests
- [ ] Complete standard phase gate
## Phase 5 — Four Wardens / MVP [NOT STARTED]
- [ ] Implement distinct brains, scheduling, basic pause and restart
- [ ] Run behavior suite and MVP playtest
- [ ] Complete standard phase gate
## Phase 6 — Power reversal [NOT STARTED]
- [ ] Implement frightened, capture chain, returning and respawn
- [ ] Run unit and GUI tests
- [ ] Complete standard phase gate
## Phase 7 — Relic and progression [NOT STARTED]
- [ ] Implement relic, bonus life and round scaling
- [ ] Run unit and GUI tests
- [ ] Complete standard phase gate
## Phase 8 — Menus and save [NOT STARTED]
- [ ] Implement menus, settings and resilient save store
- [ ] Run unit and GUI tests
- [ ] Complete standard phase gate
## Phase 9 — Presentation [NOT STARTED]
- [ ] Implement original art, audio and accessibility
- [ ] Run visual, audio and regression tests
- [ ] Complete standard phase gate
## Phase 10 — Release [NOT STARTED]
- [ ] Run full test matrix, prepare exports and README
- [ ] Smoke-test Windows/Linux builds
- [ ] Complete standard phase gate

## Testing results
| Timestamp | Phase | Command or manual test | Result PASS/FAIL/BLOCKED | Evidence / notes |
|---|---|---|---|---|

## Known issues and blockers
- None yet (do not use this statement if any defect exists).

## Technical decisions
| Date | Decision | Reason | Files affected |
|---|---|---|---|

## Session history
| Date/time | Work completed | Tests run | Next action | Approval received? |
|---|---|---|---|---|

## Latest review
Phase: 0
Status: IN PROGRESS
User approval: PENDING
```

**Template expansion rule:** Phase checklists above are seeds, not permission to omit the detailed requirements. At the start of each phase Qwen must copy each implementation task, acceptance criterion, and all nine gate boxes from Section 5 into that phase's section, preserving checked items and history. The next task must be explicit and small enough to do/test in one session.

## 7. Testing Matrix and Verification Commands

Adapt the executable path to installed Godot. Confirm CLI flags against its version. Example commands, **not claims of executed tests**:

```bash
godot --version
godot --headless --path . --editor --import --quit
godot --headless --path . --script res://tests/test_board.gd
godot --headless --path . --script res://tests/test_motor.gd
godot --headless --path . --script res://tests/test_rules.gd
godot --path .
```

Every headless test script should extend `SceneTree`, run its assertions, and call `quit(0)` or `quit(1)` based on outcome. A test suite run must report individual count and failures. Do not fabricate test results. Headless parsing and scene loading do **not** verify human input, rendering, audio, or full gameplay.

| Test domain | Automatic verification | Manual verification |
|---|---|---|
| Map | Dimensions, symbols, portals, flood-fill | Visually inspect original corridors and walls |
| Movement | Tile center alignment, turn queue, blocked path | Rapid turns and portal traversal |
| Pickup/score | Once-only score, counters, core timer | HUD matches actions |
| Collision | Overlap + segment crossing, no double death | Live death/grace sequence |
| AI | Targets, ties, bounds, BFS return | Four personalities visible in game |
| Round flow | Death precedence, resets, speed caps | Clear two rounds and restart |
| Persistence | Valid/corrupt save, bonus threshold | Relaunch app and check high score |
| Rendering/performance | Node counts, optional profiler logs | 30-min 60-FPS soak and no input lag |
| Export | Templates, launch and file existence | Run exported Windows/Linux binaries |

**Failure process:** Stop phase advancement; reproduce with smallest failing case; identify ownership (Board/Motor/Controller/Brain/UI); fix only affected subsystem; rerun targeted test plus previously passing regression tests; record commands, output, and changed files in TODO. No blind rewrites of functioning files. If Godot or GUI is unavailable, mark GUI-dependent work BLOCKED, not PASS.

## 8. Risk Register and Controls

| Risk | Impact | Prevention/test |
|---|---|---|
| Grid coordinate drift | Turns clip walls | One tile conversion API and snap at centers; long-run movement tests |
| Non-reachable crumb | Round cannot end | Flood-fill validation and count asserts |
| Portal infinite loop | Freeze or teleport oscillation | Arrival latch until exit tile crossed |
| AI impossible target | Warden stops or phases through walls | Clamp to nearest valid tile; legal neighbor selection; BFS return |
| Double death/score | Unfair play | Single controller collision resolver and atomic pickups |
| Unpaused timers | Core runs out in menu | Clock only advances in ACTIVE and respects pause |
| Ghost overlap on power expiry | Instant unfair collision | Explicit mode-order and transition collision test |
| Stale round callbacks | Relic/AI starts after reset | Central lifecycle and generation token on delayed actions |
| Oversized Qwen edits | Regression/context loss | 11 gated phases, small scripts, TODO resumption, diff review |
| Infringing art/audio | Release risk | Original character silhouettes, maze, synths, asset audit |
| Missing export dependency | Broken release | Fresh-machine packages and smoke tests |

## 9. Release Checklist

- [ ] Phases 0–10 ACCEPTED by user; all gate checklists complete.
- [ ] Clean Godot import, zero parse errors, relevant tests PASS.
- [ ] GUI tested with keyboard through full game/round/death/restart.
- [ ] Map original and every collectible reachable; no softlock.
- [ ] Gameplay 60 FPS in target resolution with 30-minute stable memory.
- [ ] High score survives restart; corrupt save harmless.
- [ ] All graphics, sound and music original and audited.
- [ ] Windows and Linux exported packages launched and smoke-tested.
- [ ] README contains Godot version, controls, build and license/credits.
- [ ] Unresolved issues noted, no critical or high-priority defects.

## 10. Local Model Operating Workflow

1. New session: read **this plan**, `TODO.md`, relevant source files, and Git status. Do not assume prior chat context.
2. Choose the smallest unchecked task in current phase. If phase is READY FOR REVIEW, **stop** and ask for approval.
3. Describe target files, acceptance check, and expected behavior before edits.
4. Implement one bounded slice; do not alter completed features or add future-phase features.
5. Run import/parse tests, targeted tests, and GUI check when available; do not call unrun tests passing.
6. Fix failures and rerun regressions. Update TODO with exact output, blockers, decisions, and next step.
7. On successful gate, mark READY FOR REVIEW, request explicit user approval, and stop.
8. Only after approval, record ACCEPTED and move to next phase.

**Context budget:** Keep working context focused on the plan, TODO, and current files; avoid pasting all source files into one prompt. Use short session summaries in TODO, not a separate competing progress file. Commit accepted phases if Git is configured; avoid destructive resets and preserve any unrelated user changes.

## 11. Master Qwen Coding Agent Handoff Prompt

Copy the following prompt into Pi/Qwen at the project root with this plan available:

```text
You are the implementation engineer for Maze Munchers, a Godot 4.x 2D game.
Your authoritative specification is maze-munchers-godot-game-plan.md.
Your single progress tracker is TODO.md. Do not create IMPLEMENTATION_STATUS.md.

First read the ENTIRE plan. Read TODO.md if present. Inspect the actual Godot
project, source tree, Git status, installed Godot version, and test tools.
If TODO.md is absent, CREATE IT from the template inside the plan, expanding
Phase 0 tasks and gates. Do not create the game in one pass.

Identify the CURRENT unfinished phase, its status, and the next unchecked task.
If the phase is READY FOR REVIEW, STOP and request user approval. If BLOCKED,
report the blocker and do not work around permission restrictions.
Never advance to the next phase without explicit user approval.

Implement ONLY the next small task in the current approved phase. List the
files you expect to touch and the test that will prove the task works. Use
native Godot scenes and small GDScript files. Match documented Godot APIs.
Preserve working behavior and any user changes. Do not rewrite unrelated
systems, introduce future features, or use reference-game copyrighted assets.

After changes, execute the applicable headless import and automated checks.
Perform GUI gameplay tests if a GUI is available. If unavailable, mark manual
checks BLOCKED/NOT RUN—not PASS. Investigate failed tests, fix, rerun relevant
tests and regressions before claiming completion.

Update TODO.md: checked verified tasks, phase status, exact commands/results,
files changed, outstanding bugs, decisions, session history, next action.
Only mark READY FOR REVIEW when ALL gate checks except approval have passed.
Do not mark ACCEPTED unless the user explicitly approves.

Final response each session: current phase/status; what you changed; files
modified; exact tests run and results; blockers; TODO next task; approval
request if the phase is ready. Then STOP.

Begin with Phase 0 only unless TODO.md proves later phases were accepted.
```

## 12. Short Recovery Prompt

```text
STOP feature work. Re-read maze-munchers-godot-game-plan.md and TODO.md fully.
Inspect Git status and current Godot project. State current phase, last
ACCEPTED phase, first unchecked task, and suspected regression. Do not redo
verified tasks or implement features outside the active phase. If a regression
exists, reproduce it, make the smallest fix, rerun targeted and regression
tests, and record actual results in TODO.md. If phase is READY FOR REVIEW,
stop for explicit approval. Never claim unrun tests passed. Report changed
files, test evidence, blockers and next action, then stop.
```

## 13. References and Assumptions

1. [Pac-Man — Wikipedia](https://en.wikipedia.org/wiki/Pac-Man), accessed October 2026: original release, gameplay, ghosts, cores, tunnels, relic, progressive difficulty.
2. [Godot 4 documentation](https://docs.godotengine.org/en/stable/): API and export verification source during implementation. Always consult installed-version docs if APIs differ.

**Explicit design assumptions (not historic facts):** 25×27 original map, 24px tiles, 112px/s Nib, original Warden target formulas, release timings, difficulty constants, 3 lives, 10k bonus, scene ownership and saving approach. They are intentional game rules, not claims about arcade hardware. The plan prioritizes a complete, testable, original spiritual successor over pixel-perfect imitation.
