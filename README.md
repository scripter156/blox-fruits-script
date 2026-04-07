--[[
	WARNING: Heads up! This script has not been verified by ScriptBlox. Use at your own risk!
]]
--[[ 
THIS IS MADE BY C00L_SAM
--]]

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")
local Debris = game:GetService("Debris")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local camera = workspace.CurrentCamera

-- Lighting base (optimized for realism)
Lighting.Brightness = 2
Lighting.GlobalShadows = true
Lighting.EnvironmentDiffuseScale = 1
Lighting.EnvironmentSpecularScale = 1.1
Lighting.ClockTime = 10
Lighting.Technology = Enum.Technology.Future
Lighting.Ambient = Color3.fromRGB(90, 100, 120)
Lighting.OutdoorAmbient = Color3.fromRGB(140, 160, 190)
Lighting.ShadowSoftness = 0.15

for _, effect in pairs(Lighting:GetChildren()) do
    if effect:IsA("PostEffect") or effect:IsA("Atmosphere") then effect:Destroy() end
end

-- Core effects
local Bloom = Instance.new("BloomEffect", Lighting) Bloom.Intensity = 1.4 Bloom.Size = 28 Bloom.Threshold = 1.0
local SunRays = Instance.new("SunRaysEffect", Lighting) SunRays.Intensity = 0.28 SunRays.Spread = 0.75
local Depth = Instance.new("DepthOfFieldEffect", Lighting) Depth.FarIntensity = 0.4 Depth.InFocusRadius = 60 Depth.FocusDistance = 45 Depth.NearIntensity = 0.35
local Blur = Instance.new("BlurEffect", Lighting) Blur.Size = 3
local ColorCorr = Instance.new("ColorCorrectionEffect", Lighting) ColorCorr.Brightness = 0.12 ColorCorr.Contrast = 0.45 ColorCorr.Saturation = 0.55 ColorCorr.TintColor = Color3.fromRGB(245, 245, 255)
local Atmos = Instance.new("Atmosphere", Lighting) Atmos.Density = 0.22 Atmos.Offset = 0.18 Atmos.Color = Color3.fromRGB(210, 225, 245) Atmos.Decay = Color3.fromRGB(180, 200, 230) Atmos.Glare = 0.5 Atmos.Haze = 3.5

-- Rain particles & sound (unchanged)
local rainPartFolder = Instance.new("Folder") rainPartFolder.Name = "RainParts" rainPartFolder.Parent = camera
local rainSound = Instance.new("Sound", camera) rainSound.SoundId = "rbxassetid://184766182" rainSound.Volume = 0.4 rainSound.Looped = true

local function startRain()
    rainSound:Play()
    RunService:BindToRenderStep("RainUpdate", Enum.RenderPriority.Camera.Value + 1, function()
        for i = 1, 40 do
            local drop = Instance.new("Part")
            drop.Anchored = false
            drop.CanCollide = false
            drop.Transparency = 0.3
            drop.Color = Color3.fromRGB(180, 200, 255)
            drop.Material = Enum.Material.Glass
            drop.Size = Vector3.new(0.15, 1.2, 0.15)
            drop.Position = camera.CFrame.Position + Vector3.new(math.random(-200, 200), 100, math.random(-200, 200))
            drop.Velocity = Vector3.new(0, -80, 0)
            drop.Parent = rainPartFolder
            Debris:AddItem(drop, 3)

            local splash = Instance.new("ParticleEmitter")
            splash.Texture = "rbxassetid://243660364"
            splash.Color = ColorSequence.new(Color3.fromRGB(200, 220, 255))
            splash.Lifetime = NumberRange.new(0.4, 0.6)
            splash.Rate = 0
            splash.Speed = NumberRange.new(5, 10)
            splash.SpreadAngle = Vector2.new(360, 360)
            splash.Enabled = true
            splash.Parent = drop
            game.Debris:AddItem(splash, 1)
        end
    end)
end

local function stopRain()
    rainSound:Stop()
    RunService:UnbindFromRenderStep("RainUpdate")
    for _, p in pairs(rainPartFolder:GetChildren()) do p:Destroy() end
