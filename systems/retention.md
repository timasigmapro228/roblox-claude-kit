# Retention & Game Design Patterns

What makes players come back. Based on analysis of top Roblox games (Adopt Me, Pet Simulator 99, Blox Fruits).

---

## The Retention Loop

Every successful Roblox game has this structure:

```
Short session reward  →  "I got something"
Daily reason to return →  "I should come back tomorrow"
Long-term goal        →  "I'm working towards something"
Social hook           →  "My friends are here"
```

Miss any one of these and retention collapses.

---

## Daily Rewards System

```lua
-- ServerScriptService/Server/Systems/DailyRewardSystem.lua
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local DataManager    = require(script.Parent.DataManager)
local CurrencySystem = require(script.Parent.CurrencySystem)

local RE_DailyReward = ReplicatedStorage.Shared.RemoteEvents.DailyReward

local DailyRewardSystem = {}

-- Rewards by streak day (loops after day 7)
local REWARDS = {
    [1] = { coins = 50,  gems = 0, label = "Day 1" },
    [2] = { coins = 75,  gems = 0, label = "Day 2" },
    [3] = { coins = 100, gems = 1, label = "Day 3" },
    [4] = { coins = 125, gems = 1, label = "Day 4" },
    [5] = { coins = 175, gems = 2, label = "Day 5" },
    [6] = { coins = 200, gems = 2, label = "Day 6" },
    [7] = { coins = 300, gems = 5, label = "Day 7 🎉", special = true },
}

local SECONDS_PER_DAY = 86400  -- 24 hours

function DailyRewardSystem.init()
    Players.PlayerAdded:Connect(function(player)
        DataManager.onLoaded(player, function(data)
            local now = os.time()
            local lastClaim = data.DailyReward and data.DailyReward.lastClaim or 0
            local streak = data.DailyReward and data.DailyReward.streak or 0
            local timeSince = now - lastClaim

            -- Check if eligible (24 hours passed)
            if timeSince < SECONDS_PER_DAY then
                -- Send time remaining to client
                RE_DailyReward:FireClient(player, "Waiting", SECONDS_PER_DAY - timeSince)
                return
            end

            -- Check if streak broken (48+ hours = reset to day 1)
            if timeSince > SECONDS_PER_DAY * 2 then
                streak = 0
            end

            -- Next day in cycle
            local nextDay = (streak % 7) + 1
            local reward = REWARDS[nextDay]

            -- Grant reward
            CurrencySystem.add(player, "Coins", reward.coins)
            if reward.gems > 0 then
                CurrencySystem.add(player, "Gems", reward.gems)
            end

            -- Update data
            data.DailyReward = {
                lastClaim = now,
                streak    = streak + 1,
            }

            RE_DailyReward:FireClient(player, "Claimed", {
                day    = nextDay,
                reward = reward,
                streak = streak + 1,
            })
        end)
    end)
end

return DailyRewardSystem
```

---

## Collection / Index System

The "Pokédex effect" — players complete collections compulsively.

```lua
-- Add to DEFAULT_DATA:
CollectionIndex = {},  -- { itemKey = { discovered = true, rarity = "Epic", count = 5 } }

-- When player discovers a new item:
local function onItemDiscovered(player, itemKey, rarity)
    local data = DataManager.getData(player)
    if not data then return end

    local isNew = not data.CollectionIndex[itemKey]

    data.CollectionIndex[itemKey] = data.CollectionIndex[itemKey] or {}
    data.CollectionIndex[itemKey].discovered = true
    data.CollectionIndex[itemKey].count = (data.CollectionIndex[itemKey].count or 0) + 1

    -- Track best rarity
    local rarityRank = { Common=1, Rare=2, Epic=3, Legendary=4 }
    local currentBest = data.CollectionIndex[itemKey].bestRarity
    if not currentBest or rarityRank[rarity] > rarityRank[currentBest] then
        data.CollectionIndex[itemKey].bestRarity = rarity
    end

    if isNew then
        -- Fire "new discovery" notification
        local RE = game:GetService("ReplicatedStorage").Shared.RemoteEvents
        RE.NewDiscovery:FireClient(player, itemKey, rarity)
    end
end
```

