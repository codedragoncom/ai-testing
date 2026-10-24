<!--
Type: Implementation prompt
Target: WebGL2, GLSL ES 3.00, vanilla JavaScript
Output: One standalone HTML file
Related: None
-->

# LAVA — A Living Light
## Photorealistic Interactive WebGL2 Lava Lamp

You are an expert real-time graphics engineer specializing in WebGL2, GLSL ES 3.00, signed distance fields, raymarching, procedural materials, and physically inspired animation.

## YOUR MISSION

Create **LAVA — A Living Light**, a beautiful, interactive 3D lava lamp with translucent glass, glowing liquid wax, polished metal, realistic lighting, and graceful heat-driven motion.

The experience should resemble an expensive mid-century modern lava lamp photographed in a professional design studio.

Deliver exactly one complete, self-contained `index.html` file.

**The highest priority is believable wax behavior:** large soft blobs must rise, stretch, narrow into necks, separate into droplets, merge, cool, sink, and accumulate at the bottom.

The wax must appear viscous, heavy, organic, and continuous.

Avoid the appearance of independent rubber balls bouncing inside a glass container.

---

## 1. NON-NEGOTIABLE REQUIREMENTS

- Exactly one standalone HTML file.
- All HTML, CSS, JavaScript, and GLSL shaders embedded inline.
- Pure WebGL2. No Three.js, Babylon.js, gl-matrix, frameworks, imports, or CDNs.
- No downloaded images, fonts, textures, models, or other assets.
- No build process or server requirements.
- Must run directly when opened as a local HTML file in a modern WebGL2-capable browser.
- Use `#version 300 es` for all GLSL shaders.
- Use `precision highp float;` and suitable integer precision declarations.
- Include explicit shader compile, program link, and WebGL2 capability checks.
- Provide a clean error screen if WebGL2 initialization fails.
- Use `requestAnimationFrame` for rendering.
- All controls must change real application state.
- No unfinished shader functions, placeholder implementations, or omitted code.

The result must be functional immediately after the HTML file opens.

## 2. RENDERING ARCHITECTURE

Use a full-screen triangle or quad with a raymarching fragment shader.

The scene is constructed procedurally using signed distance functions.

Recommended architecture:

**JavaScript**
- WebGL2 initialization.
- Shader compilation.
- Camera controls.
- Thermal wax animation.
- Blob position and radius updates.
- UI controls.
- Rendering loop.
- Resize handling.

**GLSL**
- Camera ray generation.
- SDF evaluation.
- Wax metaball blending.
- Vessel intersection.
- Surface normal calculation.
- Glass shading.
- Wax lighting.
- Metal material.
- Studio background.
- Tone mapping.

Prefer updating a small array of metaball positions from JavaScript and passing them as shader uniforms.

Do not implement a full Navier–Stokes fluid solver. This project uses procedural, physically inspired thermal-convection animation with implicit wax surfaces.

Do not describe the metaball system as a physically exact fluid simulation.

## 3. SCENE COMPOSITION

Create a premium product-photography scene.

### Environment

- Warm charcoal or deep navy studio background.
- Subtle vertical background gradient.
- Dark matte tabletop.
- Soft contact shadow beneath the lamp.
- Large rectangular softbox reflection.
- Narrow vertical rim light.
- Gentle ambient fill.
- Subtle reflected light on the tabletop.

### Lamp Placement

- Centered horizontally.
- Occupies approximately 65–75% of the screen height.
- Slightly elevated camera angle.
- Visible metal base, full glass vessel, and upper cap.
- Clean silhouette.
- Enough surrounding space for the interface.

The default view should immediately resemble a photographic product render.

## 4. PROCEDURAL LAMP GEOMETRY

Construct the entire lamp with SDF functions.

### Glass Vessel

Create the iconic tapered lava-lamp silhouette:

- Wider lower section.
- Narrower upper section.
- Smooth conical sidewalls.
- Rounded transitions.
- Slight shoulder curvature.
- Clearly defined thick glass walls.

Use a capped cone SDF or a custom profile-of-revolution distance approximation.

The vessel must appear hollow, not solid glass.

Represent the inner and outer glass boundaries separately.

Ensure the wax remains inside the interior volume.

### Metal Base

Create a classic mid-century spun-metal base:

- Broad stable lower footprint.
- Tapered conical body.
- Rounded edge bevels.
- Narrow upper support.
- Brushed aluminum or satin stainless-steel appearance.

### Upper Cap

Add a matching metal cap:

- Tapered or softly domed silhouette.
- Thin polished rim.
- Subtle brushed finish.
- Correct physical alignment with the glass vessel.

### Construction Quality

Avoid visible intersections between the metal base, glass, and top cap.

Use smooth SDF transitions only where physically appropriate.

Do not blend metal geometry into the wax or glass.

## 5. WAX GEOMETRY — HIGHEST PRIORITY

The wax must be rendered as a continuous implicit surface.

### SDF Metaballs

Represent wax using approximately 8–16 animated blobs.

Each blob has:

- Position.
- Radius.
- Vertical stretch factor.
- Temperature.
- Movement phase.
- Velocity.

Combine neighboring spheres using polynomial smooth minimum.

Recommended formulation:

`h = max(k - abs(a - b), 0.0) / k`

`smin(a, b, k) = min(a, b) - h*h*k*0.25`

Use a stable smoothing parameter relative to blob size.

### Required Visual Behavior

The wax must form:

- Large bulbous masses.
- Narrow stretched necks.
- Teardrop shapes.
- Elongated rising columns.
- Small detached droplets.
- Merged larger blobs.
- Flattened wax resting on the bottom.
- Soft compression near the top.

Avoid uniform sphere sizes and identical motion.

### Surface Deformation

Add low-frequency, low-amplitude deformation to the wax SDF.

Use deterministic noise or a small number of sine-based spatial perturbations.

The deformation should:

- Produce organic surface asymmetry.
- Remain smooth.
- Change gradually with time.
- Preserve the overall volume impression.
- Never produce sharp spikes or noisy surfaces.

Do not rely on noise alone to create the wax motion.

The metaball centers must actually move and change shape.

## 6. STATEFUL THERMAL CONVECTION

Implement a lightweight thermal animation system in JavaScript.

This is not a full CFD simulation. It is a stateful approximation designed to reproduce the appearance of heat-driven wax circulation.

### Persistent Blob State

Each blob maintains:

- Position.
- Velocity.
- Temperature.
- Radius.
- Stretch.
- Motion state.

Suggested states:

- RESTING
- HEATING
- RISING
- COOLING
- FALLING
- MERGING

Use continuous numerical integration for position and velocity. States should influence the forces rather than simply teleport blobs between predetermined positions.

### Heating

Wax near the bottom gradually warms.

Heating depends on:

- Lamp power.
- Temperature slider.
- Distance from the heating element.
- Time spent near the base.

Heat should accumulate gradually rather than instantly.

### Rising

As wax temperature increases:

- Effective buoyancy increases.
- Vertical velocity increases.
- Blob stretches along the direction of travel.
- Larger blobs may form smaller droplets.
- Nearby blobs may merge.

Use velocity damping to keep motion slow and viscous.

### Cooling

Near the top:

- Heat gradually dissipates.
- Buoyancy weakens.
- Movement slows.
- Blobs round out or flatten near the upper boundary.
- Cooled wax begins to sink.

### Falling

Cooling blobs should:

- Fall more slowly than they rise.
- Drift gently toward the outer walls.
- Remain inside the vessel.
- Stretch modestly during downward movement.
- Settle near the base.

### Splitting

Implement believable splitting through controlled blob state changes.

For example:

- A large, hot, elongated blob produces a smaller child blob.
- The child emerges near the parent's upper region.
- Their SDF surfaces remain connected briefly through smooth blending.
- Separation increases gradually until the child detaches.

Do not suddenly spawn a distant sphere.

### Merging

When two blobs approach:

- Allow smooth implicit surface union.
- Gradually reconcile their velocities and temperatures.
- Merge small droplets into larger resting masses where practical.

The visual transition must remain continuous.

### Confinement

Calculate approximate vessel interior radius as a function of height.

Ensure:

- Blob centers remain within the liquid volume.
- Blob radius is accounted for when constraining movement.
- Wax does not visibly pass through glass.
- Wax does not intersect the metal cap.
- Extreme temperatures do not destabilize motion.

## 7. POWER SWITCH AND COOLING

The Power switch must affect both lighting and wax behavior.

### Power On

- Warm internal bulb illuminates.
- Temperature increases gradually.
- Wax near the base begins moving.
- Convective circulation develops.
- Heated blobs rise and cool.

