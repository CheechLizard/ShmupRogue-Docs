# Combat Pressure System Specification v0.1

**Location:** `/systems/`

---

## Overview

Combat pressure creates the Defender/Horde Survivor intensity. Players are under almost constant attack. Enemy density scales with time, progress, and zone.

---

## Design Goals

- **Constant engagement** — Can't ignore combat
- **Escalating threat** — Pressure builds over time
- **Strategic zones** — Some areas safer than others
- **Player agency** — Route planning matters

---

## Pressure Sources

Difficulty compounds from three independent sources:

| Source | Mechanism | Feel |
|--------|-----------|------|
| Time | Longer in arena = more spawns | Vampire Survivors style |
| Progress | More bosses killed = higher baseline | Permanent escalation |
| Zone | Spatial density varies | Strategic navigation |

**Combined formula:**
```
spawn_rate = base_rate × time_multiplier × progress_multiplier × zone_multiplier
```

---

## Time Scaling

The longer you stay, the harder it gets.

| Time in Arena | Multiplier | Notes |
|---------------|------------|-------|
| 0-5 min | 1.0x | Baseline |
| 5-10 min | 1.2x | Warming up |
| 10-20 min | 1.5x | Serious |
| 20-30 min | 2.0x | Intense |
| 30+ min | 2.5x+ | Survival mode |

---

## Progress Scaling

Each boss kill permanently raises baseline.

| Bosses Killed | Multiplier | Notes |
|---------------|------------|-------|
| 0 | 1.0x | Starting state |
| 1-2 | 1.1x | Early game |
| 3-4 | 1.3x | Mid game |
| 5-6 | 1.5x | Late game |
| 7-8 | 1.8x | Endgame push |

---

## Zone Density

Zones have base density levels. Enemies cluster around points of interest.

| Zone Type | Density | Notes |
|-----------|---------|-------|
| Open space | Low | Relatively safe travel |
| Near planet | Medium | Defenders, patrols |
| Near POI | Medium | Station guards |
| Planet surface | High | Ground forces |
| Near Sol | Very High | Solar defenders |
| Asteroid belt | Variable | Ambush potential |

---

## Enemy Types

| Type | Behavior | Spawn Context |
|------|----------|---------------|
| Fodder | Swarm, low HP | Everywhere, high volume |
| Fighter | Dogfight, chase | Patrols, pursuit |
| Bomber | Attack runs | Planet defense |
| Turret | Stationary, ground | Planet surface |
| Elite | Tough, abilities | Rare spawn, zone guards |
| Mini-boss | Very tough | POI defense, random |

---

## Spawn Patterns

| Pattern | Behavior |
|---------|----------|
| Continuous trickle | Constant low-rate spawns |
| Wave burst | Periodic large spawns |
| Pursuit squad | Spawns behind player |
| Ambush | Triggered by entering zone |
| Escalation | Ramp up over engagement |

---

## Difficulty Zones

The minimap shows danger levels.

| Color | Danger | Meaning |
|-------|--------|---------|
| Green | Low | Safe-ish travel |
| Yellow | Medium | Expect combat |
| Orange | High | Heavy resistance |
| Red | Very High | Serious threat |

---

## Balance Considerations

**Too intense:**
- Player overwhelmed constantly
- No breathing room
- Frustrating, exhausting

**Too sparse:**
- Feels empty
- Loses arcade intensity
- Combat becomes optional

**Target feel:**
- Always something to shoot
- Moments of intensity punctuated by brief lulls
- Player controls pace through positioning

---

## Interactions

| System | Interaction |
|--------|-------------|
| Fuel | Enemies = fuel source, pressure drives engagement |
| Minimap | Shows density zones for planning |
| Progression | Difficulty scales with boss kills |
| Harvesting | Must fight through defenders to reach planets |

---

## Parameters

| Parameter | Default | Range | Notes |
|-----------|---------|-------|-------|
| `base_spawn_rate` | TBD | 1-50 | Enemies per minute |
| `time_scale_rate` | TBD | 0.01-0.1 | Multiplier increase per minute |
| `progress_scale` | TBD | 0.1-0.3 | Multiplier increase per boss |
| `zone_density_*` | TBD | 0.5-3.0 | Per zone type multiplier |
| `max_active_enemies` | TBD | 50-500 | Performance cap |

---

## Open Questions

1. **Spawn cap** — Max enemies on screen? Performance vs intensity.
2. **Despawn rules** — Do distant enemies despawn?
3. **Squad AI** — How coordinated are groups?
4. **Retreat behavior** — Do enemies ever flee?
5. **Zone generation** — How are density zones placed procedurally?

---

## Related Specifications

- `/systems/Fuel.md` — Enemy drops
- `/interface/HUD_Minimap.md` — Density visualization
- `/entities/NPCs.md` — Enemy definitions
- `/levels/solar_system/Arena_Generation.md` — Zone placement

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2025-01-15 | Initial specification, extracted from Solar System Arena |
