---
name: roblox-gui-complete
description: Use when creating, modifying, or animating any Roblox GUI element — ScreenGui, Frame, TextLabel, TextButton, ImageLabel, ScrollingFrame, UICorner, UIGradient, UIStroke, UIListLayout, UIGridLayout, UIPadding, BillboardGui, SurfaceGui. Triggers on any GUI/UI task.
---

# Roblox GUI — Complete Reference

## Container Hierarchy

```
PlayerGui (auto-created per player)
  └── ScreenGui           ← main 2D UI container
        └── Frame         ← layout container
              └── TextLabel, TextButton, ImageLabel, etc.

workspace
  └── Part
        └── SurfaceGui    ← UI rendered on a 3D surface
        └── BillboardGui  ← UI that always faces camera (name tags, health bars)
```

---

## ScreenGui Properties

```lua
local gui = Instance.new("ScreenGui")
gui.Name = "MyGui"
gui.ResetOnSpawn = false      -- IMPORTANT: keep GUI on death
gui.IgnoreGuiInset = true     -- extends to top of screen (past safe area)
gui.DisplayOrder = 0          -- layering order (higher = on top)
gui.Enabled = true
gui.Parent = player.PlayerGui
```

---

## All GUI Elements — Properties Reference

### Frame
```lua
local frame = Instance.new("Frame")
frame.Size     = UDim2.new(0, 300, 0, 200)      -- width=300px, height=200px
frame.Position = UDim2.new(0.5, -150, 0.5, -100) -- centered
frame.AnchorPoint = Vector2.new(0.5, 0.5)         -- pivot point (0-1 each axis)

frame.BackgroundColor3    = Color3.fromRGB(30, 30, 46)
frame.BackgroundTransparency = 0    -- 0=opaque, 1=invisible
frame.BorderSizePixel     = 0       -- always set to 0, use UIStroke instead
frame.ClipsDescendants    = false   -- true = children clipped to bounds
frame.ZIndex              = 1       -- layer order within parent
frame.Visible             = true
frame.Active              = false   -- true = blocks mouse input
frame.Parent = screenGui
```

### TextLabel
```lua
local label = Instance.new("TextLabel")
label.Size             = UDim2.new(1, 0, 0, 30)
label.BackgroundTransparency = 1                 -- almost always 1
label.Text             = "Hello World"
label.TextColor3       = Color3.fromRGB(255, 255, 255)
label.TextTransparency = 0
label.TextSize         = 16
label.Font             = Enum.Font.GothamBold    -- see fonts below
label.TextScaled       = false                   -- true = auto-fit (use UITextSizeConstraint)
label.TextWrapped      = true                    -- wrap long text
label.TextTruncate     = Enum.TextTruncate.AtEnd -- "..." when too long
label.TextXAlignment   = Enum.TextXAlignment.Center  -- Left, Center, Right
label.TextYAlignment   = Enum.TextYAlignment.Center  -- Top, Center, Bottom
label.RichText         = false  -- true enables <b>, <i>, <font> tags
label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
label.TextStrokeTransparency = 0.8  -- outline on text (0=visible, 1=none)
label.LineHeight       = 1.0
label.Parent = frame
```

### TextButton
```lua
local btn = Instance.new("TextButton")
-- All TextLabel properties, plus:
btn.AutoButtonColor = false   -- ALWAYS false for custom styling
btn.Active          = true
btn.Modal           = false

-- Events:
btn.MouseButton1Click:Connect(function() end)    -- click (down+up)
btn.MouseButton1Down:Connect(function() end)     -- press down
btn.MouseButton1Up:Connect(function() end)       -- release
btn.MouseEnter:Connect(function() end)           -- hover in
btn.MouseLeave:Connect(function() end)           -- hover out

-- Button press animation:
btn.MouseButton1Down:Connect(function()
    TweenService:Create(btn, TweenInfo.new(0.08), {
        Size = btn.Size - UDim2.fromOffset(4, 4)
    }):Play()
end)
btn.MouseButton1Up:Connect(function()
    TweenService:Create(btn, TweenInfo.new(0.08), {
        Size = btn.Size + UDim2.fromOffset(4, 4)
    }):Play()
end)
```