### Power Off

- Bulb fades to darkness.
- Heat input stops.
- Existing blob temperatures decrease gradually.
- Buoyancy declines.
- Rising wax slows.
- Blobs eventually return to the bottom.
- Wax settles into one or more merged masses.

Do not instantly reset the wax positions when power turns off.

### Restart

When power returns:

- The bulb warms gradually.
- Resting wax begins softening.
- Heating eventually produces renewed upward movement.

Use gradual temperature transitions so the system feels physical.

## 8. TEMPERATURE CONTROL

Provide a slider from 0% to 100%.

Temperature must change actual motion and appearance.

### 0–25% — Cool

- Mostly stationary wax.
- Large masses near the bottom.
- Minimal convection.
- Weak glow.
- Slow deformation.

### 25–70% — Normal

- Graceful rising blobs.
- Smooth merging.
- Moderate stretching.
- Slow cooling and sinking.
- Classic lava-lamp behavior.

### 70–100% — Hot

- Stronger buoyancy.
- Faster ascent.
- More elongated wax.
- Increased droplet separation.
- Smaller secondary blobs.
- Brighter internal lighting.

Avoid abrupt behavior changes at slider thresholds.

Use smooth interpolation between operating regimes.

## 9. PHOTOREALISTIC WAX MATERIAL

Implement glossy, translucent wax shading.

### Material Requirements

- Rich saturated wax color.
- Smooth reflective surface.
- Fresnel edge highlights.
- Soft internal light transmission.
- Warm subsurface glow.
- Darker thickness-dependent regions.
- Glossy but not metallic specular response.

### Lighting

Use:

1. Warm point light near the bottom.
2. Large cool-white studio softbox.
3. Narrow rim light.
4. Low-intensity ambient fill.

### Subsurface Appearance

Approximate subsurface scattering using:

- Backlit surface illumination.
- Thickness-inspired absorption.
- View-dependent rim transmission.
- Soft gradient from illuminated to shadowed regions.

Thin blob edges should glow slightly.

Large thick masses should appear darker in their centers.

Avoid making wax look like opaque plastic or transparent glass.

## 10. GLASS AND LIQUID SHADING

Render the bottle as transparent glass containing a clear or colored liquid medium.

### Glass Requirements

- Strong Fresnel reflections near grazing angles.
- Visible softbox reflections.
- Mild optical distortion.
- Subtle edge thickness.
- Faint liquid tint.
- Convincing transparency.
- Readable wax shapes through the container.

### Refraction

Implement a practical refractive approximation using:

- View direction.
- Surface normal.
- Snell-inspired ray bending.
- Approximate entry and exit interfaces.
- Thickness-dependent color absorption.

Exact multi-bounce path tracing is not required.

A stable approximation that maintains a convincing glass appearance is preferable to mathematically ambitious but broken optics.

### Important Rendering Rule

The glass must not hide the wax.

When compositing glass and wax, account for visible interior surfaces and correct approximate ordering.

A single nearest-hit SDF render that stops at the outer glass surface is not acceptable.

Use a bounded multi-stage ray evaluation, separate transparent-surface handling, or equivalent technique.

### Liquid Color

Apply Beer–Lambert-inspired absorption:

`transmittance = exp(-absorptionColor * distance)`

Use lighter color along thin viewing paths and richer color where the ray travels farther through liquid.

Avoid excessively dark liquid that obscures the wax.

## 11. BRUSHED METAL MATERIAL

Give the base and upper cap a premium machined finish.

### Appearance

- Satin metallic reflections.
- Circumferential brushing.
- Subtle fine-grain variation.
- Bright but narrow edge highlights.
- Smooth curved reflections.
- Slight roughness variation.

### Procedural Texture

Generate directional noise from cylindrical or angular coordinates.

Keep grain subtle and stable in object space.

Do not animate metal texture with time.

Use a simplified anisotropic highlight model if necessary.

The result should resemble brushed aluminum, not gray plastic.

## 12. LIGHTING AND TONE MAPPING

Use physically inspired real-time lighting approximations.

### Studio Setup

- Warm internal bulb.
- Cool-white rectangular softbox.
- Narrow vertical strip reflection.
- Soft rim fill.
- Subdued environment illumination.

### Materials

Keep metal, glass, and wax visually distinct.

