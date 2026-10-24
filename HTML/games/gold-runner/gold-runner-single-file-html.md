<!--
Type: Implementation prompt
Target: HTML Canvas, vanilla JavaScript
Output: One standalone HTML file
Related: None
-->

# GOLD RUNNER — Classic Arcade Platform Puzzle Game
## Complete Single-File HTML5 Game

You are an expert JavaScript game developer specializing in HTML5 Canvas, tile-based platformers, arcade game mechanics, enemy artificial intelligence, procedural level design, and responsive game controls.

You write efficient, maintainable, reliable JavaScript without external dependencies.

## YOUR MISSION

Create **GOLD RUNNER**, a complete, polished, fully playable 2D arcade puzzle-platformer inspired by the gameplay of classic Lode Runner.

The player must navigate underground levels, climb ladders, traverse ropes, collect gold, dig temporary holes, avoid pursuing enemies, and escape to the next level.

The finished game should feel like a professionally produced retro arcade game with responsive controls, clever enemy behavior, engaging levels, and satisfying feedback.

Deliver exactly one self-contained `index.html` file.

**The highest priority is gameplay correctness.** A simple-looking game with reliable mechanics is better than a beautiful game with broken movement, unfair enemies, or impossible levels.

---

## 1. NON-NEGOTIABLE REQUIREMENTS

- Exactly one complete HTML5 file.
- All HTML, CSS, and JavaScript must be inline.
- Use the HTML5 Canvas 2D API.
- No external libraries, CDNs, frameworks, imports, images, fonts, or audio files.
- No build process, server, or network access.
- Must work directly when opened using `file://`.
- No external asset loading.
- No placeholder functions, missing features, TODO comments, or pseudocode.
- Use `requestAnimationFrame` with fixed or bounded game updates.
- Support keyboard controls and mobile-friendly touch controls.
- Include a start screen, pause, game over, victory, and level transitions.
- Include actual playable levels rather than a single demonstration scene.
- Ensure all required mechanics work together correctly.

Do not simply create a character that moves around a grid.

Implement a complete game.

## 2. GAME DESIGN

Create a fast, readable, challenging arcade puzzle game.

### Core Gameplay Loop

1. Player enters a level.
2. Player navigates platforms, ladders, and ropes.
3. Enemies actively pursue the player.
4. Player collects all required gold.
5. Player can dig temporary holes to trap enemies.
6. After all gold is collected, an exit becomes accessible.
7. Player reaches the exit to complete the level.
8. Next level introduces new challenges.

The game should reward planning, timing, movement skill, and clever use of the environment.

### Game Style

Use a retro arcade aesthetic inspired by classic 1980s puzzle-platformers.

Visual direction:

- Dark underground backgrounds.
- Warm brick platforms.
- Bright gold collectibles.
- Blue or cyan ladders.
- Thin overhead ropes.
- Clearly distinguishable player and enemies.
- Crisp pixel-art-inspired graphics.
- Smooth but grid-aligned movement.
- Restrained modern lighting and animation effects.

Avoid using emoji or text characters as the primary game sprites.

Draw graphics procedurally using Canvas.

## 3. TILE-BASED WORLD

Use a deterministic tile-based level system.

Recommended tile size: 24–32 logical pixels.

Use a consistent grid across every level.

### Required Tile Types

| Tile | Behavior |
|---|---|
| Empty | Passable space affected by gravity |
| Solid Brick | Blocks movement and supports characters |
| Diggable Brick | Can be temporarily removed |
| Indestructible Wall | Cannot be dug or passed |
| Ladder | Allows vertical climbing |
| Rope | Allows horizontal hanging traversal |
| Gold | Collectible objective |
| Exit Ladder | Activated after collecting all required gold |
| Player Spawn | Starting position |
| Enemy Spawn | Enemy starting position |

Represent levels using readable character maps or structured arrays.

Separate immutable level definitions from runtime tile state.

### Collision Requirements

- No movement through solid tiles.
- Correct support when standing on platforms.
- No falling through floors.
- Smooth transitions onto and off ladders.
- Proper movement along ropes.
- Reliable falling from unsupported locations.
- Correct collision after temporary holes refill.

Do not use inconsistent collision logic for the player and enemies.

## 4. PLAYER MOVEMENT

Implement responsive platform-puzzle movement.

### Controls

| Action | Keyboard |
|---|---|
| Move Left | Left Arrow / A |
| Move Right | Right Arrow / D |
| Climb Up | Up Arrow / W |
| Climb Down | Down Arrow / S |
| Dig Left | Z / J |
| Dig Right | X / K |
| Pause | P / Escape |
| Restart Level | R |

### Movement Rules

