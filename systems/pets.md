# Pet System

See `genres/pet-simulator.md` for full egg hatching + rarity system.

This file covers pets as a **multiplier mechanic** used across all genres —
not just pet simulators.

## When to use pets as multipliers

Any game where players collect companions that boost earnings:
- Simulator: pet gives +20% coins per click
- Tycoon: pet gives +10% cash income
- Active simulator: pet gives +luck on order results

## Quick Integration

### 1. Add to DEFAULT_DATA

```lua
OwnedPets    = {},   -- { uid = { petId, equipped } }
EquippedPets = {},   -- { uid, uid, uid } max 3
ActivePetMultiplier = 1.0,
```

### 2. PetConfig (minimal version)

```lua
-- ReplicatedStorage/Shared/Modules/PetConfig.lua
return {
    BasicCat = {
        id         = "BasicCat",
        displayName = "Basic Cat",
        rarity     = "Common",
        assetId    = 0,
        multiplier = 1.2,   -- +20% to all earnings
    },
    RareFox = {
        id         = "RareFox",
        displayName = "Rare Fox",
        rarity     = "Rare",
        assetId    = 0,
        multiplier = 1.8,
    },
    EpicDragon = {
        id         = "EpicDragon",
        displayName = "Epic Dragon",
        rarity     = "Epic",
        assetId    = 0,
        multiplier = 3.0,
    },
}
```

### 3. Apply multiplier in earning calculations

```lua
-- In any system that grants coins:
local petMult = data.ActivePetMultiplier or 1.0
local coins = math.floor(baseCoins * petMult)
CurrencySystem.add(player, "Coins", coins)
```

### 4. Pet visual (follows player)

```lua
-- Client-side: spawn pet model near player
-- Use BillboardGui for pet name above it
-- Tween position to follow HumanoidRootPart with slight lag

local function spawnPetVisual(petId: string)
    local config = PetConfig[petId]
    if not config or config.assetId == 0 then return end

    local success, asset = pcall(function()
        return game:GetService("InsertService"):LoadAsset(config.assetId)
    end)
    if not success then return end

    local model = asset:FindFirstChildOfClass("Model") or asset
    model.Parent = workspace

    -- Follow player loop
    local player = game:GetService("Players").LocalPlayer
    game:GetService("RunService").Heartbeat:Connect(function()
        local character = player.Character
        if not character then return end
        local hrp = character:FindFirstChild("HumanoidRootPart")
        if not hrp then return end

        local offset = Vector3.new(3, 0, 0)  -- to the right of player
        local target = hrp.Position + offset

        if model.PrimaryPart then
            model:SetPrimaryPartCFrame(
                model.PrimaryPart.CFrame:Lerp(CFrame.new(target), 0.1)
            )
        end
    end)
end
```

## For full egg hatching system → see `genres/pet-simulator.md`
