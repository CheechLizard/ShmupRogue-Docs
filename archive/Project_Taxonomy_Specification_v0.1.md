# Project Taxonomy Specification v0.1

**Shmup Roguelike with Sand Physics**

---

## 1. Overview

This document defines the organizational taxonomy for a shmup roguelike featuring sand physics, modular weapons, and destructible environments. The architecture enforces strict separation between **Engine**, **Game**, and **Editor** layers, following industry best practices for maintainability and scalability.

### 1.1 Structural Hierarchy

The game uses a consistent hierarchy for runtime content:

- **Level** — A complete unit containing multiple stages (e.g., Level 1: Tutorial)
- **Stage** — A segment within a level (Arena, Corridor, Boss, Store)
- **Mission** — A linear sequence of objectives tied to a stage
- **Objective** — A single completable goal within a mission

### 1.2 Technology Tiers

Five technology tiers gate weapon capabilities, visual effects, and progression. Each tier has distinct aesthetics and mechanical identity:

- **Steam** — Smoke, sparks, pistons, pressure bursts. Industrial, mechanical feel.
- **Electronic** — Arcs, glows, static, pulse waves. Crackling energy, circuit aesthetics.
- **Photonic** — Lasers, prisms, beams, light diffraction. Clean, precise, brilliant.
- **Organic** — Acid, spores, tendrils, biomass. Living, corrupted, Giger-esque.
- **Quantum** — Black holes, distortion, phasing, probability clouds. Reality-bending.

---

## 2. Design Principles

1. **Strict Layer Separation** — Engine knows nothing about Game; Game depends on Engine; Editor depends on both but neither depends on Editor.

2. **Hybrid ECS Architecture** — Pure ECS for hot-path entities (particles, projectiles, enemies); domain objects for cold-path singletons (player logic, world state, UI).

3. **Multi-Physics Coordination** — Sand, rigid body, and flow field systems run simultaneously with clean handoff interfaces.

4. **Data-Driven Configuration** — JSON for all asset definitions; no executable code in data files.

5. **Entity Symmetry** — NPCs and players follow identical rules via shared component types, enabling emergent behaviors.

6. **Tech Tier Gating** — Module availability and effect libraries are gated by technology tier, providing progression depth and visual consistency.

---

## 3. Top-Level Structure

