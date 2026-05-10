---
name: roblox-ui-styles
description: Use when generating any Roblox UI or ScreenGui. Provides style presets (Simulator, Flat, Anime, RPG) with exact colors, fonts, spacing, and stroke rules so Claude generates consistent, production-quality UI — not generic AI output.
---

# UI Style Presets

The reason AI-generated UIs look generic is because there's no design system behind them.
Before generating any UI, pick a preset and follow it exactly.

---

## How to Use

Tell Claude which style you want:
```
Build a shop UI using the Simulator style.
Build an inventory using the Flat style.
Build a leaderboard using the Anime style.
```

Or describe your existing UI and Claude will match it (Adaptive).

---

## Preset: Simulator (Stud)

Inspired by popular simulators. Bold, chunky, colorful. High contrast.

```lua
local Style = {
    -- Colors
    Background    = Color3.fromRGB(30, 30, 46),    -- dark navy
    Surface       = Color3.fromRGB(49, 49, 70),    -- card background
    SurfaceAlt    = Color3.fromRGB(60, 60, 85),    -- hover/alt card
    Primary       = Color3.fromRGB(108, 92, 231),  -- purple accent
    PrimaryDark   = Color3.fromRGB(80, 65, 200),   -- button pressed
    Success       = Color3.fromRGB(85, 239, 196),  -- green confirm
    Danger        = Color3.fromRGB(255, 118, 117), -- red danger
    Coin          = Color3.fromRGB(253, 203, 110), -- yellow coins
    Gem           = Color3.fromRGB(116, 185, 255), -- blue gems
    Text          = Color3.fromRGB(255, 255, 255),
    TextMuted     = Color3.fromRGB(180, 180, 200),

    -- Typography
    FontTitle     = Enum.Font.GothamBlack,   -- headers, big labels
    FontBold      = Enum.Font.GothamBold,    -- buttons, card titles
    FontBody      = Enum.Font.Gotham,        -- descriptions, muted text

    -- Sizing
    TitleSize     = 22,
    ButtonSize    = 16,
    BodySize      = 13,
    MutedSize     = 11,

    -- Borders & Corners
    CornerCard    = 14,   -- UDim.new(0, 14) for cards
    CornerButton  = 10,   -- UDim.new(0, 10) for buttons
    CornerSmall   = 6,    -- UDim.new(0, 6) for small elements
    StrokeThick   = 2,    -- UIStroke thickness
    StrokeColor   = Color3.fromRGB(255, 255, 255), -- stroke at 0.85 transparency

    -- Padding
    PadCard       = 12,   -- UIPadding inside cards
    PadButton     = UDim2.new(0, 8, 0, 36),  -- standard button size

    -- Effects
    BackdropTransp = 0.4,  -- semi-transparent black backdrop for popups
}
```

**Rules for Simulator style:**
- Cards have thick rounded corners (14px) + thin white stroke (transparency 0.85)
- Buttons always have gradient (slightly lighter top to darker bottom)
- Icons are large (48-64px), centered above text
- Coin/gem amounts always bold + matching color
- Panels slide in with Back easing (bouncy)

---

## Preset: Flat

Clean, modern, minimal. Light backgrounds, golden accents. Used in casual/idle games.

```lua
local Style = {
    Background    = Color3.fromRGB(245, 245, 250),
    Surface       = Color3.fromRGB(255, 255, 255),
    SurfaceAlt    = Color3.fromRGB(235, 235, 245),
    Primary       = Color3.fromRGB(255, 193, 7),   -- golden yellow
    PrimaryDark   = Color3.fromRGB(230, 170, 0),
    Success       = Color3.fromRGB(40, 200, 120),
    Danger        = Color3.fromRGB(220, 60, 60),
    Text          = Color3.fromRGB(30, 30, 46),    -- dark on light bg
    TextMuted     = Color3.fromRGB(120, 120, 140),

    FontTitle     = Enum.Font.GothamBold,
    FontBold      = Enum.Font.GothamBold,
    FontBody      = Enum.Font.Gotham,

    TitleSize     = 20,
    ButtonSize    = 15,
    BodySize      = 13,

    CornerCard    = 16,
    CornerButton  = 8,
    StrokeThick   = 1,
    StrokeColor   = Color3.fromRGB(200, 200, 215),
    BackdropTransp = 0.6,
}
```

**Rules for Flat style:**
- White/light surfaces, NOT dark backgrounds
- Thin 1px strokes in light grey
- Shadows via drop shadow or dark stroke at low transparency
- Text is dark (not white) — reverse contrast
- No gradients on buttons — solid fill only
- Animations use Quad easing (smooth, not bouncy)

---

## Preset: Anime

Dark panels, neon accents, thick outlines. For anime/RPG/fighting games.

