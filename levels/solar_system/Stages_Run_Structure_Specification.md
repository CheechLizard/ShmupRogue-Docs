# Stages & Run Structure Specification v0.1

## Overview

The game is a journey through a procedurally generated universe. Each run feels familiar yet fresh—players learn the vocabulary of the world while encountering new arrangements, challenges, and opportunities.

---

## Goal

Varied pacing with reasons to replay.

---

## Terminology

| Term | Definition |
|------|------------|
| **Level** | One complete loop: Arena → Corridor → Boss → Store |
| **Stage** | One part of a Level (Arena, Corridor, Boss, or Store) |
| **Segment** | An authored building block used to construct stages |
| **Graph** | The abstract structure defining how segments connect within a stage |

---

## Design Principles

**Space Feels Expansive** — The player should rarely feel boxed in. When in doubt, go bigger.

**Construction as Progression** — Players spend resources to build permanent portals. Once built, portals stay open for the remainder of the run.

**Familiar but Fresh** — Authored segments provide hand-crafted quality; graph-based assembly creates variety between runs.

---

## Level Structure

A **Level** is one iteration of the core gameplay loop:

```
┌─────────┐    ┌──────────┐    ┌────────┐    ┌─────────┐
│  ARENA  │───▶│ CORRIDOR │───▶│  BOSS  │───▶│  STORE  │───▶ [Next Level]
└─────────┘    └──────────┘    └────────┘    └─────────┘
     │                                             │
     └─────────── backtrack anywhere ─────────────┘
```

---

## Run Structure

- **Target run length:** ~25 minutes
- **Levels** increase in difficulty and complexity
- **Backtracking** unlimited between unlocked stages
- **Portals** persist for the run (roguelike reset between runs)
- **Endless mode:** Planned—escalating difficulty and portal costs

---

## Stage Types

### ARENA

Open free-roaming spaces for resource collection, combat, and exploration.

**Purpose:** Gather resources, defeat enemies, discover secrets, unlock portal to Corridor.

**Movement:** Player-controlled in all directions. Camera follows player. Space wraps seamlessly at edges (toroidal).

**Structure:** Single segment defines the space. Future iterations may compose multiple segments.

**Unlock condition:** Event completion (TBD—placeholder trigger for milestone).

| Property | Description |
|----------|-------------|
| `environment_type` | Reference to environment (affects mechanics and visuals) |
| `dimensions` | Width × Height |
| `portal_cost` | Resources required to construct exit portal |
| `segment_id` | Reference to the segment used |

---

### CORRIDOR

Linear spine-and-branch spaces connecting Arenas to Bosses.

**Purpose:** Navigate from entrance to boss, with optional side branches offering risk/reward choices.

**Movement:** Player-controlled with directional bias. Backtracking allowed.

**Structure:** Directed graph with a guaranteed main path (spine) and optional branch paths.

```
[Entrance] ──▶ [Segment A] ──▶ [Segment B] ──▶ [Boss Gate]
                   │                              ▲
                   ▼                              │
              [Branch 1] ─────────────────────────┘
                   │
                   ▼
              [Secret Area]
```

**Spine:** The main path from entrance to boss. Always traversable. Consistent length per difficulty tier (Dead Cells model).

**Branches:** Optional paths that diverge from spine:
- Resources, weapon modules, or upgrades
- Hazards or challenging encounters
- May reconnect to spine (loop) or dead-end with reward
- Risk (time, resources, danger) vs. reward (items that change your run)

| Property | Description |
|----------|-------------|
| `spine_length` | Number of segments in main path |
| `branch_count` | Number of branch points |
| `branch_depth` | Maximum segments per branch |
| `environment_type` | Inherited from destination Arena |

---

### BOSS

Fixed encounter space for boss battles.

**Purpose:** Defeat the boss to unlock the Store.

**Movement:** Contained arena (may or may not wrap depending on boss design).

**Structure:** For milestone, single fixed segment per boss. Future iterations may support multi-phase bosses with stage transitions.

| Property | Description |
|----------|-------------|
| `boss_id` | Reference to boss encounter definition |
| `segment_id` | Reference to the arena segment |
| `dimensions` | Width × Height |

---

### STORE

Upgrade and loadout space between levels.

**Purpose:** Spend resources, acquire modules, modify weapon loadouts.

**Movement:** Minimal—UI/interaction focused.

**Structure:** For milestone, fixed UI space.

**Future:** Progression graph where choices unlock further options based on player build path. Start down an energy weapon path → more energy weapon choices appear, plus tempting rare options that are completely incompatible with your build.

| Property | Description |
|----------|-------------|
| `inventory_pool` | Available items/modules (milestone: fixed set) |
| `pricing_tier` | Cost scaling for current level |

---

## Segment System

Segments are the building blocks of stages. Each segment is a self-contained spatial unit with defined connection points.

