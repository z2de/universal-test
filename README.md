local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local Settings = getgenv().Settings

_G.ESPEnabled = Settings.ESP.Enabled
_G.NameESPEnabled = Settings.NameESP.Enabled
_G.OutlineEnabled = Settings.PlayerEffects.Enabled  -- Updated from PlayersMT
_G.AimbotEnabled = Settings.Aimbot.Enabled
_G.AimbotSmoothness = Settings.Aimbot.Smoothness
_G.AimbotPart = Settings.Aimbot.TargetPart
_G.AimbotKey = Settings.Aimbot.Key
_G.AimbotMethod = Settings.Aimbot.Method
_G.TeamCheckEnabled = Settings.TeamCheck.Enabled
_G.TracerEnabled = Settings.Tracer.Enabled
_G.TracerOrigin = Settings.Tracer.Origin
_G.CFrameSpeedEnabled = Settings.CFrameSpeed.Enabled
_G.CFrameSpeedValue = Settings.CFrameSpeed.Speed
_G.OrbitEnabled = Settings.Orbit.Enabled
_G.OrbitHeight = Settings.Orbit.Height
_G.OrbitSpeed = Settings.Orbit.Speed
_G.OrbitDistance = Settings.Orbit.Distance
_G.OrbitFaceTarget = Settings.Orbit.FaceTarget
_G.FlyEnabled = Settings.Fly.Enabled
_G.FlySpeed = Settings.Fly.Speed
_G.AimbotBindType = Settings.Aimbot.BindType
_G.AimbotMouseBind = Settings.Aimbot.MouseBind
_G.AimbotKeyBind = Settings.Aimbot.KeyBind
_G.StickyAimEnabled = Settings.Aimbot.StickyAim
_G.AimbotToggle = Settings.Aimbot.Toggle
-- New global variables for Aimbot FOV
_G.AimbotFOVEnabled = Settings.Aimbot.FOV.Enabled
_G.AimbotFOVShape = Settings.Aimbot.FOV.Shape
_G.AimbotFOVSize = Settings.Aimbot.FOV.Size
_G.AimbotFOVMethod = Settings.Aimbot.FOV.Method  -- New global variable

-- New function to set the FOV method (modified)
_G.aimbotFOVSetMethod = function(method)
    if method == "FollowMouse" or method == "Center" then
        _G.AimbotFOVMethod = method
        -- Remove existing FOV indicator to prevent duplicates
        if aimbotFOVIndicator then
            aimbotFOVIndicator:Remove()
            aimbotFOVIndicator = nil
        end
    end
end

-- New: Function to update global settings from getgenv().Settings
local function refreshSettings()
    Settings = getgenv().Settings  -- Fix typo from 'Settingsngs'
    _G.ESPEnabled = Settings.ESP.Enabled
    _G.NameESPEnabled = Settings.NameESP.Enabled
    _G.OutlineEnabled = Settings.PlayerEffects.Enabled  -- Updated from PlayersMT
    _G.AimbotEnabled = Settings.Aimbot.Enabled
    _G.AimbotSmoothness = Settings.Aimbot.Smoothness
    _G.AimbotPart = Settings.Aimbot.TargetPart
    _G.AimbotKey = Settings.Aimbot.Key
    _G.AimbotMethod = Settings.Aimbot.Method
    _G.StickyAimEnabled = Settings.Aimbot.StickyAim
    _G.AimbotToggle = Settings.Aimbot.Toggle
    _G.AimbotFOVEnabled = Settings.Aimbot.FOV.Enabled
    _G.AimbotFOVShape = Settings.Aimbot.FOV.Shape
    _G.AimbotFOVSize = Settings.Aimbot.FOV.Size
    _G.AimbotFOVMethod = Settings.Aimbot.FOV.Method
    _G.TracerEnabled = Settings.Tracer.Enabled
    _G.TracerOrigin = Settings.Tracer.Origin
    _G.CFrameSpeedEnabled = Settings.CFrameSpeed.Enabled
    _G.CFrameSpeedValue = Settings.CFrameSpeed.Speed
    _G.OrbitEnabled = Settings.Orbit.Enabled
    _G.OrbitHeight = Settings.Orbit.Height
    _G.OrbitSpeed = Settings.Orbit.Speed
    _G.OrbitDistance = Settings.Orbit.Distance
    _G.OrbitFaceTarget = Settings.Orbit.FaceTarget
    _G.FlyEnabled = Settings.Fly.Enabled
    _G.FlySpeed = Settings.Fly.Speed
    _G.TeamCheckEnabled = Settings.TeamCheck.Enabled
    _G.AimbotBindType = Settings.Aimbot.BindType
    _G.AimbotMouseBind = Settings.Aimbot.MouseBind
    _G.AimbotKeyBind = Settings.Aimbot.KeyBind
