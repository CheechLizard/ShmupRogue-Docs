# Hierarchical Stability Texturing v0.3

## Problem

Sand physics creates blobby, uniform terrain which is difficult to texture in an attractive way using traditional pixel art techniques.

## Core Insight

Inactive terrain doesn't need per-pixel simulation. We can detect stable regions and apply progressively richer visual treatment as regions remain stable over time.

**Goal:** Create relief — edge highlights, lowlights, and interior details on dynamic, arbitrary shapes that change through destruction. This reinforces the visual qualities of materials and lighting in the scene.

**The progression:**
1. Flat color (active/small)
2. Edge lighting appears (stability begins)
3. Interior detail emerges (large stable regions)

### Dual Purpose

This system serves both **runtime stabilization** and **procedural world generation**:

- **World generation:** Pre-generated terrain has full detail already applied. Players encounter hardened, textured environments from the start. World gen uses instant hardening.
- **Runtime:** When terrain is destroyed and settles, hardening times determine how quickly detail returns. Players notice hardening as they return to areas they previously destroyed — a level design mechanism.

### Non-Goals

- Dynamic per-pixel lighting
- Real-time shadows
- Liquid/gas interior detail (they get base color + light tint only)
- Sub-pixel anti-aliasing

---

## Quadtree Stability System

Each chunk maintains its own quadtree per material present. Cross-material boundaries are naturally handled because different materials can't harden together.

| Level | Block Size | Stability Time | Visual Treatment |
|-------|------------|----------------|------------------|
| 0 | 2×2 | 0 (active) | Flat material color |
| 1 | 4×4 | ~1s | Edge lighting |
| 2 | 8×8 | ~2s | Edge + interior detail |
| 3 | 16×16 | ~4s | Full interior detail |
| 4 | 32×32 | ~8s | Full detail + variation |
| 5 | 64×64 | ~16s | Full detail + variation |

### Spatial Organization

```
Each chunk maintains one quadtree per material present in that chunk.
- Quadtree root = chunk bounds (256×256 wu)
- Max hardening level within one chunk = level 5 (64×64)
- Cross-chunk regions cannot form larger blocks (acceptable limitation)
```

This keeps memory bounded and integrates cleanly with chunk lifecycle.

### Chunk Boundary Constraint

Blocks cannot span chunk boundaries. A 64×64 stable region that crosses a chunk edge appears as two separate level-4 regions, not one level-5 region.

**Visual mitigation:** Edge lighting at chunk boundaries uses neighbor chunk data (already available via ghost halos) to avoid visible seams.

### Hardening Rule

