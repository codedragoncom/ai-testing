<!--
Type: Implementation prompt
Target: HTML Canvas, vanilla JavaScript
Output: ore-runner-2049.html
Related: None
-->

# ORE RUNNER 2049 — The Abandoned Mines
## Complete Retro Arcade Platformer

**Output filename: `ore-runner-2049.html`**

You are an expert JavaScript game developer specializing in HTML5 Canvas, classic arcade games, collision detection, platform physics, procedural pixel art, enemy AI, and reliable game-state management.

You develop compact, efficient, maintainable JavaScript with no external dependencies.

## YOUR MISSION

Create **ORE RUNNER 2049 — The Abandoned Mines**, a complete, polished, fully playable retro arcade platformer inspired by the classic 1982 game Miner 2049er.

The player is an adventurous miner exploring dangerous abandoned uranium mines.

The objective is to survey every required section of platform flooring while avoiding radioactive mutants, collecting useful equipment, navigating mine machinery, and surviving within a time limit.

The finished game must feel like an authentic early-1980s arcade platformer with modern responsiveness, polished pixel-art graphics, and carefully designed levels.

Deliver exactly one self-contained `ore-runner-2049.html` file.

**The highest priority is gameplay correctness.** Working traversal, collision physics, jumping, floor surveying, and level completion are more important than visual effects.

Create original game artwork, character designs, music, and level arrangements. Do not copy the original game's sprites, maps, or branding.

---

## 1. HARD REQUIREMENTS

- Exactly one complete HTML5 document.
- Embed all HTML, CSS, and JavaScript.
- Use HTML5 Canvas 2D for rendering.
- No frameworks, external libraries, CDNs, imports, or network access.
- No external images, spritesheets, fonts, or audio.
- No build process or backend.
- Must run directly using `file://` in Chrome or another modern browser.
- Use requestAnimationFrame and fixed-timestep gameplay updates.
- Support keyboard and touch controls.
- Include 10 complete handcrafted stages.
- Include functional enemies, elevators, slides, teleporters, and cannon mechanics.
- Provide proper title, pause, death, game-over, level-complete, and victory screens.
- No pseudocode, unfinished functions, TODO comments, or missing logic.

The application must be playable immediately after opening the HTML file.

## 2. GAME DESIGN PHILOSOPHY

The core gameplay is based on **platform surveying**.

Each level consists of multiple elevated platforms connected by ladders and special transportation systems.

The player must physically walk across every required section of platform flooring.

When the player walks over an unsurveyed floor section:

1. The floor changes color.
2. The section becomes permanently surveyed for the current attempt.
3. The remaining survey counter decreases.
4. The player earns points.

When all required sections have been surveyed, the stage is completed.

The player must finish before the timer reaches zero.

### Important Distinction

This is NOT a Lode Runner clone.

Do not implement digging as the main mechanic.

Do not replace floor surveying with gold collection.

The central challenge is navigating and covering platforms while avoiding hazards and planning safe routes.

## 3. GAME SCREEN

Use a fixed logical resolution with crisp integer-scaled graphics.

Recommended logical viewport:

- Width: 960 pixels.
- Height: 720 pixels.
- Playfield below a compact HUD.
- Fixed single-screen level layouts.

Maintain a consistent game coordinate system.

Scale the canvas responsively without stretching the scene.

Use nearest-neighbor rendering for pixel-art graphics.

### Visual Style

Create a detailed but authentic retro arcade look:

- Deep black mine background.
- Blue-gray stone platforms.
- Orange-brown structural supports.
- Radioactive green highlights.
- Bright industrial ladders.
- Colorful equipment collectibles.
- Glowing mutant enemies.
- Pixel-art miner character.
- Small animated machinery.

Use subtle modern effects without losing the classic arcade appearance.

Avoid emoji or plain text symbols as game sprites.

## 4. PLAYER CHARACTER

Create an original pixel-art miner character.

### Appearance

- Yellow or amber mining helmet.
- Small illuminated headlamp.
- Blue work clothing.
- Brown utility boots.
- Compact backpack.
- Animated arms and legs.

### Animation States

- Idle.
- Running.
- Jumping.
- Falling.
- Climbing.
- Riding an elevator.
- Sliding.
- Teleporting.
- Death.

Use procedurally drawn Canvas sprite frames or pre-rendered offscreen canvases.

Do not require image files.

## 5. PLAYER MOVEMENT

### Controls

| Action | Keyboard |
|---|---|
| Move Left | Left Arrow / A |
| Move Right | Right Arrow / D |
| Climb Up | Up Arrow / W |
| Climb Down | Down Arrow / S |
| Jump | Space / Z |
| Interact | E / Enter |
| Pause | P / Escape |
| Restart Stage | R |

