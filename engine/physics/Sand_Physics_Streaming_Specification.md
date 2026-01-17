# Sand Physics Streaming Specification v0.1

## Overview

This specification defines the chunked streaming architecture for GPU-based sand physics simulation using Margolus neighborhoods. It enables arbitrarily large worlds while maintaining 60fps performance and toroidal wrap behavior.

---

## Part 1: Units

### 1.1 Core Units

| Term | Definition | Notes |
|------|------------|-------|
| **Cell** | One position in the simulation grid | Can be empty or contain a particle |
| **Particle** | Material instance occupying a cell | Sand, water, gas, etc. |
| **World Unit (wu)** | Base coordinate unit; 1 wu = 1 cell | Used for all positions, distances, sizes |
| **Texel** | One element in a texture | Standard graphics term |
| **Display Pixel** | Physical screen pixel | After CRT upscale |

### 1.2 Simulation Units

| Term | Size | Notes |
|------|------|-------|
| **Cell** | 1×1 wu | Atomic simulation element |
| **Margolus Block** | 2×2 cells | Processing unit for cellular automata |
| **Chunk** | 256×256 wu | Streaming/processing unit |
| **Ghost Halo** | 2 wu | Overlap region for chunk boundaries |

### 1.3 Rendering Units

| Term | Size | Notes |
|------|------|-------|
| **Game Texel** | 1×1 wu | What gets drawn to game framebuffer |
| **Display Pixel** | 0.5×0.5 wu | After 2× CRT upscale |
| **Viewport** | 640×360 wu | Visible game area |
| **Display Resolution** | 1280×720 display pixels | Final output |

### 1.4 Unit Relationships

```
1 cell = 1 wu = 1 game texel
1 Margolus block = 2×2 cells = 2×2 wu
1 game texel → 2×2 display pixels (via CRT upscale)
1 wu = 2 display pixels (in each axis)

Viewport: 640×360 wu = 1280×720 display pixels
```

### 1.5 Abbreviations

| Abbreviation | Meaning |
|--------------|---------|
| wu | World unit |
| dp | Display pixel |

---

## Part 2: Resolution & Rendering Pipeline

### 2.1 Target Resolutions

| Stage | Resolution | Ratio | Notes |
|-------|------------|-------|-------|
| Game World | 640×360 wu | 1× | All gameplay, physics, rendering |
| Sand Physics | 640×360 cells | 1× | 1 cell = 1 wu = 1 game texel |
| CRT Shader Input | 640×360 texels | 1× | Game framebuffer, GL_NEAREST sampling |
| CRT Shader Output | 1280×720 dp | 2× | Final display resolution |

### 2.2 Size Relationships

| Unit | Size | Visual Result |
|------|------|---------------|
| 1 cell | 1 wu | 2×2 dp after CRT upscale |
| 1 Margolus block | 2×2 wu | 4×4 dp |
| Minimum visible element | 1 wu | 2×2 dp |

**Hard requirement:** No sub-wu elements. Everything is authored at 640×360 wu; CRT shader upscales 2× to 1280×720 dp.

### 2.3 Rendering Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│ GAME RENDER (640×360)                                       │
│   Sand chunks → composited to game framebuffer              │
│   Entities, effects, UI → same framebuffer                  │
│   All at 1:1 game resolution                                │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ CRT SHADER                                                  │
│   Input: 640×360 (GL_NEAREST)                               │
│   Upscale: 2×                                               │
│   Effects: Smear, scanlines, barrel, chromatic aberration   │
│   Output: 1280×720                                          │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ DISPLAY                                                     │
│   1280×720 final output                                     │
└─────────────────────────────────────────────────────────────┘
```

---

## Part 3: Coordinate Systems

The physics engine owns all coordinate system definitions. All systems must use these consistently.

### 3.1 Coordinate Spaces

| Space | Origin | Units | Resolution | Wrap Behavior | Owner |
|-------|--------|-------|------------|---------------|-------|
| **Absolute World** | Universe origin (0,0) | wu | Unbounded | None—extends infinitely | Physics Engine |
| **Toroidal World** | Level origin (0,0) | wu | Level-dependent | Wraps at level bounds | Physics Engine |
| **Chunk Local** | Chunk origin (0,0) | wu | 256×256 wu | None—bounded by chunk | Physics Engine |
| **Camera** | Camera center | wu | 640×360 wu viewport | None | Rendering System |
| **Screen** | Top-left of viewport | dp | 1280×720 dp | None | Rendering System |

**Note:** 1 wu = 2 dp (due to CRT upscale). All gameplay logic uses wu; only the CRT shader operates in dp.

### 3.2 Coordinate Transforms

```
Absolute World → Toroidal World (all in wu):
    toroidal.x = absolute.x % level_width
    toroidal.y = absolute.y % level_height

