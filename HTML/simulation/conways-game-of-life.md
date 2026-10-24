<!--
Type: Implementation prompt
Target: HTML Canvas, vanilla JavaScript
Output: One standalone HTML file
Related: None
-->

You are an expert frontend developer. Write a complete, self-contained single-file HTML document that implements Conway's Game of Life.

Requirements (must follow exactly):
- One single HTML file only. No external libraries or CDNs.
- Use a <canvas> for rendering.
- Pure vanilla JavaScript.
- Must work immediately when opened in a browser.

Features:
- Grid of cells (e.g. 80×60 or larger).
- Standard Game of Life rules (Birth: 3 neighbors, Survive: 2 or 3 neighbors).
- Cells are black/white or colored when alive.
- Start/Pause and Reset buttons.
- Button to randomize the grid.
- Optional: speed control and ability to click cells to toggle them.
- Use requestAnimationFrame for the loop.
- Draw grid lines or clear cells cleanly.

Output rules:
- Output ONLY the complete HTML document.
- Start with <!DOCTYPE html> and end with </html>.
- Do not add explanations or markdown code blocks.