---

## Streak & Session Design

**Session length target: 10-20 minutes**

Too short (< 5 min): players don't invest emotionally
Too long (> 30 min required): casual players drop off

Design your core loop timing:
- One complete loop: 2-4 minutes
- Natural stopping point every 10 minutes
- Always leave player with "one more thing" to do

```lua
-- Session timer (for analytics, not gameplay)
local sessionStart = os.time()

game.Players.PlayerRemoving:Connect(function(player)
    local sessionLength = os.time() - sessionStart
    -- Log or use for analytics
    -- Target: average session > 10 minutes
end)
```

---

## New Player Experience (First 30 Seconds)

This is the most important design problem.
Players who don't understand the game in 30 seconds leave forever.

**Required elements:**
1. **Instant action** — player can do something within 10 seconds of spawning
2. **Quick win** — first reward within 60 seconds
3. **Clear next step** — always visible what to do next (arrow, highlight, label)
4. **No walls of text** — teach by doing, not by reading

```lua
-- TutorialSystem.lua
-- Track tutorial state per player

local TUTORIAL_STEPS = {
    {
        id      = "step1_approach_npc",
        message = "Walk up to the NPC to get your first order!",
        arrow   = Vector3.new(0, 0, 0),  -- point at NPC position
    },
    {
        id      = "step2_take_order",
        message = "Press E to accept the order!",
    },
    {
        id      = "step3_use_machine",
        message = "Bring the order to the machine!",
        arrow   = Vector3.new(10, 0, 0),
    },
    {
        id      = "step4_collect",
        message = "Return the finished order to earn coins!",
    },
    {
        id      = "step5_upgrade",
        message = "Use your coins to upgrade in the shop!",
    },
}

-- Tutorial is complete when all steps are done
-- Store in data.TutorialComplete = true
-- Show UI arrow pointing to next objective
-- Highlight interactive objects with a glow/outline
```

---

## Progression Curves

Coins needed to feel "free" vs "grind":

```lua
-- Upgrade cost scaling that feels fair:
-- Level 1: 50 coins (earn in ~2 minutes)
-- Level 2: 120 coins (earn in ~5 minutes)
-- Level 3: 250 coins (earn in ~10 minutes)
-- Level 4: 500 coins (earn in ~20 minutes)
-- Level 5: 1000 coins (earn in ~40 minutes)

-- Formula: cost[n] = baseCost * (multiplier ^ (n-1))
-- baseCost = 50, multiplier = 2.2
local function getUpgradeCost(baseCoins: number, level: number): number
    return math.floor(baseCoins * (2.2 ^ (level - 1)))
end
```

**Rules:**
- First upgrade always in first session (< 5 min of coins)
- Each upgrade should feel impactful immediately
- Max level should require 3-5 hours of play (not 30)

---

## Top Games Analysis

| Game | Core retention hook | Monetization |
|---|---|---|
| **Pet Simulator 99** | Collection index, hatching excitement | Pet storage, boosts |
| **Adopt Me** | Social trading, nurturing loop | Bucks, egg passes |
| **Blox Fruits** | RPG progression, PvP | 2x EXP, stat refunds |
| **Brookhaven** | Social freedom, roleplay | Exclusive houses/cars |
| **Murder Mystery 2** | Short sessions, social | Knife skins, coins |

**Common thread**: All have a **daily reason to return** + **visible long-term goal** + **social element**.

---

## Anti-Patterns That Kill Games

| ❌ Pattern | Effect |
|---|---|
| No tutorial | 80% drop-off in first session |
| First session too long without reward | Players leave before investing |
| Upgrades too expensive | Players feel grind, quit |
| Upgrades too cheap | Nothing to work toward |
| No daily hook | Players don't return |
| Forcing social (can't play solo) | Huge friction for new players |
| Pay wall on core loop | Bad reviews, exodus |
| Updates too infrequent (> 4 weeks) | Player base evaporates |
