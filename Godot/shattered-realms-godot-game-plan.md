<!--
Type: Phased game plan
Target: Godot 4.x, GDScript
Output: Complete Godot game project
Related: None
-->

# SHATTERED REALMS — Godot Game Plan

**Status:** Specification v1.0 · **Target:** Godot 4.x stable, GDScript, Windows/Linux · **Agent:** Qwen 3.8 Flash Next IQ3 (64K context)

> Build a complete original turn-based strategy game inspired by the *kind* of territory conquest play found in *Risk (1988)*. This specification, not the reference game's art or rules text, is the implementation authority. Build only one approved phase at a time.

## 1. Reference research and design boundary

**Documented:** *Risk* (1988) is a turn-based computer adaptation of the board game, credited to Gang of Five for the initial version, with a 1988 Commodore 64 release. The article describes solo play against up to five computer opponents and British/American variants, including an extra-armies card feature. It lists single-player and multiplayer and several later ports. It does **not** specify exact combat dice probabilities, map geometry, adjacency, card redemption schedule, or opponent algorithms. Those details must not be presented as verified 1988-specific behavior. [Wikipedia reference](https://en.wikipedia.org/wiki/Risk_(1988_video_game)).

**Inferred genre pillars:** take turns reinforcing controlled regions, attack adjacent rivals with dice-based uncertainty, transfer surviving troops, consolidate borders, capture territories, and win by territorial control. The following rules are **new design choices** for Shattered Realms, not historical assertions about that particular software release.

**Not applicable:** scrolling movement, player health, collisions, weapons, pickups, bosses, projectile physics, lives, or frame-based enemy spawning. Their strategy-game equivalents are region selection, troop counts, legal adjacency, attacks, territory cards, win/loss states, and AI turn decisions.

### Original identity

**Shattered Realms** is a desktop 2D conquest strategy game played across a fractured fantasy archipelago. Rival guilds contest 24 islands connected by land bridges and sea routes. Build armies, gamble on sieges, defend chokepoints, and claim the Crown Beacon before other factions do. Original UI and abstract procedural map visuals evoke a clean tabletop tactical game, without reproducing the Risk board, name, tokens, world map, audio, or brand.

**Player promise:** meaningful choices in 15–30 minute matches. The player can understand why each move was legal, why dice led to a result, and why an AI chose its target.

**Design constraints:** local hot-seat and one human versus 1–3 bots; no online multiplayer, monetization, diplomacy, hidden fog, procedural map generation, or live-service features in v1. All game logic deterministic under a chosen random seed; visuals must never determine the rules.

## 2. Complete gameplay specification

### 2.1 Board and factions

- Fixed map: **24 fictional territories**, grouped into **four provinces of six**: Ember Reach (E1–E6), Verdant Ring (V1–V6), Frost March (F1–F6), Azure Shoals (A1–A6).
- Four mutually distinct player colors plus high-contrast patterns/initials. Player names: Cinder Union, Moss Assembly, Pale Wardens, Tide Consortium.
- Represent graph in `territories.json`, with fields `id`, `name`, `province`, `center` (normalized 0–1 coordinates), `neighbors` (ID array), and polygon `points` (normalized 0–1 vertices). Define every undirected edge in both territories; validate bidirectionality and connectivity at startup. Never use clickable bounding boxes as final region hit-testing: point-in-polygon.
- Each province contains an internally connected six-node chain with extra optional internal links. Four provinces form a ring via at least two cross-province bridge edges each. Deliver exact complete 24-entry JSON during Phase 1; avoid runtime generation or random map topology.
- Province bonus: owning all six territories of a province yields **+2 armies** at the beginning of that player's next reinforcement turn. Bonuses stack.
- Match setup: 2–4 total factions, one human minimum, any remaining factions local human or simple AI. For MVP, fixed 1 human + 2 AI is acceptable; full selectable setup arrives Phase 8.

### 2.2 Start conditions

1. In a seeded, shuffled territory list, assign territories round-robin to active factions. Every territory begins with **3 troops**. With 2–4 players, this produces balanced territory counts because 24 is divisible by 2, 3, and 4.
2. First player chosen by the same seeded shuffle; turn order remains circular, skipping eliminated players.
3. A captured territory retains its new owner and the number of troops moved in by the attacker. A territory can never have fewer than one troop while owned.
4. No free deployment draft at game start; it is deliberately omitted for a faster first match.

### 2.3 Turn-state machine

`SETUP -> REINFORCE -> ATTACK -> FORTIFY -> END_TURN -> next player's REINFORCE`.

- **Reinforce:** income `max(3, floor(owned_territories / 3)) + 2 * completed_provinces + card_bonus`. The player must place all awarded troops on owned territories before attacking. A single territory has no troop cap.
- **Attack:** zero or more attacks from owned territory with at least two troops to **adjacent enemy** territory. Attacker chooses dice count 1–3, capped at `source_troops - 1`. Defender rolls 1–2 dice, capped at defender troops. Roll independent fair d6 values, sort each descending, compare top corresponding pairs; ties favor defender. For each pair, loser loses one troop. Resolve one combat round per Attack button press; no unattended auto-battle in MVP.
- **Capture:** if defender reaches zero, set defender owner to attacker and move troops from the source into target. Minimum transfer is **the number of attack dice committed in the winning round**; maximum is `source_troops - 1` measured after combat casualties. Prompt to choose within range, default to minimum. Clear selection after capture. A player may continue attacking.
- **Fortify:** once per turn, move any positive troop count from one owned territory to another owned territory **connected by a path entirely owned by that player**. Leave one troop behind. May skip. Fortification completes the phase.
- **End turn:** after fortify or skip, evaluate win/loss, grant a card if eligible, advance to next active player. Win when one faction owns all 24 territories. Eliminate faction immediately when ownership reaches zero, cancel its pending actions, and never grant it another turn.
- **Card reward:** if a player conquered one or more territories this turn, draw exactly one card at end turn, if deck permits. Hand limit 5. Phase 7 implements cards; prior phases must function without cards.
- **UI:** labels always show phase, selected source and destination, troop counts, legal action and why an action is disabled. All selections made by mouse; keyboard arrows/Tab navigate territories, Enter selects, Escape cancels, `P` pauses.

### 2.4 Battle example and nonnegotiable edge cases

If source has 5 and target has 2, attacker can commit 1–3 dice; defender up to 2. On a 3-vs-2 roll, compare two sorted pairs only. If both defenders lose, target becomes conquered. The attacker must still leave at least one troop at source and must transfer at least 3, assuming it committed 3 dice. **Do not subtract transfer from the defender; transfer from the post-battle source.**

Reject: attacks against owned/nonadjacent territories; attacking with one troop; invalid dice selections; placement on enemy territory; fortify across enemy territory; troop transfers below minimum; actions outside phase; AI actions after elimination; additional card draws on multiple captures; end turn while reinforcement troops remain; negative counts; no-owner territories after setup.

### 2.5 Cards (Phase 7)

- Original cards use three invented emblems: **Flame, Leaf, Wave**, plus optional **Beacon wild**. For v1, build a fixed 36-card deck: 12 of each emblem, no wilds (simpler to test).
- Redeem exactly three cards of a kind or one of each for **+5 troops** during Reinforce; redeemed cards enter a discard pile; shuffle discard into draw pile when deck is empty. Hand limit is five; if five held during Reinforce and a set exists, redemption is mandatory before turn can proceed. Otherwise no forced action.
- Redeemable hand management appears only in Reinforce. Auto-offer valid sets, human confirms; AI prefers mixed triple then same-kind triple. A card redemption counts only once per three removed cards. Bonus is additive to current unplaced troops.

### 2.6 AI specification

AI uses the **same public rules API** as the human, never modifies troop counts directly and never cheats by reading future dice. All candidates scored deterministically; ties broken using the match RNG.

- Reinforce: identify each owned border territory; score as `2 * adjacent_enemy_count + max(0, max_adjacent_enemy_troops - own_troops) + province_completion_priority`; allocate one troop at a time to highest-scoring border, recomputing each placement. Province priority +3 where the AI owns 4 or 5 of the province's 6 regions. With no borders, choose weakest owned region.
- Attack: only adjacent enemy targets; `strength_ratio = (source_troops - 1) / max(1, target_troops)`. Attempt when ratio >= 1.6, source has >= 4 troops and at least one viable target. Score `ratio * 2 + 3 if conquest completes a province + 1 if target is a chokepoint - target_troops * 0.15`. Perform max **12 combat rounds total per AI turn**, reevaluate after each, stop early if no qualifying attack. Use highest legal dice counts. A failed attack can be retried only if it still satisfies thresholds. No infinite loop.
- Fortify: find interior owned territories (no enemy neighbors) with >1 troop; choose strongest as source, reinforce weakest threatened reachable border; transfer `source_troops - 1`. Otherwise skip.
- AI presentation: display move and dice result with adjustable **0.25s–1.0s** pause; fast-forward bypasses delay. UI stays responsive. AI decision logic is synchronous pure state evaluation; action playback asynchronous in controller, never inside rule engine.
- Tests: AI action count bounded, all actions legal, elimination safe, fixed seed reproducible, no deadlocks over 100 simulated matches.

### 2.7 Presentation, scoring, failure, accessibility

- Main playfield is a scalable 2D map with colored polygons, outlined borders, army badges and simple adjacency highlighting. Fixed 1600×900 design resolution, `canvas_items` stretch, aspect `expand`, scalable UI anchors. Windowed 1280×720 minimum recommended; test resize.
- HUD: active faction, turn/phase, reinforcements, owned territories, selected region, action button row, card count, dice log, turn event feed, next-action hint.
- Menus: New Match, Continue (when save exists), Rules, Settings, Quit; pause overlay with Resume, Save, Return to Menu; victory/defeat overlay with standings and Rematch.
- Stats: turns survived, territories captured, battles won/lost, army losses, final placement. No arcade points or high-score system. Optional local hall of victories can be added only after core release gates; not required.
- Audio: original soft ambient loops, unobtrusive dice click, capture sting, phase change, UI click; independent music/SFX sliders and mute; no external assets required initially.
- Accessibility: do not encode ownership solely by color (use emblem/initial and patterns), strong territory outlines, keyboard navigation, readable font at 100% and 125%, reduced motion, adjustable AI pace, explicit focus rectangles and action explanations.
- No physics bodies or collision masks: selection uses polygon hit-testing on the UI canvas. Input events must be consumed to prevent click-through. This avoids inappropriate action-game architecture.

## 3. Architecture and contracts

Godot **4.x stable**, GDScript, 2D `Control` UI, single-threaded deterministic game state. Pin and record the exact stable version at Phase 0. Use built-ins only; no plugins, telemetry, networking, or third-party scripts. Recommended design: rules are plain `RefCounted` classes and dictionaries; UI consumes immutable snapshots or read-only state and sends commands to one controller. Avoid global Autoloads except `SettingsStore` if genuinely needed.

### Scene tree

```text
Main (Control; main.gd)
├── ScreenHost (Control)
│   ├── MainMenu (instanced menu.tscn)
│   └── MatchScreen (instanced match_screen.tscn)
│       ├── MapBoard (Control; map_board.gd)
│       ├── HUD (Control; match_hud.gd)
│       ├── ActionPanel (Control; action_panel.gd)
│       ├── BattlePanel (Control; battle_panel.gd)
│       └── OverlayHost (Control)
└── AudioStreamPlayers (later)
```

### Proposed repository (create files only when their phase begins)

```text
project.godot
shattered-realms-godot-game-plan.md
TODO.md                      # created by agent at Phase 0, not with this plan
scenes/main.tscn
scenes/ui/menu.tscn
scenes/ui/match_screen.tscn
scenes/ui/match_hud.tscn
scenes/ui/action_panel.tscn
scenes/ui/battle_panel.tscn
scripts/main.gd
scripts/core/board_data.gd
scripts/core/match_state.gd
scripts/core/rules_engine.gd
scripts/core/battle_resolver.gd
scripts/core/turn_controller.gd
scripts/core/card_deck.gd
scripts/ai/ai_policy.gd
scripts/ui/map_board.gd
scripts/ui/match_hud.gd
scripts/ui/action_panel.gd
scripts/ui/battle_panel.gd
scripts/ui/menu.gd
scripts/persistence/save_store.gd
scripts/persistence/settings_store.gd
data/territories.json
assets/fonts/              # licensed original or redistributable assets, later
assets/audio/              # original sounds, later
tests/test_runner.gd
tests/test_board.gd
tests/test_rules.gd
tests/test_battles.gd
tests/test_ai.gd
tests/test_save.gd
docs/                      # optional screenshots and release notes only
```

### Responsibilities and message flow

| Component | Sole responsibility | Key public methods/signals |
|---|---|---|
| `BoardData` | parse/validate static territory graph and polygons | `load_map(path)`, `validate()`, `is_adjacent(a,b)`, `owned_path(state,a,b,owner)` |
| `MatchState` | serializable factual state, ownership, counts, phase, turn, seed, cards, stats | `to_dict()`, `from_dict()`; no UI nodes |
| `RulesEngine` | validate/apply domain commands atomically; return result plus event list | `apply_command(state, command, rng)`; `get_legal_actions(state)` |
| `BattleResolver` | sorted dice, casualties, transfer minimum | `resolve(attacker_dice, defender_dice, rng)`; pure result |
| `TurnController` | own live state/RNG, route human/AI commands, emit snapshots | `submit(command)`, `state_changed(snapshot)`, `events_emitted(events)`, `match_ended(winner)` |
| `AIPolicy` | derive finite sequence of legal intent candidates | `choose_action(snapshot, legal_actions, rng)` |
| `MapBoard` | draw/select regions; never change ownership itself | signal `territory_selected(id)` |
| `MatchHUD/ActionPanel/BattlePanel` | show state, gather selection, emit command requests | signal `action_requested(command)` |
| `CardDeck` | manage draw, redeem, discard with serializable state | `draw()`, `redeem(indices)`, `to_dict()` |
| `SaveStore` | validate/version saved snapshot and write/reload | `save_match(state, rng_state)`, `load_match()` |

**Command schema:** dictionary with `type` and explicit IDs, e.g. `{type:"PLACE", territory_id:"E1", amount:2}`, `{type:"ATTACK", source:"E1", target:"E2", dice:3}`, `{type:"TRANSFER", amount:3}`, `{type:"END_ATTACK"}`, `{type:"FORTIFY", source:"E1", target:"V2", amount:2}`, `{type:"SKIP_FORTIFY"}`, `{type:"REDEEM", indices:[0,1,2]}`. All calls yield `{ok:bool, error_code:String, events:Array}`; rejects do not mutate state or RNG. Set the RNG state only after validated commands and persist it. Capture opens a pending transfer substate and blocks further attack commands until transfer completes. **Only `TurnController` owns mutable state.**

**Signals:** MapBoard `territory_selected`; ActionPanel `action_requested`; controller `state_changed`, `events_emitted`, `match_ended`. UI listens downward; no signals back from model or cross-references among panels. On new match, dispose old controller and disconnect old signals; no persistent timer-based callbacks touching a freed screen.

**State example:**

```json
{
  "schema_version": 1,
  "seed": 12345,
  "rng_state": 12345,
  "turn_number": 1,
  "active_player": 0,
  "phase": "REINFORCE",
  "players": [{"id": 0, "name": "Cinder Union", "kind": "HUMAN", "eliminated": false}],
  "territories": {"E1": {"owner": 0, "troops": 3}},
  "remaining_reinforcements": 3,
  "pending_transfer": null,
  "captured_this_turn": false,
  "fortified_this_turn": false,
  "deck": {"draw": [], "discard": [], "hands": {}},
  "stats": {}
}
```

This is a **shape example**, not a valid complete match: real saves must contain all 24 territories and all active players. `rng_state` must be the actual Godot `RandomNumberGenerator.state`, not necessarily equal to seed after turns. Centralize `enum`/string phase names and error codes. `END_ATTACK` moves to FORTIFY; `SKIP_FORTIFY` or completed `FORTIFY` advances turn. Do not allow generic `END_TURN` to bypass gates.

## 4. Phased development plan and completion gates

**Process:** Each phase must leave a launchable app, even if the latest feature has a temporary button/panel. Do not jump phases. Each task gets a checkbox in `TODO.md`. A phase becomes **READY FOR REVIEW** after automated and manual verification, and **ACCEPTED** only after explicit user approval.

### Standard gate — repeat for every phase

- [ ] Required features implemented; every task verified separately
- [ ] Godot imports without errors; no GDScript parse errors
- [ ] Launchable project with no critical runtime errors
- [ ] Phase-specific automated tests pass; manual gameplay performed where applicable
- [ ] No critical regression in earlier phases
- [ ] Failed tests fixed or blocker documented; actual commands and outcomes recorded
- [ ] Files changed and architecture decisions recorded in `TODO.md`
- [ ] Phase marked READY FOR REVIEW; user reviewed and explicitly approved before ACCEPTED / next phase

### Phase 0 — Skeleton and test harness

**Goal:** Empty Godot app launches; automated testing has a reliable entry point. **Depends:** none.

**Create/modify:** `project.godot`, `scenes/main.tscn`, `scripts/main.gd`, `tests/test_runner.gd`, `TODO.md`.

**Implementation:** Set renderer to Compatibility for widest desktop support, viewport 1600×900, stretch `canvas_items` / aspect `expand`, display title; add main `Control`, centered placeholder title and current phase. Register mouse/keyboard action map (select, cancel, pause, map directions). Write one headless test runner extending `SceneTree` that prints `PASS/FAIL` and exits nonzero on failures. Record `godot --version`, installed executable name, and manual launch instruction. Ignore `.godot/` in version control. Copy this plan into project root without edits to its requirements.

**Integration:** `Main` owns screen transitions; test runner owns no live UI. **Acceptance/test:** open editor and run F6/F5; resize window; run headless import and test runner; verify zero parse errors. **Common failures:** wrong main scene, test runner never quitting, incorrect stretch mode. **Done:** standard gate plus menu placeholder visible at resize sizes.

### Phase 1 — Validated static board

**Goal:** Complete original 24-territory board renders and selects. **Depends:** Phase 0 accepted.

**Create/modify:** `data/territories.json`, `scripts/core/board_data.gd`, `scripts/ui/map_board.gd`, `scenes/ui/match_screen.tscn`, `tests/test_board.gd`.

**Implementation:** Author exact 24 unique polygon definitions with no overlap and visually separated labels, 6 per province; no external world map data. Ensure map is one connected undirected graph and every territory has >=2 neighbors; avoid accidental edges created by mere visual contact. Use `Control._draw()` with `draw_colored_polygon`, `draw_polyline` and labels via child `Label` nodes; use normalized coordinates transformed into current board `Rect2` consistently for draw and hit-tests. Implement `Geometry2D.is_point_in_polygon` or a tested equivalent. Hover highlights candidate. Selection reports territory ID, owner placeholder, neighbors in HUD. Board may run with mock ownership until Phase 2.

**Integration:** `BoardData` is data-only; `MapBoard` reads graph and emits selected ID. **Acceptance/test:** 24 IDs unique, bidirectional neighbors, connected graph, no missing polygons, each province six members; 24 hit-point selection checks, outside click selects none; manual click/resize verification. **Common failures:** polygon overlap, center outside polygon, scaled click mismatch, broken adjacency. **Done:** standard gate plus all 24 territories selectable by mouse and keyboard.

### Phase 2 — State, setup and reinforcement

**Goal:** Deterministic seeded ownership and a complete reinforcement phase. **Depends:** Phase 1 accepted.

**Create/modify:** `scripts/core/match_state.gd`, `scripts/core/rules_engine.gd`, `scripts/core/turn_controller.gd`, `scripts/ui/match_hud.gd`, `scenes/ui/match_hud.tscn`, `tests/test_rules.gd`.

**Implementation:** Implement round-robin setup, invariant assertions, active player and phase state. Show distinct owners and troop counts. Calculate income by formula; command `PLACE` validates owner, phase, positive whole number, available pool. Allow placement of multiple troops per click/selection. Display remaining pool and disable Advance until zero. Implement temporary `END_ATTACK`/`SKIP_FORTIFY` hooks so a turn advances without battles; do not simulate captures yet. Seed RNG once per match. No state mutation in HUD.

**Integration:** UI submits commands through controller; controller emits snapshot; board redraws. **Acceptance/test:** same seed yields same setup; 2/3/4 factions have balanced assignments; initial 3 troops each; legal placement changes only selected territory; invalid placement causes zero state changes; exact income at 1, 8, 9 and 24 territories, plus provinces. **Common failures:** illegal pool underflow, integer division, phase bypass. **Done:** standard gate plus two complete reinforcement turns playable manually.

### Phase 3 — Dice combat and conquest

**Goal:** A legal, testable battle/capture loop. **Depends:** Phase 2 accepted.

**Create/modify:** `scripts/core/battle_resolver.gd`, `scripts/ui/battle_panel.gd`, `scenes/ui/battle_panel.tscn`, `tests/test_battles.gd`, modify `rules_engine.gd`, board/action HUD.

**Implementation:** Add owned source/enemy neighbor target selection, dice count, one-round Attack action; implement pairwise sorted d6 rule and ties to defender. Log dice and losses. On conquest set pending transfer and require valid movement before any other combat command; record capture flag and stats. Add source/target highlights and clear explanatory errors. Introduce phase transition after attack only when pending transfer is null. Defender never chooses dice manually in v1.

**Integration:** `BattleResolver` returns outcome only; `RulesEngine` applies casualties/ownership atomically; UI plays result. **Acceptance/test:** injected known dice verify both comparisons, tie behavior, max dice clamping, legal troop minima and transfer ranges, one-troop block, owner update. Exercise 20 manual rounds with no negative armies. **Common failures:** transferring pre-casualty numbers; double capture; combat RNG consumed on rejected commands. **Done:** standard gate plus full conquest interaction functional.

### Phase 4 — Fortify, elimination and victory (**MVP rule-complete**)

**Goal:** Play through a complete match locally using turns and attacks. **Depends:** Phase 3 accepted.

**Create/modify:** extend `rules_engine.gd`, `turn_controller.gd`, `tests/test_rules.gd`, HUD and simple result overlay in `match_screen.tscn`.

**Implementation:** Owned-only breadth-first search for fortify path, once-per-turn movement and skip; enforce leave-one rule. Skip eliminated players; detect last owner immediately after capture/transfer and show victory overlay. If active player eliminated through any state load, safely advance. Add minimal pause and rematch. For MVP set up 2 human hot-seat factions or invoke a temporary legal-action bot test driver in CLI; playable human-vs-AI arrives Phase 5.

**Integration:** Result overlay takes winner from controller signal; ownership changes redraw province badges. **Acceptance/test:** fortify across multi-hop owned path succeeds, enemy-interrupted path fails, second fortify fails, eliminated player skipped, win requires all 24; manual full human hot-seat match reaches victory and rematch resets. **Common failures:** BFS across enemy nodes, victory before pending transfer finalized, stale turn state after elimination. **Done:** standard gate plus full turn loop and human hot-seat victory.

### Phase 5 — Basic computer opponents (**single-player MVP**)

**Goal:** Complete and enjoy a human-versus-two-bot match. **Depends:** Phase 4 accepted.

**Create/modify:** `scripts/ai/ai_policy.gd`, `tests/test_ai.gd`; modify controller/UI.

**Implementation:** Implement exact reinforcement, attack scoring, max 12 rounds, and fortify policy from §2.6. Feed bot legal commands to controller through same validator. Pause briefly after moves with a cancel-safe timer in controller; fast-forward bypasses delay. On rejected bot command, log bug and stop AI turn safely, never busy-loop. Player can pause during AI sequence. Allow chosen match seeds for reproduction.

**Integration:** AI sees read-only snapshot and legal actions, controller owns scheduling. **Acceptance/test:** 100 headless matches complete or hit explicit 300-turn draw guard; no illegal commands or >12 combat rounds per bot turn; same seed yields same event history when AI delay excluded; manually complete one solo victory or defeat. **Common failures:** infinite loop, UI freeze, AI selecting eliminated source. **Done:** standard gate plus **human-vs-AI MVP accepted for playtesting**.

### Phase 6 — Clear interface and feedback

**Goal:** Remove guesswork from decisions. **Depends:** Phase 5 accepted.

**Create/modify:** `scenes/ui/action_panel.tscn`, `scripts/ui/action_panel.gd`, HUD and map UI.

**Implementation:** Add compact context-sensitive action buttons, legal neighbor overlays, colored province ownership badges, battle log last 20 events, turn summary, tooltips, full selection keyboard navigation, accessibility labels and strong focus outlines. Show explicit reason for invalid actions, transfer range and threat counts. No rules changes. Add in-game Rules panel summarizing this plan.

**Integration:** Present snapshots and legal-action list only. **Acceptance/test:** no invisible actions, keyboard-only placement/attack/fortify, readable at 1280×720 and 1600×900, all action panels update after AI steps. **Common failures:** focus stolen by redraw, stale disabled buttons, click propagation. **Done:** standard gate plus documented manual UX checklist.

### Phase 7 — Territory cards and optional strategic depth

**Goal:** Implement the specified simple set-collection reinforcement bonus. **Depends:** Phase 6 accepted.

**Create/modify:** `scripts/core/card_deck.gd`, new card UI within HUD/action panel, card tests in `tests/test_rules.gd`, state serialization fields.

**Implementation:** Create 36-card, 3-emblem seeded deck, draw once after any number of captures in turn, discard reshuffle, redeem sets for +5 during reinforcement, enforce hand cap and mandatory redemption at five where possible. Ensure no card can be reused after redemption. AI chooses legal set in Reinforce. Save-compatible state structure needed next phase.

**Integration:** Rules engine owns when card actions are legal, deck helper owns order; UI only lists hand. **Acceptance/test:** no draw if no capture; one draw after 1 or 5 captures; legal 3-of-kind and one-of-each; invalid set rejected; depleted draw reshuffles discard; forced redeem works; determinism remains. **Common failures:** duplicate draw, invalid indices after removal, save state missing deck. **Done:** standard gate plus hand/redeem playable in full match.

### Phase 8 — Menu, game setup, pause and persistence

**Goal:** Stable desktop experience with save/continue. **Depends:** Phase 7 accepted.

**Create/modify:** `scenes/ui/menu.tscn`, `scripts/ui/menu.gd`, `scripts/persistence/save_store.gd`, `scripts/persistence/settings_store.gd`, `tests/test_save.gd`; modify `main.gd`.

**Implementation:** New Match setup with 2–4 factions and each tagged HUMAN or AI (>=1 human); deterministic or random seed option. Add pause/settings/rules/rematch. Save to `user://match_save.json` via `FileAccess`, JSON with schema_version, all territory states, decks, stats, active phase, RNG state, pending capture transfer and state checksum/validation; write temporary file then replace via safe file operations when supported, keeping backup. Validate full map ID set, owner range, army minima, valid phase, transfer consistency, deck card conservation. Reject corrupt/unknown-version saves gracefully and offer New Match without overwriting corrupt file immediately. Store settings via `ConfigFile` in `user://settings.cfg`, not in save state.

**Integration:** `Main` manages screens, `SaveStore` handles disk, controller reconstructs from validated state. **Acceptance/test:** save in reinforce/attack/pending-transfer/fortify, close/reopen, compare deep state and next RNG rolls; malformed JSON and wrong version show friendly warning; settings persist separately; no duplicate signals after load. **Common failures:** RNG reset, cast float-vs-int from JSON, halfway transfer lost, partial write. **Done:** standard gate plus save/reload and continue tested.

### Phase 9 — Visual/audio polish and accessibility

**Goal:** Cohesive original, readable fantasy tabletop look. **Depends:** Phase 8 accepted.

**Create/modify:** assets under `assets/`; styling in scenes and UI scripts. No rule engine changes unless fixing tested defect.

**Implementation:** Distinct original region palette/textures, province boundary strokes, emblems, tasteful capture pulse and 0.3s dice reveal (reduced-motion toggle), new music and SFX, independent volume controls and mute. Procedural placeholders can ship if polished enough. Avoid strong particle effects or camera movements on a strategy board. Ensure theme contrast across faction colors, grayscale readability and layout in windowed/fullscreen. Provide captions/text event log for all audio cues.

**Integration:** visuals react to events, never mutate simulation. **Acceptance/test:** 60 FPS target on modest desktop at 1600×900, no stutter on bot turns, clean high-DPI scaling, reduced-motion tests, licensed asset inventory, manual audio mute and keyboard tests. **Common failures:** redraw cost, audio clipping, poor contrast, font clipping. **Done:** standard gate plus accessibility review.

### Phase 10 — QA, balance and desktop exports

**Goal:** Release-quality Windows/Linux builds. **Depends:** Phase 9 accepted.

**Create/modify:** test scripts, export presets (no secrets), README/release notes, test logs referenced in `TODO.md`.

**Implementation:** Run 100+ seeded bot-vs-bot games including edge-case seeds, verify no illegal state or endless runs; run human smoke matches and corrupt-save checks. Track average game length, win rates by turn order, AI stalled turns; tune only documented constants in policy, keeping rules stable. Test clean Godot import from fresh clone, Windows/Linux export templates and actual binaries, file save locations and restart. Release with full rules and credits. Release goal: zero known crash, save corruption, exploit or turn deadlock defects. Configure portable paths only and no OS-specific assumptions.

**Integration:** all prior modules frozen except regression fixes. **Acceptance/test:** no headless parse errors, test suite green, bot simulation invariants green, 60 FPS target, Windows and Linux exports launch, save/continue on both. **Common failures:** export resource paths, case sensitivity, missing import, platform font/audio variation. **Done:** standard gate plus signed-off release checklist.

## 5. Testing playbook

Use the installed Godot 4 executable (`godot` or `godot4`) and record the actual name/path in TODO. Typical commands:

```bash
godot --version
godot --headless --path . --import
godot --headless --path . --quit-after 10
godot --headless --path . --script res://tests/test_runner.gd
```

`--import` validates import work; it is **not** a manual gameplay test. The scripted test runner must return nonzero on any failed assertion and print count, seed and case ID. Run UI manually after every phase; for actions test legal, illegal, boundary, keyboard, resume and repeated input. Run the previous phase's tests before marking a new phase ready. Do not assume `--quit-after` confirms gameplay correctness; it only catches certain load/runtime errors.

**Unit cases:** symmetric connected graph; phase transitions; exact troop arithmetic; attacks/ties/sorted dice; pending conquest transfer; ownership and elimination; path-based fortify; province bonus; AI finite action budget; card conservation; save round-trip equality and deterministic next RNG outcome. **Integration cases:** new match -> reinforce -> attack -> capture -> transfer -> fortify -> next player; one human match and one AI match; save/load at every phase; rematch after victory. **Fault injection:** malformed save, invalid command, enemy node in path, eliminated faction, full hand, empty deck, repeated rapid clicks. Use 10 reproducible fixture seeds with expected board owners and a smoke set of 100 simulation seeds.

**Performance:** steady 60 FPS at 1600×900, AI action computation <100ms for typical board on a modern desktop, bounded event log (last 20 entries), no increasing node count across 10 rematches. Profile before optimizing. If any benchmark fails, log hardware, build, seed and actual numbers.

## 6. Implementation risks and mitigations

| Risk | Mitigation |
|---|---|
| Region hit-testing off after resize | Same normalized coordinate transform for render and input, center hit-tests |
| Invalid map routes | Bidirectional connected-graph validator in Phase 1 |
| Combat/transfer troop underflow | Atomic validated commands, injected dice and invariant assertions |
| UI mutates model or stale AI callback | Controller sole owner; event-driven UI; cancelable AI playback |
| Determinism lost after save | Serialize actual RNG state and deck order; test next action equality |
| AI stalls forever | Max 12 battle rounds/turn, 300-turn simulation guard and legal-action fallback |
| Card draw duplication | One `captured_this_turn` flag, reset only at beginning of turn |
| Smaller model rewrites stable code | One-phase task scope, test-before/after, preserve accepted phases |
| Rights issue from inspiration | Fictional map, names, symbols, rules prose and sounds; no lifted assets |
| Ambitious extras delaying finish | Fixed board, no online play, no diplomacy, no procedural campaign |

## 7. Intellectual-property and content policy

Use the reference to understand **general genre mechanics**, not to copy protected expression. Do not use the Risk name in the game's product title, original map shape or world geography, exact visual composition, branded cards, icons, box art, music, sound samples or source code. All in-game written rules must be newly authored. Territory graph and names must be independently designed. Track provenance/license for every imported font, sound and texture. Mention inspiration only in development documentation where appropriate; do not imply official affiliation.

## 8. `TODO.md` specification and initial template

**The coding agent creates `TODO.md` in Phase 0.** It is the **single progress tracker**. This plan is permanent specification. Never create `IMPLEMENTATION_STATUS.md`. Completed tasks and verification history survive context resets. The model may update task statuses only after evidence from actual tests. The four phase statuses are `IN PROGRESS`, `BLOCKED`, `READY FOR REVIEW`, `ACCEPTED`; only the user can approve ACCEPTED.

Copy this template into project root at kickoff, expanding each phase checklist with the concrete tasks above:

```markdown
# Shattered Realms — TODO

**Current phase:** 0 — Skeleton and test harness
**Phase status:** IN PROGRESS
**Last accepted phase:** None
**Build status:** NOT VERIFIED
**Next action:** Create project.godot and launchable main scene
**Godot executable/version:** UNVERIFIED

## Phase 0 — Skeleton and test harness
- [ ] Create/configure project and main scene
- [ ] Create test runner and test commands
- [ ] Configure inputs, resolution and resize
- [ ] Run headless import/script tests
- [ ] Manually launch and resize
- [ ] Record results; request user approval

## Phase 1 — Validated static board
- [ ] Create/validate 24-node graph and original polygon map
- [ ] Render and select all territories at multiple resolutions
- [ ] Add board tests; manual inspection; request approval

## Phase 2 — State, setup and reinforcement
- [ ] Seeded setup, state/controller, troop allocation, rule tests
- [ ] Manual two-turn playtest; request approval

## Phase 3 — Dice combat and conquest
- [ ] Battle resolver, selection, capture transfer, combat tests
- [ ] Manual combat playtest; request approval

## Phase 4 — Fortify, elimination and victory
- [ ] Fortify BFS, elimination, win/rematch, end-to-end tests
- [ ] Manual full hot-seat match; request approval

## Phase 5 — Basic computer opponents
- [ ] AI reinforcement, attacks, fortify, bounded loops, seed tests
- [ ] Manual solo match and 100 simulations; request approval

## Phase 6 — Clear interface and feedback
- [ ] Context actions, event log, focus, labels, rule help
- [ ] Manual keyboard/readability checks; request approval

## Phase 7 — Territory cards
- [ ] Deck, redemption, draw rule, AI use, tests
- [ ] Manual card round-trip; request approval

## Phase 8 — Menus, setup, pause and persistence
- [ ] Setup screens, save/load, settings, corruption handling
- [ ] Manual resume and round-trip tests; request approval

## Phase 9 — Art, audio and accessibility
- [ ] Original art/SFX, audio options, reduced motion, contrast
- [ ] Manual performance/accessibility review; request approval

## Phase 10 — QA and exports
- [ ] Full regression, bot simulations, balance, Windows/Linux builds
- [ ] Release gate and user approval

## Current phase completion gate
- [ ] All required features implemented
- [ ] All tasks individually verified
- [ ] Godot imports without errors
- [ ] No GDScript parsing errors
- [ ] Gameplay tested manually where applicable
- [ ] No critical regressions
- [ ] Known issues documented
- [ ] TODO.md updated with actual evidence
- [ ] Phase reviewed and explicitly approved by user

## Testing results
| Date/time | Phase | Command or manual scenario | Result | Evidence / error |
|---|---|---|---|---|
| TBD | 0 | Not run | NOT RUN | — |

## Known issues and blockers
- None recorded. Do not confuse untested with passed.

## Technical decisions
- Godot 4.x GDScript, fixed 24-node fictional graph, state-authoritative rules engine.

## Session history
- Initial: plan received; implementation not started.

## Approvals
| Phase | Status | Approval record/date |
|---|---|---|
| 0–10 | NOT STARTED | — |
```

At each session start: read entire plan, TODO, project tree, current git status and most recent test results; do not trust TODO checkboxes without inspecting affected files when evidence is missing. Select **one next incomplete task**. At session end: update current phase/status, actual commands and outcomes, issues, decisions, next action and session log. If a regression appears, reopen the relevant task, mark `BLOCKED` if it prevents safe progress, and state why. Never overwrite prior approval history.

## 9. Agent operational protocol

1. Work on a git branch (if git is available); check existing changes before edits and do not overwrite user changes.
2. Read the plan and TODO fully. Inspect current files before choosing work.
3. Plan modifications to the **smallest number of files** for the current task; favor scripts under ~300 lines where practical. Keep rule engine independent of view.
4. Implement a small vertical slice and run its focused tests. Fix parse/runtime errors immediately; then run regression tests.
5. Perform actual manual gameplay checks when possible. If no graphical desktop is available, record manual test as **NOT RUN / BLOCKED**, never claim it passed; do not accept phases requiring manual verification.
6. Update TODO with test command, exit code, screenshot location only if created, and concise behavioral outcome. Summarize modified files.
7. Mark phase READY FOR REVIEW when gates pass; stop and request approval. If user explicitly approves, mark ACCEPTED, then wait for another instruction before starting next phase.

**Context discipline for 64K local model:** start each session with this plan and TODO, but inspect only source files relevant to the selected task plus interfaces they call. Keep a concise current-task summary; do not dump all source files to context. One command or one phase at a time, never simultaneous large rewrites. Preserve working contracts. Search official Godot documentation before relying on version-sensitive engine APIs. Git commits at phase boundaries are recommended, but do not silently discard changes.

## 10. Final release criteria

- Fresh checkout imports; all tests green; no critical known issues.
- One full 24-region match works to victory/defeat with 2–4 factions and 1–3 bots, plus hot-seat.
- Placement, dice losses, transfer minimum, fortify path, cards and province bonuses match §2 exactly.
- Saved match resumes with identical state and future seeded randomness.
- Game usable with mouse and keyboard and distinguishable without faction colors.
- Original assets and attributions documented; no copied franchise assets.
- Stable 60 FPS target and bounded simulation/event history.
- Both Windows and Linux exports launch and preserve saves/settings.
- Every phase accepted explicitly; TODO records evidence.

## 11. Master coding agent handoff prompt

Copy everything below into Qwen at the project root:

```text
You are the implementation engineer for shattered-realms-godot-game-plan.md.
You are running locally with limited context; work carefully and incrementally.

MANDATORY START:
1. Read shattered-realms-godot-game-plan.md in full.
2. Read TODO.md if present. If absent and the project is new, create it from the plan template.
3. Inspect the existing Godot project, relevant source, git status, and recent test evidence.
4. Determine the current phase, last ACCEPTED phase, next unchecked task and dependencies.
5. Report exactly what one task or approved phase you intend to implement.

WORK RULES:
- Follow the specification literally. Build with Godot 4.x GDScript and built-ins.
- Implement only the next incomplete task in the current approved phase.
- Do not begin the following phase without explicit user approval.
- Do not rewrite stable completed systems or unrelated files.
- Keep rules/state separate from rendering, use validated commands and deterministic RNG.
- Do not invent historical requirements or introduce out-of-scope features.
- Verify version-sensitive Godot calls in official documentation.
- Run focused tests, headless import, script load and existing regression tests.
- Test interactively when possible; if unavailable, say NOT RUN instead of PASSED.
- Fix errors before claiming completion; if blocked, explain blocker with exact evidence.
- Update TODO.md tasks, gates, results, decisions, issues, current status and next action.
- Only mark phases ACCEPTED after explicit approval from the user.

REPORT:
Phase and task; files created/modified; functionality observed; exact test
commands and outcomes; manual verification status; regressions/issues;
TODO status; proposed next action. Stop for review at the phase gate.

Begin with the next unfinished task. Do not implement the whole game now.
```

### Recovery prompt

```text
STOP. Reorient to shattered-realms-godot-game-plan.md and TODO.md.
Inspect current files, git changes, last accepted phase, and most recent tests.
Find the smallest incomplete task or regression within the CURRENT phase.
Undo none of the user's work. Do not add features from later phases.
Repair only what's required, run headless import plus focused and regression
tests, and record actual results in TODO.md. If a phase is ready, ask for
explicit approval and stop. Report exact edits, risks and next task.
```

## 12. References

- [Risk (1988 video game) — Wikipedia](https://en.wikipedia.org/wiki/Risk_(1988_video_game)) — documented release, genre, AI opponent count and variants. Historical claims limited to article contents.
- [Godot 4.x official docs: custom GUI controls](https://docs.godotengine.org/en/stable/tutorials/ui/custom_gui_controls.html) — input handling for `Control`.
- [Godot official docs: saving games](https://docs.godotengine.org/en/stable/tutorials/io/saving_games.html) — `FileAccess`, JSON and save patterns.
- [Godot official docs: command-line tutorial](https://docs.godotengine.org/en/stable/tutorials/editor/command_line_tutorial.html) — headless import/test/export syntax.

**Research honesty:** other than the reference article's brief summary, the detailed mechanics and all AI algorithms in this plan are an original implementable design specification, not a claim about undocumented 1988 software behavior.
