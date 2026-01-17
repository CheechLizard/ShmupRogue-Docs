# Editor UI Specification v0.2

**Shmup Roguelike with Sand Physics**

---

## 1. Overview

This document defines the user interface specification for the game editor. The editor is a standalone application for authoring game content and real-time parameter tuning. It connects to the running game via a bridge, allowing changes to be reflected immediately during play.

### 1.1 Design Context

The editor operates as one of three windows during development:

```
┌─────────────────────────┐     ┌─────────────────────────┐
│        VS CODE          │     │         EDITOR          │
│                         │     │                         │
│   Code, AI chat, F5     │     │  ┌─────┐    ┌────────┐ │
│                         │     │  │Stats│    │Inspector│ │
└───────────┬─────────────┘     │  │     │    │        │ │
            │                    │  │ FPS │    │        │ │
            │ launches           │  │ ... │    │        │ │
            ▼                    │  └─────┘    └────────┘ │
┌─────────────────────────┐     │                         │
│          GAME           │◄────┤  (bridge connection)    │
│    (separate window)    │────►│                         │
│                         │     └─────────────────────────┘
└─────────────────────────┘
```

**Primary workflow:** Chat with AI in VS Code → F5 to launch game → tweak parameters in editor (reflected in real-time) → iterate.

The editor connects to the game via bridge — it does not contain or embed the game. All three applications run as separate windows/executables.

### 1.2 Platform Targets

| Platform | Priority | Notes |
|----------|----------|-------|
| macOS (PWA) | Primary | Runs alongside VS Code and game |
| iPad | Future | Standalone authoring, GitHub sync, parked for later |

---

## 2. Technology Stack

| Layer | Choice | Rationale |
|-------|--------|-----------|
| Framework | React 18+ | Best AI training data, large ecosystem |
| Build | Vite | Fast HMR, modern tooling |
| Styling | Tailwind | AI generates fluently, utility classes |
| State | Zustand | Minimal boilerplate, TypeScript-friendly |
| Node Graphs | React Flow | Most popular, AI knows it, custom nodes are React components |
| Deployment | PWA | Browser + installable, works on Mac and iPad |

---

## 3. Information Architecture

The editor organizes content into seven top-level categories. This structure mirrors the project documentation hierarchy (see `Project_Structure_Specification.md`).

```
WORLD
├── Biomes (material palettes, reactivities, color language)
├── Materials (8 biomes × 8 materials = 64 total)
├── Visual Style (origin/age/state, palettes, shaders)
└── Terrain
    └── Chunks (tile-edited, unique per stage)

LEVELS
├── Levels (contain stages)
│   └── Stages (arena/corridor/boss/store graphs)
├── Missions (objective sequences)
└── Story (lore nodes — text + images)

ENTITIES
├── Player Ship (movement, hitbox, starting loadout)
├── NPCs (enemies, harvesters, merchants)
├── Drones (types, behaviors, summon/cooldown)
├── Weapons (module graphs)
├── Items (pickups, artifacts)
└── Structures (doors, traps, portals, black holes, spawn points)

SYSTEMS
├── Combat (damage, spawning, pressure scaling)
├── Harvesting (bar mechanics, purity, crates)
├── Economy (stores, pricing, currency)
├── Progression (tech tiers, unlocks, meta)
├── Physics (sand, rigid body, flow field params)
└── Portals (chunk stitching, directional entry)

INTERFACE
├── HUD (harvest bar, health, fuel, minimap)
├── Menus (pause, shop UI, inventory)
└── Popups (notifications, tutorials)

ENGINE
├── Rendering (CRT tuning, camera)
├── Audio (adaptive rules, mixing)
├── Input (bindings, devices)
└── Assets (sprites, audio files, fonts, import/export)
```

### 3.1 Mapping to Documentation

The editor structure maps directly to the documentation folder structure. See `Project_Structure_Specification.md` for the complete mapping between editor sections and doc folders.

---

## 4. Layout Modes

The editor has two layout modes:

### 4.1 Edit Mode

Standard authoring layout with navigation, canvas, and inspector.

