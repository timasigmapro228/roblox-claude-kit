# Monetization Systems

Production-oriented Gamepass and Developer Product patterns.
Based on patterns from top Roblox games (Pet Simulator, Blox Fruits, Adopt Me).
Always test with real asset/product IDs before shipping.

---

## Core Philosophy

**The golden rule**: All core gameplay must be free.
Monetization enhances an already fun experience — it never gates it.

- Gamepasses = permanent benefits (buy once)
- Developer Products = consumables (buy repeatedly)
- Premium Payouts = passive income from Premium subscribers (automatic)

Players who feel forced to pay will leave and leave bad reviews.
Players who feel rewarded for paying will become loyal spenders.

---

## Gamepass System

### Setup in Studio
1. Publish your game
2. Go to Create → Experiences → your game → Associated Items → Passes
3. Create pass, note the **Asset ID**

### Server: GamepassSystem.lua

```lua
-- ServerScriptService/Server/Systems/GamepassSystem.lua
local Players = game:GetService("Players")
local MarketplaceService = game:GetService("MarketplaceService")

local DataManager = require(script.Parent.DataManager)

local GamepassSystem = {}

-- Define your passes here
local PASSES = {
    VIPPass = {
        id          = 0,        -- replace with real Asset ID
        name        = "VIP",
        description = "2x coins + exclusive VIP area access",
        perks       = { coinMultiplier = 2, vipAccess = true },
    },
    SpeedPass = {
        id          = 0,        -- replace with real Asset ID
        name        = "Speed Boost",
        description = "Permanent 1.5x movement speed",
        perks       = { speedMultiplier = 1.5 },
    },
    ExtraSlotPass = {
        id          = 0,
        name        = "Extra Slot",
        description = "Carry one extra order at a time",
        perks       = { extraSlots = 1 },
    },
}

-- Check if player owns a pass (checks Roblox AND cached data)
function GamepassSystem.owns(player: Player, passKey: string): boolean
    local pass = PASSES[passKey]
    if not pass then return false end

    -- Check Roblox (source of truth)
    local success, result = pcall(function()
        return MarketplaceService:UserOwnsGamePassAsync(player.UserId, pass.id)
    end)

    return success and result
end

-- Get all perks for a player
function GamepassSystem.getPerks(player: Player): {[string]: any}
    local perks = {}
    for passKey, passData in pairs(PASSES) do
        if GamepassSystem.owns(player, passKey) then
            for perk, value in pairs(passData.perks) do
                perks[perk] = value
            end
        end
    end
    return perks
end

-- Prompt player to buy a pass (from server)
function GamepassSystem.prompt(player: Player, passKey: string)
    local pass = PASSES[passKey]
    if not pass then return end
    MarketplaceService:PromptGamePassPurchase(player, pass.id)
end

function GamepassSystem.init()
    -- Handle purchase completion
    MarketplaceService.PromptGamePassPurchaseFinished:Connect(
        function(player: Player, passId: number, purchased: boolean)
            if not purchased then return end

            -- Find which pass was purchased
            for passKey, passData in pairs(PASSES) do
                if passData.id == passId then
                    -- Apply perk effects immediately
                    local data = DataManager.getData(player)
                    if data then
                        data.OwnedPasses = data.OwnedPasses or {}
                        data.OwnedPasses[passKey] = true
                    end
                    print(player.Name, "purchased", passData.name)
                end
            end
        end
    )
end

return GamepassSystem
```

---

## Developer Products (Consumables)

### Server: ProductSystem.lua