end

-- Spawn a loop to refresh settings every second.
task.spawn(function()
    while true do
        refreshSettings()
        task.wait(1)
    end
end)

local ESPBoxes = {}
local NameLabels = {}
local Outlines = {}
local Tracers = {}
local HealthBars = {}  -- New table for health bar drawings
local IsAiming = false
local IsCFrameSpeedActive = false
local IsOrbiting = false
local OrbitAngle = 0
local IsFlyActive = false
local LastFlyPos = nil
local CurrentTarget = nil

local function IsTeammate(player)
    local localPlayer = Players.LocalPlayer
    if not _G.TeamCheckEnabled then return false end
    return player.Team and player.Team == localPlayer.Team
end

local function GetTracerOrigin()
    local camera = workspace.CurrentCamera
    local viewportSize = camera.ViewportSize
    
    if (_G.TracerOrigin == "Bottom") then
        return Vector2.new(viewportSize.X / 2, viewportSize.Y)
    elseif (_G.TracerOrigin == "Top") then
        return Vector2.new(viewportSize.X / 2, 0)
    else
        return UserInputService:GetMouseLocation()
    end
end

local function CreateESPBox(player)
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
    
    local isTeammate = IsTeammate(player)
    local color = _G.TeamCheckEnabled and 
        (isTeammate and Settings.TeamCheck.TeamColor or Settings.TeamCheck.EnemyColor) or 
        Settings.ESP.BoxColor
    
    local box = Drawing.new("Square")
    box.Visible = _G.ESPEnabled
    box.Color = color
    box.Thickness = Settings.ESP.BoxThickness
    box.Transparency = Settings.ESP.BoxTransparency
    box.Filled = false
    
    ESPBoxes[player] = box

    local nameText = Drawing.new("Text")
    nameText.Visible = _G.NameESPEnabled
    nameText.Color = _G.TeamCheckEnabled and color or Settings.NameESP.Color
    nameText.Size = Settings.NameESP.Size
    nameText.Center = true
    nameText.Outline = false
    nameText.Text = player.Name
    NameLabels[player] = nameText
    
    local highlight = Instance.new("Highlight")
    highlight.FillColor = Settings.PlayerEffects.FillColor
    highlight.FillTransparency = Settings.PlayerEffects.Mode == "Chams" 
        and Settings.PlayerEffects.ChamsTransparency 
        or Settings.PlayerEffects.FillTransparency
    highlight.OutlineColor = _G.TeamCheckEnabled and color or Settings.PlayerEffects.Color
    highlight.OutlineTransparency = Settings.PlayerEffects.Mode == "Outline"
        and (_G.OutlineEnabled and Settings.PlayerEffects.OutlineTransparency or 1)
        or 1
    highlight.Parent = player.Character
    Outlines[player] = highlight

    local tracer = Drawing.new("Line")
    tracer.Visible = _G.TracerEnabled
    tracer.Color = _G.TeamCheckEnabled and 
        (isTeammate and Settings.TeamCheck.TeamColor or Settings.TeamCheck.EnemyColor) or 
        Settings.Tracer.Color
    tracer.Thickness = Settings.Tracer.Thickness
    tracer.Transparency = Settings.Tracer.Transparency
    Tracers[player] = tracer

    if Settings.ESP.HealthBar and Settings.ESP.HealthBar.Enabled then
        local healthBar = Drawing.new("Square")
        healthBar.Visible = _G.ESPEnabled
        healthBar.Filled = true
        healthBar.Thickness = 0
        healthBar.Color = Settings.ESP.HealthBar.Color
        HealthBars[player] = healthBar
    end
    
    return box