end

-- Presets (REALIZED moved out, will be toggle)
local presets = {
    DAY = {tint = Color3.fromRGB(245, 240, 230), bright = 0.18, contrast = 0.35, sat = 0.4, density = 0.35, atmosColor = Color3.fromRGB(220, 200, 170), decay = Color3.fromRGB(120, 100, 80), bloomInt = 1.6, sunInt = 0.3, blurSize = 5, dofFar = 0.5, dofNear = 0.4, glare = 0.45, clock = 14},
    NIGHT = {tint = Color3.fromRGB(200, 210, 245), bright = 0.0, contrast = 0.45, sat = 0.35, density = 0.45, atmosColor = Color3.fromRGB(80, 90, 140), decay = Color3.fromRGB(50, 60, 100), bloomInt = 1.4, sunInt = 0.18, blurSize = 6, dofFar = 0.55, dofNear = 0.45, glare = 0.3, clock = 0},
    CINEMATIC = {tint = Color3.fromRGB(245, 235, 200), bright = 0.12, contrast = 0.4, sat = 0.5, density = 0.4, atmosColor = Color3.fromRGB(180, 160, 140), decay = Color3.fromRGB(90, 80, 70), bloomInt = 1.9, sunInt = 0.35, blurSize = 5.5, dofFar = 0.6, dofNear = 0.5, glare = 0.4, clock = 18},
    SHARP = {tint = Color3.fromRGB(250, 250, 240), bright = 0.22, contrast = 0.4, sat = 0.45, density = 0.28, atmosColor = Color3.fromRGB(230, 210, 180), decay = Color3.fromRGB(130, 110, 90), bloomInt = 1.8, sunInt = 0.28, blurSize = 2, dofFar = 0.25, dofNear = 0.2, glare = 0.35, clock = 14},
    NEON = {tint = Color3.fromRGB(200, 255, 255), bright = 0.3, contrast = 0.6, sat = 0.8, density = 0.2, atmosColor = Color3.fromRGB(0, 200, 255), decay = Color3.fromRGB(0, 100, 200), bloomInt = 2.5, sunInt = 0.4, blurSize = 8, dofFar = 0.7, dofNear = 0.6, glare = 0.8, clock = 22},
    RAIN = {tint = Color3.fromRGB(180, 200, 220), bright = -0.1, contrast = 0.3, sat = 0.2, density = 0.55, atmosColor = Color3.fromRGB(100, 120, 150), decay = Color3.fromRGB(70, 90, 110), bloomInt = 1.3, sunInt = 0.1, blurSize = 7, dofFar = 0.75, dofNear = 0.65, glare = 0.35, clock = 12},
    FOG = {tint = Color3.fromRGB(200, 210, 230), bright = -0.05, contrast = 0.25, sat = 0.05, density = 0.7, atmosColor = Color3.fromRGB(160, 180, 220), decay = Color3.fromRGB(120, 140, 180), bloomInt = 0.8, sunInt = 0.05, blurSize = 4, dofFar = 0.6, dofNear = 0.5, glare = 0.2, clock = 9},
    ULTRA = {tint = Color3.fromRGB(255, 245, 235), bright = 0.4, contrast = 0.7, sat = 0.9, density = 0.25, atmosColor = Color3.fromRGB(250, 200, 150), decay = Color3.fromRGB(200, 150, 100), bloomInt = 3.0, sunInt = 0.5, blurSize = 4, dofFar = 0.4, dofNear = 0.3, glare = 1.0, clock = 15}
}

local tweenInfo = TweenInfo.new(1.2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut)
local currentPreset = "DAY"
local fpsBoostActive = false
local realizedActive = false

