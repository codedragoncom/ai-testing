<!--
Type: Master planning prompt
Target: Unity, C#
Output: Phased implementation plan for a new original game
Related: None
-->

You are an expert game designer, senior Unity/C# developer, software architect, and technical project planner.

Your task is to research a classic video game using the Wikipedia URL I provide and create a **complete, production-ready, phased implementation plan** for building an original game inspired by it using Unity.

The implementation will be performed by a local AI coding agent running **Qwen 3.8 Flash Next IQ3**, typically with a **64K context window**. Make the plan concrete enough that the agent can implement the game without repeatedly asking for missing requirements.

**GAME REFERENCE URL:**

`[PASTE WIKIPEDIA URL HERE]`

---

## 1. Research the Original Game

Read the supplied Wikipedia article and consult other reliable sources where needed. Identify:

- Original title, release date, developer, and genre
- Primary gameplay loop and player objectives
- Controls, movement, acceleration, and physics
- Enemies, obstacles, environmental hazards, and AI patterns
- Weapons, abilities, and power-ups
- Level structure, progression, scoring, and rewards
- Health, lives, failure, and game-over rules
- Difficulty progression and distinctive gameplay features
- Visual style, camera behavior, and audio direction
- Which mechanics should survive in the new design

**Separate documented facts, reasonable inferences, and new design choices.** Cite references with URLs. If a feature is too difficult for a small local model, propose a simpler equivalent that preserves the fun. Never invent historical facts or pretend to have researched inaccessible sources.

## 2. Create an Original Game Identity

Invent a memorable, original title and provide an elevator pitch, design pillars, target audience, and intended session length. Make a spiritual successor, not a reproduction.

- Create original characters, enemy designs, stages, interface, names, and lore.
- Use original or properly licensed graphics, fonts, music, and sound effects.
- Never copy copyrighted sprites, logos, names, level maps, or soundtracks.
- Preserve abstract gameplay ideas and satisfying player feedback.
- Describe specifically how this game differs from the reference.

## 3. Technical Requirements

Use the following default stack, unless the reference strongly justifies an exception:

| Area | Requirement |
| --- | --- |
| Engine | Unity 6 LTS, or another explicitly identified supported Unity LTS version |
| Language | C# with the .NET profile and C# language features supported by the chosen Unity editor |
| Platforms | Windows and Linux desktop |
| Rendering | 2D by default; use 3D only if the gameplay clearly benefits |
| Render pipeline | Built-in 2D setup or URP when its specific features are needed; select one in the plan |
| Input | Unity Input System package; keyboard first, controller optional |
| UI | uGUI (Canvas/TextMeshPro) by default; select UI Toolkit only with a clear reason |
| Physics | Physics 2D for 2D games, Physics for 3D; never mix them accidentally |
| Frame rate | Target 60 FPS on modest desktop hardware |
| Scale | Choose a reference resolution, aspect handling, pixel-art filtering if applicable, and camera strategy |
| Dependencies | Unity packages only when justified; avoid third-party frameworks |
| Assets | Begin with primitives, sprites, simple generated art, or other licensed placeholders |
| Source control | Git; commit working milestones; ignore generated Unity cache/build directories |

Create a **normal multi-file Unity project** with scenes, GameObjects, components, prefabs, ScriptableObjects when useful, and C# scripts. Do not build the game in one MonoBehaviour, rely on a single mega-manager, or introduce heavy dependency injection and event frameworks.

**Version-specific rule:** Verify APIs, package versions, build targets, editor command-line flags, and the selected rendering/input workflows against **official Unity documentation**. Do not invent package IDs or API methods. Record the exact Unity editor version and package manifest once implementation starts. Never assume an arbitrary Unity project can be generated and serialized correctly by hand: prefer Unity Editor creation, prefab authoring, and Editor scripts where needed. If the Unity Editor is unavailable, explicitly mark editor-dependent setup and playtesting as unverified.

## 4. Design for a Smaller Local Coding Model

The implementation agent uses **Qwen 3.8 Flash Next IQ3**. Follow these rules:

1. Split work into small, independently verifiable tasks and 8–12 phases.
2. Every phase should leave a runnable project; no full-game one-shot implementation.
3. Give exact script, scene, prefab, and data responsibilities.
4. Keep classes small, cohesive, and easy to inspect.
5. Limit dependencies and favor Unity-native tools.
6. Build gameplay first; polish later.
7. Do not implement future-phase systems early.
8. Test and fix failures after every task and phase.
9. Preserve accepted behavior; never rewrite it without a documented reason.
10. Specify interfaces, serialized fields, events, and prefabs before connecting systems.
11. Keep each coding prompt small enough for a 64K context window.
12. Use a written tracker so a fresh-context session can resume deterministically.
13. Avoid speculative abstractions, unnecessary packages, and premature optimization.
14. Prevent merge/serialization churn: do not re-create working scene, prefab, or meta files.
15. **Do not advance to a new phase until the user explicitly approves the current phase.**

## 5. Required Game Design Specification

Specify actual rules, parameter defaults, state transitions, and interactions for all applicable systems. Avoid vague directions like “make smart enemy AI.”

### Player

- Input Actions asset and action map design; bindings and rebinding policy
- Movement, acceleration, stopping, facing, and physics timestep
- Collider and Rigidbody/Rigidbody2D configuration
- Health/lives, damage, knockback, invulnerability, respawn
- Weapons, attacks, cooldowns, abilities, animation states
- Interaction with hazards, pickups, and checkpoints

### Enemies

- Named archetypes with speed, health, damage, attack range, and scoring values
- Explicit patrol/chase/attack/retreat state transitions where useful
- Detection logic, movement, collisions, spawn triggers, despawn behavior
- Difficulty scaling rules and limits on simultaneous enemies

### World

- Scene flow, stage layout, world bounds, terrain, scrolling, and camera rules
- Obstacles, collision layers, checkpoints, hazards, and transitions
- Procedural generation or Tilemap usage only if justified
- Level data ownership and stage completion conditions

### Combat and Interaction

- Projectiles, hitboxes/hurtboxes, friendly-fire rules, collision filtering
- Damage ownership and exactly-once hit behavior
- Fire rates, cooldowns, power-ups, and effect duration
- Object pooling only if profiling or spawn frequency justifies it

### Game Flow and Persistence

- Title screen, play, pause, game-over, restart, and victory states
- Score, lives, progression, high scores, and difficulty ramps
- Settings and save schema, persistence location, defaults, and migration behavior
- Distinction between runtime-only state and saved data

### Presentation and Accessibility

- Style guide, resolution, sprite sorting, animation, particles, camera shake
- Canvas layout, HUD, menus, readable fonts, feedback effects
- AudioSource/AudioMixer organization, SFX, music, volume categories
- Input remapping when in scope, audio controls, readable contrast, reduced-flash option

For each major system, state: **behavior, exact component(s), key data, integration points, failure cases, and validation method**. Make concrete default tuning choices and label them as assumptions.

## 6. Unity Project Architecture

Propose a minimal directory layout similar to this, adapting it to the game:

```text
Assets/
  _Game/
    Art/
    Audio/
    Materials/
    Prefabs/
      Player/
      Enemies/
      World/
      UI/
    Scenes/
    Scripts/
      Core/
      Player/
      Enemies/
      World/
      Combat/
      UI/
      Data/
    ScriptableObjects/
    Settings/
    Tests/
      EditMode/
      PlayMode/
  TextMesh Pro/                  # If imported by Unity
Packages/
ProjectSettings/
UserSettings/                    # Usually not committed
Docs/
GAME_NAME_UNITY_GAME_PLAN.md
TODO.md                          # Created later by implementation agent
```

Specify which folders are actually needed, and do not create empty folders to match the example. Include scene names and flow (for example Bootstrap, MainMenu, Gameplay); prefab responsibilities; scene ownership; serialized references; dependency direction; ScriptableObject data; interfaces/delegates/C# events or UnityEvents; and object lifetime.

Also specify:

