---
name: genre-simulator
description: Use when building a Roblox simulator game — clicking/collecting coins, upgrades, rebirth, pets, auto-collect zones. Triggers on words like simulator, clicking, collecting, farming, auto, rebirth, multiplier.
---

# Genre: Simulator

**Core loop**: Player clicks/collects → earns coins → buys upgrades → earns faster → rebirth → repeat

**Examples**: Pet Simulator 99, Mining Simulator, Coin Collecting Simulator, Throwing Simulator

---

## Folder Structure

```
ServerScriptService/Server/
  Systems/
    DataManager.lua
    CurrencySystem.lua
    ClickSystem.lua       ← handles clicking/collecting
    UpgradeSystem.lua
    RebirthSystem.lua
    PetSystem.lua         ← optional

ReplicatedStorage/Shared/
  Modules/
    SimConfig.lua         ← all game values in one place
    UpgradeConfig.lua
    PetConfig.lua
  RemoteEvents/
    Click                 ← client → server
    AutoCollect           ← server → client tick
    RebirthRequest        ← client → server
    RebirthConfirm        ← server → client
```

---

## SimConfig.lua

```lua
-- ReplicatedStorage/Shared/Modules/SimConfig.lua
return {
    -- Base values
    baseClickValue    = 1,      -- coins per click at level 0
    baseAutoValue     = 0,      -- coins per second from auto-collect
    baseClickCooldown = 0,      -- seconds between clicks (0 = unlimited)

    -- Rebirth
    rebirthRequirement = 1000,  -- coins needed to rebirth
    rebirthMultiplier  = 1.5,   -- each rebirth multiplies all earnings

    -- Zone unlocks (cost in coins to enter)
    zones = {
        { name = "Starter Zone",  cost = 0,     multiplier = 1   },
        { name = "Forest Zone",   cost = 500,   multiplier = 2   },
        { name = "Lava Zone",     cost = 5000,  multiplier = 5   },
        { name = "Space Zone",    cost = 50000, multiplier = 15  },
    },
}
```

---

## ClickSystem.lua (Server)

```lua
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local DataManager    = require(script.Parent.DataManager)
local CurrencySystem = require(script.Parent.CurrencySystem)
local SimConfig      = require(ReplicatedStorage.Shared.Modules.SimConfig)
local UpgradeConfig  = require(ReplicatedStorage.Shared.Modules.UpgradeConfig)

local RE_Click = ReplicatedStorage.Shared.RemoteEvents.Click

local ClickSystem = {}

-- Rate limiting: max clicks per second per player
local CLICK_LIMIT = 20
local clickTimestamps: {[Player]: {number}} = {}

local function isClickRateLimited(player: Player): boolean
    local now = tick()
    if not clickTimestamps[player] then clickTimestamps[player] = {} end
    local timestamps = clickTimestamps[player]

    -- Remove timestamps older than 1 second
    for i = #timestamps, 1, -1 do
        if now - timestamps[i] > 1 then
            table.remove(timestamps, i)
        end
    end

    if #timestamps >= CLICK_LIMIT then return true end
    table.insert(timestamps, now)
    return false
end

local function getClickValue(player: Player): number
    local data = DataManager.getData(player)
    if not data then return 0 end

    local base = SimConfig.baseClickValue
    local upgradeLevel = data.Upgrades and data.Upgrades.ClickPower or 0
    local rebirths = data.Rebirths or 0

    -- Apply upgrade multiplier
    local upgradeMult = 1 + (upgradeLevel * 0.5)  -- +50% per level
    -- Apply rebirth multiplier
    local rebirthMult = SimConfig.rebirthMultiplier ^ rebirths
    -- Apply zone multiplier
    local zone = data.CurrentZone or 1
    local zoneMult = SimConfig.zones[zone] and SimConfig.zones[zone].multiplier or 1
    -- Apply pet multiplier (if pet system active)
    local petMult = data.ActivePetMultiplier or 1

    return math.floor(base * upgradeMult * rebirthMult * zoneMult * petMult)
end

function ClickSystem.init()
    Players.PlayerAdded:Connect(function(player)
        clickTimestamps[player] = {}
    end)
    Players.PlayerRemoving:Connect(function(player)
        clickTimestamps[player] = nil
    end)

    RE_Click.OnServerEvent:Connect(function(player: Player)
        if isClickRateLimited(player) then return end
        local value = getClickValue(player)
        if value <= 0 then return end
        CurrencySystem.add(player, "Coins", value)
    end)
end

return ClickSystem
```

---

## RebirthSystem.lua (Server)

