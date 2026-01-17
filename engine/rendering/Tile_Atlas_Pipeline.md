# Tile Atlas Pipeline v0.1

## Purpose

Define the format for texture atlases produced by external art tools and consumed by the game runtime. Tiles replace flat procedural colors with pre-painted pixel art.

---

## Phased Approach

| Phase | Focus | Texture Format |
|-------|-------|----------------|
| **1** | Flat RGB tiles | Full-color RGBA, no lighting |
| 2 | Bump maps | Grayscale height → runtime colorization |
| 3 | Normal maps | Per-pixel surface normals |

**This spec covers Phase 1 only.**

---

## Tile Sizes

Tiles match the stability hierarchy levels:

| Level | Tile Size | Block Size (wu) | Use Case |
|-------|-----------|-----------------|----------|
| 1 | 4×4 px | 4×4 wu | Small fills, edges |
| 2 | 8×8 px | 8×8 wu | Common fill size |
| 3 | 16×16 px | 16×16 wu | Medium stable areas |
| 4 | 32×32 px | 32×32 wu | Large stable areas |
| 5 | 64×64 px | 64×64 wu | Very large stable areas |

---

## Atlas Format

### PNG File: `tiles/{pattern}.png`

- Power-of-2 dimensions (e.g., 256×256, 512×512)
- RGBA PNG (full color + alpha for edge transparency)
- Nearest-neighbor filtering (pixel-perfect)
- All tile sizes packed into single atlas

### JSON Metadata: `tiles/{pattern}.json`

```json
{
  "pattern": "rock",
  "material_ids": [1],
  "atlas_size": [256, 256],

  "tiles": {
    "4x4": {
      "interior": [
        {"x": 0, "y": 0},
        {"x": 4, "y": 0},
        {"x": 8, "y": 0}
      ],
      "edge_t": [{"x": 0, "y": 16}],
      "edge_b": [{"x": 4, "y": 16}],
      "edge_l": [{"x": 8, "y": 16}],
      "edge_r": [{"x": 12, "y": 16}],
      "corner_tl": [{"x": 0, "y": 32}],
      "corner_tr": [{"x": 4, "y": 32}],
      "corner_bl": [{"x": 8, "y": 32}],
      "corner_br": [{"x": 12, "y": 32}]
    },
    "8x8": {
      "interior": [{"x": 0, "y": 48}, {"x": 8, "y": 48}],
      "edge_t": [{"x": 0, "y": 64}]
    },
    "16x16": {},
    "32x32": {},
    "64x64": {}
  }
}
```

---

## Edge Variants

Phase 1 uses 9 edge configurations:

| Key | Edge Mask | Description |
|-----|-----------|-------------|
| `interior` | `0000` | Fully surrounded by same material |
| `edge_t` | `1000` | Top edge exposed to empty/other |
| `edge_b` | `0100` | Bottom edge exposed |
| `edge_l` | `0010` | Left edge exposed |
| `edge_r` | `0001` | Right edge exposed |
| `corner_tl` | `1010` | Top + left edges exposed |
| `corner_tr` | `1001` | Top + right edges exposed |
| `corner_bl` | `0110` | Bottom + left edges exposed |
| `corner_br` | `0101` | Bottom + right edges exposed |

Edge mask bit order: `TBLR` (top, bottom, left, right).

### Future: Full 16-variant support

All combinations of exposed edges (0000 to 1111) for complex shapes.

---

## Tile Selection Algorithm

Runtime selects tiles deterministically based on block position:

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

---

## Directory Structure

```
game/assets/textures/tiles/
├── rock.png
├── rock.json
├── sand.png
├── sand.json
├── metal.png
├── metal.json
└── ...
```

---

## Runtime Loader

New module `game/rendering/tiles.lua`:

```lua
local Tiles = {}

function Tiles.load()
    local patterns = {"rock", "sand", "metal"}
    for _, name in ipairs(patterns) do
        local path = "assets/textures/tiles/" .. name
        Tiles[name] = {
            image = love.graphics.newImage(path .. ".png"),
            meta = json.decode(love.filesystem.read(path .. ".json"))
        }
        Tiles[name].image:setFilter("nearest", "nearest")
    end
end

function Tiles.getQuad(pattern, level, edgeMask, blockX, blockY)
    local tile = selectTile(blockX, blockY, level, edgeMask, Tiles[pattern].meta)
    local size = 2 ^ (level + 1)  -- 4, 8, 16, 32, 64
    return love.graphics.newQuad(
        tile.x, tile.y, size, size,
        Tiles[pattern].meta.atlas_size[1],
        Tiles[pattern].meta.atlas_size[2]
    )
end

return Tiles
```

---

## Material → Pattern Mapping

Add to `Materials.lua`:

```lua
Materials.properties[MaterialType.ROCK].pattern = "rock"
Materials.properties[MaterialType.SAND].pattern = "sand"
Materials.properties[MaterialType.EXPLOSIVE].pattern = "metal"
```

---

## Verification

1. Create test atlas with placeholder tiles
2. Load in game, verify no errors
3. Render a hardened chunk using tiles instead of flat colors
4. Verify tile selection is deterministic (same position = same tile)
5. Verify edge detection picks correct variants