### ImageLabel & ImageButton
```lua
local img = Instance.new("ImageLabel")
img.Image           = "rbxassetid://12345678"
img.ImageColor3     = Color3.fromRGB(255, 255, 255)  -- tint
img.ImageTransparency = 0
img.ScaleType       = Enum.ScaleType.Fit    -- Fit, Fill, Crop, Slice, Tile
img.ResampleMode    = Enum.ResamplerMode.Default  -- or Pixelated

-- 9-slice (for buttons/panels that scale without distortion):
img.ScaleType       = Enum.ScaleType.Slice
img.SliceCenter     = Rect.new(10, 10, 90, 90)  -- center region in pixels
img.SliceScale      = 1                          -- scale of corners

-- Aspect ratio preservation:
local constraint = Instance.new("UIAspectRatioConstraint")
constraint.AspectRatio = 1  -- 1 = square, 16/9 = widescreen
constraint.Parent = img
```

### TextBox (Input)
```lua
local box = Instance.new("TextBox")
box.PlaceholderText   = "Enter code..."
box.PlaceholderColor3 = Color3.fromRGB(120, 120, 140)
box.ClearTextOnFocus  = true
box.MultiLine         = false
box.TextEditable      = true

box.FocusLost:Connect(function(enterPressed: boolean)
    if enterPressed then
        local text = box.Text
        -- process input
    end
end)
box:CaptureFocus()    -- force keyboard up (mobile)
box:ReleaseFocus()    -- hide keyboard
```

### ScrollingFrame
```lua
local scroll = Instance.new("ScrollingFrame")
scroll.Size              = UDim2.new(1, 0, 1, -60)
scroll.BackgroundTransparency = 1
scroll.BorderSizePixel   = 0
scroll.ScrollBarThickness = 4
scroll.ScrollBarImageColor3 = Color3.fromRGB(108, 92, 231)
scroll.ScrollingDirection = Enum.ScrollingDirection.Y   -- X, Y, or XY
scroll.CanvasSize        = UDim2.new(0, 0, 0, 0)  -- set dynamically
scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y  -- auto-expand with content
scroll.ElasticBehavior   = Enum.ElasticBehavior.WhenScrollable

-- With UIListLayout, CanvasSize auto-updates if AutomaticCanvasSize = Y
```

### VideoFrame
```lua
local video = Instance.new("VideoFrame")
video.Video   = "rbxassetid://..."
video.Playing = false
video.Looped  = false
video.Volume  = 0.5
video:Play()
video:Pause()
video.Ended:Connect(function() end)
```

### ViewportFrame (3D in GUI)
```lua
local viewport = Instance.new("ViewportFrame")
viewport.Size = UDim2.new(0, 200, 0, 200)

-- Add 3D model to viewport:
local model = petModel:Clone()
model.Parent = viewport

-- Camera for viewport:
local cam = Instance.new("Camera")
cam.CFrame = CFrame.new(0, 0, 5) * CFrame.Angles(0, 0, 0)
cam.Parent = viewport
viewport.CurrentCamera = cam
```

---

## UI Modifier Components

Every modifier is PARENTED to the element it modifies:

### UICorner
```lua
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)   -- 12px radius
corner.CornerRadius = UDim.new(0.5, 0)  -- 50% = circle (for square frames)
corner.Parent = frame
```

### UIStroke
```lua
local stroke = Instance.new("UIStroke")
stroke.Thickness    = 2
stroke.Color        = Color3.fromRGB(108, 92, 231)
stroke.Transparency = 0        -- 0=opaque stroke
stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border  -- or Contextual (for text)
stroke.LineJoinMode = Enum.LineJoinMode.Round
stroke.Parent = frame  -- or TextLabel for text outline

-- Gradient stroke:
local gradient = Instance.new("UIGradient")
gradient.Color = ColorSequence.new(
    Color3.fromRGB(255, 100, 100),
    Color3.fromRGB(100, 100, 255)
)
gradient.Parent = stroke  -- UIGradient inside UIStroke = gradient outline
```

