# Performance Optimization

Production performance guide. Apply these before shipping.

---

## The Golden Rules

- **80% of Roblox players are on mobile** — optimize for low-end devices first
- **Target: under 10,000 parts** in workspace at any time
- **Target: 60 FPS on mid-range mobile** (Roblox benchmark device)
- Profile with **Microprofiler** (Ctrl+F6 in Studio), not guesswork

---

## Parts & Rendering

```lua
-- BAD: 50 parts for one building
-- GOOD: 1 MeshPart exported from Blender, or 1 Union

-- Check part count in Explorer → right-click Workspace → "Select Children" → count

-- StreamingEnabled for large maps (200+ stud radius)
-- Enable in Workspace properties:
workspace.StreamingEnabled = true
workspace.StreamingTargetRadius = 512  -- studs around player
-- WARNING: don't use StreamingEnabled for small/arena maps — overhead not worth it
```

**Part reduction checklist:**
- Replace building walls with single Unions or MeshParts
- Use Decals/Textures instead of colored parts for patterns
- Use LOD: swap detailed models for simpler ones at distance
- Anchor all static parts (`Part.Anchored = true`) — unanchored parts run physics
- Set `Part.CastShadow = false` on small decorative parts

---

## Lighting

```lua
-- Future lighting: best quality, most expensive (avoid on mobile-first games)
-- ShadowMap: good balance
-- Compatibility: fastest, use for mobile-heavy games

-- Each PointLight/SpotLight adds GPU cost
-- Rule: max 10-15 dynamic lights visible at once
-- Use SurfaceAppearance + baked lighting for static ambience instead

-- Disable shadows on non-critical lights:
light.Shadows = false
```

---

## Particles & Effects

```lua
-- Particles are GPU-expensive
-- Keep Rate LOW, Lifetime SHORT

local emitter = Instance.new("ParticleEmitter")
emitter.Rate = 10          -- not 100
emitter.Lifetime = NumberRange.new(0.5, 1.5)  -- not 5+
emitter.Enabled = false    -- disable when not needed

-- Enable only when player is nearby:
emitter.Enabled = true
task.delay(2, function() emitter.Enabled = false end)

-- Object pooling for frequently created/destroyed effects:
local pool = {}
local function getEffect()
    return table.remove(pool) or createNewEffect()
end
local function returnEffect(effect)
    effect.Enabled = false
    table.insert(pool, effect)
end
```

---

## Luau Performance

```lua
-- 1. Cache service references at top of file, never inside loops
local RunService = game:GetService("RunService")  -- ✅
-- NOT: game:GetService("RunService") inside loop  -- ❌

-- 2. Cache property reads in tight loops
local hrp = character.HumanoidRootPart
local pos = hrp.Position  -- cache once
-- NOT: hrp.Position repeatedly inside loop

-- 3. Use ipairs for arrays, pairs for dicts
for i, v in ipairs(array) do end   -- faster for arrays
for k, v in pairs(dict) do end     -- use for mixed tables

-- 4. Preallocate tables when size is known
local t = table.create(100, false)  -- 100 slots, default false

-- 5. String concatenation in loops: use table.concat
local parts = {}
for i = 1, 100 do
    parts[i] = tostring(i)
end
local result = table.concat(parts, ", ")
-- NOT: result = result .. tostring(i) in loop  -- creates new string each time

-- 6. task.wait() not wait() — never use deprecated wait()
task.wait(0.1)   -- ✅
wait(0.1)        -- ❌ deprecated

-- 7. task.spawn() not coroutine.wrap()
task.spawn(function()
    -- async work
end)

-- 8. Disconnect events when no longer needed
local conn = RunService.Heartbeat:Connect(function() end)
-- later:
conn:Disconnect()
```

---

## RunService Usage

```lua
-- Heartbeat: after physics, use for gameplay logic
-- RenderStepped: before render, use for camera/character (CLIENT ONLY)
-- Stepped: before physics, use for physics manipulation

-- NEVER put expensive work in Heartbeat without throttling:
local lastRun = 0
RunService.Heartbeat:Connect(function(dt)
    local now = tick()
    if now - lastRun < 0.1 then return end  -- run max 10x/sec
    lastRun = now
    -- your logic
end)

-- Prefer event-driven over polling wherever possible:
-- BAD: check every frame if player has item
-- GOOD: fire event when item is picked up
```

---

## Memory Management

```lua
-- Destroy instances you no longer need
part:Destroy()  -- removes from game AND frees memory
part.Parent = nil  -- only removes from game, still in memory if referenced

-- Clear large tables when done:
for k in pairs(bigTable) do bigTable[k] = nil end

-- Use weak tables for caches (GC can collect them):
local cache = setmetatable({}, {__mode = "v"})

-- ContentProvider preload for smooth experience:
local ContentProvider = game:GetService("ContentProvider")
local assets = { "rbxassetid://123", "rbxassetid://456" }
ContentProvider:PreloadAsync(assets)
```

---

## Network / DataStore

```lua
-- DataStore rate limits (Roblox enforces these):
-- GetAsync: 60 + numPlayers * 10 requests/min
-- SetAsync: 60 + numPlayers * 10 requests/min
-- Never save more than once per 6 seconds per key

-- Batch data into one key instead of multiple:
-- BAD: separate keys for coins, gems, level, inventory
-- GOOD: one key with table { coins, gems, level, inventory }

-- RemoteEvent firing rate — don't spam:
-- BAD: FireServer() every frame
-- GOOD: FireServer() on discrete player actions only
-- If you need frequent updates (position sync), use unreliable events:
local RE = Instance.new("UnreliableRemoteEvent")  -- new 2024+, for high-frequency lossy data
```

---

## Mobile-First Checklist

Before shipping, verify on mobile:

- [ ] UI buttons minimum 44px tap target
- [ ] No keyboard-only interactions (everything has touch equivalent)
- [ ] FPS stable at 60 on mid-range device (test with Quality Level 4)
- [ ] Part count under 10,000 visible at once
- [ ] No Future lighting (use ShadowMap or Compatibility)
- [ ] Textures under 512x512 for environment, 1024x1024 max for featured items
- [ ] StreamingEnabled ON for maps larger than 500 studs
- [ ] No more than 15 active particle emitters at once
- [ ] All sounds have `.RollOffMaxDistance` set (don't play globally)

---

## Microprofiler Targets

Open with Ctrl+F6 in-game. Target these frame times:

| Category | Target |
|---|---|
| Total frame | < 16.6ms (60 FPS) |
| Physics | < 4ms |
| Rendering | < 8ms |
| Scripts | < 3ms |
| Network | < 2ms |

If Scripts > 3ms: find the hot loop with `debug.profilebegin("label")` / `debug.profileend()`
