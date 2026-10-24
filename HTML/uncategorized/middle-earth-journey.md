<!--
Type: Implementation prompt
Target: Three.js, OrbitControls, vanilla JavaScript
Output: middle-earth-journey.html
Related: middle-earth-build-plan.md
-->

# MIDDLE-EARTH — A Journey Through Legend
## Procedural 3D Diorama & Cinematic Camera Experience

**Output filename: `middle-earth-journey.html`**

You are an expert Three.js creative technologist, procedural environment artist, graphics engineer, and cinematic camera programmer.

Your specialties include procedural landscape generation, stylized architectural modeling, reusable geometry systems, animation, lighting, and deterministic rendering.

## YOUR MISSION

Build **MIDDLE-EARTH — A Journey Through Legend**, a beautiful miniature 3D fantasy-world diorama inspired by the geography, architecture, and atmosphere of J.R.R. Tolkien's Middle-earth.

The application must feature a coherent landscape, 24 recognizable landmarks, animated environmental effects, a dynamic day/night cycle, and a complete scripted cinematic journey lasting exactly **3 minutes and 47 seconds (227 seconds)**.

The visual style should resemble a handcrafted museum-quality miniature: detailed terrain, carefully arranged architecture, atmospheric lighting, and gently animated environmental effects.

Deliver one complete, working HTML file named `middle-earth-journey.html`.

**Highest priorities:**
1. All 24 landmarks exist and are visually distinct.
2. Geography and landmark placement remain consistent.
3. The 227-second cinematic journey works from start to finish.
4. Every animation supports deterministic time seeking.
5. The scene looks cohesive, detailed, and intentionally designed.

---

## 1. TECHNICAL REQUIREMENTS

- Exactly one HTML document containing application HTML, CSS, and JavaScript.
- Use Three.js for rendering.
- Vanilla JavaScript for procedural generation and application logic.
- No React, Vue, frameworks, npm installation, bundlers, or build process.
- No external models, images, textures, fonts, or audio.
- Generate terrain, architecture, vegetation, water, and particles procedurally.
- The only permitted network dependencies are Three.js and compatible OrbitControls from pinned CDN URLs.
- Use compatible versions from the same Three.js release.
- Do not use an unversioned `latest` dependency.
- Use WebGLRenderer with antialiasing.
- Enable soft shadows where practical.
- Use appropriate tone mapping and sRGB output for the selected Three.js version.
- Cap device pixel ratio at 1.5.
- Handle resizing automatically.
- Show a clear error message if WebGL or dependency loading fails.

For a single-file application, fixed Three.js ES module imports are acceptable. The application requires network access to load those modules but must not require a local build process.

## 2. IMPLEMENTATION ARCHITECTURE

Organize the application into clear internal systems:

1. Renderer and scene initialization.
2. Deterministic seeded random number generation.
3. Terrain and elevation generation.
4. Biome coloration.
5. Water and river creation.
6. Reusable architectural primitives.
7. Landmark registry and placement.
8. Vegetation generation.
9. Environmental particle systems.
10. Day/night lighting.
11. Scripted cinematic camera.
12. Deterministic timeline controller.
13. Free-camera interaction.
14. HUD and playback controls.

Prefer clear reusable functions over long repeated blocks of geometry creation.

Suggested reusable helpers:

- `createTower()`
- `createCastle()`
- `createFortress()`
- `createVillage()`
- `createMountain()`
- `createRock()`
- `createTree()`
- `createBridge()`
- `createWall()`
- `createRuins()`
- `createWaterfall()`
- `createParticleSystem()`
- `getTerrainHeight()`
- `placeOnTerrain()`

Each helper must create actual visible geometry.

Do not implement helpers as empty placeholders.

## 3. WORLD COORDINATE SYSTEM

Use a consistent geographic coordinate system.

Recommended:

- World X: west to east.
- World Z: north to south.
- World Y: elevation.
- Map bounds: X from -100 to +100.
- Map bounds: Z from -100 to +100.