local function applyPreset(name)
    local p = presets[name]
    if not p then return end
    currentPreset = name

    local mod = fpsBoostActive and 0.1 or 1
    local realMod = realizedActive and 1.4 or 1  -- boost for realism

    TweenService:Create(ColorCorr, tweenInfo, {
        TintColor = p.tint,
        Brightness = (p.bright * mod) * realMod,
        Contrast = (p.contrast * mod) * realMod,
        Saturation = (p.sat * mod) * realMod
    }):Play()

    TweenService:Create(Atmos, tweenInfo, {
        Density = p.density * mod,
        Color = p.atmosColor,
        Decay = p.decay,
        Glare = p.glare * mod
    }):Play()

    TweenService:Create(Bloom, tweenInfo, {Intensity = (p.bloomInt * mod) * realMod, Threshold = realizedActive and 1.2 or 0.8}):Play()
    TweenService:Create(SunRays, tweenInfo, {Intensity = (p.sunInt * mod) * realMod}):Play()
    TweenService:Create(Blur, tweenInfo, {Size = p.blurSize * mod}):Play()
    TweenService:Create(Depth, tweenInfo, {
        FarIntensity = p.dofFar * mod,
        NearIntensity = p.dofNear * mod,
        InFocusRadius = realizedActive and 80 or 50
    }):Play()

    TweenService:Create(Lighting, tweenInfo, {ClockTime = realizedActive and 10 or p.clock}):Play()

    Lighting.GlobalShadows = not fpsBoostActive
    Lighting.EnvironmentDiffuseScale = (fpsBoostActive and 0.2 or 0.9) * realMod
    Lighting.EnvironmentSpecularScale = (fpsBoostActive and 0.2 or 1.1) * realMod
    Lighting.Brightness = (fpsBoostActive and 0.8 or 2) * realMod
    Lighting.ShadowSoftness = fpsBoostActive and 0 or 0.12

    SunRays.Enabled = not fpsBoostActive
    Depth.Enabled = not fpsBoostActive

    if name == "RAIN" then
        startRain()
    else
        stopRain()
    end
end

local function toggleFpsBoost()
    fpsBoostActive = not fpsBoostActive
    applyPreset(currentPreset)
    if fpsBoostBtn then
        fpsBoostBtn.Text = fpsBoostActive and "FPS BOOST: ON (MAX)" or "FPS BOOST: OFF"
        fpsBoostBtn.BackgroundColor3 = fpsBoostActive and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
    end
end

local function toggleRealized()
    realizedActive = not realizedActive
    applyPreset(currentPreset)
    if realizedBtn then
        realizedBtn.Text = "REALIZED: " .. (realizedActive and "ON" or "OFF")
        realizedBtn.BackgroundColor3 = realizedActive and Color3.fromRGB(0, 220, 100) or Color3.fromRGB(20, 20, 20)
    end
end

-- GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "UltraRTXv9"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.Parent = playerGui

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 500, 0, 340)
mainFrame.Position = UDim2.new(0.5, -250, 0.5, -170)
mainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
mainFrame.BorderColor3 = Color3.fromRGB(255, 0, 0)
mainFrame.BorderSizePixel = 3
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
title.Text = "ULTRA RTX HD V9 BY SAM"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.Cartoon
title.TextSize = 28
title.Parent = mainFrame

-- Left - Presets
local left = Instance.new("Frame")
left.Size = UDim2.new(0.5, 0, 1, -40)
left.Position = UDim2.new(0, 0, 0, 40)
left.BackgroundTransparency = 1
left.Parent = mainFrame

local leftTitle = Instance.new("TextLabel")
leftTitle.Size = UDim2.new(1, 0, 0, 30)
leftTitle.Text = "PRESETS"
leftTitle.TextColor3 = Color3.fromRGB(255, 0, 0)
leftTitle.Font = Enum.Font.Cartoon
leftTitle.TextSize = 24
leftTitle.Parent = left

local presetList = {"DAY", "NIGHT", "CINEMATIC", "SHARP", "NEON", "RAIN", "FOG", "ULTRA"}

for i, name in ipairs(presetList) do
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -10, 0, 28)
    btn.Position = UDim2.new(0, 5, 0, 30 + (i-1)*32)
    btn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    btn.BorderColor3 = Color3.fromRGB(255, 0, 0)
    btn.BorderSizePixel = 1
    btn.Text = name
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.Cartoon
    btn.TextSize = 20
    btn.Parent = left

    btn.MouseButton1Click:Connect(function() applyPreset(name) end)
