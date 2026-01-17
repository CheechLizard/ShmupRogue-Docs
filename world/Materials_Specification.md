# Materials Specification v0.3

## Overview

Materials are the core resource of Shmup Rogue. Each material has three defining properties:

| Property | Options | Job |
|----------|---------|-----|
| **State** | 4 | How it exists in the physics simulation |
| **Reactivity** | 8 | How it interacts with other materials and energy |
| **Effect Profile** | 8 axes | Why NPCs value it (economic demand) |

Materials are organized by biome. Each biome has a Primary and Secondary Reactivity that defines its identity and progression.

---

## Part 1: States

How the material exists in the physics simulation. Determines movement, collision, rendering.

| State | Physics Behavior | Visual | Effect Modifier |
|-------|------------------|--------|-----------------|
| Solid | Fixed position, shatters on impact | Chunky, defined edges | Slow onset, long duration |
| Particle | Falls, piles, spreads on surfaces | Granular, dusty | Fast absorption, short peak |
| Liquid | Flows, pools, fills containers | Shiny, reflective surface | Smooth curve, easy dosing |
| Gas | Rises, disperses, fills volumes | Translucent, wispy | Instant hit, hard to control |

---

## Part 2: Reactivities

How the material interacts with other materials and energy. The chemistry layer.

### Reactivity Behavior Table

| Reactivity | Trigger | Result | Chain Behavior | Visual Cue |
|------------|---------|--------|----------------|------------|
| Stable | — | Nothing | No | Default |
| Flammable | Heat ≥ igniteTemp | Burns, emits heat | Yes → heats neighbors → spreads fire | Orange glow, flames |
| Explosive | Impact ≥ detonateImpact OR Heat ≥ detonateTemp | Instant AOE damage + heat burst | Yes → triggers nearby Explosive/Flammable | Red warning, flash |
| Corrosive | Contact (active material) | Erodes target (chance = erodeStrength vs erosionResistance) | Slow spread through contact | Green, bubbling |
| Radioactive | Always on | DOT damage field | Contaminates neighbors? (TBD) | Glow shader, particles |
| Conductive | Energy source contact | Passes energy along path | Yes → chains through Conductive materials | Arc effects, blue crackle |
| Optical | Light/laser contact | Redirects beam at angle | Redirect only, no spread | Prismatic, refractive |
| Organic | Time / Damage / Pile size | Regrows / Rots / Spawns NPC | TBD per behavior | Fleshy, pulsing |

Note: Thresholds (igniteTemp, detonateTemp, detonateImpact, erosionResistance, etc.) are per-material properties. See Part 6: Material Properties Schema.

### Flammable vs Explosive Detail

| Property | Trigger | Effect | Duration | Chain |
|----------|---------|--------|----------|-------|
| Flammable | Heat ≥ material.igniteTemp | Burns, emits heat to neighbors | Over time (burnRate) | Fire spreads outward |
| Explosive | Impact ≥ material.detonateImpact OR Heat ≥ material.detonateTemp | Instant AOE damage + heat burst | One-shot | Triggers neighbors instantly |

**Key difference:** Flammable is slow burn, Explosive is instant. Flammable gas fills a room then burns through it. Explosive gas detonates the whole pocket at once.

### Heat Interaction by State

| State | Heat Applied | Low Threshold | High Threshold |
|-------|--------------|---------------|----------------|
| Solid | Accumulates slowly | Warms | Melts → Liquid (if meltable) |
| Particle | Accumulates fast (surface area) | Ignites easily | Burns out fast |
| Liquid | Accumulates medium | Warms, convects | Evaporates → Gas |
| Gas | Accumulates fast, disperses | Warms, expands | Ignites (if Flammable) / Plasma? (TBD) |

### Corrosive Detail

| Corrosive State | Behavior |
|-----------------|----------|
| Solid | Inert until wet → activates on liquid contact |
| Particle | Irritant, erodes slowly on contact |
| Liquid | Active erosion, flows into cracks, pools |
| Gas | Acid vapor, erodes what it touches, disperses |

### Radioactive Detail

