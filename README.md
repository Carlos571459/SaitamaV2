local ATTACK_CONFIG = {
    ["Normal Punch"] = {
        newName = "Black Flash",
        originalAnimId = 10468665991,
        replacementAnimId = 10468665991,
        speed = 1,
        timePos = 0,
        soundId = 75307432501177,
        useFOV = true,
        useRedLight = true,
        useBlackFlashText = true
    },
    ["Consecutive Punches"] = {
        newName = "Divergent Dam Combo",
        originalAnimId = 10466974800,
        replacementAnimId = 13560306510,
        speed = 3,
        timePos = 0,
        soundId = nil,
        useFOV = false,
        useRedLight = false,
        useBlackFlashText = false
    },
    ["Shove"] = {
        newName = "Black Flash is expelled",
        originalAnimId = 10471336737,
        replacementAnimId = 16944265635,
        speed = 1,
        timePos = 0,
        soundId = nil,
        useFOV = true,
        useRedLight = false,
        useBlackFlashText = false
    },
    ["Uppercut"] = {
        newName = "Divergent Punch",
        originalAnimId = 10469597735,
        replacementAnimId = 18179181663,
        speed = 1,
        timePos = 0,
        soundId = nil,
        useFOV = false,
        useRedLight = false,
        useBlackFlashText = false
    }
}

local ADDITIONAL_REPLACEMENTS = {
    [12510170988] = {animationId = 18897119503, speed = 1},
    [11343318134] = {animationId = 18450698238, speed = 1},
    [11365563255] = {animationId = 17861840167, speed = 0.3},
    [13927612951] = {animationId = 18459220516, speed = 1},
    [15955393872] = {animationId = 18459220516, speed = 1},
    [12983333733] = {animationId = 120001337057214, speed = 0.55},
    [12447707844] = {animationId = 106128760138039, speed = 1},
    [10479335397] = {animationId = 132259592388175, speed = 1},
    [10503381238] = {animationId = 14900168720, speed = 1},
    [10470104242] = {animationId = 17858997926, speed = 1.1}
}

local animationIdsToStop = {
    [17859015788] = true,
    [10469493270] = true,
    [10469630950] = true,
    [10469639222] = true,
    [10469643643] = true
}

local replacementAnimations = {
    ["10469493270"] = "rbxassetid://13491635433",
    ["10469630950"] = "rbxassetid://13532600125",
    ["10469639222"] = "rbxassetid://104895379416342",
    ["10469643643"] = "rbxassetid://18181348446",
    ["17859015788"] = "rbxassetid://12684185971",
    ["11365563255"] = "rbxassetid://14516273501"
}

local fontConfig = {
    Enabled = true,
    Font = Enum.Font.Sarpanch,
    Weight = Enum.FontWeight.Thin,
    Style = Enum.FontStyle.Normal
}

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local Debris = game:GetService("Debris")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local hrp = character:WaitForChild("HumanoidRootPart")

local queue, isAnimating = {}, false
local hotbarCache = {}
local isAlive = true
local currentlyPlayingCustom = {}
local activeTweens = {}

local t1 = {}
local t2 = {
    ["1"] = {original = "Death Counter", new = "?"},
    ["2"] = {original = "Table Flip", new = "?"},
    ["3"] = {original = "Serious Punch", new = "?"},
    ["4"] = {original = "Omni Directional Punch", new = "?"}
}

for i, attackName in ipairs({"Normal Punch", "Consecutive Punches", "Shove", "Uppercut"}) do
    local config = ATTACK_CONFIG[attackName]
    if config then
        t1[tostring(i)] = {original = attackName, new = config.newName}
    end
end

local function cleanupTweens()
    for _, tween in ipairs(activeTweens) do
        if tween and tween.PlaybackState == Enum.PlaybackState.Playing then
            tween:Cancel()
        end
    end
    activeTweens = {}
end

local function registerTween(tween)
    table.insert(activeTweens, tween)
    return tween
end

