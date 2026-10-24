<!--
Type: Implementation prompt
Target: WebGPU, WGSL, vanilla JavaScript
Output: One standalone HTML file
Related: None
-->

# JELLY DICE — Interactive WebGPU Soft-Body Simulation

You are an expert JavaScript graphics engineer specializing in WebGPU, WGSL shaders, real-time physics, and interactive 3D applications.

## YOUR MISSION

Create **Jelly Dice**, a beautiful interactive 3D simulation of translucent gummy-candy dice that deform, stretch, wobble, collide, roll, and settle like real soft objects.

Deliver a single, complete, self-contained `index.html` file.

The application should resemble a polished interactive product demonstration, not a technical prototype.

**The most important requirement is genuine soft-body deformation.** The dice must behave like elastic jelly rather than rigid cubes using decorative vertex animations.

## 1. NON-NEGOTIABLE REQUIREMENTS

- One standalone HTML file containing all HTML, CSS, JavaScript, and WGSL shaders.
- Raw WebGPU only for rendering. No Three.js, Babylon.js, frameworks, imports, CDNs, or external dependencies.
- No build tools, asset downloads, network access, or backend.
- Run in a modern WebGPU-enabled browser within a secure context.
- If WebGPU is unavailable, show a polished explanation card instead of a broken application.
- Implement real physical vertex movement, collision handling, and elastic recovery.
- Never substitute procedural wobble shaders for the physics simulation.
- No fake controls, unfinished functions, placeholder shaders, or missing resources.
- Use requestAnimationFrame for rendering and a fixed-timestep physics loop.
- Target smooth 60 FPS on capable desktop hardware, with lower quality settings for slower devices.

WebGPU security note: browsers may restrict WebGPU when an HTML file is opened directly through `file://`. The application must remain self-contained, but it may require a local static server to provide a secure context. Do not attempt to bypass browser security restrictions.

## 2. IMPLEMENTATION STRATEGY

Favor a reliable hybrid architecture:

**CPU:**
- Soft-body physics.
- XPBD constraint solving.
- Collision detection and response.
- Pointer grabbing and poking.
- Sleep management.
- Dice face detection.

**GPU:**
- Surface rendering.
- Vertex transformation.
- Smooth lighting.
- Translucent candy shading.
- Shadows.
- Antialiasing.

This avoids requiring complicated compute-shader physics while retaining GPU-accelerated rendering.

Keep simulation meshes relatively small so JavaScript can solve the constraints efficiently.

Use a structured architecture with clearly separated functions or classes for physics, rendering, input, UI, and initialization.

## 3. SOFT-BODY PHYSICS — HIGHEST PRIORITY

Represent each die as an actual deformable volumetric object.

### Geometry

- Construct a rounded cube using a regular 3D lattice.
- Use approximately 4–6 nodes along each axis, adjusted for performance.
- Divide lattice cells into tetrahedra using a consistent decomposition.
- Maintain rest positions, current positions, previous positions, velocities, inverse masses, and constraint data.
- Generate a smooth surface mesh attached to the deformable volume.
- Preserve the shape and rounded appearance of the dice.

### XPBD Solver

Implement Extended Position-Based Dynamics with a fixed simulation timestep.

Required physics:

1. **Prediction:** Integrate external forces and predict new node positions.
2. **Elasticity:** Apply elastic constraints based on tetrahedral rest geometry.
3. **Volume preservation:** Preserve tetrahedral volume to resist crushing and excessive expansion.
4. **Strain limiting:** Prevent uncontrolled stretching and collapse.
5. **Collision correction:** Resolve floor, rim, and other-die penetration.
6. **Velocity update:** Recover node velocities from corrected positions.
7. **Internal damping:** Reduce oscillation while preserving overall translation and rotation.

Use XPBD compliance to provide stable, adjustable softness.

For elasticity, prefer a practical tetrahedral strain or shape-matching constraint implementation. Use co-rotational shape matching if it can be implemented correctly.

Do not pretend to implement sophisticated finite-element elasticity with a simple sine-wave displacement.

### Volume and Inversion