| Property | Behavior |
|----------|----------|
| Damage | DOT in radius, ignores armor? |
| Radius | Based on material quantity |
| Contamination | TBD — spreads to neighbors or stays contained? |
| Decay | Permanent or fades over time? |

### Conductive Detail

| Property | Behavior |
|----------|----------|
| Energy flow | Follows connected Conductive materials |
| Branching | Splits at junctions, divides power |
| Overload | Too much energy → heat buildup → melts/explodes? |
| Activation | Powers machines, opens doors, triggers traps |

### Optical Detail

| Property | Behavior |
|----------|----------|
| Reflection | Bounces beam at angle |
| Refraction | Bends beam through material |
| Focus | Concentrates beam → higher damage/heat |
| Diffusion | Scatters beam → lower damage, wider area |

### Organic Detail (TBD — Engine R&D)

| Behavior | Trigger | Result |
|----------|---------|--------|
| Regrows | Time + adjacent Organic | Material regenerates slowly |
| Rots | Time + damage + no neighbors | Material degrades, quality drops |
| Spawns | Pile size threshold | NPC emerges from pile |
| Spreads | Contact + nutrients? | Grows into adjacent empty space |

---

## Part 3: Effect Profile

Why NPCs value the material. The economic layer.

Scale: -3 (extreme low) to +3 (extreme high), 0 = neutral.

| Axis | -3 | 0 | +3 |
|------|-----|---|-----|
| **Energy** | Comatose | Baseline | Manic |
| **Mood** | Despair | Neutral | Ecstatic |
| **Focus** | Cannot think | Normal | Tunnel vision |
| **Social** | Total isolation | Normal | No boundaries |
| **Sensation** | Completely numb | Normal | Overwhelmed |
| **Perception** | Hyperreal | Normal | Dissociated |
| **Ego** | No self-worth | Normal | God complex |
| **Libido** | Dormant | Normal | Insatiable |

### Reactivity → Effect Tendencies

| Reactivity | Primary | Secondary | Flavor |
|------------|---------|-----------|--------|
| Stable | Neutral | — | Inert = boring high |
| Flammable | +Energy | +Sensation | Burns hot, feels intense |
| Corrosive | -Mood | +Perception | Eats at you, hyperaware |
| Explosive | +Energy, +Ego | -Focus | Power rush, scattered |
| Radioactive | +Perception | -Sensation | Dissociation, numbness |
| Conductive | +Focus | +Energy | Wired, locked in |
| Optical | +Perception | +Mood | Trippy, beautiful |
| Organic | +Sensation | +Social | Warm, euphoric |

---

## Part 4: Biomes

Each biome has a Primary and Secondary Reactivity.

- **Primary** = Immediate. Defines the biome's core vocabulary. Players learn this first.
- **Secondary** = Unlocked later. Reveals depth. "Oh shit" moments.

| Biome | Primary | Secondary | Unlock Moment |
|-------|---------|-----------|---------------|
| Asteroid Belt | Stable | Explosive | Reach the mining charges |
| Volcanic Core | Flammable | Explosive | Break into the gas pockets |
| Sunken Sprawl | Conductive | Corrosive | Power hits the flooded sections |
| Spore Forest | Organic | Flammable | First forest fire |
| Acid Swamp | Corrosive | Organic | Find the living acid |
| Frozen Wreck | Stable | Conductive | Restore ship power |
| Carrion Depths | Organic | Radioactive | Reach the reactor-heart |
| Glitch Zone | Optical | ??? | Endgame |

### Biome Color Language

| Reactivity | Color Signal |
|------------|--------------|
| Stable | Grey, neutral |
| Flammable | Orange, red |
| Corrosive | Green, yellow-green |
| Explosive | Red, warning stripes |
| Radioactive | Sick yellow, green glow |
| Conductive | Blue, electric arc |
| Optical | Prismatic, white |
| Organic | Pink, purple, bioluminescent |

---

## Part 5: Material Grid (64 Materials)

8 biomes × 8 materials = 64 total.

Each biome gets:
- 1 Slop (Stable, worthless filler)
- ~3 Primary Reactivity materials (different States)
- ~2 Secondary Reactivity materials
- ~2 Flex (texture, variety)

