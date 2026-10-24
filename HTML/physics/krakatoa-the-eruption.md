<!--
Type: Implementation prompt
Target: WebGL2, GLSL ES 3.00, vanilla JavaScript
Output: One standalone HTML file
Related: None
-->

# KRAKATOA — Real-Time Volcanic Eruption Simulator

You are a senior graphics engineer specializing in WebGL2, GLSL ES 3.00, GPU fluid simulation, procedural terrain, volumetric rendering, and interactive 3D visualization.

## YOUR MISSION

Build **KRAKATOA**, a visually spectacular, real-time 3D volcanic eruption simulation inspired by the 1883 Krakatoa eruption.

Create a cinematic volcanic island surrounded by a dark ocean, with a massive continuously evolving ash plume, glowing lava, atmospheric lighting, and interactive camera controls.

The result should feel like a premium real-time graphics demonstration, not a static scene or a looping particle animation.

Deliver one complete, standalone `index.html` file.

**The highest priorities are correct WebGL2 execution, believable continuous smoke motion, a convincing volcanic landscape, and visually impressive rendering.**

## 1. HARD REQUIREMENTS

- Exactly one HTML file containing all HTML, CSS, JavaScript, and GLSL shaders.
- Use raw WebGL2 without Three.js, Babylon.js, frameworks, imports, CDNs, or external dependencies.
- No build process, backend, image downloads, or network access.
- Must work when opened directly in a modern browser with WebGL2 support.
- Use `#version 300 es` as the first line of every GLSL shader.
- Declare suitable precision for floating-point and integer operations.
- Compile and link all shaders with explicit error checking.
- Verify framebuffer completeness before using render targets.
- Use `requestAnimationFrame` for rendering.
- Display a clean fallback if WebGL2 or required GPU capabilities are unavailable.
- Never leave the user with a blank screen or a permanently frozen loading screen.

All simulation and rendering code must be included.

Do not substitute precomputed video, animated images, or unrelated particle effects for the required smoke simulation.

## 2. TECHNICAL ARCHITECTURE

Use a modular rendering architecture within the single HTML file.

Separate responsibilities into:

1. WebGL2 initialization and capability detection.
2. Shader compilation and program management.
3. Geometry and framebuffer management.
4. Procedural terrain generation.
5. GPU smoke simulation.
6. Volumetric plume rendering.
7. Lava flow simulation and shading.
8. Atmosphere and lighting.
9. Camera controls.
10. User interface.
11. Animation timing and resource cleanup.

### Recommended Approach

Use:

- Procedural triangle meshes for the volcanic terrain.
- GPU fragment-shader simulation for fluid velocity and density.
- Ping-pong floating-point textures for simulation fields.
- Camera-facing raymarched volume for ash and smoke rendering.
- Procedural shader effects for hot lava and cooling crust.
- Lightweight particles for volcanic sparks and falling embers.

Particles may supplement the eruption but must not replace the simulated smoke field.

Prioritize a stable implementation over unnecessary algorithmic complexity.

## 3. VOLCANIC ISLAND AND CALDERA

Create a recognizable volcanic island surrounded by dark ocean.

### Terrain Geometry

Generate a procedural terrain mesh using polar coordinates or a displaced grid.

Features:

- Broad island base rising from the ocean.
- Steep volcanic cone.
- Large central caldera.
- Uneven crater rim.
- Radial ridges and erosion channels.
- Broken rock formations.
- Rough basaltic slopes.

The caldera must have a genuinely depressed interior rather than a flat mountain peak with a dark circle painted on it.

### Procedural Height Function

Combine:

- Radial volcano profile.
- Smooth crater depression.
- Multi-frequency deterministic noise.
- Directional erosion.
- Small-scale rocky displacement.

Use a smooth, bounded height function that avoids cracks and invalid geometry.

Calculate terrain normals from the generated mesh or height gradients.

### Surface Material

Render volcanic rock with:

- Nearly black basalt.
- Charcoal and dark gray ash.
- Rough, fractured surfaces.
- Subtle brown mineral variation.
- Emissive orange fissures.
- Dark cooled lava channels.
- Glowing cracks concentrated near the crater.

Lighting should reveal surface detail without making the mountain look artificially bright.

## 4. GPU FLUID SMOKE SIMULATION

**This is the most important technical subsystem.**

Implement an actual evolving Eulerian fluid field using WebGL2 fragment shader passes.

Prefer a two-dimensional simulation grid mapped into the vertical plane of the eruption.

Use the simulation to control the movement and distribution of volcanic ash through a three-dimensional-looking rendered plume.

This is a performance-conscious 2.5D approximation, not a claim of full 3D Navier–Stokes simulation.

### Simulation Fields

Maintain GPU textures for:

- Velocity.
- Smoke density.
- Temperature.
- Pressure.
- Divergence.

Use floating-point textures where supported.

Use ping-pong render targets whenever a pass reads and writes the same type of field.

Never sample a texture while that same texture is attached as the active render target.

### Required Simulation Passes

Implement the following steps in order:

**1. Vent Injection**

Inject smoke density, heat, and upward momentum at the caldera opening.

Injection should be continuous and slightly irregular.

Introduce controlled turbulent disturbances near the source.

**2. Velocity Advection**

Advect the velocity field using semi-Lagrangian backtracing.

Sample the previous velocity field at the backtraced location.

**3. Buoyancy**

Apply upward acceleration based on temperature and smoke density.

Hot smoke should rise forcefully before slowing and spreading.

**4. Vorticity Confinement**

Compute local curl and apply a small corrective force to preserve swirling motion.

This helps avoid an overly smooth plume.

**5. Divergence**

Calculate velocity divergence from neighboring grid samples.

**6. Pressure Solve**

Use an iterative Jacobi pressure solver.

Default to 8 iterations, with an adjustable range appropriate for performance.

**7. Projection**

Subtract the pressure gradient from velocity to reduce divergence.

**8. Density and Temperature Advection**

Advect smoke density and temperature through the corrected velocity field.

**9. Dissipation and Cooling**

Gradually dissipate density and cool the rising plume.

Apply appropriate boundary conditions to prevent unstable accumulation.

### Stability

- Use a fixed or bounded timestep.
- Clamp extreme velocity values.
- Use stable boundary sampling.
- Avoid NaN and infinite values.
- Prevent feedback loops between framebuffer attachments.
- Reinitialize simulation resources safely when resolution changes.

The plume should evolve continuously rather than resetting or visibly repeating.

## 5. PLUME SHAPE AND MOTION

The eruption must visually resemble a powerful volcanic explosion.

### Required Appearance

- Dense dark ash emerging from the crater.
- Narrow, forceful lower eruption column.
- Rising turbulent smoke.
- Widening upper plume.
- Mushroom-like rolling structures.
- Large billowing ash masses.
- Small eddies and curls.
- Orange illumination near the volcanic vent.
- Dark gray upper cloud masses.

### 2.5D-to-3D Reconstruction

Convert the simulated 2D smoke field into a three-dimensional-looking volume.

Use a camera-facing volume or simplified 3D density reconstruction.

A practical approach:

- Use simulated height and horizontal displacement from the fluid field.
- Revolve or expand smoke density around a central vertical eruption axis.
- Modulate density with animated procedural 3D noise.
- Introduce height-dependent spreading.
- Add low-frequency asymmetric distortion to avoid a perfectly circular plume.

**Important:** The simulation must drive the motion. Procedural noise should add detail, not replace actual smoke advection.

Keep the plume continuous across height and time.

Avoid obvious billboard edges and repeated smoke patterns.

## 6. VOLUMETRIC SMOKE RENDERING

Implement a raymarched volumetric smoke shader using GLSL ES 3.00.

### Raymarching

For each relevant pixel:

