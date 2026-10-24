<!--
Type: Phased game plan
Target: Godot 4.x, GDScript
Output: Complete Godot game project
Related: None
-->

# STARFALL CHRONICLES — Godot Game Implementation Plan

**Document role:** Immutable product specification and phased implementation handoff  
**Target implementer:** Qwen 3.8 Flash Next IQ3, usually 64K context  
**Engine:** Godot 4.x stable, GDScript, Windows/Linux, keyboard-first  
**Status:** Specification only; no game or `TODO.md` has been created  
**Reference:** [Ultima I: The First Age of Darkness](https://en.wikipedia.org/wiki/Ultima_I:_The_First_Age_of_Darkness)

## 1. Reference research and design boundary

Ultima was designed by Richard Garriott, released for Apple II in 1981, and substantially recoded in 1986. It is a single-player fantasy RPG with an overhead tile-map overworld, first-person dungeon exploration, character creation, random encounters, purchases, castle quests, survival resources, and an unexpected real-time space-combat sequence. Its campaign requires a collection of quest gems, space-combat accomplishments, discovery of a time machine, and a final confrontation. The world is divided into four land masses. The 1981 and 1986 versions differ in presentation and some enemy behavior; this plan is inspired by broad mechanics, not a faithful simulation of either edition.

**Documented mechanics:** Four races and four classes, stat allocation, hit points/food/experience/coin, town merchants, consumable spells, weapon and armor upgrades, exploration quests and dungeon kill quests, first-person dungeon perspective, castles, travel between continents, a space shooter, and a final time-travel confrontation. Food depletion can kill the character, and encounters generate rewards. The reference describes space combat as real-time and the primary map as top-down.

**Original design decisions, not historical claims:** All numeric values, map dimensions, character species/classes, locations, enemy behavior, progression thresholds, specific weapon statistics, quest chain, save format, space mechanics, and difficulty tuning below are invented for **Starfall Chronicles**. These are deliberate implementation requirements, not statements about Ultima I.

**Core appeal to preserve:** The pleasure of charting a large world, managing supplies, growing a fragile adventurer, descending into dangerous maze-like ruins, returning to towns for better gear, following quests across continents, and unexpectedly taking the adventure into the stars.

**Legal/creative boundary:** Do not use names such as Sosaria, Mondain, Lord British, Ultima, or recognizable character likenesses in the finished game. Do not copy maps, dialogue, quests verbatim, interface art, sprites, sound, theme music, box art, or distinctive title branding. All art and audio must be original or appropriately licensed with attribution records. Historical reference is solely design research; avoid marketing the product as an official sequel or remake.

## 2. New game concept

**Title:** Starfall Chronicles  
**Tagline:** *An old-world journey beyond the edge of the sky.*

An explorer on the shattered planet Asterra must recover four Resonance Seals from ancient wardens, clear buried labyrinths, and earn a pilot's license in orbital combat before reaching a time-bent citadel to stop the entity called the Hollow Regent. The first half is a compact open-world RPG; the second reveals a short arcade-style orbital mission that opens the final act.

**Visual direction:** Clean, original 16×16 pixel-inspired tiles, restrained jewel-toned palette, readable silhouettes, chunky font, dark dungeon walls, bright orbit HUD. Begin with self-drawn primitives and flat colors; replace only after functionality is stable. **Audio:** original low-key exploration loops, sparse dungeon drones, energetic original orbital rhythm, short unobtrusive UI cues; no external copyrighted material.

**Scope limits:** Solo only; no online play, procedural world generator, party system, crafting, dialogue tree, dynamic economy, complex spellbook, voice acting, or fully 3D level architecture. Dungeons use a deterministic 2D grid rendered as a faux-3D corridor by drawing depth slices in `Control._draw()`. Orbital combat is a separate 2D arcade scene. This choice preserves the changes in viewpoint while limiting scene and asset complexity.

## 3. Complete game rules

### 3.1 Game loop and progression

1. Create character and start at Haven Gate with 50 food, 40 HP, 20 coins and a starter blade.
2. Travel by one tile per key press on a finite overworld; explore towns, castles, ruins, and dungeons.
3. Move, restock food, fight enemies, collect XP and coins; purchase better gear.
4. Complete a landmark challenge and a dungeon target for each of four wardens to gain a Resonance Seal.
5. Visit the launch port, acquire a flight pass, complete an orbital arena mission, and receive star coordinates.
6. Find the Chrono Spire, survive a short final dungeon, destroy the Void Engine and defeat the Hollow Regent.
7. Show victory summary with turn count, defeats, relics and ending text. Continue from last manual save is possible before final encounter.

**MVP** ends after Phase 5: one fully playable continent with a town, castle, dungeon, monsters, shopping, a warden quest, reward, saving and loading, title/pause/death screens. Later phases expand to the four-continent campaign and orbital finale.

### 3.2 Character creation and stats

Choose a name (1–16 safe printable characters), lineage and vocation. Lineages: **Wayfarer** (+2 Endurance), **Sylvan** (+2 Agility), **Stoneborn** (+2 Might), **Starling** (+2 Insight). Vocations: **Vanguard** (+2 Might), **Seer** (+2 Insight), **Ranger** (+2 Agility), **Guardian** (+2 Endurance). Base attributes each 5; distribute 8 extra points, maximum 12 before bonuses. Derived max HP = 20 + 4×Endurance; attack = weapon power + Might; guard = armor value + floor(Endurance/3); hit chance = clamp(75 + 2×(Agility−enemy_evasion), 55, 95) percent. Magic ability exists only as purchased single-use spark charges: ranged 5 + Insight damage, consumes one charge, range 3 on overworld / visible corridor enemy in dungeon. No mana pool.

Start level 1, XP 0. Level threshold for *next level* = 25×current level; on level-up gain +4 max HP and +4 HP immediately; every 2 levels gain +1 attribute point assigned through a simple modal. Cap level 10 for this game. Coins integer, food cap 99, inventory capacity 24 units; equipment has separate slots. Health 0 or food 0 means death. Persist RNG seed where practical, not the global RNG object.

### 3.3 Movement, turns, encounters and survival

- Overworld grid: four fixed 48×32 maps with 16×16 tiles; visible camera displays a centered 25×15 tile region, clipped to map edges. Logical window 640×360, 2D `canvas_items` stretch with integer pixel-like art scaling and letterboxing. The map content viewport can occupy ~480×320 while HUD uses the remaining area.
- WASD/arrows: one cardinal tile at a time with 0.13 second repeat delay when held; no diagonal movement. No physics impulses. Block mountains and deep water on foot; shallow water costs two movement ticks. A ship obtained later allows water traversal using the same grid, with no ship battles.
- Every two successful traversals consume 1 food (no food burn in towns, castles, shops, or menus). Failed moves do not count. Every successful world step advances a world turn and invites one encounter roll: 10% plains, 16% forest, 20% bog, 0% safe town-adjacent tiles. Use `RandomNumberGenerator` with recorded seed.
- Battles on the overworld are turn-based: spawn one visible enemy adjacent if free, otherwise defer; player moves or attacks; enemy moves one tile toward player if cardinal path open, attacks when adjacent. One enemy action per player world action. No repeated attacks during held-key UI input. Max six concurrent enemies, each auto-despawns beyond 12 tiles for 12 turns without a quest tag. No unbounded spawn queues.
- `interact` on an entrance tile opens a town/castle/dungeon/port scene; escape/back returns to prior map coordinate. Context interaction is explicit; entering a tile alone does not silently open a scene.
- Damage = max(1, attacker_attack−target_guard + integer random variation from −2 through 2). Invulnerability is not part of turn-based combat; enforce one damage resolution per action ID. A successful hit awards XP/coins only on a single confirmed enemy death. If player HP ≤0, transition to death screen; death does not overwrite a valid save.

### 3.4 Game locations

| Region | Terrain / special destination | Warden challenge | Dungeon target |
|---|---|---|---|
| Dawnreach | Meadows, forests, Haven Gate, Glass Cairn | Activate beacon at Glass Cairn | Defeat a Burrow Wraith |
| Cinderfall | Ash plains, bogs, Ember Market, Red Obelisk | Reach Red Obelisk | Defeat an Ash Sentinel |
| Mistvale | Marsh, lakes, Lumen Town, Echo Shrine | Attune Echo Shrine | Defeat a Mire Watcher |
| Frostmere | Snowfields, mountains, Northwatch, Sky Needle | Reach Sky Needle | Defeat an Ice Keeper |

Each map has exactly one settlement, one warden keep, one dungeon, and one landmark. Dawnreach additionally hosts the starter safe point; Frostmere contains the launch port; Mistvale reveals a water-access route. Ferry gates join continents with explicit entry/exit coordinates. Entire map is handcrafted in readable text grids or Godot TileMapLayer scenes; no random topology. Keep layouts in separate `.tres`/scene resources, with stable IDs. The player can backtrack; dungeon targets respawn until the corresponding quest reward is claimed, then remain cleared.

**Settlement UI:** Food (+10 for 5 coins), medkit (+15 HP for 8), spark charge (10), blade upgrades (power 3/5/7 at 15/45/90), armor upgrades (guard 1/2/4 at 12/40/85). Show confirmation and enforce funds/inventory caps. No selling; replacement equipment doesn't occupy an inventory slot. Player can rest at inn for 10 coins to restore HP to maximum. Shops have static prices with no Charisma economy.

**Warden UI:** Display both tasks: discover landmark and defeat named dungeon target. Upon both flags, interacting with warden grants one unique seal, 30 coins, and advances campaign quest flag. Never grant same seal twice. A castle also permits manual save, and save is available through pause outside combat. Journal always lists exact objectives and region destinations.

### 3.5 Dungeons and faux-3D presentation

Each dungeon is a fixed 15×15 grid (`#` wall, `.` floor, `E` exit, `M` monster target, `C` cache) using breadth-first path validation before handoff. Enter at fixed cell facing north. `turn_left`/`turn_right` rotate facing by 90°; `forward`/`back` move one cell if open. Draw corridors up to 4 cells deep using solid polygons and flat shading in a `Control` canvas; draw wall faces front-to-back and left/right apertures based solely on map occupancy. Draw compass, small optional automap only after an explorer token is collected, and encounter sprite billboard as a simple flat overlay at the nearest reachable position. This is not raycasting or 3D physics.

Dungeon actions consume a dungeon turn; every 4 successful dungeon moves consumes 1 food. A room may hold one deterministic enemy by cell; the target enemy does not wander through walls. On each player dungeon turn, an enemy in the same or next cardinal cell attacks if not dead (one action per turn). `attack` in dungeon strikes an enemy immediately in facing cell, with the same damage rules; spark charges may strike up to three unobstructed forward cells. Chest/cache awards coins or food once; floor exit returns to entrance coordinate and retains chest flags in save. On player death transition to game over. Dungeon maps do not regenerate.

### 3.6 Enemies and tactics

| Type | HP | Attack | Guard | XP / coins | Behavior |
|---|---:|---:|---:|---|---|
| Thornling | 10 | 4 | 0 | 6 / 2 | Chase within 5 tiles, otherwise wait |
| Bog Skulk | 15 | 6 | 1 | 9 / 4 | Chase within 6; avoids mountains/water |
| Iron Shade | 22 | 8 | 2 | 14 / 7 | Approach within 5, attacks adjacent |
| Burrow Wraith | 30 | 8 | 2 | 20 / 12 | Stationary dungeon target; one melee strike/turn |
| Ash Sentinel | 36 | 10 | 3 | 25 / 15 | Dungeon target; stronger melee |
| Mire Watcher | 42 | 11 | 3 | 30 / 18 | Dungeon target; stronger melee |
| Ice Keeper | 48 | 12 | 4 | 35 / 20 | Dungeon target; stronger melee |

Spawn zone selects from Thornling/Bog Skulk/Iron Shade by continent; select weighted probabilities 60/30/10, shifting to 40/35/25 after two seals. No advanced A*: shortest Manhattan-progressing step with horizontal-first preference, fallback vertical, always bounds/collision checked. Enemies occupy unique world cells. Attack adjacency is Manhattan distance 1. The final boss uses a separate scripted state machine (see §3.8).

### 3.7 Orbital mission

The orbital segment is a separate 640×360 arcade scene: player shuttle near the bottom moves horizontally using left/right; `attack` fires one upward bolt every 0.22 seconds; enemies enter from top and move in straight or sine-pattern paths, fire simple downward shots. A 90-second mission requires 15 drones destroyed and the player surviving until the gate charge reaches 100% (charge grows +1 per confirmed drone kill and +1 per surviving second, capped 100). Ships can take 5 hits, with 0.8-second post-hit invulnerability; collision bullets removed on contact/out of bounds. Two enemy types: Scout (1 HP, straight movement, low rate of fire) and Spinner (2 HP, sinusoidal drift, slower shots). Spawn interval from 1.5s to 0.9s as mission time progresses, max 10 enemy ships and 40 projectiles. Failure returns to port with the mission replayable for a small 5-coin fuel fee. Success permanently sets `orbit_clear` and awards star coordinates. No spaceship purchasing or full space navigation needed.

### 3.8 Endgame

If all four seals and `orbit_clear` are true, Chrono Spire entrance appears at a marked Dawnreach tile. Inside is one 11×11 dungeon and a final boss screen. Boss has 100 HP and three deterministic phases: HP 100–66: telegraphed strike every second player action, 12 attack; 65–31: alternating melee and a spark that applies 8 damage unless player chooses `guard` (reduces damage by 50%); 30–0: attack every action at 14 damage, but `guard` blocks 50%. Present phase instruction in combat log. Player can `attack`, `spark`, `guard`, or `use_medkit` per turn; boss responds once after valid player action. Winning requires first interacting with and disabling the Void Engine (one action) after boss reaches 0 HP; then show ending. Prevent victory on boss HP alone. Before encounter, show clear one-way warning and allow retreat. Final area contains an inn-like full heal once for 20 coins, avoiding impossible unwinnable state.

### 3.9 Menus, controls, HUD and accessibility

| Action | Default | Context |
|---|---|---|
| Move | WASD or arrows | Overworld, ship, menus |
| Interact / confirm | E / Enter | World locations, menus |
| Attack / fire | Space | Combat, orbit |
| Spark | F | Combat with charges |
| Turn dungeon camera | Q / R | Dungeon |
| Open journal | J | World / safe interior |
| Inventory | I | World / safe interior |
| Pause / back | Escape | Anywhere permitted |
| Guard | G | Boss battle |

Title: New Game, Continue (disabled if no save), Settings, Quit. Pause: Resume, Save (disabled during orbit and boss), Settings, Main Menu. Game Over: Load Last Save, New Game, Main Menu. Settings: master/music/SFX volumes (0–100), screen mode windowed/fullscreen, integer pixel art mode on/off, high-contrast UI, reduce screen flash. Persist settings separately from run state. HUD shows HP, food, level, XP, coins, seal count, active objective, current region and log of last 4 combat events. Use clear focus borders and mouse-free navigation. Rebindable keys not required for v1, but action map makes later rebind possible. Avoid critical red/green-only cues.

## 4. Technical architecture

**Godot version:** Select an installed stable Godot 4.x at project start; record exact version in `TODO.md`. Before implementing version-specific methods, consult official Godot 4 documentation for that minor release: https://docs.godotengine.org/en/stable/. Do not silently use Godot 3-era `TileMap`, `yield`, or deprecated APIs. Use `TileMapLayer` if chosen, and verify its creation/import behavior locally.

**Architectural boundaries:**

- `GameState.gd` autoload owns plain serializable player/campaign data, current coordinates, completed quests, dungeon flags, and notification signals. It does **not** own scene nodes or render UI.
- `SceneRouter.gd` autoload changes scenes with explicit destination payload, holds return coordinates during a single transition, and ensures a single active gameplay scene.
- `SaveService.gd` autoload serializes `GameState.to_dict()` to `user://save_v1.json`, writes to a temporary file then renames only after successful serialization, validates fields/IDs and save schema version before load; invalid data offers menu error, not crash. Keep one manual slot plus optional `.bak` last-good copy. Settings stored at `user://settings_v1.cfg` using ConfigFile.
- `PlayerData.gd` is a `RefCounted` data object with attributes, derived stats, inventory and equipment. `CombatRules.gd` is a pure helper returning one combat outcome per action; never mutates UI nodes.
- Location scripts emit signals (`interaction_requested`, `combat_resolved`, `quest_completed`, `player_died`) consumed by parent scene/controllers; HUD observes GameState signals. No scene-specific scripts should directly reference another scene's nodes.
- Overworld controller owns turn scheduler and map collision occupancy. Dungeon controller owns orientation/grid turns. Orbit controller owns real-time spawns/projectiles. Boss controller owns boss phases. These modes never run concurrently.
- Resources under `data/` store fixed enemy, item, region and dungeon definitions. Use JSON or simple constant dictionaries for early phases; move to custom Resources only if necessary. Never embed duplicate balance values in scene scripts.

### Directory layout (create incrementally, not all at Phase 0)

```text
starfall_chronicles/
  project.godot
  starfall-chronicles-godot-game-plan.md
  TODO.md                         # created by implementation agent
  scenes/
    main_menu.tscn
    character_create.tscn
    overworld.tscn
    settlement.tscn
    warden_keep.tscn
    dungeon.tscn
    orbital_mission.tscn
    boss_encounter.tscn
    ending.tscn
    ui/hud.tscn
    ui/pause_menu.tscn
    ui/journal.tscn
  scripts/
    autoload/game_state.gd
    autoload/scene_router.gd
    autoload/save_service.gd
    data/player_data.gd
    logic/combat_rules.gd
    logic/quest_rules.gd
    world/overworld.gd
    world/world_enemy.gd
    world/settlement.gd
    world/warden_keep.gd
    dungeon/dungeon_controller.gd
    dungeon/corridor_renderer.gd
    orbit/orbital_controller.gd
    orbit/orbit_projectile.gd
    boss/boss_controller.gd
    ui/hud.gd
  data/
    regions/                  # fixed grids / coordinates
    dungeons/                 # fixed grids
    items.json
    enemies.json
    quests.json
  assets/
    sprites/                  # original assets
    fonts/                    # licensed / documented
    audio/music/
    audio/sfx/
  tests/
    smoke_test.gd
    combat_test.gd
    quest_test.gd
    save_test.gd
  docs/
    ASSET_LICENSES.md
```

**Collision strategy:** Grid collision on overworld and dungeon uses one occupancy lookup and explicit tile enums, not `CharacterBody2D` physics. Orbit uses `Area2D` layers: layer 1 player; 2 enemy ships; 3 player bolts; 4 enemy bolts; 5 environment; masks player [2,4,5], enemy [1,3,5], player bolts [2,5], enemy bolts [1,5]. Collision results are deduplicated before `queue_free()`. Only orbit needs physics callbacks. UI uses mouse filter and focus navigation, not collision layers.

**Save contract:** `schema_version`, `player` (name, lineage, vocation, level, xp, base stats, hp, max_hp, food, coins, equipment IDs, charges, medkits), `world` (region ID, grid coordinates, facing, turn counter, vehicle), `quests` (four landmark flags, four target flags, four seals, orbit_clear, final_boss_defeated), `dungeons` (opened chest IDs, defeated target IDs), `meta` (seed, session turns). IDs must map to known definitions; ignore unknown harmless keys, reject missing mandatory keys or invalid coordinates gracefully. Save only when outside orbital combat/boss fights, with scene transition complete. Save atomicity tested with corrupted JSON and missing file.

**Signals:** `GameState.stats_changed`, `GameState.quest_changed`, `GameState.location_changed`, `GameState.run_ended`, `SceneRouter.scene_changed`, scene-owned `action_committed` and `interaction_requested`. Main controllers change state first, then emit once; listeners render but never mutate game rules in response. All objects created by spawners free on scene exit; no persistent references to freed scene nodes.

## 5. Phased implementation contract

**Universal phase rules:** Implement one checkbox or approved phase at a time. Every phase must leave the Godot project runnable. Before editing, read this file, `TODO.md`, current relevant source and tests. Commit/checkpoint working state if git is in use. Run the phase's specific tests and at least the headless import/parser check. Do not advance until the user explicitly approves. A passed import test does not prove playable behavior. On failure: reproduce, isolate, repair the smallest affected files, rerun current and prior regression checks, document before proceeding. Record actual results, not assumed passes.

**Universal phase completion gate (copy into each phase in TODO):**

```markdown
- [ ] All required features implemented
- [ ] All tasks individually verified
- [ ] Godot imports without errors
- [ ] No GDScript parsing errors
- [ ] Gameplay tested in Godot editor or desktop binary
- [ ] No critical regressions in accepted phases
- [ ] Known issues documented
- [ ] TODO.md updated with commands and results
- [ ] Phase reviewed by user and explicitly approved
```

### Phase 0 — Bootstrap and verification harness

**Objective:** Empty project can launch a placeholder menu and run tests. **Dependencies:** Godot 4.x installed. **Files:** `project.godot`, `scenes/main_menu.tscn`, `scripts/autoload/{game_state,scene_router,save_service}.gd` (minimal stubs), `tests/smoke_test.gd`, `TODO.md`.

**Tasks:** [ ] Confirm installed version and renderer (Compatibility initially); [ ] create folders only as needed; [ ] set 640×360 window/stretch and input actions from §3.9; [ ] implement a minimalist title scene with a visible start placeholder; [ ] add stubs/autoload order GameState→SceneRouter→SaveService; [ ] add headless smoke test scene instantiation and script syntax checks; [ ] copy plan to project root; [ ] initialize TODO from §6.

**Integration:** Menu must load as main scene, stub dependencies must not trigger scene changes on boot. **Accept:** Visible title with no console errors; keyboard menu focus; headless import exit 0; smoke test exit 0. **Test:** `godot --headless --path . --editor --quit`; `godot --headless --path . --script res://tests/smoke_test.gd` after test extends SceneTree; open in editor and click/keyboard navigate. **Common failures:** wrong binary name, null main scene, unresolved autoload, nonexistent file path. **Done:** universal gate plus version and test commands recorded.

### Phase 1 — Character and one-region overworld

**Objective:** Start a character and explore a collision-correct Dawnreach region. **Dependencies:** P0 accepted. **Files:** `scenes/character_create.tscn`, `scenes/overworld.tscn`, `scripts/data/player_data.gd`, `scripts/world/overworld.gd`, `data/regions/dawnreach.*`, update game state/menu/test.

**Tasks:** [ ] Implement allocation validation and lineage/vocation bonuses; [ ] New Game creates default player and opens overworld; [ ] draw a handcrafted 48×32 Dawnreach grid and screen-follow camera; [ ] classify walkable and blocked tiles; [ ] implement repeat-limited grid movement and turn count; [ ] implement food every two moves and starvation death placeholder; [ ] display HP/food/name/position HUD labels; [ ] mark Haven Gate and entrance tiles but no interactions yet.

**Integration:** `GameState.player` is authoritative; overworld owns input and requests state changes; UI observes signals. **Accept:** Character points always total 8, cannot go negative; walls/water block; 10 successful moves reduce food exactly 5; screen never scrolls beyond grid; restart begins fresh state. **Test:** manually walk 20 steps, push boundaries, hold keys, simulate food=1 and move twice, verify death state without crash. **Common failures:** off-by-one grid conversion, key-repeat movement while in menus, food drained on blocked steps. **Done:** universal gate and movement/food checks recorded.

### Phase 2 — Overworld combat and growth

**Objective:** Repeatable risk/reward loop with turn-based fights. **Dependencies:** P1 accepted. **Files:** `scripts/logic/combat_rules.gd`, `scripts/world/world_enemy.gd`, `data/enemies.json`, `tests/combat_test.gd`, relevant scene/HUD updates.

**Tasks:** [ ] Add seeded spawn roll/biome probabilities; [ ] enforce occupancy/6-enemy cap and decay; [ ] implement enemy turn scheduler exactly once per successful player action; [ ] attack in adjacent facing direction, hit chance and damage; [ ] show attack/kill feedback; [ ] award coins/XP only once, level-ups and HP growth; [ ] player death screen with New Game; [ ] implement an optional debug overlay (F3) for position/turn/enemy count.

**Integration:** Every action gets monotonically increasing ID; combat result resolved once by pure CombatRules; no combat node writes global fields directly. **Accept:** No enemy can spawn inside obstacle/player; 50-turn manual run stable; kill rewards once; bounds and caps hold; player death presents functional restart. **Test:** seeded 100-turn simulation/unit calculations, manual adjacent attack, 3 consecutive deaths/restarts, check no leaked enemies. **Common failures:** double turn actions, duplicate death signal, enemies attacking across blocked cells. **Done:** universal gate and deterministic tests recorded.

### Phase 3 — Town economy, equipment and UI

**Objective:** Exploration pays for meaningful upgrades and supplies. **Dependencies:** P2 accepted. **Files:** `scenes/settlement.tscn`, `scripts/world/settlement.gd`, `data/items.json`, `scenes/ui/{hud,journal,pause_menu}.tscn`, `scripts/ui/hud.gd`, settings/menu updates.

**Tasks:** [ ] Interact with Haven Gate; [ ] food/heal/spark/gear price list and funds checks; [ ] inn heal; [ ] inventory and equipment stat reflection; [ ] journal with current objectives placeholder; [ ] pause/resume and settings sliders; [ ] return to exact world coordinate; [ ] show readable action log; [ ] no turn/food drain while in menus.

**Integration:** Store transaction is atomic: validate item/funds/caps, debit, grant, emit stats_changed; router preserves region and entrance coordinate. **Accept:** 0 coins cannot buy; 99 food cannot exceed cap; upgraded blade changes damage; buying never duplicates item from double-confirm; pause stops movement; return position exact. **Test:** buy each item, attempt unaffordable item, spam Enter, verify prices, pause and resume while key held. **Common failures:** double input dispatch, stale UI labels, changing scene before recording return point. **Done:** universal gate and economy checks recorded.

### Phase 4 — First-person dungeon and first warden target

**Objective:** Explore a fully playable fixed dungeon with a quest monster. **Dependencies:** P3 accepted. **Files:** `scenes/dungeon.tscn`, `scripts/dungeon/{dungeon_controller,corridor_renderer}.gd`, `data/dungeons/dawnreach.*`, dungeon tests and interaction updates.

**Tasks:** [ ] Build/connect 15×15 Dawnreach grid with guaranteed path; [ ] implement facing, turn and step, wall checks; [ ] corridor drawing at 1–4 depth and compass; [ ] target monster at known cell, attack/spark damage; [ ] food every 4 dungeon moves; [ ] one-time cache; [ ] exit returns to map; [ ] quest target flag persists during run; [ ] safe camera fallback when view ray reaches wall.

**Integration:** Dungeon state stored by stable ID, target death calls QuestRules through GameState; no duplicate loot when reentering. **Accept:** Player can navigate exit→target→exit; wall never traversable; turn left four times returns north; cache gives reward once; target death flag stays set. **Test:** follow documented coordinate route, check rendered front/side walls on all directions, reenter dungeon three times, manually test death. **Common failures:** N/E/S/W rotation mismatch, index outside grid, fake-3D walls drawn in wrong order. **Done:** universal gate and route screenshot/notes recorded.

### Phase 5 — First full quest, save/load, MVP

**Objective:** Complete one rewarding RPG loop from New Game to first Resonance Seal and save it. **Dependencies:** P4 accepted. **Files:** `scenes/warden_keep.tscn`, `scripts/world/warden_keep.gd`, `scripts/logic/quest_rules.gd`, `data/quests.json`, `scripts/autoload/save_service.gd`, `tests/{quest_test,save_test}.gd`, all relevant UI.

**Tasks:** [ ] Activate Glass Cairn landmark via interact; [ ] warden displays two independent objectives; [ ] grant Dawnreach seal only when both true; [ ] prevent repeat grant; [ ] implement versioned manual save, Continue, load, pause save, invalid-save error; [ ] persist stats, location, enemy-independent world state, quest flags, dungeon caches; [ ] add clear MVP victory message after first seal without ending full game; [ ] perform fresh install/new-save replay test.

**Integration:** Quest IDs and save schema are constants, save copy is validated before replacing current file. **Accept:** Full new-character→town→dungeon→landmark→warden→seal flow works; reload retains HP/food/equipment/seal/location; corrupted save handled; no seal duplication. **Test:** fresh play session 15+ minutes; exit/relaunch/Continue; intentionally malformed local test fixture (not real save) fails gracefully; all prior tests still pass. **Common failures:** JSON Vector2 serialization, missing save defaults, loading into wrong scene, awarding quest twice. **Done:** universal gate, user approval, explicit **MVP ACCEPTED** marker in TODO; do not proceed without it.

### Phase 6 — Four-continent campaign and transit

**Objective:** Full exploration/quest path across four varied regions. **Dependencies:** P5 accepted. **Files:** `data/regions/{cinderfall,mistvale,frostmere}.*`, `data/dungeons/{cinderfall,mistvale,frostmere}.*`, region/quest definitions, world map and dungeon updates; tests.

**Tasks:** [ ] Add three 48×32 maps with ferry boundaries; [ ] add one settlement/keep/landmark/dungeon per region; [ ] add named dungeon target stats, terrain variants and enemy probabilities; [ ] include water-travel unlock via a 30-coin skiff purchase in Mistvale with no other vehicle upgrades; [ ] present region/quest journal entries; [ ] award each of four unique seals; [ ] add quest audit UI showing 4×2 objectives; [ ] save/restore intercontinental position and skiff.

**Integration:** Shared scenes parameterized by stable region IDs; no copy/paste region scripts; save rejects unknown region. **Accept:** All four seals obtainable in one run, no unreachable required tile, land/water travel respected, ferry roundtrips preserve correct spawn and saves. **Test:** scripted BFS connectivity check per map; manual complete all 8 objectives, inspect journal and save/restore from each region. **Common failures:** wrong ferry spawn, softlock behind water, copied quest IDs, asset/path casing on Linux. **Done:** universal gate and full quest route documented.

### Phase 7 — Orbital shooter mission

**Objective:** Playable distinct space interlude gated behind four seals. **Dependencies:** P6 accepted. **Files:** `scenes/orbital_mission.tscn`, `scripts/orbit/{orbital_controller,orbit_projectile}.gd`, launch-port UI, save/quest tests.

**Tasks:** [ ] Launch requires four seals and 5+ coins fuel; [ ] shuttle horizontal movement, fire cooldown, damage/invulnerability; [ ] Scout and Spinner behaviors; [ ] timed 90s mission, 15-kill plus charge criteria, enemy/bullet caps; [ ] HUD timer/kills/charge/hull; [ ] win awards orbit_clear once; [ ] loss returns to port with retry; [ ] Escape opens pause but disables save.

**Integration:** Orbit scene alone creates/disposes projectiles; GameState only receives single result at mission end. **Accept:** Mission winnable and losable; no bullets survive scene exit; speed stable at 30/60/144 Hz due delta movement; orbit_clear persists across reload. **Test:** win/loss at least once each, hold fire for 3 min, inspect node count, verify no double awards. **Common failures:** unbounded bullets, duplicate Area2D hits, timers drifting with FPS, winning at 14 kills. **Done:** universal gate plus frame-rate test recorded.

### Phase 8 — Chrono Spire, boss and endings

**Objective:** Entire campaign can be finished. **Dependencies:** P7 accepted. **Files:** `data/dungeons/chrono_spire.*`, `scenes/{boss_encounter,ending}.tscn`, `scripts/boss/boss_controller.gd`, world flags/UI/tests.

**Tasks:** [ ] Gate Spire on all four seals + orbit_clear; [ ] add validated 11×11 final corridor; [ ] warning/retreat and one-time heal purchase; [ ] turn-based boss three phases with attack/guard/spark/medkit; [ ] defeat boss then separate Void Engine interaction; [ ] victory screen; [ ] return to title after results; [ ] enforce save restriction inside final boss.

**Integration:** End screen only after both `final_boss_defeated` and `engine_disabled`; partial victory cannot be loaded from midfight. **Accept:** Boss has valid tactics and telegraphs; defeat/death/retreat paths work; credits appear only on engine disable; new run resets all progress. **Test:** three manual boss trials with varied levels/equipment, negative gate test, reload before encounter, verify no softlock. **Common failures:** phase transitions at wrong HP, boss takes action twice, victory fires on HP 0, no heal route. **Done:** universal gate and full playthrough report.

### Phase 9 — Original art, sound, feel and access

**Objective:** Replace functional placeholders without altering proven rules. **Dependencies:** P8 accepted. **Files:** `assets/sprites/*`, `assets/audio/*`, `docs/ASSET_LICENSES.md`, menus/HUD/themes and relevant scenes.

**Tasks:** [ ] Original consistent sprite tiles/entities/UI icons; [ ] licensed readable font with provenance; [ ] original exploration/dungeon/orbit tracks; [ ] original hit/chest/shop/quest/endgame cues; [ ] restrained screen shake/particles with reduce-flash option; [ ] high-contrast HUD and focus borders; [ ] volume controls and fullscreen toggle persist; [ ] rebalance food/coin curve based on playtest telemetry; [ ] preserve game rules interfaces.

**Integration:** Art references paths only; audio bus names `Master`, `Music`, `SFX` fixed; missing audio may log warning but must not crash. **Accept:** All art original/licensed, accessible without sound, no unreadable text at 640×360, settings survive restart, unchanged combat/save tests. **Test:** screen at 720p/1080p and Linux/Windows, play with audio muted, reduced flashes, keyboard-only menu traversal. **Common failures:** blurry pixel scale, UI clipped by stretch, sound missing at export, altered collider size from sprite replacement. **Done:** universal gate plus license manifest.

### Phase 10 — Release validation and desktop exports

**Objective:** Stable v1.0 build with complete documentation. **Dependencies:** P9 accepted. **Files:** export presets, `README.md`, final tests, package metadata, final `TODO.md` notes.

**Tasks:** [ ] Run all automated and manual regression checks; [ ] complete a fresh save→campaign→orbit→boss playthrough; [ ] check no duplicate rewards/softlocks; [ ] profile memory and FPS in populated map/orbit; [ ] run exports for Windows and Linux using installed templates; [ ] launch both binaries (or note unavailable host platform honestly); [ ] provide controls, credits, attribution, known issues and installation instructions; [ ] tag release once user approves.

**Integration:** Final assets and data included in export, `user://` paths not inside application bundle. **Accept:** No critical/high severity issue, stable 60 FPS target on ordinary desktop at 1080p, <500 MB RAM typical target, no unbounded objects, clean startup/load/quit, complete start-to-finish campaign, valid export artifacts. These performance targets are goals, not claims until measured. **Test:** `godot --headless --path . --editor --quit`, automated scripts, interactive smoke on both systems, export and launch checks, long-session 30-min leak test. **Common failures:** case-sensitive paths, absent export templates, missing imported resources, saves written to read-only executable folder. **Done:** universal gate plus user approval of v1.0.

## 6. Mandatory TODO.md template

The implementation agent **creates** `TODO.md` during Phase 0. It is the only live progress tracker; this plan remains the permanent specification. Copy the following template, expanding each phase's tasks from §5 verbatim as checkboxes (do not stop with placeholders). Do not create `IMPLEMENTATION_STATUS.md`.

```markdown
# Starfall Chronicles — Implementation Tracker

Specification: starfall-chronicles-godot-game-plan.md
Godot version: [record exact version]
Current phase: 0 — Bootstrap
Current phase status: IN PROGRESS
Last accepted phase: None
Build status: NOT TESTED
Next action: Confirm installed Godot version and create minimal project
Last updated: [date/time]

## Allowed status values
- IN PROGRESS: tasks are actively being implemented or tested
- BLOCKED: cannot proceed; reason/action recorded
- READY FOR REVIEW: all checks pass, awaiting user's decision
- ACCEPTED: user explicitly approved the phase

## Phase task checklist
### Phase 0 — Bootstrap
- [ ] Confirm engine and renderer
- [ ] Create project settings/input actions
- [ ] Create minimal menu and autoloads
- [ ] Create smoke test and run it
- [ ] Populate this TODO with Phases 1–10 tasks from specification
#### Phase Completion Gate
- [ ] All required features implemented
- [ ] All tasks individually verified
- [ ] Godot imports without errors
- [ ] No GDScript parsing errors
- [ ] Gameplay tested
- [ ] No critical regressions
- [ ] Known issues documented
- [ ] TODO.md updated
- [ ] Phase reviewed and approved by user

### Phases 1–10
[Copy the full task checkbox list and the nine-item phase completion gate for EACH phase from section 5, preserving unfinished items as unchecked.]

## Latest test results
| Date | Phase | Command / manual case | Expected | Actual | PASS/FAIL/BLOCKED |
|---|---|---|---|---|---|

## Known issues and blockers
| ID | Severity | Reproduction | Impact | Owner / next fix |
|---|---|---|---|---|

## Technical decisions
| Date | Decision | Reason | Files affected |
|---|---|---|---|

## Session history (append, never erase)
- [date] Phase/task: ... ; edits: ... ; tests: ... ; next step: ...

## Approval history
| Phase | Status | User approval quote/reference | Date |
|---|---|---|---|
```

**Rules:** Mark checkboxes only on evidence. Each task gets testing results; a generated file alone does not count. If regression appears, reopen the related task and set current phase BLOCKED if critical; do not erase old results. READY FOR REVIEW is never ACCEPTED. Record exactly what the user approved (explicit approval required), then advance at next session. For a context reset, the current phase, last acceptance, next action, tests and session history must be sufficient to resume safely. Keep all historical entries; add new records rather than replacing old ones.

## 7. Quality gates and repeatable test strategy

**Automated:** Use Godot headless import `godot --headless --path . --editor --quit`, then `godot --headless --path . --script res://tests/smoke_test.gd` and similarly for named tests (each extends SceneTree, exits with 0 on pass, nonzero on assertions). If executable is called `godot4`, adapt command and record the exact binary. Tests should avoid scene-tree frame timing dependence except smoke checks. Validate map graph connectivity using BFS from start to required locations. Unit test combat damage, XP once-only, item pricing, quest completion order, serialization roundtrip and corrupted JSON.

**Manual:** At every phase run the editor-visible scene, use real keyboard controls, watch errors/warnings, pause/unpause, quit/relaunch, and verify previous accepted mechanics. On Phases 5 and 10 perform full end-to-end playthroughs with checkpoints and record time taken, FPS, max enemy/bullet counts, failure/retry behavior and bugs. Run at 30/60/144 Hz where relevant and monitor idle vs long-session memory. A headless parser pass is **never** called a gameplay pass.

**Test reporting format:**

```text
Phase/task:
Files modified:
Commands executed (exact):
Results (exit codes, actual observations):
Manual gameplay checks:
Prior-phase regression checks:
Unresolved issues / risks:
TODO.md changes:
Status: IN PROGRESS | BLOCKED | READY FOR REVIEW
Requested user review: [exact question]
```

**Recovery policy:** On failure, stop feature work; reproduce on current files, read errors, make focused fix, rerun failing check and relevant accepted-phase checks, update TODO. Do not rewrite an accepted subsystem merely because a more elegant design seems possible. Refactors require a specific defect or approved scope, targeted tests, and rollback path.

## 8. Risk register

| Risk | Preventive design | Verify |
|---|---|---|
| Four maps create too much work | One parametric region controller; fixed data grids; MVP only one region | Region transition smoke and BFS routes |
| Faux-3D dungeon perspective mistakes | Integer facing vector and fixed four-depth slice renderer, no raycasting | Turn four times, wall tests, manual screenshots |
| Food starvation makes campaign unwinnable | Town restocks, visible HUD, reasonable spawn rates | Playtest paths with limited coins |
| Duplicate XP/coins/seals | Action IDs, `defeated` and awarded flags, idempotent quest claim | Double-confirm and repeated-interaction tests |
| Save incompatibility/corruption | `schema_version`, validation, temp rename, last-good backup | Corrupted fixture and roundtrip tests |
| Scene-switch crashes/circular state | GameState data only, router owns transitions, signals for UI | Rapid enter/exit and reload tests |
| Orbit FPS dependency/object leaks | Delta time, strict spawn/projectile caps, queue cleanup | 30/60/144 Hz, 3-minute sustained fire |
| Project exceeds 64K model context | One phase/task per request, TODO as source of truth, concise scene boundaries | New-context recovery drill |
| Asset/IP exposure | Original placeholders and asset provenance, no copied names/art/audio | Asset audit before release |
| Linux/Windows differences | Case-sensitive paths, built-in dependencies, separate exports | Launch exported builds on both |

**Balance guardrails:** First seal should be reasonably obtainable within 30–60 minutes by a new player; full run 3–5 hours is a tuning goal, not a mandated content expansion. If actual playtests expose excessive grinding, adjust prices/food/XP using data files before adding content. Do not raise difficulty by injecting unlimited spawns.

## 9. Local Qwen workflow and scope control

1. Give Qwen this plan as a local file in the project root. Keep its working set to the current phase, relevant scripts, and `TODO.md`.
2. Start at Phase 0. If partial project exists, inspect and continue first unchecked task only.
3. Favor one cohesive change per coding turn, ideally 1–4 files, followed by test and TODO update. Do not ask Qwen to generate a dozen scenes in one prompt.
4. Keep functions small and named, static type annotate where useful, avoid fragile dynamic string calls. Add comments for coordinate conventions, save schema, and action-ID ownership.
5. On failure, paste exact Godot output to Qwen. Require root-cause fix in affected files and regression run.
6. At phase completion, request READY FOR REVIEW. The user must approve before phase number changes.
7. If the context is reset, feed the plan, TODO, `git status --short` (if git), recent errors and the requested task. Do not trust an old conversational summary over files.

## 10. Release checklist

- [ ] Fresh character can complete all four region quests, orbital mission and final engine shutdown
- [ ] Save and Continue work in all allowed states, including after restart
- [ ] Death/retry/new game never corrupts saved progress
- [ ] No broken paths, parsing errors, warnings causing instability, duplicate rewards, softlocks or leaks
- [ ] Keyboard navigation and accessible high-contrast/reduced-flash mode work
- [ ] Music/SFX and visual assets are original/licensed and documented
- [ ] 60 FPS goal measured; performance/memory documented on actual machines
- [ ] Windows and Linux export templates and runtime tests completed or limitations stated
- [ ] `README.md` contains controls, install and troubleshooting
- [ ] `TODO.md` records accepted phases and release user approval

## 11. Master coding-agent handoff prompt

Copy and send the following to Qwen at project start and at each new session:

> You are the lead Godot 4 GDScript engineer implementing **Starfall Chronicles**. First read the **entire** `starfall-chronicles-godot-game-plan.md`, then read the **entire** `TODO.md` if present, then inspect current project files and source-control changes. The plan is the specification; TODO is the only live progress tracker. If TODO is missing, create it during Phase 0 and expand **all Phases 0–10** and all nine-item acceptance gates from the plan; do not create a second progress document.
>
> Identify the last explicitly ACCEPTED phase, the first incomplete current-phase task, and the minimum files needed. Implement **only that task**, or the current phase if I explicitly approve phase-wide work. Never implement future-phase mechanics. Preserve accepted features, existing files, and unrelated changes. Use Godot-native scenes/GDScript, short single-purpose scripts, no new dependencies. Verify version-sensitive APIs against installed Godot documentation. Do not invent assets or copy Ultima intellectual property.
>
> After editing, run actual headless import/parser checks and relevant automated tests; manually test when a GUI is available. If GUI is unavailable, mark gameplay testing BLOCKED, never PASS. Fix errors before claiming completion; verify regressions. Record exact commands, exit codes, observations, known bugs, decisions and next action in TODO. Mark completion only for verified tasks. If all phase gates pass, set READY FOR REVIEW, show concise report with files modified/test results/issues and **STOP for my explicit approval**. Never mark ACCEPTED yourself and never advance without permission. If blocked, clearly state why and ask one focused question. Start by reporting current phase/task and your immediate proposed action.

## 12. Recovery prompt

> STOP feature development. Read `starfall-chronicles-godot-game-plan.md`, `TODO.md`, current files and `git status --short` (if available). State current phase/status, last explicitly accepted phase, next unchecked task and observed regression or drift. Preserve working code. Reproduce the precise issue, fix **only the smallest related area**, rerun failed test and prior-phase regression tests, then update TODO with exact evidence. Do not revisit completed tasks unless a verified regression requires it. Do not advance phase. Report files/tests and wait for my approval.

## 13. Research references

1. [Ultima I: The First Age of Darkness — Wikipedia](https://en.wikipedia.org/wiki/Ultima_I:_The_First_Age_of_Darkness). Consulted for release history, setting, core control/view modes, character classes, food/economy, dungeon quests, space segment, and endgame structure (accessed 2026-10-23 UTC). This secondary overview describes more than one edition; original mechanics are labeled broadly rather than claimed as exact for every port.
2. [Godot Engine 4 official documentation](https://docs.godotengine.org/en/stable/). API verification resource for implementation; not a claim that each API has been validated in this specification.

**Research vs. invention reminder:** All Starfall names, coordinates, probabilities, formulas, maps, enemies, purchases, plot beats, and specific phase design are original prescriptions for this project, not reconstructed original game data.
