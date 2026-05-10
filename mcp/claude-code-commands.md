# Claude Code Commands — Roblox + MCP

Copy-paste prompts for common tasks. Works best with Studio MCP connected.

---

## 🏗️ Project Setup

```
Read roblox-claude-kit/SKILL.md and roblox-claude-kit/patterns/modulescript.md.
Then inspect the current ServerScriptService and ReplicatedStorage hierarchy.
Create the standard folder structure:
- ServerScriptService/Server/Systems/
- ServerScriptService/Server/Main.server.lua
- StarterPlayerScripts/Client/Systems/
- StarterPlayerScripts/Client/Main.client.lua
- ReplicatedStorage/Shared/Modules/
- ReplicatedStorage/Shared/RemoteEvents/
```

---

## 💰 Add Currency System

```
Read roblox-claude-kit/systems/currency.md.
Inspect ReplicatedStorage/Shared/RemoteEvents/ to see what already exists.
Implement the full CurrencySystem — server module, client controller, and UpdateCurrency RemoteEvent.
Wire it into Main.server.lua and Main.client.lua.
Tell me exactly how to test it works.
```

---

## 🗃️ Add DataStore

```
Read roblox-claude-kit/patterns/datastore.md.
Implement DataManager.lua in ServerScriptService/Server/Systems/.
Use this DEFAULT_DATA: [paste your data structure]
Add it as the first system in Main.server.lua (must init before all others).
Verify BindToClose is included.
```

---

## 🧑 Add NPC with Orders

```
Read roblox-claude-kit/systems/npc.md and roblox-claude-kit/systems/orders.md.
Inspect workspace to see what NPC models exist.
NPC asset ID: [paste ID]
NPC spawn position: [paste Vector3]
Item types for orders: [paste list]

Implement NPCSystem and OrderSystem. Wire both into Main.server.lua.
NPCSystem must call OrderSystem.giveOrder() server-side on interaction.
List all RemoteEvents that need to be created.
```

---

## 🎨 Build Shop UI

```
Read roblox-claude-kit/ui/shop.md and roblox-claude-kit/ui/styles.md.
Style preset: [Simulator / Flat / Anime / RPG]
Upgrades to show: [Speed, Value, Luck — or custom list]

Build the ShopPopup ScreenGui in StarterGui.
Backdrop must be TextButton (not Frame) so clicking outside closes the popup.
ShopController.close() must use a single tween variable with .Completed before .Play().
Wire the shop button in HUD to ShopController.open().
```

---

## 🎯 Add Genre Template

```
Read roblox-claude-kit/genres/[simulator/tycoon/obby/pet-simulator/active-simulator].md.
Inspect the current workspace to see what's already placed.
Implement the core loop for this genre using the patterns in that file.
Start with the config file, then the server system, then wire into Main.server.lua.
Do NOT write the UI yet — confirm the server logic works first.
```

---

## 🐛 Debug an Error

```
I'm getting this error in Studio output:
[paste full error with line number]

Open [ScriptName] and show me lines [X]-[Y].
Identify the cause. Check if it's one of these common issues:
- getData() returning nil (DataManager not yet loaded)
- RemoteEvent argument type mismatch
- DataStore call missing pcall
- Circular require between modules
Fix it and explain what was wrong.
```

---

## 🔒 Security Audit

```
Read roblox-claude-kit/patterns/remoteevents.md.
Inspect all OnServerEvent handlers in ServerScriptService/Server/Systems/.
Check each one for:
1. typeof() validation on all arguments
2. Range/sanity checks on numbers
3. Whitelist check on string IDs
4. No client-side reward grants

List every handler that fails any check and fix them.
```

---

## 📊 Performance Check

```
Read roblox-claude-kit/patterns/performance.md.
Inspect the workspace part count.
Check all Heartbeat/RunService connections for rate limiting.
Check particle emitters — are any running at high rates?
List the top 3 performance risks in this project and how to fix them.
```

---

## 💾 Save System Audit

```
Read roblox-claude-kit/patterns/datastore.md.
Open DataManager.lua and check:
1. Is every DataStore call wrapped in pcall?
2. Is BindToClose implemented?
3. Is PlayerRemoving implemented?
4. Is there retry logic?
5. Is data reconciled against DEFAULT_DATA for missing keys?
Fix anything that's missing.
```

---

## 📱 Mobile UI Audit

```
Inspect all ScreenGui elements in StarterGui.
Check every interactive element (TextButton, ProximityPrompt, ImageButton) for:
- Minimum 44px touch target size
- Not positioned in screen corners (< 20px from edge)
- No keyboard-only interactions
List what needs fixing and apply the fixes.
```

---

## 🚀 Pre-Publish Checklist

```
Read roblox-claude-kit/testing/playtest-checklist.md.
Work through each section and tell me what passes and what fails.
For each failure, either fix it or explain what I need to do manually.
```