### Distribution Template

| Slot | Reactivity | Role |
|------|------------|------|
| 1 | Stable | Slop — filler, always present |
| 2 | Primary | Main vocabulary |
| 3 | Primary | Different State variant |
| 4 | Primary | Third variant or rare |
| 5 | Secondary | Unlocked later |
| 6 | Secondary | Escalation variant |
| 7 | Flex | Seasoning |
| 8 | Flex | Seasoning |

---

## Part 7: Material Properties Schema

Every material has the same fields. Reactivity enum indicates which fields are active.

### Core Properties

| Property | Type | Notes |
|----------|------|-------|
| name | string | Display name |
| state | enum | Solid, Particle, Liquid, Gas (design language) |
| reactivity | enum | Stable, Flammable, Corrosive, Explosive, Radioactive, Conductive, Optical, Organic |
| effectProfile | float[8] | Economic value: En/Mo/Fo/So/Se/Pe/Eg/Li |

### Thermal Properties

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| heatConductivity | float | 0.5 | 0-1, how fast heat spreads through material |
| igniteTemp | int | 255 | 0-255, temperature to trigger Flammable behavior |
| detonateTemp | int | 255 | 0-255, temperature to trigger Explosive behavior |

Note: Heat capacity is derived from State, not stored per material.

### Impact Properties

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| detonateImpact | float | 9999 | Force threshold to trigger Explosive behavior |

### Erosion Properties

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| erosionResistance | float | 0.9 | 0-1, chance to resist erosion per frame |
| erodeStrength | float | 0 | 0-1, how strongly this material erodes others (Corrosive only) |
| decayRate | float | 0 | 0-1, chance per frame this material decays (Corrosive only) |

### Conductive Properties

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| conductivity | float | 0 | 0-1, how well it passes energy |
| maxLoad | float | 100 | Energy threshold before overload |

### Organic Properties (TBD)

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| regrowRate | float | 0 | Chance per frame to regenerate |
| rotRate | float | 0 | Chance per frame to decay |
| spawnThreshold | int | 0 | Pile size to trigger NPC spawn |

### Example: Stable Material

```json
{
  "name": "Ironite",
  "state": "Solid",
  "reactivity": "Stable",
  "effectProfile": [0, 0, 1, 0, 0, -1, 0, 0],
  "heatConductivity": 0.8,
  "igniteTemp": 255,
  "detonateTemp": 255,
  "detonateImpact": 9999,
  "erosionResistance": 0.95,
  "erodeStrength": 0,
  "decayRate": 0,
  "conductivity": 0.1,
  "maxLoad": 100
}
```

### Example: Flammable Material

```json
{
  "name": "Magmite",
  "state": "Liquid",
  "reactivity": "Flammable",
  "effectProfile": [3, 1, -1, 0, 2, 0, 1, 1],
  "heatConductivity": 0.7,
  "igniteTemp": 50,
  "detonateTemp": 255,
  "detonateImpact": 9999,
  "erosionResistance": 0.3,
  "erodeStrength": 0,
  "decayRate": 0,
  "conductivity": 0.2,
  "maxLoad": 50
}
```

### Example: Corrosive Material

```json
{
  "name": "Acidite",
  "state": "Liquid",
  "reactivity": "Corrosive",
  "effectProfile": [0, -2, 0, 0, 1, 1, -1, 0],
  "heatConductivity": 0.5,
  "igniteTemp": 255,
  "detonateTemp": 255,
  "detonateImpact": 9999,
  "erosionResistance": 0.99,
  "erodeStrength": 0.1,
  "decayRate": 0.002,
  "conductivity": 0.3,
  "maxLoad": 100
}
```

### Example: Explosive Material

```json
{
  "name": "Boomite",
  "state": "Solid",
  "reactivity": "Explosive",
  "effectProfile": [2, 0, -1, 0, 1, 0, 1, 0],
  "heatConductivity": 0.4,
  "igniteTemp": 255,
  "detonateTemp": 180,
  "detonateImpact": 50,
  "erosionResistance": 0.7,
  "erodeStrength": 0,
  "decayRate": 0,
  "conductivity": 0,
  "maxLoad": 20
}
```

