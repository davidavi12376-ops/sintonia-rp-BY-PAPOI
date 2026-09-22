--[[
    Script sintonia by dvzx
    ESP Esqueleto + Staff | Aimbot (Camera/Mouse/Silent) | Prediction | Speed | Anti-Lag
    F9 Limpo + Detecta Times
]]

-- ========== SILENCIA F9 ==========
local function silent() end
print = silent
warn = silent
pcall(function()
    getgenv().print = silent
    getgenv().warn = silent
end)
-- =================================

local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

local Window = Fluent:CreateWindow({
    Title = "Script sintonia by dvzx",
    SubTitle = "ESP | Aimbot Silent | Speed | Anti-Lag",
    TabWidth = 155,
    Size = UDim2.fromOffset(540, 600),
    Acrylic = false,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.RightControl
})

local Tabs = {
    Combat = Window:AddTab({ Title = "Combat", Icon = "crosshair" }),
    Visuals = Window:AddTab({ Title = "Visuals", Icon = "eye" }),
    Player = Window:AddTab({ Title = "Player", Icon = "user" }),
    Settings = Window:AddTab({ Title = "Settings", Icon = "settings" })
}

local Players = game:GetService("Players")
local Teams = game:GetService("Teams")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local VirtualInputManager = game:GetService("VirtualInputManager")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

local FoundTeams = {}
local IgnoreTeams = {}
local TeamToggles = {}
local SilentTarget = nil

local Config = {
    ESP = {
        Enabled = false,
        Skeleton = true,
        Boxes = false,
        Names = true,
        Distance = true,
        Health = true,
        Color = Color3.fromRGB(0, 255, 140),
        StaffColor = Color3.fromRGB(170, 0, 255),
        StaffEnabled = true
    },
    Aimbot = {
        Enabled = false,
        AimAssist = false,
        FOV = 130,
        Smoothness = 0.15,
        TeamCheck = true,
        WallCheck = true,
        TargetPart = "Head",
        Mode = "Camera", -- Camera / Mouse / Silent
        Prediction = true,
        PredictionAmount = 0.12
    },
    Player = {
        Speed = 22,
        SpeedEnabled = false
    },
    AntiLag = {
        Enabled = false
    }
}

-- ====================== STAFF DETECT ======================
local function IsStaff(player)
    if not player then return false end
    local staffKeywords = {
        "staff", "admin", "adm", "mod", "moderador", "moderadora",
        "diretor", "diretora", "dono", "dona", "supervisor",
        "coordenador", "gerente", "owner", "dev", "developer", "equipe"
    }
    if player.Team then
        local teamName = string.lower(player.Team.Name)
        for _, word in ipairs(staffKeywords) do
            if string.find(teamName, word) then return true end
        end
    end
    local sources = {player, player.Character}
    local names = {"Team", "Facção", "Faccao", "Cargo", "Job", "Role", "Staff", "Admin", "Rank", "Grupo", "Organization"}
    for _, source in ipairs(sources) do
        if source then
            for _, name in ipairs(names) do
                local val = source:FindFirstChild(name)
                if val and (val:IsA("StringValue") or val:IsA("IntValue") or val:IsA("NumberValue")) then
                    local v = string.lower(tostring(val.Value))
                    for _, word in ipairs(staffKeywords) do
                        if string.find(v, word) then return true end
                    end
                end
                local attr = source:GetAttribute(name)
                if attr then
                    local v = string.lower(tostring(attr))
                    for _, word in ipairs(staffKeywords) do
                        if string.find(v, word) then return true end
                    end
                end
            end
        end
    end
    return false
end