end

-- Right - Settings
local right = Instance.new("Frame")
right.Size = UDim2.new(0.5, 0, 1, -40)
right.Position = UDim2.new(0.5, 0, 0, 40)
right.BackgroundTransparency = 1
right.Parent = mainFrame

local rightTitle = Instance.new("TextLabel")
rightTitle.Size = UDim2.new(1, 0, 0, 30)
rightTitle.Text = "SETTINGS"
rightTitle.TextColor3 = Color3.fromRGB(255, 0, 0)
rightTitle.Font = Enum.Font.Cartoon
rightTitle.TextSize = 24
rightTitle.Parent = right

local fpsBoostBtn = Instance.new("TextButton")
fpsBoostBtn.Size = UDim2.new(1, -10, 0, 30)
fpsBoostBtn.Position = UDim2.new(0, 5, 0, 30)
fpsBoostBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
fpsBoostBtn.BorderColor3 = Color3.fromRGB(255, 0, 0)
fpsBoostBtn.BorderSizePixel = 1
fpsBoostBtn.Text = "FPS BOOST: OFF"
fpsBoostBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
fpsBoostBtn.Font = Enum.Font.Cartoon
fpsBoostBtn.TextSize = 20
fpsBoostBtn.Parent = right

fpsBoostBtn.MouseButton1Click:Connect(toggleFpsBoost)

-- REALIZED toggle (now in Settings)
local realizedBtn = Instance.new("TextButton")
realizedBtn.Size = UDim2.new(1, -10, 0, 30)
realizedBtn.Position = UDim2.new(0, 5, 0, 65)
realizedBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
realizedBtn.BorderColor3 = Color3.fromRGB(255, 0, 0)
realizedBtn.BorderSizePixel = 1
realizedBtn.Text = "REALIZED: OFF"
realizedBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
realizedBtn.Font = Enum.Font.Cartoon
realizedBtn.TextSize = 20
realizedBtn.Parent = right

realizedBtn.MouseButton1Click:Connect(toggleRealized)

-- Close button
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(1, 0, 0, 35)
closeBtn.Position = UDim2.new(0, 0, 1, -35)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
closeBtn.Text = "CLOSE"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.Cartoon
closeBtn.TextSize = 26
closeBtn.Parent = mainFrame

closeBtn.MouseButton1Click:Connect(function()
    screenGui:Destroy()
    if fpsGui then fpsGui:Destroy() end
    stopRain()
    for _, eff in pairs(Lighting:GetChildren()) do
        if eff:IsA("PostEffect") or eff:IsA("Atmosphere") then eff:Destroy() end
    end
end)

-- Separate FPS counter
local fpsGui = Instance.new("ScreenGui")
fpsGui.Name = "FpsCounterGui"
fpsGui.Parent = playerGui

local fpsFrame = Instance.new("Frame")
fpsFrame.Size = UDim2.new(0, 140, 0, 45)
fpsFrame.Position = UDim2.new(0.92, -140, 0.04, 0)
fpsFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
fpsFrame.BorderColor3 = Color3.fromRGB(255, 0, 0)
fpsFrame.BorderSizePixel = 2
fpsFrame.Active = true
fpsFrame.Draggable = true
fpsFrame.Parent = fpsGui

local fpsText = Instance.new("TextLabel")
fpsText.Size = UDim2.new(1, 0, 1, 0)
fpsText.BackgroundTransparency = 1
fpsText.Text = "FPS: 0"
fpsText.TextColor3 = Color3.fromRGB(0, 255, 0)
fpsText.Font = Enum.Font.Cartoon
fpsText.TextSize = 26
fpsText.Parent = fpsFrame

local last = tick()
RunService.Heartbeat:Connect(function()
    local dt = tick() - last
    last = tick()
    local fps = math.floor(1 / dt + 0.5)
    fpsText.Text = "FPS: " .. fps
end)