---

## Part 8: Biome Material Lists

### Asteroid Belt

Primary: Stable | Secondary: Explosive

| # | Name | State | Reactivity | En | Mo | Fo | So | Se | Pe | Eg | Li | Notes |
|---|------|-------|------------|----|----|----|----|----|----|----|----|-------|
| 1 | Grite | Particle | Stable | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | Grey dust slop |
| 2 | Ironite | Solid | Stable | 0 | 0 | +1 | 0 | 0 | -1 | 0 | 0 | Common ore, mild focus |
| 3 |Ite | Particle | Stable | 0 | 0 | 0 | 0 | 0 | 0 | +1 | 0 |ite flakes |
| 4 | Voidstone | Solid | Stable | -1 | 0 | +1 | -1 | 0 | 0 | 0 | 0 | Dense, grounding |
| 5 | Boomite | Solid | Explosive | +2 | 0 | -1 | 0 | +1 | 0 | +1 | 0 | Mining explosive |
| 6 | Blastite | Particle | Explosive | +3 | -1 | -2 | 0 | +2 | 0 | +1 | 0 | Unstable powder |
| 7 | Sparite | Solid | Conductive | +1 | 0 | +1 | 0 | +1 | 0 | 0 | 0 | Trace conductor |
| 8 | Coldite | Solid | Stable | -1 | 0 | +1 | -1 | -1 | 0 | 0 | -1 | Dead cold rock |

---

### Volcanic Core

Primary: Flammable | Secondary: Explosive

| # | Name | State | Reactivity | En | Mo | Fo | So | Se | Pe | Eg | Li | Notes |
|---|------|-------|------------|----|----|----|----|----|----|----|----|-------|
| 9 | Ashite | Particle | Stable | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | Black powder slop |
| 10 | Embite | Solid | Flammable | +2 | +1 | 0 | 0 | +1 | 0 | 0 | 0 | Glowing coal |
| 11 | Pyrite | Particle | Flammable | +2 | 0 | -1 | 0 | +2 | 0 | 0 | 0 | Fire dust |
| 12 | Magmite | Liquid | Flammable | +3 | +1 | -1 | 0 | +2 | 0 | +1 | +1 | Liquid fire |
| 13 | Vaporite | Gas | Flammable | +3 | 0 | -2 | +1 | +1 | +1 | 0 | 0 | Flammable gas |
| 14 | Burstite | Solid | Explosive | +2 | 0 | -1 | 0 | +1 | 0 | +2 | 0 | Pressure bomb |
| 15 | Slagite | Liquid | Stable | 0 | -1 | 0 | 0 | 0 | 0 | 0 | 0 | Cooled waste |
| 16 | Obsidianite | Solid | Stable | 0 | 0 | +2 | -1 | 0 | -1 | +1 | 0 | Sharp, focused |

---

### Sunken Sprawl

Primary: Conductive | Secondary: Corrosive

| # | Name | State | Reactivity | En | Mo | Fo | So | Se | Pe | Eg | Li | Notes |
|---|------|-------|------------|----|----|----|----|----|----|----|----|-------|
| 17 | Rustite | Particle | Stable | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | Corroded metal slop |
| 18 | Voltite | Solid | Conductive | +2 | 0 | +2 | 0 | +1 | 0 | 0 | 0 | Power cell ore |
| 19 | Arcite | Liquid | Conductive | +2 | +1 | +1 | 0 | +2 | 0 | 0 | 0 | Charged fluid |
| 20 | Shockite | Gas | Conductive | +3 | 0 | +1 | 0 | +3 | +1 | 0 | 0 | Ionized air |
| 21 | Acidite | Liquid | Corrosive | 0 | -2 | 0 | 0 | +1 | +1 | -1 | 0 | Industrial acid |
| 22 | Etchite | Particle | Corrosive | 0 | -1 | +1 | 0 | 0 | +1 | 0 | 0 | Etching powder |
| 23 | Copperite | Solid | Conductive | +1 | 0 | +1 | 0 | 0 | 0 | 0 | 0 | Basic conductor |
| 24 | Murklite | Liquid | Stable | 0 | -1 | -1 | 0 | 0 | +1 | 0 | 0 | Murky water |