Toroidal World → Chunk Local (all in wu):
    chunk_index.x = floor(toroidal.x / chunk_size)
    chunk_index.y = floor(toroidal.y / chunk_size)
    local.x = toroidal.x % chunk_size
    local.y = toroidal.y % chunk_size

Toroidal World → Camera (wu):
    camera = toroidal - camera_center

Camera (wu) → Screen (dp):
    screen = camera * 2 + viewport_center
    // Factor of 2 due to CRT upscale

wu → dp:
    dp = wu * 2

dp → wu:
    wu = floor(dp / 2)
```

### 3.3 Resolution Constraints

All spatial dimensions must be powers of 2 (in wu):

| Property | Constraint | Example | Rationale |
|----------|------------|---------|-----------|
| Level dimensions | 2^n wu | 2048×1024 wu | Clean chunk division, toroidal wrap |
| Chunk size | 256×256 wu | — | Streaming unit, balances sync overhead vs load size |
| Flow field cell size | 2^n wu | 64×64 wu | Clean chunk subdivision |
| Margolus block | 2×2 wu | — | Cellular automata neighborhood |
| Viewport | 640×360 wu | — | Fixed for 720p output |

---

## Part 4: Chunk Architecture

### 4.1 Chunk Properties

| Property | Value | Rationale |
|----------|-------|-----------|
| Chunk size | 256×256 wu | Balance between streaming granularity and sync overhead |
| Ghost halo | 2 wu all edges | Margolus 2×2 block overlap |
| Texture size | 260×260 texels | Chunk + halo |
| Owned region | 256×256 wu interior | Excludes halo |

### 4.2 Chunk Layout

```
+--+--------------------+--+
|  |     Top Halo (2wu) |  |  ← Read-only (from neighbor above)
+--+--------------------+--+
|  |                    |  |
|L |                    |R |
|  |   Owned Region     |  |
|H |    (256×256)       |H |
|  |                    |  |
|  |                    |  |
+--+--------------------+--+
|  |  Bottom Halo (2wu) |  |  ← Read-only (from neighbor below)
+--+--------------------+--+
 ↑                        ↑
 Left Halo                Right Halo
 (read-only)              (read-only)
```

### 4.3 Toroidal Neighbor Mapping

For a level with `chunks_wide × chunks_tall` chunks:

```
neighbor_left  = (chunk_x - 1 + chunks_wide) % chunks_wide
neighbor_right = (chunk_x + 1) % chunks_wide
neighbor_above = (chunk_y - 1 + chunks_tall) % chunks_tall
neighbor_below = (chunk_y + 1) % chunks_tall
```

Corner halos require diagonal neighbors (4 additional):
```
neighbor_top_left     = (chunk_x - 1, chunk_y - 1) wrapped
neighbor_top_right    = (chunk_x + 1, chunk_y - 1) wrapped
neighbor_bottom_left  = (chunk_x - 1, chunk_y + 1) wrapped
neighbor_bottom_right = (chunk_x + 1, chunk_y + 1) wrapped
```

---

## Part 5: Margolus Integration

### 5.1 Block Ownership

Each Margolus 2×2 block is processed by exactly one chunk, determined by the block's top-left cell:

```
Block top-left at (x, y) in wu:
    owning_chunk.x = floor(x / chunk_size)
    owning_chunk.y = floor(y / chunk_size)
```

### 5.2 Phase Handling

| Phase | Block Origins (wu) | Boundary Behavior |
|-------|-------------------|-------------------|
| 0 (even frames) | (0,0), (2,0), (0,2), (2,2)... | Blocks align with chunk boundaries |
| 1 (odd frames) | (1,1), (3,1), (1,3), (3,3)... | Blocks straddle chunk boundaries |

**Phase 1 boundary blocks:** When a block's top-left is at position (chunk_size - 1, y) or (x, chunk_size - 1), it straddles into the neighboring chunk. The chunk owning the top-left cell processes it; the neighbor reads the result via ghost halo next frame.

### 5.3 Sync Protocol

```
Frame N:

1. COPY HALOS (GPU copy or compute shader)
   - Each chunk reads 2 wu edge strips from 8 neighbors (Frame N-1 data)
   - Toroidal wrap handled via neighbor mapping

2. RUN MARGOLUS (compute shader)
   - Process all blocks where top-left cell is in owned region
   - Sample flow field for movement probability
   - Use deterministic RNG keyed to world position (wu)

3. WRITE OUTPUT
   - Write only to owned region (256×256 wu interior)
   - Halo regions remain read-only

