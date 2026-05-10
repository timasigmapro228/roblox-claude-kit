---
name: genre-active-simulator
description: Use when building an active simulator — players physically pick up objects, bring them to machines, process them, return results to NPCs. NOT a tycoon. Triggers on words like active simulator, NPC orders, carry, process, machine, return, service game, working simulator, job simulator.
---

# Genre: Active Simulator

**Core loop**: NPC gives order → player picks up item → brings to machine → processes (timer/minigame) → gets result with rarity → returns to NPC → earns coins

**Different from clicker simulator**: Player physically moves and interacts. No auto-collect.
**Different from tycoon**: Player is active every step. Not passive income.

**Examples**: Work at a Pizza Place, Restaurant Tycoon (active mode), Laundromat-style games, Car Wash Simulator

---

## Why This Genre Works

- Simple to understand in 3 seconds
- Every loop feels satisfying (pickup → process → reward)
- Rarity creates excitement on every order
- Upgrades have immediate visible impact
- Easy to add content (new order types, new machines)

---

## Folder Structure

```
ServerScriptService/Server/Systems/
  DataManager.lua
  CurrencySystem.lua
  OrderSystem.lua     ← core loop (from systems/orders.md)
  UpgradeSystem.lua
  NPCSystem.lua       ← from systems/npc.md

ReplicatedStorage/Shared/Modules/
  OrderConfig.lua
  UpgradeConfig.lua
  NPCConfig.lua

Workspace/
  NPCArea/            ← where NPCs stand and give orders
  MachineArea/        ← processing machines
  ReturnCounter/      ← where player returns completed orders
  UpgradeShop/        ← upgrade board/NPC
```

---

## ProximityPrompt Interaction Chain

The physical loop uses ProximityPrompts at every step:

```lua
-- Step 1: NPC gives order
-- ProximityPrompt on NPC → fires PickupOrder RemoteEvent

-- Step 2: Player brings to machine
-- ProximityPrompt on Machine → fires StartWash RemoteEvent
-- Machine shows progress bar (client-side countdown)
-- Server processes in task.spawn

-- Step 3: Player picks up result
-- ProximityPrompt on Machine output slot → fires CollectResult RemoteEvent

-- Step 4: Player returns to NPC
-- ProximityPrompt on Return Counter → fires ReturnOrder RemoteEvent
-- Coins + rare result notification

-- All ProximityPrompts disabled when player has no order (or wrong state)
```

---

## Machine Visual States

```lua
-- Client-side machine state controller
local MachineStates = {
    idle     = { color = Color3.fromRGB(100,100,120), glow = false },
    working  = { color = Color3.fromRGB(80,180,255),  glow = true  },
    done     = { color = Color3.fromRGB(85,239,196),  glow = true  },
    cooldown = { color = Color3.fromRGB(200,80,80),   glow = false },
}

-- Machine PointLight color changes to show state
-- Machine hum sound while working
-- Ding sound when done
-- Particle burst when done (brief)
```

---

## Order Slot HUD

Shows what the player is currently carrying:

```lua
-- In HUDController:
-- OrderSlots frame at bottom center
-- Each slot shows: item icon, item name, state indicator
-- State: "Carrying" → "In Machine" → "Ready" → "Returned"
-- Color changes with state
-- Slot glows green when result is ready to collect

-- Max slots determined by ExtraSlot upgrade level
-- Locked slots shown as greyed out with padlock icon
```

---

## Mini-Game Inside Machine (Optional, Week 2)

Adds skill component to the processing step:

```lua
-- Instead of just waiting, player interacts during wash:
-- "Press [E] when bar hits green zone"
-- Success: bonus rarity chance (+10% Epic)
-- Fail: normal result

-- Client-side only (visual)
-- Server validates via timing window on result collect

-- Implementation:
-- Machine sends start signal to nearby player
-- Client shows timing bar UI
-- Player presses E, client fires TimingResult to server
-- Server checks if timing was valid (within reasonable window)
-- Applies bonus if valid
```

---

## Rarity Result Presentation

This is the most important moment — make it feel great:

```lua
-- On order completion:
-- 1. Machine dings + glows
-- 2. Player walks to machine to collect
-- 3. On collect: brief pause (0.5s) for anticipation
-- 4. Screen flash (rarity color)
-- 5. Result floats up with big text
-- 6. Sound plays (common = soft, epic = dramatic)
-- 7. Coins fly from NPC to player
-- 8. HUD coin counter bounces

local rarityPresentation = {
    Common    = { flashDuration = 0.3, soundId = 0, showFloating = false },
    Rare      = { flashDuration = 0.5, soundId = 0, showFloating = true  },
    Epic      = { flashDuration = 0.8, soundId = 0, showFloating = true, screenShake = true },
    Legendary = { flashDuration = 1.2, soundId = 0, showFloating = true, screenShake = true, fullScreenReveal = true },
}
```

---

## Upgrade Impact (Make It Feel Immediate)

Every upgrade should have a visible effect the moment it's purchased:

| Upgrade | Visible change |
|---|---|
| Speed | Machine animation plays faster, progress bar fills faster |
| Value | Coin amount display shows "+X%" next to coins earned |
| Luck | Star particles appear on machine while processing |
| Extra Slot | New slot appears in HUD immediately |

---

## Map Layout Principles

```
[Entry] → [NPC Zone] → [Machine Zone] → [Return Counter]

Flow direction: one-way loop, no backtracking required

Distances (in studs):
  NPC to Machine: 15-25 studs (5-8 second walk)
  Machine to Return: 15-25 studs
  Total loop: ~60 studs, ~20 second round trip

Why this matters:
  Too close = no sense of journey
  Too far = boring walking, players quit
  20 second loop = satisfying rhythm
```

---

## Default Player Data

```lua
local DEFAULT_DATA = {
    Coins      = 0,
    Gems       = 0,
    Reputation = 0,
    Upgrades = {
        Speed     = 0,
        Value     = 0,
        Luck      = 0,
        ExtraSlot = 0,
    },
    CollectionIndex  = {},
    DailyReward      = { lastClaim = 0, streak = 0 },
    ProcessedReceipts = {},
    TutorialComplete = false,
}
```

---

## Active Simulator Checklist

Before shipping:

- [ ] Loop takes 60-90 seconds end to end (not too short, not too long)
- [ ] First order completable within 2 minutes of joining
- [ ] First upgrade affordable after 2-3 orders
- [ ] Rare result visible and exciting (not just a text change)
- [ ] NPC has personality (at least 3 dialogue lines per state)
- [ ] Machine has 3 visual states (idle / working / done)
- [ ] Tutorial guides player through first full loop
- [ ] Daily reward visible on join
- [ ] Dream Index / Collection visible and tempting
