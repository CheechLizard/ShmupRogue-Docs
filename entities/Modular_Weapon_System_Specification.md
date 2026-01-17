# Modular Weapon System – Design Specification v1.1

## Overview

The weapon system is a component-based architecture that allows weapons to be constructed by chaining together directed graphs of interconnected modules. This enables procedural generation of unique weapons that can be mounted on player ships, satellites, or enemy NPCs.

All weapon behavior—guns, beams, AOEs, swarms, splitters, carrier projectiles—is built from the same module types. Modules are simple, composable, and interchangeable. A gun can fire more guns; a beam can spawn rockets; a cluster grenade can create lasers which spawn blades. All built from the same primitives.

---

## Design Principles

**Unrestricted Player Creativity**  
The system prioritizes creative freedom above all else. Energy is the core balancing mechanism, naturally shaping weapon behavior without rigid formulas.

**Emergent Complexity Through Composition**  
Simple modules combine to create complex emergent behaviors. Weapons remain deterministic and learnable with fixed inputs and seeds.

---

## Core Mechanics Summary

**Weapons are directed graphs of modules.**  
Each weapon is built by connecting modules together in a graph. Modules are simple components that perform one job—supply energy, start strikes, modify trajectory, simulate motion, apply effects.

**Strikes flow through the graph.**  
Firing a weapon creates strikes that move through the module graph as events. Each module processes these events and may modify or react to them.

**Energy pays for strike creation.**  
Energy is the fundamental constraint. Each weapon has a continuous energy supply from BATTERY modules. Creating a strike costs energy. If you can't pay, the strike doesn't happen. This naturally limits fire rates, projectile counts, and complexity.

**Domains define strike lifecycle and attachment.**  
Strikes exist in one of three domains:
- **Weapon Domain** — strikes attached to the weapon (beams, melee, channels). Player release can end them.
- **Inflight Domain** — detached projectile strikes. Ignore further player input.
- **Post-Strike Domain** — follow-up strikes spawned from hits or strike endings.

**Strike State carries the data.**  
Each strike carries Strike State—trajectory, speed, damage values, timers, lifetime. Modules read and write to Strike State as events flow through them.

---

## Module Fundamentals

The system has six module types. Each module performs a single role:

1. **BATTERY** — provides energy per second
2. **TRIGGER** — starts strikes from player input or conditions
3. **REPEATER** — converts held firing into repeated strikes
4. **BARREL** — sets aim, spread, and range; routes to multiple outputs
5. **STRIKER** — simulates the strike (projectile, beam, melee, field)
6. **PAYLOAD** — applies damage, healing, buffs, debuffs

**Modules are single-input nodes.** Each module accepts events from one upstream connection. BARREL may have multiple outputs for multi-barrel routing.

**Modules don't share state.** Each module only affects the current strike flowing through it. No cross-strike memory or persistent state.

**Modules combine through simple rules.** Most modules either SET values, ADD to them, or MULTIPLY them. The order matters—modules process events in graph order.

---

## Strike State

Strike State is the data structure carried by events through the weapon graph. It represents one strike instance.

**Created when TRIGGER emits a Fire event.**
Strike State starts fresh when a TRIGGER creates a new strike.

**Contains:**
- **Trajectory** — `weapon_aim` (current weapon orientation), `strike_aim` (defaults to random, overridden by BARREL), spread, range caps, targeting metadata
- **Motion** — speed, acceleration, motion phases
- **Damage / effect metadata** — base values, types, tags for PAYLOAD
- **Timers** — repeat, payload, motion, and general-purpose timers
- **Lifetime** — lifetime or distance limits, if used

**Modified by modules as events flow.**  
As events pass through the graph, modules write to Strike State:
- BARREL sets trajectory and range
- STRIKER sets motion and collision properties
- Modules combine values using SET, ADD, or MULT rules

**Carried by events.**
Strike State travels with Fire events. It represents the complete state of one strike.