Use a shared `getTerrainHeight(x, z)` function for terrain creation and landmark placement.

All map positions must be consistent with the same geographic conventions.

### Geographic Layout

**Northwest**
- The Grey Havens.
- The Shire.
- Bree.
- Weathertop.

**North-Central**
- Rivendell.
- Misty Mountains.
- Moria.

**Central**
- Lothlórien.
- Fangorn.
- Isengard.
- Argonath.

**West-Central / South**
- Helm's Deep.
- Edoras.
- Amon Hen.

**Southeast**
- Gondor.
- Osgiliath.
- Minas Tirith.
- Minas Morgul.
- Mordor.
- Mount Doom.
- Barad-dûr.
- Black Gate.
- Cirith Ungol.
- Dead Marshes.

**Northeast**
- Mirkwood.
- Dol Guldur.
- Dale.
- Erebor.

Preserve recognizable relative geography even though the world is compressed into a miniature diorama.

Do not randomly distribute landmark locations.

## 4. PROCEDURAL TERRAIN

Create one continuous terrain mesh.

Recommended:

- PlaneGeometry approximately 200 × 200 world units.
- Approximately 160–200 subdivisions per axis.
- Rotate into the horizontal XZ plane.
- Displace vertices using deterministic elevation functions.
- Recompute vertex normals.

### Terrain Height

Combine:

- Low-frequency continental shape.
- Multi-octave deterministic noise.
- Regional mountain masks.
- Ridgeline functions.
- Localized volcanic elevation.
- Valley depression functions.

### Regional Elevation

**Eriador and Shire**
- Low rolling hills.
- Gentle slopes.
- Broad green plains.

**Misty Mountains**
- Strong north-south ridgeline.
- Jagged peaks.
- Rocky slopes.
- Snow-covered summits.

**Rohan**
- Open rolling grassland.
- Broad valleys.
- Isolated hills.

**Gondor**
- Rocky highlands.
- River valleys.
- Elevated defensive terrain.

**Mordor**
- Encircling jagged mountain barriers.
- Dark interior plateau.
- Scorched valleys.
- Distinct volcanic cone at Mount Doom.

**Northeast**
- Dense Mirkwood woodland.
- Isolated Erebor mountain.
- Iron Hills-inspired elevated ridges.

Avoid abrupt height discontinuities.

### Deterministic Noise

Implement a seeded noise or hash function directly in JavaScript.

The same seed and world coordinates must always produce the same terrain.

Do not use uncontrolled `Math.random()` for procedural generation.

## 5. BIOME COLORATION

Use vertex colors or efficient procedural materials.

Map color to:

- Elevation.
- Slope.
- Geographic region.
- Noise variation.

### Palette

- Shire: rich rolling green.
- Eriador: muted grass and moss.
- Lothlórien: golden yellow-green.
- Rohan: dry grass and olive.
- Misty Mountains: blue-gray stone and snow.
- Gondor: gray stone and muted green.
- Mirkwood: deep forest green.
- Mordor: ash black, charcoal, scorched brown, volcanic red.

Blend boundaries smoothly.

Avoid sharply rectangular biome regions.

Use subtle material variation so terrain does not look flat.

## 6. DIORAMA PRESENTATION

The world should feel like a premium handcrafted miniature.

### Diorama Base

- Raised terrain slab.
- Visible beveled or layered sides.
- Dark stone or bronze base.
- Soft shadow beneath the diorama.
- Attractive floating-island or museum-table presentation.

### Environment

- Atmospheric gradient background.
- Gentle distance haze.
- Controlled studio lighting.
- Subtle ambient occlusion approximation.
- Soft terrain shadows.

### Scale

Architecture and geography may use intentionally exaggerated proportions for readability.

Landmarks should be recognizable from the cinematic camera.

Avoid realistic geographic scale that makes buildings microscopic.

## 7. DATA-DRIVEN LANDMARK GENERATION

