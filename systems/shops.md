# Shop System (Buy/Sell Logic)

Server-side shop logic. For UI, see `ui/shop.md`.

## Shop Types

- **Upgrade shop** — permanent stat upgrades (Speed, Value, Luck)
- **Item shop** — buy consumables or tools
- **Sell zone** — sell items from inventory for coins

## UpgradeSystem.lua (Server)

```lua
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local DataManager    = require(script.Parent.DataManager)
local CurrencySystem = require(script.Parent.CurrencySystem)
local UpgradeConfig  = require(ReplicatedStorage.Shared.Modules.UpgradeConfig)

local RE = ReplicatedStorage.Shared.RemoteEvents
local RE_PurchaseUpgrade  = RE.PurchaseUpgrade
local RE_UpgradeConfirmed = RE.UpgradeConfirmed

local UpgradeSystem = {}

function UpgradeSystem.init()
    RE_PurchaseUpgrade.OnServerEvent:Connect(function(player: Player, upgradeId: string)
        -- Validate type first, then existence
        if typeof(upgradeId) ~= "string" then return end

        local upgrade = UpgradeConfig[upgradeId]
        if not upgrade then return end

        local data = DataManager.getData(player)
        if not data then return end

        local currentLevel = (data.Upgrades and data.Upgrades[upgradeId]) or 0

        -- Check max level
        if currentLevel >= upgrade.maxLevel then
            RE_UpgradeConfirmed:FireClient(player, false, "MAX", upgradeId, currentLevel)
            return
        end

        -- Check cost
        local cost = upgrade.costs[currentLevel + 1]
        if not cost then return end

        -- Spend currency
        local success = CurrencySystem.spend(player, upgrade.currency, cost)
        if not success then
            RE_UpgradeConfirmed:FireClient(player, false, "BROKE", upgradeId, currentLevel)
            return
        end

        -- Apply upgrade
        data.Upgrades = data.Upgrades or {}
        data.Upgrades[upgradeId] = currentLevel + 1

        RE_UpgradeConfirmed:FireClient(player, true, "OK", upgradeId, currentLevel + 1)
    end)
end

-- Get upgrade level (used by other systems)
function UpgradeSystem.getLevel(player: Player, upgradeId: string): number
    local data = DataManager.getData(player)
    return data and data.Upgrades and data.Upgrades[upgradeId] or 0
end

return UpgradeSystem
```

## Sell Zone Pattern

```lua
-- Attach to a "SellZone" Part in workspace
-- When player touches it, sells all sellable items in inventory

local sellZone = workspace.SellZone
local debounce: {[Player]: boolean} = {}

sellZone.Touched:Connect(function(hit)
    local player = Players:GetPlayerFromCharacter(hit.Parent)
    if not player or debounce[player] then return end

    debounce[player] = true
    task.delay(1, function() debounce[player] = nil end)

    local data = DataManager.getData(player)
    if not data then return end

    local totalValue = 0
    for uid, item in pairs(data.Inventory or {}) do
        local config = ItemConfig[item.itemId]
        if config and config.sellValue then
            totalValue += config.sellValue * (item.quantity or 1)
            data.Inventory[uid] = nil
        end
    end

    if totalValue > 0 then
        CurrencySystem.add(player, "Coins", totalValue)
        -- Notify client
    end
end)
```

## RemoteEvents Needed

```
ReplicatedStorage/Shared/RemoteEvents/
  PurchaseUpgrade    ← client → server (upgradeId: string)
  UpgradeConfirmed   ← server → client (success, reason, upgradeId, newLevel)
  PurchaseItem       ← client → server (itemId: string)
  ItemPurchased      ← server → client (success, reason, itemId)
```
