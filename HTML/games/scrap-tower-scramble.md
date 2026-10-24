<!--
Type: Implementation prompt
Target: HTML Canvas, vanilla JavaScript
Output: scrap-tower-scramble.html
Related: None
-->

You are an expert JavaScript game developer specializing in classic arcade games.

Create a complete, polished, fully playable browser game titled "SCRAP TOWER SCRAMBLE".

OUTPUT REQUIREMENT:
Generate exactly one self-contained file named scrap-tower-scramble.html.
Include all HTML, CSS, JavaScript, game logic, graphics, animations, and sound within the file.
Use HTML5 Canvas 2D and vanilla JavaScript only.
Do not use libraries, frameworks, external images, external audio, CDN resources, or network requests.
The game must work by opening the HTML file directly in a modern desktop browser.

GAME CONCEPT:
Scrap Tower Scramble is an original 1980s-style, single-screen arcade platformer inspired by classic ladder-climbing games.

The player controls Sparky, a small yellow maintenance robot who must ascend a dangerous scrap-metal tower and rescue a friendly blue robot named BEEP. At the top, GRUDGE-9000, a giant red demolition robot, launches rolling metal drums and other hazards down the tower.

Use original character designs, artwork, names, and sounds. Do not reproduce copyrighted characters or exact stage layouts.

GAMEPLAY:
- Side-view, single-screen arcade platformer.
- Logical canvas resolution 800x600, responsively scaled while preserving aspect ratio.
- Six staggered industrial platforms form a winding climb from the bottom left toward the top.
- Platforms alternate slightly upward and downward slopes, making a zigzag climbing route.
- Fixed ladders connect neighboring platforms at appropriate endpoints.
- Player runs left/right, jumps, climbs ladders, and avoids hazards.
- GRUDGE-9000 periodically releases metal drums from the upper platform.
- Drums roll downhill along platforms, reverse direction on successive floors, and fall between floors at defined endpoints.
- Some drums occasionally descend ladders.
- Reaching BEEP at the top completes the stage.
- Each completed stage restarts the tower with more difficult hazards.
- The player has 3 lives, a score, and a decreasing bonus timer.
- The game ends when all lives are lost.

CONTROLS:
Arrow Left / A: move left
Arrow Right / D: move right
Arrow Up / W: climb ladder
Arrow Down / S: descend ladder
Space: jump
Enter: start/restart
P: pause/unpause
M: mute/unmute

PLAYER PHYSICS:
- Use delta-time based movement and requestAnimationFrame.
- Implement horizontal movement, gravity, vertical velocity, jumping, and falling.
- Land the player on sloped platforms using interpolation of the platform's endpoint heights.
- Prevent passing through platforms when descending onto them.
- Allow jumping over hazards and small platform gaps.
- Ladder climbing must snap or smoothly align the player to the ladder.
- Gravity must not pull the player down while actively climbing.
- The player must be able to exit ladders cleanly at both endpoints.
- Collisions must use predictable hitboxes.
- After damage, briefly grant invulnerability with blinking animation.
- Respawn the player at the starting position after losing a life.
- Avoid double collisions and repeated life losses.

HAZARDS:
1. Rolling steel drums:
   - Spawn at the top at controlled intervals.
   - Roll in the platform's travel direction.
   - Follow the sloped platform surface.
   - Fall at platform endpoints onto the next platform.
   - Rotate visually while moving.
   - Can be jumped over.

2. Electric sparks:
   - Patrol selected platforms.
   - Move back and forth over a limited distance.
   - Have a distinctive animated electric glow.
   - Increase in number only at higher stages.

3. Falling scrap:
   - Introduce from stage 3 onward.
   - Fall from predictable spawn points.
   - Show a brief warning indicator before falling.

POWER-UP:
Include a temporary magnetic wrench power-up.
- Collecting it activates 7 seconds of protection.
- It destroys nearby drums and sparks when they enter wrench range.
- Display its remaining duration.
- Give bonus points for destroyed hazards.
- Show an obvious visual effect around Sparky.
- Do not allow unlimited stacking of the power-up.

SCORING:
Jump over drum: +100
Destroy hazard: +200
Collect bonus gear: +300
Complete stage: +1000 plus remaining time bonus
Award one extra life at 10000 points, only once.
Save the best score using localStorage, with graceful fallback if storage is unavailable.

STAGE PROGRESSION:
Stage 1: Basic rolling drums and ladders.
Stage 2: Faster drums and one electric spark.
Stage 3: Additional sparks and falling scrap.
Stage 4+: Gradually increase hazard speed and spawn frequency, with sensible upper limits.