**Strike State contains configuration parameters, not live simulation data.** Position, velocity, and other runtime state are managed by the game engine during strike execution.

*(See Appendix for complete field definitions and defaults.)*

---

## Events

Events are messages that flow through the weapon graph. There are two event types:

### Fire
The universal event for strike progression. Carries Strike State plus position.
- Created by TRIGGER modules from player input or configured conditions
- Created by STRIKER modules when a strike ends (hit or miss)
- Contains optional `target` field: present if strike hit something, absent if it missed
- Modules can chain arbitrarily because they all communicate via Fire events

**Fire event fields:**
- `strikeState` — the strike configuration
- `spawnX`, `spawnY` — position where the event originated
- `targetX`, `targetY` — impact/end position (may differ from spawn for projectiles)
- `targetEntity` — (optional) entity that was hit, nil if miss

### Release
Ends weapon-domain strikes. Carries no Strike State.
Created by TRIGGER modules on player release.
Flows down the graph to reach active STRIKER modules. Only affects strikes in Weapon Domain (beams, melee, channels) that are not durable.

**Events flow through modules in graph order.**  
Each module processes incoming events and may:
- Forward them unchanged
- Modify Strike State on the event
- Consume the event
- Emit new events

---

## Energy System

Energy is the fundamental balancing mechanism. It determines what weapons can sustain.

### Energy Supply

**BATTERY modules provide continuous energy.**  
Each BATTERY adds energy per second to its downstream path. Multiple BATTERIES stack additively along a path.

**Energy is path-local.**  
Each path through the graph has its own rail that carries energy downstream from the BATTERY(IES).

### Energy Spending

**Energy is spent only when creating new strikes.**
TRIGGER and REPEATER are the only energy-aware modules. When either emits a Fire event, it pays the full precomputed strike cost.

**Costs are final—misses do not refund energy.**  
Once a strike is created and paid for, the energy is spent regardless of whether the strike hits.

### Strike Cost Calculation

**Strike cost is precomputed when the weapon is built.**  
The editor calculates the full cost by summing all downstream modules and any follow-up strikes created by downstream TRIGGERs. Strike costs are calculated and validated during weapon construction—if energy supply cannot sustain the weapon's firing pattern, the build is rejected.

**After strike creation, downstream modules don't check energy.**  
Once a strike is paid for, it executes to completion. STRIKER, PAYLOAD, and downstream modules never check or consume energy.

---

## Detailed Module Descriptions

### 1. BATTERY
**Purpose**  
Adds power to the downstream path.

**Properties**  
`energy_per_second`

**Strike State Combination Rules**  
- `energy_per_second` → **ADD** to the path energy rail.

**Behavior**  
- Contributes to the path's energy rail for all downstream nodes.
- Multiple BATTERY modules stack additively.
- BATTERY placement in the graph determines energy distribution: BATTERYs placed after branch points (like BARREL outputs) only affect that specific branch.

**Event Handling**  
- Forwards all events unchanged.

---

### 2. TRIGGER
**Purpose**
Starts strikes from player input or from configured conditions.

**Properties**
`condition` — defines when to emit Fire/Release events:
- **player_press**: Fire on player input
- **player_hold**: Fire while player holds input
- **on_fire**: Fire when receiving any Fire event (chain reactions)
- **on_target**: Fire only when receiving Fire event with target (hit something)
- **on_no_target**: Fire only when receiving Fire event without target (missed)

**Strike State Combination Rules**
- No numeric combination; TRIGGER creates a fresh Strike State on Fire.
- Stores `weapon_aim` from current weapon orientation at the moment of trigger.
- Initializes `strike_aim` to a random direction (overridden by BARREL if present).

**Behavior**
- On configured trigger condition: emits one Fire event with new Strike State.
- On player release (if player-triggered): emits one Release.
- On matching Fire event (if configured): emits one Fire seeded from that event. New Fire events inherit the triggering event's position as their spawn location.
- Pays the strike cost when emitting Fire events. Strike costs are calculated and validated during weapon construction.

