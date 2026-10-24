<!--
Type: Implementation prompt
Target: HTML Canvas, vanilla JavaScript
Output: jungle-assault.html
Related: jungle-assault-phased-game-plan.md; jungle-assault-build-instructions.md
-->

# JUNGLE ASSAULT — Operation Ironstorm
## Complete Single-File HTML5 Arcade Run-and-Gun Game

**Output filename: `jungle-assault.html`**

You are an expert JavaScript game developer specializing in HTML5 Canvas, classic arcade shooters, real-time combat systems, enemy AI, collision detection, scrolling environments, and procedural pixel-art graphics.

You write efficient, reliable, maintainable JavaScript with no external dependencies.

## YOUR MISSION

Create **JUNGLE ASSAULT — Operation Ironstorm**, a complete, polished, fully playable top-down military arcade shooter inspired by Capcom's classic 1985 Commando.

The player controls an elite soldier advancing north through hostile jungle territory while fighting enemy troops, avoiding incoming gunfire, destroying fortified positions, rescuing prisoners, and surviving heavily defended enemy bases.

Capture the fast, intense, arcade-style gameplay of classic vertically scrolling run-and-gun games.

The result should feel like a professionally developed retro arcade shooter, not a simple character movement demo.

Deliver exactly one standalone `jungle-assault.html` file.

**The highest priorities are responsive movement, accurate eight-direction shooting, intelligent enemy spawning, reliable collision detection, and smooth vertical scrolling.**

Use original characters, graphics, maps, audio, and branding. Do not copy sprites or level layouts from the original game.

---

## 1. NON-NEGOTIABLE REQUIREMENTS

- Exactly one complete HTML5 document.
- Embed all HTML, CSS, and JavaScript.
- Use Canvas 2D for rendering.
- No Three.js, frameworks, imports, CDNs, or external libraries.
- No external images, spritesheets, fonts, or audio files.
- No build process or backend.
- No network requests.
- Must run directly using `file://`.
- Use requestAnimationFrame with fixed-timestep gameplay updates.
- Include keyboard, mouse, and touch controls.
- Include eight complete vertically scrolling stages.
- Include functional enemies, bullets, grenades, obstacles, pickups, and stage-end battles.
- Include start, pause, death, game-over, and victory screens.
- No TODOs, pseudocode, placeholder mechanics, missing stages, or omitted code.

The game must be playable immediately after opening the file.

## 2. GAMEPLAY DESIGN

Create a classic vertically scrolling military arcade experience.

### Core Gameplay Loop

1. The player starts near the bottom of a jungle battlefield.
2. The player advances north.
3. The camera scrolls upward as the player progresses.
4. Enemy soldiers enter from multiple directions.
5. The player fires an unlimited-ammunition rifle.
6. Limited grenades damage groups and fortified positions.
7. The player avoids enemy bullets and environmental hazards.
8. Optional prisoners can be rescued for bonus points.
9. Each stage ends at a fortified enemy position.
10. Defeating the final enemy wave unlocks progression to the next stage.

### Arcade Philosophy

The game should reward:

- Constant movement.
- Positioning.
- Accurate shooting.
- Grenade timing.
- Recognizing enemy attack patterns.
- Fast reactions.
- Strategic use of cover.

Avoid long periods without enemy activity.

Do not create an overwhelming number of enemies that makes survival impossible.

The difficulty must be challenging but fair.

## 3. GAME SCREEN AND COORDINATE SYSTEM

Use a logical viewport approximately:

- Width: 640 pixels.
- Height: 800 pixels.

Use a portrait-oriented battlefield suited to vertical scrolling.

Scale the canvas responsively while maintaining aspect ratio.

### Coordinate System

- World X increases to the right.
- World Y increases downward.
- North is negative Y.
- Player world coordinates remain independent of camera position.
- Screen Y = world Y - camera Y.

Use one consistent coordinate system for:

- Player.
- Enemies.
- Bullets.
- Grenades.
- Obstacles.
- Pickups.
- Stage objectives.

Do not mix screen-space and world-space positions during collision detection.

### Camera

The camera follows northward player progression.

It should:

- Scroll smoothly upward.
- Keep the player in the lower portion of the screen.
- Allow limited movement toward the bottom of the visible playfield.
- Never scroll backward excessively.
- Clamp movement at stage boundaries.
- Stop scrolling during stage-end encounters.

The environment must remain aligned with all entities as the camera moves.

## 4. PLAYER CHARACTER