### UIGradient
```lua
local gradient = Instance.new("UIGradient")
gradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0,   Color3.fromRGB(108, 92, 231)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(180, 100, 255)),
    ColorSequenceKeypoint.new(1,   Color3.fromRGB(80, 60, 200)),
})
gradient.Transparency = NumberSequence.new({
    NumberSequenceKeypoint.new(0, 0),
    NumberSequenceKeypoint.new(1, 0.3),
})
gradient.Rotation = 90  -- 0=left→right, 90=top→bottom, 45=diagonal
gradient.Offset   = Vector2.new(0, 0)  -- -1 to 1 (shifts gradient)

-- ❌ UIGradient does NOT work on: ScrollingFrame, TextBox
-- ✅ Works on: Frame, TextLabel, TextButton, ImageLabel, ImageButton, ViewportFrame
-- Max 6 color stops for performance

gradient.Parent = frame
```

### UIPadding
```lua
local pad = Instance.new("UIPadding")
pad.PaddingTop    = UDim.new(0, 12)
pad.PaddingBottom = UDim.new(0, 12)
pad.PaddingLeft   = UDim.new(0, 16)
pad.PaddingRight  = UDim.new(0, 16)
pad.Parent = frame
```

### UIListLayout
```lua
local list = Instance.new("UIListLayout")
list.FillDirection    = Enum.FillDirection.Vertical   -- or Horizontal
list.HorizontalAlignment = Enum.HorizontalAlignment.Center  -- Left, Center, Right
list.VerticalAlignment   = Enum.VerticalAlignment.Top        -- Top, Center, Bottom
list.SortOrder        = Enum.SortOrder.LayoutOrder    -- or Name
list.Padding          = UDim.new(0, 8)                -- gap between children
list.Parent = frame

-- Get total content size (for ScrollingFrame):
local height = list.AbsoluteContentSize.Y
scroll.CanvasSize = UDim2.new(0, 0, 0, height)
```

### UIGridLayout
```lua
local grid = Instance.new("UIGridLayout")
grid.CellSize        = UDim2.new(0, 160, 0, 200)
grid.CellPadding     = UDim2.new(0, 8, 0, 8)
grid.FillDirection   = Enum.FillDirection.Horizontal
grid.HorizontalAlignment = Enum.HorizontalAlignment.Center
grid.SortOrder       = Enum.SortOrder.LayoutOrder
grid.Parent = scrollFrame
```

### UITableLayout
```lua
local table = Instance.new("UITableLayout")
table.FillDirection  = Enum.FillDirection.Vertical
table.MajorAxis      = Enum.TableMajorAxis.ColumnMajor
table.Padding        = UDim2.new(0, 4, 0, 4)
table.FillEmptySpaceColumns = false
table.FillEmptySpaceRows    = false
table.Parent = frame
```

### UIPageLayout (for swipeable pages)
```lua
local pages = Instance.new("UIPageLayout")
pages.PageTransitionTime = 0.3
pages.EasingStyle        = Enum.EasingStyle.Quad
pages.EasingDirection    = Enum.EasingDirection.Out
pages.Animated           = true
pages.Parent = frame

pages:Next()      -- go to next page
pages:Previous()  -- go to previous page
pages:JumpTo(frame.ChildFrame)  -- jump to specific page
```

### UISizeConstraint
```lua
local constraint = Instance.new("UISizeConstraint")
constraint.MinSize = Vector2.new(100, 40)
constraint.MaxSize = Vector2.new(400, 60)
constraint.Parent = frame
```

### UIAspectRatioConstraint
```lua
local aspect = Instance.new("UIAspectRatioConstraint")
aspect.AspectRatio   = 16/9   -- width/height ratio
aspect.AspectType    = Enum.AspectType.FitWithinMaxSize
aspect.DominantAxis  = Enum.DominantAxis.Width
aspect.Parent = frame
```

### UITextSizeConstraint
```lua
local textConstraint = Instance.new("UITextSizeConstraint")
textConstraint.MinTextSize = 10
textConstraint.MaxTextSize = 24
-- Use with TextScaled = true to clamp font size
textConstraint.Parent = textLabel
```

### UIFlexItem (newer, for flex layouts)
```lua
local flex = Instance.new("UIFlexItem")
flex.FlexMode = Enum.UIFlexMode.Grow  -- Shrink, GrowAndShrink
flex.Parent = child  -- parent frame needs UIListLayout with FillDirection
```

