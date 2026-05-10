# ModuleScript Architecture

Every system is a ModuleScript. Scripts only bootstrap. Nothing else.

## The Pattern

```lua
-- Every system follows this structure
local MySystem = {}

-- Private state
local _initialized = false

-- Dependencies (required at top, not inside functions)
local DataManager = require(script.Parent.DataManager)

function MySystem.init()
    assert(not _initialized, "MySystem already initialized")
    _initialized = true

    -- Setup events, connections, loops here
end

-- Public API
function MySystem.doSomething(player: Player, arg: string): boolean
    -- ...
    return true
end

return MySystem
```

## Folder Structure

```
ServerScriptService/
  Server/
    Main.server.lua         ← ONLY file that runs logic; requires systems
    Systems/
      DataManager.lua
      CurrencySystem.lua
      OrderSystem.lua
      UpgradeSystem.lua
      NPCSystem.lua

StarterPlayerScripts/
  Client/
    Main.client.lua
    Systems/
      HUDController.lua
      OrderController.lua
      ShopController.lua

ReplicatedStorage/
  Shared/
    Modules/
      OrderConfig.lua       ← shared config, readable by both sides
      UpgradeConfig.lua
      RarityUtil.lua
    RemoteEvents/           ← RemoteEvent instances (not scripts)
    RemoteFunctions/
```

## Dependency Rules

```
DataManager        ← no dependencies (loads first)
CurrencySystem     ← depends on DataManager
OrderSystem        ← depends on DataManager, CurrencySystem
UpgradeSystem      ← depends on DataManager, CurrencySystem
NPCSystem          ← depends on OrderSystem
```

Init order in Main.server.lua must match this dependency chain.

## Shared Config Pattern

Config files live in ReplicatedStorage so both server and client can read them.
They return static tables — no logic, no state.

```lua
-- ReplicatedStorage/Shared/Modules/UpgradeConfig.lua
return {
    Speed = {
        id          = "Speed",
        displayName = "Washer Speed",
        description = "Dreams clean faster",
        icon        = "rbxassetid://0",
        currency    = "Coins",
        maxLevel    = 5,
        costs       = { 50, 120, 250, 500, 1000 },
        -- Effect applied in OrderSystem.getWashTime()
    },
    Value = {
        id          = "Value",
        displayName = "Dream Value",
        description = "Cleaned dreams give more coins",
        icon        = "rbxassetid://0",
        currency    = "Coins",
        maxLevel    = 5,
        costs       = { 75, 150, 300, 600, 1200 },
    },
    Luck = {
        id          = "Luck",
        displayName = "Dream Luck",
        description = "Higher chance for rare results",
        icon        = "rbxassetid://0",
        currency    = "Coins",
        maxLevel    = 5,
        costs       = { 100, 200, 400, 800, 1500 },
    },
    ExtraSlot = {
        id          = "ExtraSlot",
        displayName = "Extra Dream Slot",
        description = "Carry more orders at once",
        icon        = "rbxassetid://0",
        currency    = "Gems",
        maxLevel    = 3,
        costs       = { 5, 15, 30 },
    },
}
```

## Common Mistakes

❌ **Requiring modules inside functions** — always require at top level
```lua
-- BAD
function doThing()
    local DataManager = require(...)  -- required every call
end

-- GOOD
local DataManager = require(...)     -- required once at module load
function doThing() ... end
```

❌ **Circular dependencies** — A requires B requires A
```lua
-- BAD: CurrencySystem requires OrderSystem, OrderSystem requires CurrencySystem
-- GOOD: OrderSystem requires CurrencySystem (one direction only)
```

❌ **State in shared modules** — ReplicatedStorage modules run on both sides
```lua
-- BAD: storing server-only state in a Shared module
-- GOOD: configs only in Shared; state in server or client Systems
```
