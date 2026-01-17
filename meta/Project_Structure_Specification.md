# Project Structure Specification v0.1

## Overview

This document defines the organizational structure for Shmup Rogue's documentation and editor. The structure mirrors the conceptual hierarchy of the game, ensuring consistency between specs, editor UI, and mental model.

---

## Conceptual Hierarchy

```
GAME (root)
│
├── WORLD (materials, biomes, terrain)
│
├── LEVELS (arenas, corridors, missions, story)
│
├── ENTITIES (player, NPCs, drones, weapons, items, structures)
│
├── SYSTEMS (combat, harvesting, economy, progression, physics, portals)
│
├── INTERFACE (HUD, menus, popups)
│
└── ENGINE (rendering, audio, input, assets)
```

Game-wide overviews (Game_Overview, Progression_Overview) live at docs root.
Level overviews live inside their level folder.

---

## Documentation Structure

### Folder Layout

```
/docs
├── Game_Overview.md                        ← Game-wide vision
├── Progression_Overview.md                 ← Game-wide progression
│
├── /world
│   ├── Biomes.md
│   ├── Materials.md
│   ├── Visual_Style.md
│   └── Terrain_Chunks.md
│
├── /levels
│   ├── /solar_system
│   │   ├── Solar_System_Overview.md        ← Level vision lives with content
│   │   ├── Arena_Generation.md
│   │   ├── Planets.md
│   │   ├── Sol_Chunk.md
│   │   └── Corridors.md
│   └── /underwater_world (future)
│       └── Underwater_World_Overview.md
│
├── /entities
│   ├── Player_Ship.md
│   ├── /npcs
│   │   ├── Solar_Dragon.md
│   │   └── Corridor_Bosses.md
│   ├── Drones.md
│   ├── Weapons.md
│   ├── Items.md
│   └── Structures.md
│
├── /systems
│   ├── Combat_Pressure.md
│   ├── Fuel.md
│   ├── Harvesting.md
│   ├── Economy.md
│   ├── Progression.md
│   ├── Portal_Stitching.md
│   └── Celestial_Upgrades.md
│
├── /interface
│   ├── HUD_Minimap.md
│   ├── HUD_Fuel_Gauge.md
│   └── Menus.md
│
├── /engine
│   ├── Physics.md
│   ├── Rendering.md
│   ├── Audio.md
│   └── Input.md
│
└── /meta
    ├── Project_Structure.md (this doc)
    ├── Architecture_Overview.md
    ├── Glossary.md
    └── Editor_UI.md
```

### Doc Types

| Type | Purpose | Location |
|------|---------|----------|
| **Overview** | Vision, feel, high-level design | Root level or inside level folder |
| **Specification** | Detailed requirements, parameters | Respective category folder |
| **Entity Spec** | Single entity definition | `/entities/` |
| **Level Spec** | Level-specific content | `/levels/[level_name]/` |
| **Meta** | Project organization docs | `/meta/` |

---

## Editor Structure

The editor UI mirrors this hierarchy. Each top-level section maps to a doc folder.

### Editor IA

```
WORLD
├── Biomes
├── Materials
├── Visual Style
└── Terrain / Chunks

LEVELS
├── Levels
│   └── Stages (arena/corridor/boss/store)
├── Missions
└── Story

ENTITIES
├── Player Ship
├── NPCs
├── Drones
├── Weapons
├── Items
└── Structures

SYSTEMS
├── Combat
├── Harvesting
├── Economy
├── Progression
├── Physics
└── Portals

INTERFACE
├── HUD
├── Menus
└── Popups

ENGINE
├── Rendering
├── Audio
├── Input
└── Assets
```

### Editor → Doc Mapping

| Editor Section | Doc Folder | Notes |
|----------------|------------|-------|
| WORLD | `/world/` | 1:1 mapping |
| LEVELS | `/levels/` | Overview lives inside level folder |
| ENTITIES | `/entities/` | 1:1 mapping |
| SYSTEMS | `/systems/` | Reusable mechanics |
| INTERFACE | `/interface/` | 1:1 mapping |
| ENGINE | `/engine/` | 1:1 mapping |
| — | `/meta/` | Project docs, not game content |
| — | Root `/docs/` | Game-wide overviews |

---

## Layer Definitions

### OVERVIEWS

Vision documents that set direction. Game-wide overviews live at root; level overviews live inside their level folder.

| Doc | Location | Purpose |
|-----|----------|---------|
| Game_Overview | `/docs/` | Pillars, core loop, references |
| Progression_Overview | `/docs/` | Meta-progression, difficulty curve |
| [Level]_[Level]_Overview | `/docs/levels/[level]/` | Per-level vision, win condition, feel |

### WORLD

The universe's building blocks. Shared across all levels.

| Section | Content |
|---------|---------|
| Biomes | Material palettes, reactivities, color language |
| Materials | 64 materials (8 biomes × 8), state transitions |
| Visual Style | Origin/age/state framework, palettes, shaders |
| Terrain / Chunks | Tile-edited terrain pieces |

### LEVELS

Playable content organized by level, then by stage type.

| Section | Content |
|---------|---------|
| Levels | Container for stages, level-wide params |
| Stages | Arena, Corridor, Boss, Store — graphs of chunks |
| Missions | Objective sequences per stage |
| Story | Lore nodes (text + images) |

### ENTITIES

Things that exist in the world.

| Section | Content |
|---------|---------|
| Player Ship | Movement, hitbox, starting loadout |
| NPCs | Enemies, harvesters, merchants — behavior, stats |
| Drones | Types, behaviors, summon/cooldown |
| Weapons | Module graphs (Battery → Trigger → Barrel → Striker → Payload) |
| Items | Pickups, artifacts, collectibles |
| Structures | Doors, traps, portals, black holes, spawn points |

### SYSTEMS

How things work. Reusable mechanics across levels.

| Section | Content |
|---------|---------|
| Combat | Damage resolution, spawning, pressure scaling |
| Harvesting | Bar mechanics, purity, crates |
| Economy | Stores, pricing, currency conversion |
| Progression | Tech tiers, unlocks, meta-progression |
| Physics | Sand, rigid body, flow field parameters |
| Portals | Chunk stitching, directional entry |

### INTERFACE

Player-facing UI.

| Section | Content |
|---------|---------|
| HUD | Harvest bar, health, fuel, drone status, minimap |
| Menus | Pause, shop UI, inventory, settings |
| Popups | Notifications, tutorials, dialogs |

### ENGINE

Technical implementation. Mostly code, minimal editor exposure.

| Section | Content |
|---------|---------|
| Rendering | CRT shader tuning, camera params |
| Audio | Adaptive music rules, mixing |
| Input | Bindings, device support |
| Assets | Sprites, audio files, fonts, import/export |

---

## Naming Conventions

### Docs

```
[Category]_[Name]_Specification_v[Major]_[Minor].md

Examples:
- Materials_Specification.md
- Solar_System_Overview.md
- NPC_Solar_Dragon.md
```

### Editor Items

```
[Type]_[Name]

Examples:
- Arena_AsteroidBelt_01
- NPC_SolarDragon
- Weapon_PlasmaCannon
```

---

## Cross-References

Docs should reference related docs explicitly:

```markdown
## Related Specifications

- `Materials_Specification.md` — Material properties
- `System_Combat.md` — Spawning rules
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2025-01-15 | Initial specification |