### UIScale
```lua
local scale = Instance.new("UIScale")
scale.Scale = 1.2  -- scale entire GUI (useful for hover effects)
scale.Parent = frame

-- Hover scale animation:
frame.MouseEnter:Connect(function()
    TweenService:Create(scale, TweenInfo.new(0.15, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Scale = 1.05
    }):Play()
end)
frame.MouseLeave:Connect(function()
    TweenService:Create(scale, TweenInfo.new(0.1), { Scale = 1.0 }):Play()
end)
```

---

## 3D GUI: BillboardGui & SurfaceGui

### BillboardGui (always faces camera — name tags, health bars)
```lua
local billboard = Instance.new("BillboardGui")
billboard.Size         = UDim2.new(0, 200, 0, 50)
billboard.StudsOffset  = Vector3.new(0, 3, 0)   -- offset from parent
billboard.MaxDistance  = 50                      -- hide beyond 50 studs
billboard.AlwaysOnTop  = false                   -- true = visible through walls
billboard.Adornee      = part                    -- optional explicit target
billboard.Parent       = part  -- or character HumanoidRootPart

-- Name tag pattern:
local nameTag = Instance.new("TextLabel")
nameTag.Size = UDim2.new(1, 0, 1, 0)
nameTag.Text = player.DisplayName
nameTag.TextColor3 = Color3.fromRGB(255, 255, 255)
nameTag.TextStrokeTransparency = 0
nameTag.BackgroundTransparency = 1
nameTag.Font = Enum.Font.GothamBold
nameTag.TextSize = 14
nameTag.Parent = billboard
```

### SurfaceGui (renders on part surface)
```lua
local surfGui = Instance.new("SurfaceGui")
surfGui.Face     = Enum.NormalId.Front    -- Top, Bottom, Left, Right, Front, Back
surfGui.SizingMode = Enum.SurfaceGuiSizingMode.PixelsPerStud
surfGui.PixelsPerStud = 50
surfGui.AlwaysOnTop = false
surfGui.Parent = part
```

---

## Fonts Reference

```lua
-- Modern (most used):
Enum.Font.GothamBlack   -- ultra bold headlines
Enum.Font.GothamBold    -- bold, buttons, titles
Enum.Font.Gotham        -- regular body text
Enum.Font.GothamLight   -- light weight

-- Classic:
Enum.Font.SourceSansBold
Enum.Font.SourceSans
Enum.Font.Arial
Enum.Font.ArialBold

-- Stylized:
Enum.Font.Bangers          -- comic/action style
Enum.Font.FredokaOne       -- rounded, friendly
Enum.Font.Merriweather     -- serif, formal
Enum.Font.Oswald           -- condensed bold
Enum.Font.PermanentMarker  -- handwritten
Enum.Font.RobotoCondensed
Enum.Font.Sarpanch         -- aggressive/tech

-- Monospace:
Enum.Font.RobotoMono
Enum.Font.Code             -- code editor style

-- Custom fonts (2023+):
-- Upload .ttf to Roblox, use asset ID:
label.FontFace = Font.new("rbxassetid://123", Enum.FontWeight.Bold)
```

---

## Common GUI Animation Patterns

### Popup Open/Close
```lua
local function openPopup(panel: Frame)
    panel.Parent.Enabled = true
    panel.Position = UDim2.new(0.5, 0, 1.5, 0)
    panel.AnchorPoint = Vector2.new(0.5, 0.5)
    TweenService:Create(panel, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Position = UDim2.new(0.5, 0, 0.5, 0)
    }):Play()
end

local function closePopup(panel: Frame, screenGui: ScreenGui)
    local tween = TweenService:Create(panel,
        TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {
        Position = UDim2.new(0.5, 0, 1.5, 0)
    })
    tween.Completed:Connect(function()
        screenGui.Enabled = false
    end)
    tween:Play()
end
```

### Fade In/Out
```lua
local function fadeIn(gui: ScreenGui)
    gui.Enabled = true
    for _, obj in ipairs(gui:GetDescendants()) do
        if obj:IsA("GuiObject") then
            obj.BackgroundTransparency = 1
            if obj:IsA("TextLabel") or obj:IsA("TextButton") then
                obj.TextTransparency = 1
            end
        end
    end
    TweenService:Create(gui.Frame, TweenInfo.new(0.3), {
        BackgroundTransparency = 0
    }):Play()
end
```

