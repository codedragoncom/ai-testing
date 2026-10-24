<!--
Type: Build instructions
Target: HTML Canvas, vanilla JavaScript
Output: Phased implementation of jungle-assault.html
Related: jungle-assault.md; jungle-assault-phased-game-plan.md
-->

# Games



# JUNGLE ASSAULT — Full Phased Game Development Plan

For Qwen 27B Q3 running locally, I recommend developing Jungle Assault as a 12-phase project rather than attempting to generate the entire game in one response.

The key difference is that we'll treat this like a real game development project: separate gameplay systems, clear milestones, automated testing, and a working game at the end of every phase.

Final deliverable: `jungle-assault.html` — a complete, standalone HTML5 arcade game with eight scrolling stages, intelligent enemies, grenades, destructible environments, prisoner rescues, and polished retro graphics.

## Development roadmap

01

Foundation & game architecture

Canvas, modules, state machine, game loop

02

Player movement & camera

Eight-way movement, scrolling, collision

03

Weapons & combat physics

Rifle, bullets, grenades, explosions

04

Enemy AI & spawning

Six enemy classes, behavior and waves

05

World & terrain

Jungle, rivers, cover, environmental collision

06

Level design & progression

Eight playable, distinct scrolling stages

07

Fortress encounters

Defensive battles, wave triggers, bosses

08

Pickups & rescue systems

Prisoners, ammunition, bonus lives

09

Procedural pixel art & VFX

Characters, animation, explosions, particles

10

Audio & interface

Synthesized sounds, menus, HUD, accessibility

11

Mobile & performance

Twin-stick controls, pooling, optimization

12

QA, balancing & single-file export

Full playthrough, bug fixes, shipping

## Architecture recommendation

Don't force Qwen to edit a 20,000-line HTML file throughout development. Use modular JavaScript, then bundle everything into one HTML file during Phase 12.

```
jungle-assault/
├── GAME_PLAN.md
├── index.html
├── package.json
├── src/
│   ├── main.js
│   ├── core/
│   │   ├── Game.js
│   │   ├── GameLoop.js
│   │   ├── Input.js
│   │   ├── Camera.js
│   │   └── Collision.js
│   ├── entities/
│   │   ├── Player.js
│   │   ├── Enemy.js
│   │   ├── Bullet.js
│   │   ├── Grenade.js
│   │   └── Pickup.js
│   ├── systems/
│   │   ├── Combat.js
│   │   ├── EnemyAI.js
│   │   ├── Spawning.js
│   │   ├── Terrain.js
│   │   ├── Missions.js
│   │   ├── Audio.js
│   │   └── Effects.js
│   ├── levels/
│   │   ├── level01.js
│   │   └── ...level08.js
│   └── ui/
│       ├── HUD.js
│       └── Menus.js
├── tests/
├── tools/
│   └── build.mjs
└── dist/
    └── jungle-assault.html
```

The development build may use local modules and testing tools. The shipping artifact must contain all game JavaScript, CSS, audio synthesis, and graphics inline, with no external dependencies.

## Download the complete implementation plan

jungle-assault-phased-game-plan.md

12 phases · 301 lines · Markdown

Download the full game development plan

The file contains the full specification, implementation requirements for every phase, test criteria, progress tracker, reusable agent instructions, and final release checklist.

## How I'd run this with Qwen 27B Q3

Start your coding agent in a clean project directory, add the Markdown file as `GAME_PLAN.md`, and give it this instruction:

Read `GAME_PLAN.md` completely. Treat it as the authoritative specification. Implement Phase 01 only. Run available tests, fix failures, update the progress tracker, and stop when Phase 01 is complete.

&#x20;Copy instruction

Once a phase passes its quality gate, instruct the agent to continue with the next phase. A Git commit after each successful phase gives you a safe rollback point.

The major advantage: Qwen no longer needs to implement an entire arcade game in one generation. Each pass addresses a specific subsystem, while the project plan keeps the finished game consistent with the original vision.