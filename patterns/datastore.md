# DataStore Pattern

Safe, production-ready DataStore with pcall, retry logic, and BindToClose support.

## Server: DataManager.lua

```lua
-- ServerScriptService/Server/Systems/DataManager.lua
local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")

local DataManager = {}

local store = DataStoreService:GetDataStore("PlayerData_v1")

-- Change version string (e.g. v2) to wipe all data on next load (migration)
local DEFAULT_DATA = {
    Coins      = 0,
    Gems       = 0,
    Reputation = 0,
    Upgrades   = {
        Speed     = 0,
        Value     = 0,
        Luck      = 0,
        ExtraSlot = 0,
    },
    DreamIndex = {},
}

local loadedData: {[Player]: any} = {}
local loadedCallbacks: {[Player]: {() -> ()}} = {}

local function deepCopy(t: any): any
    if typeof(t) ~= "table" then return t end
    local copy = {}
    for k, v in pairs(t) do copy[k] = deepCopy(v) end
    return copy
end

local function reconcile(data: any, template: any): any
    for key, defaultValue in pairs(template) do
        if data[key] == nil then
            data[key] = deepCopy(defaultValue)
        elseif typeof(data[key]) == "table" and typeof(defaultValue) == "table" then
            reconcile(data[key], defaultValue)
        end
    end
    return data
end

local function loadData(player: Player)
    local key = tostring(player.UserId)
    local success, result

    -- Retry up to 3 times with exponential backoff
    for attempt = 1, 3 do
        success, result = pcall(function()
            return store:GetAsync(key)
        end)
        if success then break end
        task.wait(2 ^ attempt)
    end

    local data
    if success and result then
        data = reconcile(result, deepCopy(DEFAULT_DATA))
    else
        data = deepCopy(DEFAULT_DATA)
        if not success then
            warn("[DataManager] Failed to load data for", player.Name, result)
        end
    end

    loadedData[player] = data

    -- Fire pending onLoaded callbacks
    if loadedCallbacks[player] then
        for _, cb in ipairs(loadedCallbacks[player]) do
            task.spawn(cb, data)
        end
        loadedCallbacks[player] = nil
    end
end

local function saveData(player: Player)
    local data = loadedData[player]
    if not data then return end

    local key = tostring(player.UserId)

    for attempt = 1, 3 do
        local success, err = pcall(function()
            store:SetAsync(key, data)
        end)
        if success then return end
        warn("[DataManager] Save attempt", attempt, "failed for", player.Name, err)
        task.wait(2 ^ attempt)
    end
end

function DataManager.init()
    Players.PlayerAdded:Connect(function(player)
        loadedCallbacks[player] = {}
        task.spawn(loadData, player)
    end)

    Players.PlayerRemoving:Connect(function(player)
        saveData(player)
        loadedData[player] = nil
        loadedCallbacks[player] = nil
    end)

    -- Critical: save all on server shutdown
    game:BindToClose(function()
        for player in pairs(loadedData) do
            task.spawn(saveData, player)
        end
        task.wait(2)  -- give saves time to complete
    end)
end

-- Get live data table (modify directly — saved on leave)
function DataManager.getData(player: Player): any
    return loadedData[player]
end

-- Register callback for when player data finishes loading
function DataManager.onLoaded(player: Player, callback: (any) -> ())
    if loadedData[player] then
        task.spawn(callback, loadedData[player])
    else
        if not loadedCallbacks[player] then
            loadedCallbacks[player] = {}
        end
        table.insert(loadedCallbacks[player], callback)
    end
end

return DataManager
```

## Key Rules

- **Never save more than once per 6 seconds** — Roblox rate limit
- **Always pcall** — DataStore calls fail in Studio and on timeouts
- **reconcile()** fills in missing keys when DEFAULT_DATA gains new fields
- **Version the store name** (`PlayerData_v1`, `v2`, etc.) to wipe cleanly during development
- **BindToClose** is mandatory — PlayerRemoving doesn't fire on server shutdown

## When to Save

| Event | Save? |
|---|---|
| Player leaves | ✅ Always |
| Server shutdown | ✅ Always (BindToClose) |
| Purchase / important action | ✅ Optional immediate save |
| Every loop tick | ❌ Never |
| Every currency change | ❌ No — data is in memory, saved on leave |
