# Engine Overview

The Engine layer provides platform-agnostic infrastructure. It has no knowledge of game-specific concepts like weapons, enemies, or progression.

---

## Core

| Module | Responsibility |
|--------|----------------|
| Math/ | Vector operations, matrix transforms, random number generation, spatial hashing. |
| Memory/ | Custom allocators, object pools, arena allocators for particle systems. |
| Events/ | Event bus, message queuing, observer pattern implementation. |
| Serialization/ | JSON parsing, binary serialization, schema validation. |

---

## Physics

Three physics systems operate simultaneously. The **PhysicsCoordinator** manages transitions between systems (e.g., projectile hits terrain, spawns sand particles, particles enter gravity well).

| Module | Responsibility |
|--------|----------------|
| PhysicsCoordinator/ | Routes entities between physics systems; handles transition events and handoffs. |
| Sand/ | Cellular automata simulation, material state transitions (solid, particle, liquid, gas, plasma), terrain collision. |
| RigidBody/ | Collision detection, force application, constraints, projectile dynamics. |
| FlowField/ | Vector field generation (gravity wells, wind, currents), particle advection through fields. |

See also:
- `/engine/physics/Sand_Physics_Streaming_Specification.md`
- `/engine/physics/Coordinate_System.md`

---

## Rendering

| Module | Responsibility |
|--------|----------------|
| RenderPipeline/ | Draw call batching, render order management, frame timing. |
| Sprites/ | Sprite sheets, animation state machines, texture atlases. |
| Particles/ | GPU particle rendering, instanced drawing for sand/effects. |
| Shaders/ | CRT emulation, post-processing, material-specific rendering. |
| Camera/ | Viewport management, screen shake, smooth following, zoom. |

See also:
- `/engine/rendering/Hierarchical_Stability_Texturing.md`
- `/engine/rendering/Tile_Atlas_Pipeline.md`

---

## Audio

| Module | Responsibility |
|--------|----------------|
| AudioManager/ | Channel management, volume control, 3D audio positioning. |
| SFX/ | Sound effect pooling, variation selection, priority queuing. |
| Music/ | Adaptive music, crossfading, beat synchronization. |

---

## Input

| Module | Responsibility |
|--------|----------------|
| InputManager/ | Input polling, action mapping, input buffering. |
| Bindings/ | Rebindable controls, profiles, conflict detection. |
| Devices/ | Keyboard, gamepad, touch input abstraction. |

---

## Resources

| Module | Responsibility |
|--------|----------------|
| AssetLoader/ | Async loading, dependency resolution, hot reloading (dev mode). |
| Pooling/ | Object pools for entities, particles, projectiles. |
| Caching/ | LRU caches, texture caching, computed data caching. |

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2025-01-17 | Extracted from Project_Taxonomy_Specification_v0.1 §4. |