Define a registry containing all 24 landmarks.

Every entry must include:

- Unique ID.
- Display name.
- Region.
- X coordinate.
- Z coordinate.
- Scale.
- Visual archetype.
- Camera focus height.
- Optional VFX configuration.

Use a consistent structure such as:

`{ id, name, region, x, z, scale, archetype, focusHeight }`

Iterate over the registry to construct every landmark.

Each landmark must produce a THREE.Group containing actual geometry.

Use landmark-specific construction functions or reusable archetype builders.

**Do not create 24 copies of the same generic tower or castle.**

## 8. REQUIRED LANDMARKS

Implement all 24 landmarks below.

### 1. The Grey Havens

- Pale stone coastal architecture.
- Dock structures.
- White arches.
- Small harbor buildings.
- Coastal placement.

### 2. Hobbiton

- Rounded grassy hillocks.
- Circular hobbit doors.
- Small windows.
- Chimneys.
- Tiny garden plots.
- Warm welcoming colors.

### 3. Bree and The Prancing Pony

- Clustered timber-framed buildings.
- Steep gabled roofs.
- Larger central inn.
- Small village square.

### 4. Weathertop

- Elevated rocky hill.
- Ruined circular stone ring.
- Broken columns.
- Ancient weathered masonry.

### 5. Rivendell

- Elegant pale stone pavilions.
- Tall slender columns.
- Layered terraces.
- Decorative bridges.
- Waterfalls.
- Autumn-colored trees.

### 6. Moria West Gate

- Sheer mountain wall.
- Dark recessed arched entrance.
- Massive stone framing.
- Subtle carved geometric details.

### 7. Lothlórien

- Tall silver-barked trees.
- Golden foliage.
- Elevated circular talan platforms.
- Narrow connecting bridges.
- Gentle magical illumination.

### 8. The Argonath

- Two monumental stone statues.
- Humanoid silhouettes with heads and torsos.
- Raised arms.
- River passage between them.

### 9. Amon Hen

- Steep hill.
- Small ruined summit structure.
- Circular Seat of Seeing.
- Weathered stone steps.

### 10. Fangorn Forest

- Dense ancient forest.
- Thick dark trunks.
- Twisted branches.
- Irregular heavy canopy.
- Deep shadowed undergrowth.

### 11. Isengard

- Circular fortified boundary.
- Central Orthanc tower.
- Four dark angular spines.
- Sharp geometric silhouette.

### 12. Helm's Deep

- Mountain gorge.
- Curved defensive wall.
- Main Hornburg fortress.
- Battlements.
- Gate and ramparts.

### 13. Edoras

- Elevated green hill.
- Golden Hall of Meduseld.
- Long timber building.
- Steep golden roof.
- Small surrounding settlement.

### 14. Minas Tirith

- Seven visually distinct concentric defensive tiers.
- White stone walls.
- Multiple stacked levels.
- Central upper citadel.
- Protruding rocky prow.
- Strong vertical silhouette.

### 15. Osgiliath

- Ruined stone city.
- Broken towers.
- Damaged river bridges.
- Partial arches.
- Scattered battlements.

### 16. Minas Morgul

- Pale fortress with sickly green illumination.
- Narrow spires.
- Tall central tower.
- Rotating crown-inspired upper geometry.

### 17. The Black Gate

- Massive dark fortified gate.
- Two intimidating towers.
- Iron-like vertical teeth.
- Mountain-pass placement.

### 18. Mount Doom

- Large volcanic cone.
- Open crater.
- Bright molten interior.
- Lava streams.
- Ash plume.
- Rising glowing embers.

### 19. Barad-dûr

- Towering black citadel.
- Layered jagged spikes.
- Narrow central silhouette.
- Fiery Eye above the summit.
- Animated red-orange glow.

### 20. The Dead Marshes

- Low muddy terrain.
- Scattered stagnant pools.
- Sparse dead vegetation.
- Floating ghostly lights.
- Thin drifting mist.