Create an original animated pixel-art commando.

### Appearance

- Olive-green combat uniform.
- Dark military helmet.
- Small backpack.
- Visible rifle.
- Distinct directional silhouette.
- Running leg animation.
- Muzzle flash while firing.

Draw all graphics procedurally using Canvas.

### Player State

Maintain:

- Position.
- Velocity.
- Facing direction.
- Current movement state.
- Health or lives.
- Grenade count.
- Invulnerability timer.
- Animation timer.
- Weapon cooldown.

### Movement

Support eight-direction movement.

Normalize diagonal input so diagonal speed does not exceed straight-line movement speed.

Use fast, responsive acceleration with minimal unwanted sliding.

### Facing

The character must visually face the firing direction.

When not shooting, use the most recent movement direction.

Support eight directional poses:

- North.
- Northeast.
- East.
- Southeast.
- South.
- Southwest.
- West.
- Northwest.

## 5. PLAYER CONTROLS

### Keyboard

| Action | Control |
|---|---|
| Move | WASD / Arrow Keys |
| Fire | Space / J |
| Throw Grenade | K / G |
| Pause | P / Escape |
| Restart Stage | R |

### Firing Direction

Use the classic arcade-style option:

- Movement determines facing direction.
- Holding Fire shoots along the current facing direction.
- Shooting while moving diagonally fires diagonally.

Also support optional mouse aiming on desktop.

If mouse aiming is enabled, aim toward the cursor in world space.

### Input Rules

- Support simultaneous movement and firing.
- Support movement and grenade throwing together.
- Do not interrupt movement when firing.
- Prevent browser scrolling during gameplay.
- Clear stuck keys when the window loses focus.

### Touch Controls

Implement two virtual thumbsticks:

- Left stick: movement.
- Right stick: aiming and automatic shooting.

Provide a separate grenade button.

Support simultaneous multi-touch input through Pointer Events.

The touch interface must work without keyboard input.

## 6. RIFLE COMBAT — HIGHEST PRIORITY

The player's main weapon is a rapid-fire rifle.

### Rules

- Unlimited ammunition.
- Consistent firing cooldown.
- Fast straight-moving projectiles.
- Eight-direction aiming.
- Small muzzle flash.
- Brief firing sound.
- Bullet impact effects.

### Recommended Defaults

- Fire rate: approximately 7–10 rounds per second.
- Bullet speed: much faster than enemy movement.
- Projectile lifetime: bounded.
- Player bullets and enemy bullets have distinct appearances.

### Projectile Data

Each bullet maintains:

- Position.
- Velocity.
- Radius.
- Damage.
- Owner/faction.
- Active state.
- Lifetime.

### Collision

Bullets must:

- Hit enemies.
- Hit solid obstacles where appropriate.
- Damage destructible structures.
- Despawn outside the active simulation area.
- Never damage the firing entity.
- Never continue damaging multiple unrelated targets after impact unless explicitly designed as piercing projectiles.

Use swept segment collision or bounded movement substeps to prevent tunneling.

Never use visual sprite overlap as the sole collision mechanism.

## 7. GRENADE SYSTEM

The player starts with a limited supply of grenades.

Recommended starting amount: 5.

### Mechanics

- Grenades follow a visible throwing arc or travel trajectory.
- They land at a valid target position.
- Detonate after a short fuse.
- Create circular area-of-effect damage.
- Damage enemies and designated destructible structures.
- Produce a visible explosion.
- Do not damage through indestructible cover unless intentionally allowed.

For mouse aiming, throw toward a clamped target point.

For classic keyboard control, throw forward in the facing direction.

### Grenade Animation

- Small spinning projectile.
- Shadow indicating ground position.
- Brief fuse indicator.
- Bright explosion flash.
- Smoke and debris.

### Collision Logic

Apply explosion damage once per affected entity.

Use a bounded explosion radius.

Avoid repeated damage during explosion animation.

### Pickups

Add occasional grenade supply crates.

Cap maximum grenade inventory.

## 8. ENEMY TYPES

Implement several distinct enemy behaviors.

### Type 1 — Patrol Soldier

- Walks a predefined patrol route.
- Detects nearby player movement.
- Fires occasionally.
- Low health.

### Type 2 — Rifleman

- Maintains distance.
- Fires aimed projectiles.
- Uses cover when possible.
- Moderate health.

### Type 3 — Charging Soldier