- Player moves horizontally at a consistent speed.
- Gravity applies when unsupported.
- Player can climb ladders vertically.
- Player can traverse horizontal ropes.
- Player cannot jump.
- Player cannot pass through solid tiles.
- Player may descend through appropriate ladder openings.
- Player aligns naturally to ladder and tile coordinates.
- Player stops correctly when directional input is released.

Movement should feel responsive rather than slippery.

### Grid Alignment

Use sub-tile position values for smooth animation while keeping collision logic tied to the grid.

Implement controlled tile-center snapping where appropriate.

Avoid accumulating coordinate drift that makes ladders difficult to use.

### Input Handling

Support held keys and simultaneous directional inputs.

Prevent browser scrolling from arrow keys and Space when the game has focus.

Clear held input when the window loses focus.

## 5. DIGGING MECHANIC

**Digging is one of the most important gameplay systems.**

The player can dig a temporary hole into a diggable brick diagonally down-left or down-right.

### Rules

- Player must be in a valid position to dig.
- Digging targets the appropriate diagonal tile.
- Only diggable bricks can be removed.
- Indestructible walls cannot be dug.
- Digging has a brief cooldown.
- The tile becomes temporarily empty.
- The hole remains open for a limited duration.
- The brick eventually regenerates.

### Hole Timing

Suggested default:

- Dig cooldown: 0.3 seconds.
- Hole lifetime: 4–6 seconds.
- Visible warning before the hole closes.

Use elapsed simulation time rather than arbitrary frame counts.

### Enemy Trapping

Enemies that fall into a hole should become trapped temporarily.

A trapped enemy:

- Stops pursuing the player.
- Performs a visible trapped animation.
- Remains immobilized for a limited time.
- Eventually attempts to climb out.
- Can be defeated if the brick regenerates while the enemy remains trapped.

If a player is inside a hole when it closes, apply a consistent death or crushing rule.

Never regenerate a solid tile without correctly resolving the character occupying it.

### Visual Feedback

Add:

- Small brick fragments when digging.
- Brief debris animation.
- Dark open hole.
- Flashing or cracking indicator before regeneration.
- Clear trap feedback.

Visual effects must not affect collision behavior.

## 6. ENEMY ARTIFICIAL INTELLIGENCE

Create intelligent guards that actively pursue the player.

Enemies must navigate the same environment as the player.

### Required Behaviors

- Walk left and right.
- Climb ladders.
- Descend ladders.
- Traverse ropes.
- Fall from platforms.
- Avoid solid walls.
- Pursue the player.
- Become trapped in dug holes.
- Escape traps when allowed.
- Respawn when defeated.

### Pathfinding

Use a grid-based pathfinding system such as BFS or A*.

However, do not treat the map as a simple four-direction maze.

Navigation must respect actual game movement rules:

- Horizontal movement requires valid collision behavior.
- Ladder movement requires a ladder connection.
- Rope movement follows rope traversal rules.
- Falling is permitted where unsupported.
- Solid walls block travel.
- Temporary holes modify available routes.
- Trapped enemies cannot follow normal paths.

Build a movement-aware navigation graph or equivalent reachable-neighbor function.

### Intelligent Pursuit

Enemies should:

- Seek a reachable route toward the player.
- Recalculate periodically rather than every rendering frame.
- React when a brick is dug or regenerated.
- Handle temporary route obstructions.
- Choose alternatives when a direct route is unavailable.
- Continue moving when no direct path exists rather than freezing permanently.

### Enemy Difficulty

Allow variation through:

- Movement speed.
- Repath frequency.
- Reaction delay.
- Number of enemies.

Avoid unfair instant direction changes.

### Collision With Player

Contact with an active enemy costs one life.

Provide a brief death animation and then restart the current level.

Do not repeatedly subtract lives during the same collision.

Add a short invulnerability period after respawn if necessary.

## 7. GOLD COLLECTION

Gold must be collected through actual player contact.

### Rules

- Gold disappears immediately when collected.
- Score increases.
- Remaining gold counter updates.
- Collection produces visual and audio feedback.
- Each gold piece can only be collected once.
- The level exit unlocks only when all required gold is collected.

### Optional Enemy Gold Carrying

If practical, allow enemies to temporarily carry gold and drop it after being trapped or defeated.

This must not create unreachable gold or prevent level completion.

Omit this optional behavior rather than implementing it incorrectly.

## 8. EXIT AND LEVEL COMPLETION

When all gold is collected:

- Activate the exit.
- Reveal or illuminate the exit ladder.
- Display a clear "EXIT OPEN" notification.
- Allow the player to navigate to the exit.
- Complete the level only when the player physically reaches the exit.

Do not automatically complete the level when the final gold is collected.

### Transitions

On completion:

- Freeze gameplay safely.
- Award a level completion bonus.
- Show a short transition.
- Load the next level.
- Preserve score and remaining lives.

## 9. LEVEL DESIGN

Include at least **8 complete handcrafted levels**.