```lua
-- ServerScriptService/Server/Systems/ProductSystem.lua
local MarketplaceService = game:GetService("MarketplaceService")
local Players = game:GetService("Players")

local DataManager    = require(script.Parent.DataManager)
local CurrencySystem = require(script.Parent.CurrencySystem)

local ProductSystem = {}

-- Define products — replace IDs with real ones from Roblox dashboard
local PRODUCTS = {
    Coins100 = {
        id     = 0,     -- Developer Product Asset ID
        name   = "100 Coins",
        action = function(player)
            CurrencySystem.add(player, "Coins", 100)
        end,
    },
    Coins500 = {
        id     = 0,
        name   = "500 Coins",
        action = function(player)
            CurrencySystem.add(player, "Coins", 550)  -- +10% bonus
        end,
    },
    LuckBoost = {
        id     = 0,
        name   = "Lucky Boost (30 min)",
        action = function(player)
            local data = DataManager.getData(player)
            if data then
                data.LuckBoostExpiry = os.time() + 1800  -- 30 minutes
            end
        end,
    },
}

-- Build reverse lookup: productId → PRODUCTS key
local productById = {}
for key, product in pairs(PRODUCTS) do
    productById[product.id] = { key = key, data = product }
end

function ProductSystem.prompt(player: Player, productKey: string)
    local product = PRODUCTS[productKey]
    if not product then return end
    MarketplaceService:PromptProductPurchase(player, product.id)
end

function ProductSystem.init()
    -- CRITICAL: ProcessReceipt is the only safe place to grant products
    -- It fires on purchase AND on retry if the server crashed mid-purchase
    MarketplaceService.ProcessReceipt = function(receiptInfo)
        local player = Players:GetPlayerByUserId(receiptInfo.PlayerId)
        if not player then
            -- Player left — return NotProcessedYet so Roblox retries later
            return Enum.ProductPurchaseDecision.NotProcessedYet
        end

        local product = productById[receiptInfo.ProductId]
        if not product then
            -- Unknown product — grant nothing but mark as processed
            return Enum.ProductPurchaseDecision.PurchaseGranted
        end

        -- Check for duplicate receipt (Roblox can send same receipt twice)
        local data = DataManager.getData(player)
        if not data then
            return Enum.ProductPurchaseDecision.NotProcessedYet
        end

        local receiptKey = tostring(receiptInfo.PurchaseId)
        if data.ProcessedReceipts and data.ProcessedReceipts[receiptKey] then
            -- Already processed — safe to acknowledge
            return Enum.ProductPurchaseDecision.PurchaseGranted
        end

        -- Grant the product
        local success, err = pcall(product.data.action, player)
        if not success then
            warn("[ProductSystem] Failed to grant product:", err)
            return Enum.ProductPurchaseDecision.NotProcessedYet
        end

        -- Mark receipt as processed
        data.ProcessedReceipts = data.ProcessedReceipts or {}
        data.ProcessedReceipts[receiptKey] = true

        return Enum.ProductPurchaseDecision.PurchaseGranted
    end
end

return ProductSystem
```

---

## Monetization Design Patterns

### What top games monetize (safe, high-converting)

| Type | Examples | Why it works |
|---|---|---|
| Time savers | 2x speed, auto-collect | Players value time |
| Visual cosmetics | Skins, trails, effects | FOMO, expression |
| Luck boosters | +X% rare chance (30 min) | Excitement, not P2W |
| Extra slots | Carry more, store more | QoL, not required |
| Currency packs | Coin bundles with bonus | Value perception |
| VIP area | Exclusive zone, not better drops | Prestige |

### What to AVOID

| ❌ Pattern | Why it's bad |
|---|---|
| Paywalling core gameplay | Players leave, bad reviews |
| Pay-to-win stats | Toxic community, exodus of free players |
| Loot boxes without odds shown | Roblox policy violation |
| Forcing purchase pop-ups | Roblox ToS violation |
| Misleading descriptions | Refund requests, trust loss |

### Starter Pack (proven converter)

Show once, 24-48 hours after first session:
```
🎁 STARTER PACK — Limited Time!
• 500 Coins
• Lucky Boost (1 hour)  
• Exclusive Starter Skin
[75 Robux] ~~150 Robux~~
```
One-time purchase, 50% discount framing. Converts 3-5x better than regular shop.

### Premium Payouts (free money)

Automatic. Just make your game fun enough to retain Premium subscribers.
Premium subscribers staying in your game = Robux for you, passively.
No code needed. Focus on retention.

---

## Default Player Data (Monetization section)

```lua
local DEFAULT_DATA = {
    -- ... other fields ...
    OwnedPasses      = {},   -- { PassKey = true }
    ProcessedReceipts = {},  -- { purchaseId = true } — prevents double grants
    LuckBoostExpiry  = 0,    -- os.time() timestamp, 0 = no boost
}
```

---

## RemoteEvents for Shop UI

```
ReplicatedStorage/Shared/RemoteEvents/
  PromptGamepass    ← client → server: player clicked buy gamepass
  PromptProduct     ← client → server: player clicked buy product
  PurchaseResult    ← server → client: outcome (success/fail + message)
```

Client never calls MarketplaceService directly — always goes through server.