end

local function UpdateESPBox(player, box)
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
        box.Visible = false
        if NameLabels[player] then NameLabels[player].Visible = false end
        if Tracers[player] then Tracers[player].Visible = false end
        if HealthBars[player] then HealthBars[player].Visible = false end
        return
    end
    
    local camera = workspace.CurrentCamera
    local vector, onScreen = camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
    
    if onScreen and _G.ESPEnabled then
        -- Use box dimensions from settings
        local size = Vector2.new(Settings.ESP.BoxSize.Width / vector.Z, Settings.ESP.BoxSize.Height / vector.Z)
        box.Size = size
        box.Position = Vector2.new(vector.X - size.X / 2, vector.Y - size.Y / 2)
        box.Visible = true

        if HealthBars[player] and Settings.ESP.HealthBar and Settings.ESP.HealthBar.Enabled then
            local humanoid = player.Character:FindFirstChild("Humanoid")
            if humanoid then
                local healthPerc = humanoid.Health / humanoid.MaxHealth
                local barWidth = 2  -- Fixed health bar width
                local barHeight = size.Y * healthPerc
                HealthBars[player].Size = Vector2.new(barWidth, barHeight)
                HealthBars[player].Position = Vector2.new(box.Position.X - barWidth - 2, box.Position.Y)
                HealthBars[player].Visible = true
            else
                HealthBars[player].Visible = false
            end
        end
    else
        box.Visible = false
        if HealthBars[player] then HealthBars[player].Visible = false end
    end
    
    if NameLabels[player] and onScreen then
        local head = player.Character:FindFirstChild("Head")
        if head then
            local headPos = camera:WorldToViewportPoint(head.Position + Settings.NameESP.Offset)
            NameLabels[player].Position = Vector2.new(headPos.X, headPos.Y)
            NameLabels[player].Visible = _G.NameESPEnabled
        end
    elseif NameLabels[player] then
        NameLabels[player].Visible = false
    end

    if Tracers[player] and onScreen then
        local origin = GetTracerOrigin()
        Tracers[player].From = origin
        Tracers[player].To = Vector2.new(vector.X, vector.Y)
        Tracers[player].Visible = _G.TracerEnabled
    elseif Tracers[player] then
        Tracers[player].Visible = false
    end
end

local function RemoveESPBox(player)
    if ESPBoxes[player] then
        if type(ESPBoxes[player].Remove) == "function" then
            ESPBoxes[player]:Remove()
        end
        ESPBoxes[player] = nil
    end
    if NameLabels[player] then
        if type(NameLabels[player].Remove) == "function" then
            NameLabels[player]:Remove()
        end
        NameLabels[player] = nil
    end
    if Outlines[player] then
        if type(Outlines[player].Destroy) == "function" then
            Outlines[player]:Destroy()
        end
        Outlines[player] = nil
    end
    if Tracers[player] then
        if type(Tracers[player].Remove) == "function" then
            Tracers[player]:Remove()
        end
        Tracers[player] = nil
    end
    if HealthBars[player] then
        if type(HealthBars[player].Remove) == "function" then
            HealthBars[player]:Remove()
        end
        HealthBars[player] = nil
    end
end