Do not rely solely on random tile generation.

### Progression

**Level 1 — First Steps**
- Teaches movement and ladder climbing.
- Few enemies.
- Simple gold placement.

**Level 2 — The Dig**
- Introduces digging and temporary holes.
- Safe opportunities to trap enemies.

**Level 3 — High Wire**
- Introduces horizontal rope traversal.
- More vertical complexity.

**Level 4 — The Pursuit**
- Stronger enemy pressure.
- Multiple routes.

**Level 5 — Deep Shafts**
- Vertical ladder networks.
- Falling hazards and alternate paths.

**Level 6 — Split Paths**
- Navigation puzzle with several possible routes.
- Digging required.

**Level 7 — Gold Rush**
- More enemies and collectibles.
- Increased timing pressure.

**Level 8 — The Final Vault**
- Multi-layer maze.
- Challenging but fair enemy placement.
- Combines all previous mechanics.

### Level Validity

Every level must be completable using implemented movement rules.

Ensure:

- Every required gold item is reachable.
- Exit can be reached after collection.
- Player does not spawn inside a wall.
- Enemies spawn in valid positions.
- No essential route requires nonexistent mechanics.
- Difficulty increases gradually.

Prefer clearly valid layouts over large complicated maps that may be impossible.

### Replay

After completing the final level:

- Show a victory screen.
- Display final score.
- Allow restarting from Level 1.

## 10. GAME STATE MANAGEMENT

Use an explicit game state machine.

Recommended states:

- TITLE
- PLAYING
- PAUSED
- PLAYER_DYING
- LEVEL_COMPLETE
- GAME_OVER
- VICTORY

Handle transitions explicitly.

Do not let input update gameplay while paused, dying, or between levels.

### Lives

Start with 3–5 lives.

After death:

- Deduct exactly one life.
- Play a death animation.
- Restart the current level.
- Preserve score according to the selected game rules.

### Score

Award points for:

- Collecting gold.
- Trapping or defeating enemies.
- Completing levels.

Display score continuously.

### High Score

Use localStorage when available.

Handle storage being unavailable without breaking the game.

No network leaderboard is required.

## 11. VISUAL DESIGN

Create a polished modern-retro arcade presentation.

### Game Screen

- Dark cave-like background.
- Brick textures drawn procedurally.
- Clear tile boundaries.
- Bright colored ladders.
- Metallic ropes.
- Glowing gold pieces.
- Animated player sprite.
- Clearly visible enemy sprites.

### Procedural Sprite Rendering

Use Canvas drawing operations to construct compact pixel-art characters.

Player:

- Distinctive outfit.
- Visible head, torso, arms, and legs.
- Different facing directions.
- Walking animation.
- Climbing animation.
- Falling pose.
- Digging animation.

Enemies:

- Contrasting colors.
- Recognizable guard silhouette.
- Walking and climbing animations.
- Trapped pose.

Animations must correspond to actual character state.

### Effects

Use lightweight visual feedback:

- Gold sparkle.
- Digging debris.
- Small dust particles.
- Exit illumination.
- Enemy trap animation.
- Death flash.
- Level completion transition.

Avoid excessive particle effects.

## 12. AUDIO

Generate simple retro arcade sounds using the Web Audio API.

No external audio files.

Include sounds for:

- Gold collection.
- Digging.
- Enemy trapping.
- Player death.
- Exit activation.
- Level completion.
- Button selection.

Use short synthesized tones.

Default sound to a sensible state and provide a mute toggle.

Initialize or resume audio only after a user gesture.

Never allow audio failure to break gameplay.

## 13. GAME INTERFACE

### Main Menu

Display:

**GOLD RUNNER**

Subtitle: *Dig. Dodge. Escape.*

Include:

- Start Game.
- How to Play.
- Sound toggle.

### In-Game HUD

Display:

- Score.
- High score.
- Lives.
- Level number.
- Remaining gold.
- Pause indicator when paused.

### Pause Menu

Include:

- Resume.
- Restart Level.
- Restart Game.
- Sound toggle.

### Game Over

Display:

- Final score.
- Level reached.
- Retry button.

### Victory

Display:

- Final score.
- Levels completed.
- High score status.
- Play Again button.

## 14. MOBILE AND TOUCH CONTROLS

Provide usable touch controls on phones and tablets.

### Touch Layout

- Directional pad on lower-left.
- Dig-left and dig-right buttons on lower-right.
- Pause button within reach.

### Requirements

- Large tap targets.
- Pointer Events support.
- Support holding movement buttons.
- Support simultaneous movement and digging.
- Prevent page scrolling during gameplay.
- Avoid sticky buttons after pointer cancellation.
- Adapt to landscape orientation.

Keyboard controls must remain available on desktop.

### Responsive Canvas