- Calculate signed tetrahedral volumes using cross and dot products.
- Use meaningful positive minimum-volume constraints.
- Detect collapsed or inverted tetrahedra.
- Apply corrective constraints to restore valid orientation.
- Cap extreme correction magnitudes.
- Keep the solver stable during rapid grabbing and heavy impacts.

Approximate volume conservation is acceptable if it produces convincing deformation and reliable recovery.

### Jelly Behavior

The physical response should exhibit:

- Local squashing at contact points.
- Elastic stretching under dragging.
- Visible delayed recovery.
- Small secondary oscillations.
- Shape preservation over time.
- Strong resistance to complete collapse.
- Rapid settling after heavy impacts.

The motion should look heavy, soft, and slightly sticky, rather than rubbery or weightless.

## 4. COLLISIONS AND FRICTION

Implement physical collision response rather than simply preventing the object's center from crossing a boundary.

### Floor

- Flat horizontal plate.
- Node-level floor contact.
- Penetration correction.
- Static friction for resting contact.
- Kinetic friction during sliding.
- Damped impact response.

A resting die should stay still unless deliberately disturbed.

### Plate Rim

Add a shallow raised boundary around the plate.

Dice must visibly contact the rim rather than pass through it.

A simplified rectangular or circular boundary is acceptable if the visual plate matches its physical collision geometry.

### Die-to-Die Contact

Support 1–5 dice simultaneously.

Implement an efficient approximate collision approach:

- Broad-phase bounding sphere or AABB checks.
- Narrow-phase surface-node versus another die's collision volume or a conservative rounded-box proxy.
- Position correction to prevent deep interpenetration.
- Contact friction and damping.
- Wake nearby sleeping dice when a collision is expected.

Use approximate contacts if necessary, but avoid frequent clipping.

### Settling

Include a sleep system:

- Measure aggregate linear and angular movement.
- Require low motion for multiple consecutive steps.
- Freeze settled objects to prevent perpetual jitter.
- Wake sleeping objects on interaction or meaningful contact.

Impact damping should depend on collision severity.

Heavy drops should squash noticeably, lose energy, wobble briefly, and settle.

## 5. DIRECT INTERACTION

### Grabbing and Stretching

Users can click or touch a die and drag it.

Implement ray casting from the camera into the scene.

Select the nearest visible die surface and associate the grab with nearby physical nodes.

Use a compliant spring-like attachment:

- Gentle movement stretches the selected region.
- The bottom can remain held by static floor friction.
- Stronger movement lifts and translates the die.
- Pulling creates localized deformation.
- Releasing allows elastic recovery and wobble.

Clamp the maximum applied grab force and pointer-derived velocity.

Do not launch or violently spin the die when released.

### Poking

A quick click or tap applies a short localized inward impulse.

The surface should dent, then recover with a small damped oscillation.

A poke must affect real simulation nodes.

### Camera

- Drag empty space to orbit the camera.
- Mouse wheel to zoom.
- One-finger dragging over empty space orbits.
- Two-finger pinch zooms.
- Clamp camera angles and zoom distances.
- Keep the dice visible.
- Smoothly center the camera on the dice arrangement when no die is held.

Use damped camera interpolation, but respect reduced-motion preferences.

## 6. ROLLING THE DICE

Provide a prominent **Roll** button.

Also support Space and R when keyboard focus is not inside a control.

Allow selection of 1–5 dice.

When rolling:

1. Wake all active dice.
2. Reset excessive residual deformation if needed.
3. Lift them slightly above the plate.
4. Apply varied linear and angular impulses through the physical nodes.
5. Let gravity, collisions, and deformation determine the result.
6. Wait for all dice to settle.
7. Determine each upward-facing value.
8. Display the individual results and total.

### Correct Dice Faces

Each die must have six correctly numbered faces with opposite sides adding to seven.

Use standard die numbering:

- 1 opposite 6.
- 2 opposite 5.
- 3 opposite 4.

Determine the result from the actual orientation of the deformed die.

Calculate a best-fit rigid orientation from the die's rest positions and current node positions, or use deformed face normals for a practical approximation.

Compare each numbered face normal against world up.

Do not randomly generate a displayed number independently of the simulated orientation.

### Cocked Dice

If a die settles at an ambiguous angle:

- Detect that the top face is not sufficiently aligned with world up.
- Apply a small physical nudge.
- Retry at most twice.
- If still ambiguous, display a clear "Cocked" status rather than inventing a result.

