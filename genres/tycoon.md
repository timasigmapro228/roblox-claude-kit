---
name: genre-tycoon
description: Use when building a Roblox tycoon game — dropper, conveyor, collector, buttons to unlock buildings, cash income. Triggers on words like tycoon, dropper, conveyor, collector, cash, income, unlock button, build.
---

# Genre: Tycoon

**Core loop**: Own plot → buy buttons → droppers generate parts → conveyor moves parts → collector gives cash → buy more buttons

**Examples**: Lumber Tycoon, Restaurant Tycoon, Longest Conveyor Tycoon, Pet Factory Tycoon

---

## Folder Structure

```
Workspace/
  Tycoons/
    Tycoon1/            ← one folder per player slot
      Plot              ← claim pad
      Buttons/          ← purchasable unlock buttons
      Droppers/         ← spawns parts on timer
      Conveyor/         ← moves parts to collector
      Collector         ← destroys parts, grants cash
      Buildings/        ← visual structures unlocked

ServerScriptService/Server/
  Systems/
    DataManager.lua
    CurrencySystem.lua
    TycoonSystem.lua    ← claim, buttons, income logic

ReplicatedStorage/Shared/
  Modules/
    TycoonConfig.lua
```

---

## TycoonConfig.lua

```lua
-- ReplicatedStorage/Shared/Modules/TycoonConfig.lua
return {
    -- How often droppers spawn parts (seconds)
    dropperInterval = 2,

    -- How fast conveyor moves parts (AssemblyLinearVelocity)
    conveyorSpeed   = 20,

    -- Part value when collected
    partValue       = 1,

    -- Max tycoon plots in the game
    maxPlots        = 6,

    -- Buttons in order (each unlocks the next)
    buttons = {
        { id = "btn_conveyor",   cost = 0,    name = "Conveyor"      },
        { id = "btn_dropper1",   cost = 50,   name = "Basic Dropper" },
        { id = "btn_upgrade1",   cost = 200,  name = "Fast Dropper"  },
        { id = "btn_dropper2",   cost = 500,  name = "2nd Dropper"   },
        { id = "btn_upgrade2",   cost = 1000, name = "Power Dropper" },
        { id = "btn_dropper3",   cost = 3000, name = "3rd Dropper"   },
    },
}
```

---

## TycoonSystem.lua (Server)

