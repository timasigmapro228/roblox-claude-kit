# NPC System

NPCs with personality, dialogue states, and proximity interaction.
Makes NPCs feel like characters, not vending machines.

---

## NPC Personality Template

Each NPC has a name, visual description, and dialogue for every state.
Write dialogue that's funny, weird, or charming — generic NPCs kill immersion.

```lua
-- ReplicatedStorage/Shared/Modules/NPCConfig.lua
local NPCConfig = {}

NPCConfig.Characters = {
    Grumpy = {
        name        = "Grumpy",
        description = "A bear who hasn't slept in three days",
        assetId     = 0,  -- Toolbox asset ID for the NPC model
        spawnPosition = Vector3.new(0, 0, 0),  -- set in Studio

        dialogue = {
            idle = {
                "...",
                "Are you just going to stand there?",
                "I need help. Obviously.",
                "My order won't fix itself.",
            },
            orderGiven = {
                "Don't drop it.",
                "I've had this since last Tuesday. Please hurry.",
                "Be careful. It's fragile. Everything is fragile.",
            },
            waiting = {
                "...",
                "Still waiting.",
                "I'm very patient. No I'm not.",
                "Is it done yet? It's been 4 seconds.",
            },
            resultCommon = {
                "Fine. It's fine.",
                "I've seen better. Thank you.",
                "Acceptable.",
            },
            resultRare = {
                "Oh. That's actually... nice.",
                "I wasn't expecting that. Good job I suppose.",
                "Hm. You're better than I thought.",
            },
            resultEpic = {
                "What. How did you do that.",
                "I take back everything I almost said about you.",
                "This is the best thing that's happened to me this week.",
            },
        },
    },

    Bubbly = {
        name        = "Bubbly",
        description = "Extremely enthusiastic about everything",
        assetId     = 0,
        spawnPosition = Vector3.new(10, 0, 0),

        dialogue = {
            idle = {
                "HI!! Are you the one who fixes things??",
                "Oh oh oh I have something for you!!",
                "You look like you can help me!!!",
            },
            orderGiven = {
                "YAY!! Thank you so much already!!",
                "I believe in you!!! You're amazing!!!",
                "This is so exciting I can barely stand it!!",
            },
            waiting = {
                "I'm so excited I might burst!!",
                "Any minute now... any minute... YESSS",
                "La la la la la~",
            },
            resultCommon = {
                "Yay!! It's all better!!",
                "You did it!! I knew you could!!",
            },
            resultRare = {
                "WAIT IT'S SPARKLY?! I LOVE IT SO MUCH!!",
                "OH WOW OH WOW OH WOW LOOK AT IT!!",
            },
            resultEpic = {
                "I'M GOING TO CRY. THIS IS PERFECT.",
                "THE MOST BEAUTIFUL THING I'VE EVER SEEN!!!",
            },
        },
    },
}

return NPCConfig
```

---

## Server: NPCSystem.lua

```lua
-- ServerScriptService/Server/Systems/NPCSystem.lua
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local NPCConfig  = require(ReplicatedStorage.Shared.Modules.NPCConfig)
local OrderSystem = require(script.Parent.OrderSystem)

local RemoteEvents = ReplicatedStorage.Shared.RemoteEvents
local RE_NPCDialogue  = RemoteEvents.NPCDialogue   -- server → client: show bubble
local NPCSystem = {}

-- Active NPC instances: { npcKey = { model, currentOrder, cooldown } }
local activeNPCs: {[string]: any} = {}

local function getRandomDialogue(npcKey: string, state: string): string
    local char = NPCConfig.Characters[npcKey]
    if not char then return "..." end
    local lines = char.dialogue[state]
    if not lines or #lines == 0 then return "..." end
    return lines[math.random(1, #lines)]
end

local function showDialogue(npcKey: string, state: string, nearbyPlayer: Player?)
    local line = getRandomDialogue(npcKey, state)
    local npcData = activeNPCs[npcKey]
    if not npcData then return end

    if nearbyPlayer then
        RE_NPCDialogue:FireClient(nearbyPlayer, npcKey, line)
    else
        -- Broadcast to all nearby players
        for _, player in ipairs(Players:GetPlayers()) do
            local char = player.Character
            if char then
                local hrp = char:FindFirstChild("HumanoidRootPart")
                local npcHRP = npcData.model:FindFirstChild("HumanoidRootPart")
                if hrp and npcHRP and (hrp.Position - npcHRP.Position).Magnitude < 30 then
                    RE_NPCDialogue:FireClient(player, npcKey, line)
                end
            end
        end
    end
end

local function spawnNPC(npcKey: string)
    local config = NPCConfig.Characters[npcKey]
    if not config then return end

    -- Load model from Toolbox asset
    local success, model = pcall(function()
        return game:GetService("InsertService"):LoadAsset(config.assetId)
    end)

    if not success or not model then
        warn("[NPCSystem] Failed to load NPC asset for", npcKey)
        return
    end

    local npcModel = model:FindFirstChildOfClass("Model") or model
    npcModel.Name = npcKey
    npcModel.Parent = workspace

    -- Position
    local hrp = npcModel:FindFirstChild("HumanoidRootPart")
    if hrp then
        hrp.CFrame = CFrame.new(config.spawnPosition)
    end

    -- Add ProximityPrompt
    local prompt = Instance.new("ProximityPrompt")
    prompt.ActionText = "Talk"
    prompt.ObjectText = config.name
    prompt.MaxActivationDistance = 8
    prompt.Parent = hrp or npcModel.PrimaryPart or npcModel

    -- Name label (BillboardGui above head)
    local billboard = Instance.new("BillboardGui")
    billboard.Size = UDim2.new(0, 200, 0, 40)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.AlwaysOnTop = false
    billboard.Parent = hrp or npcModel

    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, 0, 1, 0)
    nameLabel.Text = config.name
    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    nameLabel.TextStrokeTransparency = 0
    nameLabel.BackgroundTransparency = 1
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextSize = 16
    nameLabel.Parent = billboard

    activeNPCs[npcKey] = {
        model        = npcModel,
        prompt       = prompt,
        currentOrder = nil,
        onCooldown   = false,
    }

    -- Handle interaction
    prompt.Triggered:Connect(function(player: Player)
        local npcData = activeNPCs[npcKey]
        if not npcData or npcData.onCooldown then return end

        -- Cooldown to prevent spam
        npcData.onCooldown = true
        task.delay(1, function()
            if activeNPCs[npcKey] then
                activeNPCs[npcKey].onCooldown = false
            end
        end)

        -- Give a random order to the player
        local orderKeys = {}
        for key in pairs(require(ReplicatedStorage.Shared.Modules.OrderConfig).Items) do
            table.insert(orderKeys, key)
        end
        local randomOrder = orderKeys[math.random(1, #orderKeys)]

        -- Give a random order server-side (server-authoritative)
        local orderId = OrderSystem.giveOrder(player, randomOrder)
        if orderId then
            showDialogue(npcKey, "orderGiven", player)
        else
            showDialogue(npcKey, "waiting", player)  -- slots full or invalid order
        end
    end)

    -- Idle dialogue loop
    task.spawn(function()
        while activeNPCs[npcKey] do
            task.wait(math.random(8, 15))
            if activeNPCs[npcKey] then
                showDialogue(npcKey, "idle")
            end
        end
    end)
end

function NPCSystem.init()
    -- Spawn all configured NPCs
    for npcKey in pairs(NPCConfig.Characters) do
        task.spawn(spawnNPC, npcKey)
    end
end

-- Call when player returns a completed order to an NPC
function NPCSystem.onOrderComplete(player: Player, npcKey: string, rarity: string)
    local state = "resultCommon"
    if rarity == "Rare" then state = "resultRare"
    elseif rarity == "Epic" then state = "resultEpic" end
    showDialogue(npcKey, state, player)
end

return NPCSystem
```

