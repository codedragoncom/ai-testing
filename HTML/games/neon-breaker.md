<!--
Type: Implementation prompt
Target: HTML Canvas, vanilla JavaScript
Output: neon-breaker.html
Related: None
-->

# NEON BREAKER — Arcade Recharged
## Complete Single-File HTML5 Brick-Breaking Game

**Output filename: `neon-breaker.html`**

You are an expert JavaScript game developer specializing in HTML5 Canvas, arcade physics, collision detection, procedural visual effects, responsive controls, game-state management, and performance optimization.

You write complete, reliable, maintainable JavaScript with no external dependencies.

## YOUR MISSION

Create **NEON BREAKER — Arcade Recharged**, a polished, fully playable brick-breaking arcade game inspired by Atari's classic 1976 Breakout.

The player controls a paddle at the bottom of the screen, bouncing one or more balls toward rows of destructible bricks.

The objective is to destroy every required brick without allowing all balls to fall below the paddle.

Create the satisfying simplicity of a classic arcade game with modern enhancements:

- Responsive paddle movement.
- Accurate and predictable ball physics.
- Colorful destructible bricks.
- Meaningful paddle-angle control.
- Progressive difficulty.
- Multiple brick types.
- Collectible power-ups.
- Multiball gameplay.
- Combo scoring.
- Particle effects.
- Synthesized arcade audio.
- Ten complete levels.

**Gameplay correctness is the highest priority.** Ball physics, collisions, paddle control, and level completion must work before visual polish is added.

Create original artwork and level designs rather than copying copyrighted sprites or existing game assets.

## 1. HARD REQUIREMENTS

- Exactly one complete HTML5 document.
- All HTML, CSS, and JavaScript must be embedded.
- Use the HTML5 Canvas 2D API.
- No external libraries, frameworks, CDNs, imports, fonts, images, or audio.
- No build tools, backend, or network requests.
- Must run when opened directly using `file://`.
- Use `requestAnimationFrame` for rendering.
- Use fixed or bounded gameplay timesteps.
- Support keyboard, mouse, and touch controls.
- Include fully working menus and game states.
- Include 10 complete handcrafted levels.
- All power-ups and brick types must function.
- No TODO comments, placeholder functions, pseudocode, or omitted implementation sections.

Return a complete executable HTML file, not an implementation plan.

## 2. CORE GAMEPLAY

The game follows the classic brick-breaking loop:

1. The player launches a ball from the paddle.
2. The ball moves through the playfield.
3. It reflects off walls, bricks, and the paddle.
4. Brick collisions deal damage or destroy bricks.
5. Destroyed bricks award points.
6. Some bricks release falling power-ups.
7. The player catches power-ups with the paddle.
8. All required bricks must be destroyed.
9. The next level begins after completion.
10. Losing the final active ball costs one life.

After completing all 10 levels, display a victory screen.

### Fundamental Rules

- The paddle remains near the bottom of the screen.
- The ball follows consistent collision physics.
- Paddle contact changes the ball's horizontal direction based on impact position.
- Bricks disappear or take damage when hit.
- Ball velocity increases gradually as difficulty progresses.
- The ball must never pass through a solid brick without registering a collision.

Do not replace physics with scripted ball paths.

## 3. GAME CANVAS

Use a logical game area of approximately:

- Width: 800.
- Height: 1000.

The canvas should scale to available desktop and mobile space while maintaining its aspect ratio.

### Layout

Top:

- Score.
- High score.
- Level.
- Lives.
- Active power-up indicators.

Middle:

- Brick formation.
- Ball playfield.
- Animated effects.

Bottom:

- Player paddle.
- Ball launch position.
- Optional touch control indicators.

Use consistent world coordinates regardless of display resolution.

### Visual Style

Create an original neon-retro arcade aesthetic.

- Deep navy or charcoal background.
- Cyan, magenta, violet, orange, and gold bricks.
- Soft luminous edges.
- Bright white ball with a subtle trail.
- Sleek glowing paddle.
- Restrained particle bursts.
- Fine background grid or geometric patterns.
- Clean arcade typography.

Avoid excessive bloom and screen shake that obscure gameplay.

The ball must always remain clearly visible.