### 21. Erebor

- Dominant solitary mountain.
- Tall steep rocky slopes.
- Snow or pale stone highlights.
- Carved dwarven entrance.

### 22. Dale

- Compact walled city.
- Multi-story buildings.
- Small towers.
- Colorful pitched roofs.
- Situated near Erebor.

### 23. Mirkwood and Dol Guldur

- Dense dark forest.
- Uneven heavy canopy.
- Central ruined hill fortress.
- Broken stone towers.
- Sinister silhouette.

### 24. Cirith Ungol

- Narrow jagged mountain pass.
- Tall isolated dark watchtower.
- Multiple stacked defensive sections.
- Harsh surrounding cliffs.

## 9. LANDMARK IDENTITY AND QUALITY

Each landmark must be recognizable through silhouette and composition.

Use:

- Distinct proportions.
- Different architectural structures.
- Appropriate material colors.
- Regional vegetation.
- Landmark-specific lighting.
- Small environmental details.

Use shared low-cost primitives where appropriate, but vary their construction meaningfully.

Do not rely only on changing colors to distinguish landmarks.

### Placement

Before placing a landmark:

1. Read the registry coordinates.
2. Sample terrain height.
3. Adjust the local foundation where needed.
4. Position the landmark above the terrain.
5. Align or level the construction appropriately.
6. Ensure the landmark is visible and not buried.

Large buildings may require a flattened local foundation zone.

## 10. PROCEDURAL VEGETATION

Generate forests, trees, and grass clusters.

### Tree Types

Create several reusable procedural tree archetypes:

- Rounded deciduous trees.
- Tall conifers.
- Golden Lothlórien trees.
- Dark twisted Fangorn trees.
- Gnarled Mirkwood trees.

Use InstancedMesh for repeated vegetation where practical.

### Placement

Respect regional vegetation density.

Keep dense trees away from:

- Major structures.
- Rivers.
- Sand or stone pathways.
- Volcanic terrain.
- Camera sightlines where visibility matters.

Use seeded randomness for placement.

## 11. WATER SYSTEM

### Great Sea

Generate a western ocean surface.

Use:

- Deep blue material.
- Gentle animated waves.
- Soft specular highlights.
- Atmospheric shoreline blending.

### River Anduin

Construct a winding river corridor passing through appropriate central and southern regions.

Use a spline-defined path.

Create a ribbon mesh following the river.

The river should relate visually to the Argonath and Osgiliath.

### Water Animation

Animate:

- Subtle wave displacement.
- Surface highlights.
- Flowing directional texture coordinates.
- Small waterfall particles.

Avoid expensive per-frame geometry reconstruction.

All animation must depend on the deterministic timeline.

## 12. VOLCANIC AND MAGICAL VFX

### Mount Doom

Add:

- Pulsing caldera glow.
- Moving lava ribbons.
- Rising dark ash particles.
- Orange glowing embers.
- Slow volcanic smoke motion.

Use a fixed-size particle pool or deterministic particle formulas.

### Barad-dûr

The fiery Eye must:

- Be visibly positioned above the tower.
- Pulse with orange-red light.
- Remain recognizable during night scenes.

### Minas Morgul

Add:

- Green spectral illumination.
- Subtle rotating crown or spire effect.
- Local nighttime glow.

### Dead Marshes

Add:

- Floating pale green-blue lights.
- Low mist.
- Gentle drifting motion.

### Rivendell

Add:

- Waterfalls.
- Fine mist.
- Soft reflective water highlights.

### Determinism

Every effect must be calculated from the authoritative timeline.

Do not use uncontrolled frame-to-frame randomness.

Seeking to the same time must reproduce the same visible state.

## 13. DAY/NIGHT CYCLE

Create a complete cinematic lighting cycle.

### Dawn

- Soft pink horizon.
- Warm golden sunlight.
- Cool terrain shadows.
- Gentle atmospheric haze.

### Midday

