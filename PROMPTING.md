# How to Prompt Claude for Roblox Dev

Inspired by small-step, test-driven Roblox Studio workflows. Follow these to get production-quality results.

---

## 1. Plan Before Building

Before writing any code, ask Claude to outline the approach.
This catches architecture issues early and leads to better structure.

**Do this first:**
```
I want to build a dream order system for my simulator.
What scripts, ModuleScripts, and RemoteEvents would you recommend?
Walk me through the structure before writing any code.
```

Then after you agree on the plan — ask for the code.

---

## 2. Iterate in Small Steps

Never ask for a complete system in one prompt.
Chain small prompts where each step builds on the last.

**Example chain:**
```
Step 1: "Create a basic proximity prompt on the NPC that prints 'interacted'"
Step 2: "Now make it give the player a dream item and show it in a ScreenGui label"
Step 3: "Add a cooldown so the player can only pick up one order at a time"
Step 4: "Now save the order state so it persists if the player resets"
```

Each step is verified before the next one starts.

---

## 3. Be Specific — Always

Vague prompts produce vague code.

| ❌ Bad | ✅ Good |
|---|---|
| "Add a processing machine" | "Create a Part named 'ProcessingMachine' with a ProximityPrompt. When triggered, start an 8-second timer shown as a progress bar in a ScreenGui. On completion, fire a RemoteEvent to the server to roll rarity and grant coins." |
| "Make the shop work" | "The shop should deduct coins on the server, validate the player has enough balance, update the upgrade level in DataStore, and fire a confirmation RemoteEvent back to the client." |
| "Fix the NPC" | "The NPC dialogue label disappears after 2 seconds instead of staying visible. Here's the error: `[paste exact error]`" |

---

## 4. Define Success Criteria

Tell Claude exactly what "working" looks like.
This lets Claude verify its own output.

**Template:**
```
Build [X]. It should work like this:
1. Player does [action]
2. Server validates [condition]  
3. Client sees [result]
4. DataStore saves [data]

To test: [exact steps to verify it works in Studio]
```

**Example:**
```
Build the coin reward system. It works when:
- Player returns a cleaned dream to the NPC
- Server grants exactly (baseCoins × valueMultiplier) coins
- HUD label updates within 0.5 seconds
- Balance persists after leaving and rejoining

To test: complete one order, note coin balance, rejoin, verify balance matches.
```

---

## 5. Use Images — A Lot

Paste screenshots directly into the chat for:
- UI layouts you want recreated
- Examples from other Roblox games
- Your own rough mockups (even hand-drawn)
- Error messages from Studio output

Claude reads images. A screenshot of a shop UI gives more context than 3 paragraphs describing it.

---

## 6. Paste Errors Exactly

Never paraphrase error messages. Copy the full output including line numbers.

**Bad:**
```
"it doesn't work, says something about nil"
```

**Good:**
```
ServerScriptService.OrderSystem:87: attempt to index nil value 'data'
Stack trace:
  OrderSystem:87 - getData(player)
  OrderSystem:134 - collectResult()
```

---

## 7. Keep Scripts Modular

Never ask for one giant script. Ask for focused modules.

**Instead of:**
```
"Create the entire game system in one script"
```

**Do:**
```
"Create just the DataManager module"
→ "Now create CurrencySystem that uses DataManager"
→ "Now create OrderSystem that uses both"
```

One module at a time. Test each before moving on.

---

## 8. Ask for Explanations

When Claude makes an architectural choice, ask why.
This helps you catch issues and learn.

```
"Why did you use RemoteFunction here instead of RemoteEvent?"
"Walk me through how this rarity roll logic works."
"Why is DataManager initialized first?"
"What happens if the player leaves mid-wash?"
```

Push back if you disagree. Claude will revise.

---

## 9. Verify Security-Critical Code Yourself

Claude can hallucinate deprecated Roblox APIs.
Always manually review code that handles:

- Currency grants (`CurrencySystem.add`)
- DataStore writes
- Any `OnServerEvent` handler
- Ownership or permission checks

**Red flags to check:**
- Is every RemoteEvent argument validated on the server?
- Are DataStore calls wrapped in `pcall`?
- Does the server grant rewards, or does the client?
- Could a client send a fake `orderId` and collect someone else's order?

---

## 10. Reset Session When Stuck

If Claude keeps referencing old context or making the same mistake:
- Start a fresh conversation
- Re-paste the relevant system file from the kit
- Describe only what you're working on now

Long sessions accumulate noise. Fresh context = better code.

---

## Starter Prompt Template

Copy this at the start of any new feature:

```
I'm building [game name], a Roblox [genre] game.
Current state: [what already exists]
I'm using roblox-claude-kit for architecture patterns.

I want to add: [feature name]

Before writing code, outline:
- What scripts/modules are needed
- What RemoteEvents are needed  
- Dependency order
- How to test it works

Success looks like: [exact description]
```