## 4. BALL PHYSICS — HIGHEST PRIORITY

Implement robust continuous 2D ball movement.

Each ball maintains:

- Position.
- Velocity.
- Radius.
- Speed.
- Active state.
- Trail history.

### Fixed Timestep

Use a fixed simulation timestep, preferably 1/120 second, with a bounded accumulator.

A lower timestep or collision substepping may be used when necessary.

Do not tie movement directly to rendering FPS.

### Wall Collisions

- Left wall reflects horizontal velocity.
- Right wall reflects horizontal velocity.
- Top wall reflects vertical velocity.
- Bottom boundary causes ball loss.
- Apply position correction after collisions.

### Paddle Collisions

Detect when a descending ball contacts the paddle.

Ball reflection depends on where the ball strikes:

- Left edge: reflect strongly left.
- Left-center: moderate left angle.
- Center: mostly upward.
- Right-center: moderate right angle.
- Right edge: strongly right.

Use normalized impact offset:

`offset = (ball.x - paddle.centerX) / (paddle.width / 2)`

Clamp offset to [-1, 1].

Calculate outgoing direction from the impact angle.

Preserve or slightly increase speed.

The ball must always leave a normal paddle hit traveling upward.

Enforce a minimum horizontal velocity so the ball does not get stuck in a perfectly vertical loop.

Limit extreme horizontal angles so the ball does not move nearly sideways forever.

### Paddle Movement Influence

Optionally apply a small influence from paddle velocity.

This should reward skilled movement without making rebounds unpredictable.

### Brick Collisions

Use circle-versus-rectangle collision detection.

For simple cases, determine collision normal from closest-point contact.

For high-speed motion, use swept circle-versus-expanded-AABB collision detection or sufficient adaptive substeps.

Correctly handle:

- Top contacts.
- Bottom contacts.
- Side contacts.
- Corner contacts.
- Multiple nearby bricks.
- High-speed ball movement.

Avoid destroying multiple unrelated bricks from one collision.

### Critical Physics Rules

- Never allow the ball to tunnel through multiple brick rows.
- Never reflect the ball in the wrong direction because of stale collision information.
- Never count the same brick impact repeatedly during overlap.
- Never allow the ball to become permanently embedded in a paddle, brick, or wall.
- Never produce NaN or infinite velocity.
- Enforce reasonable minimum and maximum speeds.
- Prevent repeated ball collisions from increasing speed without limit.

If simultaneous collision contacts occur, resolve the earliest valid collision first and continue the remaining timestep with a bounded number of collision iterations.

## 5. PADDLE CONTROL

### Keyboard

| Action | Control |
|---|---|
| Move Left | Left Arrow / A |
| Move Right | Right Arrow / D |
| Launch Ball | Space / Enter |
| Pause | P / Escape |
| Restart Level | R |

### Mouse

- Move paddle horizontally with the mouse.
- Click to launch a docked ball.
- Prevent unwanted text selection or drag behavior.

### Touch

- Drag finger horizontally to move paddle.
- Tap or use a launch button to release the ball.
- Support Pointer Events.
- Prevent unwanted page scrolling over the game.
- Handle pointer cancellation safely.

### Paddle Behavior

- Clamp paddle within the playfield.
- Smooth movement without noticeable input lag.
- Keep the paddle aligned to mouse/touch X coordinates.
- Do not allow the paddle to move outside the canvas.
- Keep the docked ball attached until launched.

## 6. BRICK TYPES

Implement several functional brick types.

### Normal Brick

- One hit destroys it.
- Awards points.
- Bright color and particle effect.

### Reinforced Brick

- Requires two hits.
- Changes appearance after the first hit.
- Awards extra points.

### Armored Brick

- Requires three hits.
- Displays visible damage states.
- Produces stronger impact effects.

### Explosive Brick

- One hit triggers an explosion.
- Damages nearby destructible bricks.
- Creates a clearly visible blast effect.
- Does not recursively explode without a bounded rule.

### Indestructible Brick

- Reflects the ball.
- Does not take damage.
- Does not count toward the required completion total.

### Bonus Brick

- Releases a guaranteed power-up.
- Distinct visual style.
- Counts toward level completion.

### Level Completion

Complete a level only when all required destructible bricks are gone.