-- ====================== DETECTAR TIMES ======================
local function GetPlayerTeamName(player)
    if not player then return nil end
    if player.Team then return player.Team.Name end
    local sources = {player, player.Character}
    local names = {"Team", "Facção", "Faccao", "Organization", "Org", "Grupo", "Side", "Job", "Cargo", "Time"}
    for _, source in ipairs(sources) do
        if source then
            for _, name in ipairs(names) do
                local val = source:FindFirstChild(name)
                if val and (val:IsA("StringValue") or val:IsA("IntValue") or val:IsA("NumberValue")) then
                    local v = tostring(val.Value)
                    if v ~= "" and v ~= "nil" then return v end
                end
                local attr = source:GetAttribute(name)
                if attr ~= nil and tostring(attr) ~= "" then return tostring(attr) end
            end
        end
    end
    return nil
end

local function ScanTeams()
    FoundTeams = {}
    for _, team in pairs(Teams:GetTeams()) do
        if team.Name and team.Name ~= "" then FoundTeams[team.Name] = true end
    end
    for _, plr in ipairs(Players:GetPlayers()) do
        local name = GetPlayerTeamName(plr)
        if name and name ~= "" then FoundTeams[name] = true end
    end
    return FoundTeams
end

-- ====================== ANTI-LAG ======================
local OriginalSettings = {}

local function SaveOriginalSettings()
    OriginalSettings = {
        QualityLevel = settings().Rendering.QualityLevel,
        GlobalShadows = Lighting.GlobalShadows,
        Brightness = Lighting.Brightness,
        ClockTime = Lighting.ClockTime,
        FogEnd = Lighting.FogEnd,
        FogStart = Lighting.FogStart,
        Ambient = Lighting.Ambient,
        OutdoorAmbient = Lighting.OutdoorAmbient
    }
end

local function EnableAntiLag()
    settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
    Lighting.ClockTime = 14
    Lighting.Brightness = 3
    Lighting.GlobalShadows = false
    Lighting.FogEnd = 9e9
    Lighting.FogStart = 9e9
    Lighting.Ambient = Color3.fromRGB(160, 160, 160)
    Lighting.OutdoorAmbient = Color3.fromRGB(160, 160, 160)

    for _, v in pairs(Lighting:GetChildren()) do
        if v:IsA("Sky") or v:IsA("BloomEffect") or v:IsA("BlurEffect") or
           v:IsA("SunRaysEffect") or v:IsA("ColorCorrectionEffect") or
           v:IsA("DepthOfFieldEffect") or v:IsA("Atmosphere") or v:IsA("Clouds") then
            pcall(function() v:Destroy() end)
        end
    end

    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Texture") or obj:IsA("Decal") then pcall(function() obj:Destroy() end) end
        if obj:IsA("BasePart") then
            pcall(function()
                obj.Material = Enum.Material.SmoothPlastic
                obj.Reflectance = 0
            end)
        end
        if obj:IsA("MeshPart") then
            pcall(function()
                obj.Material = Enum.Material.SmoothPlastic
                obj.RenderFidelity = Enum.RenderFidelity.Performance
            end)
        end
        if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Smoke") or
           obj:IsA("Fire") or obj:IsA("Sparkles") or obj:IsA("Beam") then
            pcall(function() obj.Enabled = false end)
        end
    end
    pcall(function() workspace.Terrain.Decoration = false end)
end

local function DisableAntiLag()
    if not OriginalSettings.QualityLevel then return end
    pcall(function()
        settings().Rendering.QualityLevel = OriginalSettings.QualityLevel
        Lighting.GlobalShadows = OriginalSettings.GlobalShadows
        Lighting.Brightness = OriginalSettings.Brightness
        Lighting.ClockTime = OriginalSettings.ClockTime
        Lighting.FogEnd = OriginalSettings.FogEnd
        Lighting.FogStart = OriginalSettings.FogStart
        Lighting.Ambient = OriginalSettings.Ambient
        Lighting.OutdoorAmbient = OriginalSettings.OutdoorAmbient
    end)
end

SaveOriginalSettings()

-- ====================== ESP ======================
local ESPFolder = Instance.new("Folder")
ESPFolder.Name = "Cache_" .. math.random(100000,999999)
ESPFolder.Parent = game:GetService("CoreGui")