### Counter Animation (number ticking up)
```lua
local function animateCounter(label: TextLabel, from: number, to: number, duration: number)
    local startTime = tick()
    task.spawn(function()
        while tick() - startTime < duration do
            local t = (tick() - startTime) / duration
            local value = math.floor(from + (to - from) * t)
            label.Text = tostring(value)
            task.wait()
        end
        label.Text = tostring(to)
    end)
end
```

### Progress Bar
```lua
local function updateProgressBar(bar: Frame, progress: number)
    -- progress = 0 to 1
    TweenService:Create(bar, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
        Size = UDim2.new(progress, 0, 1, 0)
    }):Play()
end

-- Usage: updateProgressBar(progressFill, 0.75)  -- 75% full
```

### Shake Animation
```lua
local function shake(frame: Frame, intensity: number, duration: number)
    local origin = frame.Position
    local endTime = tick() + duration
    task.spawn(function()
        while tick() < endTime do
            local x = math.random(-intensity, intensity)
            local y = math.random(-intensity, intensity)
            frame.Position = origin + UDim2.fromOffset(x, y)
            task.wait(0.05)
        end
        frame.Position = origin
    end)
end
```

---

## RichText Reference

```lua
label.RichText = true

-- Tags:
label.Text = "<b>Bold</b>"
label.Text = "<i>Italic</i>"
label.Text = "<u>Underline</u>"
label.Text = "<s>Strikethrough</s>"
label.Text = '<font color="rgb(255,100,100)">Red text</font>'
label.Text = '<font size="24">Big text</font>'
label.Text = '<font face="GothamBold">Custom font</font>'
label.Text = "Normal <b>bold</b> and <i>italic</i> mixed"

-- Escape literal < and >:
label.Text = "&lt;not a tag&gt;"
```

---

## Responsive UI (Mobile vs PC)

```lua
local UserInputService = game:GetService("UserInputService")

local isMobile = UserInputService.TouchEnabled and not UserInputService.MouseEnabled

-- Scale UI for mobile:
if isMobile then
    frame.Size = UDim2.new(1, -20, 0, 60)  -- wider on mobile
    btn.Size   = UDim2.new(0, 0, 0, 52)    -- bigger touch targets
else
    frame.Size = UDim2.new(0, 400, 0, 50)
    btn.Size   = UDim2.new(0, 0, 0, 36)
end

-- Screen size detection:
local viewport = workspace.CurrentCamera.ViewportSize
local isSmallScreen = viewport.X < 600

-- UIScale for global scaling:
local uiScale = Instance.new("UIScale")
if isSmallScreen then
    uiScale.Scale = 0.85
end
uiScale.Parent = screenGui
```

---

## StarterGui:SetCore() — System UI Control

```lua
-- Only works on CLIENT:
local StarterGui = game:GetService("StarterGui")

-- Hide default UI elements:
StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.Health, false)      -- health bar
StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.Backpack, false)    -- hotbar
StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.Chat, false)        -- chat
StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.PlayerList, false)  -- tab list
StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.EmotesMenu, false)
StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.All, false)         -- all at once

-- Send notification (top-right toast):
StarterGui:SetCore("SendNotification", {
    Title    = "Achievement!",
    Text     = "You found a rare item!",
    Duration = 5,
    Icon     = "rbxassetid://..."  -- optional
})
```

---

## GUI Performance Tips

```lua
-- 1. Use BackgroundTransparency=1 on containers, not colored fills
--    (colored fills = more draw calls)

-- 2. Avoid ClipsDescendants unless needed (expensive)

-- 3. Minimize nested Frames (max 3-4 levels deep)

-- 4. Don't animate more than 5-10 elements simultaneously

-- 5. Reuse TweenInfo objects (don't create new ones every frame):
local BOUNCE_INFO = TweenInfo.new(0.15, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
-- use BOUNCE_INFO everywhere instead of creating new

-- 6. Disable ScreenGui when not visible:
screenGui.Enabled = false  -- stops all rendering for that gui

-- 7. ImageLabel > Frame with UIGradient for complex backgrounds
--    Pre-render gradients as images, upload, use ImageLabel

-- 8. TextScaled=true + UITextSizeConstraint is better than fixed TextSize
--    for responsive layouts

-- 9. Use AutomaticSize on Frames containing dynamic content:
frame.AutomaticSize = Enum.AutomaticSize.Y  -- auto-height
-- Enum.AutomaticSize.X, Y, or XY
```
