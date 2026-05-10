---
name: roblox-api-reference
description: Use for any Roblox scripting task requiring knowledge of services, APIs, data types, or Luau syntax. Triggers on any mention of a Roblox service name, API call, data type (Vector3, CFrame, UDim2, Color3), or when writing any Luau script.
---

# Roblox API Reference — Complete Cheat Sheet

## Services Quick Reference

Always get services at the TOP of files with `game:GetService()`:

```lua
-- CORE
local Players            = game:GetService("Players")
local ReplicatedStorage  = game:GetService("ReplicatedStorage")
local ServerScriptService = game:GetService("ServerScriptService")
local ServerStorage      = game:GetService("ServerStorage")
local RunService         = game:GetService("RunService")
local TweenService       = game:GetService("TweenService")
local UserInputService   = game:GetService("UserInputService")
local CollectionService  = game:GetService("CollectionService")

-- GAMEPLAY
local MarketplaceService = game:GetService("MarketplaceService")
local DataStoreService   = game:GetService("DataStoreService")
local BadgeService       = game:GetService("BadgeService")
local GroupService       = game:GetService("GroupService")
local SoundService       = game:GetService("SoundService")
local Lighting           = game:GetService("Lighting")

-- UTILITY
local HttpService        = game:GetService("HttpService")
local InsertService      = game:GetService("InsertService")
local ContentProvider    = game:GetService("ContentProvider")
local TextService        = game:GetService("TextService")
local TeleportService    = game:GetService("TeleportService")
local PhysicsService     = game:GetService("PhysicsService")
local GuiService         = game:GetService("GuiService")
local HapticService      = game:GetService("HapticService")  -- mobile vibration
local PolicyService      = game:GetService("PolicyService")  -- region compliance
local AnalyticsService   = game:GetService("AnalyticsService")
local Debris             = game:GetService("Debris")

-- CLIENT ONLY
local ContextActionService = game:GetService("ContextActionService")
local StarterGui           = game:GetService("StarterGui")
```

---

## Data Types — Complete Reference

### Vector3
```lua
Vector3.new(x, y, z)
Vector3.zero        -- (0,0,0)
Vector3.one         -- (1,1,1)

v3.X, v3.Y, v3.Z
v3.Magnitude        -- length
v3.Unit             -- normalized
v3:Dot(other)       -- dot product
v3:Cross(other)     -- cross product
v3:Lerp(other, t)   -- interpolate (t = 0..1)
Vector3.FromNormalId(Enum.NormalId.Front)
```

### CFrame
```lua
CFrame.new(x, y, z)                    -- position only
CFrame.new(pos, lookAt)                -- look at point
CFrame.Angles(rx, ry, rz)             -- rotation in radians
CFrame.fromEulerAnglesXYZ(rx, ry, rz)
CFrame.fromAxisAngle(axis, angle)

cf.Position   -- Vector3
cf.LookVector -- forward direction (Vector3)
cf.RightVector
cf.UpVector
cf.X, cf.Y, cf.Z

-- Combine position + rotation:
CFrame.new(pos) * CFrame.Angles(0, math.rad(90), 0)

-- Offset from origin:
cf + Vector3.new(0, 5, 0)       -- move up 5
cf * CFrame.new(0, 0, -10)      -- move 10 forward (local space)

cf:ToWorldSpace(localCFrame)     -- local → world
cf:ToObjectSpace(worldCFrame)    -- world → local
cf:Lerp(other, t)                -- interpolate
```

### UDim2 (GUI sizing/positioning)
```lua
UDim2.new(xScale, xOffset, yScale, yOffset)
-- Scale: 0-1, relative to parent size
-- Offset: pixels, absolute

UDim2.fromScale(x, y)    -- UDim2.new(x,0, y,0)
UDim2.fromOffset(x, y)   -- UDim2.new(0,x, 0,y)

-- Common patterns:
UDim2.new(1, 0, 1, 0)      -- full parent size
UDim2.new(0.5, 0, 0.5, 0)  -- half parent size
UDim2.new(0, 200, 0, 50)   -- 200x50 pixels
UDim2.new(0.5, -100, 0.5, -25) -- centered 200x50

-- With AnchorPoint = Vector2.new(0.5, 0.5):
Position = UDim2.new(0.5, 0, 0.5, 0)  -- centered on screen
```