- Runs toward the player.
- Attempts to close distance.
- Rarely shoots.
- Dangerous at short range.

### Type 4 — Trench Soldier

- Occupies a fixed defensive position.
- Fires from partial cover.
- May be difficult to hit from certain angles.
- Vulnerable to grenades.

### Type 5 — Machine Gunner

- Remains near a defensive structure.
- Fires short bullet bursts.
- Has a visible firing cooldown.
- Higher health.

### Type 6 — Elite Guard

- Faster movement.
- More accurate shooting.
- Appears in later stages.
- Moderate-to-high health.

### Enemy Behavior Requirements

- All enemies have explicit states.
- They activate near the camera.
- They do not shoot through solid obstacles.
- They avoid invalid terrain.
- They do not occupy exactly the same position indefinitely.
- They use bounded movement and firing rates.
- They are removed or recycled after death.

## 9. ENEMY AI

Use a lightweight state machine.

Recommended states:

- IDLE
- PATROL
- ALERT
- CHASE
- ATTACK
- RETREAT
- DEAD

### Detection

Enemies detect the player based on:

- Distance.
- Forward visibility.
- Line of sight.
- Stage-specific activation triggers.

### Attacking

Enemies should:

- Face the player before shooting.
- Use a reaction delay.
- Fire according to enemy type.
- Lead the player slightly only at higher difficulty.
- Avoid firing continuously every frame.

### Cover

Implement simple cover-aware behavior.

Enemies may:

- Move behind sandbags.
- Fire from trenches.
- Step out briefly.
- Return to cover.

Do not attempt an expensive general-purpose tactical AI system.

### Performance

Use periodic AI updates rather than recalculating every decision at 60 Hz.

Movement and projectiles should still update smoothly.

## 10. ENEMY SPAWNING AND DIFFICULTY

Use a deterministic, data-driven enemy spawn system.

### Spawn Events

Each stage defines enemy groups with:

- Spawn world position.
- Enemy type.
- Activation distance.
- Group size.
- Spawn interval.
- Optional patrol or destination data.

### Spawning Rules

- Spawn enemies when the camera approaches their region.
- Do not spawn enemies directly on top of the player.
- Do not instantly surround the player without warning.
- Limit simultaneous active enemies.
- Avoid unlimited waves unless part of a defined encounter.

### Difficulty

Increase challenge through:

- Enemy combinations.
- More complex terrain.
- Faster but bounded enemy reaction.
- Stronger fortified positions.
- More frequent crossfire.
- Limited supplies.

Do not increase difficulty by making collisions unfair or bullets unavoidable.

## 11. PROCEDURAL JUNGLE ENVIRONMENT

Create rich procedural 2D battlefield graphics.

### Terrain Types

- Jungle grass.
- Dirt roads.
- Mud.
- Shallow streams.
- Wooden bridges.
- Rocky clearings.
- Military encampments.
- Fortified bases.

### Decorations

- Trees.
- Bushes.
- Rocks.
- Tree stumps.
- Sandbags.
- Wooden crates.
- Supply barrels.
- Small trenches.
- Military fences.

### Terrain Rules

Distinguish between:

- Decorative objects.
- Solid obstacles.
- Destructible cover.
- Water that slows movement.
- Hazardous areas.

Collision geometry must match visible objects.

A tree should not block the player when it visually appears passable, and vice versa.

### Rendering

Use deterministic procedural drawing and reusable offscreen canvases where appropriate.

Avoid redrawing expensive static decoration geometry from scratch every frame.

## 12. DESTRUCTIBLE OBJECTS

Add battlefield objects that can be destroyed.

Examples:

- Wooden crates.
- Supply barrels.
- Sandbag positions.
- Guard towers.
- Light barricades.

### Behavior

- Each destructible object has health.
- Bullets and grenades may damage it.
- Damage produces visual feedback.
- Destruction removes or changes collision geometry.
- Some objects release supplies.

Indestructible objects must remain solid.

Do not allow bullets to pass through solid cover accidentally.

## 13. PRISONER RESCUE SYSTEM

Add optional prisoner rescue events.

### Behavior

Prisoners appear in designated locations.

They may be:

- Held near enemy camps.
- Escorted by guards.
- Locked behind destructible barriers.

### Rescue Rules

- Defeat guards or open the designated route.
- Reach the prisoner.
- Award bonus points.
- Trigger a short rescue animation.
- Mark the prisoner as rescued.

Prisoners must never count as enemies.

