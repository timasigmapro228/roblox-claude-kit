---
name: genre-pet-simulator
description: Use when building a Roblox pet simulator — eggs, hatching, rarity, pet equipping, multipliers, pet index. Triggers on words like pet, egg, hatch, rarity, common/rare/epic/legendary, equip pet, pet multiplier, shiny.
---

# Genre: Pet Simulator

**Core loop**: Hatch eggs → get pets with rarity → equip pets → earn coin multiplier → buy better eggs → discover rare pets

**Examples**: Pet Simulator 99, Pet Simulator X, Adopt Me, Collect All Pets

---

## Folder Structure

```
ReplicatedStorage/Shared/
  Modules/
    EggConfig.lua     ← egg types and their pet pools
    PetConfig.lua     ← all pets with rarity weights
  RemoteEvents/
    HatchEgg          ← client → server
    HatchResult       ← server → client
    EquipPet          ← client → server
    UnequipPet        ← client → server
    PetUpdate         ← server → client (sync equipped pets)

ServerScriptService/Server/Systems/
  PetSystem.lua
```

---

## PetConfig.lua

```lua
-- ReplicatedStorage/Shared/Modules/PetConfig.lua
local PetConfig = {}

-- Rarity tiers
PetConfig.Rarities = {
    Common    = { weight = 600, color = Color3.fromRGB(180,180,180), multiplier = 1.0  },
    Uncommon  = { weight = 250, color = Color3.fromRGB(100,220,100), multiplier = 1.5  },
    Rare      = { weight = 100, color = Color3.fromRGB(80,120,255),  multiplier = 2.5  },
    Epic      = { weight = 40,  color = Color3.fromRGB(160,80,255),  multiplier = 5.0  },
    Legendary = { weight = 9,   color = Color3.fromRGB(255,200,0),   multiplier = 15.0 },
    Mythical  = { weight = 1,   color = Color3.fromRGB(255,80,80),   multiplier = 50.0 },
}

-- All pets in the game
PetConfig.Pets = {
    BasicCat = {
        id          = "BasicCat",
        displayName = "Basic Cat",
        rarity      = "Common",
        assetId     = 0,    -- Toolbox model ID
        multiplier  = 1.0,
    },
    ForestFox = {
        id          = "ForestFox",
        displayName = "Forest Fox",
        rarity      = "Uncommon",
        assetId     = 0,
        multiplier  = 1.5,
    },
    StormDragon = {
        id          = "StormDragon",
        displayName = "Storm Dragon",
        rarity      = "Rare",
        assetId     = 0,
        multiplier  = 2.5,
    },
    VoidPhoenix = {
        id          = "VoidPhoenix",
        displayName = "Void Phoenix",
        rarity      = "Epic",
        assetId     = 0,
        multiplier  = 5.0,
    },
    CosmicTiger = {
        id          = "CosmicTiger",
        displayName = "Cosmic Tiger",
        rarity      = "Legendary",
        assetId     = 0,
        multiplier  = 15.0,
    },
    DreamGod = {
        id          = "DreamGod",
        displayName = "Dream God",
        rarity      = "Mythical",
        assetId     = 0,
        multiplier  = 50.0,
    },
}

return PetConfig
```

---

## EggConfig.lua

```lua
-- ReplicatedStorage/Shared/Modules/EggConfig.lua
return {
    StarterEgg = {
        id          = "StarterEgg",
        displayName = "Starter Egg",
        cost        = 100,
        currency    = "Coins",
        assetId     = 0,
        -- Pet pool: which pets can hatch from this egg
        -- Weights are relative (higher = more likely)
        pets = {
            BasicCat   = 600,
            ForestFox  = 250,
            StormDragon = 100,
            VoidPhoenix = 40,
            CosmicTiger = 9,
            DreamGod    = 1,
        },
    },
    ForestEgg = {
        id          = "ForestEgg",
        displayName = "Forest Egg",
        cost        = 1000,
        currency    = "Coins",
        assetId     = 0,
        pets = {
            ForestFox   = 500,
            StormDragon = 300,
            VoidPhoenix = 150,
            CosmicTiger = 45,
            DreamGod    = 5,
        },
    },
    GemEgg = {
        id          = "GemEgg",
        displayName = "Gem Egg",
        cost        = 10,
        currency    = "Gems",
        assetId     = 0,
        pets = {
            StormDragon  = 400,
            VoidPhoenix  = 300,
            CosmicTiger  = 250,
            DreamGod     = 50,
        },
    },
}
```

