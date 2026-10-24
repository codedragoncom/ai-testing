<!--
Type: Implementation prompt
Target: HTML Canvas, vanilla JavaScript
Output: One standalone HTML file
Related: None
-->

You are an expert frontend developer. Write a complete, self-contained single-file HTML document that implements a simple but functional ant colony simulator.

Requirements (must follow exactly):
- One single HTML file only. No external files, no CDN, no frameworks, no libraries.
- Use a `<canvas>` for rendering.
- Pure vanilla JavaScript inside the HTML file.
- The page must work immediately when opened in a browser.

Core simulation features:
1. A nest (home) located near the center of the canvas.
2. 2–4 food sources placed at random locations (or fixed corners). Each food source has a limited amount of food.
3. Multiple ants (start with 60–100). Each ant has:
   - Position (x, y)
   - Angle (direction)
   - State: "searching" or "carrying food"
4. Two pheromone fields stored as 2D arrays (or Float32Array grids):
   - Food pheromone (deposited by ants that found food)
   - Home pheromone (deposited by ants carrying food back to the nest)
5. Pheromones slowly decay every frame.
6. Ant behavior:
   - While searching: deposit food pheromone, sense nearby food pheromone, and bias movement toward stronger food pheromone. Also randomly wander.
   - When an ant touches a food source: it picks up food, switches to "carrying" state, and turns around.
   - While carrying: deposit home pheromone, sense nearby home pheromone, and bias movement toward stronger home pheromone (to return to the nest).
   - When an ant reaches the nest while carrying food: drop the food and switch back to searching.
7. Visuals:
   - Draw the nest as a colored circle.
   - Draw food sources as colored circles (size or color based on remaining food).
   - Draw ants as small lines or triangles (different color when carrying food).
   - Draw pheromone trails with low-opacity colored pixels or cells (food pheromone one color, home pheromone another).
8. Controls (simple HTML elements above the canvas):
   - Start / Pause button
   - Reset button
   - Slider or input for number of ants
   - Slider for simulation speed
9. Canvas should resize reasonably or be fixed at a large size (e.g. 800×600 or full window).
10. Use requestAnimationFrame for the game loop.
11. Keep the code clean, well-commented in key places, and reasonably performant (avoid heavy operations every frame).

Implementation guidelines:
- Use a grid for pheromones (cell size 3–5 pixels).
- Ant movement should be continuous (not locked to grid).
- Add a small amount of random steering so ants don’t get stuck.
- Make sure ants can successfully find food and return home so trails form over time.
- Include a simple info text showing current status (running/paused, ant count, etc.).

- Output ONLY the complete HTML document.
- Start with `<!DOCTYPE html>` and end with `</html>`.
- Do not add explanations before or after the code.
- Do not wrap the code in markdown code blocks.
- The file must be ready to save as .html and open directly.

Now write the full single-file HTML ant colony simulator.