Players.PlayerAdded:Connect(function(player)
    if player ~= Players.LocalPlayer then
        if player.Character then
            CreateESPBox(player)
        end
        
        player.CharacterAdded:Connect(function(char)
            task.wait(0.5)
            if ESPBoxes[player] then
                RemoveESPBox(player)
            end
            CreateESPBox(player)
        end)
    end
end)

Players.PlayerRemoving:Connect(function(player)
    RemoveESPBox(player)
end)

for _, player in ipairs(Players:GetPlayers()) do
    if player ~= Players.LocalPlayer then
        local box = CreateESPBox(player)
        
        player.CharacterAdded:Connect(function()
            if ESPBoxes[player] then
                RemoveESPBox(player)
            end
            box = CreateESPBox(player)
        end)
    end
end

RunService.RenderStepped:Connect(function()
    for player, box in pairs(ESPBoxes) do
        UpdateESPBox(player, box)
    end
end)

_G.toggle = function()
    _G.ESPEnabled = not _G.ESPEnabled
    
    if _G.ESPEnabled then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= Players.LocalPlayer then
                if not ESPBoxes[player] then
                    CreateESPBox(player)
                else
                    ESPBoxes[player].Visible = true
                end
            end
        end
    else
        for player, box in pairs(ESPBoxes) do
            RemoveESPBox(player)
        end
    end
end

_G.nameESP = function()
    _G.NameESPEnabled = not _G.NameESPEnabled
    
    if _G.NameESPEnabled then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= Players.LocalPlayer and not NameLabels[player] then
                CreateESPBox(player)
            end
        end
    end
    
    for _, nameLabel in pairs(NameLabels) do
        if nameLabel and nameLabel.Remove then
            nameLabel.Visible = _G.NameESPEnabled
        end
    end
end

_G.playerEffects = function()
    _G.OutlineEnabled = not _G.OutlineEnabled
    
    for player, highlight in pairs(Outlines) do
        if highlight and highlight.Parent then
            if not _G.OutlineEnabled then
                -- When disabling, set both transparencies to 1 (invisible)
                highlight.OutlineTransparency = 1
                highlight.FillTransparency = 1
            else
                -- When enabling, apply proper transparencies based on mode
                if Settings.PlayerEffects.Mode == "Outline" then
                    highlight.OutlineTransparency = Settings.PlayerEffects.OutlineTransparency
                    highlight.FillTransparency = 1
                else  -- Chams mode
                    highlight.OutlineTransparency = 1
                    highlight.FillTransparency = Settings.PlayerEffects.ChamsTransparency
                end
            end
        end
    end
    
    if _G.OutlineEnabled then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= Players.LocalPlayer and not Outlines[player] and player.Character then
                CreateESPBox(player)
            end
        end
    end
end

_G.playerEffectsSetColor = function(newColor)
    if typeof(newColor) == "Color3" then
        for player, highlight in pairs(Outlines) do
            if highlight and highlight.Parent then
                highlight.FillColor = newColor
                highlight.OutlineColor = newColor
            end
        end
    end
end

_G.playerEffectsSetMode = function(mode)
    if mode == "Outline" or mode == "Chams" then
        Settings.PlayerEffects.Mode = mode
        for player, highlight in pairs(Outlines) do
            if highlight and highlight.Parent then
                if mode == "Outline" then
                    highlight.FillTransparency = 1
                    highlight.OutlineTransparency = _G.OutlineEnabled and Settings.PlayerEffects.OutlineTransparency or 1
                else -- Chams mode
                    highlight.FillTransparency = Settings.PlayerEffects.ChamsTransparency
                    highlight.OutlineTransparency = 1
                end
            end
        end
    end
end

_G.teamCheck = function()
    _G.TeamCheckEnabled = not _G.TeamCheckEnabled
    
    for player, box in pairs(ESPBoxes) do
        local isTeammate = IsTeammate(player)
        local color = _G.TeamCheckEnabled and 
            (isTeammate and Settings.TeamCheck.TeamColor or Settings.TeamCheck.EnemyColor) or 
            Settings.ESP.BoxColor
        
        if box then box.Color = color end
        if NameLabels[player] then NameLabels[player].Color = color or Settings.NameESP.Color end
        if Outlines[player] then Outlines[player].OutlineColor = color or Settings.PlayerEffects.Color end
        if Tracers[player] then Tracers[player].Color = color or Settings.Tracer.Color end
    end