---

## PetSystem.lua (Server)

```lua
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local DataManager    = require(script.Parent.DataManager)
local CurrencySystem = require(script.Parent.CurrencySystem)
local PetConfig      = require(ReplicatedStorage.Shared.Modules.PetConfig)
local EggConfig      = require(ReplicatedStorage.Shared.Modules.EggConfig)

local RE = ReplicatedStorage.Shared.RemoteEvents
local RE_HatchEgg   = RE.HatchEgg
local RE_HatchResult = RE.HatchResult
local RE_EquipPet   = RE.EquipPet
local RE_UnequipPet = RE.UnequipPet
local RE_PetUpdate  = RE.PetUpdate

local PetSystem = {}

local MAX_EQUIPPED = 3  -- max pets equipped at once

local function rollPet(eggId: string, luckMultiplier: number): string?
    local egg = EggConfig[eggId]
    if not egg then return nil end

    -- Build weighted pool
    local pool = {}
    local total = 0
    for petId, weight in pairs(egg.pets) do
        -- Apply luck to legendary+ pets
        local pet = PetConfig.Pets[petId]
        local adjustedWeight = weight
        if pet and (pet.rarity == "Legendary" or pet.rarity == "Mythical") then
            adjustedWeight = math.floor(weight * luckMultiplier)
        end
        table.insert(pool, { petId = petId, weight = adjustedWeight })
        total += adjustedWeight
    end

    -- Roll
    local roll = math.random(1, total)
    local cumulative = 0
    for _, entry in ipairs(pool) do
        cumulative += entry.weight
        if roll <= cumulative then
            return entry.petId
        end
    end
    return pool[1].petId
end

local function getLuckMultiplier(player: Player): number
    local data = DataManager.getData(player)
    local luckLevel = data and data.Upgrades and data.Upgrades.Luck or 0
    return 1 + (luckLevel * 0.1)  -- +10% per luck level
end

local function updateEquippedMultiplier(player: Player)
    local data = DataManager.getData(player)
    if not data then return end

    local equipped  = data.EquippedPets or {}
    local ownedPets = data.OwnedPets or {}
    local totalMult = 1.0

    for _, uid in ipairs(equipped) do
        -- EquippedPets stores UIDs, look up petId through OwnedPets
        local ownedPet = ownedPets[uid]
        local pet = ownedPet and PetConfig.Pets[ownedPet.petId]
        if pet then
            totalMult += pet.multiplier - 1  -- additive bonus
        end
    end

    data.ActivePetMultiplier = totalMult
    RE_PetUpdate:FireClient(player, equipped, totalMult)
end

local function generatePetUID(): string
    return tostring(math.random(100000000, 999999999))
end

function PetSystem.init()
    RE_HatchEgg.OnServerEvent:Connect(function(player: Player, eggId: string)
        if typeof(eggId) ~= "string" then return end

        local egg = EggConfig[eggId]
        if not egg then return end

        -- Validate and spend currency
        local success = CurrencySystem.spend(player, egg.currency, egg.cost)
        if not success then
            RE_HatchResult:FireClient(player, false, "Not enough " .. egg.currency)
            return
        end

        -- Roll pet
        local luck = getLuckMultiplier(player)
        local petId = rollPet(eggId, luck)
        if not petId then return end

        -- Add to inventory
        local data = DataManager.getData(player)
        if not data then return end

        local uid = generatePetUID()
        data.OwnedPets = data.OwnedPets or {}
        data.OwnedPets[uid] = { petId = petId, equipped = false }

        -- Track in collection index
        data.CollectionIndex = data.CollectionIndex or {}
        data.CollectionIndex[petId] = true

        local petData = PetConfig.Pets[petId]
        RE_HatchResult:FireClient(player, true, {
            uid    = uid,
            petId  = petId,
            rarity = petData and petData.rarity or "Common",
            name   = petData and petData.displayName or petId,
        })
    end)

    RE_EquipPet.OnServerEvent:Connect(function(player: Player, uid: string)
        if typeof(uid) ~= "string" then return end

        local data = DataManager.getData(player)
        if not data or not data.OwnedPets then return end
        if not data.OwnedPets[uid] then return end

        data.EquippedPets = data.EquippedPets or {}

        -- Prevent equipping the same pet twice
        if table.find(data.EquippedPets, uid) then return end

        -- Max equipped check
        if #data.EquippedPets >= MAX_EQUIPPED then
            -- Auto-unequip oldest
            local oldest = table.remove(data.EquippedPets, 1)
            if data.OwnedPets[oldest] then
                data.OwnedPets[oldest].equipped = false
            end
        end

        table.insert(data.EquippedPets, uid)
        data.OwnedPets[uid].equipped = true
        updateEquippedMultiplier(player)
    end)

    RE_UnequipPet.OnServerEvent:Connect(function(player: Player, uid: string)
        if typeof(uid) ~= "string" then return end

        local data = DataManager.getData(player)
        if not data then return end

        data.EquippedPets = data.EquippedPets or {}
        for i, equippedUID in ipairs(data.EquippedPets) do
            if equippedUID == uid then
                table.remove(data.EquippedPets, i)
                break
            end
        end

        if data.OwnedPets and data.OwnedPets[uid] then
            data.OwnedPets[uid].equipped = false
        end

        updateEquippedMultiplier(player)
    end)
end

return PetSystem
```

