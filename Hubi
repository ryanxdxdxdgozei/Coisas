local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "Simple Hub",
    Icon = 0,
    LoadingTitle = "Loading",
    LoadingSubtitle = "by UmFemboyqualquer",
    ShowText = "Rayfield",
    Theme = "Default",
    ToggleUIKeybind = "K",
    DisableRayfieldPrompts = false,
    DisableBuildWarnings = false,
    ConfigurationSaving = {
        Enabled = true,
        FolderName = nil,
        FileName = "SimpleHub"
    },
    Discord = { Enabled = false, Invite = "noinvitelink", RememberJoins = true },
    KeySystem = false,
})

local BattleTab = Window:CreateTab("BattleAssist", 4483362458)

BattleTab:CreateButton({
    Name = "NoStun (beta)",
    Callback = function()
        if getgenv().NoStunActive then return end
        getgenv().NoStunActive = true
        local player = game:GetService("Players").LocalPlayer
        local function setup(char)
            local hum = char:WaitForChild("Humanoid")
            local boostEnd = 0
            local lastHP = hum.Health
            hum:GetPropertyChangedSignal("Health"):Connect(function()
                if hum.Health < lastHP then boostEnd = tick() + 2 end
                lastHP = hum.Health
            end)
            task.spawn(function()
                while hum.Parent do
                    if tick() < boostEnd then hum.WalkSpeed = 50 end
                    task.wait(0.1)
                end
            end)
        end
        if player.Character then setup(player.Character) end
        player.CharacterAdded:Connect(setup)
    end,
})

BattleTab:CreateButton({
    Name = "DashCdRemover",
    Callback = function()
        if getgenv().DashCdActive then return end
        getgenv().DashCdActive = true
        local objs = {}
        local function scan()
            for _, v in pairs(getgc(true)) do
                if typeof(v) == "table" and rawget(v,"_cooltime") and rawget(v,"_dashStreakCount") then
                    local found = false
                    for _, e in ipairs(objs) do if e == v then found = true break end end
                    if not found then table.insert(objs, v) end
                end
            end
        end
        scan()
        task.spawn(function()
            local t = 0
            while getgenv().DashCdActive do
                t += 1
                if t >= 200 then t = 0 scan() end
                for _, d in ipairs(objs) do
                    pcall(function()
                        if rawget(d,"_dashStreakCount") == 0 then
                            if d._canDash == false then d._canDash = true end
                            if d._lastDashTime then d._lastDashTime = 0 end
                            if d._cooltime and d._cooltime.reset then d._cooltime:reset(0) end
                        end
                    end)
                end
                task.wait(0.05)
            end
        end)
    end,
})


local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer
local RequestSkillRemote = ReplicatedStorage
    :WaitForChild("@rbxts/wcs:source/networking@GlobalEvents")
    :WaitForChild("requestSkill")

local BLOCK_RANGE = 18
local BLOCK_COOLDOWN = 0.25
local AUTOBLOCK_COOLDOWN = 1.5
local BLOCK_DURATION = 0.35
local autoblockEnabled = false
local lastBlockTime = 0
local lastAutoBlockTime = 0
local isBlocking = false
local hookedModels = {}

local function doBlock()
    if not autoblockEnabled then return end
    local now = os.clock()
    if now - lastAutoBlockTime < AUTOBLOCK_COOLDOWN then return end
    if now - lastBlockTime < BLOCK_COOLDOWN or isBlocking then return end
    lastBlockTime = now
    lastAutoBlockTime = now
    isBlocking = true
    RequestSkillRemote:FireServer(unpack({{
        buffer = buffer.fromstring("\r\000\000\000General/Block\001\000\000\000\000"), blobs = {}
    }}))
    task.delay(BLOCK_DURATION, function()
        RequestSkillRemote:FireServer(unpack({{
            buffer = buffer.fromstring("\r\000\000\000General/Block\000\000\000\000\000"), blobs = {}
        }}))
        isBlocking = false
    end)
end

local IGNORED_ANIMS = {
    idle=true, walk=true, run=true, sprint=true, jump=true, fall=true,
    land=true, heavy_land=true, run_land=true, run_land_backward=true,
    walk_backward=true, swim=true, swimidle=true, climb=true, sit=true,
    dance=true, dance2=true, dance3=true, cheer=true, laugh=true,
    point=true, wave=true, toollunge=true, toolnone=true, toolslash=true,
    hit_animation1=true, hit_animation2=true, hit_animation3=true,
    block_hit_animation1=true, block_hit_animation2=true, block_hit_animation3=true,
}