### Horizontal Movement

- Responsive acceleration.
- Consistent maximum speed.
- Immediate or quick stopping.
- No excessive sliding.
- Solid platform collision.

### Jumping

Implement proper platform physics:

- Gravity.
- Vertical jump impulse.
- Airborne horizontal control.
- Collision with platforms.
- Safe landing detection.
- Fall-height tracking.

The player must be able to jump both vertically and horizontally.

Jump height and distance must remain consistent.

### Falling

A short fall is safe.

A fall exceeding approximately 1.5–2 character heights causes death, unless the player lands on a designated safe transport mechanism.

Track actual fall distance rather than simply checking downward velocity.

### Ladders

- Allow climbing and descending.
- Snap gently to ladder center.
- Suspend gravity while actively climbing.
- Allow correct transitions at ladder endpoints.
- Prevent climbing through unrelated floors.

### Collision

- Use consistent player bounding boxes.
- Implement robust horizontal and vertical collision.
- Prevent falling through platforms.
- Avoid tunneling during fast movement.
- Handle moving platform contact correctly.
- Never teleport the player except through explicit transport mechanics.

## 6. FLOOR SURVEY SYSTEM — CORE MECHANIC

Every required platform is divided into surveyable segments.

### Data Model

Each survey segment has:

- Unique ID.
- Rectangle or tile coordinates.
- Surveyed state.
- Color state.
- Optional associated platform ID.

### Rules

A section becomes surveyed only when the player walks on its upper surface.

Touching a platform from below or the side must not mark it surveyed.

Landing on a valid section may mark that section.

Standing in one location must not survey the entire platform.

### Visual Feedback

Uns​​urveyed platforms:

- Dark blue or gray.
- Subtle flashing mineral marks.

Surveyed platforms:

- Bright orange, green, or gold.
- Clear color change.
- Brief sparkle.

### Progress

Display:

`SURVEY: 72%`

Also track exact segment counts.

When every required section has been surveyed:

1. Stop the stage timer.
2. Award remaining-time bonus.
3. Play a completion effect.
4. Advance to the next station.

Do not require an exit door.

### Validation

Every required segment must be reachable with the implemented movement and transport mechanics.

No hidden or inaccessible segment may prevent completion.

## 7. RADIOACTIVE MUTANT ENEMIES

Implement multiple mutant creatures patrolling platforms.

### Appearance

- Small glowing creatures.
- Distinct silhouette.
- Animated body movement.
- Radioactive particle or shimmer effect.
- Clear dangerous and vulnerable colors.

### Normal Behavior

- Patrol designated platform routes.
- Reverse at walls or specified endpoints.
- Avoid walking off platform edges unless intended.
- Vary movement speed slightly.
- Follow level-specific movement patterns.

Do not require complex pursuit pathfinding.

The original style relies heavily on platform patrols and timing rather than universal enemy chasing.

### Collision

Touching an active mutant causes player death.

Prevent repeated life loss from a single collision.

### Vulnerable Mode

Collecting certain equipment activates a temporary power-up.

While active:

- Mutants change to bright green.
- Their dangerous animation changes.
- Player contact defeats them instead of killing the player.
- A short warning flash occurs before the effect expires.

Defeated mutants disappear for the remainder of the stage attempt.

The effect duration must be measured using game simulation time.

## 8. COLLECTIBLE EQUIPMENT

Place useful equipment throughout the levels.

Examples:

- Mining pickaxe.
- Helmet lamp.
- Drill.
- Lantern.
- Geiger counter.
- Toolbox.

### Rules

Each pickup:

- Has a fixed location.
- Uses a recognizable procedural sprite.
- Disappears on collection.
- Awards points.
- Activates temporary mutant vulnerability.
- Produces sound and visual feedback.

Optional: Different equipment may have different point values.

Do not allow equipment collection to create an unwinnable state.

## 9. LADDERS AND ELEVATORS

### Ladders

Support multiple ladder segments connecting platforms at different heights.

Ensure correct entry, exit, collision, and gravity behavior.

### Moving Elevators

Implement vertical or horizontal platforms.

Elevators must:

- Move between defined endpoints.
- Carry the player correctly.
- Reverse or pause at endpoints.
- Keep their position deterministic.
- Avoid crushing or trapping the player unfairly.

The player should remain attached while standing on a moving platform unless they walk or jump away.

## 10. SLIDES

Create inclined mine slides.

When the player enters a slide:

- Activate guided sliding motion.
- Accelerate along the slide.
- Follow the intended path.
- Safely exit onto the destination platform.

Use continuous interpolated motion.

Slides must not teleport the player or allow them to escape the playfield.

Allow input to be temporarily restricted during forced sliding.

## 11. TELEPORTERS

Implement paired transport stations.

### Behavior