Never allow indestructible bricks to prevent completion.

## 7. POWER-UPS

Some destroyed bricks should release a falling collectible.

The player activates the power-up by catching it with the paddle.

### Required Power-Ups

**Wide Paddle**

- Temporarily increases paddle width.
- Makes ball control easier.

**Narrow Paddle**

- Temporarily reduces paddle width.
- Acts as a challenge pickup.

**Slow Ball**

- Temporarily decreases ball speed.
- Respects a safe minimum.

**Fast Ball**

- Temporarily increases ball speed.
- Respects a maximum.

**Multiball**

- Splits an active ball into three balls.
- Gives the new balls slightly different trajectories.
- Prevents infinite multiball generation.

**Laser Paddle**

- Allows the paddle to fire limited projectiles.
- Projectiles damage bricks.
- Has a firing cooldown.
- Uses keyboard, mouse, or touch fire controls.

**Sticky Paddle**

- Temporarily catches a ball when it lands on the paddle.
- Player can launch the captured ball again.

**Extra Life**

- Awards one life.
- Enforce a reasonable maximum lives count.

### Rules

- Each power-up has a visual identifier.
- Timed effects display their remaining duration.
- Expired effects restore normal behavior.
- Conflicting size or speed modifiers have explicit resolution rules.
- Power-up timers pause when gameplay is paused.
- Falling power-ups despawn when they leave the playfield.
- Power-ups do not create invalid player or ball state.

## 8. MULTIBALL SYSTEM

Support multiple simultaneously active balls.

### Rules

- Each ball has independent position and velocity.
- Balls collide with bricks and paddle independently.
- Balls do not need to collide with one another.
- Losing one ball does not immediately cost a life.
- A life is lost only when no active balls remain.
- New balls spawn slightly separated from the original.
- Every ball uses the same collision rules.

### Limits

Cap active balls at a reasonable count, such as 8–12.

Prevent exponential growth during repeated multiball pickups.

Ensure performance remains smooth.

## 9. TEN HANDCRAFTED LEVELS

Create 10 complete levels with different visual formations and difficulty.

Use readable arrays or structured level data.

### Level 1 — First Contact

- Standard brick rows.
- Basic normal bricks.
- Gentle ball speed.
- Teaches paddle control.

### Level 2 — Color Cascade

- Alternating brick colors.
- Small gaps in formation.
- Introduces reinforced bricks.

### Level 3 — The Fortress

- Brick walls surrounding central targets.
- Armored bricks.
- Requires side-angle shots.

### Level 4 — Chain Reaction

- Explosive bricks.
- Dense formations.
- Rewards strategic aiming.

### Level 5 — The Maze

- Indestructible obstacles.
- Multiple channels.
- Requires careful paddle-angle control.

### Level 6 — Double Trouble

- More power-ups.
- Designed for multiball.
- Wider brick formations.

### Level 7 — Neon Spiral

- Spiral or staggered geometry.
- Difficult bounce angles.
- Higher ball speed.

### Level 8 — The Gauntlet

- Mixed armored and reinforced bricks.
- Narrow passages.
- Faster gameplay.

### Level 9 — Reactor Core

- Central armored structure.
- Explosive outer groups.
- Indestructible barriers.

### Level 10 — Final Breakthrough

- Large complex formation.
- All brick types.
- Strategic use of power-ups.
- Challenging final level.

### Level Validation

Every level must:

- Have at least one destructible brick.
- Be completable.
- Fit within the game area.
- Avoid overlapping invalid brick positions.
- Use valid brick types.
- Avoid permanently inaccessible destructible bricks.
- Have a clear completion condition.

Do not use randomly generated brick layouts as a substitute for 10 designed levels.

## 10. SCORING AND COMBOS

### Scoring

Suggested base values:

| Brick Type | Points |
|---|---|
| Normal | 10 |
| Reinforced | 20 |
| Armored | 30 |
| Explosive | 25 |
| Bonus | 15 |

Award points when bricks are actually destroyed, not repeatedly for the same damage event.

### Combo System

Increase the combo multiplier for consecutive brick destructions without a paddle bounce.

Reset combo when the ball returns to the paddle.

Cap the multiplier to avoid overflow or unreasonable scoring.