Frame N+1:
   - Neighbors see this frame's output in their halo copy
   - 1-frame latency at chunk boundaries
```

---

## Part 6: Flow Field Interface

### 6.1 Contract

The Flow Field System provides:

| Property | Specification |
|----------|---------------|
| Format | RGBA texture |
| Encoding | RGB = normalized direction XY (signed), A = magnitude |
| Filtering | GL_LINEAR (hardware bilinear interpolation) |
| Wrap mode | GL_REPEAT (toroidal) |
| Update frequency | As needed (not every frame) |
| Coverage | Entire toroidal world |

The Sand Physics System:
- Samples flow field at Margolus block centers
- Converts flow vector to movement probability
- Does not modify flow field

### 6.2 Flow Vector Encoding

```
R = (direction.x + 1.0) * 0.5    // Map [-1,1] to [0,1]
G = (direction.y + 1.0) * 0.5    // Map [-1,1] to [0,1]
B = reserved (future use)
A = magnitude [0,1]
```

Decoding in shader:
```glsl
vec4 sample = texture(flow_field, uv);
vec2 direction = sample.rg * 2.0 - 1.0;
float magnitude = sample.a;
vec2 flow = normalize(direction) * magnitude;
```

### 6.3 Flow to Probability Conversion

For Margolus movement, convert continuous flow vector to discrete cardinal probability:

```glsl
vec2 flow = sample_flow(block_center);

// Decompose into cardinal components
float prob_right = max(0.0, flow.x);
float prob_left  = max(0.0, -flow.x);
float prob_down  = max(0.0, flow.y);
float prob_up    = max(0.0, -flow.y);

// Normalize to probabilities (or use magnitude scaling)
float total = prob_right + prob_left + prob_down + prob_up;
if (total > 0.0) {
    prob_right /= total;
    prob_left  /= total;
    prob_down  /= total;
    prob_up    /= total;
}
```

### 6.4 Interpolation Behavior

Bilinear interpolation naturally handles:

| Scenario | Result |
|----------|--------|
| Opposing flows meet | Vectors cancel → low/zero magnitude |
| Perpendicular flows | Vectors blend → diagonal flow |
| Flow vs zero | Gradient ramp at boundary |

**No hard flow boundaries.** All transitions are smooth gradients.

---

## Part 7: Deterministic RNG

### 7.1 Requirements

- Same world position + frame → same random value (parallel chunk agreement)
- No visible patterns in dense particle fields
- Minimal GPU cost

### 7.2 Implementation

**World-space blue noise with frame offset:**

```glsl
uniform sampler2D blue_noise_128;  // 128×128 tileable blue noise

float get_probability(ivec2 world_pos, uint frame) {
    // Frame multipliers coprime to 128 for full coverage
    ivec2 offset = ivec2(frame * 17u, frame * 31u);
    ivec2 noise_coord = (world_pos + offset) & 127;  // Fast modulo for power of 2
    return texelFetch(blue_noise_128, noise_coord, 0).r;
}
```

### 7.3 Blue Noise Texture

| Property | Value |
|----------|-------|
| Resolution | 128×128 |
| Format | R8 (single channel, 8-bit) |
| Size | 16KB |
| Generation | Void-and-cluster or similar algorithm |
| Tiling | Seamless (tileable in both axes) |

---

## Part 8: Memory Layout

### 8.1 Per-Chunk Textures

| Texture | Size | Format | Purpose |
|---------|------|--------|---------|
| Sand State A | 260×260 | RGBA8 | Double buffer A (material ID + metadata) |
| Sand State B | 260×260 | RGBA8 | Double buffer B |

**Double buffering:** Read from A, write to B, swap each frame.

### 8.2 Global Textures

| Texture | Size | Format | Purpose |
|---------|------|--------|---------|
| Flow Field | Level-dependent | RGBA8 | Gravity/flow vectors |
| Blue Noise | 128×128 | R8 | Deterministic RNG |
| Material LUT | 256×1 | RGBA8 | Material properties lookup |

### 8.3 Streaming Budget

For a 4096×4096 wu level (16×16 chunks):

| Resource | Calculation | Size |
|----------|-------------|------|
| Chunk textures | 256 chunks × 2 buffers × 260² × 4 bytes | ~138 MB |
| Flow field (64 wu cells) | 64×64 × 4 bytes | 16 KB |
| Blue noise | 128×128 × 1 byte | 16 KB |

**Active set (viewport + buffer):** ~12 chunks loaded at once = ~6.5 MB

### 8.4 Streaming & Load-Ahead

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| Max ship speed | 500 wu/s | Design constraint for streaming |
| Chunk cross time | 0.51 sec (256 wu ÷ 500 wu/s) | Time to traverse one chunk at max speed |
| Load-ahead distance | 1 chunk (256 wu) | Buffer beyond visible edge |
| Load-ahead time | ~0.5 sec | Time available to async load |

**Viewport chunk coverage:**
```
Viewport: 640×360 wu
Chunks visible: ceil(640/256) × ceil(360/256) = 3×2 = 6 chunks
With load-ahead buffer: 5×4 = 20 chunks max loaded
```

**Chunk lifecycle:**
```
UNLOADED → LOADING → READY → ACTIVE → READY → UNLOADING → UNLOADED
              ↑                                    ↓
              └────────────────────────────────────┘
                    (re-enter view later)
