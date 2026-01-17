# Solar System Overview v0.1

**Location:** `/levels/solar_system/Solar_System_Overview.md`

---

## Vision

A massive solar system arena where players harvest planets, fight endless waves, and ultimately destroy the sun. Combines Defender's intensity with modern horde survivor density (Brotato, Stand Survivors).

---

## Feel

- **Constant pressure** — Always under attack, can't hide
- **Strategic navigation** — Plan routes through danger zones
- **Layered mastery** — Dig smarter, not just deeper
- **Escalating power** — From scrappy survival to screen-filling chaos

---

## Win Condition

```
DEFEAT 8 CORRIDOR BOSSES
    ↓
COLLECT 8 SUN BUSTER FRAGMENTS
    ↓
DESTROY THE SUN
    ↓
FINAL BOSS EMERGES
    ↓
DEFEAT FINAL BOSS
    ↓
ESCAPE SYSTEM COLLAPSE
```

---

## Core Loop

```
ENTER SOLAR SYSTEM (persistent seed)
    ↓
CONSTANT COMBAT (kill NPCs → fuel)
    ↓
HARVEST PLANET(S)
    ↓
FILL HARVEST BAR → PORTAL ACTIVATES
    ↓
CHOOSE PORTAL DIRECTION (8 options)
    ↓
CORRIDOR GAUNTLET → BOSS → FRAGMENT
    ↓
SHOP → RETURN TO ARENA
    ↓
REPEAT (8 bosses for Sun Buster)
    ↓
DESTROY SUN → ESCAPE
```

---

## Key Mechanics

| Mechanic | Purpose |
|----------|---------|
| Fuel (from NPC kills) | Forces constant combat |
| Directional portal | Player chooses boss/corridor |
| Escaped bosses | Consequence for aborting corridor |
| Celestial upgrades | Zelda-style tool transformation |
| Layered planets | "Dig smarter" progression |

---

## Content Summary

| Element | Count | Notes |
|---------|-------|-------|
| Planets | 5-7 per system | Variable archetypes |
| Corridor bosses | 8 | One per direction |
| Celestial upgrades | 8 | Transform base tools |
| Planet archetypes | 6 | Rocky, Gas Giant, Ice, Molten, Dead, Water |

---

## References

| Game | What We're Taking |
|------|-------------------|
| Defender | Speed, intensity, screen-wrap |
| Brotato / Stand Survivors | Horde density, constant pressure |
| Motherload | Depth = harder materials |
| Zelda | Sequential item upgrades |
| Metroid | Return with new tools for secrets |

---

## Related Specifications

**Content (what):**
- `/levels/solar_system/Arena_Generation.md` (TODO)
- `/levels/solar_system/Planets.md` (TODO)
- `/levels/solar_system/Sol_Chunk.md` (TODO)
- `/levels/solar_system/Corridors.md` (TODO)

**Systems (how):**
- `/systems/Combat_Pressure.md`
- `/systems/Fuel.md`
- `/systems/Portal_Stitching.md`
- `/systems/Celestial_Upgrades.md`

**Entities:**
- `/entities/npcs/Solar_Dragon.md` (TODO)
- `/entities/npcs/Corridor_Bosses.md` (TODO)

---

## Open Questions

See individual specs for detailed open questions.

High-level:
1. Final boss design — What emerges from the sun?
2. Escape sequence — How long? What's the threat?
3. Level 2 (Underwater) — How different is the structure?

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2025-01-15 | Extracted from Solar_System_Arena_Specification_v0_3, moved to /levels/solar_system/ |