### Score Display

Show:

- Current score.
- High score.
- Current level.
- Lives.
- Remaining bricks.
- Combo multiplier when active.

### High Score

Store high score in localStorage.

Handle unavailable storage gracefully.

## 11. GAME STATES

Implement an explicit game state machine:

- TITLE
- READY
- PLAYING
- PAUSED
- LIFE_LOST
- LEVEL_COMPLETE
- GAME_OVER
- VICTORY

### Ready State

- Paddle is controllable.
- Ball remains attached to paddle.
- Player can launch the ball.
- No other game physics advances unnecessarily.

### Paused State

Freeze:

- Ball movement.
- Power-up movement.
- Timers.
- Projectiles.
- Game effects that influence state.

Rendering and menu input may continue.

### Life Lost

- Deduct exactly one life.
- Clear temporary ball states.
- Remove or resolve temporary power-ups consistently.
- Return to READY if lives remain.
- Enter GAME_OVER otherwise.

### Level Complete

- Stop gameplay.
- Award level bonus.
- Show transition effect.
- Load next level.

### Victory

After Level 10:

- Display final score.
- Display high score.
- Offer Play Again.

## 12. VISUAL EFFECTS

Create polished but lightweight effects using Canvas.

### Ball Trail

- Short glowing motion trail.
- Smooth fade.
- Trail follows actual ball positions.
- No excessive blur.

### Brick Destruction

- Small particles matching brick color.
- Brief impact flash.
- Subtle screen feedback.

### Paddle

- Glossy neon material.
- Edge highlights.
- Small glow on ball contact.

### Power-Ups

- Distinct glowing icons.
- Slow rotation or bobbing.
- Readable colors.
- Small collection burst.

### Background

- Dark futuristic arcade environment.
- Subtle grid.
- Dim geometric patterns.
- Gentle animated stars or particles.

Keep background activity subdued so gameplay remains readable.

### Screen Shake

Use only brief, subtle shake for major effects.

Do not shake the collision coordinate system.

## 13. AUDIO

Use the Web Audio API to synthesize retro arcade sounds.

Include:

- Paddle impact.
- Brick impact.
- Brick destruction.
- Explosive brick.
- Power-up pickup.
- Laser firing.
- Life lost.
- Level completion.
- Game over.
- Victory.

Use pitch variation to create satisfying feedback.

Do not create a new AudioContext for each effect.

Create or resume audio only after user interaction.

Include a mute toggle.

Do not let audio errors interrupt gameplay.

## 14. USER INTERFACE

### Title Screen

Display:

**NEON BREAKER**

Subtitle:

*Arcade Recharged*

Buttons:

- Start Game.
- How to Play.
- Sound On/Off.

### In-Game HUD

Display:

- Score.
- High score.
- Level.
- Lives.
- Remaining bricks.
- Active power-ups.

### Pause Menu

- Resume.
- Restart Level.
- Restart Game.
- Sound toggle.

### Game Over Screen

- Final score.
- Level reached.
- High score status.
- Retry button.

### Victory Screen

- Congratulations message.
- Final score.
- Levels cleared.
- Play Again button.

## 15. RESPONSIVE DESIGN

The game must work on desktop and mobile.

### Desktop

- Centered canvas.
- Responsive scaling.
- Clear HUD.
- Mouse and keyboard controls.

### Mobile

- Support landscape and portrait layouts where practical.
- Paddle movement through touch.
- Large accessible launch/fire/pause controls.
- Prevent unintended scrolling.
- Preserve game aspect ratio.

### High-DPI

- Match the canvas backing resolution to CSS size times capped DPR.
- Cap DPR at approximately 1.5–2.
- Keep logical gameplay coordinates independent of device pixels.
- Avoid stretched or blurry graphics.

## 16. PERFORMANCE

Target 60 FPS on ordinary desktop hardware.

### Requirements

- Use requestAnimationFrame.
- Use a fixed simulation timestep.
- Avoid allocating excessive objects inside update loops.
- Reuse particle and trail buffers.
- Limit active power-ups and balls.
- Avoid unnecessary full-screen blur effects.
- Avoid repeated gradient construction where practical.
- Do not recreate static geometry or level data every frame.
- Keep collision checks efficient.