```lua
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local DataManager    = require(script.Parent.DataManager)
local CurrencySystem = require(script.Parent.CurrencySystem)
local SimConfig      = require(ReplicatedStorage.Shared.Modules.SimConfig)

local RE_RebirthRequest = ReplicatedStorage.Shared.RemoteEvents.RebirthRequest
local RE_RebirthConfirm = ReplicatedStorage.Shared.RemoteEvents.RebirthConfirm

local RebirthSystem = {}

function RebirthSystem.init()
    RE_RebirthRequest.OnServerEvent:Connect(function(player: Player)
        local data = DataManager.getData(player)
        if not data then return end

        local coins = CurrencySystem.getBalance(player, "Coins")
        local required = SimConfig.rebirthRequirement

        -- Scale requirement per rebirth
        local currentRebirths = data.Rebirths or 0
        local actualRequired = required * (2 ^ currentRebirths)

        if coins < actualRequired then
            RE_RebirthConfirm:FireClient(player, false,
                "Need " .. actualRequired .. " coins to rebirth!")
            return
        end

        -- Reset coins, keep gems and pets
        data.Coins    = 0
        data.Rebirths = currentRebirths + 1
        data.Upgrades = {}  -- reset upgrades on rebirth

        -- Update client balance
        local UpdateCurrency = ReplicatedStorage.Shared.RemoteEvents.UpdateCurrency
        UpdateCurrency:FireClient(player, "Coins", 0)

        RE_RebirthConfirm:FireClient(player, true, data.Rebirths)
    end)
end

return RebirthSystem
```

---

## Auto-Collect Loop (Server)

```lua
-- In ClickSystem or a separate AutoSystem:
local RunService = game:GetService("RunService")

local AUTO_INTERVAL = 1  -- seconds between auto-collect ticks

task.spawn(function()
    while true do
        task.wait(AUTO_INTERVAL)
        for _, player in ipairs(Players:GetPlayers()) do
            local data = DataManager.getData(player)
            if data then
                local autoLevel = data.Upgrades and data.Upgrades.AutoCollect or 0
                if autoLevel > 0 then
                    local value = SimConfig.baseAutoValue * autoLevel
                    local rebirthMult = SimConfig.rebirthMultiplier ^ (data.Rebirths or 0)
                    local payout = math.floor(value * rebirthMult)
                    if payout > 0 then
                        CurrencySystem.add(player, "Coins", payout)
                    end
                end
            end
        end
    end
end)
```

---

## UpgradeConfig.lua (Simulator version)

```lua
return {
    ClickPower = {
        id          = "ClickPower",
        displayName = "Click Power",
        description = "+50% coins per click",
        icon        = "rbxassetid://0",
        currency    = "Coins",
        maxLevel    = 10,
        costs       = { 10, 25, 60, 140, 320, 720, 1600, 3500, 8000, 18000 },
    },
    AutoCollect = {
        id          = "AutoCollect",
        displayName = "Auto Collect",
        description = "Earn coins automatically per second",
        icon        = "rbxassetid://0",
        currency    = "Coins",
        maxLevel    = 5,
        costs       = { 50, 200, 800, 3000, 12000 },
    },
    Luck = {
        id          = "Luck",
        displayName = "Lucky Drops",
        description = "+20% chance of rare items",
        icon        = "rbxassetid://0",
        currency    = "Gems",
        maxLevel    = 5,
        costs       = { 5, 15, 40, 100, 250 },
    },
}
```

---

## Default Player Data

```lua
local DEFAULT_DATA = {
    Coins    = 0,
    Gems     = 0,
    Rebirths = 0,
    CurrentZone = 1,
    ActivePetMultiplier = 1,
    Upgrades = {},
    OwnedPets = {},
    EquippedPets = {},
    DailyReward = { lastClaim = 0, streak = 0 },
    CollectionIndex = {},
    ProcessedReceipts = {},
}
```

---

## Simulator UI Checklist

- [ ] Coins counter (top center, large and bold)
- [ ] Click button or clickable object in world
- [ ] "Per second" auto-earn display
- [ ] Rebirth button with requirement meter
- [ ] Upgrade shop (bottom or side panel)
- [ ] Zone selector / teleporter pads
- [ ] Pet display (equipped pets follow player)
- [ ] Leaderboard (top players by coins/rebirths)

---

## Simulator Map Layout

```
[Spawn Area]
  - Click object in center (glowing orb, chest, item)
  - Upgrade shop NPC nearby
  - Zone teleporter pads around perimeter

[Zone 1 - Starter]  [Zone 2 - Forest]  [Zone 3+]
  Each zone: separate area, themed visuals,
  click objects give zone multiplier,
  unlock cost paid once (saved in data)
```