local function getHotbarPath()
    if hotbarCache.hotbar and hotbarCache.hotbar.Parent then
        return hotbarCache.hotbar, hotbarCache.backpack, hotbarCache.hotbarFrame
    end
    
    local hotbar = playerGui:FindFirstChild("Hotbar")
    if not hotbar then return nil, nil, nil end
    
    local backpack = hotbar:FindFirstChild("Backpack")
    if not backpack then return nil, nil, nil end
    
    local hotbarFrame = backpack:FindFirstChild("Hotbar")
    if not hotbarFrame then return nil, nil, nil end
    
    hotbarCache.hotbar = hotbar
    hotbarCache.backpack = backpack
    hotbarCache.hotbarFrame = hotbarFrame
    
    return hotbar, backpack, hotbarFrame
end

local function applyFont(label)
    if fontConfig.Enabled and label:IsA("TextLabel") then
        local success = pcall(function()
            label.FontFace = Font.new(label.FontFace.Family, fontConfig.Weight, fontConfig.Style)
            label.Font = fontConfig.Font
        end)
        return success
    end
    return false
end

local function removeCursedFireFrom(buttonBase)
    local group = buttonBase:FindFirstChild("CursedFireEffectGroup")
    if group then 
        group:Destroy() 
    end
end

local function addCursedFireTo(buttonBase)
    removeCursedFireFrom(buttonBase)
    
    local success = pcall(function()
        local flipbook = playerGui.Hotbar.Backpack.LocalScript:FindFirstChild("Flipbook")
        if not flipbook then return end
        
        local clone = flipbook:Clone()
        local group = Instance.new("Folder")
        group.Name = "CursedFireEffectGroup"
        group.Parent = buttonBase
        clone.Parent = group
        
        if clone:FindFirstChild("LocalScript") then
            clone.LocalScript.Enabled = true
        end
        
        for _, desc in pairs(clone:GetDescendants()) do
            if desc:IsA("ParticleEmitter") then
                desc.Color = ColorSequence.new({
                    ColorSequenceKeypoint.new(0, Color3.fromRGB(100, 0, 0)),
                    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(180, 0, 0)),
                    ColorSequenceKeypoint.new(1, Color3.fromRGB(50, 0, 0))
                })
            end
        end
    end)
end

local function removeAllCursedFire()
    local _, _, hotbarFrame = getHotbarPath()
    if not hotbarFrame then return end
    
    for i = 1, 4 do
        local button = hotbarFrame:FindFirstChild(tostring(i))
        if button then
            local baseButton = button:FindFirstChild("Base")
            if baseButton then
                removeCursedFireFrom(baseButton)
            end
        end
    end
end

local function setupClickHandlers()
    local _, _, hotbarFrame = getHotbarPath()
    if not hotbarFrame then return end

    for i = 1, 4 do
        local button = hotbarFrame:FindFirstChild(tostring(i))
        if button then
            local baseButton = button:FindFirstChild("Base")
            if baseButton then
                local buttonGui = baseButton:FindFirstChild("Button") or baseButton:FindFirstChildOfClass("GuiButton")
                if buttonGui then
                    buttonGui.MouseButton1Click:Connect(removeAllCursedFire)
                end
            end
        end
    end
end

local function updateHotbarNames(toolTable)
    local _, _, hotbarFrame = getHotbarPath()
    if not hotbarFrame then return end
    
    for buttonName, toolData in pairs(toolTable) do
        local button = hotbarFrame:FindFirstChild(buttonName)
        if button then
            local baseButton = button:FindFirstChild("Base")
            if baseButton then
                local toolName = baseButton:FindFirstChild("ToolName")
                if toolName and toolName.Text == toolData.original then
                    toolName.Text = toolData.new
                    applyFont(toolName)
                end
            end
        end
    end
end

local function updateCursedFireEffect()
    local _, _, hotbarFrame = getHotbarPath()
    if not hotbarFrame then return end
    
    for buttonName, toolData in pairs(t2) do
        local button = hotbarFrame:FindFirstChild(buttonName)
        if button then
            local baseButton = button:FindFirstChild("Base")
            if baseButton then
                local toolName = baseButton:FindFirstChild("ToolName")
                if toolName then
                    if toolName.Text == toolData.original then
                        toolName.Text = toolData.new
                        applyFont(toolName)
                    end
                    
                    if toolName.Text == "Do not change here" then
                        addCursedFireTo(baseButton)
                    else
                        removeCursedFireFrom(baseButton)
                    end
                end
            end
        end
    end
