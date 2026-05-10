# RemoteEvents Pattern

Client-server communication in Roblox. Server is always authoritative.

## The Golden Rule

```
Client  →  RemoteEvent:FireServer()   →  Server validates → Server acts
Server  →  RemoteEvent:FireClient()   →  Client updates visuals only
```

**Never trust the client. Validate everything on the server.**

## RemoteEvent vs RemoteFunction

| | RemoteEvent | RemoteFunction |
|---|---|---|
| Direction | Both ways (fire & forget) | Request → Response |
| Server invoke client | ✅ Safe | ❌ NEVER — client can freeze server |
| Use for | Actions, updates | Rarely — use events + callbacks instead |

## Naming Convention

```
ReplicatedStorage/Shared/RemoteEvents/
  PickupOrder        ← client → server: player picked up an order
  StartWash          ← client → server: player put order in washer
  CollectResult      ← client → server: player returns order to NPC
  OrderUpdate        ← server → client: order state changed
  UpdateCurrency     ← server → client: balance changed
  PurchaseUpgrade    ← client → server: player wants to buy upgrade
  UpgradeConfirmed   ← server → client: upgrade purchase result
```

Rules:
- Client→Server events are **verbs** (actions the player requests)
- Server→Client events are **nouns/updates** (state the server broadcasts)
- Never create RemoteEvents in code — place them as instances in Studio

## Validation Template (Server-side)

```lua
SomeRemoteEvent.OnServerEvent:Connect(function(player: Player, arg1: string, arg2: number)
    -- 1. Type check every argument
    if typeof(arg1) ~= "string" then return end
    if typeof(arg2) ~= "number" then return end

    -- 2. Range / sanity check
    if arg2 <= 0 or arg2 > 1000 then return end

    -- 3. Whitelist check (if arg is an ID)
    if not ValidIds[arg1] then return end

    -- 4. Rate limiting (optional for spam protection)
    -- (track last fire time per player)

    -- 5. Business logic
    doSomething(player, arg1, arg2)
end)
```

## Shop Purchase Pattern (Full Example)

```lua
-- SERVER
PurchaseUpgrade.OnServerEvent:Connect(function(player: Player, upgradeId: string)
    -- typeof check first, then whitelist lookup
    if typeof(upgradeId) ~= "string" then return end

    local upgrade = UpgradeConfig[upgradeId]
    if not upgrade then return end

    local data = DataManager.getData(player)
    if not data then return end

    data.Upgrades = data.Upgrades or {}
    local currentLevel = data.Upgrades[upgradeId] or 0

    if currentLevel >= upgrade.maxLevel then
        UpgradeConfirmed:FireClient(player, false, "MAX", upgradeId, currentLevel)
        return
    end

    local cost = upgrade.costs[currentLevel + 1]
    local success = CurrencySystem.spend(player, upgrade.currency, cost)

    if not success then
        UpgradeConfirmed:FireClient(player, false, "BROKE", upgradeId, currentLevel)
        return
    end

    data.Upgrades[upgradeId] = currentLevel + 1
    UpgradeConfirmed:FireClient(player, true, "OK", upgradeId, currentLevel + 1)
end)

-- CLIENT
PurchaseUpgrade:FireServer("Speed")  -- fire when player clicks buy

-- Signature: (success, reason, upgradeId?, newLevel?)
-- reason: "OK" | "MAX" | "BROKE"
UpgradeConfirmed.OnClientEvent:Connect(function(
    success: boolean,
    reason: string,
    upgradeId: string?,
    newLevel: number?
)
    if success and upgradeId and newLevel then
        updateUpgradeUI(upgradeId, newLevel)
        showNotification("Upgraded!")
    else
        showNotification("Can't upgrade: " .. reason)
    end
end)
```

## Anti-Exploit Checklist

Before shipping any OnServerEvent handler, check:

- [ ] All arguments type-checked with `typeof()`
- [ ] Numeric arguments have min/max bounds
- [ ] String arguments validated against a whitelist
- [ ] Player can only act on their own data (never trust player-sent UserId)
- [ ] Actions have cooldowns if they could be spammed
- [ ] Rewards only granted server-side after validation
