<!--
Type: Implementation prompt
Target: Phaser 3.90.0, HTML, vanilla JavaScript
Output: One HTML document
Related: rivet-rush.md
-->

Create a polished, playable retro platform game called **Rivet Rush**, starring **Rory Rivet**, an original construction worker.

Return one complete HTML document only, from `<!DOCTYPE html>` through `</html>`. Use Phaser **3.90.0** from the pinned CDN below, with all game-specific CSS and JavaScript embedded in the HTML. Do not use other libraries, external images, audio files, network requests, placeholders, or unfinished code.

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/phaser/3.90.0/phaser.min.js"></script>
```

This Phaser version is pinned so the code uses a known API. Build the art from Phaser Graphics, shapes, and text; do not load assets.

## Visual and presentation goals

- Create an attractive 640×480 game area that scales proportionally to fit the browser window, with a dark navy industrial palette, warm yellow safety accents, clear silhouettes, and readable contrast.
- Add a designed title screen, stage title cards, a clean HUD, and brief overlays for pickup, delivery, damage, stage completion, round completion, and Game Over.
- Use restrained polish that supports play: subtle background details, platform edge highlights, character movement animation, spring compression, elevator motion, conveyor stripes, a brief camera shake on damage, and small particle bursts on pickup or delivery.
- Keep text legible and game space uncluttered. Support keyboard input and prevent arrow keys/Space from scrolling the page while the game is active.

## Manageable technical structure

- Use Phaser 3.90.0 Arcade Physics for gravity, platform collisions, overlaps, and enemy collisions. Use simple rectangles and a small amount of custom logic for ladders, carrying, conveyors, springboards, and elevators.
- Keep each stage on one fixed screen with no scrolling. Use a 640×480 logical game size with Phaser scale mode `FIT` and auto-centering.
- Organize the code into a small set of scenes: `BootScene` (create generated textures if needed), `TitleScene`, `GameScene`, and `UIScene` only if it simplifies implementation. Prefer one reusable gameplay scene configured with stage data over copied stage logic.
- Implement stage layouts, item locations, ladder zones, moving platforms, and enemy patrol bounds as data. Destroy or reset stage objects cleanly when changing stages.
- Keep the HTML self-contained apart from the pinned Phaser CDN script. If the CDN cannot load, display a clear message in the page instead of a blank screen.

## Controls and movement

- Left/Right arrows: walk.
- Up/Down arrows: climb a ladder while overlapping it.
- Space: jump; one fresh key press causes one jump, and holding the key must not auto-repeat.
- X: pick up, place, or deliver an item while close to a valid target. Display a short contextual prompt, such as “X: Pick up girder”.
- Rory can jump from platforms, but not while climbing or carrying a girder. Walking off an edge makes him fall. Clamp movement at the left and right screen edges.
- Use forgiving, consistent jump physics and responsive movement. Ensure each objective and exit can be reached with the stated controls.

## Shared rules and interface

- New game starts with 3 lives, score 0, and a 75-second timer for each stage. Pause time during title, damage/respawn, and stage-transition overlays.
- Keep stage number/name, objective progress, lives, score, and time visible in a compact HUD.
- Preserve completed objective progress after a death. Drop a carried item at Rory’s respawn point. Grant brief invulnerability after respawn, shown with a clear blink effect.
- Enemy/hazard contact, falling off-screen, and timer expiration cost one life. Respawn Rory at the current stage’s start. At zero lives, show Game Over; Space starts a fresh game at Stage 1.
- Completing an objective immediately triggers a “Stage Complete” overlay for about 1.5 seconds, then loads the next stage. Preserve score and lives and reset the timer. After Stage 3, show a round-complete overlay, then repeat the stages with slightly faster enemies and no new mechanics.
- Enemies patrol fixed bounds at steady speeds. Falling bolts show a warning for about one second. Keep hazards avoidable and readable.
- Award a small score bonus for finishing a stage without losing a life. Score is never required for progression.

## Stage objectives

Use 3–4 platforms per stage, connected by ladders. Make routes, items, enemies, and hazards distinct and easy to read. Provide a short stage intro that names the objective.

### Stage 1: Building Framework

- Place four loose girder pieces and four corresponding floor gaps. Rory can carry one girder at a time. X picks it up near Rory; X places it when he is beside an unfilled gap. Carrying slightly reduces walk speed. Show 0/4 through 4/4 and visibly fill each gap.
- Only after all gaps are filled, activate a jackhammer that moves along a platform. Touching it completes the stage.
- Add occasional falling bolts with a warning, plus one vandal and one safety inspector patrolling defined platform sections. Keep escape paths open.
- Place one springboard to help reach an upper platform. On landing, it launches Rory upward to that platform and briefly compresses/rebounds.

### Stage 2: Construction Site

- Build four platform tiers. Place six lunchboxes; collect by touching them and show 0/6 through 6/6.
- Add one enemy guarding the final hurdle. Leave enough room for the player to see and time a jump over it.
- After all lunchboxes are collected, activate a conveyor belt leading toward a marked electromagnet zone. Reaching the zone triggers a short lift animation and completes the stage. The clearly marked incinerator below is a lethal pit.
- Add one elevator with two fixed stops. It carries Rory while he stands on it. Add a springboard only if it does not clutter the route.

### Stage 3: Factory

- Place six boxes and a processor with six visible slots. Rory carries one box at a time: X picks up a nearby box, and X beside the processor delivers it. Show 0/6 through 6/6 and visibly fill a processor slot after each delivery.
- Add one enemy patrolling near a box, with a readable patrol and safe jump route.
- Place a portable toilet hazard at the bottom center. Keep it visible without blocking the only route.
- Completing all six deliveries completes the stage.

## Finish criteria

Before returning the HTML, check that it contains a title/start flow, all three distinct playable stages, working controls, item interactions, platform and ladder movement, springboard, conveyor, elevator, enemies, falling-bolt warning, timer, lives, score, death/respawn, stage transitions, Game Over/restart, and the faster repeat round. Handle missing Phaser CDN access visibly. Return only the complete HTML document.
