# AI Testing Prompts

A collection of prompts and build specifications for exercising AI coding models. The repository focuses on browser games, interactive simulations, and retro game projects planned for Godot or Unity. Most files are instructions for generating a project; they are not the finished games themselves.

## What's here

| Area | Contents |
| --- | --- |
| [HTML games](HTML/games/) | Self-contained browser game prompts, including arcade games and a Phaser variant. Some larger projects have separate phased plans and build instructions. |
| [HTML simulations](HTML/simulation/) | Small canvas-based agent, cellular automaton, and particle simulation prompts. |
| [HTML physics and graphics](HTML/physics/) | More demanding fluid, soft-body, WebGL, and Three.js experiences. |
| [Godot plans](Godot/) | Seven detailed, phased game specifications for Godot 4 and GDScript. |
| [Master prompts](Master%20Prompts/) | Reusable prompts that turn a classic game reference URL into a Godot or Unity project plan. |
| [Other HTML](HTML/uncategorized/README.md) | The Middle-earth cinematic experience prompt and its multi-pass build plan. |

## HTML games

- [Gold Runner](HTML/games/gold-runner/gold-runner-single-file-html.md) — A classic platform-puzzle game prompt for a single HTML file.
- [Jungle Assault](HTML/games/jungle-assault/jungle-assault.md) — A vertical run-and-gun game prompt. See also its [phased plan](HTML/games/jungle-assault/jungle-assault-phased-game-plan.md) and [build instructions](HTML/games/jungle-assault/jungle-assault-build-instructions.md).
- [Neon Breaker](HTML/games/neon-breaker.md) — A single-file arcade brick-breaker.
- [Ore Runner 2049](HTML/games/ore-runner.md) — A retro platformer set in an abandoned mine.
- [Rivet Rush](HTML/games/rivet-rush.md) — A single-file, vanilla JavaScript platform game.
- [Rivet Rush (Phaser)](HTML/games/rivet-rush-phaser.md) — A Phaser 3.90.0 version of the same game concept; useful for comparing framework and no-framework generations.
- [Scrap Tower Scramble](HTML/games/scrap-tower-scramble.md) — A self-contained classic arcade game prompt.
- [Void Sentinel](HTML/games/void-sentinel/void-sentinel-prompt.md) — A Defender-inspired scrolling space shooter. Its [README](HTML/games/void-sentinel/README.md) describes the concept and technical scope.

## Simulations

These prompts generally ask for a standalone HTML implementation using canvas and vanilla JavaScript.

- [Ant colony](HTML/simulation/ant-simulation.md)
- [Boids flocking](HTML/simulation/boids-flocking.md)
- [Bouncing balls](HTML/simulation/bouncing-balls.md)
- [Conway's Game of Life](HTML/simulation/conways-game-of-life.md)
- [Forest-fire cellular automaton](HTML/simulation/forest-fire-cellular-automaton.md)
- [Langton's Ant](HTML/simulation/langtons-ant.md)
- [Particle fireworks](HTML/simulation/particle-fireworks.md)
- [Predator–prey agents](HTML/simulation/predator-prey-agents.md)
- [Reaction–diffusion (simplified Gray–Scott)](HTML/simulation/reaction-diffusion-gray-scott-style-simplified.md)
- [Schelling's segregation model](HTML/simulation/schellings-segregation-model.md)
- [Slime mold (Physarum-style)](HTML/simulation/slime-mold-physarum-style.md)

## Physics and graphics

- [2D fluid dynamics](HTML/physics/2d-fluid-dynamics-simulator.md) — An interactive fluid solver, not just a particle effect.
- [Jelly Dice](HTML/physics/jelly-dice-optimized.md) — A WebGPU soft-body simulation.
- [Krakatoa](HTML/physics/krakatoa-the-eruption.md) — A real-time volcanic eruption visualization using WebGL2.
- [Lava: A Living Light](HTML/physics/lava-lamp.md) — A raymarched WebGL2 lava-lamp scene.
- [Nature: The Living Aquascape](HTML/physics/nature-the-living-aquascape.md) — A procedural Three.js aquarium experience.

## Godot game plans

These are long-form implementation specifications rather than finished Godot projects. Each defines an original game concept and a staged implementation process.

- [Dustline Rover](Godot/dustline-rover-godot-game-plan.md) — A Moon Patrol-inspired lunar rover shooter with jumps, hazards, and scrolling stages.
- [Maze Munchers](Godot/maze-munchers-godot-game-plan.md) — A maze-chase game inspired by classic arcade play.
- [Prism Hopper](Godot/prism-hopper-godot-game-plan.md) — A Q*bert-inspired isometric platform game.
- [Rift Current](Godot/rift-current-godot-game-plan.md) — A vertically scrolling river-flight shooter inspired by River Raid.
- [Shadow Intercept](Godot/shadow-intercept-godot-game-plan.md) — Retro top-down vehicular combat.
- [Shattered Realms](Godot/shattered-realms-godot-game-plan.md) — An original turn-based territory strategy game.
- [Starfall Chronicles](Godot/starfall-chronicles-godot-game-plan.md) — A fantasy RPG with an overhead overworld, first-person dungeons, and a space-combat endgame.

## Reusable planning prompts

- [Retro game to Godot](Master%20Prompts/retro-game-to-godot-master-prompt.md) — Researches a supplied classic-game reference URL and produces a phased plan for an original Godot game.
- [Retro game to Unity](Master%20Prompts/retro-game-to-unity-master-prompt.md) — The corresponding workflow for Unity and C#.

The Middle-earth collection has a [single-file Three.js experience prompt](HTML/uncategorized/middle-earth-journey.md) and a separate [10-pass build plan](HTML/uncategorized/middle-earth-build-plan.md).

## Using and adding prompts

Open a prompt and provide it to the target model with any required reference material. For staged projects, use the accompanying plan as the specification and follow its gates; do not treat a plan as evidence that its game has already been implemented or tested. Some prompts require external libraries or a CDN, while others explicitly require a standalone file; check the individual prompt before running it.

Prompt and plan files start with a short HTML-comment metadata block listing the document type, target technology, expected output, and related files. It stays hidden in normal Markdown rendering.

When adding a prompt, place it in the closest existing category, use a lowercase kebab-case filename, and link it here. Keep related variants and phased plans together, state the expected output and dependencies, and avoid committing credentials or personal data. Add a new category only when existing sections do not fit.
