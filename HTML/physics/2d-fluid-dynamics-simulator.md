<!--
Type: Implementation prompt
Target: HTML Canvas; WebGL2 preferred, implementation approach flexible
Output: One standalone HTML file
Related: None
-->

You are an expert graphics programmer, numerical simulation engineer, and frontend developer specializing in real-time interactive physics demonstrations.

## OBJECTIVE

Build a polished, visually impressive, interactive **2D fluid dynamics simulator** that runs entirely within a single standalone `index.html` file.

The result must be a real fluid simulation, not a particle animation pretending to be fluid.

Treat this as a production-quality programming challenge. The finished application will be evaluated on correctness, visual quality, numerical stability, responsiveness, interaction, performance, and completeness.

## 1. HARD CONSTRAINTS

- Deliver exactly one complete `index.html` document.
- Embed all HTML, CSS, JavaScript, GLSL shaders, and visual resources.
- No external libraries, frameworks, CDNs, imports, network requests, build tools, or servers.
- Must run when opened directly using `file://` in a modern desktop browser.
- Prefer WebGL2 for GPU-accelerated simulation, but choose another approach if it produces a more reliable implementation.
- Do not use placeholder implementations, fake physics, nonfunctional controls, or unfinished features.
- Do not depend on browser features requiring special permissions.
- Output must be complete, executable code.

## 2. FLUID PHYSICS

Implement a continuous two-dimensional Eulerian fluid simulation using a grid-based velocity field.

Prefer a stable-fluid or semi-Lagrangian Navier–Stokes approach with these components:

1. **Velocity advection:** Transport velocity through the fluid using the current velocity field.
2. **Dye advection:** Transport RGB dye through the same velocity field.
3. **Pressure projection:** Solve a pressure Poisson equation and subtract the pressure gradient to reduce velocity divergence.
4. **Incompressibility:** Keep the flow approximately divergence-free.
5. **Vorticity confinement:** Reinforce swirling structures and prevent motion from becoming visually lifeless.
6. **Viscosity:** Provide adjustable momentum diffusion, implemented with a stable numerical approximation.
7. **Dissipation:** Independently control velocity damping and dye fading.
8. **Boundary conditions:** Prevent invalid sampling and handle flow along simulation edges coherently.

Recommended simulation order:

1. Apply queued user forces and dye injections.
2. Advect velocity.
3. Apply viscosity.
4. Apply vorticity confinement.
5. Compute divergence.
6. Iteratively solve pressure.
7. Subtract pressure gradient.
8. Advect dye using the projected velocity.
9. Apply dissipation and render.

You may adjust the order when mathematically justified.

Use ping-pong buffers where needed. Never read from and write to the same GPU texture during one simulation pass.

Prioritize stable behavior over complex numerical techniques.

## 3. INTERACTIVE FLUID CONTROLS

The user must be able to interact with the simulation using mouse, pen, or touch input.

Implement:

- Click and drag to inject colored dye.
- Drag motion injects momentum into the velocity field.
- Drag direction determines force direction.
- Drag speed influences force magnitude.
- Continuous pointer movement produces smooth, connected fluid strokes.
- Interpolate between pointer samples to avoid gaps during fast movement.
- Support pointer capture and proper pointer release.
- Handle pointer cancellation and moving outside the canvas safely.
- Prevent unwanted touch scrolling during simulation interaction.
- Correctly map pointer coordinates to simulation coordinates, accounting for canvas dimensions, display scaling, and vertical-axis conventions.

The interaction should feel immediate and fluid, including during rapid directional changes.

Do not replace fluid motion with independent animated particles.

## 4. LIVE CONTROL PANEL

Create a compact, attractive floating control panel with clearly labeled controls.

Required controls:

| Control | Behavior |
|---|---|
| Pause / Resume | Stops or resumes simulation advancement |
| Reset | Restores the initial simulation state |
| Clear Dye | Removes visible dye without necessarily clearing velocity |
| Resolution | Changes the simulation grid dimensions |
| Simulation Speed | Controls the rate of simulated time |
| Viscosity | Changes momentum diffusion |
| Pressure Iterations | Adjusts projection solver quality |
| Vorticity | Changes swirling intensity |
| Velocity Dissipation | Changes how quickly momentum decays |
| Dye Dissipation | Changes how quickly colors fade |
| Interaction Force | Controls injected momentum |
| Interaction Radius | Controls brush size |
| Dye Color | Selects injection color |

Additional requirements:

- Every control must update actual simulation behavior.
- Display current numerical values where useful.
- Use sensible minimums, maximums, and defaults.
- Resolution changes must safely recreate affected simulation resources.
- Provide responsive sliders, buttons, and color selectors.
- Keep controls readable over the fluid background.
- Make the control panel collapsible to maximize simulation space.
- Support keyboard shortcuts for pause and reset if practical.

Do not implement decorative controls that have no effect.

## 5. VISUALIZATION MODES

Provide a selector that switches between multiple live rendering modes without restarting the simulation.

Required modes:

1. **Dye:** Beautiful RGB dye rendering with smooth gradients and natural mixing.
2. **Velocity:** Visualize velocity magnitude or direction using an informative color mapping.
3. **Pressure:** Visualize positive and negative pressure values.
4. **Divergence:** Show local expansion and compression, useful for evaluating incompressibility.
5. **Vorticity:** Visualize clockwise and counterclockwise rotation.

Diagnostic visualizations must use actual simulation fields or correctly computed derived fields.

Use meaningful color mappings, including signed coloring where appropriate.

Do not generate unrelated decorative images and label them as diagnostics.

## 6. VISUAL DESIGN

Create an application that feels like a polished interactive science exhibit rather than a developer prototype.

Design direction:

- Full-screen or near-full-screen fluid canvas.
- Dark, sophisticated background.
- Luminous, richly colored dye.
- Smooth gradients and layered swirling forms.
- Subtle glow where computationally inexpensive.
- Clean glass-like control panel with readable typography.
- Restrained borders, shadows, and accent colors.
- Clear active states and interaction feedback.
- Excellent contrast and spacing.
- Responsive layout for desktop and smaller browser windows.

The fluid should be visually engaging immediately after startup.

Initialize with several aesthetically pleasing colored dye concentrations and nonzero swirling motion, or create a short deterministic startup disturbance.

The startup effect must use the real simulation system rather than a separate visual animation.

Avoid constant forced motion unless it is an explicitly implemented, controllable feature.

## 7. PERFORMANCE AND NUMERICAL STABILITY

Target smooth interaction and approximately 60 FPS on capable desktop hardware, without assuming that every system can achieve it.

Implement sensible performance safeguards:

- Use `requestAnimationFrame`.
- Separate simulation resolution from display resolution.
- Cap device pixel ratio to avoid excessive rendering costs.
- Reuse GPU textures, framebuffers, shaders, and buffers.
- Avoid unnecessary allocations inside the animation loop.
- Avoid reading GPU pixels back to the CPU every frame.
- Clamp extreme injected forces.
- Clamp or subdivide excessive simulation time steps.
- Use a fixed or bounded time step and prevent unbounded catch-up work after tab suspension.
- Avoid NaN or infinite values entering simulation calculations.
- Minimize shader compilation and GPU resource recreation.
- Release old GPU resources during resolution changes.
- Keep the application responsive during rapid pointer movement.

Where supported, use suitable GPU texture formats and extensions for floating-point simulation.

Check framebuffer completeness and shader compile/link status.

If the necessary GPU capabilities are unavailable, show a clear error message or provide a genuinely functional fallback. Do not silently display a nonworking simulation.

## 8. RESIZE AND INPUT HANDLING

The application must:

- Resize its drawing surface when the browser window changes.
- Maintain correct aspect ratio and pointer mapping.
- Avoid stretching or distorted fluid motion.
- Handle high-DPI displays.
- Work with mouse and touch through Pointer Events.
- Handle WebGL context loss gracefully when WebGL is used.
- Restore or reinitialize GPU resources when the context returns.
- Keep the controls accessible at common viewport sizes.

## 9. PERFORMANCE OVERLAY

Display a small live status overlay containing:

- Frames per second.
- Simulation grid dimensions.
- Current visualization mode.
- Running or paused status.
- Rendering backend.
- Pressure solver iteration count.

Metrics must reflect real application state.

FPS should be calculated from measured frame timing, preferably smoothed to avoid distracting fluctuations.

## 10. IMPLEMENTATION QUALITY

Organize the application into clear internal components or functions covering:

- Initialization.
- GPU capability detection.
- Shader compilation.
- Texture and framebuffer management.
- Fluid simulation passes.
- Pointer input and force injection.
- Dye management.
- Visualization.
- UI event handling.
- Resize handling.
- Animation timing.
- Error handling.
- Resource cleanup.

Keep everything in the single HTML file.

Write maintainable JavaScript with meaningful variable and function names.

Prefer correct, straightforward implementations over clever abstractions.

Ensure shader uniforms, attributes, framebuffer attachments, texture formats, and texture bindings are consistent.

Avoid relying on undefined variables, implicit global state, incompatible GLSL versions, or unavailable browser APIs.

## 11. PRIORITY ORDER

When implementation complexity creates a tradeoff, use this strict priority:

1. The HTML opens and runs successfully.
2. The fluid solver behaves correctly and remains stable.
3. Dye and momentum injection work smoothly.
4. Every required control works.
5. Visualization modes display real simulation data.
6. Performance remains responsive.
7. The interface looks polished.
8. Optional visual enhancements are added.

Never sacrifice simulation correctness to add unnecessary visual effects.

If optional enhancements threaten stability, omit them.

## 12. INTERNAL VALIDATION

Before producing your answer, review the implementation against these checks:

**Execution**
- Does the file open directly in a browser?
- Are all required shaders and resources embedded?
- Are all referenced functions and variables defined?
- Are the shaders syntactically compatible with the selected WebGL version?

**Simulation**
- Are velocity and dye actually advected?
- Is pressure solved iteratively?
- Is the pressure gradient subtracted?
- Is divergence reduced by the projection step?
- Does vorticity influence the velocity field?
- Does viscosity actually diffuse momentum?
- Are dissipation controls independent?

**Interaction**
- Does dragging inject dye and momentum?
- Does stroke interpolation prevent visible gaps?
- Does pointer mapping remain correct after resizing?
- Does interaction continue smoothly at high pointer speeds?

**Interface**
- Does each control change the correct parameter?
- Can visualization modes switch without resetting?
- Does clear dye preserve velocity?
- Do pause, reset, and resolution changes work?
- Does the performance overlay display meaningful values?

**Robustness**
- Are GPU capability checks implemented?
- Are framebuffer operations valid?
- Are simulation time steps bounded?
- Are resources recreated safely?
- Is unsupported hardware handled clearly?

Resolve any issues you identify before finalizing the code.

Do not claim to have executed or tested the application unless you actually have.

## 13. OUTPUT REQUIREMENTS

Return ONLY the complete working HTML document.

Start with `<!DOCTYPE html>` and end with `</html>`.

Do not include Markdown fences, introductions, explanations, implementation plans, or text outside the HTML.

Do not replace implementation sections with comments such as "TODO", "implement later", "remaining code", or equivalent placeholders.

Produce the entire file, including every shader, handler, render pass, UI element, and initialization routine.

**SUCCESS CRITERION:** A user can save your response as `index.html`, open it directly in Chrome or another capable modern browser, immediately see colorful moving fluid, interact with it by dragging, adjust working physics controls, inspect real diagnostic modes, and use the application without installing anything.