**Event Handling**
- Forwards incoming Fire events unchanged.
- Emits new Fire events from configured conditions.

---

### 3. REPEATER
**Purpose**
Produces Fire events at a fixed cadence while firing window is open.

**Properties**
`fire_rate` — rate of Fire event emission when firing
`generation_time` — time to regenerate one stored strike
`capacity` — max stored strikes (0 = infinite capacity, no cooldown)

**Strike State Combination Rules**
- **N/A** — REPEATER does not modify Strike State.

**Behavior**
- On receiving Fire event: opens firing window, begins emitting Fire events at `fire_rate` while strikes are available.
- On receiving Release event: closes firing window, stops emitting.
- Each emitted Fire clones the incoming Strike State (preserving energy context), then updates `weapon_aim` from current weapon orientation and re-randomizes `strike_aim`.
- Regenerates strikes at 1/`generation_time` rate.
- Full reload time = `generation_time` × `capacity`.
- When capacity is 0: infinite strikes, no cooldown period.

*Implementation Note: REPEATER requires branch-local storage for maintaining firing state and strike availability independently across branches.*

**Event Handling**
- Consumes Fire to open firing window.
- Forwards Release only when closing an open fire window; otherwise swallows it.

---

### 4. BARREL
**Purpose**
Defines aim, spread, and range; routes events through barrel ports.

**Properties**
`barrel_count` — number of output barrels
`angle` — aim offset from weapon orientation (degrees)
`spread` — total arc to distribute barrels across (degrees). For N barrels, they are evenly spaced across this arc centered on `angle`
`spread_variance` — random variance added per shot (degrees)
`range_max` — maximum strike range
`last_barrel` — tracks round-robin state

**Strike State Combination Rules**
- `strike_aim` → **SET** to `weapon_aim` + barrel angle (computed from `angle` and `spread`).
- `spread_variance` → applied as random offset.
- `range_max` → **SET** if less than current value.

**Behavior**
- Routes events round-robin across barrel outputs using `last_barrel` tracker.
- Clones Strike State for each barrel output—each branch operates independently with its own copy.
- Energy is divided evenly across barrel outputs. Example: 100 energy/sec → 3 barrels = ~33.3 energy/sec per barrel.
- Computes barrel angles: for 3 barrels with `spread=30`, angles are -15°, 0°, +15° relative to `angle`.
- Applies `spread_variance` randomization around the computed direction.

**Event Handling**
- Forwards all events through round-robin barrel routing.

---

### 5. STRIKER
**Purpose**
Creates and simulates projectile/beam strikes, emitting Fire events downstream when strikes end.

**Properties**
`trigger_behavior` — strike behavior mode:
- **Consumable**: Strike ends after first collision (bullets, missiles). Emits one Fire downstream.
- **Retriggerable**: Strike can hit multiple times over its lifetime (beam DPS ticks, melee blades). Emits Fire each tick.

`durable` — If true, ignores Release events and runs to completion
`expires` — If true, emits Fire on natural completion (range/lifetime exhausted); if false, strike ends silently
`motion_type` — movement pattern (Linear, Gravity, Sine)
`base_speed` — initial speed at launch
`speed_variance` — random variance added to base speed
`speed_multiplier` — multiplier applied to speed
`acceleration` — rate of speed change
`gravity` — downward acceleration (for lobbed projectiles)
`lifetime_max` — maximum time before strike ends
`collision_size` — collision radius or extent
`visual_params` — type-specific visual parameters (passed to renderer)

**Strike State Combination Rules**
- `base_speed` → **SET** speed at launch.
- `speed_multiplier` → **MULT** speed.
- `acceleration` → **ADD** to acceleration.
- `gravity` → **SET** downward force.
- behavior/collision/visual fields → **SET**.

**Behavior**
- Consumes Fire to create a strike instance.
- Projectile strikes detach to Inflight Domain after launch.
- Beam/melee strikes remain in Weapon Domain while active.
- When strike ends (collision or expiration), emits Fire downstream with:
  - `targetEntity` set if something was hit, nil if missed
  - `targetX`, `targetY` set to impact/end position
