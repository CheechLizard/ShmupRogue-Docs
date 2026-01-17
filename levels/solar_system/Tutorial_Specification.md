# Tutorial Specification v0.1

## Overview

Level 1 tutorial structure. Each objective introduces a mechanic through narrative.

---

## Level 1: First Contact

### Setting

Edge of the solar system. Asteroid field. Empty space. You're on routine patrol, dragging minerals, when the artifact appears.

### Beat Structure

#### Opening

You're dragging ice back to the depot. Routine haul. Then something pings on your sensors - strange gravitational signatures, way off the shipping lanes. Could be nothing. Could be salvage. A marker appears on your nav.

*Player learns: Movement, navigation, following objectives.*

---

#### Objective 1: Investigate the Artifact

**Objective:** Reach the alien object.

**Gameplay:** Fly to the marker. No combat yet. Establish controls and movement.

**Visual Progression:** Player starts in empty space (just starfield). As they travel toward the marker, asteroids and terrain begin appearing ahead - they're entering the asteroid belt where the artifact waits. This reinforces the "way off the shipping lanes" narrative.

**Story Beat:** As you approach, the object resolves into something clearly alien. Geometric. Wrong. Your sensors can't fully parse it.

---

#### Objective 2: Scan the Artifact

**Objective:** Get close and scan.

**Gameplay:** Approach and hold position near the artifact. A tether connects your ship to the artifact during the scan.

**Story Beat:** Your sensors confirm what you hoped: alien technology. A matter transformer of some kind. This is the find of a lifetime.

---

#### Objective 3: Possession

**Trigger:** Scan completes.

**Gameplay:** This is not interactive. The player watches.

**Sequence:**
1. Artifact speaks: **"I awake. My power returns."**
2. Angular lightning flows along the tether toward the player's ship
3. Ship explodes. Screen goes black.
4. Player respawns at the artifact.
5. Ship is now artifact-colored (visual proof of corruption).
6. Minimap appears in top-right corner (a power bestowed by the artifact).

**Story Beat:** The artifact has consumed you. You are its creature now. Death is no longer an escape - it will always pull you back.

*Player learns: Respawn mechanic. You are bound to this thing.*

---

#### Objective 4: Retrieve the Harvester

**Trigger:** After respawn.

**Sequence:**
1. Artifact speaks: **"You have been consumed, human. Retrieve what they stole from me..."**
2. Red dot appears on minimap.

**Gameplay:** Follow the minimap marker to the mining area.

**Visual:** Player finds miners - bombers dropping bombs on asteroids, collection drone gathering debris. (For tutorial, miners are static/scripted. Full AI to be developed later.)

---

#### Objective 5: Combat

**Trigger:** Player attacks miners.

**Gameplay:** Flyer squadron spawns to defend the miners. First combat encounter. Basic enemies, basic weapons. Learn to shoot.

**Story Beat:** The probes are small, fast, mechanical. They don't communicate. They just attack. When destroyed, they leave debris and particles.

*Player learns: Combat basics.*

---

#### Objective 6: Acquire Harvester

**Trigger:** All enemies destroyed.

**Gameplay:** Harvester power-up drops from destroyed miners. Player picks it up.

**On Pickup:**
1. Ram scoop drone attaches to front of player's ship
2. Material bar appears in HUD

**Story Beat:** The harvester is yours. Whatever the artifact needs, you can now gather it.

*Player learns: Power-ups exist. Material collection is possible.*

---

#### Objective 7: Harvest Materials

**Objective:** Fill the material bar.

**Gameplay:**
- Destroy asteroids to create debris
- Fly through debris - ram scoop drone collects automatically
- Material bar fills as resources accumulate

**Story Beat:** The harvester works. You're gathering alien matter. For what purpose, you don't know yet.

*Player learns: The matter economy loop. Destroy → debris → harvest.*

---

#### Objective 8: Return to Artifact

**Trigger:** Material bar full.

**Gameplay:** Minimap shows artifact location. Return to deliver materials.

**On Arrival:**
1. Materials are consumed by the artifact
2. Gate construction begins
3. Gate progress bar appears (30 seconds)

**Story Beat:** The artifact begins to transform. Something is being built. A gate. A way out - or a way in.

---

#### Objective 9: Survive

**Trigger:** Gate construction begins.

**Gameplay:**
- Endless waves of enemies attack
- Survive 30 seconds until gate completes
- Gate opens when timer reaches zero

**Victory Condition:** Gate completes. Portal opens. Level 1 complete.

**Failure Condition:** You die. But you respawn at the artifact - the loop continues. Try again.

**Story Beat:** They're coming for you. For the artifact. Hold them off. Just a little longer.

*Player learns: Survival pressure. The stakes.*

---

## Artifact Dialogue

| Moment | Line |
|--------|------|
| Awakening | "I awake. My power returns." |
| After respawn | "You have been consumed, human. Retrieve what they stole from me..." |

---

## UI Elements Introduced

| Element | When | Location |
|---------|------|----------|
| Minimap | After possession | Top-right corner |
| Material bar | After acquiring harvester | HUD |
| Gate progress bar | During survival wave | HUD |

---

## Visual Changes

| Change | When |
|--------|------|
| Ship color changes to artifact palette | After possession/respawn |
| Tether lightning effect | During possession sequence |
| Screen blackout | Ship explosion during possession |

---

## Resolved Design Questions

1. **Harvester Drone Behavior:** Ram scoop style - sits in front of player's craft, collects debris automatically as player flies through.

2. **Miner Behavior:** Static/scripted for tutorial. Full AI to be specced separately.

3. **Minimap:** Actual spatial minimap in top-right. Framed as a power the artifact bestows. Only appears when needed.

4. **Ship Colors:** Starts neutral gray/blue, becomes artifact-colored (purple/magenta alien palette) after possession.

5. **Respawn Point:** Always at the artifact.