### Color3
```lua
Color3.fromRGB(r, g, b)     -- 0-255
Color3.fromHSV(h, s, v)     -- 0-1 each
Color3.new(r, g, b)         -- 0-1 each

color:Lerp(other, t)
Color3.fromHex("#FF5733")   -- hex string
```

### UDim
```lua
UDim.new(scale, offset)
-- Used for UICorner, UIListLayout padding, etc.
UDim.new(0, 12)   -- 12px corner radius
UDim.new(0.5, 0)  -- 50% radius (circle)
```

### Other Data Types
```lua
-- Rect (for SliceCenter in 9-slice images):
Rect.new(x0, y0, x1, y1)

-- NumberRange:
NumberRange.new(min, max)     -- for particle lifetime, etc.
NumberRange.new(5)            -- min == max

-- NumberSequence (for particle size/transparency):
NumberSequence.new(0)         -- constant
NumberSequence.new({
    NumberSequenceKeypoint.new(0, 0),    -- time=0, value=0
    NumberSequenceKeypoint.new(1, 1),    -- time=1, value=1
})

-- ColorSequence (for UIGradient, beams):
ColorSequence.new(Color3.fromRGB(255,0,0), Color3.fromRGB(0,0,255))
ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(255,0,0)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(0,0,255)),
})

-- BrickColor (legacy, use Color3 instead):
BrickColor.new("Bright red")
BrickColor.new(21)  -- by number
part.BrickColor = BrickColor.new("Bright blue")

-- Ray:
Ray.new(origin: Vector3, direction: Vector3)

-- Region3 (deprecated, use RaycastParams + Raycast):
-- Use workspace:Raycast() instead
```

---

## TweenService — Complete Reference

```lua
local TweenService = game:GetService("TweenService")

-- TweenInfo.new(time, easingStyle, easingDirection, repeatCount, reverses, delayTime)
local info = TweenInfo.new(
    0.3,                         -- duration (seconds)
    Enum.EasingStyle.Back,       -- easing style
    Enum.EasingDirection.Out,    -- direction
    0,                           -- repeat count (0 = play once)
    false,                       -- reverses
    0                            -- delay before starting
)

local tween = TweenService:Create(instance, info, {
    -- any numeric/Color3/CFrame/Vector3 property:
    Position = Vector3.new(0, 10, 0),
    BackgroundColor3 = Color3.fromRGB(255, 0, 0),
    BackgroundTransparency = 0.5,
    Size = UDim2.new(1, 0, 1, 0),
    TextTransparency = 0,
    ImageTransparency = 0,
    CFrame = CFrame.new(0, 5, 0),
    -- etc.
})

tween:Play()
tween:Pause()
tween:Cancel()
tween.Completed:Wait()  -- yield until done
tween.Completed:Connect(function(playbackState) end)

-- Playback states:
-- Enum.PlaybackState.Begin, Playing, Paused, Completed, Cancelled
```

### Easing Styles (visual feel)

| Style | Feel | Best for |
|---|---|---|
| `Linear` | Constant speed | Progress bars, day cycle |
| `Quad` | Gentle curve | General UI movement |
| `Cubic` | Moderate curve | Popups, panels |
| `Quart` | Strong curve | Dramatic slides |
| `Quint` | Very strong | Emphasis animations |
| `Sine` | Smooth wave | Idle animations, loops |
| `Expo` | Explosive then slow | Impact effects |
| `Circ` | Circular curve | Smooth arcs |
| `Back` | Overshoots then returns | Bouncy popups ✨ |
| `Elastic` | Springy oscillation | Physics feel |
| `Bounce` | Bounces at end | Ball/drop effects |

### Easing Directions

| Direction | Behavior |
|---|---|
| `In` | Starts slow, ends fast |
| `Out` | Starts fast, ends slow (most natural for UI) |
| `InOut` | Slow → fast → slow (smoothest) |

---

## RunService — When to Use What

```lua
local RunService = game:GetService("RunService")

-- Heartbeat: fires every frame AFTER physics step
-- Use for: game logic, NPC movement, coin collection checks
RunService.Heartbeat:Connect(function(deltaTime: number)
    -- deltaTime = seconds since last frame
end)

-- Stepped: fires every frame BEFORE physics step
-- Use for: manipulating physics before simulation
RunService.Stepped:Connect(function(time: number, deltaTime: number)
end)

-- RenderStepped: fires every frame BEFORE rendering (CLIENT ONLY)
-- Use for: camera, character visuals, smooth follow
RunService.RenderStepped:Connect(function(deltaTime: number)
end)

-- Check context:
RunService:IsServer()   -- true on server
RunService:IsClient()   -- true on client
RunService:IsStudio()   -- true in Studio

-- Throttle expensive Heartbeat work:
local lastRun = 0
RunService.Heartbeat:Connect(function()
    if tick() - lastRun < 0.1 then return end  -- max 10/sec
    lastRun = tick()
    -- expensive code
end)
```