- Retriggerable strikes emit Fire events repeatedly while active.
- Consumable strikes emit one Fire maximum when they end.
- Durable strikes ignore Release events; non-durable strikes in Weapon Domain end on Release.

**Event Handling**
- Consumes Fire to create strike.
- Receives Release and ends non-durable weapon-domain strikes.
- Emits Fire downstream when strike ends (with target info if hit).

---

### 6. PAYLOAD
**Purpose**
Applies effects when receiving Fire events: damage (if target exists) and effects (always).

**Properties**
Damage:
`damage_radius` — area of effect radius for terrain damage
`damage_power` — damage amount

Effects (data-driven via `data/effects/*.json`):
`effect` — name of effect definition to trigger (e.g., "explosion", "black_hole", "acid_spray")
`effect_params` — optional parameter overrides merged with effect definition

*Effects are defined in JSON files under `data/effects/`. Each effect specifies its type (particle spawner, attractor, terrain modifier, etc.) and parameters. PAYLOAD triggers the named effect at the event position.*

**Strike State Combination Rules**
- **N/A** — PAYLOAD reads Strike State but does not modify it.

**Behavior**
- On receiving Fire event:
  - If `targetEntity` exists: apply damage to entity
  - If no entity but position valid: apply terrain damage at radius
  - If `effect` is configured: trigger effect at event position
- Forwards the incoming Fire event unchanged.

**Event Handling**
- Reacts to Fire events.
- Applies damage if event has target, triggers effects unconditionally.
- Forwards all events unchanged.

---

## Graph Structure
- Modules are 1-input → 1-output. BARREL may have multiple outputs.
- No module accepts multiple inputs.
- BATTERY contributions accumulate along a path and divide at BARREL branches.

---

## Example Builds

### Example 1: Cone Beam DPS
```text
[TRIGGER] → [BATTERY] → [REPEATER] → [BARREL] → [STRIKER] → [PAYLOAD]
```
- High-rate REPEATER creates rapid Fire events.
- STRIKER is retriggerable/non-durable, emits Fire with target each tick.
- PAYLOAD applies DPS per Fire event (damage if target exists).
- Player release ends the beam (non-durable).

### Example 2: Projectile → AOE Field
```text
[TRIGGER] → [BATTERY] → [BARREL] → [STRIKER] → [PAYLOAD] → [TRIGGER on_target] → [BATTERY] → [STRIKER] → [PAYLOAD]
```
- First strike is a consumable projectile with impact PAYLOAD.
- When projectile hits, STRIKER emits Fire with target.
- Downstream TRIGGER (condition: on_target) spawns follow-up Fire.
- Second strike is a durable stationary field applying DPS while active.

### Example 3: Acid Gun (No Striker)
```text
[TRIGGER] → [BATTERY] → [REPEATER] → [BARREL] → [PAYLOAD effect:acid_spray]
```
- Fire flows directly from BARREL to PAYLOAD (no STRIKER).
- PAYLOAD triggers acid_spray effect at event position on each Fire.
- No damage (no target in event from BARREL).

### Example 4: Gravity Gun
```text
[TRIGGER] → [BATTERY] → [BARREL] → [STRIKER gravity:80] → [PAYLOAD effect:black_hole]
```
- Lobbed projectile travels, then STRIKER emits Fire when it ends.
- Fire has target if projectile hit something, nil if it expired.
- PAYLOAD triggers black_hole effect at impact/end position regardless of hit/miss.