---

### Spore Forest

Primary: Organic | Secondary: Flammable

| # | Name | State | Reactivity | En | Mo | Fo | So | Se | Pe | Eg | Li | Notes |
|---|------|-------|------------|----|----|----|----|----|----|----|----|-------|
| 25 | Pulpite | Gel | Stable | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | Organic mush slop |
| 26 | Sporeite | Gas | Organic | 0 | +1 | -1 | +1 | +1 | +2 | 0 | 0 | Floating spores |
| 27 | Fungite | Solid | Organic | -1 | +1 | 0 | +1 | +1 | +1 | 0 | 0 | Mushroom matter |
| 28 | Sapite | Liquid | Organic | +1 | +2 | 0 | +1 | +2 | 0 | 0 | +1 | Tree sap |
| 29 | Pollenite | Particle | Organic | +1 | +1 | -1 | +2 | +1 | +1 | 0 | +1 | Pollen cloud |
| 30 | Kindlite | Solid | Flammable | +2 | 0 | 0 | 0 | +1 | 0 | 0 | 0 | Dry wood |
| 31 | Resinite | Liquid | Flammable | +1 | +1 | +1 | 0 | +1 | 0 | 0 | 0 | Flammable resin |
| 32 | Glowite | Solid | Organic | 0 | +2 | 0 | +1 | +1 | +1 | 0 | 0 | Bioluminescent |

---

### Acid Swamp

Primary: Corrosive | Secondary: Organic

| # | Name | State | Reactivity | En | Mo | Fo | So | Se | Pe | Eg | Li | Notes |
|---|------|-------|------------|----|----|----|----|----|----|----|----|-------|
| 33 | Sludite | Liquid | Stable | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | Green goop slop |
| 34 | Vitriolite | Liquid | Corrosive | +1 | -2 | +1 | 0 | +1 | +1 | 0 | 0 | Strong acid |
| 35 | Causticite | Gas | Corrosive | +1 | -2 | 0 | 0 | +2 | +2 | -1 | 0 | Acid vapor |
| 36 | Burnite | Particle | Corrosive | 0 | -1 | 0 | 0 | +2 | +1 | -1 | 0 | Irritant dust |
| 37 | Digestite | Liquid | Organic | 0 | -1 | 0 | 0 | +1 | +1 | 0 | 0 | Digestive enzyme |
| 38 | Leechite | Gel | Organic | -1 | 0 | 0 | +1 | +2 | 0 | -1 | 0 | Parasitic slime |
| 39 | Bogite | Solid | Organic | 0 | 0 | -1 | 0 | +1 | +1 | 0 | 0 | Compressed peat |
| 40 | Venomite | Liquid | Corrosive | +1 | -2 | +1 | -1 | +2 | +1 | 0 | 0 | Biological acid |

---

### Frozen Wreck

Primary: Stable | Secondary: Conductive

| # | Name | State | Reactivity | En | Mo | Fo | So | Se | Pe | Eg | Li | Notes |
|---|------|-------|------------|----|----|----|----|----|----|----|----|-------|
| 41 | Frostslop | Particle | Stable | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | Frozen nothing slop |
| 42 | Cryite | Solid | Stable | -2 | 0 | +1 | -1 | -1 | 0 | 0 | -1 | Pure cold |
| 43 | Glacite | Solid | Stable | -1 | +1 | +1 | 0 | 0 | -1 | 0 | 0 | Ancient ice |
| 44 | Hullite | Solid | Stable | 0 | 0 | +1 | 0 | 0 | -1 | +1 | 0 | Ship plating |
| 45 | Cellite | Solid | Conductive | +1 | 0 | +2 | 0 | 0 | 0 | +1 | 0 | Power cell |
| 46 | Cablite | Solid | Conductive | +1 | 0 | +1 | 0 | +1 | 0 | 0 | 0 | Frozen wiring |
| 47 | Cryogel | Liquid | Stable | -2 | 0 | 0 | 0 | -2 | 0 | 0 | -1 | Cryo fluid |
| 48 | Corpsicle | Solid | Organic | -1 | -1 | 0 | -1 | -1 | +1 | 0 | -1 | Frozen remains |