Player bullets must not accidentally harm prisoners unless explicitly implementing a separate mechanic.

Use original prisoner character designs.

## 14. STAGE-END FORTRESS BATTLES

Each stage must end with a distinct fortified encounter.

### Trigger

When the player reaches the stage-end position:

1. Stop forward camera scrolling.
2. Activate the fortress encounter.
3. Spawn defined enemy waves.
4. Display a warning or visual transition.
5. Require the player to defeat all mandatory defenders.
6. Unlock the stage completion state.

### Fortress Elements

- Guard towers.
- Sandbag positions.
- Defensive walls.
- Machine-gun nests.
- Central command structure.

### Wave Limits

Waves must be finite.

A stage must not become impossible because enemies continuously respawn.

### Final Stage

Stage 8 should feature a larger fortress encounter with several defensive positions and an elite commander enemy.

Keep boss patterns readable and avoid excessive bullet density.

## 15. EIGHT HANDCRAFTED STAGES

Create eight complete vertically scrolling stages.

Each stage has:

- Unique terrain layout.
- Enemy spawn definitions.
- Environmental obstacles.
- Pickup locations.
- Rescue opportunities.
- A fortified final encounter.

### Stage 1 — Jungle Landing

- Helicopter arrival introduction.
- Basic patrol enemies.
- Open jungle corridors.
- Teaches movement and shooting.

### Stage 2 — River Crossing

- Shallow water.
- Wooden bridges.
- Enemies positioned across riverbanks.
- First major machine-gun nest.

### Stage 3 — Enemy Encampment

- Tents and barricades.
- Patrol groups.
- Supply crates.
- Prisoner rescue mission.

### Stage 4 — The Canyon

- Narrow rocky passages.
- Ambush points.
- Destructible barriers.
- Grenades become important.

### Stage 5 — Swamp Territory

- Mud that slows movement.
- Restricted pathways.
- Dense enemy patrols.
- Hidden supplies.

### Stage 6 — Mountain Outpost

- Rocky defensive positions.
- Elevated-looking gun emplacements.
- Heavy riflemen.
- Fortified checkpoint.

### Stage 7 — The War Zone

- Dense enemy formations.
- Crossfire.
- Multiple machine-gun nests.
- Several grenade supply opportunities.

### Stage 8 — Ironstorm Fortress

- Large fortified approach.
- Elite enemy troops.
- Multiple defensive waves.
- Final commander encounter.
- Complete-game victory sequence.

### Level Design Quality

Do not generate eight interchangeable straight corridors.

Create distinct terrain arrangements and battle scenarios.

Ensure each level is completable.

## 16. WORLD SCROLLING AND STAGE DATA

Represent each stage with:

- World dimensions.
- Terrain seed.
- Background layers.
- Obstacle definitions.
- Enemy spawn events.
- Pickups.
- Rescue events.
- Fortress encounter data.

Recommended world height: approximately 3–5 screen heights per stage.

Use object coordinates in world space.

### Camera Progression

The camera should:

- Follow the player's northern advance.
- Remain within stage bounds.
- Keep upcoming threats visible.
- Avoid sudden jumps.
- Stop at fortress encounters.

Enemies and bullets should not shift incorrectly when the camera scrolls.

## 17. COMBAT COLLISION SYSTEM

This is a critical benchmark requirement.

Use consistent hitboxes for:

- Player.
- Enemies.
- Bullets.
- Grenades.
- Explosions.
- Obstacles.
- Pickups.

### Requirements

- Bullets do not pass through intended solid objects.
- Collision response does not depend on rendering FPS.
- Grenades damage targets once per explosion.
- Player and enemy projectiles use separate teams.
- Dead enemies stop firing.
- Offscreen objects are cleaned up safely.
- Player cannot move through solid barriers.
- Collision checks use world coordinates.

Use circle-circle and circle-rectangle tests where appropriate.

For fast bullets, use swept collision tests or sufficiently small fixed substeps.

## 18. PLAYER HEALTH, LIVES, AND SCORING

### Lives

Start with 3 lives.

A hit from an enemy bullet or direct hostile contact costs a life, following classic arcade rules.

After a hit:

- Trigger a death animation.
- Deduct exactly one life.
- Respawn safely.
- Apply a brief invulnerability period.

Do not deduct multiple lives from one overlapping collision.

### Scoring

Award points for:

- Defeating enemies.
- Destroying defensive positions.
- Rescuing prisoners.
- Collecting supplies.
- Completing stages.

