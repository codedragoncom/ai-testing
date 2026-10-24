<!--
Type: Implementation prompt
Target: HTML Canvas, Web Audio API, vanilla JavaScript
Output: void-sentinel.html
Related: README.md
-->

# VOID SENTINEL — Single-File Arcade Space Shooter

You are an expert JavaScript game developer specializing in classic arcade games, HTML5 Canvas, game physics, enemy AI, and procedural graphics.

Create a complete, polished, fully playable arcade space shooter called **VOID SENTINEL**, inspired by the fast-paced gameplay of the 1981 arcade game *Defender*.

The result must be a standalone file named `void-sentinel.html`.

## 1. Technical Requirements

- Use HTML5 Canvas 2D for all rendering.
- Use vanilla JavaScript and CSS.
- Everything must exist inside ONE HTML file.
- No external libraries, frameworks, images, fonts, or audio files.
- No CDN dependencies or network requests.
- Game must work by opening the HTML file directly in Chrome, Firefox, or Edge.
- Internal resolution: 960 × 540.
- Scale the canvas to fit the browser while preserving aspect ratio.
- Use `requestAnimationFrame` with delta-time-based movement.
- Keep the code organized into clearly named classes and functions.

## 2. Game Concept

The player pilots the VX-9 Sentinel, a fast combat spacecraft defending a mining colony from alien invasion.

Enemy ships invade a large, horizontally scrolling planet and attempt to abduct colonists from the surface.

The player must:

1. Destroy hostile spacecraft.
2. Prevent colonists from being abducted.
3. Rescue colonists falling from destroyed abductors.
4. Return rescued colonists safely to the planet.
5. Survive increasingly difficult enemy waves.
6. Earn the highest possible score.

The experience should feel like a fast, responsive 1980s arcade cabinet with modern visual polish.

## 3. World and Camera

Create a continuous, horizontally wrapping world.

- World width: 4,800 pixels.
- World height: 540 pixels.
- Visible viewport: 960 × 540.
- The player's ship remains near the horizontal center of the screen.
- The world scrolls smoothly as the player travels.
- Crossing either world boundary wraps to the opposite side.
- All moving objects use world coordinates.
- Implement wrap-aware distance calculations for movement, targeting, collisions, and rendering.
- Draw visible objects correctly when near the world's wrapping seam.

Generate a fixed mountainous planet surface using a deterministic combination of sine waves.

The surface should feature:

- Jagged alien mountains.
- Glowing mining installations.
- Occasional antenna towers.
- Small landing platforms.
- Atmospheric haze.
- Parallax stars and distant terrain.

The surface is dangerous to the ship, but colonists can stand on it safely.

## 4. Player Spacecraft

Create an original triangular combat fighter with animated thrusters, glowing engines, and directional laser cannons.

Movement:

- W / Up Arrow: Fly upward.
- S / Down Arrow: Fly downward.
- A / Left Arrow: Accelerate left and face left.
- D / Right Arrow: Accelerate right and face right.
- Space: Fire lasers.
- B: Detonate smart bomb.
- H: Hyperspace jump.
- P or Escape: Pause.

Physics:

- Acceleration-based movement.
- Horizontal and vertical velocity.
- Maximum speed of 340 pixels/second.
- Smooth deceleration when movement keys are released.
- Responsive directional reversal.
- Keep the player within the vertical playable area.
- Ship faces its horizontal travel direction.
- Lasers always fire in the direction the ship faces.

Player starts with:

- 3 lives.
- 3 smart bombs.
- 1 hyperspace charge per wave.

After taking damage:

- Lose one life.
- Play an explosion animation.
- Respawn at a safe location.
- Grant 2 seconds of flashing invulnerability.

## 5. Weapons

### Laser Cannon

- Rapid horizontal laser fire.
- Maximum 6 player lasers active simultaneously.
- Fire cooldown: 120 milliseconds.
- Lasers travel at 950 pixels/second.
- Destroy enemies on collision.
- Add bright muzzle flashes and impact particles.

### Smart Bomb

- Press B to activate.
- Limited ammunition.
- Instantly destroys all currently visible enemies.
- Creates a large expanding energy-wave effect.
- Does not destroy colonists.
- Award normal enemy destruction points.
- Prevent the same enemy from being scored twice.

### Hyperspace