Use appropriate roughness and Fresnel responses.

### Tone Mapping

Apply an ACES-inspired or neutral tone-mapping approximation.

Convert linear output to sRGB correctly.

Avoid:

- Excessive bloom.
- Clipped orange or yellow highlights.
- Flat ambient lighting.
- Overexposed wax.
- Crushed black metal surfaces.

The lamp should look like a high-end photographed physical object.

## 13. CAMERA INTERACTION

Implement an orbit camera using JavaScript.

### Controls

- Left mouse drag: orbit.
- Mouse wheel: zoom.
- One-finger touch: orbit.
- Two-finger pinch: zoom.
- Smooth camera damping.
- Reset View button.

### Camera Limits

- Restrict vertical angle to prevent awkward upside-down views.
- Clamp zoom to practical distances.
- Prevent the camera from passing through the lamp.
- Keep the lamp centered and framed.

### Default View

Use a slightly elevated three-quarter camera angle.

Make the glass body and wax clearly visible immediately.

## 14. USER INTERFACE

Create a minimal modern HUD that feels like a premium design product.

### Title

**LAVA**

Subtitle: *A Living Light*

Use elegant system font stacks.

No external font loading.

### Controls

Include:

| Control | Behavior |
|---|---|
| Power | Turns heating and bulb illumination on/off |
| Temperature | 0–100% heating intensity |
| Colorway | Cycles through wax/liquid color combinations |
| Pause | Freezes wax animation |
| Reset | Restores initial thermal and blob state |
| Reset View | Restores camera position |
| FPS | Displays measured frame rate |

### Colorways

Include at least:

**Classic**
- Orange-red wax.
- Purple liquid.

**Electric**
- Cyan wax.
- Deep blue liquid.

**Atomic**
- Neon green wax.
- Mostly clear liquid.

**Sunset**
- Amber wax.
- Rose-colored liquid.

**Dream**
- Pink wax.
- Soft violet liquid.

Use carefully chosen colors with readable contrast.

### UI Style

- Small floating glass-like controls.
- Subtle blur where supported.
- Rounded corners.
- Fine borders.
- Restrained shadows.
- Clear slider values.
- Visible focus states.
- Responsive mobile layout.

The controls should not dominate the scene.

## 15. OPTIONAL QUALITY CONTROLS

If practical, add a compact rendering-quality selector:

- Low.
- Medium.
- High.

Quality may adjust:

- Raymarch step count.
- Shadow sampling.
- Secondary reflection approximations.
- Render resolution.

Changing quality must not reset the wax animation.

Do not implement this feature at the expense of core functionality.

## 16. RAYMARCHING IMPLEMENTATION

### Primary Rays

Generate a normalized world-space ray from the camera.

Intersect relevant scene bounds before detailed marching.

### SDF Evaluation

Separate SDF functions for:

- Metal base.
- Metal cap.
- Glass vessel.
- Wax metaballs.
- Tabletop.

Use meaningful material identifiers.

### Surface Normals

Calculate normals using central differences or tetrahedral sampling.

Choose epsilon based on world scale.

Avoid noisy normals and unstable gradients.

### Marching

Use bounded iterations.

Recommended starting limits:

- 72–96 primary steps.
- Approximately 6–12 shadow steps.
- Reduced work for rays missing the lamp.
- Early exit when surfaces or background are resolved.

### Acceleration

Use analytical ray-sphere or bounding-cylinder intersection before evaluating expensive wax geometry.

Avoid evaluating every metaball for every pixel when the ray cannot intersect the wax volume.

### Blending

Use smooth minimum only for wax-to-wax blending.

Do not smooth-union wax with glass, metal, or the tabletop.

### Transparency

Handle glass and wax intersection ordering explicitly.

Do not terminate all rays at the first glass surface.

Use practical bounded ray tracing rather than unrestricted recursive reflections and refractions.

## 17. PERFORMANCE AND TIMING

Target smooth 60 FPS on capable desktop GPUs.

Performance defaults:

- Device pixel ratio capped at 1.5.
- Approximately 10–12 active wax blobs.
- 72–96 raymarch steps.
- Limited secondary lighting samples.
- Fixed or bounded animation timestep.

### JavaScript

- Reuse typed arrays.
- Avoid per-frame object allocation where practical.
- Cache shader uniform locations.
- Avoid repeated shader compilation.
- Avoid creating buffers every frame.
- Use a bounded delta time.
- Handle browser-tab suspension safely.

