# Coordinate System

## Overview

The game uses a toroidal (wrapping) world. Three coordinate spaces exist.

## Spaces

| Space | Description | Example |
|-------|-------------|---------|
| **World (wrapped)** | Position within world bounds, wraps at edges | `player.x`, `player.y` |
| **World (unwrapped)** | Continuous position, never wraps | `player.worldX`, `player.worldY` |
| **Screen** | Pixel position on display | Player always at screen center |

## When to Use Each

**Toroidal objects** (terrain, enemies): Use wrapped coordinates. Use `bounds:wrapDelta()` for shortest-path calculations.

**Non-toroidal objects** (artifact during tutorial): Use unwrapped coordinates throughout. Store position relative to `player.worldX/Y`, render using `camera:getWorldPosition()` (unwrapped), calculate distances with direct subtraction. This prevents visual "flipping" when player crosses world boundaries.

## Key Functions

- `bounds:wrap(x, y)` - Wrap position to world bounds
- `bounds:wrapDelta(x1, y1, x2, y2)` - Shortest path delta (can flip direction at boundaries)
- `camera:getWrappedWorldPosition()` - Camera position in wrapped coords
- `camera:getWorldPosition()` - Camera position in unwrapped coords
- `camera:getScreenCenter()` - Screen center in pixels

## Converting Between Spaces

**World to Screen (standard approach):** Use LÖVE's transform stack. Draw objects at world coordinates inside the transform block.
```lua
local camX, camY = camera:getWrappedWorldPosition()
local screenCenterX, screenCenterY = camera:getScreenCenter()

love.graphics.push()
love.graphics.translate(-camX + screenCenterX, -camY + screenCenterY)
-- Draw objects at their world (x, y) coordinates here
entity:draw()
love.graphics.pop()
```

**World to Screen (manual):** For objects drawn outside the transform block (e.g., artifact):
```lua
local camX, camY = camera:getWorldPosition()  -- or getWrappedWorldPosition()
local screenCenterX, screenCenterY = camera:getScreenCenter()
local screenX = screenCenterX + (entity.x - camX)
local screenY = screenCenterY + (entity.y - camY)
```

**Wrapped to Unwrapped:** Not directly possible - unwrapped position must be tracked separately (see `player.worldX/Y`).

**Unwrapped to Wrapped:**
```lua
local wrappedX, wrappedY = bounds:wrap(unwrappedX, unwrappedY)
```

## Rule

Pick one coordinate system per object and use it consistently. Mixing wrapped/unwrapped causes bugs.