1. Construct a world-space camera ray.
2. Intersect the ray with the plume bounding volume.
3. March through the volume at bounded intervals.
4. Sample reconstructed smoke density.
5. Calculate local extinction.
6. Accumulate transmittance and scattered light.
7. Composite the result with the background.

### Absorption

Use Beer–Lambert attenuation:

`T = exp(-density * absorption * stepLength)`

Stop marching early when accumulated transmittance becomes very low.

### Lighting

Approximate single scattering using:

- Warm light emitted from the caldera.
- Cool ambient skylight.
- Directional storm lighting.
- Density-dependent shadowing.
- Limited secondary light sampling for self-shadowing.

Use an efficient directional approximation rather than an expensive full global illumination system.

### Visual Quality

The smoke must have:

- Soft volumetric edges.
- Dense opaque interiors.
- Rich gray tonal variation.
- Warm glow near the vent.
- Cooler upper regions.
- Visible internal shadowing.
- Irregular turbulent silhouette.

Do not render the plume as a stack of obviously separated transparent planes.

### Performance

Use:

- Limited raymarch steps.
- Early ray termination.
- Adaptive render resolution where practical.
- Low-cost density lookup.
- Bounded self-shadow sampling.

Balance detail against stable frame rate.

## 7. DYNAMIC LAVA FLOWS

Create lava streams moving from the caldera down the volcanic slopes.

The lava should look like hot viscous molten rock, not glowing orange paint.

### Flow Paths

Generate several irregular downhill channels.

Use local terrain slope to influence movement.

Flow should:

- Begin near the crater rim.
- Follow descending terrain.
- Accelerate on steep slopes.
- Slow on shallow terrain.
- Branch into smaller streams.
- Progressively cool as it moves downhill.

A simplified surface flow model is acceptable if it produces coherent downhill movement.

### Thermal Shading

Use a temperature-inspired color ramp:

- White-yellow for extreme heat.
- Bright orange for active liquid lava.
- Deep red for cooling lava.
- Dark brown and black for crust.

### Crusting

Add:

- Black fractured crust along stream edges.
- Glowing molten cracks within cooled regions.
- Dark floating crust patterns.
- Variable cooling along stream length.
- Bright exposed lava near the source.

The lava must exhibit visible directional flow.

Use advection or animated flow coordinates tied to terrain direction, not simple global UV scrolling.

### Caldera Glow

Create intense light near the eruption source.

The caldera should illuminate nearby rock and the underside of the smoke plume.

## 8. ERUPTION PARTICLES

Add a modest number of secondary particles.

Examples:

- Glowing volcanic embers.
- Small ejected rocks.
- Falling sparks.
- Brief hot fragments.

Particles should:

- Originate near the vent.
- Follow gravity.
- Have varied velocities.
- Fade or cool over time.
- Remain visually subordinate to the smoke.

Use a compact GPU or CPU particle system.

Avoid allocating large numbers of JavaScript objects every frame.

## 9. OCEAN AND ATMOSPHERE

### Ocean

Surround the island with a dark, stormy ocean.

Use a low-cost shader featuring:

- Deep blue-gray water.
- Subtle animated wave normals.
- Broad distorted reflections.
- Small highlights.
- Dark horizon blending.

A procedural plane is sufficient.

### Sky

Create a dramatic volcanic atmosphere.

Features:

- Dark storm clouds.
- Muted blue-gray upper atmosphere.
- Warm volcanic glow near the horizon.
- Subtle distance haze.
- Strong silhouette contrast.

Use procedural gradients and noise instead of external skybox images.

### Lighting

Use three primary light sources:

1. Cold directional storm light.
2. Warm volcanic emission.
3. Low-intensity ambient fill.

Keep the scene cinematic but readable.

Avoid excessive bloom that obscures terrain or plume detail.

## 10. CAMERA CONTROLS

Implement pure JavaScript orbit controls.

### Mouse