end

_G.tracer = function()
    _G.TracerEnabled = not _G.TracerEnabled
    
    for _, tracer in pairs(Tracers) do
        if tracer and tracer.Remove then
            tracer.Visible = _G.TracerEnabled
        end
    end
end

_G.tracerOrigin = function(origin)
    if origin == "Bottom" or origin == "Top" or origin == "Mouse" then
        _G.TracerOrigin = origin
    end
end

local function GetFOVOrigin()
    if _G.AimbotFOVMethod == "FollowMouse" then
        return UserInputService:GetMouseLocation()
    else
        local camera = workspace.CurrentCamera
        local viewportSize = camera.ViewportSize
        return Vector2.new(viewportSize.X/2, viewportSize.Y/2)
    end
end

local function GetClosestPlayer()
    -- If sticky aim is enabled and we have a valid target, stick to it
    if _G.StickyAimEnabled and CurrentTarget and 
       CurrentTarget.Character and 
       CurrentTarget.Character:FindFirstChild(_G.AimbotPart) then
        local camera = workspace.CurrentCamera
        local _, onScreen = camera:WorldToViewportPoint(CurrentTarget.Character[_G.AimbotPart].Position)
        if onScreen then
            return CurrentTarget
        else
            -- Only reset target if they go off screen
            CurrentTarget = nil
        end
    end

    -- Only find new target if we don't have one and sticky aim is enabled, or if sticky aim is disabled
    if (not _G.StickyAimEnabled) or (_G.StickyAimEnabled and not CurrentTarget) then
        local closestPlayer = nil
        local shortestDistance = math.huge
        local localPlayer = Players.LocalPlayer
        local fovOrigin = GetFOVOrigin()  -- use the FOV origin based on method
        local camera = workspace.CurrentCamera

        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= localPlayer and player.Character and player.Character:FindFirstChild(_G.AimbotPart) then
                local partPos = camera:WorldToViewportPoint(player.Character[_G.AimbotPart].Position)
                if partPos.Z > 0 then
                    if _G.AimbotFOVEnabled then
                        if _G.AimbotFOVShape == "Circle" then
                            local fovDist = (Vector2.new(fovOrigin.X, fovOrigin.Y) - Vector2.new(partPos.X, partPos.Y)).Magnitude
                            if fovDist > _G.AimbotFOVSize then
                                -- Skip this candidate if outside circular FOV
                            else
                                local distance = fovDist
                                if distance < shortestDistance then
                                    closestPlayer = player
                                    shortestDistance = distance
                                end
                            end
                        elseif _G.AimbotFOVShape == "Box" then
                            if math.abs(fovOrigin.X - partPos.X) > _G.AimbotFOVSize/2 or math.abs(fovOrigin.Y - partPos.Y) > _G.AimbotFOVSize/2 then
                                -- Skip this candidate if outside square FOV
                            else
                                local distance = (Vector2.new(fovOrigin.X, fovOrigin.Y) - Vector2.new(partPos.X, partPos.Y)).Magnitude
                                if distance < shortestDistance then
                                    closestPlayer = player
                                    shortestDistance = distance
                                end
                            end
                        end
                    else
                        local distance = (Vector2.new(fovOrigin.X, fovOrigin.Y) - Vector2.new(partPos.X, partPos.Y)).Magnitude
                        if distance < shortestDistance then
                            closestPlayer = player
                            shortestDistance = distance
                        end
                    end
                end
            end
        end
        
        -- Only update CurrentTarget if sticky aim is disabled
        if not _G.StickyAimEnabled then
            CurrentTarget = closestPlayer
        else
            -- Set initial target for sticky aim
            if not CurrentTarget then
                CurrentTarget = closestPlayer
            end
        end
        return closestPlayer
    end
    
    return CurrentTarget
