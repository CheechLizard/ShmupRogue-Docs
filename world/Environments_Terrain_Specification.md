# Environments & Terrain Specification v0.1

## Overview

Environments define where the player is and how that space behaves. Terrain fills those spaces with destructible matter that reshapes during play.

---

## Goal

Make each space feel distinct and destructible.

---

## Design Principles

**Progressive Complexity** — Early environments are open and simple. Later environments introduce terrain, hazards, and navigational challenges. Terrain becomes a "star of the show" as players progress.

**Destruction Has Consequences** — Terrain isn't decoration. Destroying it changes navigation, creates cover, triggers chain reactions, and reshapes the battlefield.

**Speed and Scale** — Fast multi-layer parallax backgrounds create velocity and scale; gameplay occurs in a controlled play space for consistent difficulty.

---

## Environment Types

Environments apply mechanical modifiers to stages. Each environment type has distinct physics, hazards, and visual identity.

### Space

**Role:** Baseline. Teaches core mechanics without spatial pressure.

**Mechanics:**
- No gravity
- No terrain obstacles
- Full wrap (all 4 edges)

**Terrain:** None (background only—star fields, nebulae, distant objects)

**Progression:** Early game

---

### Asteroid Field

**Role:** Introduces destruction and navigational complexity.

**Mechanics:**
- No gravity
- Moving asteroids cause collision damage
- Full wrap (all 4 edges)

**Terrain:** Floating destructible chunks. Some contain resources.

**Progression:** Mid game

---

### Atmosphere

**Role:** Highest navigational difficulty. Full terrain variety.

**Mechanics:**
- Gravity pulls toward ground
- Wind currents push player directionally
- Bottom edge is solid ground (no wrap); other 3 edges wrap

**Terrain:** Solid terrain with maze-like structures. Full material variety.

**Progression:** Late game

---

## Terrain System

All terrain is fully destructible using falling sand simulation.

### Why Sand Physics?

- **Player agency** — Blast your own path
- **Weapon integration** — AOE weapons reshape terrain
- **Emergent gameplay** — Chain reactions, collapsing structures
- **Visual spectacle** — Satisfying destruction

### Material Types

| Material | Behavior | Examples |
|----------|----------|----------|
| **Inert** | Destructible, stays destroyed | Rock, metal, ice |
| **Organic** | Regenerates over time after damage | Flesh walls, mould, membrane |
| **Liquid** | Flows, fills gaps, may spawn more | Water, lava, acid |
| **Hazard** | Damages on contact, may spread | Fire, plasma, radiation |

**Organic materials** (R-Type/Gradius style) create pressure—players must push through before it heals.

**Liquids** enable emergent interactions (lava meets water → creates rock).

### Terrain Progression

1. **Space (Early)** — No terrain. Open star fields. Teaches combat.
2. **Asteroid Field (Mid)** — Floating destructible chunks. Introduces destruction.
3. **Atmosphere (Late)** — Full sand physics, maze structures, material variety.

This pacing turns terrain into a reward—each new environment introduces new mechanics and possibilities.

---

## Technical Constraints

### Screen and Cell Dimensions

| Dimension | Value |
|-----------|-------|
| Screen resolution | 1280×720 pixels |
| Cell size | 2×2 pixels |
| Chunk size | 32×32 cells (64×64 pixels) |
| Arena dimensions | Defined per-level (see level specs). Must be power-of-2 for GPU sand physics. |

### Coordinate Systems

The camera system uses three coordinate spaces:

| Space | Description |
|-------|-------------|
| **World Space** | Where entities exist. Arenas wrap toroidally at bounds. |
| **Camera Space** | World offset by camera position. |
| **Screen Space** | Pixel coordinates. Player stays at screen center (640, 360). |

### Minimum Grain Size: 2×2 Pixels

Due to CRT shader color smearing, 1×1 particles are invisible. All sand/particles use 2×2 minimum. See `/world/Visual_Style_Specification.md` for details.

This also improves simulation performance vs. 1×1 approaches.

### Chunk System

*(Implementation detail—likely Noita-style chunk optimization for sand simulation. Details TBD.)*

Note: "Chunk" here refers to simulation regions, not level segments. Different system.

---

## Enemy-Terrain Interaction

Enemies use the same terrain physics as players:
- Mining through obstacles
- Burrowing into terrain
- Building or spawning terrain

*(Details in NPCs & Enemies Specification.)*

---

## Perspective & Collision Layers

The game's perspective isn't a fixed mode—it's controlled by the **flow field gravity** system. This enables smooth transitions between top-down and side-scrolling gameplay within a single level.

### Flow Field Gravity

Sand physics uses probabilistic flow field gravity rather than fixed directional gravity:

- **Strength (0.0–1.0):** Probability that each particle follows the gravity direction per tick
- **Direction:** Global pull direction (off, n, s, e, w, etc.)
- **Attractors:** Local gravity wells that pull particles inward (with optional inner orbit zone)

At `strength = 0.3`, particles drift gently. At `strength = 1.0`, gravity is deterministic.

### Perspective Gradient

| Flow Field State | Perspective | Feel |
|------------------|-------------|------|
| `strength = 0`, `dir = off` | Top-down | Free movement, no gravity |
| `strength = 0.3–0.7` | Transitional | Soft drift, partial gravity |
| `strength = 1.0`, `dir = s` | Side-scroll | Full "down" gravity |

This gradient enables seamless transitions:
- Player flies through space (top-down)
- Approaches planet, gravity ramps up
- Enters atmosphere (side-scroll)
- Can escape back to space anytime

### Collision Layer Rules

Perspective changes how entities relate to terrain:

| Perspective | Entity-Terrain Relationship | Collision Rules |
|-------------|----------------------------|-----------------|
| **Top-down** | Entities fly OVER sand | Only gravity weapons (bombs) hit terrain. Bullets pass over. |
| **Side-scroll** | Entities IN SAME PLANE as sand | All weapons and entities collide with terrain. |

**Why this matters:**
- Top-down: Sand is a landscape below—fly over asteroids, shoot enemies above them
- Side-scroll: Sand is obstacles—navigate around, use cover, terrain blocks shots

### Local Attractors

Attractors create localized gravity wells independent of global direction:
- **Black holes:** Pull everything inward (top-down feels natural)
- **Artifact singularity:** Orbiting debris, dramatic visual
- **Multiple attractors:** Complex gravity landscapes

Attractors work in any perspective—they're additive to global gravity.

### Future Work

- **Player/NPC gravity:** Entities should feel the same flow field as sand (TODO)
- **Shadows:** In top-down, entities cast shadows on terrain below
- **Rigid obstacles:** Flight-layer obstacles that block movement but aren't sand

---

## Open Questions

1. **Simulation bounds:** How large can destructible terrain areas be before performance suffers?
2. **Material interactions:** Full interaction matrix (what happens when X meets Y)?
3. **Regeneration rates:** How fast should organic terrain heal?
4. **Hazard spreading:** How aggressively do fire/plasma/etc. propagate?

---

## System Interactions

- **Stages & Run Structure** — Stages reference environment types; terrain data stored in segments
- **Physics** — Sand simulation is part of physics system
- **Weapon System** — Weapons destroy and reshape terrain
- **Visual Style** — Environment framework (Origin × Age × State) defines visual identity; 2×2 grain minimum
- **NPCs & Enemies** — Enemies interact with terrain