- Bright clear sky.
- Strong directional light.
- Crisp terrain colors.
- High visibility.

### Dusk

- Deep gold and amber lighting.
- Long shadows.
- Darkening distant landscape.
- Warm architectural highlights.

### Midnight

- Dark navy sky.
- Low-intensity moonlight.
- Visible stars.
- Stronger magical illumination.
- Mount Doom and Mordor glowing against darkness.

### Implementation

Use a directional sun and complementary moonlight approximation.

Interpolate sky and lighting colors continuously.

Adjust:

- Directional light intensity.
- Ambient intensity.
- Sky color.
- Fog color.
- Local emissive visibility.

Avoid abrupt lighting changes.

Do not drive the day/night cycle from an independent clock.

## 14. EXACT CINEMATIC DURATION

The cinematic journey must last exactly:

**227 seconds — 3 minutes and 47 seconds.**

At 60 FPS, the nominal frame count is:

**13,620 frame intervals.**

Treat frame indices 0 through 13,619 as the 13,620 output frames. The timeline endpoint at t = 227 seconds may be rendered separately if needed.

Use one authoritative timeline variable:

`cinematicTime`

All time-dependent systems must derive their state from this value.

Do not mix:

- Date.now().
- performance.now() for animation state.
- THREE.Clock elapsed time.
- Independent shader time counters.
- Unseeded Math.random().

Wall-clock time may be used only to advance cinematicTime during real-time playback.

## 15. CINEMATIC CAMERA PATH

Create a scripted camera journey using CatmullRomCurve3 or a suitable collection of interpolated camera segments.

The camera must move smoothly through the landscape.

Use separate position and look-at trajectories.

### Required Story Progression

**00:00–00:25 — The Shire**

- Begin at dawn.
- Low sweeping camera movement.
- Reveal Hobbiton and rolling green hills.
- Warm peaceful atmosphere.

**00:25–00:50 — Eriador and Rivendell**

- Travel past Bree and Weathertop.
- Approach Rivendell.
- Show waterfalls and elegant architecture.

**00:50–01:15 — Misty Mountains and Lothlórien**

- Rise above mountain ridges.
- Reveal Moria.
- Descend toward golden Lothlórien.

**01:15–01:40 — The Great River and Rohan**

- Pass the Argonath.
- Reveal Fangorn and Isengard.
- Sweep toward Edoras and Helm's Deep.

**01:40–02:05 — Gondor**

- Approach Minas Tirith.
- Show its seven defensive tiers.
- Pass over Osgiliath.
- Transition toward darker lighting.

**02:05–02:30 — The Northeast**

- Reveal Mirkwood and Dol Guldur.
- Approach Erebor and Dale.
- Show the Lonely Mountain silhouette.

**02:30–03:20 — Mordor**

- Lighting becomes dark and ominous.
- Pass the Dead Marshes and Black Gate.
- Reveal Minas Morgul and Cirith Ungol.
- Sweep toward Mount Doom.
- Finish on Barad-dûr and the fiery Eye.

**03:20–03:47 — Grand Finale**

- Camera rises above the entire diorama.
- Reveal the full landscape.
- Transition toward a dramatic atmospheric overview.
- End at exactly 227 seconds.

### Camera Quality

- Smooth position interpolation.
- Smooth target interpolation.
- No sudden snapping.
- No camera penetration through terrain.
- Avoid clipping through towers.
- Maintain readable landmark framing.
- Use stable camera orientation.
- Prevent excessive roll or abrupt vertical movement.

Prefer landmark readability over fast camera movement.

## 16. DETERMINISTIC TIMELINE ENGINE

Implement:

`seekTime(seconds)`

This function must:

1. Clamp or normalize the requested time.
2. Update the authoritative cinematic time.
3. Calculate camera position and target.
4. Calculate day/night lighting.
5. Update lava animation.
6. Update particles.
7. Update rotating structures.
8. Update water animation.
9. Update environmental effects.
10. Update HUD values.
11. Render the requested state.

