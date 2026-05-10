# Currency System

Handles multiple currencies (coins, gems, etc.) with server authority and client display.

## Folder Setup

```
ReplicatedStorage/Shared/RemoteEvents/
  UpdateCurrency   ← server → client: fires when balance changes
  RequestCurrency  ← client → server: used only for UI refresh (no grants from client)
```

## Server: CurrencySystem.lua

```lua
-- ServerScriptService/Server/Systems/CurrencySystem.lua
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local DataManager = require(script.Parent.DataManager)

local RemoteEvents = ReplicatedStorage.Shared.RemoteEvents
local UpdateCurrency = RemoteEvents.UpdateCurrency

local CurrencySystem = {}

-- Currency types supported
local CURRENCIES = { "Coins", "Gems", "Cash" }

function CurrencySystem.init()
    Players.PlayerAdded:Connect(function(player)
        -- Send initial balances to client once data loads
        DataManager.onLoaded(player, function(data)
            for _, currency in ipairs(CURRENCIES) do
                UpdateCurrency:FireClient(player, currency, data[currency] or 0)
            end
        end)
    end)
end

-- Add currency (server only, never call from client)
function CurrencySystem.add(player: Player, currency: string, amount: number): boolean
    assert(table.find(CURRENCIES, currency), "Unknown currency: " .. currency)
    assert(typeof(amount) == "number" and amount > 0, "Amount must be positive number")

    local data = DataManager.getData(player)
    if not data then return false end

    data[currency] = (data[currency] or 0) + amount
    UpdateCurrency:FireClient(player, currency, data[currency])
    return true
end

-- Spend currency — returns false if insufficient
function CurrencySystem.spend(player: Player, currency: string, amount: number): boolean
    assert(table.find(CURRENCIES, currency), "Unknown currency: " .. currency)
    assert(typeof(amount) == "number" and amount > 0, "Amount must be positive number")

    local data = DataManager.getData(player)
    if not data then return false end

    local balance = data[currency] or 0
    if balance < amount then return false end

    data[currency] = balance - amount
    UpdateCurrency:FireClient(player, currency, data[currency])
    return true
end

-- Get balance (server)
function CurrencySystem.getBalance(player: Player, currency: string): number
    local data = DataManager.getData(player)
    return data and (data[currency] or 0) or 0
end

return CurrencySystem
```

## Client: CurrencyController.lua

```lua
-- StarterPlayerScripts/Client/Systems/CurrencyController.lua
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local UpdateCurrency = ReplicatedStorage.Shared.RemoteEvents.UpdateCurrency

local CurrencyController = {}

-- Local cache for display
local balances: {[string]: number} = {}
local listeners: {[string]: {(number) -> ()}} = {}

function CurrencyController.init()
    UpdateCurrency.OnClientEvent:Connect(function(currency: string, amount: number)
        balances[currency] = amount
        -- Notify any registered UI listeners
        if listeners[currency] then
            for _, callback in ipairs(listeners[currency]) do
                callback(amount)
            end
        end
    end)
end

-- Get cached balance (display only — not authoritative)
function CurrencyController.getBalance(currency: string): number
    return balances[currency] or 0
end

-- Register a UI callback when currency updates
function CurrencyController.onUpdate(currency: string, callback: (number) -> ())
    if not listeners[currency] then
        listeners[currency] = {}
    end
    table.insert(listeners[currency], callback)
end

return CurrencyController
```

## HUD Wire-up Example

```lua
-- In HUDController.init():
CurrencyController.onUpdate("Coins", function(amount)
    CoinsLabel.Text = "🪙 " .. tostring(amount)
end)

CurrencyController.onUpdate("Gems", function(amount)
    GemsLabel.Text = "💎 " .. tostring(amount)
end)
```

## Default Player Data

```lua
-- In DataManager default template:
local DEFAULT_DATA = {
    Coins = 0,
    Gems  = 0,
}
```
