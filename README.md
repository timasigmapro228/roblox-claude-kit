# roblox-claude-kit 🎮

> **A free, open-source Claude Code kit for building Roblox games.**
> Not a plugin. Not a product. An architecture brain for Claude Code.

Production-oriented Roblox architecture patterns for Claude Code.

---

## What is this?

A **Claude Code kit** - structured instructions, patterns, and ready-to-use Luau code that Claude reads as project context when building Roblox games. Instead of explaining Roblox architecture every time, Claude just knows it.

**Best used with:**
- Claude Code
- Roblox Studio MCP Server (for direct Studio integration)
- Rojo (optional, for file-based workflow)

**This is not:**
- A Roblox Studio plugin
- A drop-in replacement for Roblox AI plugins - it's a different tool for a different workflow

**This is:**
- Architecture rules Claude follows automatically
- Production-oriented Luau patterns (DataStore, RemoteEvents, security)
- Genre templates (simulator, tycoon, obby, pet sim, active sim)
- UI system with 4 style presets + complete GUI reference
- Full Roblox API cheat sheet
- MCP workflow guide for Studio integration

Think of it as a senior Roblox developer's playbook that Claude reads before writing any code.

---

## What's included

| File | What it gives Claude |
|---|---|
| `SKILL.md` | Core rules, architecture, project structure |
| `systems/currency.md` | Coins, gems, multi-currency |
| `systems/orders.md` | NPC orders, processing, rarity rolls |
| `systems/inventory.md` | Grid inventory, item slots |
| `systems/pets.md` | Pet system with rarity and merge |
| `systems/shop.md` | Buy/sell with server validation |
| `systems/daily-reward.md` | Streak-based daily rewards |
| `ui/hud.md` | Always-visible HUD |
| `ui/shop.md` | Shop popup with upgrade cards |
| `ui/notification.md` | Toast notifications + rarity flash |
| `patterns/modulescript.md` | Architecture and dependency rules |
| `patterns/remoteevents.md` | Client-server security patterns |
| `patterns/datastore.md` | Safe save/load with retry logic |
| `examples/dream-laundromat/` | Example project blueprint using the kit |

## Quick Start

### Option 1: Claude Code + MCP (recommended)

```bash
# Clone into your Roblox project folder
git clone https://github.com/timasigmapro228/roblox-claude-kit ./roblox-claude-kit
```

Add to your project's `CLAUDE.md`:
```markdown
Use ./roblox-claude-kit as the Roblox architecture kit.
Read roblox-claude-kit/SKILL.md before implementing any Roblox system.
When connected to Studio via MCP, follow roblox-claude-kit/mcp/studio-mcp-workflow.md.
```

Then in Claude Code:
```
Build me a currency system for my Roblox simulator
```

### Option 2: Claude Code without MCP

Clone as above, then reference files manually in prompts:
```
Read roblox-claude-kit/genres/simulator.md and implement the click system for my game
```

### Optional: Adapt as a Claude Skill

This kit is designed for Claude Code first. It may be adapted into a Claude.ai Skill, but Skill packaging should be tested separately for your use case:
1. Download this repo as a ZIP
2. Go to **Claude.ai → Settings → Features → Skills**
3. Upload the ZIP and verify it loads correctly

## Example Prompts

Once loaded, try:

```
Build a DataStore system for my Roblox game using the kit patterns
```

```
Create an NPC order system with 3 dream types and rarity rolls
```

```
Build a shop UI with upgrade cards for Speed, Value, and Luck upgrades
```

```
Set up the full project folder structure for a new simulator game
```

```
Add a notification system that shows when players get a rare result
```

## Showcase: Example Projects

The `examples/` folder contains project blueprints showing how to combine the kit's systems and patterns into complete game concepts.

## Architecture Philosophy

- **Server is authoritative** — clients request, server validates, server acts
- **Every system is a ModuleScript** — no logic in standalone Scripts
- **DataStore is always safe** — pcall, retry, BindToClose
- **RemoteEvents are always validated** — type check every argument
- **Configs are shared** — both sides can read, neither side stores state

## Contributing

PRs welcome. To add a new system:

1. Create `systems/your-system.md` or `ui/your-ui.md`
2. Follow the existing pattern (config → server → client → usage examples)
3. Add it to the table in `SKILL.md`
4. Test with a real Claude Code session

## Note on SKILL.md

`SKILL.md` is the main instruction file for Claude Code - it tells Claude when and how to use the kit. This repository is a **Claude Code kit first**. It may be adapted as a Claude.ai Skill, but that packaging should be tested separately for your use case.

## Known Limitations

- **MCP dependency**: The MCP workflow requires a Roblox Studio MCP server (e.g. `roblox-studio-mcp`). Setup varies by plugin — check the specific MCP server's docs.
- **No live assets**: The kit provides code patterns and configs. 3D models, textures, and sounds must be sourced from Roblox Toolbox or created separately.
- **Genre templates are starting points**: The genre files give you the core loop. Production games need additional polish, balancing, and iteration.
- **Not tested in all Studio versions**: Luau APIs evolve — always check the deprecated API list in `roblox-api-reference.md`.

## License

MIT — free to use, modify, and distribute.
Build whatever you want. Ship it. Make money. No credit required (but appreciated).

---

*Made for Roblox developers who want Claude to actually understand Roblox.*