local ScreenESP = Instance.new("ScreenGui")
ScreenESP.Name = "UI_" .. math.random(100000,999999)
ScreenESP.ResetOnSpawn = false
ScreenESP.IgnoreGuiInset = true
ScreenESP.Parent = ESPFolder

local ESPObjects = {}

local BonesR15 = {
    {"Head", "UpperTorso"}, {"UpperTorso", "LowerTorso"},
    {"UpperTorso", "LeftUpperArm"}, {"LeftUpperArm", "LeftLowerArm"}, {"LeftLowerArm", "LeftHand"},
    {"UpperTorso", "RightUpperArm"}, {"RightUpperArm", "RightLowerArm"}, {"RightLowerArm", "RightHand"},
    {"LowerTorso", "LeftUpperLeg"}, {"LeftUpperLeg", "LeftLowerLeg"}, {"LeftLowerLeg", "LeftFoot"},
    {"LowerTorso", "RightUpperLeg"}, {"RightUpperLeg", "RightLowerLeg"}, {"RightLowerLeg", "RightFoot"}
}

local BonesR6 = {
    {"Head", "Torso"}, {"Torso", "Left Arm"}, {"Torso", "Right Arm"},
    {"Torso", "Left Leg"}, {"Torso", "Right Leg"}
}

local function CreateLine()
    local line = Drawing.new("Line")
    line.Thickness = 1.7
    line.Color = Config.ESP.Color
    line.Visible = false
    return line
end

local function CreateESP(player)
    if player == LocalPlayer or ESPObjects[player] then return end
    local data = { Lines = {} }
    for i = 1, 14 do data.Lines[i] = CreateLine() end

    local box = Instance.new("Frame")
    box.BackgroundTransparency = 0.84
    box.BorderSizePixel = 0
    box.BackgroundColor3 = Config.ESP.Color
    box.Visible = false
    box.Parent = ScreenESP
    Instance.new("UIStroke", box).Thickness = 1.3
    data.Box = box

    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(0, 200, 0, 18)
    nameLabel.BackgroundTransparency = 1
    nameLabel.TextColor3 = Color3.new(1,1,1)
    nameLabel.TextStrokeTransparency = 0.3
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextSize = 13
    nameLabel.Visible = false
    nameLabel.Parent = ScreenESP
    data.NameLabel = nameLabel

    local infoLabel = Instance.new("TextLabel")
    infoLabel.Size = UDim2.new(0, 200, 0, 16)
    infoLabel.BackgroundTransparency = 1
    infoLabel.TextColor3 = Color3.fromRGB(210,210,210)
    infoLabel.TextStrokeTransparency = 0.4
    infoLabel.Font = Enum.Font.Gotham
    infoLabel.TextSize = 11
    infoLabel.Visible = false
    infoLabel.Parent = ScreenESP
    data.InfoLabel = infoLabel

    ESPObjects[player] = data
end

local function RemoveESP(player)
    local data = ESPObjects[player]
    if not data then return end
    for _, line in pairs(data.Lines) do pcall(function() line:Remove() end) end
    pcall(function() data.Box:Destroy() end)
    pcall(function() data.NameLabel:Destroy() end)
    pcall(function() data.InfoLabel:Destroy() end)
    ESPObjects[player] = nil
end

local function GetBonePos(char, name)
    local p = char:FindFirstChild(name)
    return p and p.Position
end