-- Init
applyPreset("DAY")

print("ULTRA RTX HD V9 BY SAM - REALIZED toggle added to Settings. Minecraft-shader realism (sharp textures, no orange sunset). Rain & fog unchanged. Enjoy!")
    CINEMATIC = {tint = Color3.fromRGB(245, 235, 200), bright = 0.12, contrast = 0.4, sat = 0.5, density = 0.4, atmosColor = Color3.fromRGB(180, 160, 140), decay = Color3.fromRGB(90, 80, 70), bloomInt = 1.9, sunInt = 0.35, blurSize = 5.5, dofFar = 0.6, dofNear = 0.5, glare = 0.4, clock = 18},
    SHARP = {tint = Color3.fromRGB(250, 250, 240), bright = 0.22, contrast = 0.4, sat = 0.45, density = 0.28, atmosColor = Color3.fromRGB(230, 210, 180), decay = Color3.fromRGB(130, 110, 90), bloomInt = 1.8, sunInt = 0.28, blurSize = 2, dofFar = 0.25, dofNear = 0.2, glare = 0.35, clock = 14},
    NEON = {tint = Color3.fromRGB(200, 255, 255), bright = 0.3, contrast = 0.6, sat = 0.8, density = 0.2, atmosColor = Color3.fromRGB(0, 200, 255), decay = Color3.fromRGB(0, 100, 200), bloomInt = 2.5, sunInt = 0.4, blurSize = 8, dofFar = 0.7, dofNear = 0.6, glare = 0.8, clock = 22},
    RAIN = {tint = Color3.fromRGB(180, 200, 220), bright = -0.1, contrast = 0.3, sat = 0.2, density = 0.55, atmosColor = Color3.fromRGB(100, 120, 150), decay = Color3.fromRGB(70, 90, 110), bloomInt = 1.3, sunInt = 0.1, blurSize = 7, dofFar = 0.75, dofNear = 0.65, glare = 0.35, clock = 12},
    FOG = {tint = Color3.fromRGB(200, 210, 230), bright = -0.05, contrast = 0.25, sat = 0.05, density = 0.7, atmosColor = Color3.fromRGB(160, 180, 220), decay = Color3.fromRGB(120, 140, 180), bloomInt = 0.8, sunInt = 0.05, blurSize = 4, dofFar = 0.6, dofNear = 0.5, glare = 0.2, clock = 9},
    ULTRA = {tint = Color3.fromRGB(255, 245, 235), bright = 0.4, contrast = 0.7, sat = 0.9, density = 0.25, atmosColor = Color3.fromRGB(250, 200, 150), decay = Color3.fromRGB(200, 150, 100), bloomInt = 3.0, sunInt = 0.5, blurSize = 4, dofFar = 0.4, dofNear = 0.3, glare = 1.0, clock = 15}
}

local tweenInfo = TweenInfo.new(1.2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut)
local currentPreset = "DAY"
local fpsBoostActive = false
local realizedActive = false

local function applyPreset(name)
    local p = presets[name]
    if not p then return end
    currentPreset = name

    local mod = fpsBoostActive and 0.1 or 1
    local realMod = realizedActive and 1.4 or 1  -- boost for realism

    TweenService:Create(ColorCorr, tweenInfo, {
        TintColor = p.tint,
        Brightness = (p.bright * mod) * realMod,
        Contrast = (p.contrast * mod) * realMod,
        Saturation = (p.sat * mod) * realMod
    }):Play()

    TweenService:Create(Atmos, tweenInfo, {
        Density = p.density * mod,
        Color = p.atmosColor,
        Decay = p.decay,
        Glare = p.glare * mod
    }):Play()

    TweenService:Create(Bloom, tweenInfo, {Intensity = (p.bloomInt * mod) * realMod, Threshold = realizedActive and 1.2 or 0.8}):Play()
    TweenService:Create(SunRays, tweenInfo, {Intensity = (p.sunInt * mod) * realMod}):Play()
    TweenService:Create(Blur, tweenInfo, {Size = p.blurSize * mod}):Play()
    TweenService:Create(Depth, tweenInfo, {
        FarIntensity = p.dofFar * mod,
        NearIntensity = p.dofNear * mod,
        InFocusRadius = realizedActive and 80 or 50
    }):Play()

    TweenService:Create(Lighting, tweenInfo, {ClockTime = realizedActive and 10 or p.clock}):Play()

    Lighting.GlobalShadows = not fpsBoostActive
    Lighting.EnvironmentDiffuseScale = (fpsBoostActive and 0.2 or 0.9) * realMod
    Lighting.EnvironmentSpecularScale = (fpsBoostActive and 0.2 or 1.1) * realMod
    Lighting.Brightness = (fpsBoostActive and 0.8 or 2) * realMod
    Lighting.ShadowSoftness = fpsBoostActive and 0 or 0.12

    SunRays.Enabled = not fpsBoostActive
    Depth.Enabled = not fpsBoostActive

    if name == "RAIN" then
        startRain()
    else
        stopRain()
    end