- Press H to teleport.
- Teleports the ship to a random safe location.
- Creates a distortion and particle effect.
- Uses one charge.
- Regenerate the charge at the beginning of each wave.
- Choose a destination that does not overlap terrain or a known enemy.

## 6. Enemy Types

Implement four distinct enemy types using simple, reliable AI state machines.

### A. Raiders

Primary enemy.

Behavior:

- Spawn high above the planet.
- Locate an available colonist.
- Descend toward that colonist.
- Capture the colonist upon reaching them.
- Carry the colonist upward.
- If the Raider reaches the upper atmosphere with its captive, the colonist is lost and the Raider transforms into a Hunter.

Points: 150.

### B. Hunters

Aggressive combat enemies.

Behavior:

- Actively pursue the player.
- Move faster than Raiders.
- Change direction as the player moves.
- Occasionally fire hostile projectiles.
- Never attempt colonist abduction.

Points: 250.

### C. Drone Carriers

Large, slow alien vessels.

Behavior:

- Patrol the upper atmosphere.
- Move horizontally.
- Occasionally change direction.
- When destroyed, release three small Attack Drones.

Points: 300.

### D. Attack Drones

Small, fast enemies.

Behavior:

- Chase the player.
- Move in short directional bursts.
- Have low health.
- Can collide with the player.

Points: 100.

All enemies must use procedural geometric sprites with distinct colors and readable silhouettes.

## 7. Colonist Rescue System

Place 10 colonists at fixed, evenly distributed ground positions around the planet.

Each colonist has one of these states:

- GROUNDED
- CAPTURED
- FALLING
- CARRIED
- LOST

Required behavior:

1. Raiders can capture GROUNDED colonists.
2. Captured colonists visibly hang beneath the Raider.
3. Destroying the Raider releases the colonist.
4. Released colonists fall under gravity.
5. The player catches a FALLING colonist through proximity collision.
6. A caught colonist attaches beneath the player's ship.
7. Flying near the ground at a safe speed delivers the colonist.
8. Delivered colonists return to GROUNDED state.
9. Falling colonists who hit the ground without rescue become LOST.
10. A Raider reaching the top boundary while carrying a colonist causes the colonist to become LOST and the Raider to transform into a Hunter.

Only one colonist can be carried by the player at a time.

Award:

- 500 points for catching a falling colonist.
- 1,000 points for safely returning a colonist.

Keep rescue mechanics reliable and prevent duplicate scoring.

## 8. Radar System

Display a horizontal radar at the top of the screen.

Radar requirements:

- Represent the entire 4,800-pixel world.
- Show the player's current location.
- Show enemy positions.
- Show grounded colonists.
- Highlight captured colonists.
- Indicate the player's visible viewport.
- Account for horizontal world wrapping.

Use different colors:

- Cyan: player.
- Red: enemies.
- Green: safe colonists.
- Yellow: captured or falling colonists.

Keep the radar readable without covering the gameplay.

## 9. Wave Progression

Implement unlimited waves with increasing difficulty.

| Wave | Raiders | Carriers | Difficulty |
|---|---:|---:|---|
| 1 | 5 | 0 | Easy |
| 2 | 7 | 1 | Moderate |
| 3 | 9 | 2 | Hard |
| 4 | 11 | 3 | Very hard |
| 5+ | Scales upward | Scales upward | Escalating |

Rules:

- Spawn enemies gradually rather than all at once.
- Advance when all required enemies and their spawned descendants are destroyed.
- Increase enemy movement speed slightly each wave.
- Keep a reasonable maximum number of active enemies.
- Show a brief wave-complete animation.
- Award bonus points for surviving colonists.
- Restore one smart bomb after each completed wave, up to a maximum of three.
- Do not automatically restore lost colonists.

If every colonist is lost, the player can continue fighting, but receives no survival bonus.

## 10. Scoring and HUD

Display:

- Current score.
- High score.
- Lives remaining.
- Smart bombs remaining.
- Hyperspace availability.
- Current wave.
- Surviving colonist count.

Save the high score using localStorage.

Handle unavailable localStorage without crashing.

Award an extra life every 10,000 points, up to a maximum of five lives.

## 11. Visual Direction

Create a visually impressive neon-vector arcade style.

Palette:

- Deep black and navy background.
- Cyan player spacecraft.
- Red and magenta enemies.
- Green planetary installations.
- Orange weapon explosions.
- Yellow rescue effects.

Effects:

- Parallax starfield.
- Animated thruster trails.
- Laser glow.
- Explosion particles.
- Screen shake on major impacts.
- Smooth camera tracking.
- Small engine sparks.
- Floating score popups.
- Subtle CRT scanlines.
- Animated wave announcements.

Use Canvas drawing operations only.

Do not load external image assets.

Prioritize clear gameplay silhouettes over complicated decorative effects.

## 12. Sound

Use the Web Audio API to generate:

- Laser fire.
- Enemy explosions.
- Player explosions.
- Rescue pickup.
- Successful delivery.
- Smart bomb detonation.
- Hyperspace activation.
- Wave completion.
- Game over.

Initialize or resume audio only after a user interaction.

Add an M key to toggle sound.

## 13. Game Screens

Implement the following game states:

- TITLE
- PLAYING
- PAUSED
- WAVE_COMPLETE
- GAME_OVER

Title screen must show:

- VOID SENTINEL title.
- Animated spacecraft.
- Brief mission description.
- Keyboard controls.
- Press Enter to Start.
- Stored high score.

Game Over screen must show:

- Final score.
- Highest wave reached.
- Colonists saved.
- High score.
- Press Enter to Restart.

## 14. Architecture

Organize the code with these logical components:

- Game
- Player
- Enemy
- Colonist
- Projectile
- Particle
- Camera
- World
- InputManager
- AudioManager
- Renderer

These can be ES6 classes inside a single script tag.

Use arrays for entities and particles.

Use a clear update/render separation.

Keep functions small and understandable.

Important implementation rules:

- Use delta time for all movement.
- Clamp large delta-time spikes.
- Reset input state when the browser loses focus.
- Prevent default browser scrolling for gameplay keys.
- Avoid modifying entity arrays while iterating forward during removal.
- Use circular-world distance calculations consistently.
- Avoid spawning enemies directly on the player.
- Clean up expired projectiles and particles.
- Prevent rapid collision damage during respawn invulnerability.
- Maintain stable performance with many active objects.
- Do not use undefined helper functions or placeholder implementations.

## 15. Gameplay Balance

The first wave should be accessible but engaging.

Targets:

- First enemy encounter within 5 seconds.
- First possible abduction within 10–20 seconds.
- Typical first-wave duration: 45–90 seconds.
- Clear visual feedback for approaching abductions.
- Player movement must feel fast and responsive.
- Laser collisions should be forgiving enough for arcade gameplay.

Do not make the enemies perfectly accurate.

## 16. Testing Requirements

Before finishing, mentally trace and verify these scenarios:

1. Starting the game initializes all entities.
2. Player movement and world scrolling work in both directions.
3. The world wraps cleanly.
4. Laser collisions destroy enemies.
5. Raiders correctly locate colonists.
6. Raiders can capture and carry colonists.
7. Captured colonists are released when Raiders are destroyed.
8. Falling colonists can be caught.
9. Carried colonists can be returned safely.
10. Captured colonists are lost if taken to the upper boundary.
11. Hunters correctly pursue the player.
12. Destroying a Carrier spawns exactly three Drones.
13. Smart bombs destroy visible enemies.
14. Hyperspace relocates the player safely.
15. Player death reduces lives exactly once.
16. Clearing all wave enemies advances the game.
17. Pause freezes all simulation timers.
18. Game over and restart reset the game correctly.
19. High scores persist when storage is available.
20. The game works without an internet connection.

## 17. Output Requirements

Return ONLY the complete source code for `void-sentinel.html`.

The file must contain:

- Complete HTML document.
- Embedded CSS.
- Embedded JavaScript.
- All graphics rendered procedurally.
- Complete playable game logic.
- No TODO comments.
- No pseudocode.
- No missing functions.
- No placeholders.
- No external dependencies.

Ensure the game can be saved as `void-sentinel.html` and played immediately by opening it in a desktop browser.

**Priority order:**

1. Complete playable game.
2. Correct movement and collision.
3. Reliable enemy and rescue mechanics.
4. Stable scrolling and radar.
5. Wave progression.
6. Graphics and sound polish.

If implementation complexity becomes an issue, simplify visual effects before removing core gameplay mechanics.

Build the complete game now.