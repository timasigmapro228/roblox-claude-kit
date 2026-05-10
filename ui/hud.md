# HUD UI Pattern

The always-visible heads-up display: currency, reputation, order slots, buttons.

## Layout Structure (Studio hierarchy)

```
StarterGui/
  HUD (ScreenGui, ResetOnSpawn=false)
    TopBar (Frame)
      CoinsDisplay (Frame)
        CoinIcon (ImageLabel)
        CoinsLabel (TextLabel)
      GemsDisplay (Frame)
        GemIcon (ImageLabel)
        GemsLabel (TextLabel)
      RepDisplay (Frame)
        RepLabel (TextLabel)
    OrderSlots (Frame)           ← bottom center
      Slot1 (Frame)
      Slot2 (Frame)              ← hidden until ExtraSlot upgrade
    Buttons (Frame)              ← bottom right
      ShopButton (TextButton)
      IndexButton (TextButton)
    NotificationArea (Frame)     ← top center, above TopBar
```

## HUDController.lua (Client)

```lua
-- StarterPlayerScripts/Client/Systems/HUDController.lua
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local CurrencyController = require(script.Parent.CurrencyController)

local player = Players.LocalPlayer
local playerGui = player.PlayerGui

local HUDController = {}
local hud: ScreenGui

-- UI references (set in init after GUI loads)
local coinsLabel: TextLabel
local gemsLabel: TextLabel
local repLabel: TextLabel

local function animateLabelBounce(label: TextLabel)
    local tween = TweenService:Create(label, TweenInfo.new(0.1, Enum.EasingStyle.Back), {
        TextSize = label.TextSize + 4
    })
    tween:Play()
    tween.Completed:Connect(function()
        TweenService:Create(label, TweenInfo.new(0.1), {
            TextSize = label.TextSize - 4
        }):Play()
    end)
end

local function formatNumber(n: number): string
    if n >= 1_000_000 then
        return string.format("%.1fM", n / 1_000_000)
    elseif n >= 1_000 then
        return string.format("%.1fK", n / 1_000)
    end
    return tostring(n)
end

function HUDController.init()
    -- Wait for GUI to load
    hud = playerGui:WaitForChild("HUD")
    local topBar = hud:WaitForChild("TopBar")

    coinsLabel = topBar.CoinsDisplay.CoinsLabel
    gemsLabel  = topBar.GemsDisplay.GemsLabel
    repLabel   = topBar.RepDisplay.RepLabel

    -- Wire currency updates
    CurrencyController.onUpdate("Coins", function(amount)
        coinsLabel.Text = "🪙 " .. formatNumber(amount)
        animateLabelBounce(coinsLabel)
    end)

    CurrencyController.onUpdate("Gems", function(amount)
        gemsLabel.Text = "💎 " .. formatNumber(amount)
        animateLabelBounce(gemsLabel)
    end)

    -- Buttons
    local buttons = hud:WaitForChild("Buttons")
    buttons.ShopButton.MouseButton1Click:Connect(function()
        -- Open shop popup
        local ShopController = require(script.Parent.ShopController)
        ShopController.open()
    end)

    buttons.IndexButton.MouseButton1Click:Connect(function()
        local IndexController = require(script.Parent.IndexController)
        IndexController.open()
    end)
end

-- Update order slot display
function HUDController.updateSlot(slotIndex: number, dreamData: any?)
    local slots = hud.OrderSlots
    local slot = slots:FindFirstChild("Slot" .. slotIndex)
    if not slot then return end

    if dreamData then
        slot.Icon.Image = dreamData.icon
        slot.Label.Text = dreamData.displayName
        slot.BackgroundColor3 = Color3.fromRGB(85, 239, 196)  -- filled: green
    else
        slot.Icon.Image = ""
        slot.Label.Text = "Empty"
        slot.BackgroundColor3 = Color3.fromRGB(49, 49, 70)    -- empty: surface
    end
end

return HUDController
```

## UI Colors (paste into Studio properties)

| Element | Color |
|---|---|
| TopBar background | `30, 30, 46` (dark) |
| Card/slot background | `49, 49, 70` (surface) |
| Coin label | `253, 203, 110` (yellow) |
| Gem label | `116, 185, 255` (blue) |
| Active/filled slot | `85, 239, 196` (green) |
| Button primary | `108, 92, 231` (purple) |
| Text | `255, 255, 255` |
| Text muted | `180, 180, 200` |

## TopBar Properties (Studio)

```
TopBar:
  Size: {1, 0}, {0, 56}
  Position: {0, 0}, {0, 0}
  AnchorPoint: 0, 0
  BackgroundColor3: 30, 30, 46
  BackgroundTransparency: 0.15
  BorderSizePixel: 0
  
  UICorner: CornerRadius 0, 8 (bottom corners only via UICorner on child)
  UIPadding: Left=12, Right=12, Top=8, Bottom=8
  UIListLayout: FillDirection=Horizontal, Padding=12px, VerticalAlignment=Center
```
