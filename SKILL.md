---
name: roblox-claude-kit
description: Use when building any Roblox game or place. Covers Luau code generation, game systems (currency, inventory, orders, pets, shop, DataStore), UI templates, and architecture patterns. Triggers on any mention of Roblox, Studio, Luau, place, game loop, NPC, DataStore, RemoteEvent, or simulator/tycoon/obby game types.
---

# Roblox Claude Kit

A production-oriented Roblox architecture kit for Claude Code.
Use it as the project brain for Luau systems, Studio structure, UI, DataStore, RemoteEvents, and MCP workflows.
An open alternative to plugin-only Roblox AI workflows.

## How to Use This Kit

Before writing any code, read the relevant reference file for the system you need:

| Task | Read |
|---|---|
| **GENRE TEMPLATES** | |
| Simulator (clicking/collecting/rebirth) | `genres/simulator.md` |
| Tycoon (dropper/conveyor/buttons) | `genres/tycoon.md` |
| Obby (stages/checkpoints/kill bricks) | `genres/obby.md` |
| Pet Simulator (eggs/hatch/rarity/equip) | `genres/pet-simulator.md` |
| Active Simulator (NPC orders/machines) | `genres/active-simulator.md` |
| **GAME SYSTEMS** | |
| Currency / coins / gems | `systems/currency.md` |
| NPC orders / processing | `systems/orders.md` |
| NPC characters + dialogue | `systems/npc.md` |
| Inventory grid | `systems/inventory.md` |
| Pet system (multipliers) | `systems/pets.md` |
| Shop / upgrade logic | `systems/shop.md` |
| Gamepass + Developer Products | `systems/monetization.md` |
| Daily rewards | `systems/daily-reward.md` |
| Retention + progression design | `systems/retention.md` |
| **UI** | |
| HUD layout | `ui/hud.md` |
| Shop / upgrade UI | `ui/shop.md` |
| Notifications / toasts | `ui/notification.md` |
| UI style presets (Simulator/Flat/Anime/RPG) | `ui/styles.md` |
| **PATTERNS** | |
| Module architecture | `patterns/modulescript.md` |
| Client-server events + security | `patterns/remoteevents.md` |
| Save / load data (DataStore) | `patterns/datastore.md` |
| Performance + mobile optimization | `patterns/performance.md` |
| **REFERENCE** | |
| Complete Roblox API (services, data types, deprecated) | `roblox-api-reference.md` |
| Complete GUI reference (all elements, modifiers, animations) | `roblox-gui-complete.md` |
| MCP Studio workflow (inspect, create, patch, debug) | `mcp/studio-mcp-workflow.md` |
| Build map from image/sketch | `map-from-image.md` |
| How to prompt Claude effectively | `PROMPTING.md` |

## Core Rules — Always Follow

### Architecture
- Every system is a **ModuleScript** — no logic in standalone Scripts beyond bootstrapping
- Server is **authoritative**: clients request, server validates, server confirms
- Client only does visuals — never trust client math for rewards or state
- All systems have an `init()` function called at startup

### Luau Style
- Use `game:GetService()` at the top of every file, alphabetical order
- Use typed Luau: annotate function params and return types
- Use `task.spawn()` not `coroutine.wrap()` for async work
- Use `task.wait()` not `wait()` (deprecated)
- camelCase for variables and functions, PascalCase for modules and classes

### Safety
- Wrap **every** DataStore call in `pcall`
- Validate **every** RemoteEvent argument on the server
- Never use `RemoteFunction:InvokeClient()` from server — it can yield forever
- Save data on both `Players.PlayerRemoving` AND `game:BindToClose()`

### Folder Structure
```
ReplicatedStorage/
  Shared/
    Modules/        ← shared ModuleScripts (configs, utils)
    RemoteEvents/   ← all RemoteEvent instances live here
    RemoteFunctions/
ServerScriptService/
  Server/
    Systems/        ← server-side ModuleScripts
    Main.server.lua ← bootstrapper: requires all systems, calls init()
StarterPlayerScripts/
  Client/
    Systems/        ← client-side ModuleScripts
    Main.client.lua ← client bootstrapper
StarterGui/
  Screens/          ← ScreenGui instances with LocalScripts
```

## Quick-Start Template

