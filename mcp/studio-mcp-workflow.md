---
name: roblox-mcp-workflow
description: Use when Claude Code is connected to Roblox Studio via MCP. Covers how to inspect the scene, create instances, patch scripts, run playtests, and debug — all through MCP tool calls instead of copy-paste.
---

# Roblox Studio MCP Workflow

How to use Claude Code + Roblox Studio MCP to build games without copy-pasting code.

## Setup

```
Required:
1. Roblox Studio MCP Server (roblox-studio-mcp or similar)
2. Claude Code with MCP configured
3. Roblox Studio open with your place

Connection:
- MCP server runs locally, listens on a port
- Claude Code connects via ~/.claude/mcp.json or CLAUDE.md config
- Studio plugin connects to same port
```

---

## Core MCP Operations

### 1. Inspect the Scene First

Before writing any code, always inspect what exists:

```
"Show me the current Workspace hierarchy"
"List all scripts in ServerScriptService"
"What ModuleScripts exist in ReplicatedStorage/Shared?"
"Show me the properties of the Part named 'DreamWasher'"
```

Claude reads the actual scene — not guesses.

### 2. Create Instances

```
"Create a RemoteEvent named 'PickupOrder' in ReplicatedStorage/Shared/RemoteEvents"
"Add a ProximityPrompt to the NPC model named 'Grumpy'"
"Create a ScreenGui named 'HUD' in StarterGui with ResetOnSpawn = false"
"Add a UICorner with CornerRadius 12px to the ShopPanel Frame"
```

### 3. Write & Patch Scripts

```
"Create OrderSystem.lua in ServerScriptService/Server/Systems with this code: [code]"
"Add the init() function to CurrencySystem.lua"
"Fix line 87 in DataManager.lua — wrap the GetAsync call in pcall"
"Add validation to the PickupOrder OnServerEvent handler"
```

### 4. Playtest Debug Loop

```
"Run the game and tell me what errors appear in the output"
"The HUD coins label isn't updating — check what RemoteEvents are firing"
"Player coins aren't saving on leave — trace the DataManager.PlayerRemoving path"
```

### 5. Asset Insertion

```
"Search Toolbox for 'washing machine cartoon' and insert the first free result"
"Load asset ID 12345678 and place it at position (0, 0, 10) in workspace"
"Find a suitable NPC model for a grumpy bear character"
```

---

## Recommended MCP Workflow Per Feature

### Adding a new system:

```
Step 1 — Plan (no MCP yet):
"I want to add a daily reward system. 
Read systems/retention.md and outline the modules, RemoteEvents, and data fields needed."

Step 2 — Inspect (MCP):
"Check what already exists in ServerScriptService/Server/Systems"
"Check ReplicatedStorage/Shared/RemoteEvents for existing events"

Step 3 — Create (MCP):
"Create DailyRewardSystem.lua in Systems/ with the code from retention.md"
"Create RemoteEvent 'DailyReward' in RemoteEvents/"

Step 4 — Wire (MCP):
"Add DailyRewardSystem to Main.server.lua — require it and call .init()"

Step 5 — Test (MCP):
"Run a playtest and check if DailyReward fires correctly on join"
```

### Fixing a bug:

```
"Read the error: ServerScriptService.OrderSystem:87: attempt to index nil value 'data'"
"Show me line 87 of OrderSystem.lua"
"The issue is getData() returning nil — check if DataManager.init() is called before OrderSystem.init() in Main.server.lua"
"Fix the init order in Main.server.lua"
```

---

## MCP-Specific Prompt Patterns

### Scene inspection prompts:
```
"Before writing any code, show me what's in [folder]"
"List all [ClassName] instances in the game"
"What properties does [Instance] have?"
"Show me the full hierarchy under [Model]"
```

### Instance creation prompts:
```
"Create [InstanceClass] named '[Name]' parented to [Path]"
"Set [Property] = [Value] on [Instance]"
"Clone [Instance] and place it at [Position]"
```

### Script patching prompts:
```
"Open [ScriptPath] and show me lines [X]-[Y]"
"Replace the [FunctionName] function in [Script] with this version:"
"Add this code after line [N] in [Script]:"
"Find all references to [deprecated API] and replace with [new API]"
```

### Debug prompts:
```
"Run the game and capture output for 10 seconds"
"Fire the [RemoteEvent] from the client and show what the server receives"
"Check if [Player] data is loading correctly — add a print to DataManager"
"Profile the Heartbeat loop — is anything running too frequently?"
```

---

## MCP + Kit Integration

When using the kit with MCP, reference kit files directly:

```
"Read patterns/datastore.md and implement that exact pattern for PlayerData"
"Follow the UI style from ui/styles.md Simulator preset for this shop"
"Use the tycoon pattern from genres/tycoon.md — inspect workspace first to see what's already placed"
```

Claude reads the kit file, then reads the actual Studio scene, then writes code that fits both.

---

## Common MCP Gotchas

```
❌ Don't ask Claude to "create the whole game" in one prompt via MCP
   → Too many operations, context gets confused

✅ One feature at a time, verify each step before proceeding

❌ Don't assume MCP can read scripts that aren't open in Studio
   → Explicitly ask "open and read [ScriptPath]"

✅ Always inspect before creating: "does X already exist?"

❌ Don't use MCP to set properties that require Studio UI (lighting modes, etc.)
   → Do those manually, tell Claude you've done it

✅ Use MCP for: instance creation, script writing, hierarchy inspection, property setting
```

---

## Best MCP Setup for This Kit

### CLAUDE.md in project root:
```markdown
# Roblox Project

## Kit
This project uses roblox-claude-kit. Read SKILL.md for architecture rules.

## MCP
Connected to Roblox Studio via MCP. Always inspect the scene before creating instances.

## Structure
- Server systems: ServerScriptService/Server/Systems/
- Client systems: StarterPlayerScripts/Client/Systems/
- Shared: ReplicatedStorage/Shared/
- RemoteEvents: ReplicatedStorage/Shared/RemoteEvents/

## Rules
- Always read the relevant kit file before implementing a system
- Always check what exists before creating new instances
- Always validate RemoteEvent arguments on the server
- Always wrap DataStore calls in pcall
```