```
┌──────────────────────────────────────────────────────────────────────────┐
│ ☰ │ World › Terrain › Chunks › Arena_01│              🎛️  📊 │ 🔍      │
├───┴──────────────────────────────────────────────────────┬───────────────┤
│                                                          │               │
│                                                          │  INSPECTOR    │
│                                                          │  (collapsible)│
│                        CANVAS                            │               │
│                   (view-dependent)                       │               │
│                                                          │               │
│                                                          │               │
├──────────────────────────────────────────────────────────┴───────────────┤
│ ✓ Saved │ status info                                                    │
└──────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Runtime Mode

Profiler panels at edges, resizable. Game runs in separate window (not embedded).

```
┌──────────────────────────────────────────────────────────────────────────┐
│                                                           [Pause][Step]  │
├────────┬─────────────────────────────────────────────────────────────────┤
│ FPS    │                                                                 │
│ ▁▂▃▄▅▆ │                                                                 │
│ 60     │                   (game runs separately)                        │
│        │                                                                 │
│ Physics│                                                                 │
│ ▁▂▃▄▅▆ │                                                                 │
│ 0.8ms  │                                                                 │
│    [◀▶]│                                                                 │
└────────┴─────────────────────────────────────────────────────────────────┘
```

---

## 5. Shell Components

### 5.1 Top Bar

```
┌──────────────────────────────────────────────────────────────────────────┐
│ ☰ │ Breadcrumb Path                     │              🎛️  📊 │ 🔍      │
└──────────────────────────────────────────────────────────────────────────┘
```

| Element | Behavior |
|---------|----------|
| ☰ Hamburger | Opens navigation panel (expanding list) |
| Breadcrumb | Clickable path, navigate up hierarchy |
| 🎛️ Inspector | Toggle inspector panel visibility |
| 📊 Profiler | Toggle profiler panels visibility (runtime mode) |
| 🔍 Search | Command palette (Cmd+K) |

### 5.2 Status Bar

Context-sensitive information bar. Shows different content based on current state.

**Default state:** Save status, warnings, context info
```
┌──────────────────────────────────────────────────────────────────────────┐
│ ✓ Saved │ 3 warnings │ Layer: Terrain                                    │
└──────────────────────────────────────────────────────────────────────────┘
```

**Parameter focused:** Shows description when user focuses a parameter (click or arrow keys)
```
┌──────────────────────────────────────────────────────────────────────────┐
│ spread_variance: Random variance added per shot in degrees (0-360)       │
└──────────────────────────────────────────────────────────────────────────┘
```

**Invalid action:** Shows reason when action cannot be completed
```
┌──────────────────────────────────────────────────────────────────────────┐
│ ⚠️ Cannot connect: STRIKER only accepts one input                        │
└──────────────────────────────────────────────────────────────────────────┘
```

| State | Content |
|-------|---------|
| Default | Save state (✓ Saved, ● Unsaved, ⚠️ Error), warning count, context |
| Parameter focused | Parameter name + description from schema |
| Invalid action | Warning icon + explanation |
| Validation error | Error details (clickable to navigate) |

### 5.3 Navigation Panel

Expanding list, two levels deep. Opens from hamburger.

```
┌─────────────────┐
│ ▼ World         │
│    Biomes       │
│    Materials    │
│    Visual Style │
│    Terrain      │
│ ▶ Levels        │
│ ▶ Entities      │
│ ▶ Systems       │
│ ▶ Interface     │
│ ▶ Engine        │
├─────────────────┤
│ ⚙️ Settings     │
└─────────────────┘
```

- Click ▶ to expand category
- Click ▼ to collapse category
- Click item to navigate (shows gallery grid)

### 5.4 Inspector Panel

Right side, collapsible, with expandable sections.

```
┌───────────────┐
│ INSPECTOR     │
├───────────────┤
│ ▼ General     │
│   Name: [...] │
│   Type: [...] │
├───────────────┤
│ ▼ Motion      │
│   Speed: [==] │
│   Accel: [==] │
├───────────────┤
│ ▶ Collision   │
├───────────────┤
│ ▶ Visual      │
├───────────────┤
│   (scroll)    │
└───────────────┘
```

- Sections collapse/expand independently
- Scrollable when content exceeds height
- Remember open/closed state per node type

---

## 6. Canvas Views

The canvas area changes based on content being edited.

### 6.1 Gallery Grid

Displayed when navigating to a category (L3). Shows all items as cards.

```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  thumbnail  │  │  thumbnail  │  │  thumbnail  │  │             │
│             │  │             │  │             │  │      +      │
│             │  │             │  │             │  │             │
├─────────────┤  ├─────────────┤  ├─────────────┤  ├─────────────┤
│ Item Name   │  │ Item Name   │  │ Item Name   │  │ New Item    │
└─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘
```

**Normal mode:**
- Click card to open item
- `+ New` card creates new item

**Edit mode** (toggle):
```
┌─────────────┐
│ [✓] thumb   │  ← checkbox for multi-select
│             │
├─────────────┤
│ [Name     ] │  ← click to rename inline
└─────────────┘
  ↕ drag to reorder

