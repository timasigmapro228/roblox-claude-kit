# Playtest Checklist

Run through this before publishing any update.

---

## Pre-Playtest (Studio)

- [ ] No red errors in Output window on Play
- [ ] All RemoteEvents exist in ReplicatedStorage/Shared/RemoteEvents/
- [ ] Main.server.lua requires all systems in correct dependency order
- [ ] Main.client.lua requires all client systems
- [ ] DataStore API Services enabled (Game Settings → Security)

---

## Core Loop (in-game)

- [ ] Player spawns correctly
- [ ] Tutorial triggers for new players
- [ ] First action completable within 60 seconds
- [ ] First reward granted correctly
- [ ] HUD updates when currency changes
- [ ] Upgrade shop opens and closes without errors
- [ ] Upgrade purchase deducts correct amount
- [ ] Upgrade effect is visible/felt immediately

---

## Data Integrity

- [ ] Leave and rejoin — data persists correctly
- [ ] Coins correct after rejoin
- [ ] Upgrade levels correct after rejoin
- [ ] No duplicate rewards on rejoin
- [ ] BindToClose fires (test by stopping server mid-game in Studio)

---

## Security (Server)

- [ ] All OnServerEvent handlers validate argument types
- [ ] Currency only granted server-side
- [ ] RemoteEvent spam doesn't crash server (rate limiting in place)
- [ ] Player can't trigger another player's NPC interaction
- [ ] Player can't collect another player's order result

---

## UI

- [ ] HUD visible and correct on join
- [ ] Popups open and close correctly (no stuck UI)
- [ ] Tween animations complete (nothing gets stuck mid-animation)
- [ ] ScrollingFrame scrolls correctly
- [ ] All buttons have click feedback (press scale animation)
- [ ] UI readable on mobile (test with small viewport)
- [ ] No UI elements cut off at screen edges

---

## NPCs & Orders (Active Simulator)

- [ ] NPC dialogue appears on interaction
- [ ] Order assigned correctly (server-side via OrderSystem.giveOrder)
- [ ] Machine processes order in correct time
- [ ] Rarity result rolls and displays correctly
- [ ] Coins granted on order completion
- [ ] Collection index updates on new discovery

---

## Performance

- [ ] FPS stable at 60 in Studio play mode
- [ ] Part count checked (Explorer → right-click Workspace → select)
- [ ] No infinite loops without task.wait()
- [ ] Heartbeat callbacks have rate limiting if doing expensive work
- [ ] Particle emitters disabled when not in use

---

## Mobile

- [ ] All buttons reachable with thumb (not in corners)
- [ ] Touch targets minimum 44px
- [ ] No keyboard-only interactions
- [ ] ProximityPrompt visible and reachable on mobile
- [ ] UI scales correctly on 375px wide screen

---

## Known Limitations (document these)

List any known issues that are not yet fixed:

```
[ ] Issue: ...
    Workaround: ...
    Fix planned: ...
```
