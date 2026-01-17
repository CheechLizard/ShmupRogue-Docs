# Architecture Overview

This document defines the architectural principles and top-level structure for Shmup Rogue.

---

## Design Principles

1. **Strict Layer Separation** — Engine knows nothing about Game; Game depends on Engine; Editor depends on both but neither depends on Editor.

2. **Hybrid ECS Architecture** — Pure ECS for hot-path entities (particles, projectiles, enemies); domain objects for cold-path singletons (player logic, world state, UI).

3. **Multi-Physics Coordination** — Sand, rigid body, and flow field systems run simultaneously with clean handoff interfaces.

4. **Data-Driven Configuration** — JSON for all asset definitions; no executable code in data files.

5. **Entity Symmetry** — NPCs and players follow identical rules via shared component types, enabling emergent behaviors.

6. **Tech Tier Gating** — Module availability and effect libraries are gated by technology tier, providing progression depth and visual consistency.

---

## Top-Level Structure

| Directory | Purpose |
|-----------|---------|
| **Engine/** | Platform-agnostic systems: physics, rendering, audio, input, resources. No game-specific logic. |
| **Game/** | Game-specific logic: entities, weapons, economy, world, progression. Depends on Engine. |
| **Editor/** | Development tools: weapon editor, level editor, mission editor, debug tools. Separate application. |
| **Shared/** | Constants, enums, interfaces, and data structures used across all layers. |

---

## Shared Layer

Code and definitions used across Engine, Game, and Editor. Provides type safety and consistency.

| Module | Contents |
|--------|----------|
| Constants/ | Physics constants, timing values, balance numbers. |
| Enums/ | MaterialState, TechTier, DamageType, MotionType, ObjectiveType, TriggerCondition, etc. |
| Interfaces/ | IPhysicsBody, IRenderable, ISerializable, etc. |
| DataStructures/ | StrikeState, WeaponDefinition, ModuleGraph, EffectDefinition, MissionDefinition schemas. |

---

## Data Flow Examples

### Editor to Game Pipeline

Editor exports JSON files. Game loads JSON at runtime or startup. No executable code in data files ensures clean separation and prevents injection risks.

### Physics Handoff

Projectile (RigidBody) hits terrain (Sand), spawns particles, particles enter gravity well (FlowField):

1. RigidBody system detects collision, emits PhysicsCollision event
2. PhysicsCoordinator receives event, identifies terrain material
3. Sand system spawns particles based on material type and impact force
4. Particles receive FlowFieldAffected component if within field radius
5. FlowField system advects affected particles each frame

### Weapon Fire

Player fires a Quantum-tier weapon with black hole payload:

1. WeaponDefinition loaded, tech tier validated as Quantum
2. TRIGGER emits Fire event with fresh StrikeState
3. Fire flows through graph: BATTERY (energy) → BARREL (aim) → STRIKER (projectile)
4. STRIKER creates projectile entity, detaches to Inflight Domain
5. On impact, STRIKER emits Fire to PAYLOAD with target info
6. PAYLOAD triggers 'black_hole' effect from Quantum tier library
7. EffectInstance applies designer's parameter tweaks (scale, color, intensity)

### Mission Flow

Tutorial stage objective sequence:

1. Stage loads, MissionSystem reads mission JSON for current stage
2. ObjectiveTracker initializes at index 0: "Destroy 3 training drones"
3. HintDisplay shows objective text with progress (0/3)
4. TriggerEval listens for enemy death events, increments counter
5. On completion, ObjectiveTracker advances to next objective
6. When all objectives complete, stage transition triggers

---

## Structural Hierarchy

The game uses a consistent hierarchy for runtime content:

| Term | Definition |
|------|------------|
| **Level** | A complete unit containing multiple stages (e.g., Level 1: Tutorial) |
| **Stage** | A segment within a level (Arena, Corridor, Boss, Store) |
| **Mission** | A linear sequence of objectives tied to a stage |
| **Objective** | A single completable goal within a mission |

See also: `/meta/Glossary.md`

---

## Related Documentation

- `/engine/Engine_Overview.md` — Engine layer details
- `/entities/Entities_Overview.md` — Entity architecture
- `/systems/Systems_Overview.md` — Game systems index
- `/meta/Editor_UI_Specification.md` — Editor architecture

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2025-01-17 | Extracted from Project_Taxonomy_Specification_v0.1 §2-3, §6, §8. |