local function UpdateESP()
    for player, data in pairs(ESPObjects) do
        if not player.Parent or not player.Character then
            for _, l in pairs(data.Lines) do l.Visible = false end
            data.Box.Visible = false
            data.NameLabel.Visible = false
            data.InfoLabel.Visible = false
            continue
        end

        local char = player.Character
        local hum = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso")

        if not hum or not root or hum.Health <= 0 or not Config.ESP.Enabled then
            for _, l in pairs(data.Lines) do l.Visible = false end
            data.Box.Visible = false
            data.NameLabel.Visible = false
            data.InfoLabel.Visible = false
            continue
        end

        local espColor = Config.ESP.Color
        if Config.ESP.StaffEnabled and IsStaff(player) then
            espColor = Config.ESP.StaffColor
        end

        if Config.ESP.Skeleton then
            local list = char:FindFirstChild("UpperTorso") and BonesR15 or BonesR6
            for i, pair in ipairs(list) do
                local line = data.Lines[i]
                if not line then continue end
                local p1, p2 = GetBonePos(char, pair[1]), GetBonePos(char, pair[2])
                if p1 and p2 then
                    local s1, on1 = Camera:WorldToViewportPoint(p1)
                    local s2, on2 = Camera:WorldToViewportPoint(p2)
                    if on1 and on2 and s1.Z > 0 and s2.Z > 0 then
                        line.From = Vector2.new(s1.X, s1.Y)
                        line.To = Vector2.new(s2.X, s2.Y)
                        line.Color = espColor
                        line.Visible = true
                    else
                        line.Visible = false
                    end
                else
                    line.Visible = false
                end
            end
        else
            for _, l in pairs(data.Lines) do l.Visible = false end
        end

        local pos, onScreen = Camera:WorldToViewportPoint(root.Position)
        if onScreen and pos.Z > 0 then
            local dist = (root.Position - Camera.CFrame.Position).Magnitude
            local scale = math.clamp(1000 / dist, 0.35, 4)
            local size = Vector2.new(36 * scale, 56 * scale)

            if Config.ESP.Boxes then
                data.Box.Size = UDim2.new(0, size.X, 0, size.Y)
                data.Box.Position = UDim2.new(0, pos.X - size.X/2, 0, pos.Y - size.Y/2)
                data.Box.BackgroundColor3 = espColor
                data.Box.Visible = true
            else
                data.Box.Visible = false
            end

            data.NameLabel.Text = player.Name
            data.NameLabel.TextColor3 = espColor
            data.NameLabel.Position = UDim2.new(0, pos.X - 100, 0, pos.Y - size.Y/2 - 20)
            data.NameLabel.Visible = Config.ESP.Names

            local hp = Config.ESP.Health and ("HP: " .. math.floor(hum.Health)) or ""
            local d = Config.ESP.Distance and (math.floor(dist) .. "m") or ""
            data.InfoLabel.Text = hp .. (hp ~= "" and d ~= "" and " | " or "") .. d
            data.InfoLabel.Position = UDim2.new(0, pos.X - 100, 0, pos.Y + size.Y/2 + 3)
            data.InfoLabel.Visible = Config.ESP.Health or Config.ESP.Distance
        else
            data.Box.Visible = false
            data.NameLabel.Visible = false
            data.InfoLabel.Visible = false
        end
    end
end

-- ====================== TEAM CHECK ======================
local function IsIgnoredTeam(player)
    if not Config.Aimbot.TeamCheck then return false end
    if player == LocalPlayer then return true end
    if player.Team and LocalPlayer.Team and player.Team == LocalPlayer.Team then return true end
    if player.TeamColor and LocalPlayer.TeamColor and player.TeamColor == LocalPlayer.TeamColor then return true end

    local teamName = GetPlayerTeamName(player)
    if not teamName then return false end

    for name, ignore in pairs(IgnoreTeams) do
        if ignore then
            if string.lower(name) == string.lower(teamName) or string.find(string.lower(teamName), string.lower(name)) then
                return true
            end
        end
    end
    return false
end

-- ====================== AIMBOT + SILENT + PREDICTION ======================
local function GetPredictedPosition(part)
    if not Config.Aimbot.Prediction or not part then
        return part and part.Position or Vector3.zero
    end
    local velocity = part.AssemblyLinearVelocity or Vector3.zero
    return part.Position + (velocity * Config.Aimbot.PredictionAmount)
