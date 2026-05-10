---
name: genre-obby
description: Use when building a Roblox obby (obstacle course) — stages, checkpoints, kill bricks, moving platforms, finish line. Triggers on words like obby, obstacle course, checkpoint, kill brick, stage, platform, jump, parkour.
---

# Genre: Obby

**Core loop**: Player jumps through stages → touches checkpoint → dies on kill brick → respawns at checkpoint → reaches finish

**Examples**: Tower of Hell, Mega Easy Obby, Speed Run 4, The Maze

---

## Folder Structure

```
Workspace/
  Stages/
    Stage1/   ← Model containing all parts for stage 1
    Stage2/
    ...
  Checkpoints/
    Checkpoint1  ← SpawnLocation or custom Part
    Checkpoint2
    ...
  KillBricks/   ← Tagged with CollectionService "KillBrick"
  MovingParts/  ← Tagged with CollectionService "MovingPart"

ServerScriptService/Server/
  Systems/
    DataManager.lua
    CheckpointSystem.lua
    ObbySystem.lua

StarterPlayerScripts/Client/
  Systems/
    ObbyController.lua
```

---

## CheckpointSystem.lua (Server)

```lua
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local CollectionService = game:GetService("CollectionService")

local DataManager = require(script.Parent.DataManager)

local RE_CheckpointReached = ReplicatedStorage.Shared.RemoteEvents.CheckpointReached
local RE_StageComplete     = ReplicatedStorage.Shared.RemoteEvents.StageComplete

local CheckpointSystem = {}

-- Player checkpoint cache: { player = stageNumber }
local playerCheckpoints: {[Player]: number} = {}

local function spawnAtCheckpoint(player: Player, stage: number)
    local checkpoint = workspace.Checkpoints:FindFirstChild("Checkpoint" .. stage)
    if not checkpoint then return end

    local character = player.Character
    if not character then return end

    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    -- Offset up so player doesn't spawn inside the part
    hrp.CFrame = checkpoint.CFrame + Vector3.new(0, 3, 0)
end

local function onCharacterAdded(player: Player, character: Model)
    -- Wait a frame for character to load
    task.wait(0.1)

    local stage = playerCheckpoints[player] or 1
    spawnAtCheckpoint(player, stage)
end

function CheckpointSystem.init()
    -- Kill bricks via CollectionService (efficient — one script handles all)
    local function setupKillBrick(part: BasePart)
        part.Touched:Connect(function(hit)
            local character = hit.Parent
            local player = Players:GetPlayerFromCharacter(character)
            if not player then return end

            local humanoid = character:FindFirstChild("Humanoid")
            if humanoid and humanoid.Health > 0 then
                humanoid.Health = 0  -- triggers character respawn
            end
        end)
    end

    -- Setup existing kill bricks
    for _, part in ipairs(CollectionService:GetTagged("KillBrick")) do
        setupKillBrick(part)
    end
    -- Setup future kill bricks
    CollectionService:GetInstanceAddedSignal("KillBrick"):Connect(setupKillBrick)

    -- Checkpoints
    for _, checkpoint in ipairs(workspace.Checkpoints:GetChildren()) do
        local stageNum = tonumber(checkpoint.Name:match("%d+"))
        if not stageNum then continue end

        checkpoint.Touched:Connect(function(hit)
            local player = Players:GetPlayerFromCharacter(hit.Parent)
            if not player then return end

            local current = playerCheckpoints[player] or 1
            if stageNum <= current then return end  -- don't go backwards

            -- Save checkpoint
            playerCheckpoints[player] = stageNum

            local data = DataManager.getData(player)
            if data then
                data.ObbyCheckpoint = stageNum
                if stageNum > (data.ObbyBestStage or 0) then
                    data.ObbyBestStage = stageNum
                end
            end

            RE_CheckpointReached:FireClient(player, stageNum)

            -- Visual feedback on checkpoint
            local originalColor = checkpoint.BrickColor
            checkpoint.BrickColor = BrickColor.new("Bright green")
            task.delay(1, function()
                if checkpoint.Parent then
                    checkpoint.BrickColor = originalColor
                end
            end)
        end)
    end

    -- Finish line
    local finish = workspace:FindFirstChild("Finish")
    if finish then
        finish.Touched:Connect(function(hit)
            local player = Players:GetPlayerFromCharacter(hit.Parent)
            if not player then return end

            local data = DataManager.getData(player)
            if data then
                data.ObbyCompleted = (data.ObbyCompleted or 0) + 1
            end

            RE_StageComplete:FireClient(player, data and data.ObbyCompleted or 1)
        end)
    end

    Players.PlayerAdded:Connect(function(player)
        -- Load saved checkpoint
        DataManager.onLoaded(player, function(data)
            playerCheckpoints[player] = data.ObbyCheckpoint or 1
        end)

        player.CharacterAdded:Connect(function(character)
            onCharacterAdded(player, character)
        end)
    end)

    Players.PlayerRemoving:Connect(function(player)
        playerCheckpoints[player] = nil
    end)
end

return CheckpointSystem
```