### Bonus Lives

Award an extra life at defined score thresholds.

Cap the maximum lives.

### High Score

Store the high score using localStorage when available.

Handle storage failure gracefully.

## 19. VISUAL STYLE

Use a modernized 1980s arcade aesthetic.

### Player

- Animated eight-direction soldier sprite.
- Running legs.
- Rifle muzzle flash.
- Small shadow under character.

### Enemies

- Distinct uniform colors.
- Different headgear or silhouettes by type.
- Readable shooting animation.
- Small death effects.

### Jungle

- Rich greens.
- Dark shadowed foliage.
- Brown earth.
- Blue-green rivers.
- Gray stone.
- Warm military structures.

### Combat Effects

- Bullet tracers.
- Muzzle flashes.
- Dust impacts.
- Grenade explosions.
- Smoke puffs.
- Debris.
- Small screen shake.

Use lightweight procedural Canvas effects.

Avoid excessive effects that obscure enemy bullets.

## 20. AUDIO

Use the Web Audio API.

Generate synthesized sounds for:

- Rifle firing.
- Grenade throw.
- Explosion.
- Enemy hit.
- Player death.
- Supply pickup.
- Prisoner rescue.
- Fortress warning.
- Level complete.
- Victory.

Create optional retro arcade background music using synthesized notes.

Provide a mute control.

Audio must initialize only after a user gesture.

Avoid creating a new AudioContext for every sound.

## 21. USER INTERFACE

### Main Menu

Display:

**JUNGLE ASSAULT**

Subtitle:

*Operation Ironstorm*

Buttons:

- Start Mission.
- How to Play.
- Sound On/Off.

### Gameplay HUD

Display:

- Score.
- High score.
- Lives.
- Grenades.
- Current stage.
- Rescue count.
- Optional stage progress bar.

### Pause Menu

- Resume.
- Restart Stage.
- Restart Game.
- Sound toggle.

### Game Over

- Final score.
- Stage reached.
- Enemies defeated.
- Prisoners rescued.
- Retry.

### Victory

- Mission complete.
- Final score.
- Total rescue count.
- High score status.
- Play Again.

## 22. MOBILE SUPPORT

Provide usable touch controls.

### Movement

A virtual directional joystick on the left side.

### Aiming and Shooting

A virtual aiming joystick on the right side.

When moved beyond a small dead zone, automatically fire.

### Grenades

A large separate grenade button.

### Additional Controls

- Pause.
- Mute.
- Restart when appropriate.

### Requirements

- Support multi-touch.
- Use Pointer Events and pointer capture.
- Release input after pointer cancel.
- Prevent accidental browser scrolling.
- Avoid sticky movement or firing states.
- Keep controls usable in landscape mode.

The game must remain playable on a phone without a keyboard.

## 23. GAME LOOP AND PERFORMANCE

Use a fixed simulation timestep of 1/60 second.

Render with requestAnimationFrame.

### Recommended Update Order

1. Read input.
2. Update player movement and facing.
3. Resolve player/environment collisions.
4. Update camera position.
5. Activate enemy spawns.
6. Update enemy AI.
7. Update bullets and grenades.
8. Resolve projectile collisions.
9. Apply explosion damage.
10. Process enemy/player collisions.
11. Process pickups and rescues.
12. Evaluate fortress encounter.
13. Update game state.
14. Update visual effects.
15. Render.

### Performance Requirements

- Reuse bullets through object pooling.
- Reuse particle objects.
- Limit active enemies.
- Limit active enemy projectiles.
- Cache static terrain rendering.
- Avoid expensive allocations every frame.
- Use bounded delta time.
- Limit catch-up updates after browser-tab suspension.
- Cull visual objects outside the camera viewport.
- Keep gameplay deterministic where practical.

Target approximately 60 FPS on normal desktop hardware.

## 24. GAME STATE MANAGEMENT

Use an explicit state machine.

Required states:

- TITLE
- HOW_TO_PLAY
- PLAYING
- PAUSED
- PLAYER_DYING
- RESPAWNING
- FORTRESS_BATTLE
- STAGE_COMPLETE
- GAME_OVER
- VICTORY

Transitions must be explicit.

Do not allow duplicate enemy spawns after restarting.

Do not permit gameplay updates during pause or death transitions.

## 25. BUG PREVENTION

The following failures are unacceptable:

- Player movement exceeding intended speed diagonally.
- Player shooting in the wrong direction.
- Bullets passing through enemies.
- Bullets passing through solid obstacles incorrectly.
- Grenades damaging targets multiple times.
- Enemy bullets appearing directly inside the player without warning.
- Camera scrolling in the wrong direction.
- Enemies shifting incorrectly during scrolling.
- World-space and screen-space collision mismatches.
- Enemy spawns repeating unintentionally.
- Enemies accumulating outside the stage.
- Fortress waves never ending.
- Player losing multiple lives from one collision.
- Respawning inside a hazard.
- Grenade count becoming negative.
- Player being unable to move during normal gameplay.
- Pause allowing enemy bullets to keep moving.
- Touch controls remaining pressed.
- Level transitions preserving stale projectiles.

Use explicit game-state and collision logic to prevent these issues.

## 26. IMPLEMENTATION PRIORITIES

Use this strict order when complexity creates tradeoffs.

**Priority 1 — Working Application**

The Canvas initializes and the player can move and shoot.

**Priority 2 — Scrolling and Combat**

Camera progression, bullets, collisions, and eight-direction firing work.

**Priority 3 — Enemy AI**

Patrols, chasers, riflemen, and defensive positions function.

**Priority 4 — Grenades and Obstacles**

Area damage, cover, and destructible objects work.

**Priority 5 — Stage Progression**

Eight levels, enemy waves, fortress battles, and victory condition work.

**Priority 6 — Rescues and Pickups**

Prisoners, grenade supplies, and scoring function.

**Priority 7 — Visual Polish**

Procedural pixel art, animations, particles, and transitions.

**Priority 8 — Audio and Mobile**

Synthesized sounds and multi-touch controls.

Never sacrifice gameplay correctness for decorative effects.

## 27. INTERNAL VALIDATION

Before producing the final answer, inspect your code.

### Initialization

- Canvas exists.
- All JavaScript variables are initialized.
- All functions are defined.
- No external assets are required.
- Game state begins correctly.

### Player

- Eight-direction movement works.
- Diagonal speed is normalized.
- All shooting directions are correct.
- Grenades work.
- Collision with cover works.

### Enemies

- All enemy types can spawn.
- Enemies move and attack.
- Bullets use correct collision ownership.
- Enemy death removes active attacks.
- Enemy counts remain bounded.

### Scrolling

- World coordinates remain consistent.
- Camera follows northward movement.
- Offscreen entities are handled safely.
- Fortress camera stop works.

### Gameplay

- Grenades deal one damage event per target.
- Prisoners can be rescued.
- Pickups activate correctly.
- Lives decrease exactly once per death.
- Scoring is correct.
- Fortress encounters terminate.

### Levels

- All eight stages exist.
- Every stage has a defined start and end.
- Each stage has enemy and obstacle data.
- All stage-end encounters can be completed.
- Victory occurs after Stage 8.

### Interface

- Start works.
- Pause works.
- Restart works.
- Keyboard controls work.
- Mouse aiming works.
- Touch controls work.
- Audio mute works.

Fix identifiable errors before finalizing.

Do not claim to have performed browser tests unless execution tools were actually available.

## 28. OUTPUT FORMAT

Return ONLY the complete working HTML document.

Begin with:

`<!DOCTYPE html>`

End with:

`</html>`

Do not include:

- Markdown fences.
- Explanations.
- Pseudocode.
- TODO comments.
- Placeholder functions.
- Missing enemy behaviors.
- Missing stages.
- Unimplemented fortress encounters.
- Installation instructions.

All HTML, CSS, JavaScript, rendering, physics, AI, level data, controls, audio, effects, and initialization must be fully included.

Do not truncate the response.

## FINAL SUCCESS STANDARD

The user saves the response as `jungle-assault.html` and opens it in Chrome or another modern browser.

A polished retro arcade title screen appears.

After starting, an elite soldier is dropped into hostile jungle territory.

The player moves in eight directions, fires continuously at attacking enemy forces, throws grenades, destroys defensive positions, and rescues prisoners.

The camera scrolls north as the player advances through jungle paths, rivers, swamps, military encampments, and fortified outposts.

Enemy soldiers attack from different directions using recognizable tactics.

Every stage culminates in a challenging but fair fortress battle.

Eight distinct levels provide a complete arcade experience.

After defeating the final fortress commander, the player receives a mission-complete victory screen.

The result should feel like a polished modern interpretation of a classic 1980s vertical run-and-gun arcade shooter, not a generic top-down shooting demonstration.