---

## Default Player Data

```lua
local DEFAULT_DATA = {
    Coins    = 0,
    Gems     = 0,
    Rebirths = 0,
    Upgrades = {},
    OwnedPets    = {},   -- { uid = { petId, equipped } }
    EquippedPets = {},   -- { uid, uid, uid } max 3
    ActivePetMultiplier = 1.0,
    CollectionIndex = {},
    ProcessedReceipts = {},
    DailyReward = { lastClaim = 0, streak = 0 },
}
```

---

## Hatch Animation (Client)

```lua
-- In ObbyController or PetController client-side:
RE_HatchResult.OnClientEvent:Connect(function(success, result)
    if not success then
        NotificationController.error(result)
        return
    end

    -- Show egg shake animation
    -- Show rarity flash
    -- Show pet name popup

    local rarityColors = {
        Common    = Color3.fromRGB(180,180,180),
        Uncommon  = Color3.fromRGB(100,220,100),
        Rare      = Color3.fromRGB(80,120,255),
        Epic      = Color3.fromRGB(160,80,255),
        Legendary = Color3.fromRGB(255,200,0),
        Mythical  = Color3.fromRGB(255,80,80),
    }

    -- Flash screen with rarity color
    -- Show "YOU HATCHED A [RARITY] [NAME]!" popup
    -- Play sound effect matching rarity
end)
```

---

## Pet UI Checklist

- [ ] Inventory grid (show all owned pets with rarity border color)
- [ ] Equip/Unequip buttons
- [ ] "Equip Best" auto-equip button
- [ ] Pet index / collection book
- [ ] Active multiplier display ("x3.5 Coins" in HUD)
- [ ] Egg hatching animation
- [ ] Rarity flash on hatch
- [ ] "NEW PET!" notification for first discovery

---

## Sound Design for Rarity

| Rarity | Sound style |
|---|---|
| Common | Soft pop |
| Uncommon | Light chime |
| Rare | Sparkle sound |
| Epic | Magic burst |
| Legendary | Gold fanfare |
| Mythical | Full dramatic reveal with music sting |