---

## Players Service

```lua
local Players = game:GetService("Players")

Players.LocalPlayer          -- CLIENT ONLY: current player
Players.MaxPlayers           -- max server size
Players:GetPlayers()         -- array of all players

Players.PlayerAdded:Connect(function(player: Player) end)
Players.PlayerRemoving:Connect(function(player: Player) end)
Players:GetPlayerFromCharacter(character: Model)  -- Model → Player
Players:GetPlayerByUserId(userId: number)

-- Player properties:
player.Name           -- username string
player.DisplayName    -- display name
player.UserId         -- unique number
player.TeamColor
player.Character      -- Model (can be nil if not spawned)
player.Backpack       -- tools
player.PlayerGui      -- CLIENT: player's ScreenGuis
player.leaderstats    -- leaderboard stats folder (if created)

-- Player events:
player.CharacterAdded:Connect(function(character: Model) end)
player.CharacterRemoving:Connect(function(character: Model) end)

-- Character parts:
local character = player.Character
local hrp = character:FindFirstChild("HumanoidRootPart")
local humanoid = character:FindFirstChildOfClass("Humanoid")
local head = character:FindFirstChild("Head")

-- Humanoid:
humanoid.Health
humanoid.MaxHealth
humanoid.WalkSpeed    -- default 16
humanoid.JumpPower    -- default 50
humanoid.JumpHeight   -- new property (use instead of JumpPower)
humanoid:TakeDamage(amount)
humanoid.Died:Connect(function() end)
humanoid.HealthChanged:Connect(function(health) end)
```

---

## CollectionService — Tag-Based Systems

```lua
local CollectionService = game:GetService("CollectionService")

-- Add/remove tags (usually in Studio via Tag Editor plugin):
CollectionService:AddTag(instance, "KillBrick")
CollectionService:RemoveTag(instance, "KillBrick")
CollectionService:HasTag(instance, "KillBrick")

-- Get all tagged instances:
local killBricks = CollectionService:GetTagged("KillBrick")

-- React to new tagged instances:
CollectionService:GetInstanceAddedSignal("KillBrick"):Connect(function(instance)
    setupKillBrick(instance)
end)
CollectionService:GetInstanceRemovedSignal("KillBrick"):Connect(function(instance)
    -- cleanup
end)

-- Best pattern: setup existing + listen for new:
local function setupTagged(tag, setupFn)
    for _, inst in ipairs(CollectionService:GetTagged(tag)) do
        task.spawn(setupFn, inst)
    end
    CollectionService:GetInstanceAddedSignal(tag):Connect(function(inst)
        task.spawn(setupFn, inst)
    end)
end

setupTagged("KillBrick", function(part)
    part.Touched:Connect(function(hit)
        local humanoid = hit.Parent:FindFirstChildOfClass("Humanoid")
        if humanoid then humanoid.Health = 0 end
    end)
end)
```

---

## UserInputService (Client Only)

```lua
local UserInputService = game:GetService("UserInputService")

-- Key detection:
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end  -- ignore if typing in chat/textbox
    if input.KeyCode == Enum.KeyCode.E then
        -- E pressed
    end
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        -- Left click
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
    -- key/button released
end)

-- Check if key is currently held:
UserInputService:IsKeyDown(Enum.KeyCode.LeftShift)

-- Mouse position:
UserInputService:GetMouseLocation()  -- returns Vector2 (screen pixels)

-- Touch/mobile:
UserInputService.TouchStarted:Connect(function(touch, gameProcessed) end)
UserInputService.TouchEnded:Connect(function(touch, gameProcessed) end)

-- Check device type:
UserInputService.TouchEnabled      -- true on mobile
UserInputService.KeyboardEnabled   -- true on PC
UserInputService.GamepadEnabled    -- true if gamepad connected
UserInputService.MouseEnabled      -- true on PC

-- Lock/unlock mouse:
UserInputService.MouseBehavior = Enum.MouseBehavior.LockCenter
UserInputService.MouseBehavior = Enum.MouseBehavior.Default
```