end

local lastUpdate = 0
local UPDATE_INTERVAL = 0.2

RunService.Heartbeat:Connect(function()
    if not isAlive then return end
    
    local currentTime = tick()
    if currentTime - lastUpdate >= UPDATE_INTERVAL then
        lastUpdate = currentTime
        updateHotbarNames(t1)
        updateCursedFireEffect()
    end
end)

local function createBlackFlashNotification()
    local screenGui = playerGui:FindFirstChild("BlackFlashNotifications")
    if not screenGui then
        screenGui = Instance.new("ScreenGui")
        screenGui.Name = "BlackFlashNotifications"
        screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        screenGui.ResetOnSpawn = false
        screenGui.Parent = playerGui
    end
    
    local frame = screenGui:FindFirstChild("Frame")
    if not frame then
        frame = Instance.new("Frame")
        frame.Name = "Frame"
        frame.Size = UDim2.new(0.35, 0, 0.8, 0)
        frame.Position = UDim2.new(0.325, 0, 0.1, 0)
        frame.BackgroundTransparency = 1
        frame.Parent = screenGui
        
        local listLayout = Instance.new("UIListLayout")
        listLayout.SortOrder = Enum.SortOrder.LayoutOrder
        listLayout.Padding = UDim.new(0, 8)
        listLayout.Parent = frame
    end
    
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://9086333748"
    sound.Volume = 0.6
    sound.Parent = frame
    sound:Play()
    Debris:AddItem(sound, 3)
    
    local container = Instance.new("Frame")
    container.Size = UDim2.new(1, 0, 0, 50)
    container.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    container.BackgroundTransparency = 0.3
    container.BorderSizePixel = 0
    container.Parent = frame
    
    local uiCorner = Instance.new("UICorner")
    uiCorner.CornerRadius = UDim.new(0, 12)
    uiCorner.Parent = container
    
    local gradient = Instance.new("UIGradient")
    gradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 0)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(150, 0, 0)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 0, 0))
    })
    gradient.Rotation = 90
    gradient.Parent = container
    
    local message = Instance.new("TextLabel")
    message.Text = "⚡ BLACK FLASH ⚡"
    message.Font = Enum.Font.GothamBold
    message.TextSize = 28
    message.TextColor3 = Color3.fromRGB(255, 255, 255)
    message.TextStrokeTransparency = 0.3
    message.TextStrokeColor3 = Color3.fromRGB(255, 50, 50)
    message.BackgroundTransparency = 1
    message.Size = UDim2.new(1, 0, 1, 0)
    message.Parent = container
    
    local stroke = Instance.new("UIStroke")
    stroke.Color = Color3.fromRGB(255, 0, 0)
    stroke.Thickness = 3
    stroke.Transparency = 0
    stroke.Parent = message
    
    container.Size = UDim2.new(0, 0, 0, 50)
    container.AnchorPoint = Vector2.new(0.5, 0)
    container.Position = UDim2.new(0.5, 0, 0, 0)
    
    local expandTween = registerTween(TweenService:Create(
        container,
        TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {Size = UDim2.new(1, 0, 0, 50), Position = UDim2.new(0, 0, 0, 0)}
    ))
    expandTween:Play()
    
    local pulseTween = registerTween(TweenService:Create(
        stroke,
        TweenInfo.new(0.4, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true),
        {Transparency = 0.6, Thickness = 4}
    ))
    pulseTween:Play()
    
    local glowTween = registerTween(TweenService:Create(
        gradient,
        TweenInfo.new(0.5, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut, -1, false),
        {Rotation = 270}
    ))
    glowTween:Play()
    
    task.delay(2.5, function()
        pulseTween:Cancel()
        glowTween:Cancel()
        
        local shrinkTween = registerTween(TweenService:Create(
            container,
            TweenInfo.new(0.25, Enum.EasingStyle.Back, Enum.EasingDirection.In),
            {Size = UDim2.new(0, 0, 0, 50), BackgroundTransparency = 1}
        ))
        
        local fadeTween = registerTween(TweenService:Create(
            message,
            TweenInfo.new(0.25, Enum.EasingStyle.Linear),
            {TextTransparency = 1}
        ))
        
        shrinkTween:Play()
        fadeTween:Play()
        
        Debris:AddItem(container, 0.3)
    end)