- Left drag: orbit.
- Wheel: zoom.
- Smooth camera damping.
- Sensible angle restrictions.

### Touch

- One finger: orbit.
- Two fingers: zoom.

### Camera Defaults

Start with a dramatic three-quarter view that clearly shows:

- The island silhouette.
- The volcanic caldera.
- Flowing lava.
- The rising smoke plume.
- The surrounding ocean.

Avoid positioning the camera inside geometry.

### Responsive Rendering

- Resize canvas with the browser window.
- Correctly update aspect ratio and projection.
- Cap device pixel ratio at 1.5.
- Maintain proper pointer mapping.
- Prevent unintended touch scrolling on the canvas.

## 11. CONTROL INTERFACE

Create a compact, dark, premium control overlay.

### Masthead

**KRAKATOA**

Subtitle: *The Earth Unleashed*

Use modern system typography, subtle translucent panels, and warm volcanic accent colors.

### Required Controls

| Control | Behavior |
|---|---|
| Eruption Intensity | Adjust vent impulse and material injection |
| Smoke Density | Change injected ash density |
| Turbulence | Adjust vorticity and disturbance strength |
| Plume Height | Control vertical expansion and buoyancy |
| Lava Flow | Adjust lava movement rate |
| Simulation Speed | Change simulated time rate |
| Smoke Quality | Adjust raymarch sampling quality |
| Pause / Resume | Freeze or resume simulation |
| Reset | Reinitialize eruption |
| Reset Camera | Restore initial view |

All controls must affect actual application behavior.

### Live Statistics

Display:

- FPS.
- Smoke simulation resolution.
- Active particle count.
- Current smoke quality.
- Running or paused state.

Update these values from actual application state.

### Interface Requirements

- Collapsible control panel.
- Readable labels.
- Numerical slider values.
- Responsive mobile layout.
- Visible focus states.
- Keyboard support.
- Keyboard shortcuts for pause and reset.
- Respect reduced-motion preferences.

The simulation must remain the visual focus.

## 12. WEBGL2 CORRECTNESS

### Shader Compatibility

Every shader must start with:

`#version 300 es`

Use suitable declarations such as:

`precision highp float;`
`precision highp int;`

Use GLSL ES 3.00 syntax consistently.

Use `in`, `out`, `texture()`, and explicit fragment outputs.

Do not mix WebGL1 and WebGL2 shader syntax.

### Floating-Point Framebuffers

Check for:

`EXT_color_buffer_float`

Use supported renderable formats such as `RGBA16F` where available.

For half-float data textures, use the correct format, type, and texture parameters.

If float linear filtering is unavailable, use NEAREST filtering or implement manual interpolation in shaders.

Do not assume that half-float rendering automatically guarantees linear texture filtering.

### Ping-Pong Resources

Build reusable helpers for:

- Texture creation.
- Framebuffer creation.
- Ping-pong swapping.
- Shader program creation.
- Uniform management.
- Full-screen simulation passes.

Always set the viewport to match the active render target.

Ensure every render pass writes to the intended framebuffer.

Restore the appropriate viewport for screen rendering.

### Context Errors

Check:

- Shader compile results.
- Program link results.
- Framebuffer status.
- Required extensions.
- Texture dimensions.
- Valid uniform locations where relevant.

Handle WebGL context loss and restoration gracefully.

## 13. PERFORMANCE

Target approximately 60 FPS on capable desktop hardware.

Do not guarantee 60 FPS on every GPU.

Suggested initial defaults:

- Smoke simulation: 128 × 256.
- Pressure solve: 8 iterations.
- Volumetric raymarching: 48–64 steps.
- Limited self-shadow samples.
- Modest particle count.
- Device pixel ratio capped at 1.5.

Use adjustable quality settings.

Performance requirements:

