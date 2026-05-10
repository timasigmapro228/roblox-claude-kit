# Orders System

Handles NPC order queues, pickup, processing, and return. Designed for active simulators (not tycoons).
Supports Normal and Hard order types out of the box.

## Concepts

- **Order**: a task given by an NPC. Has a type, difficulty, and cleaning steps.
- **OrderSlot**: a player can carry N orders at once (upgradeable).
- **Processing**: orders go into a machine, wait, produce a result with rarity roll.

## Config: OrderConfig.lua

```lua
-- ReplicatedStorage/Shared/Modules/OrderConfig.lua
local OrderConfig = {}

-- Item types (rename and extend for your game)
OrderConfig.Items = {
    ItemA = {
        id            = "ItemA",
        displayName   = "Item A",
        icon          = "rbxassetid://0",   -- replace with asset ID
        baseCoins     = 10,
        baseRep       = 5,
        difficulty    = 1,
        cleaningSteps = 1,                  -- Hard: set to 2
        orderType     = "Normal",           -- "Normal" | "Hard"
        rarityWeights = { Common=70, Rare=25, Epic=5 },
    },
    ItemB = {
        id            = "ItemB",
        displayName   = "Item B",
        icon          = "rbxassetid://0",
        baseCoins     = 12,
        baseRep       = 6,
        difficulty    = 1,
        cleaningSteps = 1,
        orderType     = "Normal",
        rarityWeights = { Common=65, Rare=28, Epic=7 },
    },
    ItemC = {
        id            = "ItemC",
        displayName   = "Item C",
        icon          = "rbxassetid://0",
        baseCoins     = 15,
        baseRep       = 8,
        difficulty    = 1,
        cleaningSteps = 1,
        orderType     = "Normal",
        rarityWeights = { Common=60, Rare=30, Epic=10 },
    },
    -- NIGHTMARE (Day 8-14 feature) — already configured, just not spawned yet
    --[[
    HardItemA = {
        id            = "HardItemA",
        displayName   = "Hard Item A",
        icon          = "rbxassetid://0",
        baseCoins     = 40,
        baseRep       = 20,
        difficulty    = 2,
        cleaningSteps = 2,
        orderType     = "Hard",
        rarityWeights = { Common=30, Rare=45, Epic=25 },
    },
    ]]
}

-- Cleaned result names per rarity
OrderConfig.Results = {
    Common = { id="CleanDream",    displayName="Clean Dream",    icon="rbxassetid://0" },
    Rare   = { id="SparklyDream",  displayName="Sparkly Dream",  icon="rbxassetid://0" },
    Epic   = { id="PerfectDream",  displayName="Perfect Dream",  icon="rbxassetid://0" },
}

-- Washer processing time (seconds) per difficulty, modified by Speed upgrade
OrderConfig.BaseWashTime = {
    [1] = 8,   -- Normal
    [2] = 16,  -- Hard (2 steps × 8s each handled by OrderSystem)
}

return OrderConfig
```

## Server: OrderSystem.lua