### Shader

- Minimize repeated SDF evaluations.
- Use conservative bounding volumes.
- Avoid excessively expensive nested raymarch loops.
- Avoid dynamic complexity proportional to screen size beyond normal fragment processing.
- Reduce shadow costs when unnecessary.
- Use early termination where possible.

### Resize

- Match drawing buffer size to CSS size times capped device pixel ratio.
- Recompute camera aspect ratio.
- Update viewport.
- Maintain correct pointer interaction.

## 18. ROBUSTNESS AND ERROR HANDLING

Include checks for:

- Missing WebGL2 context.
- Shader compilation failure.
- Program linking failure.
- WebGL context loss.
- Invalid uniform state.
- Invalid canvas dimensions.
- Extreme frame delta time.

If initialization fails, display a useful error message inside the page.

Do not leave an empty canvas.

Preserve normal application behavior after resizing.

## 19. IMPLEMENTATION PRIORITIES

Follow this order when complexity creates tradeoffs.

**Priority 1 — Working WebGL2 Scene**

A complete raymarched lamp with correct geometry, camera, and materials.

**Priority 2 — Believable Wax Geometry**

Smooth metaballs form realistic continuous shapes.

**Priority 3 — Stateful Wax Motion**

Heating, rising, stretching, cooling, falling, merging, and settling work.

**Priority 4 — Glass and Lighting**

Glass transparency, refraction approximation, wax transmission, and metal reflections.

**Priority 5 — Working Controls**

Power, temperature, colorways, pause, reset, camera, and FPS.

**Priority 6 — Product-Quality Presentation**

Studio lighting, shadows, typography, and visual refinements.

**Priority 7 — Optional Enhancements**

Rendering quality presets and additional polish.

Never remove real wax behavior to add decorative effects.

## 20. INTERNAL VALIDATION

Before producing the final answer, inspect the implementation.

### GLSL

- Every shader starts with `#version 300 es`.
- Correct precision declarations.
- Correct GLSL ES 3.00 syntax.
- All required uniforms declared.
- All varyings match.
- No invalid vector operations.
- No unsupported GLSL functions.
- No undefined SDF helpers.

### Geometry

- Lamp proportions are coherent.
- The glass has visible thickness.
- Wax remains within the vessel.
- Metal and glass do not intersect incorrectly.
- Smooth unions apply only between wax components.

### Animation

- Temperature changes motion.
- Power-off leads to gradual cooling.
- Wax returns toward the base when unpowered.
- Blobs remain bounded.
- Splitting does not cause abrupt teleportation.
- Motion remains continuous after pausing and resuming.

### Rendering

- Glass transparency does not hide the wax.
- Wax shading uses actual surface normals.
- Background renders correctly.
- Camera rays match camera orientation.
- Resize does not distort the lamp.
- Raymarching cannot enter an unbounded loop.

### UI

- Every control has a working event handler.
- FPS measures actual frame timing.
- Reset restores a coherent initial state.
- Colorways update material uniforms.
- Camera orbit and zoom function.
- UI interactions do not accidentally rotate the camera.

Fix identifiable errors before output.

Do not claim to have performed browser testing unless browser execution tools were actually available.

## 21. OUTPUT REQUIREMENTS

Return only the complete executable HTML document.

Begin with:

`<!DOCTYPE html>`

End with:

`</html>`

Do not include Markdown code fences, explanations, installation instructions, TODO comments, pseudocode, placeholder functions, or omitted shader logic.

Every JavaScript function, GLSL shader, rendering pass, control, and initialization routine must be fully implemented.

Do not truncate the output.

## FINAL SUCCESS STANDARD

The user opens `index.html` and sees an elegant lava lamp standing on a softly illuminated tabletop.

A warm bulb glows inside its polished metal base.

Translucent wax slowly warms, stretches upward into organic columns, separates into rounded droplets, cools near the top, and sinks back down.

The glass bends and reflects light while keeping the wax clearly visible.

Turning off the lamp gradually darkens the bulb and causes the wax to cool and collect at the bottom.

Changing the temperature produces noticeably different convection behavior.

The user can orbit the camera, zoom, and explore different colorways.

The result should feel like a small, believable physical object brought to life through real-time graphics — not a screensaver made of animated spheres.