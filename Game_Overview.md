# Shmup Rogue — Game Overview

## The Game

**Working title:** Shmup Rogue

**Genre:** Roguelike shoot-em-up with free-roaming exploration, sandbox destruction, and horde survivor elements

**Pitch:** A late-80s arcade sci-fi shooter with massive destruction, roguelike replayability, and unforgiving risk/reward systems. Feels impossible at first—then it starts clicking, and the endorphins don't stop.

---

## Design Pillars

### 1. Arcade Intensity
Fast, responsive, punishing. Every death teaches something. Mastery is earned.

### 2. Power Fantasy Escalation
Early game is scrappy survival. Late game is screen-filling chaos—swarms of drones, cascading destruction, harvest multipliers, numbers flying everywhere.

### 3. Massive Destruction
Everything breaks. Terrain is fully destructible via sand physics. Weapons reshape the environment. Chain reactions cascade.

### 4. Emergent Builds
Simple modules combine into complex behaviors. Every run plays different based on what you find and how you build.

### 5. Exploration & Replayability
The universe feels vast and open. Procedurally generated structures with hand-authored segments. Optional paths offer rewards but cost time. The map remembers what you've unlocked.

---

## Aesthetic

**Era:** Late 1980s arcade sci-fi

**Touchstones:** Alien, Aliens, Blade Runner, R-Type, Gradius, Giger

**Range:** Clean robotic stations to cyberpunk cities to ancient alien ruins to biomechanical nightmares to the inside of living organisms. The universe spans human, alien, and organic—pristine to corrupted, familiar to utterly strange.

**Visual style:** 2D pixel art with CRT shader treatment. Fast parallax backgrounds create sense of speed and scale.

*(See Visual Style Specification for environment framework and art direction.)*

---

## Core Loop

```
┌─────────────────────────────────────────────────────────────────┐
│                         ONE LEVEL                               │
│                                                                 │
│  [ARENA] ──▶ [CORRIDOR] ──▶ [BOSS] ──▶ [STORE] ──▶ next        │
│     │              ↑                        │                   │
│     │         dropped crates                │                   │
│     └──────── backtrack to retrieve ────────┘                   │
└─────────────────────────────────────────────────────────────────┘

ARENA: Destroy terrain, harvest materials. Bar fills. Pure stripes compress 
       into crates that trail behind ship. When bar is full, nothing left 
       to do—must move on.

CORRIDOR: The gauntlet. Crates slow you down, make you a target. Crates don't 
          take damage but drop when hit. Dropped crates never despawn. Survive 
          to boss with what you can protect.

BOSS: Fixed encounter. Defeat to proceed.

STORE: Cash in crates, buy upgrades. Return to previous areas for dropped 
       crates—now you're stronger.

Repeat until death or victory. Run resets. Some progress persists (meta-progression TBD).
```

**Crate Mechanics:**

| Aspect | Behavior |
|--------|----------|
| Creation | Pure harvest bar stripes compress, eject as crate |
| Towing | Crates chain behind ship, visible in world |
| Drag | More crates = slower movement |
| Damage | Crates don't take damage, but drop when hit |
| Persistence | Dropped crates never despawn |
| Retrieval | Backtrack anytime to collect dropped crates |

**The Tension:**

- Full haul through corridor = max reward, brutal gauntlet
- Drop crates under fire = survive, but loot left behind
- Return later with upgrades = easier retrieval, costs time

---

## End-Game Fantasy

The ideal late-game moment:
- Screen filled with drones (combat, harvester, options, shields)
- Constant weapon fire from ship + mirrored by option drones
- Endless horde of NPCs exploding
- Materials streaming into harvest bar
- Tetris-style "pure bar" clears with multiplier bonuses
- Numbers flying off everything
- Dangerous environmental hazards that would have one-shot you in early levels

**Progression Feel:**

| Phase | Drone Count | Feel |
|-------|-------------|------|
| Early | 1-2 | Survival, scrappy, every decision matters |
| Mid | 5-10 | Building momentum, synergies emerging |
| Late | 20-50 | Power fantasy, commanding a swarm |
| End game | 100+ | Screen-filling chaos, bullet heaven payoff |

---

## Ship Systems

The ship is a drone host. All collection, combat assistance, and utility comes from drones.

| System | Function | Upgrades Affect |
|--------|----------|-----------------|
| Weapons | Direct fire, defense, destruction | Damage, fire rate, energy, module slots |
| Drones | Deployable + ship-bound companions | See Drone Systems |

*(See Drone_Systems_Specification.md for full drone details including harvester filtration and inventory.)*

---

## Drone System

Everything is drones. Unlimited count, scales like bullet heaven.

| Type | Behavior | Movement |
|------|----------|----------|
| Harvester | Auto-collects particles | Ship-bound |
| Mining | Extracts large/precision materials | Deployable |
| Combat | Attacks hostiles | Deployable or Ship-bound |
| Options | Mirror ship weapons | Ship-bound (orbit) |
| Shield | Blocks projectiles | Ship-bound (rotate) |
| Force | Rams, blocks, detach/recall | Ship-bound (R-Type style) |