end

local function HasWallBetween(fromPos, toPos, targetChar)
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {LocalPlayer.Character, targetChar}
    return workspace:Raycast(fromPos, toPos - fromPos, params) ~= nil
end

local function GetClosest()
    local closest, shortest = nil, Config.Aimbot.FOV
    local mousePos = UserInputService:GetMouseLocation()

    for _, plr in ipairs(Players:GetPlayers()) do
        if plr == LocalPlayer then continue end
        if IsIgnoredTeam(plr) then continue end

        local char = plr.Character
        if not char then continue end

        local hum = char:FindFirstChildOfClass("Humanoid")
        local part = char:FindFirstChild(Config.Aimbot.TargetPart)
            or char:FindFirstChild("Head")
            or char:FindFirstChild("HumanoidRootPart")

        if not hum or hum.Health <= 0 or not part then continue end

        local worldPos = GetPredictedPosition(part)
        if Config.Aimbot.WallCheck and HasWallBetween(Camera.CFrame.Position, worldPos, char) then
            continue
        end

        local sp, onScreen = Camera:WorldToViewportPoint(worldPos)
        if not onScreen or sp.Z < 0 then continue end

        local dist = (Vector2.new(sp.X, sp.Y) - mousePos).Magnitude
        if dist < shortest then
            shortest = dist
            closest = part
        end
    end
    return closest
end

-- Silent Aim Hook
local SilentHooked = false
local function HookSilentAim()
    if SilentHooked then return end
    local ok = pcall(function()
        local mt = getrawmetatable(game)
        local oldIndex = mt.__index
        setreadonly(mt, false)
        mt.__index = newcclosure(function(self, key)
            if Config.Aimbot.Enabled and Config.Aimbot.Mode == "Silent" and SilentTarget and SilentTarget.Parent then
                if self == Mouse and (key == "Hit" or key == "Target") then
                    local pos = GetPredictedPosition(SilentTarget)
                    if key == "Hit" then
                        return CFrame.new(pos)
                    elseif key == "Target" then
                        return SilentTarget
                    end
                end
            end
            return oldIndex(self, key)
        end)
        setreadonly(mt, true)
        SilentHooked = true
    end)
end
pcall(HookSilentAim)

local function AimAt(target)
    if not target then
        SilentTarget = nil
        return
    end

    SilentTarget = target
    local worldPos = GetPredictedPosition(target)
    local screenPos, onScreen = Camera:WorldToViewportPoint(worldPos)
    if not onScreen then return end

    if Config.Aimbot.Mode == "Silent" then
        return -- só o hook
    elseif Config.Aimbot.Mode == "Mouse" then
        local mousePos = UserInputService:GetMouseLocation()
        local deltaX = (screenPos.X - mousePos.X) * Config.Aimbot.Smoothness
        local deltaY = (screenPos.Y - mousePos.Y) * Config.Aimbot.Smoothness
        if mousemoverel then
            mousemoverel(deltaX, deltaY)
        else
            pcall(function()
                VirtualInputManager:SendMouseMoveEvent(mousePos.X + deltaX, mousePos.Y + deltaY, game)
            end)
        end
    else
        local cf = Camera.CFrame
        Camera.CFrame = cf:Lerp(CFrame.new(cf.Position, worldPos), Config.Aimbot.Smoothness)
    end
end

-- ====================== SPEED ======================
local SpeedConn = nil

local function StartSpeed()
    if SpeedConn then SpeedConn:Disconnect() end
    SpeedConn = RunService.Heartbeat:Connect(function()
        if not Config.Player.SpeedEnabled then return end
        local char = LocalPlayer.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root or not hum then return end
        if hum.WalkSpeed ~= 16 then hum.WalkSpeed = 16 end
        local dir = hum.MoveDirection
        if dir.Magnitude > 0.05 then
            root.AssemblyLinearVelocity = Vector3.new(dir.X * Config.Player.Speed, root.AssemblyLinearVelocity.Y, dir.Z * Config.Player.Speed)
        end
    end)
