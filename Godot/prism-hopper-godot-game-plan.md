<!--
Type: Phased game plan
Target: Godot 4.x, GDScript
Output: Complete Godot game project
Related: None
-->

# PRISM HOPPER — Godot Game Implementation Plan

**Specification version:** 1.0 (2026-10-22)  
**Reference:** [Q*bert (1982)](https://en.wikipedia.org/wiki/Q%2Abert)  
**Target:** Godot 4.x stable, GDScript, Windows/Linux desktop, keyboard first, 60 FPS  
**Implementation agent:** Qwen 3.8 Flash Next IQ3, approximately 64K context  
**Status:** Planning only. No game project or `TODO.md` has been created.

## 1. Executive brief

**Prism Hopper** is a brisk, single-screen puzzle-action arcade game about a small luminous explorer called **Pip** restoring a triangular array of floating prism pads. Hop diagonally between pads, charge every pad to its target state, dodge hostile clockwork creatures, and escape emergencies using limited **Recall Gates**. Levels grow harder through multi-charge tiles, reversible states, and more threatening enemy patterns.

The game preserves the *idea* of an isometric hopping puzzle, not the original characters, graphics, audio, names, or exact presentation. Use an original crystalline observatory theme: dark indigo void, teal/gold/magenta glass, geometric enemies, warm particle trails, and short synthesized chimes.

**Scope:** One triangular board configuration, ten hand-authored stage definitions, four enemy behaviors, one temporary freeze pickup, two Recall Gates per stage, score, three lives, menus, local high scores, accessibility options, and desktop exports. No multiplayer, network features, 3D world, level editor, boss, combat weapons, or online leaderboard. These are explicitly out of scope for v1.

**Core player promise:** Every hop matters; reclaim the board while threats force calculated detours.

**MVP gate:** End of Phase 6. A complete start-to-game-over loop with at least three distinct stages, two threats, score/lives, and restart must be genuinely playable.

## 2. Source research and design boundaries

### Documented historical facts

- Q*bert was released for arcades by Gottlieb on **October 18, 1982**. It is a 2D action/puzzle game presented with a pseudo-3D isometric pyramid.
- The classic board contains **28 cubes**; the avatar hops diagonally among cube tops. Landing changes cube color. All cubes must reach the target color to clear a stage.
- Early stages need one landing per cube. Later variants need multiple landings or toggle colors, including possible regression.
- Falling off the board or touching dangerous enemies costs a life. Coily develops from an egg into a chasing snake. Ugg and Wrongway travel along cube sides. Slick and Sam undo tile progress but are safe to touch.
- Red balls are harmful; green balls freeze enemies for a limited time. Side discs carry the player to the top and can lure the pursuing snake off an edge.
- Points come from tile changes, certain enemy interactions, unused discs, and stage completion. An operator-configurable extra-life threshold exists.

Source: [Wikipedia, Q*bert — Gameplay](https://en.wikipedia.org/wiki/Q%2Abert#Gameplay). This is a secondary summary; original game-specific numbers are **not** assumed to apply to Prism Hopper.

### Adaptations and assumptions for this project

The following are **new design choices**, not claims about the 1982 game:

| Mechanic | Prism Hopper rule |
|---|---|
| Board | Seven rows, 28 prism pads, graph coordinates `(row, col)` with `0 <= col <= row < 7` |
| Movement | Exactly one neighbor per input; four directional hops; no free walking or acceleration |
| Hop cadence | 0.18 seconds in early stages; minimum 0.13 seconds later |
| Landing effect | Applies once on arrival, not continuously during overlap |
| Target | Each pad has `required_hits` of 1 or 2 and a capped `charge` |
| Toggle stages | Some pads cycle `0 → 1 → 2 → 0`, with target `2` |
| Fall | Invalid hop costs one life unless it enters a valid Recall Gate |
| Enemies | **Drifter** rolls downhill; **Seeker** tracks the player; **Saboteur** reverses charges; **Rim Skater** patrols a fixed boundary route |
| Nonlethal enemy | Touching a Saboteur removes it and gives bonus points; other enemy contacts are lethal |
| Gates | Two limited single-use edge rescue gates, enabled in selected positions; transport to apex |
| Freeze | Optional collectible suspends enemy movement for 3.0 seconds, not the player |
| Campaign | Ten preset stages; repeat Stage 10 at higher speed for endless score play |

We deliberately omit true Escher-style side-surface travel and physics-based rolling. Enemy paths are graph routes and hop schedules, far easier to test and keep synchronized.

## 3. Exact gameplay contract

### 3.1 Board and screen

- Logical viewport **960 × 720**, 16:12 aspect; stretch mode `canvas_items`, stretch aspect `keep`, integer pixel snapping optional. Minimum window 960 × 720; resizable desktop window with letterboxing where needed.
- Center the triangular board with apex at `(480, 142)`, grid horizontal step `92`, vertical step `69`. Pad center: `x = 480 + (2 * col - row) * 46`, `y = 142 + row * 69`. A pad is a stylized diamond-top prism drawn with `Polygon2D` and a highlighted top face. World coordinates are derived solely from grid coordinates.
- Seven rows produce 28 cells. Array ordering is stable `row-major`; grid coordinate is authoritative. For gameplay, use logical landing nodes; avoid relying on sprite overlap for neighbor detection.
- HUD top: score, lives, stage, progress `(charged/28)`, freeze status. HUD never obscures top node. A bottom help strip gives controls and gate count.

### 3.2 Direction map and input

| Action | Keyboard | Grid delta |
|---|---|---|
| Up-left | `Q` or `Left` | `(-1, -1)` |
| Up-right | `E` or `Up` | `(-1, 0)` |
| Down-left | `Z` or `Down` | `(+1, 0)` |
| Down-right | `C` or `Right` | `(+1, +1)` |
| Pause | `Esc` or `P` | — |
| Confirm | `Enter` or `Space` | — |

This explicit binding avoids the awkward mental model of rotated joystick directions. At launch, a four-arrow directional legend must be shown. Controls may later be remapped. Define actions `hop_ul`, `hop_ur`, `hop_dl`, `hop_dr`, `pause`, `confirm` in Input Map; support gamepad D-pad in a later phase if time permits.

Only one move may be in flight. Ignore repeated key presses until the arrival callback completes. If several presses arrive on one frame, process a single action with fixed priority UL, UR, DL, DR, not an arbitrary last value. Holding a direction may repeat after 0.26 seconds and every 0.18 seconds thereafter; this must use the same move guard.

### 3.3 Hop and collision timing

- `Player` owns logical `(row, col)`, `is_hopping`, `is_invulnerable`, `alive` and hop tween. Player input calls `Board.get_neighbor(position, direction)`; Board returns valid coordinate or `null`.
- On a valid hop, tween display position in a shallow arc (`sin(pi * t) * 20` pixels of elevation) for configured duration; commit logical destination **only at landing**; emit `landed(coord)` exactly once.
- On an invalid hop, check gate matching the attempted departure edge/direction. If active, consume gate and animate transport to apex; otherwise emit `fell` and lose one life. The game never indexes an invalid coordinate.
- Enemy-player damage is resolved at **discrete arrival events**, checking both matching destination and opposite-direction cross-over in the same tick. During scene transitions and respawn invulnerability, ignore hits. Deduplicate damage using a `damage_pending` flag per event until the respawn completes.
- On damage: decrement life once; play feedback; clear existing hazards, reset player to apex after 0.75 seconds; apply 1.25 seconds invulnerability; retain board progress unless the stage config explicitly says otherwise (v1: always retain).
- At zero lives transition to GAME_OVER, stop all timers and input, record high score only once.

### 3.4 Charge rules and win condition

Pad model: `charge: int`, `required_hits: int`, `mode: enum(STICKY, CYCLE)`, `target: int`. On player landing:

- STICKY: `charge = min(charge + 1, target)`.
- CYCLE: `charge = (charge + 1) % (target + 1)`.
- Saboteur: `charge = max(charge - 1, 0)` for STICKY; for CYCLE: `charge = (charge - 1 + target + 1) % (target + 1)`.
- A pad is complete iff `charge == target`. `Board.completed_count` recalculates from the authoritative array after each change (28 items is cheap and robust).
- Award **20 points** for a player landing that increases charge, **0** for a fully charged STICKY pad, **10** for a CYCLE landing that wraps, **200** for catching a Saboteur, **100** for collecting Freeze, **350** for luring a Seeker off a gate, and **1000 + 200 × (stage_number - 1)** for stage completion. Award **75** for each unused gate when clearing a stage. Score never decreases.
- Clear stage only after landing logic and collision resolution when all 28 pads match their target and the player survives that landing. Enter STAGE_CLEAR once, pause threat spawning, grant bonuses, show result for 1.5 seconds, then load next stage.

### 3.5 Enemy and hazard behavior

All enemies share logical positions, visual nodes, a `move_interval`, and a `tick()` method. `EnemyDirector` advances each on independent timers with seeded RNG, without physics impulses. Each tick checks valid graph neighbors. Spawn only on an unoccupied, non-apex pad. If no eligible tile exists, delay the spawn rather than stacking actors.

| Type | Look | Exact behavior | Player contact |
|---|---|---|---|
| Drifter | Amber rolling prism | Spawn at upper third; at each 0.85–0.60s tick choose randomly from valid downward neighbors; despawn at bottom edge | Lose life |
| Seeker | Indigo triangle eye | After 1.0s warning, enter at bottom; every 0.95–0.65s pick neighbor minimizing graph distance to player; tie-break by stable neighbor order; on failed gate pursuit, despawn with bonus | Lose life |
| Saboteur | Mint gear | Every 1.10–0.85s choose downward neighbor; on arrival reverse that pad's charge; when reaching bottom despawn | Remove enemy, +200 |
| Rim Skater | Red ring | Traverse a predefined list of edge coordinates at 0.90–0.65s; on reaching end reverse index; never teleport between nonadjacent nodes | Lose life |

**Important:** Bottom-to-top Seeker pathing uses shortest-path breadth-first search on the 28-node graph. Do not use Manhattan distance; it can create poor routes on triangular coordinates. Rim route must be validated so every consecutive pair is a legal neighbor; if not, replace with a legal perimeter patrol made from valid steps.

Enemy entities occupy a logical tile; their visuals can tween. Collision checks run when an actor lands and when the player lands. During Freeze, stop enemy timers/tweens coherently; on expiry resume with remaining progress or restart one full interval consistently. Spawn director continues? **No**: Freeze suspends spawns as well, to avoid bursts.

Spawn caps: Stage 1 one Drifter; Stage 2 two Drifters; Stage 3 one Seeker plus Drifter; Stage 4 Saboteur; Stage 5 Rim Skater; Stages 6–10 mixed types up to **four simultaneous hostile enemies** plus one Saboteur. Spawns at earliest every 2.6 seconds initially, reducing to 1.6 seconds by Stage 10. Director retries after 0.5 seconds if at cap.

### 3.6 Gates and collectible

- Each stage supplies two gate anchors adjacent to outer edge pads: `(row 4,col 0)` on a further up-left invalid hop, and `(row 4,col 4)` on a further up-right invalid hop **only if the attempted delta is invalid**. Implementation must validate exact coordinate/direction pairs; if those pairs are valid under the direction map, instead use a truly invalid boundary exit from those edge cells. Bind gates to `(source_coord, direction)`, not approximate pixel regions.
- A gate displays a pulsing portal symbol next to the linked pad. Entering it transports Pip to `(0,0)`, consumes one charge, and grants 0.5 seconds safe arrival time. Gates do not replenish during a stage.
- A Seeker targeting a player using a gate may enter the expired trajectory on its next move and fall if within one legal hop of departure cell; add the 350-point bonus and despawn. This is an optional narrowly defined interaction activated only if its position matched departure cell's neighboring pursuit path at gate use.
- Freeze Orb spawns once per stage from Stage 3 onward at an unoccupied non-apex tile, 8–15 seconds after stage begins. It lasts 8 seconds on the board, disappears when collected, and cannot respawn that stage.

### 3.7 Stages, difficulty, and pacing

All levels reuse the same board. `StageData` resources specify: stage id, title, target charge, pad mode, starting lives policy, permitted enemy types, spawn interval, enemy move multipliers, collectible enabled, and gate count. Stage changes reset board charge, positions, enemies, gate uses, and timers but retain score and remaining lives.

| Stages | Tile rule | Hazards | Goal |
|---|---|---|---|
| 1 | STICKY 1 hit | 1 Drifter | Teach four hops and simple coverage |
| 2 | STICKY 1 hit | 2 Drifters | Build route awareness |
| 3 | STICKY 1 hit | Drifter + Seeker, Freeze | MVP threat variety |
| 4 | STICKY 2 hits | Saboteur + Drifter | Learn revisiting |
| 5 | STICKY 2 hits | Rim Skater + Seeker | Edge timing |
| 6 | CYCLE target 2 | Drifter + Saboteur | Avoid accidental wraps |
| 7 | STICKY 2 hits | 3 mixed threats | Route optimization |
| 8 | CYCLE target 2 | 3 mixed threats | Precision under pressure |
| 9 | CYCLE target 2 | 4 mixed threats | Advanced challenge |
| 10+ | Alternate STICKY/CYCLE 2 hits | 4 mixed threats; capped speed | Endless score loop |

No timer-based loss. Pace comes from enemies and board completion; lower-skilled players can take their time. Extra life at **10,000 and 25,000 points**, once each per run; maximum five lives. Start with three.

### 3.8 States and UI

`GameSession` state enum: `MENU`, `PLAYING`, `PAUSED`, `RESPAWNING`, `STAGE_CLEAR`, `GAME_OVER`. Only PLAYING accepts hop requests; PAUSED pauses process trees via an appropriate pause mode, but pause menu stays active. Provide New Game, Continue (only in live paused run), Options, Quit; game-over screen gives score, best score, Restart, Main Menu. Display accessible HUD with icons and text, never color alone.

Settings: master/sfx/music volume 0–100%, reduced motion, high-contrast palette, screen shake on/off, optional input repeat, stored under `user://settings.cfg`. High scores: top ten local entries in `user://scores.json`, atomic temp-file save/rename or safe replacement, sanitize malformed data; no personal information or cloud access.

## 4. Architecture and files

Use one gameplay scene and small isolated scripts. Scene nodes own visuals; model/data classes own rules. No autoload except `SaveStore` (persistence) and optionally `AudioBus` (audio). Avoid a sprawling singleton `GameManager`.

```text
res://
  project.godot
  scenes/
    main_menu.tscn
    game.tscn
    board.tscn
    prism_pad.tscn
    player.tscn
    enemies/enemy.tscn
    pickups/freeze_orb.tscn
    ui/hud.tscn
    ui/pause_menu.tscn
    ui/game_over.tscn
  scripts/
    game_session.gd
    board.gd
    prism_pad.gd
    player.gd
    enemy_base.gd
    enemy_director.gd
    gate_system.gd
    stage_data.gd
    stage_catalog.gd
    save_store.gd
    hud.gd
    audio_bus.gd
  resources/stages/
  assets/graphics/
  assets/audio/
  tests/
    test_board.gd
    test_charge.gd
    test_enemy_path.gd
    test_save.gd
    smoke_test.gd
  docs/
    DESIGN_DECISIONS.md
  TODO.md                 # created by coding agent at start of Phase 0
```

**Only create files when their owning phase requires them.** The tree is the destination, not a command to generate empty files. A small test runner `SceneTree` script may replace a third-party test framework.

### Ownership and data flow

- `GameSession` owns run state, score, lives, current stage, state transitions, and scene orchestration. It requests stage setup from `Board` and `EnemyDirector`; it does not draw tiles or calculate routes.
- `Board` owns 28 authoritative pad states and neighbor queries, emits `pad_changed(coord,charge,target)` and `board_complete`. No enemies or UI references inside Board.
- `PrismPad` displays a tile and its charge. Its polygon visual is not a gameplay collision source.
- `Player` owns movement timing and emits `landed(coord)`, `invalid_hop(from,direction)`. GameSession listens and triggers Board charge and hazard resolution in the specified order.
- `EnemyDirector` creates, steps, despawns enemies and emits `enemy_landed(enemy,coord)`, `enemy_contact(enemy)`, `enemy_removed(enemy,reason)`. Shared path utility uses `Board.get_neighbors()`.
- `GateSystem` owns current gate charges, valid exit mapping and emits `gate_used`.
- `HUD` listens to session snapshot updates, never changes score directly.
- `SaveStore` alone reads/writes `user://`; version config files and tolerate missing/corrupt entries.

Signals flow from children to `GameSession`, then command handlers flow down. Never let Player call HUD or EnemyDirector directly. Use typed GDScript wherever practical.

### Collision and lifecycle policy

Use **logical occupancy** for tile-based gameplay collisions, so `Area2D`/`CollisionShape2D` are not needed to decide who is on a pad. If particle effects or future physical triggers need actual physics layers, reserve Layer 1 = player, 2 = enemies, 3 = pickups, 4 = world; masks never substitute for discrete tile checks. Avoid duplicate damage from both physics signals and logical checks.

At stage reset, stop all tweens and timers, disconnect transient event handlers when needed, call `queue_free()` on transient actors, then spawn next stage after the previous nodes are released. Never retain dangling references. Preserve the score/lives model across stage scene resets.

### Stage data sample (conceptual typed resource)

```gdscript
extends Resource
class_name StageData
@export var stage_number: int = 1
@export_range(1, 2) var target_charge: int = 1
@export_enum("STICKY", "CYCLE") var charge_mode: String = "STICKY"
@export var spawn_interval: float = 2.6
@export var enemy_cap: int = 1
@export var enemy_types: Array[StringName] = [&"drifter"]
@export var freeze_enabled: bool = false
```

Keep basic constants in named scripts; StageData resources must be data-only. Do not store runtime node references in resources.

## 5. Phase-by-phase build protocol

**Global rule:** Implement one numbered phase at a time. Each produces a runnable desktop/editor build. Run headless parsing/import checks and interactive play verification at every gate. Do not proceed until the user explicitly approves the phase. A failed test keeps its checklist item open. Every phase must maintain `TODO.md` and record evidence.

The following acceptance checks are additive: passing a new phase also requires previously accepted tests to remain passing.

### Phase 0 — Project initialization

**Objective:** Boot a Godot project into a functional menu/test screen. **Dependencies:** Godot 4.x stable installed; no prior phases.

**Create/modify:** `project.godot`, `scenes/main_menu.tscn`, minimal `scripts/game_session.gd` if needed, `tests/smoke_test.gd`, `docs/DESIGN_DECISIONS.md`, and `TODO.md`.

**Implementation:** Set 960×720 logical viewport, stretch/letterbox policy, 60 FPS target, project main scene, specified input actions. Add a visible title and Start button that opens a placeholder gameplay scene (which can display "Board coming next"). Establish Git repository and ignore `.godot/` generated cache. Create TODO template from Section 7; track Godot version, full launch command, baseline screenshots.

**Integration:** Main menu must launch the placeholder and return cleanly.

**Acceptance:** Project opens in editor; no script parse errors; menu buttons work; Escape returns or closes placeholder; input bindings exist. **Test:** `godot --headless --path . --editor --quit` then editor/F6 run and click through; record exact engine binary/version. **Likely failure:** invalid project setting names or main-scene path; inspect engine log, correct and rerun. **Done:** All checks, logs and user approval recorded.

### Phase 1 — Isometric board and graph rules

**Objective:** Render the entire seven-row board deterministically. **Dependencies:** Accepted Phase 0.

**Create/modify:** `scenes/board.tscn`, `scenes/prism_pad.tscn`, `scripts/board.gd`, `scripts/prism_pad.gd`, `tests/test_board.gd`, `scenes/game.tscn`.

**Implementation:** Build 28 pads with `(row,col)` keys; instantiate placeholder colored polygon faces using the projection formula. Implement `is_valid`, `get_neighbor`, `get_neighbors`, `coord_to_world`, `reset_board`. Write pure tests for count, uniqueness, neighbor reversibility, all border exits, and apex. Show coordinate labels in debug mode only.

**Integration:** `GameSession` owns Board node. **Acceptance:** Exactly 28 visible pads; no duplicate locations; four neighbor directions behave as spec; viewport fits. **Test:** headless import + scripted board assertions + manual visual at 960×720 and resized window. **Likely failure:** swapped row/col causing impossible diagonal moves; use formula and test pairs. **Done:** All assertions and visual tests passed, user approval.

### Phase 2 — Player hop control

**Objective:** Make the board navigable with responsive, discrete hops. **Dependencies:** Accepted Phase 1.

**Create/modify:** `scenes/player.tscn`, `scripts/player.gd`, `scenes/game.tscn`, `tests/smoke_test.gd`.

**Implementation:** Render original placeholder Pip (small glowing diamond with eyes); spawn at apex. Map four directions exactly. Tween a 0.18s arc and land once. Add move guard and controlled held-key repeats. For now, on invalid hop return player to apex after a brief flash; full life damage comes later. Expose landed/invalid signals.

**Integration:** Player queries Board graph; no direct tile state mutation. **Acceptance:** All four hop directions work; key presses cannot trigger two simultaneous hops; no drift; border fall recognized; no invalid array lookup. **Test:** manual direction matrix on apex/middle/edges, hold-key stress for 30 seconds, scripted signal count where possible. **Likely failure:** tile charge triggered at departure or twice; ensure only arrival emits landed. **Done:** Navigable build and tests, user approval.

### Phase 3 — Pad states, scoring, and stage clear

**Objective:** Establish the addictive land-and-charge loop. **Dependencies:** Accepted Phase 2.

**Create/modify:** `scripts/stage_data.gd`, `scripts/stage_catalog.gd`, `resources/stages/stage_01.tres`, `scripts/board.gd`, `scripts/game_session.gd`, `tests/test_charge.gd`.

**Implementation:** Implement STICKY/CYCLE pad rule as standalone model methods; drive visuals from signals. Add score increments and progress counter. Fill all 28 pads to enter STAGE_CLEAR once and transition to next configured stage (temporary repeat of Stage 1 permitted until Phase 6). Display stage-cleared overlay; block input during transitions.

**Integration:** Player landed → GameSession resolves charge → Board emits change/completion → session awards points/transition. **Acceptance:** Each pad changes color on arrival; score matches defined values; completion triggers exactly once; no restart loop or double bonus. **Test:** charge unit tests with repeats and wrap; manually clear board; scripted forced-complete test. **Likely failure:** completion checked before last pad update; order rules explicitly. **Done:** Full board-completion loop verified, user approval.

### Phase 4 — Lives, hazards, and stable collisions

**Objective:** Add risk without making collisions unpredictable. **Dependencies:** Accepted Phase 3.

**Create/modify:** `scenes/enemies/enemy.tscn`, `scripts/enemy_base.gd`, `scripts/enemy_director.gd`, `scripts/game_session.gd`, `tests/test_enemy_path.gd`.

**Implementation:** Add Drifter's random downward motion and timed spawning; damage from enemy occupancy/cross-over and falls; single-hit guard, three lives, respawn with invulnerability, game-over overlay placeholder. Cap simultaneous enemies and clean up at stage resets. Seed RNG for automated tests.

**Integration:** Director emits arrival/contact; session resolves fatal damage once. **Acceptance:** Drifters only use legal downward neighbors; no duplicate hits from one overlap; exactly one life lost per fall; death at zero lives stops play. **Test:** forced overlap, swap-crossing, fall, rapid restart, repeated scene load, unit path assertions. **Likely failure:** duplicate signal connections or two damage routes; enforce session damage latch. **Done:** Stable collision and life loop, user approval.

### Phase 5 — Seeker, Recall Gates, and Freeze

**Objective:** Add tactical rescue and active pursuit. **Dependencies:** Accepted Phase 4.

**Create/modify:** `scripts/gate_system.gd`, `scenes/pickups/freeze_orb.tscn`, `scripts/enemy_director.gd`, `tests/test_enemy_path.gd`.

**Implementation:** BFS shortest-path Seeker with tie-break order, distinct visual; validate gate exit tuples with graph assertions, consume gate on invalid mapped hop, teleport safely to apex, display gate count. Add timed Freeze Orb from Stage 3; suspend all enemy movement/spawns for 3 seconds. Implement narrowly specified Seeker lure bonus only for verified qualifying pursuits.

**Integration:** GateSystem intercepts player's invalid hop before fall damage; freeze state gates director timers. **Acceptance:** Seeker reaches an accessible player without getting stuck; gate use consumes exactly one gate; non-gate edge is fatal; freeze lasts correct duration; no frozen-spawn burst. **Test:** deterministic BFS fixture, both gate exits, exhausted gate, freeze mid-tween, stage reset during freeze. **Likely failure:** gate specified on a legal hop; assert border invalidity at initialization and correct mapping. **Done:** Tactical systems verified, user approval.

### Phase 6 — Complete arcade MVP

**Objective:** Deliver a genuinely enjoyable, end-to-end minimum viable game. **Dependencies:** Accepted Phase 5.

**Create/modify:** `scenes/ui/hud.tscn`, `scripts/hud.gd`, `scenes/ui/game_over.tscn`, `scenes/ui/pause_menu.tscn`, `scripts/game_session.gd`, `resources/stages/stage_02.tres`, `stage_03.tres`, `tests/smoke_test.gd`.

**Implementation:** Build Stage 1–3 preset configurations, transitions, complete HUD, pause/resume, functional menus, restart, game-over score; ensure every run can be completed or lost. Prevent pause input from propagating into player hops. Add basic instructions overlay with mapped directions.

**Integration:** HUD updates only from session signals; stage data drives board and director. **Acceptance:** Three different stages play sequentially; score and lives persist correctly; victory and defeat flow to right states; pause freezes action; restart resets score/stage/lives; stable 60 FPS target on ordinary desktop. **Test:** full manual run from menu through three clears and a separate run to game-over; test pause during hop, death and stage clear. **Likely failure:** state transition races; centralize state setter and block duplicate transitions. **Done:** Mark **MVP READY FOR REVIEW**, not accepted until user explicitly approves Phase 6.

### Phase 7 — Full stage variety and enemy roster

**Objective:** Complete ten-stage content and meaningful escalation. **Dependencies:** Accepted Phase 6.

**Create/modify:** `scripts/enemy_director.gd`, `scripts/enemy_base.gd`, stage resources 04–10, `scripts/stage_catalog.gd`, tests.

**Implementation:** Add Saboteur reverse-charge behavior and nonfatal capture bonus, Rim Skater legal patrol, two-hit and CYCLE tile modes. Cap threats at four and pace with table in Section 3.7. At Stage 10 clear, advance to Stage 11 equivalent endless variant alternating modes with capped enemy speeds; never allocate infinite stage assets.

**Integration:** Each enemy remains a small behavior branch or subclass with common lifecycle; Board alone changes charges. **Acceptance:** All ten stages playable; distinct types visibly readable; cycle charging never produces impossible win condition; no enemy teleports across nonadjacent graph nodes; endless repeat works. **Test:** scripted stage load for 1–12; charge regression tests; legal-path assertion for rim; manual Stage 6/8 precision tests. **Likely failure:** cycling with saboteurs prevents completion indefinitely; cap sabotage rate, tune spawn director; do not silently alter board rules. **Done:** Full content approved.

### Phase 8 — Original visual/audio polish and accessibility

**Objective:** Replace functional placeholders with cohesive, original presentation. **Dependencies:** Accepted Phase 7.

**Create/modify:** `assets/graphics/*`, `assets/audio/*`, `scripts/audio_bus.gd`, UI and scene visuals, settings UI scripts.

**Implementation:** Draw original vector-like prism forms and Pip within Godot; use particles sparingly for landings, gate travel and stage clear. Synthesize original short blips/chimes via Godot or licensed self-created waveforms, a subtle 2-minute looping ambient track, volume buses. Add distinct shape/symbol on each charge state, high-contrast palette, reduced-motion option, screen shake toggle, clear button focus, keyboard navigation. Record asset source/licensing in design decisions.

**Integration:** Presentation subscribes to events and cannot change gameplay timing. **Acceptance:** No unlicensed asset; visual states understandable without color; all menus navigable by keyboard; sliders persist; reduced-motion suppresses shake and large flashes. **Test:** headphones/speakers, muted audio, high-contrast mode, 960×720 and enlarged window, keyboard-only menu walkthrough. **Likely failure:** particles obscure collision cues or audio repeats; cap emitters and debounce SFX. **Done:** Aesthetic and accessibility checklist approved.

### Phase 9 — Persistence, resilience, and balancing

**Objective:** Make repeated play stable and rewarding. **Dependencies:** Accepted Phase 8.

**Create/modify:** `scripts/save_store.gd`, `tests/test_save.gd`, options UI, `docs/DESIGN_DECISIONS.md`.

**Implementation:** Implement top-ten local score file and settings file under `user://`; handle missing/invalid JSON and config gracefully; never overwrite on mere menu navigation. Add extra lives at thresholds, cap five. Tune 10-stage progression based on measured play; document target initial session 5–12 minutes for a competent player, without pretending it was achieved until measured.

**Integration:** GameSession gives final score to SaveStore exactly once; menus display best score. **Acceptance:** Settings/high scores survive app restart; corrupt files fall back safely; score recorded once; extra lives awarded once each; no infinite lives from stage transitions. **Test:** save/load roundtrip, malformed file, read-only user directory behavior, repeated game-over, score threshold boundaries. **Likely failure:** non-atomic files, duplicated append; use temporary write and verified replacement or safe fallback. **Done:** Persistence and balance reviewed.

### Phase 10 — QA, packaging, and release

**Objective:** Produce reliable Windows and Linux builds. **Dependencies:** Accepted Phase 9.

**Create/modify:** Godot export presets, `README.md`, license/credits, release notes, automated smoke tests and export artifacts.

**Implementation:** Freeze scope. Run headless import, all unit/smoke tests, full run progression, pause/restart/save testing, desktop export validation. Run in release and debug settings; capture approximate FPS/memory on available machine. Export desktop builds with local Godot export templates. No installer required; ZIP each platform build and provide controls/help file.

**Integration:** Test exported binaries, not merely project editor runs. **Acceptance:** Windows and Linux packages launch; title/gameplay/game-over/restart work; zero known critical or high-severity defects; no missing assets or copyrighted material; 60 FPS target on designated test hardware; gameplay and settings usable offline. **Test:** record OS, GPU, Godot version, export command, checksum, manual results, issues and limitations. **Likely failure:** missing export templates/resource path case differences; fix and rebuild each platform. **Done:** User explicitly accepts release candidate; tag version `v1.0.0` only after acceptance.

## 6. Verification commands and defect handling

Use the actual Godot 4 executable name (`godot`, `godot4`, or absolute path) after checking `--version`; substitute it below. `--headless --editor --quit` tests project import and parsing, **not gameplay**.

```bash
godot --version
godot --headless --path . --editor --quit
godot --headless --path . --script res://tests/test_board.gd
godot --headless --path . --script res://tests/test_charge.gd
godot --headless --path . --script res://tests/test_enemy_path.gd
godot --headless --path . --script res://tests/test_save.gd
godot --headless --path . --quit-after 120
```

Only execute test scripts once they exist. Test scripts that extend `SceneTree` should exit nonzero on assertion failure (`quit(1)`); explicitly print failure details. Verify version-sensitive CLI flags and runtime APIs against the installed [official Godot 4 documentation](https://docs.godotengine.org/en/stable/), especially input/viewport settings, Tween, pause processing, and export flags. A `--quit-after` timed run is a smoke check, not proof of correct UI behavior. Open the editor and play manually after every phase.

Suggested manual regression matrix:

| Area | Checks |
|---|---|
| Movement | 4 directions, held keys, simultaneous keys, all edge exits, apex |
| Tile logic | 1-hit, 2-hit, CYCLE wrap, sabotage undo, completion last pad |
| Hazards | enemy and player arrive together, cross-over, multiple enemies, invulnerability |
| Gate/freeze | each entrance, depleted gate, seeker lure, expiry and restart |
| Game flow | fresh game, 3-stage MVP, stage 10+, pause during tween, game-over, restart |
| Persist | restart app, malformed settings/score, duplicate game-over, high-score ordering |
| Presentation | keyboard focus, high contrast, reduced motion, muted audio, resized viewport |
| Performance | 60 FPS, memory not steadily rising after 20 stage/restart cycles, ≤4 threats |

**Bug workflow:** Reproduce with exact stage/coordinates/actions; record current and expected behavior in TODO; add a focused regression test; change the smallest owning file; rerun failed test, phase tests, all previous critical regression tests; manually verify; only then close the bug. If baseline functionality breaks, revert only the agent's own unaccepted change using a safe patch or version-control commit; never discard user changes or rewrite working systems wholesale.

## 7. Mandatory `TODO.md` template

At the start of Phase 0, the implementation agent copies the following template into the project root as `TODO.md`, expands each phase into the corresponding tasks above, and keeps **this file as the single progress source**. Do not create `IMPLEMENTATION_STATUS.md`.

```markdown
# Prism Hopper — Implementation TODO

## Status
- Current phase: 0 — Project initialization
- Phase status: IN PROGRESS
- Last accepted phase: None
- Build status: NOT TESTED
- Next actionable task: Create project.godot and menu scene
- Godot version: Unknown — verify
- Last updated: YYYY-MM-DD HH:MM timezone

## Workflow rules
- Read prism-hopper-godot-game-plan.md and this file before editing.
- Status values: IN PROGRESS, BLOCKED, READY FOR REVIEW, ACCEPTED.
- Only user-approved phases may be marked ACCEPTED.
- Never check a task without implementation and verification evidence.
- Do not advance a phase without user approval.
- Reopen previously checked items after regressions.

## Phase 0 — Project initialization [IN PROGRESS]
- [ ] Create and verify Godot project settings and main menu
- [ ] Define input actions
- [ ] Add runnable placeholder game and return path
- [ ] Establish test script and run import check
- [ ] Record Godot version and design decisions
- [ ] Complete Phase Completion Gate

## Phase 1 — Board and graph [NOT STARTED]
- [ ] Render 28 unique pad positions
- [ ] Implement legal graph neighbors and tests
- [ ] Verify display and resizing
- [ ] Complete Phase Completion Gate

## Phase 2 — Player hop [NOT STARTED]
- [ ] Implement four-direction grid hop and tween
- [ ] Implement input guard and border detection
- [ ] Test all edges and held input
- [ ] Complete Phase Completion Gate

## Phase 3 — Pad charges and scoring [NOT STARTED]
- [ ] Implement STICKY/CYCLE charge model
- [ ] Score landings and track completion
- [ ] Implement stage transition and tests
- [ ] Complete Phase Completion Gate

## Phase 4 — Lives and Drifter [NOT STARTED]
- [ ] Implement director and Drifter pathing
- [ ] Implement single-hit damage/respawn/game-over
- [ ] Test collision and resource cleanup
- [ ] Complete Phase Completion Gate

## Phase 5 — Seeker, gates, freeze [NOT STARTED]
- [ ] Implement BFS Seeker
- [ ] Implement validated Recall Gates and lure
- [ ] Implement Freeze Orb and timers
- [ ] Complete Phase Completion Gate

## Phase 6 — Complete MVP [NOT STARTED]
- [ ] Create stages 1–3 and transitions
- [ ] Complete HUD, menus, pause, restart and game-over
- [ ] Run complete MVP playthrough
- [ ] Complete Phase Completion Gate

## Phase 7 — Full game content [NOT STARTED]
- [ ] Add Saboteur and Rim Skater
- [ ] Add stages 4–10, 2-hit and cycling pads
- [ ] Add endless loop and progression tests
- [ ] Complete Phase Completion Gate

## Phase 8 — Polish and accessibility [NOT STARTED]
- [ ] Original visuals, FX, music and SFX
- [ ] Readable HUD and keyboard UI navigation
- [ ] High contrast, reduced motion and audio options
- [ ] Complete Phase Completion Gate

## Phase 9 — Persistence and balance [NOT STARTED]
- [ ] Save/load scores and settings with corruption fallback
- [ ] Implement extra-life threshold rules
- [ ] Test pacing and repeat-run stability
- [ ] Complete Phase Completion Gate

## Phase 10 — QA and release [NOT STARTED]
- [ ] Full regression and performance suite
- [ ] Windows/Linux export and launch verification
- [ ] README, credits, release notes, acceptance
- [ ] Complete Phase Completion Gate

## Phase Completion Gate — copy separately under each phase
- [ ] All required features implemented
- [ ] All tasks individually verified
- [ ] Godot imports without errors
- [ ] No GDScript parsing errors
- [ ] Relevant automated tests actually run and pass
- [ ] Interactive gameplay tested
- [ ] No critical regressions
- [ ] Known issues documented
- [ ] TODO.md updated with evidence
- [ ] Phase reviewed and explicitly approved by user

## Test results (append; never fabricate)
| Date | Phase | Command or manual steps | Result | Evidence / error |
|---|---|---|---|---|
| TBD | 0 | godot --version | NOT RUN | — |

## Known issues / blockers
| ID | Severity | Reproduction | Expected | Actual | Owner/next action |
|---|---|---|---|---|---|

## Technical decisions (append only or supersede with rationale)
| Date | Decision | Reason | Consequences |
|---|---|---|---|

## Session history (newest first)
| Date/time | Phase | Files changed | Tests run | Next task | Approval state |
|---|---|---|---|---|---|

## Approval log
| Phase | Submitted for review | Approved by user on | Notes |
|---|---|---|---|
```

**Status semantics:** IN PROGRESS = work started; BLOCKED = cannot proceed with documented obstacle; READY FOR REVIEW = implementation/tests done, waiting for user; ACCEPTED = user has expressly approved. Never infer approval from silence or passing tests. If regressions occur in an accepted phase, open the affected task and log the regression without falsifying the prior approval date.

## 8. Quality constraints and risk controls

| Risk | Preventive rule | Regression check |
|---|---|---|
| Wrong triangular coordinates | One Board projection and graph authority | Every adjacent move reversible; 28 unique cells |
| Broken gate border mapping | Validate source/direction gives `null` neighbor | Automated gate-pair test |
| Double scoring or damage | Commit events only on discrete landing; central damage guard | Simultaneous arrival tests |
| Stuck Seeker | Deterministic BFS with bounded graph | All source/target paths reachable |
| Rim teleports | Prevalidated adjacent patrol pairs | Assert every consecutive edge is legal |
| Stage transition leaks | Stop timers/tweens and clear transient actors | 20 restart/clear cycles without object growth |
| Impossible cycling goal | Complete iff all pad states match; cap enemy pressure | Forced-state completion and manual high-stage play |
| Pause desynchronizes timers | Explicit session state and director pause | Pause during move, freeze, respawn |
| Model context drift | Plan immutable; TODO authoritative; one task per pass | Session startup checklist |
| Scope expansion | No feature outside named phase without approval | Changed-files/requirements audit |
| Inconsistent exports | Official installed Godot version + real binary launch | Separate Windows/Linux release checklist |

**Performance budget:** 60 FPS target on a midrange desktop GPU or integrated graphics, 960×720 logical resolution, 28 pads, max four harmful enemies plus one Saboteur, one pickup, small bounded particles. Do not use per-frame pathfinding; compute routes only on enemy hops. Avoid allocations per frame and unbounded audio/particle instances. Treat memory increase after repeated restarts as a release blocker until investigated.

**Security/privacy:** Entirely offline; do not collect telemetry, bundle third-party analytics, execute shell commands in-game, or download assets at runtime. Persist only settings and scores under `user://`.

**IP:** No Q*bert sprite, names, enemy character art, arcade sound samples, logo, commercial music, screenshots, traced levels or promotional material. Mechanics inspire the design; presentation and expression must be independently made. Credit the historic reference in development notes, but do not market this as an official remake or sequel. Review asset licenses before distribution.

## 9. Release acceptance checklist

- [ ] Every phase 0–10 marked ACCEPTED with dated user approval
- [ ] Import/script/test suite clean on exact shipping Godot version
- [ ] All 10 stages and endless continuation verified manually
- [ ] Collision, fall, freeze, gates, score and bonus-life cases verified
- [ ] Settings and top-ten scores persist after restart and tolerate corrupt files
- [ ] Keyboard-only flow and high-contrast/reduced-motion settings verified
- [ ] No critical or high-severity open issues
- [ ] Exports launched on Windows and Linux, with recorded OS/test hardware
- [ ] Offline play, licensing/credits, README, checksums and release notes complete
- [ ] Measured FPS/memory results recorded; performance target met or disclosed

## 10. Local coding agent operating procedure

1. Open a fresh session with only this specification, `TODO.md`, and a short current-project file inventory. Keep the working context below the available 64K rather than dumping logs and every source file.
2. Read current status. Check the last accepted phase and any regression items. Inspect files that own the next task.
3. State the precise task, files to change, tests to run, and one potential risk **before coding**.
4. Make the smallest useful edit. Do not scaffold future systems. Do not rewrite accepted components merely to simplify a new feature.
5. Run an import/parse check and focused automated tests. Then perform or request manual play where needed. A claim of "tested" without command output or actual interaction is invalid.
6. If any test fails, diagnose and fix in the current phase; log unresolved cases and keep tasks unchecked.
7. Update `TODO.md` with actual results, modified files, decisions, and next step. Mark phase READY FOR REVIEW only after all checks pass.
8. Stop. Ask user to approve phase; only then mark ACCEPTED and begin next phase.

When no Godot executable is available, mark testing **BLOCKED**, give installation/execution instructions, and do **not** mark the phase ready merely because code looks plausible.

## 11. Master handoff prompt for Qwen 3.8 Flash Next IQ3

Copy the entire block below as the first message in your coding-agent session after placing this file in the project root.

```text
You are the sole implementation engineer for PRISM HOPPER, a Godot 4.x
GDScript desktop game. You are running with limited context. Favor correctness,
small diffs, and verified work over speed.

MANDATORY FIRST ACTIONS:
1. Read prism-hopper-godot-game-plan.md completely.
2. Read TODO.md completely. If absent, create it from Section 7 of the plan,
   with every phase/task unchecked and Phase 0 IN PROGRESS.
3. Inspect the actual Godot project and relevant source files; do not assume
   the file tree in the plan has already been created.
4. Identify the last explicitly ACCEPTED phase, current phase, known blockers,
   and the single next unchecked actionable task.
5. Do not begin a phase until all earlier phases are accepted by the user.

IMPLEMENTATION LIMIT:
- Implement only the next task or the current phase if explicitly authorized.
- Use Godot-native scenes, GDScript, and small focused files.
- Keep the specification as source of product requirements and TODO.md as the
  single source of implementation progress. No IMPLEMENTATION_STATUS.md.
- Preserve all working features and uncommitted user changes. Do not refactor
  accepted code without a documented reason and regression test.
- Never invent a passing test or imply a headless import proves gameplay.
- Look up version-sensitive APIs in official Godot documentation when needed.
- Avoid dependencies, network, copyrighted assets, and out-of-scope features.

FOR THIS TASK:
A. State phase, objective, exact files, proposed changes, test plan.
B. Implement the smallest complete change.
C. Run Godot import/parse and relevant automated checks when available.
D. Perform interactive test if accessible; otherwise label it NOT TESTED and
   request the exact human playtest steps before phase approval.
E. Fix encountered errors and rerun tests; log blockers honestly.
F. Update TODO.md with checked tasks only when verified, actual commands,
   outputs/results, files, known issues, decisions, and next action.
G. Output a concise report: phase/status, changes, tests PASSED/FAILED/NOT RUN,
   outstanding problems, manual steps, and request for approval.
H. STOP. Never self-approve and never advance to the next phase without the
   user's explicit approval.

If the project differs from the specification, report the conflict and propose
the smallest correction. Do not silently reinterpret game mechanics.
```

## 12. Recovery prompt

```text
STOP FEATURE WORK. Recover project state safely.

Read prism-hopper-godot-game-plan.md and TODO.md again. Inspect repository
status/diff and recent changes. Identify last user-accepted phase, current
unfinished task, any regressions, and the exact owner files. Do not overwrite
unrelated or user edits. If a test broke, reproduce it, isolate the smallest
cause, add/repair its regression test, and fix only that issue. Run the relevant
headless checks and request/manual-test gameplay as needed. Update TODO.md with
real evidence, current status, and the next SINGLE task. Report recovery
results and STOP for user approval. Do not skip tasks or self-approve phases.
```

## 13. References

1. [Q*bert — Wikipedia](https://en.wikipedia.org/wiki/Q%2Abert), especially Gameplay and Development. Historical reference; not an asset source.
2. [Godot official documentation — stable](https://docs.godotengine.org/en/stable/), verify APIs against the specific installed stable 4.x version during implementation.
3. [Godot documentation — command-line tutorial](https://docs.godotengine.org/en/stable/tutorials/editor/command_line_tutorial.html), for import/test/export syntax.

**End of specification.** Do not begin coding Phase 0 until the human instructs the implementation agent to start.