---

### Carrion Depths

Primary: Organic | Secondary: Radioactive

| # | Name | State | Reactivity | En | Mo | Fo | So | Se | Pe | Eg | Li | Notes |
|---|------|-------|------------|----|----|----|----|----|----|----|----|-------|
| 49 | Gorite | Gel | Stable | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | Visceral goop slop |
| 50 | Fleshite | Solid | Organic | 0 | 0 | 0 | +1 | +2 | 0 | 0 | +1 | Wall meat |
| 51 | Veinite | Liquid | Organic | +1 | +1 | 0 | +1 | +2 | 0 | 0 | +1 | Blood analog |
| 52 | Nervite | Solid | Organic | +1 | 0 | +1 | 0 | +3 | +1 | 0 | 0 | Nerve cluster |
| 53 | Tumite | Gel | Organic | +1 | -1 | -1 | 0 | +1 | +1 | -1 | 0 | Growth mass |
| 54 | Glowgut | Liquid | Radioactive | 0 | -1 | 0 | 0 | +1 | +2 | 0 | 0 | Irradiated bile |
| 55 | Corite | Solid | Radioactive | +1 | -2 | +1 | -1 | +1 | +2 | 0 | 0 | Reactor heart |
| 56 | Decayite | Particle | Radioactive | -1 | -2 | 0 | -1 | 0 | +2 | -1 | 0 | Radioactive rot |

---

### Glitch Zone

Primary: Optical | Secondary: ???

| # | Name | State | Reactivity | En | Mo | Fo | So | Se | Pe | Eg | Li | Notes |
|---|------|-------|------------|----|----|----|----|----|----|----|----|-------|
| 57 | Glitchslop | Particle | Stable | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | Flickering garbage slop |
| 58 | Prismite | Solid | Optical | 0 | +2 | 0 | 0 | +1 | +2 | 0 | 0 | Light bender |
| 59 | Spectrumite | Liquid | Optical | +1 | +2 | -1 | +1 | +1 | +3 | 0 | 0 | Liquid rainbow |
| 60 | Mirrorite | Solid | Optical | 0 | +1 | +1 | 0 | 0 | +2 | +1 | 0 | Perfect reflection |
| 61 | Fractalite | Solid | Optical | 0 | +1 | -1 | 0 | +1 | +3 | 0 | 0 | Infinite recursion |
| 62 | Phaseit | Particle | Stable | 0 | 0 | -1 | 0 | 0 | +3 | 0 | 0 | Phase-shifted dust |
| 63 | Nullite | Solid | Stable | -1 | -1 | 0 | -1 | -1 | +1 | -1 | -1 | Absence of something |
| 64 | Infinitite | Solid | Optical | +1 | +2 | +1 | +1 | +1 | +2 | +2 | +1 | Endgame peak |

---

## Part 7: Spawn Weights

Materials spawn based on biome weights. No hard categories — just tuning.

Example for Asteroid Belt:

| Material | Asteroid | Volcanic | Sunken | Spore | Acid | Frozen | Carrion | Glitch |
|----------|----------|----------|--------|-------|------|--------|---------|--------|
| Grite | 70% | 5% | 5% | 0% | 0% | 10% | 0% | 5% |
| Ironite | 50% | 10% | 20% | 0% | 0% | 20% | 0% | 5% |
| Boomite | 20% | 30% | 5% | 0% | 0% | 5% | 0% | 10% |

Full spawn table TBD during level design.

---

## Part 9: Customer Prescriptions

Side quests as customer requests for specific Effect Profiles.