end

local function StopSpeed()
    if SpeedConn then SpeedConn:Disconnect() SpeedConn = nil end
    local char = LocalPlayer.Character
    if char and char:FindFirstChildOfClass("Humanoid") then
        char.Humanoid.WalkSpeed = 16
    end
end

-- ====================== LOOPS ======================
RunService.RenderStepped:Connect(function()
    UpdateESP()
    if Config.Aimbot.Enabled or Config.Aimbot.AimAssist then
        local t = GetClosest()
        if t then
            if Config.Aimbot.Enabled then
                AimAt(t)
            else
                local old = Config.Aimbot.Smoothness
                Config.Aimbot.Smoothness = 0.06
                AimAt(t)
                Config.Aimbot.Smoothness = old
            end
        else
            SilentTarget = nil
        end
    else
        SilentTarget = nil
    end
end)

Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function() task.wait(0.5) CreateESP(plr) end)
    if plr.Character then task.delay(0.5, CreateESP, plr) end
end)
Players.PlayerRemoving:Connect(RemoveESP)

for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= LocalPlayer then
        CreateESP(plr)
        plr.CharacterAdded:Connect(function() task.wait(0.5) CreateESP(plr) end)
    end
end

-- ====================== GUI ======================
Tabs.Combat:AddToggle("Aimbot", {Title = "Aimbot", Default = false, Callback = function(v)
    Config.Aimbot.Enabled = v
    if v then Config.Aimbot.AimAssist = false end
end})

Tabs.Combat:AddToggle("AimAssist", {Title = "Aim Assist", Default = false, Callback = function(v)
    Config.Aimbot.AimAssist = v
    if v then Config.Aimbot.Enabled = false end
end})

Tabs.Combat:AddDropdown("AimMode", {
    Title = "Modo do Aimbot",
    Values = {"Camera", "Mouse", "Silent"},
    Default = "Camera",
    Callback = function(Value)
        Config.Aimbot.Mode = Value
    end
})

Tabs.Combat:AddDropdown("AimPart", {
    Title = "Parte do Corpo",
    Values = {"Head", "HumanoidRootPart", "UpperTorso"},
    Default = "Head",
    Callback = function(Value)
        Config.Aimbot.TargetPart = Value
    end
})

Tabs.Combat:AddSlider("FOV", {Title = "FOV", Default = 130, Min = 40, Max = 300, Rounding = 0, Callback = function(v) Config.Aimbot.FOV = v end})
Tabs.Combat:AddSlider("Smooth", {Title = "Smoothness", Default = 0.15, Min = 0.04, Max = 0.4, Rounding = 2, Callback = function(v) Config.Aimbot.Smoothness = v end})

Tabs.Combat:AddToggle("Prediction", {
    Title = "Prediction (puxa na frente)",
    Default = true,
    Callback = function(v) Config.Aimbot.Prediction = v end
})

Tabs.Combat:AddSlider("PredAmount", {
    Title = "Força da Prediction",
    Default = 0.12,
    Min = 0.02,
    Max = 0.35,
    Rounding = 2,
    Callback = function(v) Config.Aimbot.PredictionAmount = v end
})

Tabs.Combat:AddToggle("TeamCheck", {Title = "Team Check", Default = true, Callback = function(v) Config.Aimbot.TeamCheck = v end})
Tabs.Combat:AddToggle("WallCheck", {Title = "Wall Check", Default = true, Callback = function(v) Config.Aimbot.WallCheck = v end})

Tabs.Combat:AddParagraph({
    Title = "Times Detectados",
    Content = "Clique em Atualizar Times e marque os que o Aimbot deve IGNORAR."
})

local function ClearTeamToggles()
    for _, toggle in pairs(TeamToggles) do
        pcall(function()
            if toggle and toggle.Container then toggle.Container:Destroy() end
        end)
    end
    TeamToggles = {}