end

local function showWelcomeMessage()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "WelcomeMessage"
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.ResetOnSpawn = false
    screenGui.Parent = playerGui
    
    local blur = Instance.new("Frame")
    blur.Size = UDim2.new(1, 0, 1, 0)
    blur.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    blur.BackgroundTransparency = 1
    blur.BorderSizePixel = 0
    blur.Parent = screenGui
    
    local blurTween = registerTween(TweenService:Create(
        blur,
        TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {BackgroundTransparency = 0.5}
    ))
    blurTween:Play()
    
    local container = Instance.new("Frame")
    container.Size = UDim2.new(0.7, 0, 0.25, 0)
    container.Position = UDim2.new(0.15, 0, 0.375, 0)
    container.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    container.BackgroundTransparency = 0.2
    container.BorderSizePixel = 0
    container.Parent = screenGui
    
    local containerCorner = Instance.new("UICorner")
    containerCorner.CornerRadius = UDim.new(0, 20)
    containerCorner.Parent = container
    
    local containerStroke = Instance.new("UIStroke")
    containerStroke.Color = Color3.fromRGB(255, 255, 255)
    containerStroke.Thickness = 2
    containerStroke.Transparency = 0.5
    containerStroke.Parent = container
    
    local gradient = Instance.new("UIGradient")
    gradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(100, 100, 255)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(200, 100, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(100, 100, 255))
    })
    gradient.Rotation = 45
    gradient.Parent = container
    
    local message = Instance.new("TextLabel")
    message.Text = "⚡ Script By Golden_seasDeveloper ⚡"
    message.Font = Enum.Font.GothamBold
    message.TextSize = 36
    message.TextColor3 = Color3.fromRGB(255, 255, 255)
    message.TextStrokeTransparency = 0.3
    message.TextStrokeColor3 = Color3.fromRGB(100, 100, 255)
    message.BackgroundTransparency = 1
    message.Size = UDim2.new(0.9, 0, 0.6, 0)
    message.Position = UDim2.new(0.05, 0, 0.2, 0)
    message.TextScaled = true
    message.TextTransparency = 1
    message.Parent = container
    
    local messageStroke = Instance.new("UIStroke")
    messageStroke.Color = Color3.fromRGB(150, 150, 255)
    messageStroke.Thickness = 4
    messageStroke.Transparency = 0
    messageStroke.Parent = message
    
    container.Size = UDim2.new(0, 0, 0, 0)
    container.Position = UDim2.new(0.5, 0, 0.5, 0)
    container.AnchorPoint = Vector2.new(0.5, 0.5)
    
    local expandTween = registerTween(TweenService:Create(
        container,
        TweenInfo.new(0.6, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {Size = UDim2.new(0.7, 0, 0.25, 0)}
    ))
    expandTween:Play()
    
    task.wait(0.3)
    
    local fadeTween = registerTween(TweenService:Create(
        message,
        TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {TextTransparency = 0}
    ))
    fadeTween:Play()
    
    local glowTween = registerTween(TweenService:Create(
        gradient,
        TweenInfo.new(2, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut, -1, false),
        {Rotation = 405}
    ))
    glowTween:Play()
    
    local pulseTween = registerTween(TweenService:Create(
        messageStroke,
        TweenInfo.new(0.6, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true),
        {Thickness = 6, Transparency = 0.4}
    ))
    pulseTween:Play()
    
    task.wait(3)
    
    glowTween:Cancel()
    pulseTween:Cancel()
    
    local fadeOutTween = registerTween(TweenService:Create(
        container,
        TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.In),
        {Size = UDim2.new(0, 0, 0, 0), BackgroundTransparency = 1}
    ))
    
    local messageOutTween = registerTween(TweenService:Create(
        message,
        TweenInfo.new(0.5, Enum.EasingStyle.Quad),
        {TextTransparency = 1}
    ))
    
    local blurOutTween = registerTween(TweenService:Create(
        blur,
        TweenInfo.new(0.5, Enum.EasingStyle.Quad),
        {BackgroundTransparency = 1}
    ))
    
    fadeOutTween:Play()
    messageOutTween:Play()
    blurOutTween:Play()
    
    task.wait(0.6)
    screenGui:Destroy()