[Delete Selected (2)]                      [Done]
```

- Multi-select via checkboxes
- Inline rename (click name)
- Drag to reorder
- Delete selected items
- Implicit saving on all changes

### 6.2 Graph View

Node-based editor using React Flow. Used for weapons, level structure, story, material states, portal routing.

```
┌─────────────────────────────────────────────────────────────────┐
│                                                           [+]   │
│    ┌──────────┐      ┌──────────┐      ┌──────────┐            │
│    │  NODE A  │      │  NODE B  │      │  NODE C  │            │
│    │          │─────▶│          │─────▶│ ●selected│            │
│    │  params  ○      ○  params  ○      ○  params  │            │
│    └──────────┘      └──────────┘      └────┬─────┘            │
│                                             │                   │
│                                             ▼                   │
│                                        ┌──────────┐            │
│                                        │  NODE D  │            │
│                                        │          │            │
│                                        ○  params  │            │
│                                        └──────────┘            │
│                                                                 │
│         [minimap]                                               │
│         ┌───────┐                                               │
│         │ ● · · │                                               │
│         │ · · ● │                                               │
│         └───────┘                                               │
└─────────────────────────────────────────────────────────────────┘
```

#### Graph Interactions

| Action | Interaction |
|--------|-------------|
| Pan canvas | Drag on empty space, or Hand tool |
| Zoom | Scroll wheel, or pinch |
| Select node | Click node |
| Multi-select | Shift+click or marquee drag |
| Move node | Drag node body |
| Add node | Click `+` → floating picker appears near cursor |
| Delete node | Select → Delete key |
| Delete multiple | Multi-select → Delete key |
| Connect nodes | Drag from output port → drop on input port |
| Disconnect | Drag connection away from port to empty space |

#### Node Type Picker

Floating panel appears near cursor when `+` clicked:

```
┌─────────────────┐
│ Add Node        │
├─────────────────┤
│ 🔋 Battery      │
│ 🎯 Trigger      │
│ 🔁 Repeater     │
│ 🎯 Barrel       │
│ ⚡ Striker      │
│ 💥 Payload      │
└─────────────────┘
```

Click node type to place at cursor location.

#### Port Cardinality

Port shapes indicate connection rules:

```
○  Single — accepts 1 connection max
◎  Multiple — accepts N connections
```

**Example — BARREL with 3 outputs:**
```
┌──────────────┐
│   BARREL     │
│ count: 3     │
│              │
│──○        ◎──│──▶ [Striker 1]
│           ◎──│──▶ [Striker 2]  
│           ◎──│    (available)
└──────────────┘
```

BARREL dynamically shows N output ports based on its `barrel_count` property.

#### Invalid Connections

When a connection cannot be made:
- Target port does not highlight (cannot drop)
- Status bar shows reason: "STRIKER only accepts one input"

#### Node Selection

- Click node to select → properties appear in Inspector
- Selected node shows visual highlight
- Only one node selected at a time (unless multi-select)

### 6.3 Tile Editor

Grid-based painting for chunks.

```
┌─────────────────────────────────────────────────────────────────┐
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                                                           │  │
│  │                                                           │  │
│  │                        TILE CANVAS                        │  │
│  │                       (pan + zoom)                        │  │
│  │                                                           │  │
│  │                                                           │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌───┬───┬───┬───┬───┬───┐                                     │
│  │ ✏️ │ 🪣 │ ◻️ │ 🔲 │ 👆 │ 🔍 │  ← toolbar                      │
│  └───┴───┴───┴───┴───┴───┘                                     │
└─────────────────────────────────────────────────────────────────┘
```

**Toolbar:**
| Tool | Function |
|------|----------|
| ✏️ Pencil | Paint single tiles |
| 🪣 Fill | Flood fill area |
| ◻️ Rectangle | Draw filled rectangle |
| 🔲 Select | Select region |
| 👆 Hand | Pan canvas |
| 🔍 Zoom | Zoom in/out |

**Inspector sections for tile editor:**
- **Chunk**: Size, biome assignment
- **Brush**: Size slider, material dropdown
- **Layers**: Terrain (✓), Entities (✓), toggles
- **Connections**: Entry/exit points (N/S/E/W)

### 6.4 Table View

For data with many similar items (future use if needed).

```
┌──────────────────────────────────────────────────────────────┐
│  Filter: [dropdown ▼] [dropdown ▼] [search________]         │
│                                                              │
│  ┌───┬──────────┬──────────┬────────┬────────┐              │
│  │   │ Column A │ Column B │ Col C  │ Col D  │              │
│  ├───┼──────────┼──────────┼────────┼────────┤              │
│  │   │ value    │ value    │ value  │ value  │              │
│  │ ● │ value    │ value    │ value  │ value  │ ← selected  │
│  │   │ value    │ value    │ value  │ value  │              │
│  └───┴──────────┴──────────┴────────┴────────┘              │
│                                                              │
│  ◀ 1 2 3 4 5 ▶                         Showing 1-20 of N    │
└──────────────────────────────────────────────────────────────┘
```

- Filterable, sortable columns
- Click row to select → details in inspector
- Pagination for large datasets

---

## 7. Specific Views

### 7.1 Materials (WORLD)

**Gallery (L3):** 64 materials shown as cards with color swatches, grouped by biome.

**State Graph (item view):** Each material opens as a graph showing state transitions.

```
                     ┌──────────┐
                     │   GAS    │
                     │ Vaporite │
                     └────▲─────┘
                          │ evaporates
                          │
 ┌──────────┐       ┌─────┴────┐       ┌──────────┐
 │  SOLID   │ melts │  LIQUID  │ cools │ PARTICLE │
 │  ite     │──────▶│  Magmite │◀──────│  ite     │
 └──────────┘       └──────────┘       └──────────┘
                          │
                          │ solidifies
                          ▼
                     ┌──────────┐
                     │  SOLID   │
                     │   ite    │
                     └──────────┘