| Directory | Purpose |
|-----------|---------|
| **Engine/** | Platform-agnostic systems: physics, rendering, audio, input, resources. No game-specific logic. |
| **Game/** | Game-specific logic: entities, weapons, economy, world, progression. Depends on Engine. |
| **Editor/** | Development tools: weapon editor, level editor, mission editor, debug tools. Separate application. |
| **Shared/** | Constants, enums, interfaces, and data structures used across all layers. |

---

## 4. Engine Layer

The Engine layer provides platform-agnostic infrastructure. It has no knowledge of game-specific concepts like weapons, enemies, or progression.

### 4.1 Core

| Module | Responsibility |
|--------|----------------|
| Math/ | Vector operations, matrix transforms, random number generation, spatial hashing. |
| Memory/ | Custom allocators, object pools, arena allocators for particle systems. |
| Events/ | Event bus, message queuing, observer pattern implementation. |
| Serialization/ | JSON parsing, binary serialization, schema validation. |

### 4.2 Physics

Three physics systems operate simultaneously. The **PhysicsCoordinator** manages transitions between systems (e.g., projectile hits terrain, spawns sand particles, particles enter gravity well).

| Module | Responsibility |
|--------|----------------|
| PhysicsCoordinator/ | Routes entities between physics systems; handles transition events and handoffs. |
| Sand/ | Cellular automata simulation, material state transitions (solid, particle, liquid, gas, plasma), terrain collision. |
| RigidBody/ | Collision detection, force application, constraints, projectile dynamics. |
| FlowField/ | Vector field generation (gravity wells, wind, currents), particle advection through fields. |

### 4.3 Rendering

| Module | Responsibility |
|--------|----------------|
| RenderPipeline/ | Draw call batching, render order management, frame timing. |
| Sprites/ | Sprite sheets, animation state machines, texture atlases. |
| Particles/ | GPU particle rendering, instanced drawing for sand/effects. |
| Shaders/ | CRT emulation, post-processing, material-specific rendering. |
| Camera/ | Viewport management, screen shake, smooth following, zoom. |

### 4.4 Audio

| Module | Responsibility |
|--------|----------------|
| AudioManager/ | Channel management, volume control, 3D audio positioning. |
| SFX/ | Sound effect pooling, variation selection, priority queuing. |
| Music/ | Adaptive music, crossfading, beat synchronization. |

### 4.5 Input

| Module | Responsibility |
|--------|----------------|
| InputManager/ | Input polling, action mapping, input buffering. |
| Bindings/ | Rebindable controls, profiles, conflict detection. |
| Devices/ | Keyboard, gamepad, touch input abstraction. |

### 4.6 Resources

| Module | Responsibility |
|--------|----------------|
| AssetLoader/ | Async loading, dependency resolution, hot reloading (dev mode). |
| Pooling/ | Object pools for entities, particles, projectiles. |
| Caching/ | LRU caches, texture caching, computed data caching. |

---

## 5. Game Layer

The Game layer contains all game-specific logic. It uses a **hybrid architecture**: pure ECS for high-volume entities (particles, projectiles, enemies) and domain objects for low-volume, complex state (player, world, UI).

### 5.1 ECS Core

Custom ECS implementation optimized for cache-friendly iteration of physics-heavy entities.

| Module | Responsibility |
|--------|----------------|
| EntityManager/ | Entity ID generation, component storage, archetype management. |
| Components/ | Component definitions: Transform, Velocity, Health, Damage, PhysicsBinding, etc. |
| Systems/ | System definitions: MovementSystem, CollisionSystem, DamageSystem, etc. |
| Queries/ | Component query builders, filtered iteration, spatial queries. |

### 5.2 Entities

Entity archetypes define component bundles. Hot-path entities are pure ECS; cold-path entities use domain wrappers that hold entity IDs.

| Category | Path | Notes |
|----------|------|-------|
| Player/ | Cold | Domain object wrapping ECS entity. Complex state, input handling, weapon switching. |
| NPCs/Enemies/ | Hot | Pure ECS. Spawned in bulk, identical component sets to player for entity symmetry. |
| NPCs/Harvesters/ | Hot | Pure ECS. NPC harvester mechanic entities. |
| NPCs/Merchants/ | Cold | Domain object. Stateful, inventory management. |
| Items/Pickups/ | Hot | Pure ECS. High volume, simple behavior. |
| Items/Materials/ | Hot | Pure ECS. Five states: solid, particle, liquid, gas, plasma. |
| Items/Artifacts/ | Cold | Domain object. Black box recipe system, complex discovery logic. |
| Projectiles/ | Hot | Pure ECS. Thousands per frame, cache-critical. |
| Structures/ | Hot | Pure ECS. Destructible terrain elements, debris. |
| Interactables/ | Cold | Domain object. Portals, stores, stateful interactions. |

### 5.3 Weapons

Implements the Modular Weapon System (see separate specification). Weapons are designer-created definitions containing a directed graph of modules. Each weapon belongs to a technology tier, which gates available modules and effects.

#### 5.3.1 Weapon Definition

| Module | Responsibility |
|--------|----------------|
| WeaponDefinition/ | Top-level weapon data: name, icon, description, tech tier, graph reference. |
| WeaponRegistry/ | Loads and indexes all weapon definitions; provides lookup by ID or tier. |

#### 5.3.2 Module Graph

| Module | Responsibility |
|--------|----------------|
| ModuleGraph/ | Graph construction, validation, traversal, energy calculation. |
| Modules/Battery/ | Provides energy per second to downstream path. |
| Modules/Trigger/ | Starts strikes from player input or conditions (press, hold, on_fire, on_target, on_no_target). |
| Modules/Repeater/ | Converts held firing into repeated strikes at fixed cadence. |
| Modules/Barrel/ | Sets aim, spread, range; routes to multiple outputs. |
| Modules/Striker/ | Simulates strike (projectile, beam, melee, field). Tech tier gates special types. |
| Modules/Payload/ | Applies damage and triggers effects. References effect library by name. |
| Events/Fire/ | Universal strike progression event. Contains StrikeState, spawn position, optional target. |
| Events/Release/ | Ends weapon-domain strikes (beams, melee). No payload. |

*Note: StrikeState is a data structure (defined in Shared/DataStructures/) carried by Fire events through the graph—not a separate subsystem.*

#### 5.3.3 Effects System

Effects are data-driven (JSON) and organized by technology tier. Designers select from tier-appropriate effects and can tweak parameters (color, scale, intensity) per weapon.

| Module | Responsibility |
|--------|----------------|
| EffectLibrary/ | Loads effect definitions from data/effects/. Validates tier requirements. |
| EffectInstance/ | Runtime effect with parameter overrides from weapon definition. |
| TierEffects/Steam/ | Smoke, sparks, pistons, pressure bursts. |
| TierEffects/Electronic/ | Arcs, glows, static, pulse waves. |
| TierEffects/Photonic/ | Lasers, prisms, beams, light diffraction. |
| TierEffects/Organic/ | Acid, spores, tendrils, biomass. |
| TierEffects/Quantum/ | Black holes, distortion, phasing, probability clouds. |

### 5.4 Economy

| Module | Responsibility |
|--------|----------------|
| Materials/ | Material definitions, state transition rules, physical properties. |
| Crafting/ | Recipe system, artifact discovery, black box experimentation. |
| TechTiers/ | Tier definitions, unlock requirements, capability gating for weapons and effects. |
| Harvesting/ | Collection mechanics, NPC harvester timing, risk/reward calculations. |

### 5.5 World

| Module | Responsibility |
|--------|----------------|
| LevelManager/ | Level loading, unloading, transition management. |
| Arenas/ | Two-directional toroidal zones for resource collection. |
| Corridors/ | One-directional linear segments with bosses and stores. |
| TerrainGeneration/ | Procedural terrain, destructible layers, material distribution. |
| Portals/ | Portal construction, fuel requirements, warp mechanics. |

### 5.6 Mission

Handles linear objective sequences per stage. Missions are simple chains; no branching or prerequisites.

| Module | Responsibility |
|--------|----------------|
| ObjectiveTracker/ | Tracks current objective index, completion state, progress counters. |
| TriggerEval/ | Evaluates completion conditions: destroy count, collect amount, reach location, etc. |
| HintDisplay/ | Shows current objective text, progress indicators, completion feedback. |

### 5.7 Systems (Game-Specific)

| Module | Responsibility |
|--------|----------------|
| Combat/ | Damage resolution, hit detection coordination, death handling. |
| Progression/ | Tech tier unlocks, permanent upgrades, meta-progression. |
| Spawning/ | Wave management, spawn patterns, difficulty scaling. |
| AI/ | Behavior trees, steering behaviors, threat assessment. |

### 5.8 State

| Module | Responsibility |
|--------|----------------|
| GameState/ | Global game state machine (menu, playing, paused, etc.). |
| SaveLoad/ | Serialization, save file management, versioning. |
| RunState/ | Current roguelike run tracking: seed, progress, accumulated resources. |

---

## 6. Shared Layer

Code and definitions used across Engine, Game, and Editor. Provides type safety and consistency.

| Module | Contents |
|--------|----------|
| Constants/ | Physics constants, timing values, balance numbers. |
| Enums/ | MaterialState, TechTier, DamageType, MotionType, ObjectiveType, TriggerCondition, etc. |
| Interfaces/ | IPhysicsBody, IRenderable, ISerializable, etc. |
| DataStructures/ | StrikeState, WeaponDefinition, ModuleGraph, EffectDefinition, MissionDefinition schemas. |

---

## 7. Editor

The editor is a standalone React/TypeScript web application. See **Editor_UI_Specification_v0_2.md** for UI architecture, information hierarchy, and implementation details.

---

## 8. Data Flow

### 8.1 Editor to Game Pipeline

Editor exports JSON files. Game loads JSON at runtime or startup. No executable code in data files ensures clean separation and prevents injection risks.

### 8.2 Physics Handoff Example

Projectile (RigidBody) hits terrain (Sand), spawns particles, particles enter gravity well (FlowField):

1. RigidBody system detects collision, emits PhysicsCollision event
2. PhysicsCoordinator receives event, identifies terrain material
3. Sand system spawns particles based on material type and impact force
4. Particles receive FlowFieldAffected component if within field radius
5. FlowField system advects affected particles each frame

### 8.3 Weapon Fire Example

Player fires a Quantum-tier weapon with black hole payload:

1. WeaponDefinition loaded, tech tier validated as Quantum
2. TRIGGER emits Fire event with fresh StrikeState
3. Fire flows through graph: BATTERY (energy) → BARREL (aim) → STRIKER (projectile)
4. STRIKER creates projectile entity, detaches to Inflight Domain
5. On impact, STRIKER emits Fire to PAYLOAD with target info
6. PAYLOAD triggers 'black_hole' effect from Quantum tier library
7. EffectInstance applies designer's parameter tweaks (scale, color, intensity)

### 8.4 Mission Flow Example

Tutorial stage objective sequence:

1. Stage loads, MissionSystem reads mission JSON for current stage
2. ObjectiveTracker initializes at index 0: "Destroy 3 training drones"
3. HintDisplay shows objective text with progress (0/3)
4. TriggerEval listens for enemy death events, increments counter
5. On completion, ObjectiveTracker advances to next objective
6. When all objectives complete, stage transition triggers

---

## 9. Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2026-01-02 | Initial specification. Hybrid ECS architecture, three-layer separation, multi-physics coordination, tech tier system, weapon/effect hierarchy, mission system. |
| 0.1.1 | 2026-01-11 | Removed Section 6 (Editor Layer) — now documented in Editor_UI_Specification_v0_2.md. |
