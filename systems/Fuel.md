# Fuel System Specification v0.1

**Location:** `/systems/`

---

## Overview

Fuel is the survival resource. Players must kill NPCs to generate fuel. Running out ends the run. This forces constant combat engagement — you cannot hide and harvest.

---

## Fuel Loop

```
KILL NPC
    ↓
NPC DROPS CELLS
    ↓
COLLECT CELLS → FUEL
    ↓
FUEL DRAINS (constant + actions)
    ↓
EMPTY = RUN OVER
```

---

## Drain Model

Fuel drains from two sources:

| Source | Rate | Notes |
|--------|------|-------|
| Baseline | Constant per second | Always ticking down |
| Actions | Per use | Boost, weapons, shields |

**Formula:**
```
drain_per_second = base_drain + (action_costs / time)
```

Player must kill enough NPCs to outpace drain.

---

## Parameters

| Parameter | Default | Range | Notes |
|-----------|---------|-------|-------|
| `base_drain` | TBD | 0.1 - 10 | Fuel/second baseline |
| `boost_cost` | TBD | 1 - 20 | Fuel per boost |
| `weapon_cost` | TBD | 0 - 10 | Fuel per shot (weapon-dependent?) |
| `shield_cost` | TBD | 1 - 10 | Fuel per second while active |
| `max_fuel` | TBD | 100 - 1000 | Tank capacity |

---

## Cell Drops

NPCs drop fuel cells on death.

| NPC Type | Cell Value | Notes |
|----------|------------|-------|
| Fodder | Low | Common, many kills needed |
| Elite | Medium | Worth hunting |
| Mini-boss | High | Risk/reward |
| Boss | Very high | Corridor bosses |

---

## Fuel States

| State | Threshold | Feedback |
|-------|-----------|----------|
| Full | 100% | Normal HUD |
| Healthy | 50-99% | Normal HUD |
| Low | 20-49% | Warning color, audio cue |
| Critical | 1-19% | Flashing, alarm, screen effect? |
| Empty | 0% | Run over |

---

## Balance Considerations

**Too punishing:**
- Player spends all time chasing fodder
- No time for harvesting or exploration
- Feels like work

**Too lenient:**
- Player ignores combat
- Hides and harvests
- Loses intensity

**Target feel:**
- Combat is mandatory but not overwhelming
- Skilled players can build surplus
- Poor play creates fuel anxiety

---

## Interactions

| System | Interaction |
|--------|-------------|
| Combat Pressure | More enemies = more fuel opportunities |
| Progression | Upgrades might reduce drain or increase capacity |
| Harvesting | Must balance combat time vs harvest time |
| Difficulty zones | Denser zones = more fuel, more danger |

---

## Open Questions

1. **Drain rate tuning** — What numbers feel right? Needs playtesting.
2. **Weapon fuel costs** — Per weapon, or flat rate?
3. **Fuel capacity upgrades** — Available in shop?
4. **Cell magnet** — Auto-collect nearby cells, or manual pickup?
5. **Cell decay** — Do dropped cells despawn over time?

---

## Related Specifications

- `/systems/Combat_Pressure.md` — Enemy spawning
- `/interface/HUD_Fuel_Gauge.md` — Display
- `/entities/NPCs.md` — Drop rates per enemy type

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2025-01-15 | Initial specification, extracted from Solar System Arena |
