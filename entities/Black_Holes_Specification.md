# Black Holes Specification v0.1

## Overview

Black holes are created by singularity devices or extreme quantum effects. They act as material sinks during their lifecycle and emit exotic material on collapse.

---

## Structure

```
┌─────────────────────────────────────┐
│                                     │
│     GRAVITY FIELD (fixed radius)    │
│     - Flow vectors point inward     │
│     - Materials spiral toward center│
│                                     │
│         ┌───────────────┐           │
│         │               │           │
│         │  SINGULARITY  │           │
│         │  (variable r) │           │
│         │               │           │
│         └───────────────┘           │
│                                     │
└─────────────────────────────────────┘
```

| Component | Behavior |
|-----------|----------|
| Singularity (center) | Black circle, variable radius. Destroys materials on contact ("spaghettification"). Grows when fed. Visual: black circle with shader FX at perimeter. |
| Gravity Field (outer) | Fixed radius around singularity. Flow vectors point inward. Materials spiral toward center. |
| Total Size | Singularity radius (chunk-rounded) + gravity field radius (fixed) |

---

## Lifecycle

| Phase | Behavior |
|-------|----------|
| Creation | Singularity device detonates, initial radius established |
| Active | Materials pulled inward by gravity field, destroyed on contact with singularity, singularity radius grows |
| Decay | Singularity shrinks over time, minimum lifespan timer prevents instant collapse |
| Collapse | When singularity radius → 0, emits exotic material, disappears |

---

## Parameters

| Parameter | Description | Tuning Notes |
|-----------|-------------|--------------|
| Initial singularity radius | Starting size on creation | Per-device type |
| Gravity field radius | Fixed outer pull zone | Constant, affects danger zone |
| Growth rate | How much singularity grows per material consumed | Balance harvesting vs danger |
| Decay rate | How fast singularity shrinks when starved | Affects lifespan |
| Minimum lifespan | Timer before collapse can occur | Prevents instant pops |
| Exotic output ratio | Materials consumed → exotic output on collapse | Reward scaling |

---

## Gameplay Implications

| Behavior | Player Interaction |
|----------|-------------------|
| Feed it | Grows, persists longer, stronger pull |
| Starve it | Shrinks, dies faster |
| Larger size | More dangerous but better gravitational harvesting |
| Collapse reward | Exotic material output proportional to materials consumed |

**Risk/Reward:**
- Small black hole: Safe, low yield
- Large black hole: Dangerous, high exotic output
- Feeding enemy materials into black hole: Denial + future reward

---

## Visual Design

| Element | Treatment |
|---------|-----------|
| Singularity | Pure black circle |
| Perimeter | Shader FX (distortion, glow, accretion disc?) |
| Gravity field | Subtle visual indicator? Or invisible (learned through play) |
| Spaghettification | Stretch FX on materials entering singularity |
| Collapse | Burst FX, exotic material spawn |

---

## Open Questions

1. **Gravity field visibility:** Should players see the pull zone, or learn it through experience?

2. **Entity interaction:** Can black holes pull/destroy ships and NPCs, or only materials?

3. **Multiple black holes:** Can they merge? Interact?

4. **Maximum size:** Is there a cap on singularity growth?

5. **Exotic output formula:** Linear with consumption, or diminishing returns?

---

## References

- Equipment Economy Specification v0.1 — Parent system
- Quantum/Gravitational Devices — Creation mechanisms
