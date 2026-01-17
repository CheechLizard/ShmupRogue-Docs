# Drone Systems Specification v0.1

## Overview

Drones are deployable companions that extend ship capabilities. They follow a roguelike summon model: deploy, use until destroyed, wait for cooldown, repeat.

---

## Core Mechanics

**Summon Model:**

| Phase | Behavior |
|-------|----------|
| Deploy | Summon from ship, costs energy/cooldown |
| Active | Performs function (mining, combat, etc.) |
| Damaged | Takes damage from NPCs, environment, explosions |
| Destroyed | Cooldown timer starts |
| Regenerate | After cooldown, can summon again at full health |

**All Drones Share:**

- Health pool
- Destruction → cooldown → regeneration cycle
- Upgrade paths
- Vulnerable to environment and hostiles

---

## Drone Types

| Type | Function | Tradeoff |
|------|----------|----------|
| Harvester | Auto-collects particles | Ship-bound, passive |
| Mining | Extracts large/precision materials | Unarmed, vulnerable |
| Combat | Turret, attacks hostiles | Doesn't harvest |
| Hybrid | Both capabilities, weaker at each | Jack of all trades |

### Harvester Drone

Core collection system. Two variants exist:

**Ship-Bound Harvester (Default):**

- Attached to ship (R-Type Force style)
- Auto-collects particles within range
- Feeds into Harvest Bar
- Always active, no deployment needed

**Captured Alien Harvester:**

- Follows player as separate entity
- Captured from defeated alien ships
- Can be lost if destroyed (no regeneration)
- Larger capacity, different filtration options

**Filtration:**

Harvester drones include filtration systems that determine what materials to keep vs discard.

| Filtration Level | Behavior |
|------------------|----------|
| None | Collects everything (including trash) |
| Basic | Filters out common rock/sand |
| Advanced | Selective by material type |
| Quantum | Can filter by state (gas, liquid) |

Early game: No filters, inventory clogs with trash. Motivation to upgrade.

**Inventory (Harvest Bar):**

Harvester feeds into ship inventory, displayed as the Harvest Bar.

| Aspect | Description |
|--------|-------------|
| Capacity | Total storage, upgradeable |
| Visual | Bar sorts materials into colored stripes |
| Purity Bonus | Pure stripes trigger Tetris-style clears with multipliers |

**Harvester Upgrades:**

| Tier | Improvement |
|------|-------------|
| Basic | Short range, no filtration, small capacity |
| Improved | Larger range |
| Filtered | Basic filtration unlocked |
| Advanced | Advanced filtration, larger capacity |
| Quantum | Can collect gases/liquids, quantum filtration |

### Mining Drone

Extracts materials that can't be harvested through destruction.

**Capabilities:**

| Capability | Description |
|------------|-------------|
| Large solids | Diamonds, chunks that don't particulate |
| Precision extraction | Fragile materials that break with explosives |
| Reactive materials | Contained extraction of hazardous materials |

**Behavior:**

- Flies to designated target
- Extracts over time (not instant)
- Returns materials to ship inventory
- Vulnerable while extracting

### Combat Drone

Autonomous turret that attacks hostiles.

**Behavior Modes:**

| Mode | Description |
|------|-------------|
| Follow | Orbits ship, attacks nearby hostiles |
| Guard | Stays at position, defends area |
| Escort | Follows mining drone, protects it |
| Aggressive | Seeks out hostiles, wider range |

**Weapon Options:**

Inherits from modular weapon system? Or fixed loadout per drone tier?

### Hybrid Drone

Combines mining and combat. Weaker at both.

**Use Case:**

- Solo runs where flexibility matters more than specialization
- Early game before player can afford multiple specialized drones
- Backup when one specialized drone is on cooldown

---

## Loadout Decisions

Players choose drone loadout before run:

| Loadout | Strategy |
|---------|----------|
| 2× Mining | Max harvest, rely on ship weapons for defense |
| 2× Combat | Heavy defense, limited to destruction-based harvesting |
| 1× Mining + 1× Combat | Balanced, mining drone has protection |
| 2× Hybrid | Flexible but suboptimal at everything |

---

## Upgrade Paths

### Mining Drone Upgrades

| Tier | Improvement |
|------|-------------|
| Basic | Slow extraction, fragile, long cooldown |
| Improved | Faster extraction speed |
| Armored | Increased health pool |
| Advanced | Shorter cooldown |
| Quantum | Can extract exotic materials, gases, liquids |

### Combat Drone Upgrades

| Tier | Improvement |
|------|-------------|
| Basic | Low damage, short range, long cooldown |
| Improved | Increased damage |
| Armored | Increased health pool |
| Advanced | Shorter cooldown, longer range |
| Quantum | Energy weapons, can damage shielded enemies |

### Hybrid Drone Upgrades

| Tier | Improvement |
|------|-------------|
| Basic | Weak at both functions |
| Improved | Better extraction OR better damage (player choice?) |
| Armored | Increased health pool |
| Advanced | Shorter cooldown |
| Specialized | Leans toward mining or combat based on upgrades taken |

---

## Tactical Implications

| Situation | Decision |
|-----------|----------|
| Dense hostiles | Deploy combat first, clear area |
| Valuable deposit | Risk mining drone, or clear first? |
| Drone destroyed | Wait out cooldown or proceed without? |
| Multiple deposits | Prioritize—can't mine everything |
| Boss fight | Combat drones for DPS, mining drones stay home |

**Emergent Gameplay:**

- Escort missions: Protect your mining drone
- Distraction: Combat drone draws aggro while you harvest
- Sacrifice: Let drone die to buy escape time

---

## Parameters

| Parameter | Description | Tuning Notes |
|-----------|-------------|--------------|
| Base health | Starting HP per drone type | Combat > Hybrid > Mining? |
| Cooldown duration | Time to regenerate after destruction | Major balance lever |
| Deploy energy cost | Ship energy to summon | Prevents spam |
| Extraction speed | Time to mine a target | Affects risk window |
| Combat range | Engagement distance | Affects positioning |
| Combat damage | DPS output | Balance vs ship weapons |

---

## Open Questions

1. **Drone limit:** How many drones can be active simultaneously? Fixed, or upgradeable?

2. **Drone persistence:** Do drones persist between arena and corridor segments, or reset?

3. **Drone AI:** How smart? Pathfinding complexity? Avoid hazards automatically?

4. **Drone weapons:** Use modular weapon system, or fixed loadouts per tier?

5. **Drone visuals:** Distinct silhouettes per type? Upgrade visuals?

6. **Drone commands:** How does player direct drones? Quick select? Contextual?

7. **Drone synergies:** Can drones buff each other? Ship buff drones?

---

## References

- Resource Economy Specification v0.1 — Harvesting context
- Modular Weapon System Design Specification v1.1 — Potential weapon integration