```lua
local Style = {
    Background    = Color3.fromRGB(10, 10, 20),
    Surface       = Color3.fromRGB(20, 20, 40),
    SurfaceAlt    = Color3.fromRGB(30, 30, 55),
    Primary       = Color3.fromRGB(255, 50, 120),   -- hot pink/red
    Secondary     = Color3.fromRGB(80, 200, 255),   -- cyan accent
    Success       = Color3.fromRGB(80, 255, 150),
    Danger        = Color3.fromRGB(255, 80, 80),
    Text          = Color3.fromRGB(255, 255, 255),
    TextMuted     = Color3.fromRGB(160, 160, 200),
    GlowColor     = Color3.fromRGB(255, 50, 120),   -- for UIGradient effects

    FontTitle     = Enum.Font.GothamBlack,
    FontBold      = Enum.Font.GothamBold,
    FontBody      = Enum.Font.Gotham,

    TitleSize     = 24,
    ButtonSize    = 16,
    BodySize      = 13,

    CornerCard    = 6,    -- sharper corners for anime style
    CornerButton  = 4,
    StrokeThick   = 3,    -- thick neon strokes
    StrokeColor   = Color3.fromRGB(255, 50, 120),  -- colored stroke matches Primary
    BackdropTransp = 0.3,
}
```

**Rules for Anime style:**
- Thick colored strokes (3px), matching primary color
- Gradient backgrounds on cards (dark top → slightly lighter bottom)
- Neon glow effect: UIStroke + same color frame behind at low transparency
- Sharp corners (4-6px), NOT round
- Titles often have gradient coloring via UIGradient on TextLabel
- Entry animations use Elastic or Spring easing

---

## Preset: RPG / Fantasy

Warm, parchment-like. Browns, golds, deep reds. For RPGs, medieval, adventure games.

```lua
local Style = {
    Background    = Color3.fromRGB(25, 15, 10),
    Surface       = Color3.fromRGB(45, 28, 18),
    SurfaceAlt    = Color3.fromRGB(60, 38, 24),
    Primary       = Color3.fromRGB(212, 175, 55),  -- gold
    PrimaryDark   = Color3.fromRGB(170, 135, 30),
    Success       = Color3.fromRGB(100, 180, 80),
    Danger        = Color3.fromRGB(180, 50, 50),
    Text          = Color3.fromRGB(240, 220, 180), -- warm white
    TextMuted     = Color3.fromRGB(170, 145, 110),

    FontTitle     = Enum.Font.Merriweather,  -- or Cinzel if available
    FontBold      = Enum.Font.GothamBold,
    FontBody      = Enum.Font.Gotham,

    TitleSize     = 22,
    ButtonSize    = 15,
    BodySize      = 13,

    CornerCard    = 4,    -- slightly rounded, parchment-like
    CornerButton  = 6,
    StrokeThick   = 2,
    StrokeColor   = Color3.fromRGB(212, 175, 55),  -- gold borders
    BackdropTransp = 0.5,
}
```

**Rules for RPG style:**
- Gold strokes/borders everywhere
- Warm text color (not pure white)
- Panels feel heavy and solid, not light
- Use ImageLabels with parchment/wood textures from Toolbox for panel backgrounds
- Dividers between sections (thin gold line, 1-2px height Frame)

---

## Adaptive Mode

When the user already has UI in their game:

```
"Match the style of my existing UI. Here's a screenshot: [image]"
```

Claude should:
1. Identify background color, surface color, accent color from the image
2. Identify font weight (bold/regular), corner radius (sharp/round), stroke presence
3. Build new UI matching those exact values
4. Ask: "Does this match your existing style?" before finalizing

---

## Universal UI Rules (all styles)

These apply regardless of preset:

**Layout:**
- Popups: centered, AnchorPoint (0.5, 0.5), max width 520px on desktop
- ScrollFrames: always `ScrollBarThickness = 4`, `CanvasSize` auto-calculated
- Buttons: minimum 36px height, never smaller
- Touch targets on mobile: minimum 44px

**Animations:**
- Popups open: slide from below + fade in, 0.25-0.35s
- Popups close: fade out, 0.2s
- Button press: scale to 0.95, 0.08s, then back
- Currency update: bounce label size +4px, 0.1s

**Accessibility:**
- Never rely on color alone to convey state (add icon or text)
- Minimum TextSize 11 for any readable label
- Contrast: text color must be readable against background

**Performance:**
- Use `BackgroundTransparency = 1` on container Frames (not colored)
- Avoid more than 3 levels of nested Frames where possible
- ClipsDescendants only where needed (ScrollFrames)

---

## Prompt Templates by Style

```
-- Simulator style shop:
"Build a shop popup UI using the Simulator style from the kit.
It should have: title 'UPGRADES', close button, scrollable grid of upgrade cards.
Each card: icon (64px), name, level indicator, buy button with coin cost.
Inject into StarterGui as 'ShopGui'."

-- Flat style HUD:
"Build a HUD using the Flat style. Show: coins (top left), gems (top right),
a settings button (top right corner). Light background bar, dark text, golden accents."

-- Anime style notification:
"Build a toast notification system using the Anime style.
Notifications slide in from the top, stay 2.5s, slide out.
Use the hot pink primary color for the stroke. Support 'success', 'error', 'rare' types."
```