```lua
-- ServerScriptService/Server/Systems/OrderSystem.lua
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local OrderConfig    = require(ReplicatedStorage.Shared.Modules.OrderConfig)
local CurrencySystem = require(script.Parent.CurrencySystem)
local DataManager    = require(script.Parent.DataManager)

local RemoteEvents = ReplicatedStorage.Shared.RemoteEvents
local RE_PickupOrder   = RemoteEvents.PickupOrder
local RE_StartWash     = RemoteEvents.StartWash
local RE_CollectResult = RemoteEvents.CollectResult
local RE_OrderUpdate   = RemoteEvents.OrderUpdate   -- server → client UI sync

local OrderSystem = {}

-- Active orders per player: { [player] = { orderId = { dreamId, step, result } } }
local playerOrders: {[Player]: {[string]: any}} = {}

local function rollRarity(weights: {[string]: number}): string
    local total = 0
    for _, w in pairs(weights) do total += w end
    local roll = math.random(1, total)
    local cumulative = 0
    for rarity, w in pairs(weights) do
        cumulative += w
        if roll <= cumulative then return rarity end
    end
    return "Common"
end

local function getLuckMultiplier(player: Player): number
    local data = DataManager.getData(player)
    local luckLevel = data and data.Upgrades and data.Upgrades.Luck or 0
    -- Each luck level adds +5% epic chance (example scaling)
    return 1 + (luckLevel * 0.05)
end

local function getWashTime(player: Player, difficulty: number): number
    local data = DataManager.getData(player)
    local speedLevel = data and data.Upgrades and data.Upgrades.Speed or 0
    local base = OrderConfig.BaseWashTime[difficulty] or 8
    -- Each speed level reduces time by 10%, min 2s
    return math.max(2, base * (1 - speedLevel * 0.1))
end

local function getMaxSlots(player: Player): number
    local data = DataManager.getData(player)
    local slotLevel = data and data.Upgrades and data.Upgrades.ExtraSlot or 0
    return 1 + slotLevel  -- base 1 slot, upgradeable
end

function OrderSystem.init()
    Players.PlayerAdded:Connect(function(player)
        playerOrders[player] = {}
    end)

    Players.PlayerRemoving:Connect(function(player)
        playerOrders[player] = nil
    end)

    -- Client picks up an order from NPC
    -- Client picks up order from NPC — delegates to giveOrder for single code path
    RE_PickupOrder.OnServerEvent:Connect(function(player: Player, dreamId: string)
        if typeof(dreamId) ~= "string" then return end
        OrderSystem.giveOrder(player, dreamId)
    end)

    -- Client puts order into washer
    RE_StartWash.OnServerEvent:Connect(function(player: Player, orderId: string)
        if typeof(orderId) ~= "string" then return end

        local orders = playerOrders[player]
        if not orders or not orders[orderId] then return end

        local order = orders[orderId]
        if order.step ~= 0 then return end  -- already washing

        local dream = OrderConfig.Items[order.dreamId]
        if not dream then return end

        order.step = 1
        RE_OrderUpdate:FireClient(player, "Washing", orderId, {
            washTime = getWashTime(player, dream.difficulty)
        })

        -- Async: wait for wash, then produce result
        task.spawn(function()
            local washTime = getWashTime(player, dream.difficulty)
            task.wait(washTime)

            -- Check player still connected and order still valid
            if not playerOrders[player] or not playerOrders[player][orderId] then return end

            -- Roll rarity (apply luck)
            local weights = {}
            for rarity, w in pairs(dream.rarityWeights) do
                weights[rarity] = rarity == "Epic"
                    and math.floor(w * getLuckMultiplier(player))
                    or w
            end
            local rarity = rollRarity(weights)
            local result = OrderConfig.Results[rarity]

            order.step     = 2
            order.result   = rarity
            order.complete = true

            RE_OrderUpdate:FireClient(player, "Done", orderId, {
                rarity = rarity,
                result = result,
            })
        end)
    end)

    -- Client returns completed order to NPC
    RE_CollectResult.OnServerEvent:Connect(function(player: Player, orderId: string)
        if typeof(orderId) ~= "string" then return end
        local orders = playerOrders[player]
        if not orders or not orders[orderId] then return end

        local order = orders[orderId]
        if not order.complete then return end

        local dream  = OrderConfig.Items[order.dreamId]
        local rarity = order.result
        if not dream or not rarity then return end

        -- Calculate rewards (apply value upgrade)
        local data = DataManager.getData(player)
        local valueLevel = data and data.Upgrades and data.Upgrades.Value or 0
        local coinMultiplier = 1 + (valueLevel * 0.2)  -- +20% per level
        local coins = math.floor(dream.baseCoins * coinMultiplier)
        local rep   = dream.baseRep

        -- Grant rewards
        CurrencySystem.add(player, "Coins", coins)

        -- Update reputation
        if data then
            data.Reputation = (data.Reputation or 0) + rep
        end

        -- Add to Dream Index (collection)
        if data and data.DreamIndex then
            data.DreamIndex[order.dreamId] = true
            data.DreamIndex[rarity .. "_" .. order.dreamId] = true
        end

        -- Remove order
        orders[orderId] = nil

        RE_OrderUpdate:FireClient(player, "Collected", orderId, {
            coins = coins,
            rep   = rep,
            rarity = rarity,
        })
    end)
end

-- Public API: give an order to a player (called by NPCSystem server-side)
-- Also used by PickupOrder RemoteEvent — single code path for both
function OrderSystem.giveOrder(player: Player, dreamId: string): string?
    if typeof(dreamId) ~= "string" then return nil end

    local dream = OrderConfig.Items[dreamId]
    if not dream then return nil end

    local orders = playerOrders[player]
    if not orders then return nil end

    -- Check slot capacity
    local count = 0
    for _ in pairs(orders) do count += 1 end
    if count >= getMaxSlots(player) then return nil end

    local orderId = tostring(math.random(100000, 999999))
    orders[orderId] = {
        dreamId  = dreamId,   -- consistent field name used by StartWash/CollectResult
        step     = 0,
        result   = nil,
        complete = false,
    }

    RE_OrderUpdate:FireClient(player, "Added", orderId, dream)
    return orderId
end

return OrderSystem
```

## RemoteEvents Needed

Create these instances in `ReplicatedStorage/Shared/RemoteEvents/`:
- `PickupOrder` (RemoteEvent)
- `StartWash` (RemoteEvent)
- `CollectResult` (RemoteEvent)
- `OrderUpdate` (RemoteEvent)

## Default Player Data (Orders section)

```lua
local DEFAULT_DATA = {
    Coins      = 0,
    Gems       = 0,
    Reputation = 0,
    Upgrades   = {
        Speed     = 0,
        Value     = 0,
        Luck      = 0,
        ExtraSlot = 0,
    },
    DreamIndex = {},  -- tracks discovered dreams and rarities
}
```
