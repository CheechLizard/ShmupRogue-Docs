# Celestial Upgrades System Specification v0.1

**Location:** `/systems/`

---

## Overview

Celestial Upgrades are the progression reward from corridor bosses. Each of 8 bosses drops a fragment that transforms a base tool into its celestial version. Zelda-style sequential upgrades — not inventory items, but permanent tool evolution.

---

## Core Concept

```
DEFEAT BOSS → GET FRAGMENT → TOOL TRANSFORMS

Base Drill  +  Fragment 3  =  Core Piercer
```

Celestial versions **replace** base tools (not separate slot).

---

## Upgrade Table

| # | Base Tool | Celestial Version | What Changes | Boss Direction |
|---|-----------|-------------------|--------------|----------------|
| 1 | Laser | Solar Lance | Cuts through Sol's plasma | TBD |
| 2 | Bomb | Nova Charge | Detonates without gas ignition, cracks Sol crust | TBD |
| 3 | Drill | Core Piercer | Penetrates metallic hydrogen, dense cores | TBD |
| 4 | Shield | Corona Ward | Survives Sol heat zones | TBD |
| 5 | Harvester | Stellar Collector | Harvests plasma particles | TBD |
| 6 | Boost | Flare Dash | Escape solar flare eruptions | TBD |
| 7 | Missile | Sun Seeker | Tracks through Sol's gravity distortion | TBD |
| 8 | ??? | ??? | TBD | TBD |

---

## Design Principles

**Each upgrade must:**
1. Have a meaningful base version for normal play
2. Enable something specific in Sol assault
3. Feel like a power spike, not incremental stat boost

**Transformation, not addition:**
- Celestial replaces base
- No "use old version" option
- Avoids inventory bloat

---

## Slot 8 Candidates

The 8th upgrade is undefined. Options:

| Option | Base | Celestial | Purpose |
|--------|------|-----------|---------|
| Scanner | Reveals secrets | Sun's Heart Sight | Find Sol weak point |
| Tractor Beam | Pull items | Gravity Anchor | Resist Sol's pull |
| Emergency Warp | Short teleport | Phase Shift | Pass through plasma |
| Armor | Damage reduction | Corona Shell | Final boss survival |

---

## Boss-Direction Mapping

Each direction has a thematic boss and matching upgrade.

**Possible themes:**

| Direction | Theme | Boss Concept | Upgrade Match |
|-----------|-------|--------------|---------------|
| Up | Light/Fire | Solar guardian | Solar Lance? |
| Down | Depth/Pressure | Abyssal creature | Core Piercer? |
| Left | Defense | Shield construct | Corona Ward? |
| Right | Speed/Pursuit | Hunter | Flare Dash? |
| Diagonals | Mixed | TBD | TBD |

---

## Acquisition Flow

```
PLAYER ENTERS CORRIDOR (direction X)
    ↓
FIGHTS THROUGH ENEMIES
    ↓
REACHES BOSS (boss X)
    ↓
DEFEATS BOSS
    ↓
FRAGMENT DROPS
    ↓
TOOL TRANSFORMS (automatic)
    ↓
PROCEEDS TO SHOP
```

No manual collection or activation — transformation is immediate.

---

## Order Strategy

Players can attempt bosses in any order, but:

| Consideration | Effect |
|---------------|--------|
| Tool synergy | Some upgrades help with other bosses |
| Difficulty | Some bosses harder than others |
| Escaped boss risk | Wake hard boss early = haunted arena |
| Sol progress | Need specific upgrades to reach Sol core |

**Recommended first-timer order:** TBD based on difficulty tuning

---

## Sol Assault Requirements

Not all 8 upgrades are needed for Sol surface. Deeper layers require more:

| Sol Layer | Upgrades Needed | Key Upgrades |
|-----------|-----------------|--------------|
| Approach | 1-2 | Corona Ward (heat) |
| Surface | 3-4 | Solar Lance (plasma), Core Piercer (crust) |
| Mantle | 5-6 | Nova Charge (cracking), Stellar Collector (plasma harvest) |
| Core | 7-8 | All, including final boss counter |

---

## Visual/Audio Treatment

| Moment | Treatment |
|--------|-----------|
| Fragment drop | Dramatic pause, glow effect |
| Transformation | Tool morphs visually, sound sting |
| First use | Tutorial popup? Or discovery? |
| UI indicator | Show celestial status in HUD |

---

## Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| Fragment count | 8 | One per boss |
| Upgrade type | Replace | Not additive |
| Order | Player choice | Any sequence |
| Partial completion | Allowed | Can attempt Sol with fewer |

---

## Open Questions

1. **8th upgrade** — What fills the final slot?
2. **Boss-direction pairing** — Which boss guards which direction?
3. **Difficulty ordering** — Is there a "correct" order?
4. **Partial Sol attempts** — How far can you get with fewer upgrades?
5. **Visual design** — What do celestial tools look like?
6. **New Game+** — Keep upgrades? Reset?

---

## Related Specifications

- `/levels/solar_system/Corridors.md` — Corridor structure
- `/levels/solar_system/Sol_Chunk.md` — What upgrades enable
- `/entities/npcs/Corridor_Bosses.md` — Boss definitions
- `/systems/Portal_Stitching.md` — How you reach bosses

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2025-01-15 | Initial specification, extracted from Solar System Arena |
