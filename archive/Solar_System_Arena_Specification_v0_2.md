# Solar System Arena Specification v0.2

## Overview

A massive toroidal arena representing a solar system. Central sun, orbital rings, procedurally generated planets, asteroid belts, and interplanetary space. All terrain is destructible. Progression is gated by material hardness and environmental hazards.

**Win Condition:** Defeat 8 corridor bosses, collect Sun Buster fragments, destroy the sun.

---

## Design Pillars

1. **Everything is terrain** — Sun, planets, asteroids are all destructible sand physics
2. **Layered progression** — Deeper = harder materials = better rewards = need better tools
3. **Physics consequences** — Wrong approach = death (volcano, gas detonation, heat)
4. **Discovery over instruction** — Secrets exist; players find them or don't
5. **Destroy and harvest** — Core verbs are blowing shit up and collecting particles

---

## Arena Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| Topology | Toroidal | Wrap at edges |
| Size | Variable | 65,536+ wu tested, no known ceiling |
| Sun radius | ~5% of arena | Large enough to appear nearly flat at edge |
| Sun attractor | ~2% of arena | Gravity pull zone |
| Planet count | ~5-7 baseline | Tunable parameter |
| Persistence | Per seed, per run | Same layout when returning |

---

## Core Loop

```
ENTER SOLAR SYSTEM (persistent seed)
    ↓
HARVEST ACCESSIBLE PLANET(S)
    ↓
FILL HARVEST BAR → PORTAL ACTIVATES
    ↓
FLY TO PORTAL BUBBLE (HUD guided)
    ↓
CHOOSE ENTRY ANGLE (selects corridor direction + boss)
    ↓
CORRIDOR GAUNTLET (kill N enemies + boss)
    ↓
    ├── SUCCESS → SHOP → return to arena (new tech tier)
    ├── ABORT → boss may escape into arena
    └── DEATH → run over
    ↓
REPEAT (3-5 planets to progress, 8 bosses for Sun Buster)
    ↓
DESTROY THE SUN → FINAL BOSS → ESCAPE COLLAPSE
    ↓
NEXT LEVEL (Underwater World, etc.)
```

---

## Portal System

### Portal Bubble

Seamless spatial stitching — no loading screens, no interaction prompts. Fly through space and you're in the corridor.

| Aspect | Behavior |
|--------|----------|
| Shape | Spherical bubble, larger than screen |
| Activation | Harvest bar fills → dialog/story beat → HUD waypoint |
| Entry | Directional — 8 angles = 8 corridor orientations = 8 bosses |
| Visual | Cool shader, hides corridor chunk loading |
| Bidirectional | Can fly back out anytime |

### Directional Entry

```
                    VERTICAL UP (Boss A)
                           ↑
              DIAG-UL ↖    ↑    ↗ DIAG-UR
                    ╲      ↑      ╱
               ┌───────────────────────┐
               │                       │
HORIZONTAL ←←←←│    PORTAL BUBBLE      │→→→→ HORIZONTAL
LEFT (Boss D)  │                       │      RIGHT (Boss B)
               │                       │
               └───────────────────────┘
                    ╱      ↓      ╲
              DIAG-DL ↙    ↓    ↘ DIAG-DR
                           ↓
                    VERTICAL DOWN (Boss C)
```

Entry angle determines:
- Corridor scroll direction (horizontal, vertical, diagonal)
- Which of 8 bosses you fight
- Which Sun Buster fragment you earn

### Corridor Abort

| Scenario | Consequence |
|----------|-------------|
| Abort mid-corridor | Return to arena |
| Boss was engaged | Boss may follow you into arena |
| Escaped boss | Haunts your solar system until killed |
| Strategy | Be careful which boss you wake first |

---

## Win Condition: Sun Buster

### Progression

```
8 CORRIDOR BOSSES
    ↓
8 SUN BUSTER FRAGMENTS
    ↓
CELESTIAL WEAPON UPGRADES (Zelda-style)
    ↓
DESTROY THE SUN
    ↓
FINAL BOSS EMERGES (something was inside)
    ↓
DEFEAT FINAL BOSS
    ↓
SYSTEM COLLAPSE / ESCAPE SEQUENCE
```

### Celestial Upgrades

Each fragment transforms a base tool into its celestial version. Replaces base (not separate slot).

| # | Base | Celestial | What Changes |
|---|------|-----------|--------------|
| 1 | Laser | Solar Lance | Cuts through Sol's plasma |
| 2 | Bomb | Nova Charge | Detonates without gas ignition, cracks Sol crust |
| 3 | Drill | Core Piercer | Penetrates metallic hydrogen, dense cores |
| 4 | Shield | Corona Ward | Survives Sol heat zones |
| 5 | Harvester | Stellar Collector | Harvests plasma particles |
| 6 | Boost | Flare Dash | Escape solar flare eruptions |
| 7 | Missile | Sun Seeker | Tracks through Sol's gravity distortion |
| 8 | ??? | ??? | TBD — possibly final boss counter or escape tool |

### Boss Order Strategy

- Player chooses order based on readiness
- Escaped bosses persist in arena if you abort
- Wake hard boss early = it haunts your run
- All 8 required for Sun Buster completion