Use broad-phase filtering for brick collision checks where appropriate.

## 17. COLLISION CORRECTNESS

This is the most important validation category.

The following bugs are unacceptable:

- Ball passing through bricks.
- Ball penetrating paddle.
- Ball stuck inside walls.
- Incorrect collision normal.
- Same brick taking repeated damage from one contact.
- Ball teleporting after collision.
- Ball moving horizontally forever.
- Ball becoming trapped between adjacent bricks.
- Multiball losing all lives prematurely.
- Ball speed increasing without bound.
- Explosive bricks causing repeated recursive destruction.
- Indestructible bricks being counted toward level completion.
- Paddle movement failing near screen edges.
- Stuck mouse or touch input.
- Power-up timers continuing while paused.

Use swept collision detection or bounded substeps to prevent tunneling.

Maintain consistent ball speed after reflections unless a defined game mechanic changes it.

## 18. IMPLEMENTATION PRIORITIES

Follow this order when complexity creates tradeoffs.

**Priority 1 — Working Application**

The page loads, renders, and supports a playable paddle and ball.

**Priority 2 — Accurate Physics**

Wall, paddle, and brick collisions behave reliably.

**Priority 3 — Complete Core Game**

Lives, scoring, launch, level progression, and game states work.

**Priority 4 — Ten Levels**

Every level is playable and completable.

**Priority 5 — Brick Types**

Reinforced, armored, explosive, indestructible, and bonus bricks work.

**Priority 6 — Power-Ups**

Multiball, paddle modifiers, sticky paddle, laser, and speed effects work.

**Priority 7 — Visual Polish**

Particles, glow, trails, animations, and transitions.

**Priority 8 — Audio and Mobile**

Synthesized sounds and touch controls.

Never sacrifice physics correctness for decorative effects.

## 19. INTERNAL VALIDATION

Before producing the final answer, inspect the implementation.

### Execution

- All functions and variables exist.
- Canvas initializes successfully.
- Game loop starts.
- No external dependencies.
- No unfinished code.

### Physics

- Ball reflects from all walls correctly.
- Paddle impact angle affects ball direction.
- Corner collisions work.
- High-speed collisions do not tunnel.
- Ball speed stays within bounds.
- Ball never becomes permanently embedded in geometry.

### Bricks

- Every brick type behaves correctly.
- Damage counts are accurate.
- Destroyed bricks disappear.
- Remaining brick count is correct.
- Indestructible bricks do not block completion.

### Power-Ups

- Pickups fall correctly.
- Paddle catches them.
- Effects activate and expire.
- Multiball has a ball limit.
- Laser uses proper cooldown.
- Sticky paddle launches captured balls correctly.

### Levels

- All 10 levels are present.
- All level definitions are valid.
- Level transitions work.
- All levels are completable.
- Victory activates after the final level.

### Game States

- Pause freezes gameplay.
- Life loss occurs exactly once.
- Restart resets appropriate state.
- Game over activates correctly.
- High scores are saved where available.

### Interface

- Mouse controls work.
- Keyboard controls work.
- Touch controls work.
- Buttons function.
- Audio mute works.
- Canvas resizes without distorting gameplay.

Fix any identifiable issues before finalizing.

Do not claim to have performed browser testing unless it was actually available.

## 20. OUTPUT FORMAT

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
- Missing levels.
- Omitted game systems.
- Installation instructions.

All HTML, CSS, JavaScript, physics, levels, controls, effects, audio, and initialization must be included.

Do not truncate the output.

## FINAL SUCCESS STANDARD

The user saves the response as `neon-breaker.html` and opens it in a modern browser.

A polished neon arcade title screen appears.

After starting, the player launches a glowing ball from a responsive paddle.

The ball bounces naturally, changes angle depending on paddle contact, destroys colorful bricks, and triggers satisfying visual and audio effects.

Power-ups introduce wider paddles, multiball, lasers, and other gameplay changes.

Ten increasingly challenging levels provide a complete arcade experience.

The game handles fast movement, corner collisions, and simultaneous balls without broken physics.

The result should feel like a modern, polished interpretation of classic brick-breaking arcade gameplay rather than a basic Canvas programming demonstration.