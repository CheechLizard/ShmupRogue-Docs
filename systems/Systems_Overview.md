# Systems Overview

Game systems handle cross-cutting concerns that span multiple entity types. This document provides an index of all system specifications.

---

## Mission System

Handles linear objective sequences per stage. Missions are simple chains; no branching or prerequisites.

| Module | Responsibility |
|--------|----------------|
| ObjectiveTracker/ | Tracks current objective index, completion state, progress counters. |
| TriggerEval/ | Evaluates completion conditions: destroy count, collect amount, reach location, etc. |
| HintDisplay/ | Shows current objective text, progress indicators, completion feedback. |

---

## Core Game Systems

| Module | Responsibility |
|--------|----------------|
| Combat/ | Damage resolution, hit detection coordination, death handling. |
| Progression/ | Tech tier unlocks, permanent upgrades, meta-progression. |
| Spawning/ | Wave management, spawn patterns, difficulty scaling. |
| AI/ | Behavior trees, steering behaviors, threat assessment. |

---

## Technology Tiers

Five technology tiers gate weapon capabilities, visual effects, and progression. Each tier has distinct aesthetics and mechanical identity:

| Tier | Aesthetics | Examples |
|------|------------|----------|
| **Steam** | Smoke, sparks, pistons, pressure bursts | Industrial, mechanical feel |
| **Electronic** | Arcs, glows, static, pulse waves | Crackling energy, circuit aesthetics |
| **Photonic** | Lasers, prisms, beams, light diffraction | Clean, precise, brilliant |
| **Organic** | Acid, spores, tendrils, biomass | Living, corrupted, Giger-esque |
| **Quantum** | Black holes, distortion, phasing, probability clouds | Reality-bending |

---

## Related Specifications

- `/systems/Combat_Pressure.md` — Combat pressure mechanics
- `/systems/Fuel.md` — Fuel system
- `/systems/Portal_Stitching.md` — Portal construction and travel
- `/systems/Celestial_Upgrades.md` — Zelda-style tool upgrades
- `/systems/Resource_Economy_Specification.md` — Material flow and harvesting

---

## Gaps (TODO)

- Harvesting system specification
- Progression system specification
- AI system specification

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2025-01-17 | Extracted from Project_Taxonomy_Specification_v0.1 §5.6-5.7, §1.2. |