---

## MarketplaceService

```lua
local MarketplaceService = game:GetService("MarketplaceService")

-- Check ownership (SERVER):
local owned = MarketplaceService:UserOwnsGamePassAsync(player.UserId, gamepassId)

-- Prompt purchase (SERVER or CLIENT):
MarketplaceService:PromptGamePassPurchase(player, gamepassId)
MarketplaceService:PromptProductPurchase(player, productId)

-- Purchase completed (SERVER ONLY):
MarketplaceService.PromptGamePassPurchaseFinished:Connect(
    function(player, passId, purchased)
        if purchased then
            -- apply perk
        end
    end
)

-- CRITICAL: ProcessReceipt for Developer Products:
MarketplaceService.ProcessReceipt = function(receiptInfo)
    local player = Players:GetPlayerByUserId(receiptInfo.PlayerId)
    if not player then
        return Enum.ProductPurchaseDecision.NotProcessedYet
    end
    -- grant product...
    return Enum.ProductPurchaseDecision.PurchaseGranted
end

-- Get product info:
local info = MarketplaceService:GetProductInfo(assetId, Enum.InfoType.Asset)
local info = MarketplaceService:GetProductInfo(passId, Enum.InfoType.GamePass)
```

---

## SoundService & Sound

```lua
local SoundService = game:GetService("SoundService")

-- Play a sound (in workspace or SoundService for global):
local sound = Instance.new("Sound")
sound.SoundId = "rbxassetid://12345678"
sound.Volume = 0.5         -- 0-10 (default 0.5)
sound.Pitch = 1            -- speed/pitch multiplier
sound.Looped = false
sound.RollOffMaxDistance = 40  -- how far sound travels in world
sound.Parent = workspace   -- 3D positional (attach to part for position)
-- OR:
sound.Parent = SoundService  -- global (no distance falloff)
sound:Play()
sound:Stop()
sound:Pause()

-- Completion event:
sound.Ended:Connect(function() sound:Destroy() end)

-- Background music:
local bgm = Instance.new("Sound")
bgm.SoundId = "rbxassetid://..."
bgm.Volume = 0.3
bgm.Looped = true
bgm.Parent = SoundService
bgm:Play()
```

---

## Debris Service

```lua
local Debris = game:GetService("Debris")

-- Auto-destroy instance after delay:
Debris:AddItem(instance, lifetime)

-- Example: temp effect that cleans itself up:
local effect = Instance.new("Part")
effect.Parent = workspace
Debris:AddItem(effect, 3)  -- destroyed after 3 seconds
-- Better than: task.delay(3, function() effect:Destroy() end)
```

---

## HttpService

```lua
local HttpService = game:GetService("HttpService")

-- JSON encode/decode:
local json = HttpService:JSONEncode(table)
local table = HttpService:JSONDecode(json)

-- Generate unique ID:
local uid = HttpService:GenerateGUID(false)  -- no dashes
-- Returns something like: "A1B2C3D4E5F6..."
```

---

## BadgeService

```lua
local BadgeService = game:GetService("BadgeService")

-- Award badge (server only):
local success, err = pcall(function()
    BadgeService:AwardBadge(player.UserId, badgeId)
end)

-- Check if player has badge:
local hasBadge = BadgeService:UserHasBadgeAsync(player.UserId, badgeId)
```

---

## InsertService (Load Toolbox assets)

```lua
local InsertService = game:GetService("InsertService")

-- Load a model by Asset ID:
local success, result = pcall(function()
    return InsertService:LoadAsset(assetId)
end)

if success then
    local model = result:FindFirstChildOfClass("Model") or result
    model.Parent = workspace
    -- position it:
    if model.PrimaryPart then
        model:SetPrimaryPartCFrame(CFrame.new(0, 0, 0))
    end
end
```

---

## ContentProvider (Preloading)

```lua
local ContentProvider = game:GetService("ContentProvider")

-- Preload assets before game starts:
local assets = {
    "rbxassetid://123",
    workspace.SomePart,  -- can pass instances too
}

ContentProvider:PreloadAsync(assets, function(asset, status)
    -- status: Enum.AssetFetchStatus.Success / Failure / None
    print(asset, status)
end)
```

---

## TeleportService