Calling `seekTime(100)` repeatedly must produce the same visible scene.

Calling `seekTime(180)` and then `seekTime(100)` must reproduce the same appearance as calling `seekTime(100)` directly.

Do not implement VFX using accumulated frame-to-frame motion unless the state can be reconstructed deterministically from time.

### Real-Time Mode

Advance cinematicTime using a bounded real-time delta.

Stop at 227 seconds or loop according to the selected playback setting.

### Fixed-Frame Mode

Support frame stepping:

`seekFrame(frameIndex)`

Calculate:

`time = frameIndex / 60`

Frame stepping must not depend on the browser frame rate.

## 17. CAMERA MODES

Provide two modes.

### Cinematic Mode

- Camera follows the scripted timeline.
- Manual orbit input is disabled or switches to free mode.
- Camera transitions remain smooth.

### Free Camera Mode

- User controls camera with OrbitControls.
- Drag to orbit.
- Wheel to zoom.
- Touch gestures supported.
- Cinematic time can continue or pause independently.
- Returning to cinematic mode restores the scripted camera state.

Do not allow OrbitControls and the cinematic camera to overwrite each other simultaneously.

## 18. USER INTERFACE

Create a minimal cinematic overlay.

### Masthead

**MIDDLE-EARTH**

Subtitle: *A Journey Through Legend*

### HUD

Display:

- Current region.
- Current landmark when appropriate.
- Elapsed time.
- Total time: 03:47.
- Current frame.
- Playback state.

### Playback Controls

Include:

| Control | Behavior |
|---|---|
| Play / Pause | Start or stop cinematic time |
| Timeline Slider | Scrub between 0 and 227 seconds |
| Restart | Return to 00:00 |
| Cinematic / Free Cam | Toggle camera control |
| Previous Landmark | Jump to previous scripted landmark |
| Next Landmark | Jump to next scripted landmark |
| Day/Night Preview | Optional manual lighting preview while paused |
| FPS | Show measured rendering frame rate |

### Formatting

Display time as:

`MM:SS / 03:47`

Frame readout:

`Frame 00000 / 13620`

Ensure the user can scrub accurately.

### Visual Style

- Dark translucent panels.
- Elegant serif title.
- Compact sans-serif controls.
- Fine gold accent lines.
- Minimal UI over the scene.
- Readable typography.
- Responsive layouts.

No external font downloads.

## 19. PERFORMANCE REQUIREMENTS

Target approximately 60 FPS on capable desktop hardware.

Do not guarantee identical performance across all GPUs.

### Geometry

- Use low- to medium-polygon primitives.
- Reuse geometry and materials.
- Use InstancedMesh for repetitive objects.
- Avoid excessive shadow casters.
- Keep landmark silhouette quality high.
- Reduce fine geometric detail where it will not be visible.

### Terrain

- Generate geometry once.
- Recompute normals once during creation.
- Avoid rebuilding terrain every frame.

### VFX

- Use BufferGeometry and Points for particles.
- Use deterministic analytical motion.
- Avoid per-frame allocations.
- Limit particle counts.
- Avoid expensive postprocessing.

### Lighting

- Use a limited number of active lights.
- Prefer emissive materials for distant glowing objects.
- Limit shadow resolution.
- Avoid shadows on every tiny structure.
- Keep nighttime lighting readable.

### Animation

- Animate only components requiring movement.
- Reuse temporary vectors and matrices.
- Avoid rebuilding geometries in the animation loop.
- Keep seekTime updates deterministic and bounded.

## 20. ERROR HANDLING

Handle:

- Failed Three.js imports.
- Missing WebGL support.
- Shader or renderer initialization errors.
- Invalid terrain coordinates.
- Missing landmark definitions.
- Camera path failures.
- Invalid timeline values.
- Window resizing.
- WebGL context loss.

Show a meaningful error overlay rather than leaving a blank page.

