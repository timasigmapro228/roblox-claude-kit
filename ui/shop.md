# Shop / Upgrade UI Pattern

Reusable shop for upgrades, items, and cosmetics.

## Structure (Studio hierarchy)

```
StarterGui/
  ShopPopup (ScreenGui)
    Backdrop (TextButton, full screen, semi-transparent black) ← TextButton not Frame, so MouseButton1Click works
      Panel (Frame, centered card)
        TitleBar (Frame)
          TitleLabel (TextLabel) "UPGRADES"
          CloseButton (TextButton) "✕"
        TabBar (Frame)           ← optional: Upgrades | Cosmetics | Boosts
          Tab_Upgrades (TextButton)
          Tab_Cosmetics (TextButton)
        ScrollFrame (ScrollingFrame)
          UIGridLayout
          -- UpgradeCard instances generated in code
        UICorner
        UIStroke
```

## ShopController.lua (Client)

```lua
-- StarterPlayerScripts/Client/Systems/ShopController.lua
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local UpgradeConfig = require(ReplicatedStorage.Shared.Modules.UpgradeConfig)
local CurrencyController = require(script.Parent.CurrencyController)

local PurchaseUpgrade  = ReplicatedStorage.Shared.RemoteEvents.PurchaseUpgrade
local UpgradeConfirmed = ReplicatedStorage.Shared.RemoteEvents.UpgradeConfirmed

local player = Players.LocalPlayer
local ShopController = {}

local popup: ScreenGui
local isOpen = false

-- Track displayed upgrade cards: { [upgradeId] = { card, levelLabel, costLabel, button } }
local cards: {[string]: any} = {}

local function createUpgradeCard(parent: Instance, upgradeId: string, config: any): Frame
    local card = Instance.new("Frame")
    card.Size = UDim2.new(0, 160, 0, 200)
    card.BackgroundColor3 = Color3.fromRGB(49, 49, 70)
    card.BorderSizePixel = 0
    card.Name = upgradeId

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = card

    local icon = Instance.new("ImageLabel")
    icon.Size = UDim2.new(0, 64, 0, 64)
    icon.Position = UDim2.new(0.5, -32, 0, 12)
    icon.Image = config.icon
    icon.BackgroundTransparency = 1
    icon.Parent = card

    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, -16, 0, 20)
    nameLabel.Position = UDim2.new(0, 8, 0, 84)
    nameLabel.Text = config.displayName
    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    nameLabel.TextSize = 14
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.BackgroundTransparency = 1
    nameLabel.TextWrapped = true
    nameLabel.Parent = card

    local levelLabel = Instance.new("TextLabel")
    levelLabel.Size = UDim2.new(1, -16, 0, 18)
    levelLabel.Position = UDim2.new(0, 8, 0, 108)
    levelLabel.Text = "Level 0 / " .. config.maxLevel
    levelLabel.TextColor3 = Color3.fromRGB(180, 180, 200)
    levelLabel.TextSize = 12
    levelLabel.Font = Enum.Font.Gotham
    levelLabel.BackgroundTransparency = 1
    levelLabel.Parent = card

    local buyButton = Instance.new("TextButton")
    buyButton.Size = UDim2.new(1, -16, 0, 36)
    buyButton.Position = UDim2.new(0, 8, 1, -44)
    buyButton.Text = "🪙 " .. tostring(config.costs[1])
    buyButton.TextColor3 = Color3.fromRGB(30, 30, 46)
    buyButton.BackgroundColor3 = Color3.fromRGB(253, 203, 110)
    buyButton.TextSize = 14
    buyButton.Font = Enum.Font.GothamBold
    buyButton.BorderSizePixel = 0
    buyButton.Parent = card

    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = buyButton

    buyButton.MouseButton1Click:Connect(function()
        PurchaseUpgrade:FireServer(upgradeId)
    end)

    card.Parent = parent
    cards[upgradeId] = {
        card       = card,
        levelLabel = levelLabel,
        buyButton  = buyButton,
    }
    return card
end

local function updateCard(upgradeId: string, newLevel: number)
    local entry = cards[upgradeId]
    if not entry then return end

    local config = UpgradeConfig[upgradeId]
    entry.levelLabel.Text = "Level " .. newLevel .. " / " .. config.maxLevel

    if newLevel >= config.maxLevel then
        entry.buyButton.Text = "MAX"
        entry.buyButton.BackgroundColor3 = Color3.fromRGB(85, 239, 196)
        entry.buyButton.Active = false
    else
        local nextCost = config.costs[newLevel + 1]
        local currencyIcon = config.currency == "Gems" and "💎" or "🪙"
        entry.buyButton.Text = currencyIcon .. " " .. tostring(nextCost)
    end
end

function ShopController.init()
    popup = player.PlayerGui:WaitForChild("ShopPopup")
    popup.Enabled = false

    local panel = popup.Backdrop.Panel
    local scroll = panel.ScrollFrame

    -- Generate upgrade cards
    for upgradeId, config in pairs(UpgradeConfig) do
        createUpgradeCard(scroll, upgradeId, config)
    end

    -- Close button
    panel.TitleBar.CloseButton.MouseButton1Click:Connect(ShopController.close)
    popup.Backdrop.MouseButton1Click:Connect(ShopController.close)

    -- Listen for purchase results
    UpgradeConfirmed.OnClientEvent:Connect(function(
        success: boolean,
        reason: string,
        upgradeId: string?,
        newLevel: number?
    )
        if success and upgradeId and newLevel then
            updateCard(upgradeId, newLevel)
        end
    end)
end

function ShopController.open()
    if isOpen then return end
    isOpen = true
    popup.Enabled = true
    -- Animate in
    local panel = popup.Backdrop.Panel
    panel.Position = UDim2.new(0.5, 0, 1.5, 0)  -- start below screen
    panel.AnchorPoint = Vector2.new(0.5, 0.5)
    TweenService:Create(panel, TweenInfo.new(0.3, Enum.EasingStyle.Back), {
        Position = UDim2.new(0.5, 0, 0.5, 0)
    }):Play()
end

function ShopController.close()
    if not isOpen then return end
    isOpen = false
    local panel = popup.Backdrop.Panel
    local tween = TweenService:Create(panel,
        TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {
        Position = UDim2.new(0.5, 0, 1.5, 0)
    })
    tween.Completed:Connect(function()
        popup.Enabled = false
    end)
    tween:Play()
end

return ShopController
```

## Panel Properties (Studio)

```
Panel:
  Size: {0, 480}, {0, 560}
  AnchorPoint: 0.5, 0.5
  Position: {0.5, 0}, {0.5, 0}
  BackgroundColor3: 30, 30, 46
  BorderSizePixel: 0
  
  UICorner: 16px
  UIStroke: Color=108,92,231 Thickness=2

ScrollFrame:
  Size: {1, -24}, {1, -80}
  Position: {0, 12}, {0, 70}
  BackgroundTransparency: 1
  ScrollBarThickness: 4
  
  UIGridLayout:
    CellSize: {0, 160}, {0, 200}
    CellPadding: {0, 12}, {0, 12}
    HorizontalAlignment: Center
```