end

local function AimAt(targetPos)
    local camera = workspace.CurrentCamera
    local mousePos = UserInputService:GetMouseLocation()
    local targetPoint = camera:WorldToViewportPoint(targetPos)
    
    local smoothness = (_G.AimbotSmoothness * _G.AimbotSmoothness) * 0.8
    
    if (_G.AimbotMethod == "Mouse") then
        local targetVector = Vector2.new(targetPoint.X, targetPoint.Y) - Vector2.new(mousePos.X, mousePos.Y)
        mousemoverel(
            targetVector.X * smoothness,
            targetVector.Y * smoothness
        )
    else
        local currentCFrame = camera.CFrame
        local targetCFrame = CFrame.new(camera.CFrame.Position, targetPos)
        camera.CFrame = currentCFrame:Lerp(targetCFrame, smoothness)
    end
end

RunService.RenderStepped:Connect(function()
    if _G.AimbotEnabled and IsAiming then
        local target = GetClosestPlayer()
        if target and target.Character and target.Character:FindFirstChild(_G.AimbotPart) then
            AimAt(target.Character[_G.AimbotPart].Position)
        end
    end
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    local isAimbotInput = (_G.AimbotBindType == "Mouse" and input.UserInputType == _G.AimbotMouseBind) or
                         (_G.AimbotBindType == "Key" and input.KeyCode == _G.AimbotKeyBind)
    
    if isAimbotInput then
        if _G.AimbotToggle then
            IsAiming = not IsAiming
            if not IsAiming then
                CurrentTarget = nil
            end
        else
            IsAiming = true
        end
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    local isAimbotInput = (_G.AimbotBindType == "Mouse" and input.UserInputType == _G.AimbotMouseBind) or
                         (_G.AimbotBindType == "Key" and input.KeyCode == _G.AimbotKeyBind)
    
    if isAimbotInput and not _G.AimbotToggle then
        IsAiming = false
        if not _G.StickyAimEnabled then
            CurrentTarget = nil
        end
    end
end)

_G.aimbot = function()
    _G.AimbotEnabled = not _G.AimbotEnabled
end

_G.aimbotSmoothness = function(value)
    _G.AimbotSmoothness = math.clamp(value, 0, 1)
end

_G.aimbotPart = function(part)
    if part == "Head" or part == "HumanoidRootPart" or part == "Torso" then
        _G.AimbotPart = part
    end
end

_G.aimbotMethod = function(method)
    if method == "Mouse" or method == "Camera" then
        _G.AimbotMethod = method
    end
end

_G.aimbotBindType = function(bindType)
    if bindType == "Mouse" or bindType == "Key" then
        _G.AimbotBindType = bindType
    end
end

_G.aimbotMouseBind = function(mouseBind)
    if typeof(mouseBind) == "EnumItem" and mouseBind.EnumType == Enum.UserInputType then
        _G.AimbotMouseBind = mouseBind
    end
end

_G.aimbotKeyBind = function(keyBind)
    if typeof(keyBind) == "EnumItem" and keyBind.EnumType == Enum.KeyCode then
        _G.AimbotKeyBind = keyBind
    end
end

_G.stickyAim = function()
    _G.StickyAimEnabled = not _G.StickyAimEnabled
    CurrentTarget = nil
end

_G.aimbotToggleMode = function()
    _G.AimbotToggle = not _G.AimbotToggle
    if not _G.AimbotToggle then
        IsAiming = false
        if not _G.StickyAimEnabled then
            CurrentTarget = nil
        end
    end
end

-- New functions for Aimbot FOV control
_G.aimbotFOVToggle = function()
    _G.AimbotFOVEnabled = not _G.AimbotFOVEnabled
end