## 7. WEBGPU RENDERING

Create a polished 3D scene using raw WebGPU and WGSL.

### Scene

- Warm, softly lit studio background.
- Large matte tabletop.
- Frosted-glass serving plate.
- Rounded translucent candy dice.
- Soft shadows.
- Subtle color bleeding.
- A premium product-photography composition.

### Candy Material

Support six selectable candy colors:

- Emerald.
- Amethyst.
- Amber.
- Sapphire.
- Raspberry.
- Lime.

Use physically inspired translucent shading:

- Colored absorption through the candy.
- Darker interior and richer color in thicker regions.
- Bright translucent edges.
- Fresnel reflection.
- Glossy specular highlights.
- Soft internal lighting.
- Smooth deformation-dependent normals.

Prioritize stable and believable translucency over complicated but incorrect optical effects.

For a first implementation, approximate refraction using a view-dependent screen-space distortion or surface-thickness approximation.

If implementing multiple rendering passes, ensure dependencies and framebuffer usage are valid.

Never label simple alpha blending as physically correct refraction.

### Deformed Surface Normals

The rendered geometry must update from actual simulated node positions.

Recompute surface normals from deformed triangle geometry, or transform normals using a valid deformation gradient.

Do not rely on unchanged rest-pose normals.

Update GPU vertex data efficiently.

### Pips

Each die must have six numbered faces using standard pip arrangements.

- Pips should be ivory or warm cream.
- Position them directly on the deforming candy surface.
- Make the markings follow the deformation.
- Add subtle dimpling and shallow depth cues.
- Preserve readability during rolling.

Use embedded mesh geometry or shader-based markings derived from undeformed face coordinates.

Avoid floating labels or flat screen overlays masquerading as physical pips.

### Lighting

Use a procedural studio lighting setup:

- Large, warm overhead key light.
- Narrow bright rim light from behind.
- Soft cool fill.
- Gentle ambient illumination.
- Glossy highlights on the candy.

Use a filmic or neutral tone-mapping approximation with correct linear-to-display color handling.

### Shadows

Provide grounded, soft shadows beneath the dice.

Prefer a small shadow-map pass or a stable contact-shadow approximation.

Shadows should follow the actual location of the dice.

Use lighter, subtly color-tinted shadows to suggest translucent candy.

### Antialiasing

Use MSAA where supported and appropriate, or another practical antialiasing method.

Avoid excessive GPU memory usage.

## 8. UI AND VISUAL DESIGN

Build a minimal editorial interface around the 3D scene.

### Masthead

Top-left:

**JELLY DICE**

Subtitle: *A little physics. A lot of wobble.*

Use an elegant serif headline and clean sans-serif body typography.

Because external fonts are prohibited, use attractive local system font stacks rather than downloading fonts.

### Bottom Control Bar

Include:

- Six candy-color swatches.
- Dice count selector, 1–5.
- Large Roll button.
- Individual rolled values.
- Total score.

Make the controls visually balanced and readable.

### Settings Panel

Provide a compact collapsible panel containing:

| Setting | Function |
|---|---|
| Softness | Adjust XPBD elastic compliance |
| Wobble Decay | Control internal deformation damping |
| Slow Motion | Toggle 0.25× simulation speed |
| Show Mesh | Display deformable lattice or wireframe |
| Sound | Toggle procedurally generated impact audio |
| Pause | Stop physics advancement |
| Reset Dice | Restore initial dice state |
| Reset View | Restore camera |
| FPS | Display measured performance |

All settings must work.

### Sound

Use the Web Audio API without external files.

When enabled, generate a short, soft, low-frequency impact sound.

Vary volume and tone with impact intensity.

Default sound to off.

Create or resume the audio context only after a user gesture.

### Responsive Design

- Adapt control layout to narrow screens.
- Support touch and pinch gestures.
- Avoid blocking the simulation with oversized panels.
- Support visible keyboard focus.
- Use accessible labels and semantic controls.
- Honor `prefers-reduced-motion`.

## 9. SIMULATION TIMING AND PERFORMANCE

Use a fixed physics timestep, preferably 1/120 or 1/60 second.

Recommended sequence per timestep:

1. Accumulate and clamp elapsed time.
2. Process queued pointer interaction.
3. Apply gravity and external forces.
4. Predict node positions.
5. Solve elastic and volume constraints.
6. Solve floor and rim collisions.
7. Solve die-to-die collisions.
8. Repeat constraint iterations as needed.
9. Update node velocities.
10. Apply deformation-only damping.
11. Evaluate sleep states.
12. Update render geometry.

Use a bounded number of substeps to avoid the spiral of death.

Performance guidelines:

- Favor small tetrahedral lattices.
- Precompute tetrahedral connectivity and rest-state data.
- Reuse typed arrays.
- Avoid allocations inside solver iterations.
- Cache matrix and vector operations where practical.
- Avoid recreating WebGPU buffers and pipelines every frame.
- Use queue.writeBuffer to update changing geometry.
- Limit GPU synchronization.
- Lower simulation resolution or rendering quality when appropriate.

The simulation should remain responsive with five active dice.

## 10. ERROR HANDLING

Handle:

- Missing `navigator.gpu`.
- WebGPU adapter unavailable.
- Device acquisition failure.
- Shader or pipeline creation errors.
- Canvas configuration errors.
- GPU device loss.
- Invalid simulation state.
- Browser resizing.
- Pointer cancellation.

For unsupported WebGPU, show a centered fallback card with a concise explanation and instructions for using a compatible browser.

Do not leave the user staring at a blank screen.

## 11. IMPLEMENTATION PRIORITIES

Use this exact priority order:

**Priority 1 — Working application**

The HTML initializes, creates the WebGPU renderer, and draws the complete scene.

**Priority 2 — Genuine deformable dice**

Dice have actual volumetric simulation nodes and visible deformation.

**Priority 3 — Stable contacts**

Dice land, squash, collide, roll, and settle without exploding or penetrating the floor.

**Priority 4 — User interaction**

Grabbing, stretching, poking, and camera movement work.

**Priority 5 — Rolling and scoring**

Dice roll physically, stop reliably, and display orientation-derived values.

**Priority 6 — Visual quality**

Candy materials, lighting, pips, studio composition, and shadows.

**Priority 7 — Polish**

Audio, diagnostics, responsive layout, and visual refinements.

Do not replace a higher-priority requirement with a lower-priority visual trick.

## 12. INTERNAL VALIDATION

Before output, review the code for correctness.

Verify:

- All JavaScript identifiers are defined.
- All WGSL shader entry points and bindings match their pipelines.
- All required WebGPU resources are initialized.
- Vertex buffer sizes and layouts match shader expectations.
- Render targets use compatible formats and sample counts.
- Every tetrahedron starts with valid rest geometry.
- XPBD corrections use stable numerical calculations.
- Volume constraints handle inversion or near-zero volume.
- Grab forces remain bounded.
- Floor contact does not generate energy.
- Dice sleep without permanent jitter.
- Collision handling wakes relevant neighbors.
- Face numbering and displayed scores match die orientation.
- Pips follow surface deformation.
- Every UI control has a working handler.
- Rendering works after canvas resizing.
- GPU initialization failure displays fallback UI.

If a feature cannot be implemented correctly, use the simplest technically honest approximation that preserves its intended behavior.

Do not silently substitute unrelated animation or random values for physical simulation.

Do not claim to have run browser tests or captured screenshots unless browser execution and screenshots were actually available.

## 13. FINAL OUTPUT FORMAT

Output only one complete HTML document.

Begin with:

`<!DOCTYPE html>`

End with:

`</html>`

Do not include Markdown code fences.

Do not include explanations, introductions, commentary, TODO sections, pseudocode, omitted functions, or placeholder implementations.

Ensure all HTML, CSS, JavaScript, and WGSL source code is included.

The file must be complete and runnable.

## FINAL SUCCESS STANDARD

A user opens Jelly Dice in a WebGPU-capable environment and sees beautiful gummy dice resting on a frosted plate.

Dragging stretches the actual geometry. Poking creates a local dent. Dropping creates convincing squash and wobble. Multiple dice collide and settle. Rolling produces real orientation-based scores.

The result feels like interacting with tiny pieces of soft candy, not watching an animation of rigid plastic cubes.