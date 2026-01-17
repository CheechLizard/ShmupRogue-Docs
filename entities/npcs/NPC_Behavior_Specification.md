# NPC Behavior – Design Specification v0.1

## Overview

NPCs (enemies and neutral entities) share the same physics rules as players. They interact with terrain, compete for resources, and respond to environmental changes. This consistency makes the world feel believable and creates emergent gameplay.

---

## Design Principles

**Shared Physics** — NPCs obey the same terrain and destruction rules as players. No special cases. If the player can dig through it, so can a worm.

**Emergent Behavior** — Simple rules create complex situations. A miner competing for resources is also clearing paths. A burrower fleeing the player might accidentally release lava.

**Readable Intent** — Despite emergent complexity, player should understand what an NPC is trying to do. Visual and audio cues telegraph behavior.

---

## NPC Categories

### By Allegiance

| Type                  | Behavior                                            |
| --------------------- | --------------------------------------------------- |
| **Hostile**     | Attacks player on sight or proximity                |
| **Neutral**     | Ignores player unless provoked                      |
| **Territorial** | Attacks if player enters zone or takes resources    |
| **Competing**   | Pursues same goals as player (resources, territory) |

### By Terrain Interaction

| Type                | Ability                                 | Gameplay Impact                         |
| ------------------- | --------------------------------------- | --------------------------------------- |
| **Standard**  | No terrain interaction                  | Navigates around obstacles              |
| **Miner**     | Harvests/destroys terrain for resources | Competes with player; clears paths      |
| **Burrower**  | Digs through terrain to move            | Flanks player; creates tunnels; escapes |
| **Builder**   | Creates terrain (webs, walls, nests)    | Blocks paths; creates chokepoints       |
| **Destroyer** | Massive terrain destruction             | Area denial; reshapes battlefield       |

### By Movement

| Type | Ability | Gameplay Impact |
| ---- | ------- | --------------- |
| **Flying** | Free movement, ignores terrain | Most common; forms flocks and attack patterns |
| **Walking** | Ground-bound, affected by gravity | Predictable paths; terrain blocks them |
| **Amphibious** | Can land/take off | Versatile; switches between modes |

### Flocking & Formations

Flying enemies should form coordinated attack patterns—not just random swarms. Classic shmup formations (waves, spirals, pincer attacks) combined with horde survivor density. Most horde survivors miss on this; we want legible, threatening patterns even with high enemy counts.

---

## Terrain Interaction Details

### Miners

- Seek out resource-rich terrain
- Harvest at visible rate (player can see competition)
- May stockpile or consume resources
- Killing miners could drop harvested resources

### Burrowers

- Use terrain for movement, not harvesting
- Create persistent tunnels (or tunnels that collapse?)
- Can ambush from unexpected angles
- May flee into terrain when damaged

### Builders

- Create obstacles from materials or spawned substance
- Webs slow player; walls block shots; nests spawn more enemies
- Built structures are destructible
- Could repair/rebuild if left alone

### Destroyers (Worms, etc.)

- Large-scale terrain removal
- Terrifying presence—can reach player anywhere
- High threat but also clears paths
- Rare; late-game encounters

---

## Damage & Defense

NPCs receive damage from the weapon system through projectile collision and payload effects.

### Receiving Damage

Projectiles check collision against NPCs each frame:
- Point-vs-circle collision (projectile position vs NPC collision radius)
- On hit, the weapon's payload applies damage to the NPC
- Damage flows through the modular weapon system—same graph that handles terrain damage

**Projectile behavior on hit:**
- **Consumable:** Projectile destroyed after first hit (bullets, missiles)
- **Retriggerable:** Projectile continues, can hit multiple targets (piercing, beams)

### Defense Types (Future)

NPCs will have defense properties that modify incoming damage and visual feedback:

| Defense Type | Description | Visual Feedback on Hit |
|--------------|-------------|----------------------|
| **Unarmored** | Standard damage | Flash, knockback |
| **Armored** | Reduced damage, immune to some types | Sparks, ricochet sound |
| **Organic** | May regenerate, vulnerable to fire/acid | Splatter, burn marks |
| **Shielded** | Absorbs damage until depleted | Shimmer, overload effect |

**Weapon × Defense Matrix:**
Different weapon types (kinetic, energy, explosive, acid) interact differently with defense types. This creates tactical variety—some weapons are more effective against certain enemies.

*(Full matrix TBD—placeholder for future design work.)*

### Friendly Fire

By default, all projectiles can damage all entities (including player). This creates:
- Risk/reward for explosive weapons
- Tactical opportunities (enemies damaging each other)
- Potential for immunity upgrades or defensive abilities

---

## Open Questions

1. Do NPCs take damage from hazard materials (lava, acid)?
2. Can NPCs trigger material interactions (carry fire into gas)?
3. How do NPCs pathfind in destructible terrain?
4. Do burrower tunnels persist or collapse over time?
5. Resource drop behavior when miners are killed
6. Builder structure limits (prevent soft-locks)
7. Movement mode distribution: what % of enemies fly vs walk vs land?
8. Flocking algorithm: boids-style, scripted formations, or hybrid?
9. Formation complexity vs enemy count tradeoff

---

## Related Specifications

- Environments & Terrain — terrain types, material behaviors
- Resource Economy — what NPCs compete for
- Boss Design — large destroyers, terrain-based boss arenas
