# Procedural Level Generation v0.1

## Purpose

Define the complete pipeline for generating game levels procedurally, from high-level terrain placement down to pre-computed visual stability data. Levels should appear fully formed and textured on first frame.

---

## Current System Overview

The existing generation pipeline:

| Component | File | Purpose |
|-----------|------|---------|
| Generator | `game/terrain/generator.lua` | Grid-based feature placement |
| Features | `game/terrain/features.lua` | Individual terrain shapes (asteroids, platforms, etc.) |
| Shapes | `game/terrain/shapes.lua` | Material placement helpers |
| Streamer | `game/terrain/streamer.lua` | Progressive generation ahead of player |
| Config | `game/data/Terrain.json` | All tunable parameters |

**Current flow:**
1. `LevelBuilder.build()` calls `Generator.generate()` on game start
2. Generator iterates arena in grid steps, places features by probability
3. Features call Shapes to write material cells to the world
4. Sand physics simulates, stability shader detects settled material over 1-16 seconds
5. Textures gradually improve as quadtree promotes blocks

**Problem:** Pre-generated terrain takes 16+ seconds to reach full visual detail.

---

## Generation Pipeline (New)

```
┌─────────────────────────────────────────────────────────────────┐
│                    GENERATION PIPELINE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. LAYOUT GENERATION                                           │
│     ├── Biome assignment (future)                               │
│     ├── Feature density maps                                    │
│     └── Placement coordinates                                   │
│                                                                 │
│  2. MATERIAL PLACEMENT                                          │
│     ├── Features → Shapes → Grid cells                          │
│     ├── Per-chunk material bitmasks                             │
│     └── Track placed cells for quadmap                          │
│                                                                 │
│  3. QUADMAP GENERATION                                          │
│     ├── Build stability quadtree from material data             │
│     ├── Bottom-up promotion to higher levels                    │
│     └── Mark all nodes as fully hardened                        │
│                                                                 │
│  4. EDGE DETECTION                                              │
│     ├── Compute edge masks per stable block                     │
│     └── Determine interior vs edge tiles                        │
│                                                                 │
│  5. TEXTURE ASSIGNMENT                                          │
│     ├── Select tiles from atlas based on edge mask              │
│     └── Deterministic tile variant selection                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Phase 1: Layout Generation

Current system uses uniform random placement. Keep this for now.

```lua
-- generator.lua (existing pattern)
for y = step, h - step, step do
    for x = step, w - step, step do
        if random() < config.density then
            local ox = (random() - 0.5) * step * variation
            local oy = (random() - 0.5) * step * variation
            Features.asteroid(world, x + ox, y + oy, radius)
        end
    end
end
```

**Future enhancement:** Use noise-based density maps for biome variation.

---

## Phase 2: Material Placement

Current system writes cells directly to world grid. Add tracking for quadmap building.

### Option A: Track During Placement (Recommended)

```lua
-- shapes.lua
function Shapes.rockCircle(world, cx, cy, radius)
    local placed = {}
    for dy = -radius, radius do
        for dx = -radius, radius do
            if dx*dx + dy*dy <= radius*radius then
                local x, y = cx + dx, cy + dy
                world:set(x, y, MaterialType.ROCK)
                table.insert(placed, {x = x, y = y, material = MaterialType.ROCK})
            end
        end
    end
    return placed
end

-- features.lua
function Features.asteroid(world, cx, cy, radius, sandChance)
    local cells = Shapes.rockCircle(world, cx, cy, radius)
    world:trackCellsForQuadmap(cells)
end
```

### Option B: Scan After Placement

Read material data from chunks after all features placed:

```lua
-- After Generator.generate() completes
for _, chunk in world:iterateChunks() do
    chunk:buildQuadmapsFromMaterial()
end
```

---

## Phase 3: Quadmap Generation

Build stability quadtrees immediately from placed material data.

### Data Structure

```lua
{
    material = 1,  -- MaterialType.ROCK
    nodes = {
        [0] = {  -- Level 0: 128x128 grid of 2x2 blocks
            [key] = {stableFrames = 9999, isActive = false, hardenedLevel = 0}
        },
        [1] = {  -- Level 1: 64x64 grid of 4x4 blocks
            [key] = {stableFrames = 9999, isActive = false, hardenedLevel = 1}
        },
        -- ... up to level 5
    }
}
```

Key encoding: `key = y * gridSize + x`

### Build Algorithm

```lua
function buildQuadmapFromMaterial(chunk, materialID)
    local quadtree = StabilityQuadtree.new(materialID, hardeningRate)

    -- Phase 1: Create level-0 nodes from material data
    for y = 0, 255, 2 do  -- 256 cells, 2x2 blocks
        for x = 0, 255, 2 do
            if chunkHas2x2Block(chunk, x, y, materialID) then
                quadtree:createNode(0, x/2, y/2, {
                    stableFrames = 9999,
                    isActive = false,
                    hardenedLevel = 0
                })
            end
        end
    end

    -- Phase 2: Bottom-up promotion to higher levels
    for level = 1, 5 do
        quadtree:promoteFullyFilledBlocks(level)
    end

    return quadtree