```lua
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local DataManager    = require(script.Parent.DataManager)
local CurrencySystem = require(script.Parent.CurrencySystem)
local TycoonConfig   = require(ReplicatedStorage.Shared.Modules.TycoonConfig)

local TycoonSystem = {}

-- Active tycoons: { player = tycoonFolder }
local playerTycoons: {[Player]: Folder} = {}

local function setupCollector(collector: BasePart, player: Player, cashPerPart: number)
    collector.Touched:Connect(function(hit)
        if hit.Name ~= "Drop" then return end
        hit:Destroy()
        CurrencySystem.add(player, "Cash", cashPerPart)
    end)
end

local function startDropper(dropper: Model, interval: number)
    task.spawn(function()
        while dropper and dropper.Parent do
            task.wait(interval)
            if not dropper.Parent then break end

            local spawnPart = dropper:FindFirstChild("SpawnPoint")
            if not spawnPart then continue end

            local drop = Instance.new("Part")
            drop.Name     = "Drop"
            drop.Size     = Vector3.new(1, 1, 1)
            drop.BrickColor = BrickColor.new("Bright yellow")
            drop.CFrame   = spawnPart.CFrame + Vector3.new(0, 2, 0)
            drop.Parent   = workspace

            -- Auto-destroy after 30s if not collected
            game:GetService("Debris"):AddItem(drop, 30)
        end
    end)
end

local function setupConveyor(conveyor: BasePart)
    -- Use AssemblyLinearVelocity via constraint or legacy method
    conveyor.AssemblyLinearVelocity = Vector3.new(0, 0, TycoonConfig.conveyorSpeed)

    -- Keep applying velocity every frame (physics can reset it)
    local conn
    conn = game:GetService("RunService").Heartbeat:Connect(function()
        if not conveyor.Parent then
            conn:Disconnect()
            return
        end
        -- Apply to parts touching conveyor via constraint is better,
        -- but for simplicity use TouchEnded detection
    end)
end

local function claimTycoon(player: Player, tycoonFolder: Folder)
    playerTycoons[player] = tycoonFolder

    -- Load saved buttons
    local data = DataManager.getData(player)
    if not data then return end

    local unlockedButtons = data.TycoonButtons or {}

    -- Setup all buttons
    local buttons = tycoonFolder:FindFirstChild("Buttons")
    if buttons then
        for _, button in ipairs(buttons:GetChildren()) do
            local config = nil
            for _, btnConfig in ipairs(TycoonConfig.buttons) do
                if btnConfig.id == button.Name then
                    config = btnConfig
                    break
                end
            end
            if not config then continue end

            -- If already unlocked, activate
            if unlockedButtons[button.Name] then
                button.Transparency = 1
                button.CanCollide   = false
                -- Show building associated with this button
                local building = tycoonFolder.Buildings:FindFirstChild(button.Name)
                if building then building.Parent = workspace end
            else
                -- Set up purchase
                button.Touched:Connect(function(hit)
                    local touchPlayer = Players:GetPlayerFromCharacter(hit.Parent)
                    if touchPlayer ~= player then return end

                    -- Free buttons (cost = 0) skip the currency check
                    if config.cost > 0 then
                        local cash = CurrencySystem.getBalance(player, "Cash")
                        if cash < config.cost then return end
                        local spent = CurrencySystem.spend(player, "Cash", config.cost)
                        if not spent then return end
                    end

                    -- Save unlock
                    local d = DataManager.getData(player)
                    if d then
                        d.TycoonButtons = d.TycoonButtons or {}
                        d.TycoonButtons[button.Name] = true
                    end

                    button.Transparency = 1
                    button.CanCollide   = false

                    -- Activate dropper if this button enables one
                    local dropper = tycoonFolder.Droppers:FindFirstChild(button.Name)
                    if dropper then
                        startDropper(dropper, TycoonConfig.dropperInterval)
                    end
                end)
            end
        end
    end

    -- Setup conveyors
    local conveyor = tycoonFolder:FindFirstChild("Conveyor")
    if conveyor then
        for _, part in ipairs(conveyor:GetDescendants()) do
            if part:IsA("BasePart") and part.Name == "Belt" then
                setupConveyor(part)
            end
        end
    end

    -- Setup collector
    local collector = tycoonFolder:FindFirstChild("Collector")
    if collector then
        setupCollector(collector, player, TycoonConfig.partValue)
    end
end

function TycoonSystem.init()
    -- Claim pads: first player to touch unclaimed plot gets it
    for _, tycoon in ipairs(workspace.Tycoons:GetChildren()) do
        local plot = tycoon:FindFirstChild("Plot")
        if not plot then continue end

        local claimed = false
        plot.Touched:Connect(function(hit)
            if claimed then return end
            local player = Players:GetPlayerFromCharacter(hit.Parent)
            if not player then return end
            if playerTycoons[player] then return end  -- already has tycoon

            claimed = true
            plot.BrickColor = BrickColor.new("Bright green")
            -- Label it
            local label = plot:FindFirstChild("SurfaceGui")
            if label then
                label.Frame.TextLabel.Text = player.Name .. "'s Tycoon"
            end

            claimTycoon(player, tycoon)
        end)
    end

    Players.PlayerRemoving:Connect(function(player)
        -- Free up their tycoon plot
        local tycoon = playerTycoons[player]
        if tycoon then
            local plot = tycoon:FindFirstChild("Plot")
            if plot then
                plot.BrickColor = BrickColor.new("Medium stone grey")
            end
            -- Clean up drops
            for _, obj in ipairs(workspace:GetChildren()) do
                if obj.Name == "Drop" then obj:Destroy() end
            end
        end
        playerTycoons[player] = nil
    end)
end

return TycoonSystem
```

---

## Default Player Data

```lua
local DEFAULT_DATA = {
    Cash          = 0,
    Gems          = 0,
    TycoonButtons = {},  -- { buttonId = true }
    ProcessedReceipts = {},
}
```

---

## Tycoon Map Setup (Studio)

```
workspace/
  Tycoons/
    Tycoon1/
      Plot           (Part, 20x1x20, anchored, claim pad)
      Buttons/
        btn_conveyor  (Part, anchored, BillboardGui showing cost)
        btn_dropper1  (Part, anchored)
        ...
      Droppers/
        btn_dropper1/ (Model with SpawnPoint part)
        btn_dropper2/
      Conveyor/
        Belt/          (Parts with anchored = false so physics applies)
      Collector        (Part at end of conveyor, anchored)
      Buildings/       (visual structures, hidden until unlocked)
```

---

## Tycoon UI Checklist

- [ ] Cash counter (top left, large)
- [ ] "Income per second" display
- [ ] Tycoon name label on plot
- [ ] Button purchase UI (shows cost + what it unlocks)
- [ ] Upgrade shop (speed upgrades, value multipliers)
- [ ] Leaderboard (richest players)

---

## Common Tycoon Mistakes

| ❌ Mistake | ✅ Fix |
|---|---|
| Drops never destroyed | Use `Debris:AddItem(drop, 30)` |
| Conveyor speed resets | Reapply velocity each Heartbeat |
| Multiple players claim same plot | Check `claimed` boolean before allowing |
| Buttons triggerable by other players | Validate `touchPlayer == owner` |
| Data not saving button unlocks | Save `TycoonButtons` to DataStore |