- Avoid GPU readback during normal rendering.
- Reuse textures and buffers.
- Reuse JavaScript typed arrays.
- Avoid shader recompilation during animation.
- Avoid excessive draw calls.
- Keep simulation timestep bounded.
- Avoid unlimited frame catch-up after tab suspension.
- Skip unnecessary work when paused.
- Scale expensive volumetric effects according to quality.

Performance should remain stable during camera movement.

## 14. IMPLEMENTATION PRIORITIES

If complexity requires tradeoffs, use this strict order.

**Priority 1: Working WebGL2 application**

The page initializes correctly and renders a complete 3D scene.

**Priority 2: Convincing volcanic terrain**

The island and crater have real three-dimensional structure.

**Priority 3: Working GPU smoke simulation**

Smoke density and velocity evolve through real advection, buoyancy, divergence, pressure, and projection passes.

**Priority 4: Volumetric eruption**

The simulated smoke creates a continuously evolving, visually convincing eruption column.

**Priority 5: Lava**

Visible downhill movement, heat variation, and cooling crust.

**Priority 6: Atmosphere**

Ocean, storm lighting, shadows, haze, and glow.

**Priority 7: Interactivity**

Camera controls, simulation settings, and responsive interface.

**Priority 8: Optional polish**

Additional particles, enhanced reflections, and further visual refinements.

Never sacrifice a working simulation to implement a decorative effect.

## 15. INTERNAL VALIDATION

Before outputting the implementation, inspect your code for the following failures.

### JavaScript

- Undefined functions.
- Uninitialized resources.
- Incorrect matrix multiplication.
- Invalid buffer sizes.
- Incorrect attribute layouts.
- Missing resize handlers.
- Incorrect animation loop state.

### GLSL

- Missing version declaration.
- Incorrect precision declarations.
- Mismatched uniforms.
- Unsupported shader syntax.
- Incorrect texture coordinates.
- Invalid vector or matrix operations.
- Inconsistent color spaces.

### Simulation

- Reading and writing the same texture simultaneously.
- Pressure solver using stale or incorrect textures.
- Divergence computed from the wrong velocity field.
- Missing ping-pong swaps.
- Unbounded time steps.
- Broken boundary conditions.
- Density injection not reaching the renderer.
- Simulation values producing NaN.

### Rendering

- Invalid framebuffer attachments.
- Incorrect render target dimensions.
- Incorrect depth testing.
- Incorrect alpha blending.
- Volumetric smoke appearing behind foreground geometry.
- Incorrect camera rays.
- Raymarching outside the plume volume.
- Missing transparency compositing.

### Controls

- Sliders not connected to real simulation parameters.
- Reset not clearing simulation textures.
- Pause stopping rendering instead of only simulation advancement.
- Camera controls conflicting with UI.
- Incorrect pointer coordinates after resizing.

Fix any identified problems before completing the file.

Do not claim to have executed or tested the code unless browser execution tools were actually available.

## 16. OUTPUT FORMAT

Return only the complete working HTML document.

Begin with:

`<!DOCTYPE html>`

End with:

`</html>`

Do not include:

- Markdown fences.
- Explanations.
- Pseudocode.
- TODO comments.
- Missing shader bodies.
- Placeholder functions.
- Instructions to install dependencies.
- References to omitted code.

All HTML, CSS, JavaScript, shader sources, geometry generation, simulation passes, rendering logic, UI controls, and initialization must be fully implemented.

Do not truncate the answer.

## FINAL SUCCESS CRITERION

When the HTML file opens, the user sees a dramatic volcanic island surrounded by a dark ocean.

A thick, turbulent ash column rises continuously from a glowing caldera.

The smoke billows, spreads, twists, and evolves according to an actual GPU fluid field.

Bright lava streams creep down the crater slopes while dark crust forms along their edges.

The scene is illuminated by storm lighting and intense volcanic emission.

The user can orbit the island, zoom, adjust eruption parameters, and watch the volcano continue evolving in real time.

The result should resemble a compelling interactive volcanic eruption visualization, not a static mountain with scrolling textures and decorative particles.