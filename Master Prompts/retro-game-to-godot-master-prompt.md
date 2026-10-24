<!--
Type: Master planning prompt
Target: Godot 4.x, GDScript
Output: Phased implementation plan for a new original game
Related: None
-->

You are an expert game designer, senior Godot developer, software architect, and technical project planner.

Your task is to research a classic video game using the Wikipedia URL I provide and create a **complete, production-ready, phased implementation plan** for building an original game inspired by it using Godot.

The implementation will be performed by a local AI coding agent running **Qwen 3.8 Flash Next IQ3**, typically with a 64K context window.

The plan must be detailed enough that the coding agent can build the entire game without repeatedly asking for missing requirements.

**GAME REFERENCE URL:**

`[PASTE WIKIPEDIA URL HERE]`

---

## 1. Research the Original Game

Read the provided Wikipedia article and research additional reliable sources where needed.

Identify:

- Original game title, release date, developer, and genre
- Primary gameplay loop
- Player objectives
- Player controls and movement mechanics
- Enemies, obstacles, and hazards
- Weapons, abilities, and power-ups
- Levels, stages, and progression
- Scoring and reward systems
- Lives, health, failure, and game-over conditions
- Enemy AI and behavioral patterns
- Difficulty progression
- Distinctive features that make the game enjoyable
- Visual style and audio direction
- Important mechanics that should be preserved

Distinguish documented mechanics from assumptions.

If the original game has mechanics that would be excessively difficult for a smaller local coding model, propose practical alternatives that preserve the gameplay experience.

Do not invent historical facts or claim research was performed when it was not.

## 2. Create an Original Game Identity

Invent a new, memorable game title that suits the genre.

The game should be a spiritual successor, not an unauthorized reproduction.

Requirements:

- Original game title
- Original characters and enemy designs
- Original visual assets
- Original audio and music
- No copyrighted sprites, logos, or soundtracks
- Preserve the essential gameplay ideas that make the original fun

Provide a short elevator pitch and explain the design direction.

## 3. Technical Requirements

Use the following development stack:

- Engine: Godot 4.x stable
- Language: GDScript
- Platform: Windows and Linux desktop
- Architecture: Native Godot scenes and scripts
- Graphics: 2D by default, unless the reference clearly benefits from 3D
- Controls: Keyboard first, optional controller support
- Performance: Target 60 FPS
- Resolution: Choose an appropriate logical resolution and scaling strategy
- Dependencies: Prefer built-in Godot functionality
- Assets: Start with procedural or simple placeholder graphics

Use a normal multi-file Godot project.

Do not attempt to build the entire game in one script.

Choose the simplest architecture that supports the complete game without unnecessary complexity.

Verify version-sensitive Godot APIs against official documentation.

## 4. Design for a Smaller Local Coding Model

The implementation will use Qwen 3.8 Flash Next IQ3.

Plan accordingly.

Mandatory rules:

1. Divide development into small, independently testable phases.
2. Never require implementing the entire game in one pass.
3. Each phase must produce a runnable build.
4. Each phase must have explicit acceptance criteria.
5. Define exact file responsibilities.
6. Avoid unnecessarily complicated architecture.
7. Avoid large scripts when smaller components are appropriate.
8. Minimize external dependencies.
9. Use Godot-native features wherever practical.
10. Implement core mechanics before visual polish.
11. Avoid introducing future-phase functionality prematurely.
12. Require testing after every implementation phase.
13. Include instructions for correcting failed implementations.
14. Preserve already completed features.
15. Prevent the AI from rewriting working systems without justification.

Assume the model may lose context between sessions. The plan must explicitly support restarting development with a fresh context.

## 5. Required Game Design Specification

Document the following systems where applicable:

### Player System

- Movement
- Input handling
- Acceleration and physics
- Health or lives
- Damage and invulnerability
- Attacks and abilities
- Animation states
- Respawning

### Enemy System

