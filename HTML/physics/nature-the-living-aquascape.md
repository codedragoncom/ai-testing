<!--
Type: Implementation prompt
Target: Three.js, vanilla JavaScript
Output: One standalone HTML file
Related: None
-->

# NATURE — The Living Aquascape
## Interactive 3D Nature Aquarium Experience

You are an expert Three.js graphics engineer, procedural 3D artist, simulation programmer, and creative frontend developer.

You specialize in realistic natural environments, GPU-efficient rendering, procedural geometry, believable organic animation, and interactive visual experiences.

## YOUR MISSION

Create **NATURE — The Living Aquascape**, an exceptionally beautiful interactive 3D Nature Aquarium inspired by the aquascaping principles of Takashi Amano and Aqua Design Amano.

The result should resemble a professionally designed, photographed, and maintained competition-quality planted aquarium.

The scene must feel calm, organic, balanced, and alive.

Deliver one complete `index.html` file containing the application.

**The most important requirement is visual composition.** The aquascape must look intentionally designed rather than randomly populated with procedural objects.

The second priority is believable living motion: schooling fish, gently swaying aquatic plants, rising CO₂ bubbles, and subtle water movement.

---

## 1. TECHNICAL REQUIREMENTS

- Deliver exactly one HTML file containing all application HTML, CSS, and JavaScript.
- Use Three.js for 3D rendering.
- Use vanilla JavaScript for application logic and controls.
- No React, Vue, build tools, npm packages, bundlers, or backend.
- No external images, models, textures, fonts, or audio.
- Generate all aquarium assets procedurally.
- Avoid GLTF, OBJ, and external asset loaders.
- Use InstancedMesh where appropriate.
- Use object pooling or reusable buffers for animated objects.
- Target approximately 60 FPS on capable desktop hardware.
- Provide usable mouse and touch camera interaction.
- Handle resizing and high-DPI displays.
- Include graceful error handling.

### Three.js Dependency Strategy

Use compatible Three.js and OrbitControls versions from the same release.

Recommended: Three.js r128 with legacy global-script OrbitControls, using fixed CDN URLs.

Alternatively, use one fixed modern Three.js version with a matching module import map, but avoid mixing incompatible module and nonmodule scripts.

Prefer the legacy global-script arrangement for simple standalone execution.

Do not use a moving `latest` dependency URL.

Three.js and OrbitControls are the only permitted external scripts.

All other application resources must be generated within the HTML file.

The file must not require a build step, but loading CDN scripts requires network access.

If the CDN fails, display a clear error message.

## 2. ART DIRECTION

Create an aquarium inspired by classic Nature Aquarium design principles:

- Asymmetrical natural composition.
- Golden-ratio focal placement.
- Strong foreground, middle-ground, and background separation.
- Natural negative space.
- Distinctive stone and wood silhouettes.
- Deliberate planting groups.
- Strong depth and scale.
- Subtle color variation.
- Calm and realistic motion.

### Design Philosophy

The aquascape must resemble a miniature natural landscape.

Think of a dramatic mountain valley with lush green foothills and an open riverbed winding through it.

Avoid evenly spaced rocks, symmetrical plants, random wood placement, or a flat carpet covering the entire tank.

### Composition

Use a hybrid Iwagumi and Ryoboku layout.

Primary focal point:

- Large angled Seiryu stone at approximately 38% of tank width.
- Strong directional lean toward the center.
- Two smaller supporting stones arranged naturally around it.
- Several secondary stones that reinforce the same geological direction.

Driftwood:

- Branching from the primary stone cluster.
- Extending diagonally toward the upper right.
- Following the larger composition rather than appearing randomly scattered.

Open space:

- Curving golden sand path.
- Wider in the foreground.
- Narrowing toward the background.
- Passing between planted rocky formations.

The layout should convey depth even when viewed from the front.

## 3. AQUARIUM AND DISPLAY

### Tank

Create a rimless ultra-clear glass aquarium inspired by premium low-iron glass aquariums.

Recommended dimensions in scene units:

- Width: 10.
- Height: 6.
- Depth: 5.

Use realistic proportions and thin glass walls.

### Glass

Use a transparent glass material with:

- Subtle Fresnel reflections.
- Very low surface roughness.
- Mild green-blue edge tint.
- Clean, nearly invisible front panel.
- Slight visible thickness along the edges.

Do not allow transmission effects or excessive reflection to obscure the interior.

Avoid duplicate overlapping transparent faces that cause flickering or heavy sorting artifacts.

A simplified transparent material is preferable to broken physically based transmission.

### Water

Fill the aquarium almost to the top.

Use a slightly tinted, highly transparent water volume.

Create a water surface approximately 0.15 units below the rim.

The surface should have:

- Subtle ripples.
- Gentle movement near the filter outflow.
- Mild light distortion.
- Restrained reflections.

Avoid large ocean-like waves.

### Stand

Place the aquarium on a premium minimalist display cabinet.

Material:

- Matte black or dark gunmetal.
- Sharp architectural edges.
- Fine subtle surface roughness.
- Small shadow gap below the tank.

The stand must support the visual composition without dominating it.

## 4. STUDIO LIGHTING AND RENDERING

### Renderer

Initialize:

- WebGLRenderer with antialiasing.
- High-performance preference.
- ACES filmic tone mapping.
- Exposure near 1.15.
- Correct sRGB output.
- Soft shadow maps.
- Pixel ratio capped at 1.5.

Use API properties compatible with the selected Three.js release.

### Lighting

Create a realistic aquarium lighting rig.

**Primary Light**

A large overhead white aquarium fixture inspired by ADA Solar RGB lighting.