---

## Moving Platforms (Server)

```lua
-- ObbySystem.lua
local CollectionService = game:GetService("CollectionService")
local TweenService = game:GetService("TweenService")

local function setupMovingPart(part: BasePart)
    -- Moving parts need attributes set in Studio:
    -- MoveAxis: "X" | "Y" | "Z"
    -- MoveDistance: number (studs)
    -- MoveSpeed: number (seconds for one way)

    local axis    = part:GetAttribute("MoveAxis") or "X"
    local distance = part:GetAttribute("MoveDistance") or 10
    local speed   = part:GetAttribute("MoveSpeed") or 2

    local origin = part.CFrame
    local offset = Vector3.new(
        axis == "X" and distance or 0,
        axis == "Y" and distance or 0,
        axis == "Z" and distance or 0
    )

    local tweenInfo = TweenInfo.new(speed, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut,
        -1,     -- repeat infinite
        true    -- reverses
    )

    local tween = TweenService:Create(part, tweenInfo, {
        CFrame = origin + offset
    })
    tween:Play()
end

-- Setup all moving parts
for _, part in ipairs(CollectionService:GetTagged("MovingPart")) do
    task.spawn(setupMovingPart, part)
end
CollectionService:GetInstanceAddedSignal("MovingPart"):Connect(function(part)
    task.spawn(setupMovingPart, part)
end)
```

---

## Stage Design Patterns

```
Stage 1: Tutorial — simple gaps, no tricks. Players learn to jump.
Stage 2: Rising challenge — slightly larger gaps or lower platforms.
Stage 3: First gimmick — moving platform OR kill bricks mixed in.
Stage 5: First "hard" section — combination of moving + kill bricks.
Stage 10: Midpoint reward — new visual theme, difficulty reset.
Stage 15+: Advanced — speed required, tight timing, multiple gimmicks.
```

**Rule**: Each stage should be completable in under 2 minutes by a new player who tries.
Frustration = quit. Challenge = retry. Know the difference.

---

## Part Tags (Studio — use CollectionService)

Tag parts in Studio via Tag Editor plugin:
- `KillBrick` — any part that kills on touch
- `MovingPart` — any part that moves (set attributes: MoveAxis, MoveDistance, MoveSpeed)
- `Checkpoint` — already handled by name, but tag works too
- `BouncePad` — launches player upward on touch
- `SpeedPad` — gives temporary speed boost

---

## Default Player Data

```lua
local DEFAULT_DATA = {
    ObbyCheckpoint = 1,       -- last reached checkpoint
    ObbyBestStage  = 1,       -- furthest ever reached
    ObbyCompleted  = 0,       -- number of full completions
    TotalDeaths    = 0,       -- optional stat
    ProcessedReceipts = {},
}
```

---

## Obby UI Checklist

- [ ] Stage counter "Stage X / Y" (top center)
- [ ] Death counter (optional, top right)
- [ ] Checkpoint notification (toast when reached)
- [ ] Completion screen (when finish touched)
- [ ] Leaderboard (fastest completion time, or stage reached)
- [ ] Skip Stage button (gamepass monetization)

---

## Monetization for Obby

| Gamepass | Effect |
|---|---|
| Skip Stage | Skip current stage once |
| Infinite Skips | Skip any stage anytime |
| Speed Coil | Permanent run speed boost |
| Gravity Coil | Permanent jump height boost |
| VIP | Exclusive skin + name tag |

**Rule**: Never make skips required. Players should be able to complete the entire obby free.
