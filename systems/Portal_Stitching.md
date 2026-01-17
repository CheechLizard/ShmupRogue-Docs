# Portal Stitching System Specification v0.1

**Location:** `/systems/`

---

## Overview

Portals seamlessly connect arena and corridor spaces. No loading screens, no interaction prompts. Players fly through a spherical bubble and find themselves in a corridor. Direction of entry determines corridor orientation and boss.

---

## Core Concept

```
ARENA SPACE                    CORRIDOR SPACE
                              
         [Player flies →]  
              →→→→→→→→→→→→→→→→→→→→ [Now in corridor]
                              
         [Player flies ←]   
              ←←←←←←←←←←←←←←←←←←←← [Back in arena]
```

The portal is a **spatial threshold**, not an object to interact with.

---

## Portal Bubble

The portal is a world entity, not just a system mechanic. It has physical presence, visual design, and can interact with other game systems.

### Entity Properties

| Property | Type | Notes |
|----------|------|-------|
| Position | World coords | Placed by level generation |
| Radius | Float | Larger than screen viewport |
| Terrain | Optional | Could have structure around it |
| Sprites | Yes | Visual design, direction indicators |
| Attractor | Optional | Gravity pull toward center? |
| Shader | Yes | Transition effect, direction feedback |

### Direction Locking

Directions are locked until material requirements are met.

| Direction | Requirement Example | Visual |
|-----------|---------------------|--------|
| UP | 50 Uranium | Locked gate, Uranium icon, progress bar |
| RIGHT | 100 Ironite | Locked gate, Ironite icon, progress bar |
| DOWN | Unlocked | Open, no gate |
| etc. | ... | ... |

**Behavior:**
- Cannot enter from locked angle
- Visual gate/barrier at that segment
- Shows material icon + amount needed
- Progress bar fills as you collect
- Unlocks when requirement met

**Level design implications:**
- Control which bosses are accessible early
- Force players to harvest specific planets
- Create natural progression through directions
- Some directions could start unlocked (easy bosses first?)

### Visual Design Requirements

The portal must communicate:

| Information | How |
|-------------|-----|
| "This is the exit" | Distinct visual, HUD waypoint |
| Which direction you're approaching | Segment highlighting as you orbit |
| Which directions are locked | Gates, barriers, icons per segment |
| What unlocks locked directions | Material/boss/tier icons |
| Which bosses already defeated | Cleared indicator per direction |

### Physical Structure Options

| Option | Description |
|--------|-------------|
| Pure shader | Bubble is just an effect, no collision |
| Ring structure | Physical terrain ring around threshold |
| Gate segments | 8 physical gates, some locked |
| Hybrid | Shader bubble inside physical frame |

---

## Directional Entry

Entry angle determines corridor properties:

```
                    UP (Boss A)
                       ↑
          DIAG-UL ↖    ↑    ↗ DIAG-UR
                ╲      ↑      ╱
           ┌───────────────────────┐
           │                       │
LEFT ←←←←←←│    PORTAL BUBBLE      │→→→→→→ RIGHT
(Boss D)   │                       │       (Boss B)
           │                       │
           └───────────────────────┘
                ╱      ↓      ╲
          DIAG-DL ↙    ↓    ↘ DIAG-DR
                       ↓
                    DOWN (Boss C)
```

**8 directions = 8 corridors = 8 bosses = 8 fragments**

---

## Direction Detection

| Direction | Angle Range | Corridor Scroll |
|-----------|-------------|-----------------|
| Right | -22.5° to 22.5° | Horizontal → |
| Up-Right | 22.5° to 67.5° | Diagonal ↗ |
| Up | 67.5° to 112.5° | Vertical ↑ |
| Up-Left | 112.5° to 157.5° | Diagonal ↖ |
| Left | 157.5° to 202.5° | Horizontal ← |
| Down-Left | 202.5° to 247.5° | Diagonal ↙ |
| Down | 247.5° to 292.5° | Vertical ↓ |
| Down-Right | 292.5° to 337.5° | Diagonal ↘ |

Player velocity at threshold crossing determines direction.

---

## Chunk Stitching

The portal connects two chunk systems seamlessly.

```
ARENA CHUNKS          PORTAL          CORRIDOR CHUNKS
                       ZONE
┌───┬───┬───┐      ┌────────┐      ┌───┬───┬───┬───┐
│   │   │   │      │        │      │   │   │   │   │
├───┼───┼───┤  ←→  │ BUBBLE │  ←→  ├───┼───┼───┼───┤
│   │   │   │      │        │      │   │   │   │   │
└───┴───┴───┘      └────────┘      └───┴───┴───┴───┘
```

**Requirements:**
- Bubble larger than screen viewport
- Corridor chunks pre-loaded while in bubble
- Seamless camera transition
- No visible seam or pop-in

---

## Bidirectional Travel

Players can exit corridor at any time by flying backwards.

| Action | Result |
|--------|--------|
| Fly through portal → | Enter corridor |
| Fly through portal ← | Return to arena |
| Reach corridor end | Boss encounter |
| Defeat boss | Proceed to shop |
| Abort mid-corridor | Return to arena |
| Abort after engaging boss | Boss may escape |

---

## Escaped Boss Mechanic

If player aborts after engaging a boss:

| Scenario | Probability | Consequence |
|----------|-------------|-------------|
| Abort before boss | 0% | Safe exit |
| Abort during boss fight | High | Boss follows to arena |
| Escaped boss in arena | — | Haunts system until killed |

**Escaped boss behavior:**
- Pursues player in arena
- Does not despawn
- Can be killed in arena (harder than corridor)
- Cannot be lured back into corridor (?)

---

## Visual Design

| Element | Treatment |
|---------|-----------|
| Bubble exterior | Shimmer/distortion shader |
| Bubble interior | Transitional space, hyperspace feel |
| Corridor entrance | Bleeds through faintly |
| Direction indicator | Subtle visual cue for chosen direction? |

---

## Technical Requirements

| Requirement | Notes |
|-------------|-------|
| Pre-load corridor | While player approaches bubble |
| Determine direction | Sample velocity at threshold |
| Camera handoff | Smooth transition, no jump |
| Physics continuity | Momentum carries through |
| Entity handling | What follows through? (see Escaped Boss) |

---

## Parameters

| Parameter | Default | Range | Notes |
|-----------|---------|-------|-------|
| `bubble_radius` | TBD | 2-5 screens | Must hide loading |
| `direction_sample_distance` | TBD | 0.5-2 screens | How far before threshold to sample |
| `escape_probability` | TBD | 0.5-1.0 | Chance boss follows on abort |

---

## Open Questions

1. **Bubble visual** — What shader effect? Distortion, color shift, particles?
2. **Direction feedback** — Does player see which direction they're choosing?
3. **Velocity threshold** — Minimum speed to trigger crossing?
4. **Multi-crossing** — What if player oscillates in bubble?
5. **Boss escape rules** — Always follow, or probability? Can be lured back?
6. **Physical structure** — Pure shader, ring, gates, or hybrid?
7. **Attractor field** — Should portal pull player/particles toward it?
8. **Starting unlocks** — How many directions unlocked at start? Which ones?
9. **Material amounts** — How much per direction? Escalating or flat?
10. **Defeated boss indicator** — How to show which directions are "cleared"?

---

## Related Specifications

- `/levels/solar_system/Corridors.md` — Corridor content
- `/entities/npcs/Corridor_Bosses.md` — Boss definitions
- `/systems/Celestial_Upgrades.md` — Fragment rewards

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2025-01-15 | Initial specification, extracted from Solar System Arena |
