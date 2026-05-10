# Dream Laundromat — Example Project Blueprint

This is a **game concept blueprint** showing how to combine roblox-claude-kit systems into a complete game.
It is not a fully implemented reference — it shows design decisions, system combinations, and config structure.

Use it as a starting point for your own active simulator game.

## Project Overview

- **Genre**: Active Simulator (NOT a tycoon)
- **Core loop**: Take dream → Wash dream → Get rare result → Return to NPC → Earn coins → Upgrade
- **Hook**: "NPCs bring you broken dreams. Fix them before morning."
- **Tone**: Cute, surreal, cozy, soft horror-comedy

## Week 1 Scope (MVP)

- 1 NPC customer
- 1 Dream Washer machine
- 3 dirty dream types
- 3 rarity results
- Dream Coins currency
- Reputation stat
- 3 upgrades: Speed, Value, Luck
- Basic HUD
- Basic tutorial
- Dream Index (collection book)

## Week 2 Features

- Nightmare Orders (2-step cleaning)
- Mini-interaction inside wash (tap at right moment for bonus)
- NPC personalities and dialogue
- Daily customer system
- Cosmetic dream effects

## Systems Used from Kit

| System | Kit File |
|---|---|
| Dream Coins | `systems/currency.md` |
| Dream orders + NPC | `systems/orders.md` |
| DataStore | `patterns/datastore.md` |
| RemoteEvents | `patterns/remoteevents.md` |
| HUD | `ui/hud.md` |
| Upgrade shop | `ui/shop.md` |
| Notifications | `ui/notification.md` |

## NPC Dialogue Template

Each NPC has a personality. Use these states:

```lua
-- NPCDialogue.lua (ReplicatedStorage/Shared/Modules/)
local NPCDialogue = {}

NPCDialogue.PillowBear = {
    name = "Pillow Bear",
    idle = {
        "Excuse me... I have a dream emergency.",
        "My dream got all tangled up last night.",
        "Do you think you can fix it? It's a bit... muddy.",
    },
    orderGiven = {
        "Please be careful with it. It's my favorite dream.",
        "I'll wait right here. I'm very good at waiting.",
        "It had a flying toaster in it. Very important.",
    },
    waiting = {
        "...",
        "Is it done yet?",
        "I can smell the dream soap from here.",
    },
    resultCommon = {
        "Oh! It's clean. Thank you.",
        "Much better. It was smelling like Tuesday.",
    },
    resultRare = {
        "Oh WOW. It's sparkling!",
        "I didn't know dreams could be this shiny!",
    },
    resultEpic = {
        "IT'S PERFECT. I'm going to frame it.",
        "This is the best dream I've ever had and I haven't even had it yet.",
    },
}

return NPCDialogue
```

## Map Layout

```
[Entrance / NPC waiting area]  ←  NPCs spawn here, player picks up orders

[Washer Row]  ←  1-3 Dream Washer machines

[Return Counter]  ←  player returns completed dreams to NPCs here

[Upgrade Shop]  ←  bulletin board or vending machine UI trigger

[Dream Index Wall]  ←  glowing display showing collected dreams
```

## Washer Interaction Flow

```
Player walks to washer with order →
  Proximity prompt appears →
  Player presses E →
  Dream capsule enters washer (animation) →
  Washer runs (progress bar in HUD) →
  Washer dings, glows →
  Player picks up result →
  Player walks to Return Counter →
  Proximity prompt appears →
  Player presses E →
  Result given to NPC, coins fly out →
  NPC says result dialogue →
  Order complete
```

## Color Palette (Dream Laundromat specific)

```lua
-- Extends the kit's universal Theme
local DreamTheme = {
    -- Base from kit
    Primary    = Color3.fromRGB(108, 92, 231),   -- purple (main brand)
    Secondary  = Color3.fromRGB(253, 203, 110),  -- yellow (coins)
    Background = Color3.fromRGB(20, 18, 36),     -- deep night blue

    -- Dream Laundromat specific
    WasherGlow   = Color3.fromRGB(100, 200, 255), -- blue washer light
    DreamOrb     = Color3.fromRGB(200, 180, 255), -- soft lavender
    NightmareOrb = Color3.fromRGB(255, 100, 150), -- pinkish red (not scary)
    SoapBubble   = Color3.fromRGB(220, 240, 255), -- near white blue
    NeonSign     = Color3.fromRGB(255, 220, 100), -- warm yellow neon
}
```

## Audio Suggestions

| Event | Sound |
|---|---|
| Washer starts | Soft hum + bubble sounds |
| Washer done | Pleasant bell/chime |
| Common result | Soft pop |
| Rare result | Sparkle sound |
| Epic result | Dream harp glissando |
| Coins earned | Coin jingle |
| NPC dialogue | Soft character voice bubble |
| Ambient | Slow lofi / dream jazz |