### Example 5: Asymmetric Triple Barrel
```text
                        ┌─[BARREL port 1]─[STRIKER]─[PAYLOAD]
                        │
[TRIGGER]─[BATTERY 1]─[BARREL 3-way]─[BARREL port 2]─[STRIKER]─[PAYLOAD]
                        │
                        └─[BARREL port 3]─[BATTERY 2]─[STRIKER]─[PAYLOAD]
```
- BATTERY 1 (90 energy/sec) placed before 3-way BARREL → each port gets 30 energy/sec
- Ports 1 & 2: 30 energy/sec each (standard power)
- Port 3: 30 + 60 from BATTERY 2 = 90 energy/sec (super barrel)
- Result: Two weak barrels, one powerful barrel—asymmetric distribution by design.

---

## Appendix: Strike State Fields

Strike State is created fresh by TRIGGER and modified by modules as events flow through the graph. Below are all fields, their defaults, and how modules interact with them.

### Trajectory

| Field | Type | Default | Description | Module Interactions |
|-------|------|---------|-------------|-------------------|
| `weapon_aim` | Vector2 | Set by TRIGGER | Current weapon orientation at trigger moment | SET by TRIGGER from weapon |
| `strike_aim` | Vector2 | Random unit vector | Direction the strike will travel | Defaults to random; SET by BARREL to `weapon_aim` + barrel angle |
| `angle_spread` | Float (degrees) | 0.0 | Per-shot spread variance | SET by BARREL |
| `range_max` | Float (units) | ∞ | Maximum distance strike can travel | SET by BARREL (if less than current) |

### Motion

| Field | Type | Default | Description | Module Interactions |
|-------|------|---------|-------------|-------------------|
| `motion_type` | Enum | Linear | Movement pattern (Linear, Gravity, Sine) | SET by STRIKER |
| `base_speed` | Float (units/sec) | 0.0 | Initial speed at launch | SET by STRIKER |
| `speed_multiplier` | Float | 1.0 | Multiplier applied to speed | MULT by STRIKER |
| `acceleration` | Float (units/sec²) | 0.0 | Rate of speed change | ADD by STRIKER |
| `gravity` | Float (units/sec²) | 0.0 | Downward acceleration for lobbed projectiles | SET by STRIKER |
| `inherited_vx` | Float (units/sec) | 0.0 | Inherited X velocity from shooter | SET by TRIGGER/REPEATER |
| `inherited_vy` | Float (units/sec) | 0.0 | Inherited Y velocity from shooter | SET by TRIGGER/REPEATER |

### Collision

| Field | Type | Default | Description | Module Interactions |
|-------|------|---------|-------------|-------------------|
| `collision_size` | Float (units) | 1.0 | Collision radius or extent | SET by STRIKER |

### Behavior

| Field | Type | Default | Description | Module Interactions |
|-------|------|---------|-------------|-------------------|
| `trigger_behavior` | Enum | Consumable | Strike behavior mode (Consumable, Retriggerable) | SET by STRIKER |
| `durable` | Bool | false | If true, ignores Release events | SET by STRIKER |
| `expires` | Bool | true | If true, emits Fire on natural completion; if false, ends silently | SET by STRIKER |

### Damage & Effects

| Field | Type | Default | Description | Module Interactions |
|-------|------|---------|-------------|-------------------|
| `base_damage` | Float | 0.0 | Base damage value | Read by PAYLOAD |
| `aoe_radius` | Float (units) | 0.0 | Area of effect radius (0 = single target) | Read by PAYLOAD |

### Lifetime

| Field | Type | Default | Description | Module Interactions |
|-------|------|---------|-------------|-------------------|
| `lifetime_max` | Float (seconds) | ∞ | Maximum time before strike ends | Used by STRIKER with `expires` |

### Visual

| Field | Type | Default | Description | Module Interactions |
|-------|------|---------|-------------|-------------------|
| `visual_params` | Table | {} | Type-specific visual parameters passed to renderer | SET by STRIKER |

### Notes

- **SET** operations replace the value completely
- **ADD** operations accumulate values
- **MULT** operations multiply the current value
- Fields not modified by any module retain their defaults
- PAYLOAD only reads Strike State to determine effect parameters, then applies effects directly to game entities
- Inherited velocity allows projectiles to maintain relative motion with moving shooters