| Customer Type | Request | Example Materials |
|---------------|---------|-------------------|
| Party Host | +Social, +Energy | Pollenite, Charmite |
| Student | +Focus | Voltite, Obsidianite |
| Artist | +Perception, +Mood | Prismite, Sporeite |
| Athlete | +Energy, +Ego | Magmite, Burstite |
| Insomniac | -Energy, +Mood | Cryite, Glacite |
| Mourner | -Sensation | Cryogel, Numbsalt |
| CEO | +Ego, +Focus | Cellite, Mirrorite |
| Escapist | +Perception, -Sensation | Fractalite, Glowgut |

---

## Part 10: Game Systems

Materials interact with these underlying systems. Each system tracks something and materials respond based on their State and Reactivity.

### System Overview

| System | What It Tracks | Materials Interact Via |
|--------|----------------|------------------------|
| **Heat** | Temperature per cell/region | Flammable ignites, Explosive detonates, solids melt, liquids evaporate |
| **Flow Field** | Liquid/gas movement direction | Liquids pool and flow, Gas rises and disperses, currents carry Particles |
| **Rigid Body** | Solid chunk physics | Falling debris, shattered solids, structural collapse |
| **Pressure** | Contained gas/liquid force | Sealed areas, explosive decompression, geysers |
| **Energy Grid** | Power routing through Conductive | Chains activate, machines power on, overload damage |
| **Light Field** | Beam paths and shadows | Optical redirects, laser puzzles, darkness zones |
| **Damage Field** | AOE zones (radius + intensity) | Radioactive DOT, explosion radius, acid pools |
| **Spawn System** | NPC generation from materials | Organic pile thresholds → creature emerges |

### Heat System

**Implementation:** Temperature is a separate scalar field (0-255, 8-bit bitmap) passed to GPU alongside gravity (RGB vec3).

| Property | Behavior |
|----------|----------|
| Source | Weapons, fire, environmental (lava, vents), burning materials |
| Propagation | Spreads via conduction (neighbors) and convection (rises in density > 0) |
| Accumulation | Materials store heat based on State-derived heat capacity |
| Thresholds | Per-material: igniteTemp triggers burning (Flammable reactivity), detonateTemp triggers explosion (Explosive reactivity) |
| Dissipation | Bleeds to neighbors based on conductivity, escapes upward in air |

**Temperature Field:**

| Property | Value |
|----------|-------|
| Range | 0-255 |
| Format | 8-bit scalar bitmap |
| Update | Per frame, parallel with gravity field |
| Vacuum | No transmission (density = 0) |

**Heat Capacity (derived from State):**

| State | Heat Capacity | Heating Speed | Why |
|-------|---------------|---------------|-----|
| Gas | 0.2 | Very fast | Low mass, high surface area |
| Particle | 0.4 | Fast | Surface area, but piled |
| Liquid | 0.6 | Medium | Flows, convects |
| Solid | 1.0 | Slow | Dense, stable chunks |

Lower capacity = heats faster. Temperature change = `heat_input / capacity`.

**Heat Conductivity (per-material property):**

| Example Material | Conductivity | Behavior |
|------------------|--------------|----------|
| Metal ore | 0.9 | Spreads heat quickly |
|iteite | 0.5 | Normal heat spread |
|ite ash | 0.2 | Insulates, traps heat |
|Ite | 0.05 | Barely conducts |

Heat spread = `temp_diff * conductivity * neighbor_conductivity`.

**Convection:**

- Reads gravity field direction
- Hot cells (temp > threshold) + air density > 0 → rises (opposite gravity)
- Creates updrafts, heat chimneys
- Sealed pockets accumulate heat (no escape path)

### Flow Field System

| Property | Behavior |
|----------|----------|
| Liquids | Flow downward, pool in depressions, seek lowest point |
| Gas | Rises, disperses outward, fills containers from top |
| Particles | Fall, pile, carried by liquid/gas currents |
| Currents | Environmental flow (wind, water current) pushes materials |
| Mixing | Liquids can mix (Corrosive + Water = diluted acid?) |

### Rigid Body System

| Property | Behavior |
|----------|----------|
| Chunks | Solids break into physics objects when shattered |
| Falling | Unsupported chunks fall, deal impact damage |
| Collision | Chunks trigger Explosive on impact, crush entities |
| Settling | Chunks come to rest, become static terrain again? |

### Pressure System

