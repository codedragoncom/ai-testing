Act as an expert retro game developer. Write a complete, standalone, single-file HTML5/JavaScript game inspired by the 1983 classic "Hard Hat Mack".

All HTML, CSS, and JavaScript must be fully written inside a single self-contained file. Do not use external libraries, external images, or external audio.

### GAMEPLAY SPECIFICATIONS:
1. Setting & Canvas:
   - Fixed canvas size: 640x480, dark industrial background.
   - 3 to 4 horizontal girder platforms connected by vertical climbable ladders.
2. Player ("Mack"):
   - Rendered using simple canvas shapes (yellow hard hat, blue body).
   - Controls: Arrow Left/Right to walk, Arrow Up/Down to climb ladders, Spacebar to jump.
   - Platformer gravity: fall when walking off edges unless touching a ladder.
3. Mechanics & Objective:
   - Place 4 collectible items (rivets/iron pieces) across different girder levels.
   - Mack must collect all rivets to complete the level.
   - Display a HUD at the top: Score, Lives (start with 3), and "Level Cleared" notification.
4. Hazards:
   - One patrolling enemy ("Vandal" or "Inspector") on one girder moving back and forth.
   - Occasional falling debris/tool dropping from the top screen down a fixed vertical chute.
   - Touching a hazard or falling into the bottom pit resets Mack to the starting girder and deducts 1 life.
5. Game Loop & State:
   - Handle Start, Playing, Death/Respawn, Level Win, and Game Over states.
   - Restart button or 'Press Space to Restart' on Game Over.

### CODE CONSTRAINTS FOR COMPACT EXECUTION:
- Keep the physics simple: basic AABB (bounding box) collisions for platforms, ladders, and collectibles.
- Write clean, robust, vanilla ES6 JavaScript inside a single <script> tag.
- DO NOT use placeholders, ellipses (...), or comments like "// implement later".
- Deliver the full, working, unbroken code starting with <!DOCTYPE html> and ending with </html>.