- Bright but balanced.
- Cool-neutral white.
- Positioned above the water.
- Illuminates plants and substrate.
- Creates controlled shadows.

**Ambient Light**

Use HemisphereLight:

- Cool aqua-white sky.
- Deep earthy ground bounce.
- Moderate intensity.

**Accent Light**

Add subtle fill to improve glass edge visibility.

Avoid artificial theatrical colors inside the aquarium.

### Visual Atmosphere

- Dark studio background.
- Slightly illuminated rear wall.
- Soft floor shadow.
- Gentle environmental reflections.
- Clear aquarium silhouette.

### Underwater Light

Add animated caustics to the substrate and selected rocks.

Use a procedural CanvasTexture or lightweight custom shader.

Caustics should be subtle.

They must not look like moving white stripes.

## 5. PROCEDURAL SUBSTRATE

Create an actual shaped substrate surface.

### Aquasoil

Use dark volcanic aquasoil.

Base color: `#2b2622`

Characteristics:

- Irregular granular appearance.
- Small-scale height variation.
- Natural organic slopes.
- Higher elevation toward the rear.
- Lower elevation toward the front.

The rear substrate should be approximately twice as deep as the front substrate where visually appropriate.

### Sand Path

Create a winding pale-gold cosmetic sand path.

Base color: `#dfc49e`

The path must:

- Begin wide in the foreground.
- Curve gently toward the central valley.
- Narrow as it approaches the rear.
- Follow the terrain surface.
- Avoid perfectly straight edges.
- Include small natural variations along its border.

Use a curved polygon, spline-defined boundary, or a terrain mask.

The sand should not simply be a flat rectangle placed above the soil.

### Surface Detail

Add restrained procedural variation:

- Tiny soil granules.
- Fine sand color noise.
- Scattered small pebbles.
- Subtle transitions near stone boundaries.

Avoid thousands of individual draw calls.

## 6. SEIRYU STONE HARDSCAPE

Generate realistic rugged Seiryu-style stones procedurally.

### Main Stone — Oyaishi

The dominant stone must:

- Be tall and angular.
- Have a strong directional tilt.
- Feature fractured planes.
- Use blue-gray slate coloration.
- Contain subtle white mineral veins.
- Be partially buried in the substrate.

Do not use a smooth sphere or an unchanged default dodecahedron.

### Supporting Stones

Generate:

- Fukuishi supporting stones.
- Smaller secondary stones.
- Several partially buried fragments.

All stones should share:

- Similar geological texture.
- Compatible fracture directions.
- Natural proportions.
- Related orientation.

### Geometry

Start from a subdivided primitive or custom irregular mesh.

Displace vertices using deterministic multi-frequency noise.

Introduce directional ridges and faceted breaks.

Recompute vertex normals.

Add light and dark surface variation using vertex colors or procedural material logic.

### White Veins

Generate restrained calcite vein details using object-space procedural coordinates.

The veins should appear integrated into the stone, not as random painted white stripes.

### Placement

Partially bury stones to make them feel heavy and grounded.

Avoid hovering stones or obvious intersections with the front glass.

## 7. PROCEDURAL DRIFTWOOD

Create a natural branching driftwood structure.

### Architecture

Use a recursive branching algorithm with:

- Primary trunk sections.
- Tapered secondary branches.
- Smaller delicate tips.
- Controlled curvature.
- Irregular branching angles.

### Geometry

Use tapered cylinders or custom tube geometry.

Ensure connected segments do not show excessive gaps.

### Material

- Dark warm brown.
- Weathered gray highlights.
- Slight lengthwise grain.
- Matte to satin finish.

### Placement

Branches should grow diagonally from the main stone cluster.

Their shapes should reinforce the overall direction of the aquascape.

Do not generate an evenly symmetrical tree.

Limit branch count for performance.

## 8. AQUATIC CARPET PLANTS

Create a dense but natural carpet of aquatic vegetation.

### Species Inspiration

Use forms inspired by:

- Glossostigma elatinoides.
- Dwarf hairgrass.
- Monte Carlo.

### Instance Count

Target approximately 2,000–4,000 visible plant elements on desktop.

Use InstancedMesh or batched geometry.

Do not create thousands of separate Mesh objects.

### Geometry

Use simple procedural forms:

- Two-leaf sprigs.
- Small rounded leaf clusters.
- Thin grass blades.
- Tapered ribbons.

Use more than one plant shape to prevent uniform repetition.

### Colors

Use a natural range of greens:

- Deep emerald.
- Fresh leaf green.
- Lime growth tips.
- Muted olive shadows.

Avoid neon saturation.

### Distribution

Plant density must follow the aquascape composition.

- Dense growth near stone bases.
- Medium growth along slopes.
- Sparse growth near the sand path.
- Small natural gaps.
- Mixed growth heights.

Do not place plants inside stones, glass walls, or exposed sand.

### Plant Motion

Implement very subtle current-driven movement.

Preferred method:

- Custom ShaderMaterial or onBeforeCompile vertex modification.
- Instance-dependent phase variation.
- Movement increases toward plant tips.
- Root positions remain fixed.

Avoid updating thousands of instance matrices every frame.

The plants should gently sway, not flap violently.

## 9. BACKGROUND STEM PLANTS

Create dense groups of taller aquatic plants.

### Rotala-Inspired Plants

Place clusters in the rear corners and behind the main hardscape.

Characteristics:

- Tall thin stems.
- Small paired leaves.
- Dense layered growth.
- Soft reddish-orange tips.
- Green lower foliage.

### Composition

Plant clusters must vary in height.

Use rising and