end

local function CreateTeamToggles()
    ClearTeamToggles()
    ScanTeams()
    for teamName, _ in pairs(FoundTeams) do
        local toggle = Tabs.Combat:AddToggle("Team_" .. teamName:gsub("%s+", "_"), {
            Title = "Ignorar: " .. teamName,
            Default = IgnoreTeams[teamName] or false,
            Callback = function(v) IgnoreTeams[teamName] = v end
        })
        TeamToggles[teamName] = toggle
    end
end

Tabs.Combat:AddButton({
    Title = "🔄 Atualizar Times do Servidor",
    Callback = function() CreateTeamToggles() end
})

task.spawn(function()
    task.wait(1.2)
    CreateTeamToggles()
end)

-- Visuals
Tabs.Visuals:AddToggle("ESP", {Title = "ESP Ativado", Default = false, Callback = function(v) Config.ESP.Enabled = v end})
Tabs.Visuals:AddToggle("Skeleton", {Title = "Esqueleto (FiveM)", Default = true, Callback = function(v) Config.ESP.Skeleton = v end})
Tabs.Visuals:AddToggle("Boxes", {Title = "Caixas", Default = false, Callback = function(v) Config.ESP.Boxes = v end})
Tabs.Visuals:AddToggle("Names", {Title = "Nomes", Default = true, Callback = function(v) Config.ESP.Names = v end})
Tabs.Visuals:AddToggle("Distance", {Title = "Distância", Default = true, Callback = function(v) Config.ESP.Distance = v end})
Tabs.Visuals:AddToggle("Health", {Title = "Vida", Default = true, Callback = function(v) Config.ESP.Health = v end})

Tabs.Visuals:AddToggle("StaffESP", {
    Title = "ESP Staff/Admin (cor diferente)",
    Default = true,
    Callback = function(v) Config.ESP.StaffEnabled = v end
})

Tabs.Visuals:AddColorpicker("ESPColor", {
    Title = "Cor do ESP Normal",
    Default = Color3.fromRGB(0, 255, 140),
    Callback = function(v) Config.ESP.Color = v end
})

Tabs.Visuals:AddColorpicker("StaffColor", {
    Title = "Cor do Staff/Admin",
    Default = Color3.fromRGB(170, 0, 255),
    Callback = function(v) Config.ESP.StaffColor = v end
})

-- Player
Tabs.Player:AddToggle("SpeedToggle", {
    Title = "Speed (Bypass)",
    Default = false,
    Callback = function(v)
        Config.Player.SpeedEnabled = v
        if v then StartSpeed() else StopSpeed() end
    end
})
Tabs.Player:AddSlider("SpeedValue", {
    Title = "Velocidade",
    Default = 22,
    Min = 16,
    Max = 50,
    Rounding = 0,
    Callback = function(v) Config.Player.Speed = v end
})

Tabs.Player:AddToggle("AntiLag", {
    Title = "Anti-Lag (Texturas + Dia)",
    Default = false,
    Callback = function(v)
        Config.AntiLag.Enabled = v
        if v then EnableAntiLag() else DisableAntiLag() end
    end
})

Tabs.Settings:AddButton({
    Title = "Destruir Script",
    Callback = function()
        StopSpeed()
        if Config.AntiLag.Enabled then DisableAntiLag() end
        for plr,_ in pairs(ESPObjects) do RemoveESP(plr) end
        pcall(function() ESPFolder:Destroy() end)
        Window:Destroy()
    end
})

SaveManager:SetLibrary(Fluent)
InterfaceManager:SetLibrary(Fluent)
InterfaceManager:SetFolder("SintoniaDVZX")
SaveManager:SetFolder("SintoniaDVZX")
InterfaceManager:BuildInterfaceSection(Tabs.Settings)
SaveManager:BuildConfigSection(Tabs.Settings)

Window:SelectTab(1)