- MonoBehaviour responsibilities (`Awake`, `OnEnable`, `Start`, `Update`, `FixedUpdate`, `OnDisable`, `OnDestroy` as appropriate)
- Which systems are scene-bound and which, if any, persist through scene loads
- 2D/3D collision matrix and trigger-vs-collision decisions
- Reference assignment strategy and handling of missing inspector references
- Event subscription/unsubscription strategy to avoid dangling listeners
- Prefab variants only when needed; avoid nested prefab complexity
- Sorting layers, render layers, camera projection, and UI scaling
- Save/load responsibility and safe handling of corrupted or absent saves
- Assembly Definition files only when they make test isolation or compile times simpler
- `.meta` files and Unity YAML scene/prefab assets preserved in version control
- Version-control exclusions for `Library/`, `Temp/`, `Obj/`, `Logs/`, `Build/`, and other generated caches

Explain how components communicate without circular dependencies or an oversized GameManager. **Never treat a C# class file as a substitute for a configured Unity prefab or scene.** Every scene/prefab requiring Inspector wiring must have a concrete assembly checklist and verification step.

## 7. Phased Implementation Plan

Create approximately **8–12 phases**, adjusted to the original game.

**Phase 0 — Project Initialization**

- Create the Unity project with the selected LTS editor, template, rendering pipeline, and packages.
- Define source control, folder structure, scene list, resolution and quality settings.
- Configure Input System actions, physics layers, and editor/test foundation.
- Create a minimal scene that opens and enters Play Mode without errors.

**Early phases — Gameplay foundation**

- Player movement and camera
- Environment and collision setup
- Basic hazards/enemies, interactions, projectiles as relevant
- Score, health, lives, and restart

**Middle phases — Playable MVP**

- A complete, repeatable gameplay loop
- Game states, HUD, essential menu screens
- Difficulty rules, stage completion, and loss conditions
- Gamepad support if it is part of the core experience

Mark a precise **MVP milestone**; the game must already be enjoyable and playable from start to finish in a limited form.

**Later phases — Expansion**

- Enemy/stage variations and advanced mechanics
- Pickups, progression, bosses, or special encounters only when warranted
- Additional accessibility, tuning, and settings

**Final phases — Polish and delivery**

- Replace placeholders with original or properly licensed assets
- Animation, original audio, UI polish, feedback, and particles
- Performance profiling, saves, regression testing, and desktop builds

For **every phase**, supply all eleven items:

1. Phase number and name
2. Objective and player-visible outcome
3. Prerequisites / accepted dependency phases
4. Small, numbered implementation tasks
5. Exact files, scenes, prefabs, assets, or settings to create/modify
6. Step-by-step Unity/C# implementation directions
7. Inspector, package, scene, and prefab integration checklist
8. Measurable acceptance criteria
9. Tests to run and manual playtest procedure
10. Likely failure modes and recovery guidance
11. Definition of Done and approval gate

**Every phase must be independently verifiable.** Stop after the phase gate, summarize work, and wait for user approval before starting the next phase.

## 8. Mandatory `TODO.md` Progress Tracking

The eventual Unity project must have one `TODO.md` as the **single source of truth for implementation progress**. The game plan is the permanent specification; `TODO.md` is the living tracker. Do not also create `IMPLEMENTATION_STATUS.md`.

The coding agent must:

1. Read the plan and `TODO.md` at the start of every session.
2. Create `TODO.md` on first implementation run, not during plan generation.
3. Copy all phase/task checklists from the approved plan.
4. Keep unverified tasks unchecked; mark them done only after real verification.
5. Record commands, actual output/results, manual test status, bugs, and blockers.
6. Preserve history and completed work; reopen tasks when regressions occur.
7. Record relevant architecture/API decisions and why they were made.
8. Keep exactly one actionable next task and the current gate status.
9. Resume incomplete work after a context reset; avoid repeat implementations.
10. Never treat code generation or compilation alone as phase acceptance.
11. Never move phases without direct user approval.

Include this **full template** inside the generated game-specific plan, populated with its actual phases and tasks:

```markdown
# TODO — [GAME NAME]

## Current Status
- Current phase: Phase 0 — [name]
- Phase status: IN PROGRESS
- Last accepted phase: None
- Unity editor version: Not yet recorded
- Build status: NOT TESTED
- Next action: [one specific, numbered task]
- Last verified on: Not yet tested

## Phase Checklist
### Phase 0 — [Name]
- [ ] 0.1 [Concrete task]
- [ ] 0.2 [Concrete task]

#### Phase Completion Gate
- [ ] All required features implemented
- [ ] Every task individually verified
- [ ] Unity project opens in the chosen editor version
- [ ] C# scripts compile with zero errors
- [ ] No unresolved blocking Console errors
- [ ] Edit Mode tests passed (if applicable)
- [ ] Play Mode tests passed (if applicable)
- [ ] Actual gameplay manually tested in the Editor
- [ ] Existing features regression-tested
- [ ] Known issues documented
- [ ] TODO.md updated
- [ ] Phase reviewed and explicitly approved by user

### Phase 1 — [Name]
- [ ] 1.1 [Concrete task]
- [ ] 1.2 [Concrete task]

#### Phase Completion Gate
- [ ] All required features implemented
- [ ] Every task individually verified
- [ ] Unity project opens, scripts compile, and no blocking Console errors
- [ ] Relevant Edit Mode / Play Mode tests passed
- [ ] Actual gameplay manually tested
- [ ] No critical regressions
- [ ] Known issues and test evidence recorded
- [ ] TODO.md updated
- [ ] Phase reviewed and explicitly approved by user

<!-- Repeat each concrete phase and its gate through final release. -->

## Test Log
| Date | Phase/task | Environment and exact command/action | Result | Evidence / failures |
| --- | --- | --- | --- | --- |
| YYYY-MM-DD | 0.1 | [actual step] | NOT RUN / PASS / FAIL | [details] |

## Known Issues and Blockers
- None recorded.

## Technical Decisions
| Date | Decision | Reason | Impact |
| --- | --- | --- | --- |

## Session History
| Date | Work completed | Files changed | Tests actually run | Next step |
| --- | --- | --- | --- | --- |

## Phase Approval History
| Phase | Status | Requested on | Approved by user on | Notes |
| --- | --- | --- | --- | --- |
```

Valid phase states: **IN PROGRESS**, **BLOCKED**, **READY FOR REVIEW**, **ACCEPTED**. Only directly approved phases become **ACCEPTED**. A phase may be READY FOR REVIEW after passing its tests; that is not approval.

## 9. Testing and Quality Requirements

Define practical tests for the selected game and project environment:

- Unity Editor opening the project, asset refresh/import, package resolution, and clean C# compilation
- Edit Mode tests for pure rules (scoring, damage calculations, progression, save parsing)
- Play Mode tests for component interactions where reliable automation is practical
- Input action activation, movement, physics, collisions, hit detection, and enemy state changes
- Health/lives, score, pause/resume, death, game-over, restart, and victory
- Camera framing, sprite sorting, UI scaling, and aspect ratios
- Save/load, missing or corrupt save data, and settings
- Performance and memory in a representative play session
- Windows/Linux standalone development builds and launch smoke tests
- Regression tests for previously accepted phases

**Distinguish** the following outcomes explicitly:

1. Static inspection only
2. Unity C# compilation / Editor asset import
3. Automated Edit Mode tests
4. Automated Play Mode tests
5. Human or automated interactive gameplay observation
6. Standalone player build and launch test

The Unity Test Framework/Unity Test Runner can execute Edit Mode and Play Mode tests. When using command-line batch tests, verify the chosen Unity version's documented options and ensure test reports and exit status are captured. Batch mode may not reproduce graphics, timing, user input, or GUI play; it **does not** prove gameplay feels correct. Do not assert passing tests without running them. If Unity is unavailable, specify exact manual/editor verification steps and mark them **NOT RUN / BLOCKED**, not PASS.

For every phase, show **exact commands where applicable**, test scene/prefab setup, expected output, and what to do if the test fails. Ensure the coding agent never attempts to bypass a failing test by deleting it or loosening its assertions without explicit explanation and approval.

## 10. Implementation Risks

Identify risks specific to the reference game and explain mitigation through architecture, tuning limits, and development phases. Consider:

- Incorrect world/local coordinates or delta-time calculations
- `Update` vs `FixedUpdate` timing mistakes
- Rigidbody movement conflicting with transform-based movement
- Incorrect Physics/Physics2D collision matrix or trigger configuration
- Duplicate damage from multiple collision callbacks
- Broken prefab references, inactive GameObjects, missing serialized fields
- Script lifecycle and event subscription errors
- ScriptableObject asset state accidentally mutated at runtime
- Bad scene loading, duplicate persistent singleton instances
- Enemy spawn storms and objects never despawning
- Animation/physics state disagreement
- Pixel scaling, canvas clipping, and camera aspect bugs
- Asset GUID/`.meta` mismatches, Unity YAML merge conflicts
- Package version/API mismatch and platform-specific build failures
- Scope growth, model context resets, and unverified claims

## 11. Master Coding Agent Handoff Prompt

At the **end of the game-specific document**, provide a complete copy-paste prompt for Qwen, using concrete filenames and phase names:

```text
You are implementing [GAME NAME] in Unity using C#.

1. Read [GAME_NAME_UNITY_GAME_PLAN.md] completely.
2. Read TODO.md; create it from the plan's template only if missing.
3. Inspect the actual Unity project: Assets, Packages/manifest.json,
   ProjectSettings, scenes, prefabs, C# scripts, tests, and Git changes.
4. Record the installed Unity Editor version and confirm the project's
   Unity packages/API choices match the approved plan.
5. Find the current incomplete task in the current approved phase.
6. Implement only that task (or the explicitly approved scope).
7. Preserve existing working functionality, scene/prefab wiring,
   serialized references, GUIDs, .meta files, and accepted behavior.
8. Describe any Inspector, prefab, scene, layer, Input Actions, or
   project-setting changes; implement them in the Editor when available.
9. Run the relevant tests and compilation/build checks when possible.
   Never call a test PASS if you did not execute it.
10. Fix errors within scope; record any remaining manual checks or blockers.
11. Update TODO.md with verified checkboxes, actual test evidence,
    modified files, open issues, and the single next action.
12. Report exactly what changed, what passed/failed/was not run,
    and whether the current phase is READY FOR REVIEW.
13. STOP. Do not start the next phase until I approve it.

Prefer small changes and Unity-native systems. Avoid unnecessary packages,
new abstractions, or refactoring working code. When blocked by a missing
Unity Editor or an unclear design decision, state the blocker plainly.
```

Also provide this **short recovery prompt**, customized for the game:

```text
STOP implementing new features. Read [GAME_NAME_UNITY_GAME_PLAN.md]
and TODO.md, inspect Git changes and current Unity project files,
and determine the last ACCEPTED phase plus the next unchecked task.
Check that existing scenes, prefabs, .meta files, and Inspector references
still work. Reproduce the reported failure, identify the smallest fix,
run the applicable tests, update TODO.md truthfully, and report results.
Do not rewrite completed systems or advance phases without approval.
```

## 12. Output Requirements

Create **one complete Markdown document**, named:

`GAME_NAME_UNITY_GAME_PLAN.md`

Use uppercase letters and underscores in the filename. The finished document must contain:

1. Researched original-game overview and sources
2. New game title, elevator pitch, and original identity
3. Concrete gameplay requirements and defaults
4. Unity version, packages, rendering and input choices
5. Scenes, prefabs, components, scripts, data, and dependency architecture
6. Actual directory structure and Inspector integration requirements
7. All detailed implementation phases (approximately 8–12)
8. Acceptance tests and approval gate for every phase
9. Complete game-specific `TODO.md` template
10. Testing procedures, test commands, and honest verification rules
11. Performance targets and profiling plan
12. Engine-specific and design-specific risks
13. Local coding agent workflow and context-recovery strategy
14. Intellectual-property and asset guidelines
15. Release and standalone build criteria
16. Full master implementation prompt
17. Short recovery prompt
18. Source references

**Write the actual plan, not merely an outline.** Choose practical mechanics and make reasonable assumptions explicit. It must guide a smaller model from a blank Unity project to a finished playable game without requiring unrelated design choices at every step.

## 13. Final Delivery

- Save the generated game-specific plan as a downloadable Markdown file.
- **Do not create the Unity game itself.**
- Do not create separate phase documents.
- **Do not create `TODO.md` yet.** The implementation agent creates it later.
- Provide the file link, original reference, original/new title, phase count, MVP phase, key features, and recommended first Qwen prompt.

**Begin by researching the supplied game URL. Then generate the complete plan without unnecessary clarification questions.**