---

## Client: NPCDialogueController.lua

```lua
-- StarterPlayerScripts/Client/Systems/NPCDialogueController.lua
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local RE_NPCDialogue = ReplicatedStorage.Shared.RemoteEvents.NPCDialogue

local player = Players.LocalPlayer
local NPCDialogueController = {}

-- Active dialogue bubbles: { npcKey = BillboardGui }
local activeBubbles: {[string]: BillboardGui} = {}

local function showBubble(npcKey: string, text: string)
    -- Find NPC model
    local npcModel = workspace:FindFirstChild(npcKey)
    if not npcModel then return end
    local hrp = npcModel:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    -- Remove existing bubble
    if activeBubbles[npcKey] then
        activeBubbles[npcKey]:Destroy()
    end

    -- Create bubble
    local billboard = Instance.new("BillboardGui")
    billboard.Size = UDim2.new(0, 240, 0, 60)
    billboard.StudsOffset = Vector3.new(0, 5.5, 0)
    billboard.AlwaysOnTop = false
    billboard.Parent = hrp

    local bubble = Instance.new("Frame")
    bubble.Size = UDim2.new(1, 0, 1, 0)
    bubble.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    bubble.BorderSizePixel = 0
    bubble.BackgroundTransparency = 0.1
    bubble.Parent = billboard

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = bubble

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -16, 1, 0)
    label.Position = UDim2.new(0, 8, 0, 0)
    label.Text = text
    label.TextColor3 = Color3.fromRGB(30, 30, 46)
    label.TextSize = 13
    label.Font = Enum.Font.Gotham
    label.BackgroundTransparency = 1
    label.TextWrapped = true
    label.Parent = bubble

    activeBubbles[npcKey] = billboard

    -- Fade in
    bubble.BackgroundTransparency = 1
    label.TextTransparency = 1
    TweenService:Create(bubble, TweenInfo.new(0.2), { BackgroundTransparency = 0.1 }):Play()
    TweenService:Create(label, TweenInfo.new(0.2), { TextTransparency = 0 }):Play()

    -- Auto-dismiss after 4 seconds
    task.delay(4, function()
        if activeBubbles[npcKey] == billboard then
            local bubbleFade = TweenService:Create(bubble, TweenInfo.new(0.3), {
                BackgroundTransparency = 1,
            })
            local labelFade = TweenService:Create(label, TweenInfo.new(0.3), {
                TextTransparency = 1,
            })
            labelFade.Completed:Connect(function()
                billboard:Destroy()
                if activeBubbles[npcKey] == billboard then
                    activeBubbles[npcKey] = nil
                end
            end)
            bubbleFade:Play()
            labelFade:Play()
        end
    end)
end

function NPCDialogueController.init()
    RE_NPCDialogue.OnClientEvent:Connect(function(npcKey: string, text: string)
        showBubble(npcKey, text)
    end)
end

return NPCDialogueController
```

---

## RemoteEvents Needed

```
ReplicatedStorage/Shared/RemoteEvents/
  NPCDialogue    ← server → client: show dialogue bubble (npcKey, text)
  -- Note: NPCInteract is NOT needed — ProximityPrompt.Triggered fires on server directly
  -- Do not add client→server NPCInteract; the ProximityPrompt handles interaction server-side
```