_G.aimbotFOVSetShape = function(shape)
    if shape == "Circle" or shape == "Box" then
        _G.AimbotFOVShape = shape
    end
end

_G.aimbotFOVSetSize = function(size)
    _G.AimbotFOVSize = size
end

local function ApplyCFrameSpeed()
    local humanoid = Players.LocalPlayer.Character and Players.LocalPlayer.Character:FindFirstChild("Humanoid")
    local rootPart = Players.LocalPlayer.Character and Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    
    if humanoid and rootPart and _G.CFrameSpeedEnabled and IsCFrameSpeedActive then
        local moveDirection = humanoid.MoveDirection
        if moveDirection.Magnitude > 0 then
            rootPart.CFrame = rootPart.CFrame + (moveDirection * _G.CFrameSpeedValue)
        end
    end
end

RunService.Heartbeat:Connect(ApplyCFrameSpeed)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Settings.CFrameSpeed.Key then
        if Settings.CFrameSpeed.HoldToUse then
            IsCFrameSpeedActive = true
        else
            IsCFrameSpeedActive = not IsCFrameSpeedActive
        end
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Settings.CFrameSpeed.Key and Settings.CFrameSpeed.HoldToUse then
        IsCFrameSpeedActive = false
    end
end)

_G.cframespeed = function(speed)
    _G.CFrameSpeedValue = math.clamp(speed, 0, 10)
end

local function UpdateOrbit()
    local localPlayer = Players.LocalPlayer
    if not localPlayer.Character or not localPlayer.Character:FindFirstChild("HumanoidRootPart") then return end
    
    local target = GetClosestPlayer()
    if not target or not target.Character or not target.Character:FindFirstChild("HumanoidRootPart") then return end
    
    if _G.OrbitEnabled and IsOrbiting then
        OrbitAngle = OrbitAngle + _G.OrbitSpeed * 0.1
        local targetPos = target.Character.HumanoidRootPart.Position
        local orbitPos = Vector3.new(
            targetPos.X + math.cos(OrbitAngle) * _G.OrbitDistance,
            targetPos.Y + _G.OrbitHeight,
            targetPos.Z + math.sin(OrbitAngle) * _G.OrbitDistance
        )
        
        if _G.OrbitFaceTarget then
            localPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(orbitPos, targetPos)
        else
            localPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(orbitPos)
        end
    end
end

RunService.Heartbeat:Connect(UpdateOrbit)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Settings.Orbit.Key then
        if Settings.Orbit.HoldToUse then
            IsOrbiting = true
        else
            IsOrbiting = not IsOrbiting
        end
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Settings.Orbit.Key and Settings.Orbit.HoldToUse then
        IsOrbiting = false
    end
end)

_G.orbit = function()
    _G.OrbitEnabled = not _G.OrbitEnabled
end

_G.orbitHeight = function(height)
    _G.OrbitHeight = height
end

_G.orbitSpeed = function(speed)
    _G.OrbitSpeed = math.clamp(speed, 0.1, 10)
end

_G.orbitDistance = function(distance)
    _G.OrbitDistance = math.clamp(distance, 1, 50)
end

_G.orbitFace = function()
    _G.OrbitFaceTarget = not _G.OrbitFaceTarget
end

local function StartFly()
    local character = Players.LocalPlayer.Character
    if not character then return end
    
    local humanoid = character:FindFirstChild("Humanoid")
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    
    if humanoid and rootPart then
        LastFlyPos = rootPart.Position
        humanoid:ChangeState(Enum.HumanoidStateType.Swimming)
    end
end

local function StopFly()
    local character = Players.LocalPlayer.Character
    if not character then return end
    
    local humanoid = character:FindFirstChild("Humanoid")
    if humanoid then
        LastFlyPos = nil
        humanoid:ChangeState(Enum.HumanoidStateType.Running)
    end
end

