# Inventory System

Grid-based inventory for items, pets, tools, or collectibles.

## Structure

```
ReplicatedStorage/Shared/RemoteEvents/
  OpenInventory    ← client → server (request inventory data)
  InventoryData    ← server → client (sends full inventory)
  EquipItem        ← client → server
  UnequipItem      ← client → server
  DropItem         ← client → server (if dropping is allowed)
```

## Default Player Data

```lua
local DEFAULT_DATA = {
    Inventory = {},   -- { uid = { itemId, quantity, equipped } }
    MaxSlots  = 50,   -- upgradeable
}
```

## InventorySystem.lua (Server)

```lua
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local DataManager = require(script.Parent.DataManager)

local RE = ReplicatedStorage.Shared.RemoteEvents
local RE_OpenInventory = RE.OpenInventory
local RE_InventoryData = RE.InventoryData
local RE_EquipItem     = RE.EquipItem

local InventorySystem = {}

local function generateUID(): string
    return game:GetService("HttpService"):GenerateGUID(false):sub(1, 8)
end

-- Add item to player inventory
function InventorySystem.addItem(player: Player, itemId: string, quantity: number?): string?
    local data = DataManager.getData(player)
    if not data then return nil end

    quantity = quantity or 1
    local count = 0
    for _ in pairs(data.Inventory) do count += 1 end
    if count >= (data.MaxSlots or 50) then
        return nil  -- inventory full
    end

    local uid = generateUID()
    data.Inventory[uid] = {
        itemId   = itemId,
        quantity = quantity,
        equipped = false,
    }

    -- Sync to client
    RE_InventoryData:FireClient(player, data.Inventory)
    return uid
end

-- Remove item
function InventorySystem.removeItem(player: Player, uid: string): boolean
    local data = DataManager.getData(player)
    if not data or not data.Inventory[uid] then return false end
    data.Inventory[uid] = nil
    RE_InventoryData:FireClient(player, data.Inventory)
    return true
end

function InventorySystem.init()
    -- Send inventory on request
    RE_OpenInventory.OnServerEvent:Connect(function(player: Player)
        local data = DataManager.getData(player)
        if not data then return end
        RE_InventoryData:FireClient(player, data.Inventory)
    end)

    -- Equip item
    RE_EquipItem.OnServerEvent:Connect(function(player: Player, uid: string)
        if typeof(uid) ~= "string" then return end
        local data = DataManager.getData(player)
        if not data or not data.Inventory[uid] then return end
        data.Inventory[uid].equipped = true
        RE_InventoryData:FireClient(player, data.Inventory)
    end)
end

return InventorySystem
```

## Inventory UI (Client)

Grid layout using UIGridLayout inside ScrollingFrame.
Each cell: ImageLabel (icon) + TextLabel (quantity) + selection border.

```lua
-- InventoryController.lua (Client)
-- Opens inventory grid when player clicks inventory button
-- Each slot shows item icon from ItemConfig
-- Click slot = select, double-click = equip
-- Right-click or long-press (mobile) = context menu (equip/drop)
```

## Item Config Pattern

```lua
-- ReplicatedStorage/Shared/Modules/ItemConfig.lua
return {
    WoodLog = {
        id          = "WoodLog",
        displayName = "Wood Log",
        icon        = "rbxassetid://0",
        stackable   = true,
        maxStack    = 99,
        category    = "Material",
    },
    IronSword = {
        id          = "IronSword",
        displayName = "Iron Sword",
        icon        = "rbxassetid://0",
        stackable   = false,
        maxStack    = 1,
        category    = "Weapon",
        equippable  = true,
    },
}
```