```lua
local TeleportService = game:GetService("TeleportService")

-- Teleport to another place:
TeleportService:Teleport(placeId, player)

-- Teleport with data:
local options = Instance.new("TeleportOptions")
options:SetTeleportData({ level = 5, coins = 100 })
TeleportService:TeleportAsync(placeId, { player }, options)

-- Receive data on the other side:
local data = TeleportService:GetLocalPlayerTeleportData()
```

---

## PhysicsService (Collision Groups)

```lua
local PhysicsService = game:GetService("PhysicsService")

-- Create collision groups:
PhysicsService:RegisterCollisionGroup("Players")
PhysicsService:RegisterCollisionGroup("Enemies")

-- Set group collision:
PhysicsService:CollisionGroupSetCollidable("Players", "Players", false)
-- Players don't collide with each other

-- Assign part to group:
part.CollisionGroup = "Players"
-- Or: PhysicsService:SetPartCollisionGroup(part, "Players") -- older API
```

---

## PolicyService (Required for Region Compliance)

```lua
local PolicyService = game:GetService("PolicyService")

-- Check before showing certain content:
local success, policy = pcall(function()
    return PolicyService:GetPolicyInfoForPlayerAsync(player)
end)

if success then
    -- policy.IsSubjectToChinaPolicies
    -- policy.ArePaidRandomItemsRestricted  ← loot boxes
    -- policy.AllowedExternalLinkReferences ← social media links
    -- policy.IsPaidItemTradingAllowed
end

-- IMPORTANT: Check ArePaidRandomItemsRestricted before showing egg/loot mechanics
```

---

## Workspace Raycasting (Modern)

```lua
-- New raycasting API (replaces FindPartOnRay):
local raycastParams = RaycastParams.new()
raycastParams.FilterType = Enum.RaycastFilterType.Exclude
raycastParams.FilterDescendantsInstances = { player.Character }

local origin    = hrp.Position
local direction = Vector3.new(0, -10, 0)  -- 10 studs down

local result = workspace:Raycast(origin, direction, raycastParams)
if result then
    print(result.Instance)   -- what was hit
    print(result.Position)   -- hit position (Vector3)
    print(result.Normal)     -- surface normal (Vector3)
    print(result.Distance)   -- distance to hit
end

-- Spherecast (checks sphere along path):
local result = workspace:Spherecast(origin, radius, direction, raycastParams)

-- Blockcast:
local result = workspace:Blockcast(originCFrame, size, direction, raycastParams)
```

---

## Attributes (modern alternative to Values)

```lua
-- Set/get arbitrary data on any instance:
instance:SetAttribute("Health", 100)
instance:SetAttribute("Name", "Boss")
instance:SetAttribute("IsActive", true)

local health = instance:GetAttribute("Health")  -- returns nil if not set
local all    = instance:GetAttributes()         -- returns { key = value }

-- Listen for changes:
instance:GetAttributeChangedSignal("Health"):Connect(function()
    local newHealth = instance:GetAttribute("Health")
end)

-- Use attributes instead of Value objects (IntValue, StringValue, etc.)
-- They're cleaner and don't pollute the Instance hierarchy
```

---

## Common Gotchas & Deprecated APIs

```lua
-- ❌ DEPRECATED → ✅ REPLACEMENT:
wait()                → task.wait()
spawn()               → task.spawn()
delay()               → task.delay()
coroutine.wrap()      → task.spawn()
game.Players.LocalPlayer:GetMouse()  → UserInputService (for most cases)
workspace.FindPartOnRay()  → workspace:Raycast()
Humanoid.JumpPower    → Humanoid.JumpHeight (new default system)
BodyVelocity          → LinearVelocity (constraint-based)
BodyPosition          → AlignPosition
BodyGyro              → AlignOrientation
BodyAngularVelocity   → AngularVelocity

-- ❌ Never use:
RemoteFunction:InvokeClient()  -- can freeze server if client disconnects
game.Players.LocalPlayer on Server  -- always nil on server
script.Parent on ModuleScript  -- unreliable, use require path instead

-- ✅ Character safety pattern:
local character = player.Character or player.CharacterAdded:Wait()
local hrp = character:WaitForChild("HumanoidRootPart")

-- ✅ Safe FindFirstChild vs WaitForChild:
-- FindFirstChild: returns nil if not found (non-blocking)
-- WaitForChild: yields until found or timeout (use on client for assets)
-- WaitForChild("Name", 5) -- 5 second timeout, returns nil after
```