```

- Each node = one state of the material
- Edges = state transitions (melts, evaporates, solidifies, condenses)
- Click node to edit that state's properties in inspector
- Not all materials need all 4 states

### 7.2 Weapons (ENTITIES)

**Gallery (L3):** All weapons shown as cards.

**Module Graph (item view):** Weapon opens as directed graph of modules.

```
┌──────────┐      ┌──────────┐      ┌──────────┐
│ BATTERY  │      │  BARREL  │      │ STRIKER  │
│          │─────▶│          │─────▶│          │
│ 50/sec   │      │ spread:15│      │ projectile│
└──────────┘      └──────────┘      └────┬─────┘
                                         │
                                         ▼
                                    ┌──────────┐
                                    │ PAYLOAD  │
                                    │          │
                                    │ dmg: 25  │
                                    └──────────┘
```

**Status bar shows computed stats:**
- Energy supply, cost per shot, max fire rate

### 7.3 Levels / Stages (LEVELS)

**Gallery (L3):** All levels shown as cards.

**Stage Graph (item view):** Level opens as graph of connected stages.

```
┌──────────┐      ┌──────────┐      ┌──────────┐
│  ARENA   │      │ CORRIDOR │      │   BOSS   │
│ Arena_01 │─────▶│ Corr_01  │─────▶│ Boss_01  │
└──────────┘      └──────────┘      └────┬─────┘
                                         │
                                         ▼
                                    ┌──────────┐
                                    │  STORE   │
                                    │ Store_01 │
                                    └──────────┘
