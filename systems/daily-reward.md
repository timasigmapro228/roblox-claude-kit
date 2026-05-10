# Daily Reward System

Full daily reward implementation is in `systems/retention.md`.

This file is a quick reference stub.

## Quick Setup

```lua
-- DEFAULT_DATA addition:
DailyReward = { lastClaim = 0, streak = 0 },

-- RemoteEvent needed:
-- DailyReward ← server → client (status, data)

-- System to require in Main.server.lua:
local DailyRewardSystem = require(Systems.DailyRewardSystem)
DailyRewardSystem.init()
```

## Full implementation → `systems/retention.md` → "Daily Rewards System" section

That file contains:
- Complete DailyRewardSystem.lua with streak logic
- 7-day reward table
- 24-hour cooldown with 48-hour streak reset
- Client notification on claim