A block at level N can harden to level N+1 when:
1. All four child blocks are at level N
2. All four child blocks contain the same material
3. All four child blocks have been stable for the required duration (modified by material's `hardening_rate`)

### Crumbling Rule

A cell is "active" if it moved to a different cell position this frame. Velocity jitter within the same cell does not reset stability.

**Granularity:** Stability is tracked per 2×2 block (level 0), not per-pixel. If any cell in a 2×2 block moves, that block resets to active.

When any block becomes active:
1. Immediately crumble that block to level 0
2. Cascade crumbling up the hierarchy (parent blocks lose their hardened status)

---

## Visual Example: Irregular Region Over Time

### TIME 0 — Just Settled (All Level 0)

All cells are 2×2, rendered as flat material color.

```
        0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
      ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
    0 │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │
      ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
    1 │   │   │   │   │ ● │ ● │ ● │ ● │ ● │ ● │   │   │   │   │   │   │
      ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
    2 │   │   │   │ ● │ ● │ ● │ ● │ ● │ ● │ ● │ ● │   │   │   │   │   │
      ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
    3 │   │   │ ● │ ● │ ● │ ● │ ● │ ● │ ● │ ● │ ● │ ● │   │   │   │   │
      ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
    4 │   │   │ ● │ ● │ ● │ ● │ ● │ ● │ ● │ ● │ ● │ ● │   │   │   │   │
      ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
    5 │   │   │   │ ● │ ● │ ● │ ● │ ● │ ● │ ● │ ● │   │   │   │   │   │
      ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
    6 │   │   │   │   │   │ ● │ ● │ ● │ ● │   │   │   │   │   │   │   │
      ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
    7 │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │
      └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘

● = filled 2×2 cell (level 0, flat color)
```

### TIME 1s — Level 1 (4×4) Forming

4×4 blocks form where all four 2×2 children are filled and stable.

```
        0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
      ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
    0 │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │
      ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
    1 │   │   │   │   │ ● │ ● │ ● │ ● │ ● │ ● │   │   │   │   │   │   │
      ├───┼───┼───┼───┼───┴───┼───┴───┼───┴───┼───┼───┼───┼───┼───┼───┤
    2 │   │   │   │ ● │       │       │       │ ● │   │   │   │   │   │
      ├───┼───┼───┼───┤   ○   │   ○   │   ○   ├───┼───┼───┼───┼───┼───┤
    3 │   │   │ ● │ ● │       │       │       │ ● │ ● │   │   │   │   │
      ├───┼───┼───┼───┼───────┼───────┼───────┼───┼───┼───┼───┼───┼───┤
    4 │   │   │ ● │ ● │       │       │       │ ● │ ● │   │   │   │   │
      ├───┼───┼───┼───┤   ○   │   ○   │   ○   ├───┼───┼───┼───┼───┼───┤
    5 │   │   │   │ ● │       │       │       │ ● │   │   │   │   │   │
      ├───┼───┼───┼───┼───┬───┼───┬───┼───┬───┼───┼───┼───┼───┼───┼───┤
    6 │   │   │   │   │   │ ● │ ● │ ● │ ● │   │   │   │   │   │   │   │
      ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
    7 │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │
      └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘

○ = 4×4 block (level 1, simple noise pattern)
● = 2×2 cell (level 0, flat color) — boundary cells that can't merge
```

### TIME 2s — Level 2 (8×8) Forming

8×8 blocks form in the center where four 4×4 blocks are all stable.

```
        0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
      ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
    0 │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │
      ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
    1 │   │   │   │   │ ● │ ● │ ● │ ● │ ● │ ● │   │   │   │   │   │   │
      ├───┼───┼───┼───┼───┴───┼───┴───┴───┴───┼───┼───┼───┼───┼───┼───┤
    2 │   │   │   │ ● │       │               │ ● │   │   │   │   │   │
      ├───┼───┼───┼───┤   ○   │               ├───┼───┼───┼───┼───┼───┤
    3 │   │   │ ● │ ● │       │               │ ● │ ● │   │   │   │   │
      ├───┼───┼───┼───┼───────┤       ◇       ├───┼───┼───┼───┼───┼───┤
    4 │   │   │ ● │ ● │       │               │ ● │ ● │   │   │   │   │
      ├───┼───┼───┼───┤   ○   │               ├───┼───┼───┼───┼───┼───┤
    5 │   │   │   │ ● │       │               │ ● │   │   │   │   │   │
      ├───┼───┼───┼───┼───┬───┼───┬───┬───┬───┼───┼───┼───┼───┼───┼───┤
    6 │   │   │   │   │   │ ● │ ● │ ● │ ● │   │   │   │   │   │   │   │
      ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
    7 │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │
      └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘

◇ = 8×8 block (level 2, medium detail/small bricks)
○ = 4×4 block (level 1, simple noise)
● = 2×2 cell (level 0, flat color)
```

---

## Bump Maps for Relief

Both perimeter edges and interior detail are represented as bump maps (grayscale height data). The lighting system treats them identically — it just calculates normals from height and applies highlight/lowlight.

**Perimeter:** Where material meets empty = height drops to 0 (black). The quadtree tells us where these edges are.

**Interior detail:** Tileable bump map patterns (mortar, cracks, rivets, veins). Deterministic tile selection fills stable regions with appropriate tiles.

**Lighting:** Same rules apply to both. Calculate normals from height, shade based on light direction.

---

## Interior Detail via Tile Selection

Stable regions get filled with tileable bump map tiles using deterministic selection.

### Tile Library

Pre-baked grayscale bump map tiles at multiple sizes:

| Size | Weight | Use Case |
|------|--------|----------|
| 4×4 | High | Small fills, tight spaces |
| 8×8 | High | Common fill size |
| 16×16 | Medium | Larger stable areas |
| 32×32 | Low | Big expanses |
| 64×64 | Low | Very large stable areas |

**Tile set guidance:**
- 3-5 variations per size per pattern type
- Rotation/flip variants multiply variety cheaply (4 rotations × 2 flips = 8× from one tile)
- Seamless tiling matters more than quantity

Tiles are grayscale height data — white is high, black is low.

### Material Pattern Types

| Type | Visual Character | Example Materials |
|------|------------------|-------------------|
| Brick/Stone | Grid lines, rectangular cells | Stone, concrete, masonry |
| Metal Plate | Panel seams, rivets | Hull, machinery, stations |
| Organic | Curves, veins, asymmetric | Flesh, biomechanical |
| Crystal | Angular fractures | Ice, minerals |
| Rock | Irregular cracks | Cave walls, asteroids |

### Tile Selection Strategy

Tile selection is deterministic based on block position and material:

```
hash(block_position, material_type) → variant index
```

Each tile size (4×4, 8×8, 16×16, 32×32, 64×64) has variants for:
- Full interior (no perimeter edges)
- 1-edge variants (top, bottom, left, right)
- 2-edge variants (corners, opposing)
- 3-edge and 4-edge variants for small protrusions

**Seam hiding:** Tiles use overlapping edge blending. Edge pixels of adjacent tiles are blended to hide seams.

### Perimeter Constraint

Tiles touching the material edge must have height = 0 at that edge. This ensures interior detail doesn't compete with perimeter lighting:

```
QUAD WITH PERIMETER ON LEFT + TOP

0 0 0 0 0 0 0 0    ← top edge: all zeros (perimeter)
0 · · · · · · ·
0 · ■ ■ · ■ ■ ·    ← interior detail can have height
0 · ■ ■ · ■ ■ ·
0 · · · · · · ·
0 · ■ ■ ■ ■ ■ ·
0 · ■ ■ ■ ■ ■ ·
0 · · · · · · ·

0 = height 0 (perimeter, will get edge lighting)
· = low height (base)
■ = high height (interior detail bumps)

↑ left edge: all zeros (perimeter)
```

---

## Lighting

Both perimeter edges and interior detail bumps get the same lighting treatment.

### Global Light Settings

```
light_direction: vec2(-0.7, -0.7)   # normalized 2D vector (this example = top-left)
light_hue: 60°                      # highlight color shift (any hue 0-360°)
lowlight_hue: 240°                  # lowlight color shift (any hue 0-360°)
```

Configurable per biome/level. Vector allows any angle, not just cardinals.

### How It Works

1. **Calculate normals from bump map** (height differences between neighbors, stored as RG8)
2. **Dot product normal with light direction**
3. **Positive dot → highlight** (shift color toward light_hue)
4. **Negative dot → lowlight** (shift color toward lowlight_hue)

```
Light direction: vec2(-0.7, -0.7) (top-left)

░░░░░░░░
░▒▒▒▒▒▒▓
░▒▒▒▒▒▒▓
░▒▒▒▒▒▒▓
░▒▒▒▒▒▒▓
░▓▓▓▓▓▓▓

░ = highlight (normal faces toward light)
▓ = lowlight (normal faces away from light)
▒ = base (normal faces up / flat)
```

### Palette Stops

| Stop | Usage | Brightness |
|------|-------|------------|
| 1 | Max highlight | Brightest |
| 2 | Highlight | |
| 3 | Base (flat) | |
| 4 | Lowlight | |
| 5 | Max lowlight | Darkest |

Material base hue + light settings → final colors for each stop.

### Future Scope

- Scene lights (point/spot) affecting local regions
- Dynamic lighting from explosions, weapons, etc.

---

## Destruction Effects

Hardening is slow (structure builds over time). Crumbling is violent (destruction). Siblings of destroyed blocks should show damage before they break.

### Per-Material Destruction Character

Each material type breaks in a visually distinct way that reinforces its physical nature:

| Material Type | Destruction Character |
|---------------|----------------------|
| **Rock** | Chips and crumbles. Irregular fragments break away at edges. Damage spreads through existing fault lines. Heavy, grinding. |
| **Metal** | Bends and buckles under stress. Rivets pop. Panels warp before tearing. Resistant but yields suddenly. |
| **Crystal** | Fracture lines propagate instantly along geometric planes. Shatters into angular shards. Brittle, catastrophic failure. |
| **Organic** | Tears and stretches. Wounds appear as dark gaps. May show fluid at damage sites. Resilient but vulnerable to piercing. |
| **Brick/Stone** | Mortar fails first, releasing individual blocks. Cracks follow grid lines. Masonry collapses in sections. |

These visual treatments should feel like physical processes, not arbitrary tile swaps.

### Stress Propagation

When a block crumbles, its siblings receive stress. Stressed blocks show material-appropriate damage overlays:

```
DESTRUCTION SEQUENCE

1. Impact          2. Stress spreads      3. Chain reaction

   ┌───┬───┐          ┌───┬───┐            ┌───┬───┐
   │ ▓ │ ░ │          │ ▓ │▒░▒│            │ ▓ │ ▓ │
   ├───┼───┤          ├───┼───┤            ├───┼───┤
   │ ░ │ ░ │          │▒░▒│▒░▒│            │ ▓ │▒░▒│
   └───┴───┘          └───┴───┘            └───┴───┘

   ▓ = destroyed     ▒░▒ = stressed        more destruction
   ░ = stable        (showing damage)
```

- **Stress exceeds threshold** → block crumbles (chain reaction)
- **Stress below threshold** → block shows damage but remains stable; heals when block hardens

This makes destruction feel physical — not just tiles popping out of existence.

---

## Stability Progression

When visual detail appears based on quadtree level:

| Level | Block Size | What Appears |
|-------|------------|--------------|
| 0 | 2×2 | Flat material color |
| 1 | 4×4 | Perimeter edge lighting only |
| 2 | 8×8+ | Perimeter edges + interior detail |

```
LEVEL 0 (2×2)     LEVEL 1 (4×4)     LEVEL 2+ (8×8)

    ██                ░░░░            ░░░░░░░░
    ██                ░██▓            ░▒▒▒▒▒▒▓
                      ░██▓            ░▒▒■■▒▒▓
                      ░▓▓▓            ░▒▒■■▒▒▓
                                      ░▒▒▒▒▒▒▓
                                      ░▓▓▓▓▓▓▓

    flat            edges only      edges + interior
```

### Scope: Solids Only

This system applies to **solids only**. Liquids and gases receive base material color with global light tint only — no bump mapping, no interior detail. They remain fluid and simple, handled by sand physics rendering.

---

## Implementation Pipeline

### Per-Frame: Stability Detection (GPU)

After the sand simulation renders, GPU creates a **stability texture**:

```
STABILITY TEXTURE (per 2×2 block)

0 = empty (no material)
1 = active (any cell in block moved this frame)
2 = inactive (no cells moved)
```

This texture enables early-exit optimization: stable chunks can skip simulation entirely in subsequent frames.

### On Stability Change: Recalculate Texturing

Steps 2-5 only run when stability changes (not every frame):

**Step 2: Edge Normal Calculation (GPU)**

For inactive regions, calculate surface normals from height differences:

```
NORMAL TEXTURE (RG8 per pixel)

R = normal.x (0-255 maps to -1 to +1)
G = normal.y (0-255 maps to -1 to +1)

128, 128 = flat (no slope)
255, 128 = facing right
0, 128   = facing left
128, 255 = facing down
128, 0   = facing up
```

Shader uses normals directly with dot product — no lookup table needed.

GPU is good here — embarrassingly parallel, just neighbor sampling.

**Step 3: Hierarchy Level Calculation (CPU)**

Quadtree management runs on CPU:
- Track stability timers per node
- Attempt hardening when thresholds reached
- Handle crumbling cascade on activation

**Step 4: Tile Selection (CPU)**

Select appropriate grayscale tiles based on hierarchy level:

| Level | Tile Size | Tile Type |
|-------|-----------|-----------|
| 0 | 2×2 | Flat (no tile needed) |
| 1 | 4×4 | Pre-rendered edge lighting |
| 2 | 8×8 | Pre-rendered edge + interior detail |
| 3 | 16×16 | Pre-rendered full interior detail |

Tiles are **grayscale** — they encode the lighting pattern (highlight/base/lowlight) without color.

**Step 5: Colorization (GPU)**

Combine:
- Grayscale tile pattern
- Material base hue
- Global light/lowlight hue settings

Output: final colored pixels

```
COLORIZATION

grayscale_value (0-255) → palette_stop (1-5)
palette_stop + material_hue + light_settings → final_color
```

### Tile Assets

Pre-rendered grayscale tiles at 4×4, 8×8, 16×16:

```
GRAYSCALE VALUES

255 = max highlight (palette stop 1)
192 = highlight (palette stop 2)
128 = base (palette stop 3)
64  = lowlight (palette stop 4)
0   = max lowlight (palette stop 5)
```

Tiles include:
- Edge-only tiles (for small stable regions)
- Interior detail tiles per material type (brick, metal, organic, crystal, rock, etc.)

These get composed based on what the region needs.

### Early Exit Optimization

The stability texture enables skipping work:

```
IF chunk is stable AND chunk was stable last frame:
    Skip sand simulation for this chunk
    Skip edge recalculation
    Reuse existing tile assignment
```

**Question:** Is chunk-level early exit useful for GPU? Branch divergence might make this counterproductive. Need to profile.

### Update Triggers

Recalculate texturing when:
- Stability changes (active → inactive or vice versa)
- Time threshold reached (harden to higher hierarchy level)
- Adjacent region changes (edge normals may need update)

---

## Edge Detection

Cells at material boundaries need special treatment regardless of hierarchy level.

### Edge Types

| Edge | Visual Treatment |
|------|------------------|
| Top | Highlight / worn edge |
| Bottom | Lowlight / weight |
| Left/Right | Side shading |
| Corner | Special corner tile |
| Interior | Pattern based on level |

### Detection Method

For each cell, sample 4 neighbors:
- If neighbor is different material → this cell is an edge
- Edge cells use edge treatment instead of interior pattern
- Edge detection runs on the throttled schedule (not every frame)

---

## Resource Budget

### Textures per Active Chunk

| Texture | Size | Memory |
|---------|------|--------|
| Stability | 1-bit per 2×2 block = 128×128 | ~2KB |
| Normal | RG8 per pixel = 256×256×2 | ~128KB |
| Bump | R8 per pixel = 256×256 | ~64KB |
| **Total per chunk** | | **~194KB** |

With 16 active chunks (4×4 viewport + margin): **~3.1MB**

### Tile Atlas

5 sizes × 5 pattern types × 8 variants × avg 1KB = **~200KB**

### Quadtree Nodes

~340 nodes per chunk worst-case (full tree) × 16 bytes = 5.4KB/chunk

With 16 chunks: **~86KB**

---

## Implementation Considerations

### Data Structure

```
HierarchyNode {
    level: 0-5
    material: material_id (or MIXED if children differ)
    stable_since: timestamp
    stress: float (0-1, for destruction effects)
    children: [4] or null (leaf nodes)
}
```

### Update Schedule

Base timing (modified by material's `hardening_rate`):

| Level | Check Interval | Action |
|-------|----------------|--------|
| 0 | Every frame | Detect activity |
| 1 | 1s | Attempt hardening from level 0 |
| 2 | 2s | Attempt hardening from level 1 |
| 3 | 4s | Attempt hardening from level 2 |
| 4 | 8s | Attempt hardening from level 3 |
| 5 | 16s | Attempt hardening from level 4 |

### GPU Integration

Options for getting hierarchy data to the shader:

1. **Hierarchy texture** — One texture encoding level per region
2. **Indirect lookup** — Quadtree stored in texture, shader traverses
3. **Baked pattern texture** — CPU bakes patterns when hierarchy changes, GPU just samples

---

## Open Questions

1. **Chunk-level GPU early exit** — Is branch divergence a problem? Need to profile.

---

## References

- Bitmap Brothers pixel art style (Gods, Xenon 2, Speedball 2)
- Noita falling sand simulation
- Quadtree spatial data structures

---

## Changelog from v0.2

| Change | Rationale |
|--------|-----------|
| Clarified per-chunk, per-material quadtree organization | Bounded memory, clean chunk integration |
| Replaced WFC with deterministic tile selection | Simpler, predictable, no async complexity |
| Defined "active" precisely (cell moved positions) | Prevents jitter from resetting stability |
| Specified 2×2 block granularity for stability tracking | Matches level 0, reduces overhead |
| Added chunk boundary constraint section | Explicit limitation for implementation clarity |
| Added resource budget section | Memory/performance expectations |
| Expanded per-material destruction effects (conceptual) | Rock, metal, crystal, organic each break distinctly |
| Added stress field to HierarchyNode | Supports destruction propagation |