**Summon Model:** Deploy → Active → Destroyed → Cooldown → Regenerate

*(See Drone_Systems_Specification.md)*

---

## Harvest Bar

Materials stream into a visual bar during play. Bar sorts materials into colored stripes.

```
HARVEST BAR (live during run):
┌──────────────────────────────────────────────────┐
│████████████░░░░░░░░▓▓▓▓▓▓▓▓▓▓░░░░████████████████│
│  iron      │ sand  │ crystal  │sand│    iron     │
└──────────────────────────────────────────────────┘

When a "pure" stripe reaches threshold width:
┌──────────────────────────────────────────────────┐
│████████████████████████████████░░░░░░░░░░░░░░░░░░│ ← FLASH!
│         PURE IRON - BONUS!                       │
└──────────────────────────────────────────────────┘
→ Clears into inventory with multiplier
```

**Purity Mechanic:**
- Mixed harvesting = jumbled bar, no bonus
- Targeted harvesting = pure stripes form
- When pure stripe reaches threshold = Tetris-style clear
- Clear triggers multiplier bonus, materials bank to inventory

**Rewards:**
- Skill-based: targeted destruction beats indiscriminate chaos
- Synergizes with filters: better filters = easier purity
- Late game: constant clears, numbers flying everywhere

---

## Economy Loop

```
BEFORE RUN:
- Set contracts (optional challenges)
- Configure filtration
- Check market prices

DURING RUN:
- Destroy terrain → materials
- Harvest bar fills, pure stripes clear with bonuses
- Drones assist (harvester, mining, combat)
- Inventory decisions when full

AFTER RUN:
- Fulfill contracts (bonus gold)
- Sell materials (prices shift based on supply)
- Shop for upgrades
```

*(See Resource_Economy_Specification.md)*

---

## Systems

| System | Goal | Doc |
|--------|------|-----|
| Black Holes | Gravitational sinks, exotic material output | `Black_Holes_Specification.md` |
| Drone Systems | Deployable + ship-bound companions | `Drone_Systems_Specification.md` |
| Engine | Make everything run fast and feel responsive | TBD |
| Environments & Terrain | Make each space feel distinct and destructible | `Environments_Terrain_Specification.md` |
| Resource Economy | Material flow, harvesting, shop | `Resource_Economy_Specification.md` |
| NPCs & Enemies | Create threats that challenge and surprise | `NPC_Behavior_Specification.md` |
| Physics | Simulate movement, collision, and destruction | TBD |
| Player Ship | Movement that feels precise and expressive | TBD |
| Stages & Run Structure | Varied pacing with reasons to replay | `Stages_Run_Structure_Specification.md` |
| Store & Build Paths | Tempting choices that shape your run | TBD |
| Story | Narrative framing and lore | `Story_Specification.md` |
| Tutorial | Level 1 structure, teaching mechanics | `Tutorial_Specification.md` |
| Visual Style | Late-80s arcade sci-fi brought to life | `Visual_Style_Specification.md` |
| Weapon System | Emergent complexity from simple parts | `Modular_Weapon_System_Specification.md` |

Each system doc defines its own interactions with other systems.

See `Glossary.md` for terminology.

---

## Open Questions (Project-Wide)

1. **Meta-progression:** What persists between runs? Unlocks? Currency? Knowledge?
2. **Difficulty curve:** How does challenge scale across a run? Across meta-progression?
3. **Win condition:** What does "victory" look like? Final boss? Endless high score?
4. **Market system:** How do prices shift? Predictable cycles, player-reactive, events?
5. **Harvest bar tuning:** Purity threshold, multiplier values, clear frequency?

*(Narrative question resolved - see `Story_Specification.md`)*

---

## Appendix: Reference Points

| Game | What We're Taking |
|------|-------------------|
| **R-Type, Gradius** | Tight corridor sections, biomechanical aesthetics, late-80s sci-fi tone, Force/Options as drones |
| **Defender** | Speed, intensity, screen-wrap |
| **Hunter's Moon** (C64) | Free 2D scrolling in toroidal arenas |
| **Hyperdyne Side Arms** | Levels with varied scroll directions |
| **Wizball** (C64) | Loop structure (2D scrolling → hyperspace → wizard's lair) |
| **Dead Cells** | Procedural structure from authored pieces, meta-progression, branch choices |
| **Noita** | Sand physics, emergent destruction, environmental interaction |
| **Stand Survivors** | Horde survivor density, autobattler-style unit synergies, wave combat, drone scaling |
| **Balatro** | Shop design, satisfying number feedback, multiplier systems |
| **Dave the Diver** | Arena → Shop loop, inventory management decisions |
| **Salamander** | Options/multiples that mirror fire, ship-bound companions |