end

local function toggleFpsBoost()
    fpsBoostActive = not fpsBoostActive
    applyPreset(currentPreset)
    if fpsBoostBtn then
        fpsBoostBtn.Text = fpsBoostActive and "FPS BOOST: ON (MAX)" or "FPS BOOST: OFF"
        fpsBoostBtn.BackgroundColor3 = fpsBoostActive and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
    end
end

local function toggleRealized()
    realizedActive = not realizedActive
    applyPreset(currentPreset)
    if realizedBtn then
        realizedBtn.Text = "REALIZED: " .. (realizedActive and "ON" or "OFF")
        realizedBtn.BackgroundColor3 = realizedActive and Color3.fromRGB(0, 220, 100) or Color3.fromRGB(20, 20, 20)
    end
end

-- GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "UltraRTXv9"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.Parent = playerGui

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 500, 0, 340)
mainFrame.Position = UDim2.new(0.5, -250, 0.5, -170)
mainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
mainFrame.BorderColor3 = Color3.fromRGB(255, 0, 0)
mainFrame.BorderSizePixel = 3
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
title.Text = "ULTRA RTX HD V9 BY SAM"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.Cartoon
title.TextSize = 28
title.Parent = mainFrame

-- Left - Presets
local left = Instance.new("Frame")
left.Size = UDim2.new(0.5, 0, 1, -40)
left.Position = UDim2.new(0, 0, 0, 40)
left.BackgroundTransparency = 1
left.Parent = mainFrame

local leftTitle = Instance.new("TextLabel")
leftTitle.Size = UDim2.new(1, 0, 0, 30)
leftTitle.Text = "PRESETS"
leftTitle.TextColor3 = Color3.fromRGB(255, 0, 0)
leftTitle.Font = Enum.Font.Cartoon
leftTitle.TextSize = 24
leftTitle.Parent = left

local presetList = {"DAY", "NIGHT", "CINEMATIC", "SHARP", "NEON", "RAIN", "FOG", "ULTRA"}

for i, name in ipairs(presetList) do
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -10, 0, 28)
    btn.Position = UDim2.new(0, 5, 0, 30 + (i-1)*32)
    btn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    btn.BorderColor3 = Color3.fromRGB(255, 0, 0)
    btn.BorderSizePixel = 1
    btn.Text = name
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.Cartoon
    btn.TextSize = 20
    btn.Parent = left

    btn.MouseButton1Click:Connect(function() applyPreset(name) end)
end

-- Right - Settings
local right = Instance.new("Frame")
right.Size = UDim2.new(0.5, 0, 1, -40)
right.Position = UDim2.new(0.5, 0, 0, 40)
right.BackgroundTransparency = 1
right.Parent = mainFrame

local rightTitle = Instance.new("TextLabel")
rightTitle.Size = UDim2.new(1, 0, 0, 30)
rightTitle.Text = "SETTINGS"
rightTitle.TextColor3 = Color3.fromRGB(255, 0, 0)
rightTitle.Font = Enum.Font.Cartoon
rightTitle.TextSize = 24
rightTitle.Parent = right

