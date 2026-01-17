# Solar System Arena Specification v0.1

## Overview

A massive toroidal arena representing a solar system. Central sun, orbital rings, procedurally generated planets, asteroid belts, and interplanetary space. All terrain is destructible. Progression is gated by material hardness and environmental hazards.

---

## Design Pillars

1. **Everything is terrain** — Sun, planets, asteroids are all destructible sand physics
2. **Layered progression** — Deeper = harder materials = better rewards = need better tools
3. **Physics consequences** — Wrong approach = death (volcano, gas detonation, heat)
4. **Discovery over instruction** — Secrets exist; players find them or don't

---

## Arena Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| Topology | Toroidal | Wrap at edges |
| Size | Variable | 65,536+ wu tested, no known ceiling |
| Sun radius | ~5% of arena | Large enough to appear nearly flat at edge |
| Sun attractor | ~2% of arena | Gravity pull zone |
| Planet count | ~5-7 baseline | Tunable parameter |

---

## Chunk Types

The arena is composed of distinct chunk types, each with unique content and challenge.

### Sol (Central Sun)

| Aspect | Description |
|--------|-------------|
| Material | Plasma — instant kill on contact |
| Hazard | Heat zone — soft gate, requires shields to approach |
| NPCs | Solar Dragons (Salamander tribute), orbital defenders |
| Secrets | Deep within, requires endgame tools |
| Gravity | Central attractor, dominates inner system |

**Solar Dragons:**
- Japanese fire dragon aesthetic
- Orbit the sun
- Emit plasma particles that rain back down
- Tribute to Salamander/Life Force

### Planets

Each planet is a layered puzzle. Surface materials are harvestable early; deeper layers require progression.

| Aspect | Description |
|--------|-------------|
| Structure | Layered: crust → mantle → core |
| Challenge | Material hardness, physics consequences |
| Reward | Required resource per planet (quota system) |
| Moons | Cluster within planet's attractor radius |

**Planet Archetypes:**

| Type | Core | Mantle | Crust | Hazard |
|------|------|--------|-------|--------|
| Rocky | Molten iron | Dense ore | Regolith | Volcanic breach → eruption |
| Gas Giant | Small dense solid | — | Swirling gas particles | Explosives detonate in gas layer |
| Ice World | Liquid ocean? | Ice sheets | Frost crust | Cryogeysers, pressure |
| Molten | Solid core | Magma sea | Thin volcanic crust | Constant eruption |
| Dead/Shattered | Exposed core | Missing chunks | Debris field | Unstable structure |

**Gas Giant Detail:**
- No solid surface — particle storm exterior
- Small, extremely hard core
- Gas layer detonates explosives on contact
- Must use precision weapons to penetrate

**Example: Earth-like Planet**
- Surface: land, oceans — harvestable immediately
- Geological layers: progressively harder rock
- Molten core: breach wrong → volcanic eruption → death
- Puzzle: find safe approach, drain ocean first?, use right weapon

### Asteroid Belts

| Aspect | Description |
|--------|-------------|
| Content | Dense field of small bodies, artifacts |
| Challenge | Navigation, ambush NPCs, racing |
| Reward | Artifacts, materials, secrets in larger asteroids |
| Gameplay | Chasing, being chased, searching |

### Interplanetary Space

| Aspect | Description |
|--------|-------------|
| Density | Mostly empty — POIs feel special |
| Content | NPC encounters (95%), rare POIs |
| POI Types | Space stations, relics, alien artifacts, lore fragments |
| Purpose | Travel, combat, occasional discovery |

---

## Gravity System

All astral bodies have attractors in the gravity flow field.

| Property | Behavior |
|----------|----------|
| Calculation | Additive — vectors sum |
| Overlap | Possible but typically distant enough to not interact |
| Effect | Pulls player, particles, loose materials |
| Sun | Dominates center, creates orbital "current" |

---

## Orbital Motion

Terrain is static. Motion is implied through visual effects:

| Effect | Implementation |
|--------|----------------|
| Starfield rotation | Background shader rotates around sun |
| Ring particles | Non-terrain dust/ice flows along orbital paths via flow field |
| Solar flare rotation | Flares spawn and rotate around sun surface |
| Orbital wind | Tangential flow field pushes loose materials along orbital paths |

---

## Progression System

### Core Loop

```
ENTER ARENA
    ↓
HARVEST surface materials (easy planets, asteroid belts)
    ↓
SELL at station → credits
    ↓
BUY/GAMBLE for upgrades + weapons
    ↓
DIG DEEPER with new tools
    ↓
HARVEST rare materials (cores, moons, sun approach)
    ↓
REPEAT until win condition
```

### Layered Gating

| Gate Type | Example | Counter |
|-----------|---------|---------|
| Material hardness | Can't drill obsidian without plasma drill | Tool upgrades |
| Physics consequence | Breach magma = death | Knowledge, technique |
| Environmental hazard | Heat zone around sun | Shield upgrades |
| Weapon mismatch | Nukes in gas giant = suicide | Weapon variety |

### What Gates Progress

Not arbitrary locks — physics consequences:

- **Wrong tool** → Can't penetrate
- **Wrong weapon** → Backfire (gas detonation)
- **Wrong sequence** → Catastrophe (volcano, flood)
- **Missing defense** → Environmental death (heat, radiation)

---

## Economy

### No Crafting

Crafting adds busywork. Store + gambling is more interesting.

### Store Model

| Element | Function |
|---------|----------|
| Location | Space stations (fixed POIs in interplanetary space) |
| Inventory | Curated random selection per visit |
| Currency | Harvested materials → credits |
| Purchases | Tool tiers, weapon unlocks, shield capacity |

### Gambling Options

| Type | Mechanic |
|------|----------|
| Weapon slots | Pay credits, spin for random weapon (tier-weighted) |
| Shop reroll | Don't like selection? Pay to refresh |
| Upgrade gamble | Cheap with fail chance vs expensive guaranteed |

### Anti-Frustration

| Mechanic | Purpose |
|----------|---------|
| Pity timer | Guaranteed good roll after N bad ones |
| Visible odds | Player knows the math |
| No total loss | Always get *something*, varying quality |

---

## Win Condition

**TBD** — Depends on broader game mechanics.

Options under consideration:
1. **Material quotas** — Harvest X of each planet's signature material
2. **Boss unlock** — Collect planet keys, summon final encounter
3. **Sol dive** — Ultimate goal is reaching sun's core secret
4. **Survival** — Endless escalation, high score

---

## Reference Games

| Game | Relevant Mechanic |
|------|-------------------|
| Motherload | Depth = harder materials + better ore |
| Dome Keeper | Depth tiers unlock with better drills |
| Terraria | Biome layers, pickaxe tiers |
| Noita | Physics consequences, emergent destruction |
| SteamWorld Dig | Layered zones with material gates |

Key insight from references: Your system is "dig smarter" not just "dig deeper."

---

## New Materials Needed

| Material | Properties | Source |
|----------|------------|--------|
| Plasma | Instant kill, emitted by sun/dragons | Sol chunk |
| Dense Gas | Detonates explosives on contact | Gas giant atmosphere |
| Metallic Hydrogen? | Extremely hard, gas giant core | Gas giant core |
| Vacuum Ice | Various ice variants | Ice worlds |

---

## Open Questions

1. **Planet count formula** — Fixed per arena size? Random range? Difficulty scaling?
2. **Moon generation** — How many per planet? Always same materials as parent?
3. **Asteroid belt width** — Ring thickness, density falloff?
4. **Station placement** — How many? Fixed or procedural positions?
5. **Win condition** — See options above
6. **Heat zone mechanics** — Exact damage curve, shield mitigation formula
7. **Gas detonation** — Radius? Chain reaction? Escape window?

---

## Related Specifications

- `Materials_Specification_v0_3.md` — Material properties, reactivities
- `Game_Overview.md` — Core loop, design pillars
- `Modular_Weapon_System_Design_Specification_v1.1.md` — Weapon system
- `Visual_Style_Specification_v0_1.md` — Art direction

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2025-01-14 | Initial draft from design conversation |