### Main.server.lua (bootstrapper)
```lua
-- Main.server.lua
local ServerScriptService = game:GetService("ServerScriptService")

local Systems = ServerScriptService.Server.Systems

-- Require all systems
local DataManager  = require(Systems.DataManager)
local CurrencySystem = require(Systems.CurrencySystem)
local OrderSystem  = require(Systems.OrderSystem)

-- Init in dependency order
DataManager.init()
CurrencySystem.init()
OrderSystem.init()
```

### Main.client.lua (bootstrapper)
```lua
-- Main.client.lua
local Players = game:GetService("Players")
local player = Players.LocalPlayer

local ClientSystems = script.Parent.Systems

local HUDController    = require(ClientSystems.HUDController)
local OrderController  = require(ClientSystems.OrderController)

HUDController.init()
OrderController.init()
```

## Extensibility Fields

When creating any config, always include these fields for future-proofing:

```lua
-- ItemConfig example (rename to match your game)
local Items = {
    ItemA = {
        id = "ItemA",
        displayName = "Item A",
        icon = "rbxassetid://000",
        baseCoins = 10,
        baseRep = 5,
        difficulty = 1,          -- 1 = Normal, 2 = Hard
        cleaningSteps = 1,       -- extend to 2 for harder orders
        orderType = "Normal",    -- "Normal" | "Hard"
        rarityWeights = {
            Common = 70,
            Rare = 25,
            Epic = 5,
        },
    },
}
return Items
```

## Rarity System (Universal)

Use this pattern across all systems (dreams, pets, items):

```lua
local RarityConfig = {
    Common = { weight = 70, color = Color3.fromRGB(200, 200, 200), label = "Common" },
    Rare   = { weight = 25, color = Color3.fromRGB(100, 149, 237), label = "Rare" },
    Epic   = { weight = 5,  color = Color3.fromRGB(180, 100, 255), label = "Epic" },
}

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
```

## UI Color Palette (Universal)

Consistent across all UIs in the project:

```lua
local Theme = {
    Primary    = Color3.fromRGB(108, 92, 231),   -- purple
    Secondary  = Color3.fromRGB(253, 203, 110),  -- yellow
    Background = Color3.fromRGB(30, 30, 46),     -- dark
    Surface    = Color3.fromRGB(49, 49, 70),     -- card bg
    Text       = Color3.fromRGB(255, 255, 255),
    TextMuted  = Color3.fromRGB(180, 180, 200),
    Success    = Color3.fromRGB(85, 239, 196),
    Danger     = Color3.fromRGB(255, 118, 117),
    Coin       = Color3.fromRGB(253, 203, 110),
    Gem        = Color3.fromRGB(116, 185, 255),
}
```

## What NOT to Do

- ❌ Don't put game logic in LocalScripts — exploiters can modify it
- ❌ Don't use `wait()` — use `task.wait()`
- ❌ Don't save data more than once per 6 seconds per key (Roblox rate limit)
- ❌ Don't use `RemoteFunction:InvokeClient()` from server
- ❌ Don't trust any value sent from client without server-side validation
- ❌ Don't create RemoteEvents in code — create them as instances in Studio/MCP under ReplicatedStorage/Shared/RemoteEvents

## How Claude Should Behave When Using This Kit

When a user asks to build something:

1. **Always outline first** — before writing code, describe the modules, RemoteEvents, and dependency order. If the user asked for a plan or review, wait for confirmation. If they asked for direct implementation or MCP workflow, proceed in small safe steps without repeatedly asking.
2. **Build in small steps** — one module at a time, not the whole system at once.
3. **State success criteria** — after writing code, tell the user exactly how to test it in Studio step-by-step.
4. **Ask for images** — if the user is describing a UI, ask them to paste a screenshot or reference.
5. **Flag security-critical code** — when writing currency, DataStore, or RemoteEvent handlers, note what to manually review.
6. **Explain architectural choices** — briefly explain why you chose RemoteEvent vs RemoteFunction, etc.

## Reading Order for New Projects

1. `PROMPTING.md` — how to communicate with Claude for best results
2. `patterns/modulescript.md` — architecture foundation
3. `patterns/remoteevents.md` — client-server communication
4. `patterns/datastore.md` — saving data
5. Then any system files you need