local function isEnemy(model)
    if not model:IsA("Model") then return false end
    if model == LocalPlayer.Character then return false end
    if Players:GetPlayerFromCharacter(model) then return false end
    if not model:FindFirstChild("combatCooldown") then return false end
    return true
end

local function setupEnemy(model)
    if hookedModels[model] then return end

    if not model:FindFirstChild("combatCooldown") then
        local conn
        conn = model.DescendantAdded:Connect(function(child)
            if child.Name == "combatCooldown" then
                conn:Disconnect()
                task.wait(0.1)
                setupEnemy(model)
            end
        end)
        return
    end

    if not isEnemy(model) then return end

    local hum = model:FindFirstChildWhichIsA("Humanoid", true)
    if not hum then
        task.delay(0.3, function() setupEnemy(model) end)
        return
    end

    local root = model:FindFirstChild("HumanoidRootPart")
        or model:FindFirstChild("RootPart")
        or model.PrimaryPart
        or model:FindFirstChildWhichIsA("BasePart", true)
    if not root then
        task.delay(0.3, function() setupEnemy(model) end)
        return
    end

    hookedModels[model] = true

    local gotHit = false
    local lastHP = hum.Health

    hum.HealthChanged:Connect(function(hp)
        if hp < lastHP then
            gotHit = true
            task.delay(1, function() gotHit = false end)
        end
        lastHP = hp
    end)

    local function checkDist()
        local char = LocalPlayer.Character
        local myRoot = char and char:FindFirstChild("HumanoidRootPart")
        if not myRoot then return false end
        return (myRoot.Position - root.Position).Magnitude <= BLOCK_RANGE
    end

    local function shouldBlock(track)
        if not track or not track.IsPlaying then return false end
        if track.Looped then return false end
        local animObj = track.Animation
        local name = (animObj and animObj.Name or ""):lower()
        if name ~= "" and IGNORED_ANIMS[name] then return false end
        return true
    end

    task.spawn(function()
        local playingSet = {}
        while model.Parent and hum.Parent do
            if not gotHit and checkDist() and hum:GetState() ~= Enum.HumanoidStateType.Dead then
                for _, v in pairs(model:GetDescendants()) do
                    if v:IsA("Animator") then
                        local ok, tracks = pcall(function()
                            return v:GetPlayingAnimationTracks()
                        end)
                        if ok and tracks then
                            for _, track in pairs(tracks) do
                                local id = tostring(track)
                                if not playingSet[id] and shouldBlock(track) then
                                    playingSet[id] = true
                                    doBlock()
                                    task.delay(0.5, function()
                                        playingSet[id] = nil
                                    end)
                                end
                            end
                        end
                    end
                end
            end
            task.wait(0.05)
        end
        hookedModels[model] = nil
    end)
end

for _, v in pairs(workspace:GetDescendants()) do
    if v:IsA("Model") then task.spawn(setupEnemy, v) end
end

workspace.DescendantAdded:Connect(function(v)
    if v:IsA("Model") then
        task.wait(0.15)
        setupEnemy(v)
    end
end)

workspace.DescendantRemoving:Connect(function(v)
    hookedModels[v] = nil
end)

LocalPlayer.CharacterAdded:Connect(function()
    hookedModels = {}
    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("Model") then task.spawn(setupEnemy, v) end
    end
end)


BattleTab:CreateToggle({
    Name = "AutoPerfBlock",
    CurrentValue = false,
    Flag = "AutoPerfBlockToggle",
    Callback = function(v) autoblockEnabled = v end,
})

BattleTab:CreateSlider({
    Name = "AutoBlock CD (segundos)",
    Range = {0, 10}, Increment = 1, Suffix = "s",
    CurrentValue = 1, Flag = "AutoBlockCD",
    Callback = function(v) AUTOBLOCK_COOLDOWN = v end,
})

local FarmTab = Window:CreateTab("FarmHelper", 4483362458)

local teleports = {
    ["GOJO"]    = CFrame.new(2075.68042,164.128494,1157.99573,0.0221533291,0.956951737,-0.28940022,-4.65661343e-10,0.289471269,0.957186639,0.999754608,-0.0212048702,0.00641275244),
    ["SUKUNA"]  = CFrame.new(617.986206,181.247147,2959.45898,0.185924381,-0.628661394,0.755127192,7.4505806e-09,0.76852721,0.639817178,-0.982564092,-0.118957609,0.14288795),
    ["ITADORI"] = CFrame.new(-532.307556,238.75798,2415.08838,0.738092363,-0.523102105,0.426126778,-1.49011647e-08,0.631579876,0.775310814,-0.674699843,-0.572250962,0.466164201),
    ["ESO"]     = CFrame.new(1984.95032,123.46376,-2138.87695,-0.193435311,-4.02031048e-08,0.981113017,2.06824886e-08,1,4.50547759e-08,-0.981113017,2.90070439e-08,-0.193435311),
    ["URAUME"]  = CFrame.new(-133.396622,476.409729,-1265.70117,0.193170547,0.476926029,-0.857453644,0,0.873913646,0.486081272,0.98116523,-0.0938965827,0.168814361),
}