- Preserve the logical game aspect ratio.
- Scale the canvas to available space.
- Maintain crisp rendering.
- Avoid distorted tiles.
- Keep HUD readable.
- Do not crop essential gameplay areas.

Use nearest-neighbor scaling for pixel-art presentation.

## 15. GAME LOOP

Use `requestAnimationFrame` for rendering.

Use a fixed timestep for gameplay updates, preferably 1/60 second.

### Update Order

1. Process input.
2. Update player movement.
3. Apply gravity and collision resolution.
4. Update digging and temporary holes.
5. Update enemy AI and movement.
6. Resolve enemy/player interactions.
7. Process gold collection.
8. Check exit conditions.
9. Update game state.
10. Update visual effects.
11. Render the frame.

Separate update logic from rendering.

Use a bounded accumulator to prevent large simulation jumps when the browser tab resumes.

### Performance

- Cache static tile rendering where practical.
- Avoid unnecessary allocations in the main loop.
- Reuse particle arrays.
- Keep pathfinding frequency bounded.
- Rebuild navigation information only when necessary.
- Avoid expensive whole-map calculations every frame.

Target smooth 60 FPS on ordinary desktop hardware.

## 16. BUG PREVENTION

The following failures are unacceptable:

- Walking through walls.
- Falling through solid floors.
- Climbing without a ladder.
- Inability to enter ladders due to coordinate drift.
- Getting permanently stuck on ladder edges.
- Digging the wrong tile.
- Holes failing to regenerate.
- Holes regenerating without handling occupants.
- Enemies teleporting through walls.
- Enemies freezing indefinitely due to bad pathfinding.
- Losing multiple lives from a single collision.
- Gold becoming permanently unreachable.
- Exit activating before all gold is collected.
- Pause failing to stop gameplay.
- Controls remaining pressed after window focus loss.
- Starting a new level with stale previous-level entities.

Use explicit collision and state logic to prevent these failures.

## 17. IMPLEMENTATION PRIORITIES

When complexity creates tradeoffs, use this priority order:

**Priority 1 — Complete Playable Game**

The title screen, player, levels, gold collection, exit, lives, and game states all work.

**Priority 2 — Correct Movement**

Walking, gravity, ladders, ropes, and collision handling are reliable.

**Priority 3 — Digging**

Temporary holes, regeneration, and trapping work consistently.

**Priority 4 — Enemy AI**

Enemies navigate actual reachable paths and pursue the player.

**Priority 5 — Level Quality**

Handcrafted levels are fair, varied, and completable.

**Priority 6 — Visual Polish**

Animated characters, attractive tiles, effects, and transitions.

**Priority 7 — Audio and Mobile Controls**

Responsive touch interface and synthesized sound.

Never sacrifice gameplay correctness for decorative effects.

## 18. INTERNAL VALIDATION

Before outputting the code, inspect it for correctness.

### Execution

- All functions are defined.
- All game states are handled.
- No missing variables.
- No invalid event handler references.
- Canvas initialization succeeds.
- No network resources are required.

### Movement

- Player collision uses consistent coordinates.
- Ladders allow proper vertical movement.
- Ropes support horizontal traversal.
- Gravity behaves consistently.
- Controls stop when released.

### Digging

- Only valid blocks can be dug.
- Hole timers update.
- Holes regenerate.
- Trapped enemies behave correctly.
- Closing holes resolve occupants correctly.

### Enemies

- AI uses reachable movement transitions.
- Enemies cannot pass through solid walls.
- Pathfinding reacts to changed tiles.
- Trapped enemies eventually recover or are defeated.
- Player collision causes one death event.

### Levels

- Maps are rectangular or safely padded.
- Player spawn exists.
- Exit exists.
- Gold count matches actual collectibles.
- No required gold is inaccessible.
- Level completion can occur.

### UI

- Start works.
- Pause works.
- Restart works.
- Level transition works.
- Game over and victory screens work.
- Touch buttons release properly.
- Sound can be muted.

Resolve identifiable problems before finalizing.

Do not claim to have executed or tested the game unless actual browser testing tools were available.

## 19. OUTPUT REQUIREMENTS

Return ONLY the complete executable HTML document.

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
- Missing levels.
- Omitted game mechanics.
- Instructions to install dependencies.

All code must be complete and included in the document.

Do not truncate the output.

## FINAL SUCCESS STANDARD

The user saves the response as `index.html`, opens it in Chrome or another modern browser, and immediately sees a polished retro arcade game's title screen.

After starting, the player can run through underground platforms, climb ladders, cross ropes, collect gold, dig holes, trap pursuing guards, and escape through an unlocked exit.

Eight increasingly challenging levels provide a complete gameplay experience.

The controls feel responsive, enemies behave intelligently, game states work correctly, and the game can be played from beginning to end.

The final result should feel like a complete classic arcade game, not a movement demo or an unfinished prototype.