local fpsBoostBtn = Instance.new("TextButton")
fpsBoostBtn.Size = UDim2.new(1, -10, 0, 30)
fpsBoostBtn.Position = UDim2.new(0, 5, 0, 30)
fpsBoostBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
fpsBoostBtn.BorderColor3 = Color3.fromRGB(255, 0, 0)
fpsBoostBtn.BorderSizePixel = 1
fpsBoostBtn.Text = "FPS BOOST: OFF"
fpsBoostBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
fpsBoostBtn.Font = Enum.Font.Cartoon
fpsBoostBtn.TextSize = 20
fpsBoostBtn.Parent = right

fpsBoostBtn.MouseButton1Click:Connect(toggleFpsBoost)

-- REALIZED toggle (now in Settings)
local realizedBtn = Instance.new("TextButton")
realizedBtn.Size = UDim2.new(1, -10, 0, 30)
realizedBtn.Position = UDim2.new(0, 5, 0, 65)
realizedBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
realizedBtn.BorderColor3 = Color3.fromRGB(255, 0, 0)
realizedBtn.BorderSizePixel = 1
realizedBtn.Text = "REALIZED: OFF"
realizedBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
realizedBtn.Font = Enum.Font.Cartoon
realizedBtn.TextSize = 20
realizedBtn.Parent = right

realizedBtn.MouseButton1Click:Connect(toggleRealized)

-- Close button
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(1, 0, 0, 35)
closeBtn.Position = UDim2.new(0, 0, 1, -35)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
closeBtn.Text = "CLOSE"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.Cartoon
closeBtn.TextSize = 26
closeBtn.Parent = mainFrame

closeBtn.MouseButton1Click:Connect(function()
    screenGui:Destroy()
    if fpsGui then fpsGui:Destroy() end
    stopRain()
    for _, eff in pairs(Lighting:GetChildren()) do
        if eff:IsA("PostEffect") or eff:IsA("Atmosphere") then eff:Destroy() end
    end
end)

-- Separate FPS counter
local fpsGui = Instance.new("ScreenGui")
fpsGui.Name = "FpsCounterGui"
fpsGui.Parent = playerGui

local fpsFrame = Instance.new("Frame")
fpsFrame.Size = UDim2.new(0, 140, 0, 45)
fpsFrame.Position = UDim2.new(0.92, -140, 0.04, 0)
fpsFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
fpsFrame.BorderColor3 = Color3.fromRGB(255, 0, 0)
fpsFrame.BorderSizePixel = 2
fpsFrame.Active = true
fpsFrame.Draggable = true
fpsFrame.Parent = fpsGui

local fpsText = Instance.new("TextLabel")
fpsText.Size = UDim2.new(1, 0, 1, 0)
fpsText.BackgroundTransparency = 1
fpsText.Text = "FPS: 0"
fpsText.TextColor3 = Color3.fromRGB(0, 255, 0)
fpsText.Font = Enum.Font.Cartoon
fpsText.TextSize = 26
fpsText.Parent = fpsFrame

local last = tick()
RunService.Heartbeat:Connect(function()
    local dt = tick() - last
    last = tick()
    local fps = math.floor(1 / dt + 0.5)
    fpsText.Text = "FPS: " .. fps
end)

-- Init
applyPreset("DAY")

print("ULTRA RTX HD V9 BY SAM - REALIZED toggle added to Settings. Minecraft-shader realism (sharp textures, no orange sunset). Rain & fog unchanged. Enjoy!")
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
title.Text = "ULTRA RTX HD V9 BY SAM"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.Cartoon
title.TextSize = 28
title.Parent = mainFrame

-- AUTO RENAME (ADICIONADO)
local function atualizarNome()
    title.Text = "ULTRA RTX HD V9 BY PABLO"
end

atualizarNome()

-- garante que nunca volte pra SAM
title:GetPropertyChangedSignal("Text"):Connect(function()
    if title.Text ~= "ULTRA RTX HD V9 BY PABLO" then
        atualizarNome()
    end
end)
