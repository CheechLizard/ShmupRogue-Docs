# Design Specifications – TODO Tracker

## Existing Specifications

| Spec | Version | Status |
|------|---------|--------|
| Game Overview | — | Stable |
| Glossary | — | Stable |
| Modular Weapon System | v1.0 | Stable |
| Story | v0.1 | Draft |
| Tutorial | v0.1 | Draft |
| Resource Economy | v0.1 | Draft |
| Stages & Run Structure | v0.1 | Draft |
| Environments & Terrain | v0.1 | Draft |
| Visual Style | v0.1 | Draft |
| NPC Behavior | v0.1 | Draft |
| Editor | — | Stable |

*Stable = ready for use, may evolve. Draft = work in progress.*

---

## Required Specifications

| Spec | Priority | Dependencies | Notes |
|------|----------|--------------|-------|
| ~~Resource Economy~~ | ~~High~~ | ~~Stages~~ | *Started - see spec* |
| Procedural Generation | High | Stages, Environments | Algorithms, constraints, seeding |
| Boss Design | Medium | NPC Behavior | Mechanics, arenas, progression |
| Store & Economy | Medium | Resource Economy | Currencies, items, pricing |
| Artifact & Buff | Medium | — | Types, effects, acquisition |
| Player Ship | Medium | Weapon System | Stats, loadout, upgrades |
| Modular Ship Systems | Medium | Player Ship | Control schemes, thrusters, shields, subsystems |
| Input System | Low | Player Ship | Key bindings, gamepad, remapping |
| Meta-Progression | Low | All | Persistent unlocks, secrets |

---

## Open Questions by Spec

### Stages & Run Structure
- Arena unlock events: What triggers portal site activation?
- Corridor branch rewards: What makes branches worth the risk?
- Connection matching: How strict should tag matching be?
- Difficulty scaling: How do segment ratings combine with level progression?

### Environments & Terrain
- Simulation bounds: How large can destructible terrain areas be?
- Material interactions: Full interaction matrix (what happens when X meets Y)?
- Regeneration rates: How fast should organic terrain heal?
- Hazard spreading: How aggressively do fire/plasma/etc. propagate?

### Visual Style
- Target resolution and aspect ratio
- Palette constraints per environment
- Player ship visibility rules
- Corruption visual language

### NPC Behavior
- NPC damage from hazard materials
- NPC-triggered material interactions
- Pathfinding in destructible terrain
- Burrower tunnel persistence
- Miner resource drops
- Builder structure limits

---

## Research Topics

### Noita Deep Dive
- Chunk-based world management
- Performance optimizations for falling sand
- Material interaction system
- How determinism is maintained with physics

### Classic Shmup Reference
- R-Type organic regenerating walls
- Gradius terrain patterns
- How terrain difficulty scales in horizontal shmups

---

## Implementation Notes

### Required Systems (Not Yet Built)

| System | Needed For | Notes |
|--------|------------|-------|
| Mission/objective system | Tutorial | Markers, completion triggers, mission flow |
| Dialogue/message display | Tutorial | Story beats, command messages |
| Harvester drone entity | Tutorial M4-5 | Follows player, scoops particles |
| Resource counter UI | Tutorial M5-6 | Display harvested matter |
| Factory/build progress | Tutorial M6 | Progress bar, particle consumption |
| Alien AI (basic) | Tutorial M3+ | Combat behavior, harvesting behavior |
| Win/lose conditions | Tutorial M6 | Race completion, gate building |

### Existing Systems to Leverage

| System | Status | Can Use For |
|--------|--------|-------------|
| Sand physics | Working | Particle destruction already works |
| Weapon system | Working | Combat ready |
| Camera/arena | Working | Navigation ready |
| Terrain generation | Working | Asteroids exist |
| Projectile-entity collision | In Progress | Combat damage to NPCs |

---

## Future Work

Items identified during design discussions, not yet scheduled.

### Visual & Feel

| Item | Description | Related Spec |
|------|-------------|--------------|
| Entity shadows | Player/enemies cast shadows on sand layer (top-down mode) | Environments & Terrain |
| Hit feedback effects | Visual feedback varies by weapon type × defense type | NPC Behavior, Weapon System |
| Weapon × Defense matrix | Full interaction table for damage types vs defense types | NPC Behavior |

### Physics & Collision

| Item | Description | Related Spec |
|------|-------------|--------------|
| Player/NPC flow field gravity | Entities should feel the same gravity as sand particles | Environments & Terrain |
| Rigid flight-layer obstacles | Non-sand obstacles on the gameplay plane | Environments & Terrain |
| Perspective-aware collision | Collision rules change based on flow field state (top-down vs side-scroll) | Environments & Terrain |

### Combat

| Item | Description | Related Spec |
|------|-------------|--------------|
| Defense types | NPC properties (armored, organic, shielded) that modify damage | NPC Behavior |
| Damage types | Weapon properties (kinetic, energy, explosive, acid) | Weapon System |
| Friendly fire immunity | Optional upgrades/abilities to prevent self-damage | NPC Behavior |