local function UpdateFly()
    local character = Players.LocalPlayer.Character
    if not character then return end
    
    local humanoid = character:FindFirstChild("Humanoid")
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    
    if not humanoid or not rootPart or not _G.FlyEnabled or not IsFlyActive then
        return
    end

    local camera = workspace.CurrentCamera
    local moveVector = Vector3.new()
    local multiplier = Settings.Fly.Multiplier

    -- Forward/Back
    if UserInputService:IsKeyDown(Enum.KeyCode.W) then
        moveVector += camera.CFrame.LookVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.S) then
        moveVector -= camera.CFrame.LookVector
    end

    -- Left/Right
    if UserInputService:IsKeyDown(Enum.KeyCode.A) then
        moveVector -= camera.CFrame.RightVector
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.D) then
        moveVector += camera.CFrame.RightVector
    end

    -- Up/Down
    if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
        moveVector += Vector3.new(0, 1, 0)
    end
    if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
        moveVector -= Vector3.new(0, 1, 0)
    end

    if moveVector.Magnitude > 0 then
        moveVector = moveVector.Unit
    end

    -- Apply movement
    rootPart.Velocity = moveVector * (_G.FlySpeed * multiplier * 16)
    rootPart.CFrame = CFrame.new(rootPart.Position, rootPart.Position + camera.CFrame.LookVector)
end

RunService.Heartbeat:Connect(UpdateFly)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Settings.Fly.Key then
        if _G.FlyEnabled then
            IsFlyActive = not IsFlyActive
            if IsFlyActive then
                StartFly()
            else
                StopFly()
            end
        end
    end
end)

_G.fly = function()
    _G.FlyEnabled = not _G.FlyEnabled
    if not _G.FlyEnabled and IsFlyActive then
        IsFlyActive = false
        StopFly()
    end
end

_G.flySpeed = function(speed)
    _G.FlySpeed = math.clamp(speed, 0.1, 20)
end

local aimbotFOVIndicator = nil
RunService.RenderStepped:Connect(function()
    if _G.AimbotFOVEnabled then
        if not aimbotFOVIndicator then
            if _G.AimbotFOVShape == "Circle" then
                aimbotFOVIndicator = Drawing.new("Circle")
            else
                aimbotFOVIndicator = Drawing.new("Square")
            end
            aimbotFOVIndicator.Visible = true
            aimbotFOVIndicator.Color = Color3.fromRGB(255, 0, 0)
            aimbotFOVIndicator.Transparency = 0.8
            aimbotFOVIndicator.Filled = false
            aimbotFOVIndicator.Thickness = 1
        else
            if (_G.AimbotFOVShape == "Circle" and aimbotFOVIndicator.ClassName ~= "Circle") or 
               (_G.AimbotFOVShape == "Box" and aimbotFOVIndicator.ClassName ~= "Square") then
                aimbotFOVIndicator:Remove()
                if _G.AimbotFOVShape == "Circle" then
                    aimbotFOVIndicator = Drawing.new("Circle")
                else
                    aimbotFOVIndicator = Drawing.new("Square")
                end
                aimbotFOVIndicator.Visible = true
                aimbotFOVIndicator.Color = Color3.fromRGB(255, 0, 0)
                aimbotFOVIndicator.Transparency = 0.8
                aimbotFOVIndicator.Filled = false
                aimbotFOVIndicator.Thickness = 1
            end
        end

        local origin = GetFOVOrigin()  -- use new FOV origin
        if _G.AimbotFOVShape == "Circle" then
            aimbotFOVIndicator.Radius = _G.AimbotFOVSize
            aimbotFOVIndicator.Position = origin
        else
            aimbotFOVIndicator.Size = Vector2.new(_G.AimbotFOVSize, _G.AimbotFOVSize)
            aimbotFOVIndicator.Position = Vector2.new(origin.X - _G.AimbotFOVSize/2, origin.Y - _G.AimbotFOVSize/2)
        end
    else
        if aimbotFOVIndicator then
            aimbotFOVIndicator.Visible = false
        end
    end
end)