### Segment Schema

```
Segment {
    id: String                    // Unique identifier
    name: String                  // Human-readable name
    environment_type: String      // Reference to environment
    stage_type: Enum              // Arena | Corridor | Boss

    // Dimensions
    width: Int                    // Segment width in units
    height: Int                   // Segment height in units

    // Connection points (for Corridor segments)
    connections: [Connection]     // Entry/exit points

    // Content
    terrain: TerrainData          // Solid obstacles, destructibles
    spawns: SpawnData             // Enemy, resource, hazard placements
    metadata: SegmentMetadata     // Tags, difficulty, theming
}

Connection {
    id: String                    // Connection point identifier
    edge: Enum                    // North | South | East | West
    position: Float               // 0.0-1.0 along edge
    direction: Enum               // In | Out | Bidirectional
    tags: [String]                // For matching compatible connections
}

SpawnData {
    enemies: [SpawnPoint]
    resources: [SpawnPoint]
    hazards: [SpawnPoint]
    portal_site: SpawnPoint?      // For Arena segments
}

SpawnPoint {
    position: Vector2
    type: String
    spawn_conditions: Conditions? // Optional trigger conditions
}

SegmentMetadata {
    difficulty: Float             // 0.0-1.0 difficulty rating
    tags: [String]                // Theming, biome, special features
    author: String                // For tracking authorship
    version: Int                  // Schema version
}
```

### Segment Types by Stage

**Arena segments:**
- Single large segment defines the toroidal space
- No connection points (edges wrap)
- Contains portal construction site

**Corridor segments:**
- Smaller segments with 1-4 connection points
- Tags ensure compatible connections (e.g., "wide_passage" connects to "wide_passage")

**Boss segments:**
- Single fixed segment per boss
- Designed around specific boss mechanics

---

## Procedural Generation

### Arena Generation (Milestone)

1. Select segment from pool matching `environment_type`
2. Place segment as the toroidal space
3. Populate spawn points from segment data
4. Place portal construction site

### Corridor Generation

**Input:**
- `spine_length`: target number of spine segments
- `branch_probability`: chance of branch at each spine node
- `max_branch_depth`: maximum branch length
- `environment_type`: filter for segment selection

**Algorithm:**

```
1. Generate spine:
   - Create linear sequence of `spine_length` segment nodes
   - Select segments from pool, matching connection compatibility
   - First node is Entrance, last connects to Boss Gate

2. Add branches:
   - For each spine node (except first/last):
     - Roll against `branch_probability`
     - If branch: generate branch segments up to `max_branch_depth`
     - Branch may reconnect downstream or dead-end with reward

3. Validate:
   - Ensure main path is traversable
   - Check all connections are satisfied

4. Instantiate:
   - Place segments spatially based on connection points
   - Populate spawn points
   - Set up transition triggers
```

### Seed Determinism

Each stage uses a seed for reproducible generation:
- Same seed → identical segment selection and placement
- Same seed → identical spawn point locations

Enemy and resource *timing* may vary based on player actions, but *locations* are fixed.

---

## Portal Construction

Players spend resources to build portals at procedurally-placed construction sites.

- **Cost** scales with progression—early portals cheap, later portals require rare resources
- **Permanent** for the run once built
- **Locations** marked on minimap once discovered

*(Resource types and costs defined in Resource Economy Specification.)*

---

## Authoring Pipeline

Segments are created through AI-assisted authoring:

1. **Generation:** Use AI tools to generate segment concepts and initial layouts
2. **Refinement:** Import to editor, adjust parameters, test gameplay
3. **Validation:** Verify connection points, difficulty rating, spawn balance
4. **Export:** Save to segment library in schema format

---

## Milestone Scope

### In Scope
- Arena stage: single segment, toroidal wrap
- Corridor stage: spine-and-branch graph generation
- Boss stage: fixed segment, placeholder encounter
- Store stage: fixed UI, placeholder inventory
- Segment schema defined
- Basic segment library for testing
- Portal construction with placeholder costs
- Full backtracking

### Deferred
- Store progression graph
- Multi-segment Arena composition
- Complex Boss stage transitions
- Meta-progression unlocks
- Endless mode

---

## Open Questions

1. **Arena unlock events:** What triggers portal site activation?
2. **Corridor branch rewards:** What makes branches worth the risk?
3. **Connection matching:** How strict should tag matching be?
4. **Difficulty scaling:** How do segment ratings combine with level progression?

---

## System Interactions

- **Environments & Terrain** — Stages reference environment types; terrain data lives in segments
- **Resource Economy** — Portal costs, resource spawns, store pricing
- **Weapon System** — Store sells modules; branches may contain modules
- **NPCs & Enemies** — Spawn data in segments; boss encounters
- **Physics** — Terrain destruction affects stage traversal