```

**Load priority:**
1. Chunks intersecting viewport (must be ACTIVE)
2. Chunks within load-ahead distance (should be READY)
3. Recently exited chunks (keep READY for backtracking)

**Streaming active set:** Only chunks near camera need full-resolution textures. Distant chunks can be:
- Unloaded entirely
- Stored as compressed snapshots
- Simulated at reduced frequency

---

## Part 9: Dependencies

### 9.1 Required From Flow Field System

| Requirement | Specification |
|-------------|---------------|
| Texture format | RGBA8, GL_LINEAR filtering, GL_REPEAT wrap |
| Coordinate space | Toroidal world space (wu) |
| Update notification | Event/flag when flow field changes |
| Coverage | Must cover entire level bounds |

### 9.2 Required From Rendering System

| Requirement | Specification |
|-------------|---------------|
| Game framebuffer | 640×360 wu |
| Camera position | In toroidal world coordinates (wu) |
| Visible chunk set | List of chunk indices to render |
| Chunk-to-screen transform | For compositing chunk textures to game framebuffer |

### 9.3 Required From CRT Shader

| Requirement | Specification |
|-------------|---------------|
| Input resolution | 640×360 texels (game framebuffer) |
| Input filtering | GL_NEAREST (preserve sharp cells) |
| Output resolution | 1280×720 dp (2× upscale) |
| Upscale method | Nearest neighbor (each wu → 2×2 dp) |

**CRT Shader Modifications Required:**
```glsl
// Current shader uses screenSize for scanlines:
float pixelY = uv.y * screenSize.y;
float scanlinePattern = mod(floor(pixelY), 2.0);

// Should use gameSize to align scanlines with game cells:
extern vec2 gameSize;  // 640×360
float pixelY = uv.y * gameSize.y;
float scanlinePattern = mod(floor(pixelY), 2.0);

// Smear sampling should also align to game cells
vec2 pixelSize = 1.0 / gameSize;  // Not screenSize
```

### 9.4 Required From Level System

| Requirement | Specification |
|-------------|---------------|
| Level dimensions | Power of 2, in wu |
| Chunk grid dimensions | Derived: level_size / 256 |
| Toroidal flag | Whether level wraps |

### 9.5 Required From Ship/Movement System

| Requirement | Specification |
|-------------|---------------|
| Max speed | 500 wu/s hard cap |
| Position updates | In wu, toroidal wrapped |
| Speed notification | Event when approaching max speed (for prefetch priority) |

### 9.6 Provided To Other Systems

| Export | Specification |
|--------|---------------|
| Sand state textures | Per-chunk RGBA8, owned region only |
| Collision queries | Point/rect queries against sand density (wu coords) |
| Material queries | Material ID at world position (wu coords) |

---

## Part 10: Assumptions

| Assumption | Rationale |
|------------|-----------|
| GPU supports compute shaders | Required for Margolus simulation |
| GL_REPEAT wrap mode available | Toroidal texture sampling |
| 260×260 textures addressable | Chunk size (256) + halo (4) |
| Bilinear filtering available | Flow field smoothing |
| At least 64MB VRAM | Active chunk set (~20 chunks × 260² × 4 × 2) |

---

## Part 11: Open Questions

1. **Inactive chunk behavior:** Pause simulation? Reduce frequency? Snapshot?
2. **Flow field resolution:** 64 wu cells? 32 wu? Configurable?
3. **Material metadata:** What fits in RGBA8 per particle? (ID, temperature, age?)
4. **Collision interface:** How do rigid bodies query sand state efficiently?

---

## Part 12: Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2026-01-06 | Initial specification. Unit system (cell, particle, wu, dp), resolution pipeline (640×360 wu → 1280×720 dp), coordinate systems (5 spaces), chunk architecture (256×256 wu), Margolus integration, flow field interface, deterministic RNG, memory layout with streaming/load-ahead (500 wu/s max speed), dependencies. |