Keep the same core stage layout for reliability. Progression should remain fair and playable.

VISUAL STYLE:
- Authentic 1980s arcade-inspired pixel aesthetic with original artwork.
- Nearly black navy background.
- Orange and rust-red steel platforms.
- Cyan ladders.
- Yellow playable robot with expressive blue eyes.
- Red demolition boss with animated glowing eyes.
- Blue robot waiting at the tower summit.
- Gray rolling drums with bright metal highlights.
- Cyan and white electrical sparks.
- Small animated gears, bolts, and industrial particles.
- Clearly visible character silhouettes.
- Subtle scanlines and a restrained CRT glow.
- Readable retro-styled interface using local system monospace fonts.
- Do not make the image too dark or visually cluttered.

DRAWING:
All graphics must be rendered programmatically with Canvas 2D.
Draw the robot characters using rectangles, circles, lines, and simple shapes.
Animate Sparky's legs while moving, eyes while idle, and arms while climbing.
Animate GRUDGE-9000 before launching hazards.
Include small particle bursts when hazards are destroyed.
Use consistent world coordinates and rendering scale.

USER INTERFACE:
Title screen:
- SCRAP TOWER SCRAMBLE logo
- Animated tower preview or moving hazards
- Press Enter to Start
- Short control instructions

Gameplay HUD:
- Current score
- High score
- Lives
- Stage
- Remaining bonus
- Power-up timer when active

Pause screen:
- Dark overlay
- PAUSED label
- Resume instructions

Game-over screen:
- Final score
- High score
- Restart instructions

Stage-complete screen:
- Victory message
- Stage bonus
- Short transition before next stage

AUDIO:
Use Web Audio API with simple synthesized sound effects.
Include jump, climb, collect, collision, power-up, stage completion, and game-over sounds.
Do not load audio files.
Initialize or resume audio only after user interaction.
Provide mute/unmute functionality.
Audio failures must never stop the game.

ARCHITECTURE:
Use one script with organized sections:
1. Constants and configuration
2. Canvas setup and responsive scaling
3. Game state management
4. Input handling
5. Platform and ladder definitions
6. Player physics and collision
7. Hazard spawning and behavior
8. Pickups and power-ups
9. Scoring and progression
10. Audio generation
11. Rendering and visual effects
12. Main game loop

Prefer straightforward functions, arrays, and objects.
Avoid unnecessary inheritance, complex abstraction, or large dependency-style systems.
Use deterministic platform coordinates instead of procedural generation.
Limit active hazards to maintain reliable performance.
Clamp excessive frame delta after tab switching.
Use a fixed timestep for simulation if practical.

IMPORTANT IMPLEMENTATION DETAILS:
- Define platform geometry with x1,y1,x2,y2 and a platform thickness.
- Implement a helper to obtain the y coordinate of a platform at any x position.
- Ensure ladders actually connect two platforms and are accessible.
- Spawn the player standing on the lowest platform, not inside it.
- Prevent impossible platform jumps or unreachable ladders.
- Use stage data that guarantees a traversable route to the rescue target.
- Prevent hazards from spawning directly on the player.
- Never modify an active array unsafely while iterating through it.
- Clean up destroyed and offscreen hazards.
- Reset transient state after death or stage completion.
- Ensure pause freezes gameplay timers and physics.
- Prevent Space and Arrow keys from scrolling the browser.
- Avoid undefined function calls, placeholder code, and incomplete features.

PRIORITY ORDER:
1. Working game loop and controls
2. Correct platform physics and ladder climbing
3. Working hazards, collisions, and lives
4. Reachable goal and stage completion
5. Scoring, power-ups, and progression
6. Visual polish, animation, and sound

If implementation complexity becomes too high, simplify visual effects before sacrificing core gameplay.

FINAL SELF-CHECK:
Before outputting, review the implementation and correct obvious issues:
- The game starts from the title screen.
- Sparky can move, jump, and climb.
- Sparky lands correctly on every platform.
- All ladders are usable.
- Drums travel across platforms and fall correctly.
- Hazard collisions remove one life and trigger invulnerability.
- The power-up works and expires.
- The summit can be reached.
- Reaching BEEP completes the stage.
- Score, high score, pause, restart, and game-over work.
- No external dependencies are required.
- The HTML contains no TODOs, pseudocode, or omitted sections.

OUTPUT FORMAT:
Return only the complete HTML code, starting with <!DOCTYPE html> and ending with </html>.
Do not include explanations, markdown fences, or any text outside the HTML.