## 21. IMPLEMENTATION PRIORITIES

Use this exact priority order:

**Priority 1 — Working Scene**

The HTML loads, creates the renderer, terrain, camera, and lighting.

**Priority 2 — Geography**

The world has recognizable regions, mountain chains, river valleys, and Mordor.

**Priority 3 — All 24 Landmarks**

Every landmark is generated, placed, and visually distinguishable.

**Priority 4 — Cinematic Camera**

The complete 227-second journey works.

**Priority 5 — Deterministic Playback**

seekTime and seekFrame reconstruct the complete scene consistently.

**Priority 6 — Environmental VFX**

Water, lava, smoke, embers, mist, and magical illumination.

**Priority 7 — Day/Night Cycle**

Smooth cinematic environmental transitions.

**Priority 8 — Polish**

Detailed vegetation, atmosphere, material improvements, and UI refinement.

Never omit landmarks or break deterministic playback to add optional visual effects.

## 22. INTERNAL VALIDATION

Before generating your final answer, review the implementation.

### Scene

- Renderer initializes.
- Terrain is visible.
- Lighting works.
- Camera is correctly positioned.
- Resize updates the projection matrix.

### Landmark Registry

- Exactly 24 landmark entries exist.
- Every entry has valid coordinates.
- Every entry uses an implemented builder.
- Every landmark produces visible geometry.
- Every landmark is positioned above terrain.
- No landmark is accidentally omitted.

### Geography

- The Shire is in the northwest.
- Misty Mountains form a central ridge.
- Gondor is south of Rohan.
- Mordor is surrounded by mountainous terrain.
- Erebor appears in the northeast.
- Rivers follow coherent geographic paths.

### Camera

- Timeline starts at 0.
- Timeline ends at 227.
- Camera paths contain valid positions.
- No abrupt target changes.
- Camera does not travel through mountains.
- Cinematic and free camera modes do not conflict.

### Determinism

- seekTime updates every animated system.
- No independent animation clocks.
- Repeated seeks produce identical results.
- Frame stepping uses frameIndex / 60.
- Particle positions are reconstructable from time.
- Day/night lighting uses cinematicTime.

### Performance

- Vegetation uses instancing.
- Reusable materials and geometries are shared.
- No unnecessary geometry rebuilding.
- Particle counts are bounded.
- Shadows are limited to meaningful objects.

### Interface

- Play and Pause work.
- Timeline scrubbing works.
- Restart works.
- Camera mode switching works.
- Current region updates.
- Frame count and elapsed time display correctly.

Fix identifiable problems before finalizing.

Do not claim to have run browser tests unless an actual execution environment is available.

## 23. OUTPUT FORMAT

Return only the complete working HTML document.

Start with:

`<!DOCTYPE html>`

End with:

`</html>`

Do not include:

- Markdown fences.
- Explanations.
- Pseudocode.
- TODO comments.
- Empty landmark builders.
- Placeholder geometry.
- Missing landmarks.
- Instructions to install dependencies.

All procedural generation, landmark construction, animation, timeline control, camera systems, and UI code must be fully implemented.

Do not truncate the output.

## FINAL SUCCESS STANDARD

The user opens `middle-earth-journey.html` and sees an enchanting miniature fantasy landscape.

Rolling green hills lead into enormous mountain chains. Ancient forests surround delicate cities. White fortresses rise above river valleys. Dark jagged towers dominate Mordor.

The camera begins over Hobbiton at dawn, crosses mountains and rivers, visits major regions, and reaches Mount Doom and Barad-dûr beneath a dark volcanic sky.

After exactly 3 minutes and 47 seconds, the camera reveals the full miniature world in a grand final shot.

All 24 landmarks are present, recognizable, and correctly positioned.

The user can pause, scrub, jump between landmarks, switch to a free camera, and deterministically render any moment of the journey.

The result should feel like an interactive handcrafted fantasy-world miniature brought to life through procedural graphics and cinematic storytelling.