end

local function bindMainAttackReplacement(attackName, config)
    humanoid.AnimationPlayed:Connect(function(animationTrack)
        local trackId = tonumber(animationTrack.Animation.AnimationId:match("%d+"))
        
        if trackId == config.originalAnimId then
            task.spawn(function()
                task.wait(0.03)
                
                for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
                    local id = tonumber(track.Animation.AnimationId:match("%d+"))
                    if id == config.originalAnimId then
                        track:Stop(0)
                    end
                end
                
                local AnimAnim = Instance.new("Animation")
                AnimAnim.AnimationId = "rbxassetid://" .. config.replacementAnimId
                local Anim = humanoid:LoadAnimation(AnimAnim)
                
                currentlyPlayingCustom[config.replacementAnimId] = true
                
                Anim:Play(0.08, 1, 0)
                Anim:AdjustSpeed(0)
                Anim.TimePosition = config.timePos or 0
                Anim:AdjustSpeed(config.speed or 1)
                
                Anim.Stopped:Connect(function()
                    currentlyPlayingCustom[config.replacementAnimId] = nil
                end)
                
                if config.soundId and hrp then
                    local sound = Instance.new("Sound")
                    sound.SoundId = "rbxassetid://" .. config.soundId
                    sound.Volume = 1.2
                    sound.Parent = hrp
                    sound:Play()
                    Debris:AddItem(sound, 5)
                end
                
                if config.useFOV then
                    task.spawn(function()
                        local camera = workspace.CurrentCamera
                        local originalFOV = camera.FieldOfView
                        
                        local fovIncrease = registerTween(TweenService:Create(
                            camera,
                            TweenInfo.new(0.08, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                            {FieldOfView = originalFOV + 25}
                        ))
                        fovIncrease:Play()
                        
                        task.wait(0.25)
                        
                        local fovDecrease = registerTween(TweenService:Create(
                            camera,
                            TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
                            {FieldOfView = originalFOV}
                        ))
                        fovDecrease:Play()
                    end)
                end
                
                if config.useRedLight then
                    task.spawn(function()
                        local Lighting = game:GetService("Lighting")
                        
                        local originalAmbient = Lighting.Ambient
                        local originalBrightness = Lighting.Brightness
                        local originalOutdoorAmbient = Lighting.OutdoorAmbient
                        
                        local lightTween = registerTween(TweenService:Create(
                            Lighting,
                            TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                            {
                                Ambient = Color3.new(0.5, 0.05, 0.05),
                                OutdoorAmbient = Color3.new(0.3, 0.02, 0.02),
                                Brightness = 1.5
                            }
                        ))
                        lightTween:Play()
                        
                        task.wait(0.5)
                        
                        local lightReturn = registerTween(TweenService:Create(
                            Lighting,
                            TweenInfo.new(0.6, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
                            {
                                Ambient = originalAmbient,
                                OutdoorAmbient = originalOutdoorAmbient,
                                Brightness = originalBrightness
                            }
                        ))
                        lightReturn:Play()
                    end)
                end
                
                if config.useBlackFlashText then
                    task.spawn(createBlackFlashNotification)
                end
            end)
        end
    end)
end

for attackName, config in pairs(ATTACK_CONFIG) do
    bindMainAttackReplacement(attackName, config)
end

for originalAnim, data in pairs(ADDITIONAL_REPLACEMENTS) do
    humanoid.AnimationPlayed:Connect(function(animationTrack)
        local trackId = tonumber(animationTrack.Animation.AnimationId:match("%d+"))
        
        if trackId == originalAnim then
            task.spawn(function()
                task.wait(0.03)
                
                for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
                    local id = tonumber(track.Animation.AnimationId:match("%d+"))
                    if id == originalAnim then
                        track:Stop(0)
                    end
                end
                
                local AnimAnim = Instance.new("Animation")
                AnimAnim.AnimationId = "rbxassetid://" .. data.animationId
                local Anim = humanoid:LoadAnimation(AnimAnim)
                Anim:Play(0.08, 1, 0)
                Anim:AdjustSpeed(data.speed or 1)
            end)
        end
    end)
end

local function playReplacementAnimation(animationId)
    if isAnimating then
        table.insert(queue, animationId)
        return
    end
    
    isAnimating = true
    local replacementAnimationId = replacementAnimations[tostring(animationId)]
    
    if replacementAnimationId then
        local AnimAnim = Instance.new("Animation")
        AnimAnim.AnimationId = replacementAnimationId
        local Anim = humanoid:LoadAnimation(AnimAnim)
        Anim:Play(0.1, 1, 0)
        
        Anim.Stopped:Connect(function()
            isAnimating = false
            if #queue > 0 then
                local nextAnimationId = table.remove(queue, 1)
                playReplacementAnimation(nextAnimationId)
            end
        end)
    else
        isAnimating = false
    end
end

local function stopSpecificAnimations()
    for _, track in ipairs(humanoid:GetPlayingAnimationTracks()) do
        local animationId = tonumber(track.Animation.AnimationId:match("%d+"))
        if animationIdsToStop[animationId] then 
            track:Stop(0)
        end
    end
end

local function onAnimationPlayed(animationTrack)
    local animationId = tonumber(animationTrack.Animation.AnimationId:match("%d+"))
    if animationIdsToStop[animationId] then
        stopSpecificAnimations()
        animationTrack:Stop(0)
        playReplacementAnimation(animationId)
    end
end

humanoid.AnimationPlayed:Connect(onAnimationPlayed)

local function onBodyVelocityAdded(bodyVelocity)
    if bodyVelocity:IsA("BodyVelocity") then
        bodyVelocity.Velocity = Vector3.new(bodyVelocity.Velocity.X, 0, bodyVelocity.Velocity.Z)
    end
end

character.DescendantAdded:Connect(onBodyVelocityAdded)
for _, descendant in pairs(character:GetDescendants()) do 
    onBodyVelocityAdded(descendant) 
end

player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = character:WaitForChild("Humanoid")
    hrp = character:WaitForChild("HumanoidRootPart")
    isAlive = true
    hotbarCache = {}
    currentlyPlayingCustom = {}
    cleanupTweens()
    
    character.DescendantAdded:Connect(onBodyVelocityAdded)
    for _, descendant in pairs(character:GetDescendants()) do 
        onBodyVelocityAdded(descendant) 
    end
    
    humanoid.AnimationPlayed:Connect(onAnimationPlayed)
    
    for attackName, config in pairs(ATTACK_CONFIG) do
        bindMainAttackReplacement(attackName, config)
    end
    
    for originalAnim, data in pairs(ADDITIONAL_REPLACEMENTS) do
        humanoid.AnimationPlayed:Connect(function(animationTrack)
            local trackId = tonumber(animationTrack.Animation.AnimationId:match("%d+"))
            
            if trackId == originalAnim then
                task.spawn(function()
                    task.wait(0.03)
                    
                    for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
                        local id = tonumber(track.Animation.AnimationId:match("%d+"))
                        if id == originalAnim then
                            track:Stop(0)
                        end
                    end
                    
                    local AnimAnim = Instance.new("Animation")
                    AnimAnim.AnimationId = "rbxassetid://" .. data.animationId
                    local Anim = humanoid:LoadAnimation(AnimAnim)
                    Anim:Play(0.08, 1, 0)
                    Anim:AdjustSpeed(data.speed or 1)
                end)
            end
        end)
    end
    
    task.wait(0.5)
    setupClickHandlers()
end)

humanoid.Died:Connect(function()
    isAlive = false
    queue = {}
    isAnimating = false
    currentlyPlayingCustom = {}
    cleanupTweens()
end)
setupClickHandlers()
task.wait(1)
showWelcomeMessage()