### Sun Destruction

- Sun is invulnerable until all 8 fragments collected
- Celestial weapons enable penetration of Sol layers
- Final boss emerges from destroyed sun
- System collapse triggers escape sequence

---

## Chunk Types

### Sol (Central Sun)

| Aspect | Description |
|--------|-------------|
| Material | Plasma — instant kill on contact |
| Hazard | Heat zone — soft gate, requires shields (eventually Corona Ward) |
| NPCs | Solar Dragons (Salamander tribute), orbital defenders |
| Secrets | Deep within, requires celestial weapons |
| Gravity | Central attractor, dominates inner system |
| Endgame | Target of Sun Buster, contains final boss |

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
| Reward | Materials for tech upgrades |
| Moons | Cluster within planet's attractor radius, signpost proximity |
| Additional hazards | Radioactive rings (achievable with current systems) |

**Planet Archetypes:**

| Type | Core | Mantle | Crust | Hazard |
|------|------|--------|-------|--------|
| Rocky | Molten iron | Dense ore | Regolith | Volcanic breach → eruption |
| Gas Giant | Small dense solid | — | Swirling gas particles | Explosives detonate in gas layer |
| Ice World | Liquid ocean? | Ice sheets | Frost crust | Cryogeysers, pressure |
| Molten | Solid core | Magma sea | Thin volcanic crust | Constant eruption |
| Dead/Shattered | Exposed core | Missing chunks | Debris field | Unstable structure |
| Water World | TBD | TBD | Oceans | TBD — fun to test |

**Gas Giant Detail:**
- No solid surface — particle storm exterior (swirling via gravity field)
- Small, extremely hard core
- Gas layer detonates explosives on contact
- Must use precision weapons to penetrate
- Soft defense (gas), hard core (material)

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
| Density | Mostly empty — POIs feel rare and special |
| Content | NPC encounters (95%), rare POIs |
| POI Types | Space stations (shops), relics, alien artifacts, lore fragments |
| Purpose | Travel, combat, occasional discovery |

---

## Gravity System

All astral bodies have attractors in the gravity flow field.

| Property | Behavior |
|----------|----------|
| Calculation | Additive — vectors sum |
| Overlap | Possible, math is easy |
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

## Layered Progression

### Gating Philosophy

Not arbitrary locks — physics consequences:

- **Wrong tool** → Can't penetrate
- **Wrong weapon** → Backfire (gas detonation)
- **Wrong sequence** → Catastrophe (volcano, flood)
- **Missing defense** → Environmental death (heat, radiation)

### Reference Games

| Game | Relevant Mechanic |
|------|-------------------|
| Motherload | Depth = harder materials + better ore |
| Dome Keeper | Depth tiers unlock with better drills |
| Terraria | Biome layers, pickaxe tiers |
| Noita | Physics consequences, emergent destruction |
| SteamWorld Dig | Layered zones with material gates |

Key insight: This system is "dig smarter" not just "dig deeper."

---

## Economy

### No Crafting

Crafting adds busywork. Store + gambling is more interesting.

### Store Model

| Element | Function |
|---------|----------|
| Location | Space stations (interplanetary POIs) |
| Access | After each successful corridor run |
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

## New Materials Needed

| Material | Properties | Source |
|----------|------------|--------|
| Plasma | Instant kill, emitted by sun/dragons | Sol chunk |
| Dense Gas | Detonates explosives on contact | Gas giant atmosphere |
| Metallic Hydrogen? | Extremely hard, gas giant core | Gas giant core |
| Vacuum Ice | Various ice variants | Ice worlds |
| Radioactive variants | For planetary rings | Various |

---

## Open Questions

### Arena Generation
1. Planet count formula — Fixed per arena size? Random range? Difficulty scaling?
2. Moon generation — How many per planet? Always same materials as parent?
3. Asteroid belt width — Ring thickness, density falloff?
4. Station placement — How many? Fixed or procedural positions?

### Mechanics
5. Heat zone mechanics — Exact damage curve, shield mitigation formula
6. Gas detonation — Radius? Chain reaction? Escape window?
7. Water/ocean behavior — What happens? Worth testing
8. Escaped boss behavior — How aggressive? Can be lured? Despawns?

### Celestial System
9. 8th celestial upgrade — What's the final piece?
10. Final boss design — What emerges from the sun?
11. Escape sequence — How long? What's the threat?

### Level Structure
12. Required planets before corridor — Is it 3-5, or variable?
13. Can you skip planets entirely and just do corridors?

---

## Related Specifications

- `Materials_Specification_v0_3.md` — Material properties, reactivities
- `Game_Overview.md` — Core loop, design pillars
- `Modular_Weapon_System_Design_Specification_v1.1.md` — Weapon system
- `Visual_Style_Specification_v0_1.md` — Art direction
- `Corridor_Specification_v0_1.md` — TBD
- `Portal_System_Specification_v0_1.md` — TBD
- `Celestial_Upgrades_Specification_v0_1.md` — TBD

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2025-01-14 | Initial draft from design conversation |
| 0.2 | 2025-01-14 | Added portal system, core loop, win condition, celestial upgrades, escaped boss mechanic |
