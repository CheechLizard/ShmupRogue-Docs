# Resource Economy Specification v0.1

## Overview

How matter flows through the game. Destruction feeds construction. Combat and economy are the same system.

---

## The Matter Loop

1. **Solid matter** (asteroids, structures) - can't be harvested directly
2. **Destroy** → matter breaks into particles (sand physics)
3. **Harvester drone** scoops particles with ram scoop
4. **Factory** transforms particles into structures (warp gate, upgrades, etc.)

**Core Constraint:** Early game, you can only process loose particles - not solid rock. This creates the loop: break things down, scoop them up, build things.

---

## Resource Types

| Resource | Source | Used For |
|----------|--------|----------|
| Matter (generic) | Any destroyed material | Warp gate construction, basic builds |
| *Future: specialized materials?* | *TBD* | *Upgrades, weapons, specific structures* |

*Note: Start simple with one resource type. Add specialization if gameplay needs it.*

---

## Harvesting

### Player Harvester Drone

- Captured from aliens in Tutorial Mission 4
- Follows player
- Scoops loose particles on contact (ram scoop)
- Deposits to player's resource pool automatically

### Alien Harvesting

- Aliens use same method: destroy → scoop → build
- Visual distinction needed (different drone appearance, particle color when scooped?)
- Aliens deposit to their factory, competing with player

---

## Factory / Construction

### The Artifact

- Alien matter transformer, hijacked by player
- Accepts harvested particles
- Builds structures over time (progress bar)
- Primary use: warp gate construction

### Build Costs

| Structure | Cost | Build Time | Notes |
|-----------|------|------------|-------|
| Warp Gate | TBD | TBD | Level exit, race objective |
| *Future: upgrades?* | *TBD* | *TBD* | *Ship improvements, weapons* |

---

## The Race Mechanic

Level 1 introduces competition for resources:

- Player and aliens both harvesting
- Both building toward warp gates
- First to complete wins/loses the level
- Creates time pressure without arbitrary timer

### Balance Considerations

- How fast do aliens harvest?
- How much matter exists in the arena?
- Can player disrupt alien harvesting? (destroy their drones?)
- Can aliens disrupt player? (attack harvester?)

---

## Open Questions

1. **Resource Specialization:** One generic "matter" or multiple types (rock, metal, organic)?

2. **Harvester Vulnerability:** Can the drone be destroyed? What happens - respawn? Game over?

3. **Storage Limits:** Can player stockpile infinite matter, or is there a cap?

4. **Passive vs Active Harvesting:** Does drone auto-collect, or does player direct it?

5. **Disruption:** Can player destroy alien harvesters? Can aliens destroy yours?

6. **Later Levels:** Does the race mechanic continue, or does economy evolve?

---

## References

- **Noita:** Matter transformation as core mechanic
- **Defender:** Frantic resource protection under time pressure