```

### 7.4 Chunks (WORLD → Terrain)

**Gallery (L3):** All chunks shown as cards with thumbnail previews.

**Tile Editor (item view):** Chunk opens in tile editor (see 6.3).

### 7.5 Story (LEVELS)

**Gallery (L3):** Story nodes shown as cards.

**Graph View (item view):** Lore nodes connected in a graph. Nodes contain text and/or images.

### 7.6 Systems (SYSTEMS)

**Gallery (L3):** System parameters shown as cards (Combat, Harvesting, Economy, etc.).

**Inspector View (item view):** System opens with parameter groups in inspector. No graph or canvas — purely parameter editing.

```
┌──────────────────────────────────────────────────────────────────────────┐
│ Systems › Combat                                                         │
├──────────────────────────────────────────────────────────┬───────────────┤
│                                                          │ INSPECTOR     │
│                                                          ├───────────────┤
│         (description / documentation panel)              │ ▼ Fuel        │
│                                                          │   Base drain  │
│         Shows markdown docs for this system              │   [====] 2/s  │
│                                                          │   Action cost │
│                                                          │   [====] 5    │
│                                                          ├───────────────┤
│                                                          │ ▼ Spawning    │
│                                                          │   Base rate   │
│                                                          │   [====] 10/m │
│                                                          │   Time scale  │
│                                                          │   [====] 1.2x │
│                                                          ├───────────────┤
│                                                          │ ▶ Pressure    │
│                                                          │ ▶ Density     │
├──────────────────────────────────────────────────────────┴───────────────┤
│ ✓ Saved │ Combat system parameters                                       │
└──────────────────────────────────────────────────────────────────────────┘
```

### 7.7 Portals (SYSTEMS)

**Graph View:** Portal routing opens as directional graph showing 8 entry angles and their corridor/boss assignments.

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                     ┌──────────┐                                │
│                     │    UP    │                                │
│                     │ Boss_A   │                                │
│                     └────▲─────┘                                │
│         ┌──────────┐    │    ┌──────────┐                      │
│         │ DIAG-UL  │    │    │ DIAG-UR  │                      │
│         │ Boss_E   │    │    │ Boss_F   │                      │
│         └─────╲────┘    │    └────╱─────┘                      │
│                ╲        │       ╱                               │
│    ┌──────────┐ ╲  ┌────┴────┐ ╱  ┌──────────┐                 │
│    │   LEFT   │──▶│ PORTAL  │◀──│  RIGHT   │                  │
│    │ Boss_D   │   │ BUBBLE  │    │ Boss_B   │                  │
│    └──────────┘   └────┬────┘    └──────────┘                  │
│                ╱       │       ╲                                │
│         ┌────╱────┐    │    ┌────╲─────┐                       │
│         │ DIAG-DL │    │    │ DIAG-DR  │                       │
│         │ Boss_G  │    │    │ Boss_H   │                       │
│         └─────────┘    │    └──────────┘                       │
│                     ┌──┴───────┐                                │
│                     │   DOWN   │                                │
│                     │ Boss_C   │                                │
│                     └──────────┘                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

- Each direction node links to a corridor definition and boss
- Click node to edit corridor length, enemy waves, boss assignment

---

## 8. Runtime Mode

When game is running (F5), editor provides profiler panels and live inspection. The game runs as a separate window — the editor connects via bridge.

### 8.1 Layout

Editor shows profiler panels at edges. Game runs in its own window (not shown here).

```
┌──────────────────────────────────────────────────────────────────────────┐
│                                                           [Pause][Step]  │
├────────┬─────────────────────────────────────────────────┬───────────────┤
│        │                                                 │               │
│ LEFT   │                                                 │    RIGHT      │
│ PANEL  │              (game runs separately)             │    PANEL      │
│        │                                                 │               │
│        │                                                 │               │
│    [◀▶]│                                                 │               │
└────────┴─────────────────────────────────────────────────┴───────────────┘
```

Center area can show:
- Empty state with "Game not running" or "Waiting for connection"
- Entity list / selection UI for choosing what to inspect
- Any additional debug views needed

### 8.2 Resizable Edge Panels

All four edges can host profiler widgets. Panels are resizable by dragging the divider.

**Narrow (default):**
```
├────────┬─────────
│ FPS    │
│ ▁▂▃▄▅▆ │  GAME
│ 60     │
```

**Wide (dragged):**
```
├─────────────────────────────────┬─────────
│ FPS                             │
│ ▁▂▃▄▅▆▇█▇▆▅▄▃▂▁▂▃▄▅▆▇█▇▆▅▄▃▂▁▂▃│  GAME
│ 60                              │
```

- Wider panels show more timeline history
- Collapse fully to hide
- Content is implementation-defined (via Claude Code)

### 8.3 Playback Controls

| Control | Function |
|---------|----------|
| Pause | Pause game execution |
| Step | Advance one frame |

---

## 9. Interaction Patterns

### 9.1 Implicit Saving

All changes save automatically. No explicit save action required.

### 9.2 Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| Cmd+K | Open search / command palette |
| Cmd+Z | Undo |
| Cmd+Shift+Z | Redo |
| Escape | Deselect / close panel |
| Delete | Delete selected items |

### 9.3 Selection

- Single click to select
- Inspector updates to show selected item
- Only one item selected at a time (except gallery edit mode)

---

## 10. Data Flow

### 10.1 Shared JSON Files

Editor and VS Code both read/write the same JSON files. No export step — just saving.

```
VS Code / AI                    Editor                         Game
     │                            │                              │
     │  ── creates JSON ──▶      │                              │
     │                            │  ── tweaks JSON ──▶         │
     │                            │                              │
     │  ◀── can also edit ──     │                              │
     │                            │                              │
     │                            │  ── creates new ──▶         │
     │                            │     (levels, weapons, etc)   │
     │                            │                              │
     └────────────────────────────┴──────────────────────────────┘
                                  │
                           /data/*.json
                                  │
                                  ▼
                                Game
                          (loads at runtime)
```

**Three workflows:**
1. **AI/Code creates** → Editor tweaks → Game loads
2. **Editor creates** → Game loads (designer authoring new levels, weapons, etc.)
3. **Editor tweaks** → VS Code sees changes (shared JSON files)

### 10.2 Runtime Communication

Editor and game communicate via bridge for live inspection during play.

```
Editor                          Game
  │                               │
  │  ◀── entity data ──          │
  │  ◀── performance metrics ──  │
  │                               │
  │  ── pause/step commands ──▶  │
```

Implementation details TBD via Claude Code.

---

## 11. Open Questions

1. **Fields layer** — Parked. How do gravity fields, pressure maps, temperature maps integrate with chunk editing?

2. **iPad sync** — Parked. GitHub branch model planned, details TBD.

3. **Search implementation** — Command palette scope and behavior TBD.

4. **Asset browser** — Import workflow, atlas management TBD.

5. **Runtime profiler widgets** — Specific metrics and visualizations TBD during implementation.

6. **Systems parameter schemas** — What parameters are exposed per system? TBD per system spec.

---

## 12. Parked Concepts

| Concept | Status | Notes |
|---------|--------|-------|
| iPad version | Parked | GitHub branch sync, chunk/level focus |
| Fields layer in chunks | Parked | Gravity, pressure, temperature maps |
| Dockable panels | Cut | Using preset layouts with show/hide instead |
| Bottom inspector | Cut | Right panel only, simpler |

---

## 13. Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2025-01-04 | Initial specification |
| 0.2 | 2025-01-15 | Reorganized IA: added SYSTEMS section, renamed GAMEPLAY to LEVELS, moved economy systems out of standalone section. Added Systems and Portals views. Aligned with Project_Structure_Specification. |

---

## References

- `Project_Structure_Specification.md` — Documentation and editor organization
- `Architecture_Overview.md` — Code architecture and design principles
- `Modular_Weapon_System_Design_Specification.md` — Weapon graph structure
- `Materials_Specification.md` — Material properties and states
- `Game_Overview.md` — Core loop, design pillars