- Enemy types
- Movement patterns
- AI state machines
- Attack behaviors
- Collision detection
- Health and destruction
- Spawn scheduling
- Difficulty scaling

### World System

- Level layout
- Terrain and obstacles
- Scrolling or camera behavior
- Level generation, if needed
- Environmental hazards
- Transitions between areas

### Combat System

- Weapons
- Projectiles
- Hit detection
- Damage rules
- Cooldowns
- Power-ups
- Special abilities

### Game Management

- Main menu
- Pause and resume
- Game over
- Restart
- Scoring
- High scores
- Progression
- Persistent settings

### Presentation

- Visual direction
- UI and HUD
- Particle effects
- Animation
- Audio effects
- Music
- Accessibility options

For every major system, describe its behavior, required components, and integration points.

Avoid vague requirements such as "implement good enemy AI."

Specify the actual behaviors the implementation must produce.

## 6. Godot Project Architecture

Provide a proposed directory structure.

Include appropriate locations for:

- Main scenes
- Player scenes
- Enemy scenes
- Environment scenes
- UI scenes
- Scripts
- Resources
- Assets
- Audio
- Tests
- Configuration
- Documentation

Explain the responsibilities of important classes.

Identify:

- Scene ownership
- Data flow
- Signals
- State machines
- Collision layers and masks
- Shared resources
- Save-game responsibilities
- Object lifecycle and cleanup

Avoid circular dependencies and oversized manager classes.

Do not create unnecessary files simply to satisfy the proposed directory tree.

## 7. Phased Implementation Plan

Create approximately **8–12 development phases**, adjusting the number to the complexity of the reference game.

Use this general structure:

**Phase 0 — Project Initialization**

- Godot configuration
- Directory structure
- Input mappings
- Initial scenes
- Development setup
- Testing foundation

**Early Phases — Core Gameplay**

- Player movement
- Essential environment mechanics
- Basic enemies or obstacles
- Collisions
- Primary interaction or combat
- Score and lives

**Middle Phases — Complete Playable MVP**

- Core gameplay loop
- Game states
- Difficulty progression
- Basic menus
- Essential HUD
- Playability testing

Identify an explicit **MVP milestone**.

The game should already be enjoyable at this point.

**Later Phases — Advanced Features**

- Additional enemies
- Advanced abilities
- Level variations
- Power-ups
- Bosses or special encounters, where appropriate
- Improved progression systems

**Final Phases — Polish and Release**

- Original graphics
- Original audio
- Particle effects
- UI refinement
- Performance improvements
- Save systems
- Testing
- Desktop exports

For every phase, include:

1. Phase name
2. Objective
3. Dependencies
4. Features to implement
5. Expected files to create or modify
6. Implementation instructions
7. Integration requirements
8. Acceptance criteria
9. Testing procedure
10. Common failure conditions
11. Definition of Done

Each phase must be independently verifiable.

Do not advance to the next phase until the current phase passes its completion gate and the user approves.

## 8. Mandatory TODO.md Progress Tracking

The Godot project must contain a `TODO.md`.

This is the single source of truth for implementation progress.

The game plan is the permanent specification.

`TODO.md` is the living development tracker.

Do not create a competing `IMPLEMENTATION_STATUS.md`.

The implementation agent must:

1. Read the game plan and `TODO.md` at the beginning of every session.
2. Create `TODO.md` if it does not exist.
3. Populate it with all development phases and tasks.
4. Keep unfinished tasks unchecked.
5. Mark tasks complete only after successful verification.
6. Record actual tests and results.
7. Record known bugs and blockers.
8. Preserve completed task history.
9. Reopen tasks when regressions are discovered.
10. Record important technical decisions.
11. Identify the next actionable task.
12. Resume incomplete work after context resets.
13. Never skip unfinished tasks without explaining why.
14. Never mark an entire phase complete merely because code was generated.
15. Wait for user approval before advancing phases.

Include a complete `TODO.md` template inside the game plan.

The template must include:

- Current development phase
- Last accepted phase
- Build status
- Next action
- Task checklists
- Phase completion gates
- Testing results
- Known issues
- Technical decisions
- Session history

Every phase must include a completion gate similar to:

```markdown
### Phase Completion Gate

- [ ] All required features implemented
- [ ] All tasks individually verified
- [ ] Godot imports without errors
- [ ] No GDScript parsing errors
- [ ] Gameplay tested
- [ ] No critical regressions
- [ ] Known issues documented
- [ ] TODO.md updated
- [ ] Phase reviewed and approved
```

Distinguish between:

- IN PROGRESS
- BLOCKED
- READY FOR REVIEW
- ACCEPTED

Only explicitly approved phases may be marked ACCEPTED.

## 9. Testing and Quality Requirements

Define testing procedures for the entire project.

Include:

- Godot headless import tests
- Scene loading
- Script parsing
- Input handling
- Player movement
- Collision detection
- Enemy behaviors
- Scoring and progression
- Game-over conditions
- Restart functionality
- Performance and memory
- Save/load behavior
- Desktop exports

Require the implementation model to report actual test commands, results, and unresolved issues.

Never claim tests passed without running them.

A successful headless import is not equivalent to a successful gameplay test.

## 10. Implementation Risks

Identify the most likely technical problems for this particular game.

Examples:

- Coordinate system errors
- Broken collision masks
- Duplicate damage events
- Enemy spawning problems
- Unbounded objects
- Physics inconsistencies
- Scene dependency problems
- Animation state conflicts
- AI complexity
- Performance degradation
- Excessive development scope
- Model context limitations

Explain how the architecture and development phases reduce those risks.

## 11. Master Coding Agent Handoff Prompt

At the end of the document, provide a complete master prompt that I can give to Qwen.

The prompt must instruct Qwen to:

1. Read the entire game plan.
2. Read `TODO.md`.
3. Inspect the existing Godot project.
4. Determine the current incomplete phase.
5. Identify the next incomplete task.
6. Implement only that task or approved phase.
7. Preserve existing functionality.
8. Run appropriate tests.
9. Fix errors before claiming completion.
10. Update `TODO.md`.
11. Report modified files and test results.
12. Stop and wait for approval before advancing.

Also provide a short recovery prompt to use when the model:

- Loses track of its current phase
- Starts implementing unrelated features
- Breaks existing functionality
- Repeats completed tasks
- Produces invalid Godot code

## 12. Output Requirements

Create **one complete Markdown document**.

Filename format:

`GAME_NAME_GODOT_GAME_PLAN.md`

Use uppercase letters and underscores in the filename.

The document must contain:

1. Game overview and research
2. Original game concept and title
3. Gameplay specification
4. Technical architecture
5. Proposed project structure
6. All implementation phases
7. Acceptance criteria for every phase
8. Detailed `TODO.md` requirements and template
9. Testing strategy
10. Performance goals
11. Common implementation pitfalls
12. Release criteria
13. Local coding model workflow
14. Intellectual-property and asset guidelines
15. Complete master implementation prompt
16. Recovery prompt
17. Research references

Write the actual implementation plan, not merely an outline.

Make concrete design choices where reasonable. Document assumptions clearly.

Prefer practical, implementable mechanics over ambitious features that a smaller model is unlikely to complete reliably.

The document must be detailed enough to guide development from an empty Godot project to a finished playable game.

## 13. Final Delivery

Save the complete plan as a downloadable Markdown file.

Do not create the Godot game itself.

Do not create separate phase documents.

Do not create `TODO.md` yet; the coding agent will create it when implementation begins.

The result should be a complete, self-contained implementation specification.

After creating the document, provide:

- Download link
- Original game reference
- New game title
- Number of phases
- MVP milestone
- Brief summary of key features
- Recommended first prompt for Qwen

**Begin by researching the supplied game URL. Then produce the complete plan without asking unnecessary clarification questions.**