| Property | Behavior |
|----------|----------|
| Containment | Sealed areas hold Gas/Liquid under pressure |
| Breach | Breaking seal → explosive decompression, material rush |
| Compression | More material in sealed space → higher pressure |
| Release | Pressure differential drives flow direction |

### Energy Grid System

| Property | Behavior |
|----------|----------|
| Source | Power cells, generators, player weapons |
| Routing | Follows connected Conductive materials |
| Branching | Splits at junctions, power divides |
| Load | Machines require power threshold to activate |
| Overload | Excess energy → heat buildup → damage/explosion |

### Light Field System

| Property | Behavior |
|----------|----------|
| Source | Lasers, player weapons, environmental (sun, glow) |
| Propagation | Straight lines until blocked or redirected |
| Optical interaction | Reflect, refract, focus, diffuse |
| Shadow | Non-Optical materials block light |
| Damage | Concentrated beams deal heat damage |

### Damage Field System

| Property | Behavior |
|----------|----------|
| Source | Radioactive materials, explosions, acid pools |
| Shape | Radius from source, can be sphere or irregular |
| Intensity | Damage per tick, falls off with distance? |
| Duration | Persistent (Radioactive) vs momentary (Explosion) |
| Stacking | Multiple fields overlap → increased damage |

### Spawn System

| Property | Behavior |
|----------|----------|
| Trigger | Organic material pile reaches threshold |
| Threshold | Size-based (50/100/250 particles → different NPCs) |
| Check | End of frame, scan for qualifying piles |
| Spawn | Consumes material, creates NPC at pile location |
| Types | TBD — Twitcher, Crawler, Amalgam per Carrion Depths |

---

## Part 11: System × Reactivity Matrix

Which systems care about which Reactivities:

| Reactivity | Heat | Flow | Rigid | Pressure | Energy | Light | Damage | Spawn |
|------------|:----:|:----:|:-----:|:--------:|:------:|:-----:|:------:|:-----:|
| Stable | · | · | ✓ | · | · | · | · | · |
| Flammable | ✓ | · | · | · | · | · | ✓ | · |
| Explosive | ✓ | · | ✓ | ✓ | · | · | ✓ | · |
| Corrosive | · | ✓ | · | · | · | · | ✓ | · |
| Radioactive | · | · | · | · | · | · | ✓ | · |
| Conductive | ✓ | · | · | · | ✓ | · | · | · |
| Optical | · | · | · | · | · | ✓ | · | · |
| Organic | · | · | · | · | · | · | · | ✓ |

Key: ✓ = primary interaction, · = minimal/none

---

## Open Questions

### Materials
1. **Organic behavior** — Pending engine R&D for procedural generation
2. **Glitch Zone Secondary** — TBD, maybe "all" or remixes other biomes
3. **State variants** — Water/Ice/Steam families parked for later
4. **Optical earlier?** — Save for Glitch Zone or tease in mid-game?
5. **Spawn weight tuning** — During level design phase

### Systems
6. **Heat thresholds** — What actual numbers? Need playtesting
7. **Radioactive contamination** — Spreads to neighbors or contained?
8. **Radioactive decay** — Permanent or fades over time?
9. **Pressure scope** — Full simulation or scripted set pieces?
10. **Conductive overload** — Heat buildup → melt? Explode? Spark damage?
11. **Liquid mixing** — Do liquids combine? (Acid + Water = diluted?)
12. **Rigid body settling** — Fallen chunks become terrain again or stay physics objects?
13. **Plasma** — 5th State? Extreme heat result? Parked for now

---

## Parked Concepts

| Concept | Status | Notes |
|---------|--------|-------|
| Origin (Synthetic/Mineral/Organic/Mutant) | Lore/visual only | Not a material property |
| Material Tiers | Cut | Complexity handled by biome progression |
| Universal materials | Cut | All materials are biome-weighted |
| State variant families | Parked | Good idea, not v0.3 scope |

---

## References

- Game_Overview.md — Core loop
- Modular_Weapon_System_Specification.md — Weapon interactions
- Visual_Style_Specification.md — Biome aesthetics (TBD)