FarmTab:CreateDropdown({
    Name = "Boss Teleport",
    Options = {"GOJO","SUKUNA","ITADORI","ESO","URAUME"},
    CurrentOption = {"GOJO"}, MultipleOptions = false, Flag = "BossTeleport",
    Callback = function(opts)
        local cf = teleports[opts[1]]
        if cf then
            local r = (LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()):WaitForChild("HumanoidRootPart")
            r.CFrame = cf
        end
    end,
})

FarmTab:CreateButton({
    Name = "Vacuum Materials",
    Callback = function()
        for _, name in ipairs({"SlimeSample","LotusOoze","RottenBlood","EnchantedCrystals"}) do
            for _, v in pairs(workspace:GetDescendants()) do
                if v.Name == name then
                    local p = v:FindFirstChildWhichIsA("ProximityPrompt", true)
                    if p then fireproximityprompt(p) end
                end
            end
            task.wait(0.1)
        end
    end,
})

FarmTab:CreateButton({
    Name = "ChestFarm",
    Callback = function()
        local r = (LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()):WaitForChild("HumanoidRootPart")
        local targets = {["NormalChest"]=true,["LegendaryChest"]=true,["CursedChest_Purple"]=true,["CursedChest_Green"]=true}
        local visited = {}
        for _, v in pairs(workspace:GetDescendants()) do
            if targets[v.Name] and not visited[v] then
                local part = v:IsA("Model") and v.PrimaryPart or v
                if part and part:IsA("BasePart") then
                    visited[v] = true
                    r.CFrame = part.CFrame
                    task.wait(0.2)
                    r.CFrame = part.CFrame * CFrame.new(0,0.5,0)
                    task.wait(1)
                end
            end
        end
    end,
})

local MiscTab = Window:CreateTab("Misc", 4483362458)

MiscTab:CreateButton({
    Name = "FPS Booster",
    Callback = function()
        if getgenv().FPSBoostActive then return end
        getgenv().FPSBoostActive = true
        local Lighting = game:GetService("Lighting")
        pcall(function()
            Lighting.GlobalShadows = false
            Lighting.ShadowSoftness = 0
            Lighting.EnvironmentDiffuseScale = 0
            Lighting.EnvironmentSpecularScale = 0
        end)
        for _, v in ipairs(Lighting:GetChildren()) do pcall(function() v:Destroy() end) end
        Lighting.ChildAdded:Connect(function(v) task.wait() pcall(function() v:Destroy() end) end)
        local cam = workspace.CurrentCamera
        for _, v in ipairs(cam:GetChildren()) do pcall(function() v:Destroy() end) end
        cam.ChildAdded:Connect(function(v) task.wait() pcall(function() v:Destroy() end) end)
        local function isPickup(v)
            if v:IsA("ProximityPrompt") then return true end
            if v:FindFirstChildWhichIsA("ProximityPrompt", true) then return true end
            local p = v.Parent
            if p and p:FindFirstChildWhichIsA("ProximityPrompt", true) then return true end
            return false
        end

        local function process(v)
            if isPickup(v) then return end
            if v:IsA("BasePart") then pcall(function() v.CastShadow = false end) end
            if v:IsA("PointLight") or v:IsA("SpotLight") or v:IsA("SurfaceLight") then pcall(function() v:Destroy() end) end
            if v:IsA("ParticleEmitter") or v:IsA("Smoke") or v:IsA("Fire") or v:IsA("Sparkles") or v:IsA("Beam") or v:IsA("Trail") then pcall(function() v.Enabled = false end) end
            if v:IsA("BillboardGui") then pcall(function() v.Enabled = false end) end
        end
        local desc = workspace:GetDescendants()
        task.spawn(function()
            for i = 1, #desc, 75 do
                for j = i, math.min(i+74, #desc) do pcall(process, desc[j]) end
                task.wait()
            end
        end)
        workspace.DescendantAdded:Connect(function(v) task.wait() pcall(process, v) end)
    end,
})
