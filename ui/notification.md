# Notification & Popup Pattern

Toast notifications and modal popups reusable across all projects.

## Toast Notification (FloatingLabel)

Shows briefly then fades. Used for: reward earned, rare result, error messages.

```lua
-- StarterPlayerScripts/Client/Systems/NotificationController.lua
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local NotificationController = {}

local notifContainer: Frame

-- Types: "success" | "error" | "rare" | "info"
local typeColors = {
    success = Color3.fromRGB(85, 239, 196),
    error   = Color3.fromRGB(255, 118, 117),
    rare    = Color3.fromRGB(180, 100, 255),
    info    = Color3.fromRGB(180, 180, 200),
}

local function createToast(message: string, notifType: string?, duration: number?)
    local color = typeColors[notifType or "info"]
    duration = duration or 2.5

    local toast = Instance.new("Frame")
    toast.Size = UDim2.new(0, 280, 0, 48)
    toast.BackgroundColor3 = Color3.fromRGB(30, 30, 46)
    toast.BorderSizePixel = 0
    toast.AnchorPoint = Vector2.new(0.5, 0)
    toast.Position = UDim2.new(0.5, 0, 0, -60)
    toast.Parent = notifContainer

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = toast

    local stroke = Instance.new("UIStroke")
    stroke.Color = color
    stroke.Thickness = 2
    stroke.Parent = toast

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -16, 1, 0)
    label.Position = UDim2.new(0, 8, 0, 0)
    label.Text = message
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.TextSize = 14
    label.Font = Enum.Font.GothamBold
    label.BackgroundTransparency = 1
    label.TextXAlignment = Enum.TextXAlignment.Center
    label.Parent = toast

    -- Slide in
    TweenService:Create(toast, TweenInfo.new(0.3, Enum.EasingStyle.Back), {
        Position = UDim2.new(0.5, 0, 0, 8)
    }):Play()

    -- Wait, then fade out
    task.delay(duration, function()
        if not toast.Parent then return end
        local fadeOut = TweenService:Create(toast, TweenInfo.new(0.3), {
            Position = UDim2.new(0.5, 0, 0, -60),
            BackgroundTransparency = 1,
        })
        fadeOut.Completed:Connect(function()
            toast:Destroy()
        end)
        fadeOut:Play()
    end)
end

function NotificationController.init()
    -- Create a container Frame in HUD ScreenGui
    local hud = player.PlayerGui:WaitForChild("HUD")
    notifContainer = Instance.new("Frame")
    notifContainer.Size = UDim2.new(1, 0, 0, 80)
    notifContainer.Position = UDim2.new(0, 0, 0, 0)
    notifContainer.BackgroundTransparency = 1
    notifContainer.Name = "NotifContainer"
    notifContainer.Parent = hud
end

-- Public API
function NotificationController.show(message: string, notifType: string?, duration: number?)
    createToast(message, notifType, duration)
end

-- Shortcut helpers
function NotificationController.success(msg: string) NotificationController.show(msg, "success") end
function NotificationController.error(msg: string) NotificationController.show(msg, "error") end
function NotificationController.rare(msg: string) NotificationController.show(msg, "rare", 3.5) end

return NotificationController
```

## Usage Examples

```lua
-- After collecting a rare result:
NotificationController.rare("✨ Sparkly Dream discovered!")

-- After purchase:
NotificationController.success("Speed upgraded to Level 2!")

-- On error:
NotificationController.error("Not enough coins!")

-- On new Index entry:
NotificationController.rare("🌙 New dream added to your Index!")
```

## Rarity Result Popup (Full Screen Flash)

For Epic results — brief dramatic flash before returning order.

```lua
local function showRarityFlash(rarity: string)
    local rarityColors = {
        Common = Color3.fromRGB(200, 200, 200),
        Rare   = Color3.fromRGB(100, 149, 237),
        Epic   = Color3.fromRGB(180, 100, 255),
    }

    local flash = Instance.new("Frame")
    flash.Size = UDim2.new(1, 0, 1, 0)
    flash.BackgroundColor3 = rarityColors[rarity] or Color3.fromRGB(255,255,255)
    flash.BackgroundTransparency = 0.7
    flash.ZIndex = 100
    flash.Parent = player.PlayerGui:WaitForChild("HUD")

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 0, 80)
    label.Position = UDim2.new(0, 0, 0.4, 0)
    label.Text = rarity:upper() .. "!"
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.TextSize = 48
    label.Font = Enum.Font.GothamBlack
    label.BackgroundTransparency = 1
    label.ZIndex = 101
    label.Parent = flash

    local flashOut = TweenService:Create(flash, TweenInfo.new(0.8), {
        BackgroundTransparency = 1
    })
    flashOut.Completed:Connect(function()
        flash:Destroy()
    end)
    flashOut:Play()
end
```
