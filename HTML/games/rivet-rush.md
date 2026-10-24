<!--
Type: Implementation prompt
Target: HTML Canvas, vanilla JavaScript
Output: One standalone HTML file
Related: rivet-rush-phaser.md
-->

Create a complete, playable retro platform game called **Rivet Rush**, starring **Rory Rivet**, an original construction worker.

Return one complete HTML document only, from `<!DOCTYPE html>` through `</html>`. Put all CSS and vanilla JavaScript in that file. Do not use libraries, external files, network requests, placeholders, or unfinished code.

## Keep the implementation manageable

- Use a fixed 640×480 canvas. Each stage is a single screen with no scrolling.
- Build the three stages from reusable data and shared movement, collision, enemy, item, and rendering functions. Change layouts and objectives by stage; do not create three separate game engines.
- Use simple rectangles and a few canvas shapes for characters, platforms, items, hazards, and scenery. Prioritize readable gameplay over detailed art.
- Use simple AABB collisions and stable platform physics. Clamp Rory to the canvas sides. A fall below the play area costs a life.
- Use only the controls and mechanics specified below. Show the controls on the title screen and in a small in-game help line.

## Controls and movement

- Left/Right arrows: walk.
- Up/Down arrows: climb a ladder when Rory overlaps it.
- Space: jump. A fresh key press triggers one jump; holding Space must not cause repeated jumps.
- X: pick up, place, or deliver an item when Rory is close to a valid target. Show a brief prompt such as “Press X to pick up”.
- Rory can jump while standing on a platform. He cannot jump while climbing or carrying a girder. Walking off a platform makes him fall.
- Ladders let Rory move between platforms without falling. Stop climbing when he reaches a platform or ladder end.

## Shared rules and interface

- Start each new game with 3 lives, score 0, and a 75-second timer for the current stage. Pause the timer during title, death, and stage-transition screens.
- Show stage number/name, objective progress, lives, score, and remaining time in a readable HUD.
- Briefly flash or animate an objective when it changes. Keep collected and delivered progress after a death. Drop any carried item at Rory’s respawn point.
- Contact with an enemy or hazard, falling off-screen, or reaching zero time costs one life. Respawn Rory at that stage’s start after a short pause. At zero lives, show Game Over; Space starts a new game from Stage 1.
- Give Rory a short invulnerability blink after respawning so he cannot lose another life immediately.
- When a stage objective is complete, show “Stage Complete” for about 1.5 seconds, then load the next stage. Preserve score and lives; reset the timer. After Stage 3, show a round-complete message, then start a faster round with the same three stages, slightly faster enemies, and no new mechanics.
- Make enemies move at steady, reversible speeds. Give falling bolts a visible warning for about one second before they drop. Hazards must be possible to avoid with the stated controls.

## Stages

Use 3–4 platforms per screen, with ladders connecting them. Place Rory, objectives, hazards, and exits so the player can reach every required area. Use distinct colors and labels so objects are easy to recognize.

### Stage 1: Building Framework

- Place four loose girder pieces on reachable platforms and four matching floor gaps.
- Rory can carry one girder at a time. Press X beside a loose piece to pick it up; walking speed is slightly slower while carrying. Press X beside an unfilled gap to place the piece. Mark each filled gap visibly and show progress as 0/4 through 4/4.
- After all four gaps are filled, activate a moving jackhammer that travels along a platform. Rory completes the stage by touching it. Do not allow the jackhammer to finish the stage before all gaps are filled.
- Add occasional falling bolts with a warning, plus one vandal and one safety inspector patrolling fixed platform sections. Keep their paths visible and avoid blocking all routes.
- Place one springboard where it helps Rory reach an upper platform. Landing on it launches Rory upward to that platform; show a brief squash-and-rebound animation.

### Stage 2: Construction Site

- Build four platform tiers and place six lunchboxes across them. Rory collects a lunchbox by touching it. Show progress as 0/6 through 6/6.
- Put one patrolling enemy at the final hurdle. Leave enough room to see it and time a jump over it.
- Once all six lunchboxes are collected, activate the conveyor belt. It carries Rory toward a marked electromagnet zone. Reaching that zone completes the stage after a short lift animation. A clearly marked incinerator below is a lethal pit.
- Add one elevator that travels between two fixed stops and carries Rory when he stands on it. Add one springboard only if it fits without obscuring the objective route.

### Stage 3: Factory

- Place six boxes and one processor with six visible empty slots. Rory picks up a box with X, carries one at a time, and presses X beside the processor to deliver it. Show progress as 0/6 through 6/6; visibly fill a processor slot after each delivery.
- Put one patrolling enemy near one box. Make its patrol and a safe jumping route readable.
- Place a portable toilet hazard at the bottom center. Keep it clearly visible and avoid making it cover the only route between platforms.
- Completing all six deliveries completes the stage.

## Fun and feedback

- Keep jumps forgiving: consistent jump height, responsive left/right movement, and enough space to react to hazards.
- Keep the route to each objective understandable. Use platform placement, color, arrows, or a small marker to suggest the next target.
- Award a small score bonus for finishing a stage without losing a life. Do not make score necessary to progress.
- Use short visual feedback for pickup, delivery, damage, and stage completion. Audio is optional; if included, generate it with the Web Audio API and provide a mute toggle.

## Completion checklist

Before returning the HTML, ensure the code includes a title/start state, all three playable stages, all specified objective interactions, stage transitions, death/respawn, timer, Game Over, restart, and the faster repeat round. Prevent browser arrow keys and Space from scrolling the page while the game is active. Return code only, with no explanation before or after it.
