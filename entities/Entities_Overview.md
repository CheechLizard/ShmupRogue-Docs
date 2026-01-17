# Entities Overview

Entity archetypes define component bundles. The game uses a **hybrid architecture**: pure ECS for high-volume entities (particles, projectiles, enemies) and domain objects for low-volume, complex state (player, world, UI).

---

## Entity Categories

| Category | Path | Notes |
|----------|------|-------|
| Player/ | Cold | Domain object wrapping ECS entity. Complex state, input handling, weapon switching. |
| NPCs/Enemies/ | Hot | Pure ECS. Spawned in bulk, identical component sets to player for entity symmetry. |
| NPCs/Harvesters/ | Hot | Pure ECS. NPC harvester mechanic entities. |
| NPCs/Merchants/ | Cold | Domain object. Stateful, inventory management. |
| Items/Pickups/ | Hot | Pure ECS. High volume, simple behavior. |
| Items/Materials/ | Hot | Pure ECS. Five states: solid, particle, liquid, gas, plasma. |
| Items/Artifacts/ | Cold | Domain object. Black box recipe system, complex discovery logic. |
| Projectiles/ | Hot | Pure ECS. Thousands per frame, cache-critical. |
| Structures/ | Hot | Pure ECS. Destructible terrain elements, debris. |
| Interactables/ | Cold | Domain object. Portals, stores, stateful interactions. |

**Hot path**: Pure ECS, optimized for cache-friendly iteration of physics-heavy entities.

**Cold path**: Domain wrappers that hold entity IDs, used for complex state management.

---

## Entity Symmetry

NPCs and players follow identical rules via shared component types. This enables emergent behaviors—anything that affects entities affects both players and NPCs equally.

---

## Related Specifications

- `/entities/Drone_Systems_Specification.md` — Drone types and behaviors
- `/entities/Modular_Weapon_System_Specification.md` — Weapon architecture
- `/entities/npcs/NPC_Behavior_Specification.md` — NPC AI and behaviors
- `/entities/Black_Holes_Specification.md` — Black hole entity mechanics

---

## Gaps (TODO)

- Player_Ship specification
- Items specification
- Structures specification

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2025-01-17 | Extracted from Project_Taxonomy_Specification_v0.1 §5.2. |