end
```

### Promotion Logic

```lua
function StabilityQuadtree:promoteFullyFilledBlocks(level)
    local gridSize = GRID_SIZES[level]

    for y = 0, gridSize - 1 do
        for x = 0, gridSize - 1 do
            local cx, cy = x * 2, y * 2
            local children = {
                self:getNode(level - 1, cx, cy),
                self:getNode(level - 1, cx + 1, cy),
                self:getNode(level - 1, cx, cy + 1),
                self:getNode(level - 1, cx + 1, cy + 1)
            }

            local allHardened = true
            for _, child in ipairs(children) do
                if not child or child.hardenedLevel < level - 1 then
                    allHardened = false
                    break
                end
            end

            if allHardened then
                self:createNode(level, x, y, {
                    stableFrames = 9999,
                    isActive = false,
                    hardenedLevel = level
                })
            end
        end
    end
end
```

---

## Phase 4: Edge Detection

Compute edge masks for texture selection. Each block checks neighbors to determine exposed edges.

### Edge Mask Encoding

```
Bit order: TBLR (top, bottom, left, right)
  0000 = interior (fully surrounded)
  1000 = top edge exposed
  0100 = bottom edge exposed
  0010 = left edge exposed
  0001 = right edge exposed
  1010 = top-left corner
  etc.
```

### Detection Algorithm

```lua
function computeEdgeMask(quadtree, level, x, y)
    local mask = 0

    -- Check top neighbor
    if not quadtree:hasBlockAt(level, x, y - 1) then
        mask = mask + 8  -- 1000
    end
    -- Check bottom neighbor
    if not quadtree:hasBlockAt(level, x, y + 1) then
        mask = mask + 4  -- 0100
    end
    -- Check left neighbor
    if not quadtree:hasBlockAt(level, x - 1, y) then
        mask = mask + 2  -- 0010
    end
    -- Check right neighbor
    if not quadtree:hasBlockAt(level, x + 1, y) then
        mask = mask + 1  -- 0001
    end

    return mask
end
```

---

## Phase 5: Texture Assignment

Select tiles from atlas based on edge mask and block position.

```lua
function selectTile(blockX, blockY, level, edgeMask, pattern)
    local key = edgeMaskToKey(edgeMask)
    local variants = pattern.tiles[level][key]

    -- Fallback to interior if edge variant missing
    if not variants or #variants == 0 then
        variants = pattern.tiles[level]["interior"]
    end

    -- Deterministic selection from position hash
    local hash = (blockX * 73856093) ~ (blockY * 19349663)
    local index = (hash % #variants) + 1
    return variants[index]
end
```

See `/engine/rendering/Tile_Atlas_Pipeline.md` for atlas format details.

---

## Integration Points

### Skip Runtime Stability Detection

Pre-built quadmaps don't need GPU stability passes:

```lua
-- sand/init.lua
function StreamingSimulation:updateStability(frameNumber)
    for _, chunk in self.pool:iterateActive() do
        if chunk.quadmapsBuilt then
            -- Only run if terrain was modified by gameplay
            if chunk.terrainModified then
                self:runStabilityPass(chunk, frameNumber)
                chunk.terrainModified = false
            end
        else
            self:runStabilityPass(chunk, frameNumber)
        end
    end
end
```

### Mark Terrain Modified on Damage

```lua
function StreamingSimulation:setCell(worldX, worldY, material, color)
    local chunk = self:getChunkAt(worldX, worldY)
    if chunk then
        chunk.terrainModified = true
        -- ... existing cell placement
    end
end
```

---

## Modified Files

| File | Changes |
|------|---------|
| `game/terrain/shapes.lua` | Return placed cells for tracking |
| `game/terrain/features.lua` | Track cells for quadmap building |
| `game/terrain/generator.lua` | Call quadmap finalization after generation |
| `Engine/Physics/Sand/ChunkTexture.lua` | Add `buildQuadmapsFromMaterial()` |
| `Engine/Physics/Sand/StabilityQuadtree.lua` | Add `promoteFullyFilledBlocks()`, `createNode()` |
| `game/sandbox/sand/init.lua` | Skip stability detection for pre-built quadmaps |

---

## Streaming Generation

The streamer generates terrain ahead of player movement. Same pipeline applies:

```lua
-- streamer.lua
function Streamer:populateRegion(rx, ry)
    -- Existing feature placement
    self:placeFeatures(rx, ry)

    -- NEW: Build quadmaps for all chunks in region
    for _, chunk in self:chunksInRegion(rx, ry) do
        chunk:buildQuadmapsFromMaterial()
        chunk.quadmapsBuilt = true
    end
end
```

---

## Verification

1. Generate terrain, press F9 immediately
2. **Expected:** All levels (2x2 through 64x64) appear instantly
3. All terrain shows cyan (stable), not magenta (active)
4. Destroy terrain with weapon
5. **Expected:** Destroyed area shows magenta, then re-hardens over 1-16 seconds
6. Undamaged terrain remains fully hardened
7. Fly to new area (streaming generation)
8. **Expected:** New terrain appears fully hardened immediately

---

## Future Enhancements

### Biome System

Divide world into regions with different feature mixes:

```lua
biomes = {
    asteroid_field = {asteroid = 0.3, debris = 0.2},
    station_zone = {station = 0.1, platform = 0.2},
    hazard_zone = {nebula = 0.15, barrel = 0.1}
}
```

### Noise-Based Placement

Use Perlin noise for organic feature distribution:

```lua
local density = Noise.octave2d(x * 0.01, y * 0.01, 4, 0.5)
if random() < density * config.maxDensity then
    -- place feature
end
```

### Cave/Tunnel Generation

Subtract shapes from solid regions:

```lua
function Features.cave(world, cx, cy, length, width)
    -- Use noise to create winding tunnel path
    -- Subtract from existing rock
end
```
