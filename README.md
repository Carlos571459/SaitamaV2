local ATTACK_CONFIG = {
    ["Normal Punch"] = {
        newName = "Black Flash",
        animationId = 17186602996,
        speed = 1,
        timePos = 0,
        soundId = 75307432501177,
        useFOV = true,
        useRedLight = true,
        useBlackFlashText = true
    },
    ["Consecutive Punches"] = {
        newName = "Divergent Dam Combo",
        animationId = 13560306510,
        speed = 3,
        timePos = 0,
        soundId = nil,
        useFOV = false,
        useRedLight = false,
        useBlackFlashText = false
    },
    ["Shove"] = {
        newName = "Black Flash is expelled",
        animationId = 18179181663,
        speed = 1,
        timePos = 0,
        soundId = nil,
        useFOV = true,
        useRedLight = false,
        useBlackFlashText = false
    },
    ["Uppercut"] = {
        newName = "Divergent Punch",
        animationId = nil,
        speed = 1,
        timePos = 0,
        soundId = nil,
        useFOV = false,
        useRedLight = false,
        useBlackFlashText = false
    }
}

local ANIMATION_REPLACEMENTS = {
    [10468665991] = {animationId = 17186602996, speed = 1, soundId = 75307432501177, useFOV = true, useRedLight = true, useBlackFlashText = true, priority = Enum.AnimationPriority.Action4},
    [10466974800] = {animationId = 13560306510, speed = 3, priority = Enum.AnimationPriority.Action3},
    [10471336737] = {animationId = 18179181663, speed = 1, useFOV = true, priority = Enum.AnimationPriority.Action4},
    [12510170988] = {animationId = 18897119503, speed = 1, priority = Enum.AnimationPriority.Action2},
    [11343318134] = {animationId = 18450698238, speed = 1, priority = Enum.AnimationPriority.Action2},
    [11365563255] = {animationId = 17861840167, speed = 0.3, priority = Enum.AnimationPriority.Action3},
    [13927612951] = {animationId = 18459220516, speed = 1, priority = Enum.AnimationPriority.Action2},
    [15955393872] = {animationId = 18459220516, speed = 1, priority = Enum.AnimationPriority.Action2},
    [12983333733] = {animationId = 120001337057214, speed = 0.55, priority = Enum.AnimationPriority.Action3},
    [12447707844] = {animationId = 106128760138039, speed = 1, priority = Enum.AnimationPriority.Action2},
    [10479335397] = {animationId = 132259592388175, speed = 1, priority = Enum.AnimationPriority.Action2},
    [10503381238] = {animationId = 14900168720, speed = 1, priority = Enum.AnimationPriority.Action2},
    [10470104242] = {animationId = 17858997926, speed = 1.1, priority = Enum.AnimationPriority.Action2}
}

local REPLACEMENT_ANIMATIONS = {
    [10469493270] = {id = "rbxassetid://13491635433", priority = Enum.AnimationPriority.Action3, fadeTime = 0.1},
    [10469630950] = {id = "rbxassetid://13532600125", priority = Enum.AnimationPriority.Action3, fadeTime = 0.1},
    [10469639222] = {id = "rbxassetid://104895379416342", priority = Enum.AnimationPriority.Action4, fadeTime = 0.15},
    [10469643643] = {id = "rbxassetid://18181348446", priority = Enum.AnimationPriority.Action3, fadeTime = 0.1},
    [17859015788] = {id = "rbxassetid://12684185971", priority = Enum.AnimationPriority.Action2, fadeTime = 0.1},
    [11365563255] = {id = "rbxassetid://14516273501", priority = Enum.AnimationPriority.Action3, fadeTime = 0.1}
}

local fontConfig = {
    Enabled = true,
    Font = Enum.Font.Sarpanch,
    Weight = Enum.FontWeight.Thin,
    Style = Enum.FontStyle.Normal
}

local player = game.Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local animationCache = {}
local activeAnimations = {}
local animationQueue = {}
local isProcessingQueue = false
local currentPlayingTracks = {}

local t1 = {
    ["1"] = {original = og1, new = mn1},
    ["2"] = {original = og2, new = mn2},
    ["3"] = {original = og3, new = mn3},
    ["4"] = {original = og4, new = mn4}
}

local t2 = {
    ["1"] = {original = ultOG1, new = ult1},
    ["2"] = {original = ultOG2, new = ult2},
    ["3"] = {original = ultOG3, new = ult3},
    ["4"] = {original = ultOG4, new = ult4}
}

local function preloadAnimation(animationId)
    if not animationCache[animationId] then
        local anim = Instance.new("Animation")
        anim.AnimationId = "rbxassetid://" .. animationId
        animationCache[animationId] = humanoid:LoadAnimation(anim)
    end
    return animationCache[animationId]
end

local function preloadAllAnimations()
    for _, config in pairs(ANIMATION_REPLACEMENTS) do
        if config.animationId then
            preloadAnimation(config.animationId)
        end
    end
    for _, config in pairs(REPLACEMENT_ANIMATIONS) do
        local animId = config.id:match("%d+")
        if animId then
            preloadAnimation(tonumber(animId))
        end
    end
end

local function stopAllAnimationsSmooth(fadeTime)
    fadeTime = fadeTime or 0.1
    for track, _ in pairs(currentPlayingTracks) do
        if track.IsPlaying then
            track:Stop(fadeTime)
        end
        currentPlayingTracks[track] = nil
    end
end

local function stopSpecificAnimation(animationId, fadeTime)
    fadeTime = fadeTime or 0.1
    for track, id in pairs(currentPlayingTracks) do
        if id == animationId and track.IsPlaying then
            track:Stop(fadeTime)
            currentPlayingTracks[track] = nil
        end
    end
end

local function playAnimationAdvanced(animationId, config)
    local track = preloadAnimation(animationId)
    if not track then return nil end
    
    if config.priority then
        track.Priority = config.priority
    end
    
    local fadeTime = config.fadeTime or 0.1
    track:Play(fadeTime)
    
    if config.speed then
        track:AdjustSpeed(0)
        track.TimePosition = config.timePos or 0
        track:AdjustSpeed(config.speed)
    end
    
    currentPlayingTracks[track] = animationId
    
    track.Stopped:Connect(function()
        currentPlayingTracks[track] = nil
    end)
    
    return track
end

local function createSmoothFOVEffect(duration, intensity)
    local camera = workspace.CurrentCamera
    local originalFOV = camera.FieldOfView
    
    local fovIn = TweenService:Create(
        camera,
        TweenInfo.new(duration * 0.3, Enum.EasingStyle.Cubic, Enum.EasingDirection.Out),
        {FieldOfView = originalFOV + intensity}
    )
    
    local fovOut = TweenService:Create(
        camera,
        TweenInfo.new(duration * 0.7, Enum.EasingStyle.Cubic, Enum.EasingDirection.In),
        {FieldOfView = originalFOV}
    )
    
    fovIn:Play()
    fovIn.Completed:Connect(function()
        fovOut:Play()
    end)
end

local function createEnhancedRedLight(duration)
    local Lighting = game:GetService("Lighting")
    
    local originalAmbient = Lighting.Ambient
    local originalBrightness = Lighting.Brightness
    local originalColorShift_Top = Lighting.ColorShift_Top
    
    local lightIn = TweenService:Create(
        Lighting,
        TweenInfo.new(duration * 0.4, Enum.EasingStyle.Cubic, Enum.EasingDirection.Out),
        {
            Ambient = Color3.new(0.5, 0.05, 0.05),
            Brightness = 1.2,
            ColorShift_Top = Color3.new(0.3, 0, 0)
        }
    )
    
    local lightOut = TweenService:Create(
        Lighting,
        TweenInfo.new(duration * 0.6, Enum.EasingStyle.Cubic, Enum.EasingDirection.In),
        {
            Ambient = originalAmbient,
            Brightness = originalBrightness,
            ColorShift_Top = originalColorShift_Top
        }
    )
    
    lightIn:Play()
    lightIn.Completed:Connect(function()
        lightOut:Play()
    end)
end

local function playSoundWithFade(soundId, volume, fadeInTime)
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://" .. soundId
    sound.Volume = 0
    sound.Parent = hrp
    sound:Play()
    
    local fadeTween = TweenService:Create(
        sound,
        TweenInfo.new(fadeInTime or 0.1, Enum.EasingStyle.Linear),
        {Volume = volume or 1}
    )
    fadeTween:Play()
    
    game.Debris:AddItem(sound, 5)
    return sound
end

local function applyFont(label)
    if fontConfig.Enabled and label:IsA("TextLabel") then
        label.FontFace = Font.new(label.FontFace.Family, fontConfig.Weight, fontConfig.Style)
        label.Font = fontConfig.Font
    end
end

local function removeCursedFireFrom(buttonBase)
    local group = buttonBase:FindFirstChild("CursedFireEffectGroup")
    if group then group:Destroy() end
end

local function addCursedFireTo(buttonBase)
    removeCursedFireFrom(buttonBase)
    local kj = playerGui.Hotbar.Backpack.LocalScript:FindFirstChild("Flipbook")
    if kj then
        local clone = kj:Clone()
        local group = Instance.new("Folder")
        group.Name = "CursedFireEffectGroup"
        group.Parent = buttonBase
        clone.Parent = group
        clone.LocalScript.Enabled = true
        for _, desc in pairs(clone:GetDescendants()) do
            if desc:IsA("ParticleEmitter") then
                desc.Color = ColorSequence.new({
                    ColorSequenceKeypoint.new(0, Color3.fromRGB(80, 0, 0)),
                    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(140, 0, 0)),
                    ColorSequenceKeypoint.new(1, Color3.fromRGB(30, 0, 0))
                })
            end
        end
    end
end

local function removeAllCursedFire()
    local hotbar = playerGui:FindFirstChild("Hotbar")
    if not hotbar then return end
    local backpack = hotbar:FindFirstChild("Backpack")
    if not backpack then return end
    local hotbarFrame = backpack:FindFirstChild("Hotbar")
    if not hotbarFrame then return end
    for i = 1, 4 do
        local baseButton = hotbarFrame:FindFirstChild(tostring(i))
        baseButton = baseButton and baseButton:FindFirstChild("Base")
        if baseButton then removeCursedFireFrom(baseButton) end
    end
end

local function setupClickHandlers()
    local hotbar = playerGui:FindFirstChild("Hotbar")
    if not hotbar then return end
    local backpack = hotbar:FindFirstChild("Backpack")
    if not backpack then return end
    local hotbarFrame = backpack:FindFirstChild("Hotbar")
    if not hotbarFrame then return end

    for i = 1, 4 do
        local button = hotbarFrame:FindFirstChild(tostring(i))
        local baseButton = button and button:FindFirstChild("Base")
        if baseButton then
            local buttonGui = baseButton:FindFirstChild("Button") or baseButton:FindFirstChildOfClass("GuiButton")
            if buttonGui then
                buttonGui.MouseButton1Click:Connect(function()
                    removeAllCursedFire()
                end)
            end
        end
    end
end

local function N1()
    while character.Humanoid.Health > 0 do
        local hotbar = playerGui:FindFirstChild("Hotbar")
        if hotbar then
            local backpack = hotbar:FindFirstChild("Backpack")
            if backpack then
                local hotbarFrame = backpack:FindFirstChild("Hotbar")
                if hotbarFrame then
                    for buttonName, toolData in pairs(t1) do
                        local baseButton = hotbarFrame:FindFirstChild(buttonName)
                        baseButton = baseButton and baseButton:FindFirstChild("Base")
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
        end
        task.wait(0.1)
    end
end

local function N2()
    while character.Humanoid.Health > 0 do
        local hotbar = playerGui:FindFirstChild("Hotbar")
        if hotbar then
            local backpack = hotbar:FindFirstChild("Backpack")
            if backpack then
                local hotbarFrame = backpack:FindFirstChild("Hotbar")
                if hotbarFrame then
                    for buttonName, toolData in pairs(t2) do
                        local baseButton = hotbarFrame:FindFirstChild(buttonName)
                        baseButton = baseButton and baseButton:FindFirstChild("Base")
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
        end
        task.wait(0.1)
    end
end

local function createBlackFlashNotification()
    local screenGui = playerGui:FindFirstChild("BlackFlashNotifications")
    if not screenGui then
        screenGui = Instance.new("ScreenGui")
        screenGui.Name = "BlackFlashNotifications"
        screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        screenGui.Parent = playerGui
    end
    
    local frame = screenGui:FindFirstChild("Frame")
    if not frame then
        frame = Instance.new("Frame")
        frame.Name = "Frame"
        frame.Size = UDim2.new(0.3, 0, 0.8, 0)
        frame.Position = UDim2.new(0.35, 0, 0.1, 0)
        frame.BackgroundTransparency = 1
        frame.Parent = screenGui
        
        local listLayout = Instance.new("UIListLayout")
        listLayout.SortOrder = Enum.SortOrder.LayoutOrder
        listLayout.Padding = UDim.new(0, 5)
        listLayout.Parent = frame
    end
    
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://9086333748"
    sound.Volume = 0.5
    sound.Parent = frame
    sound:Play()
    game.Debris:AddItem(sound, 2)
    
    local message = Instance.new("TextLabel")
    message.Text = "BLACK FLASH"
    message.Font = Enum.Font.GothamBold
    message.TextSize = 24
    message.TextColor3 = Color3.fromRGB(255, 255, 255)
    message.TextStrokeTransparency = 0.5
    message.TextStrokeColor3 = Color3.fromRGB(255, 0, 0)
    message.BackgroundTransparency = 1
    message.BorderSizePixel = 0
    message.Size = UDim2.new(1, 0, 0, 40)
    message.TextScaled = false
    message.Parent = frame
    
    local stroke = Instance.new("UIStroke")
    stroke.Color = Color3.fromRGB(255, 0, 0)
    stroke.Thickness = 2
    stroke.Transparency = 0
    stroke.Parent = message
    
    local origSize = message.TextSize
    message.TextSize = 0
    
    local tweenIn = TweenService:Create(
        message,
        TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {TextSize = origSize}
    )
    tweenIn:Play()
    
    local pulseTween = TweenService:Create(
        stroke,
        TweenInfo.new(0.3, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true),
        {Transparency = 0.5}
    )
    pulseTween:Play()
    
    task.wait(2)
    
    pulseTween:Cancel()
    
    local tweenOut = TweenService:Create(
        message,
        TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.In),
        {TextSize = 0}
    )
    tweenOut:Play()
    
    game:GetService("Debris"):AddItem(message, 0.3)
end

local function showWelcomeMessage()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "WelcomeMessage"
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Parent = playerGui
    
    local frame = Instance.new("Frame")
    frame.Name = "Frame"
    frame.Size = UDim2.new(0.6, 0, 0.2, 0)
    frame.Position = UDim2.new(0.2, 0, 0.4, 0)
    frame.BackgroundTransparency = 1
    frame.Parent = screenGui
    
    local message = Instance.new("TextLabel")
    message.Text = "Script By Golden_seasDeveloper"
    message.Font = Enum.Font.GothamBold
    message.TextSize = 32
    message.TextColor3 = Color3.fromRGB(0, 0, 0)
    message.TextStrokeTransparency = 0
    message.TextStrokeColor3 = Color3.fromRGB(255, 255, 255)
    message.BackgroundTransparency = 1
    message.BorderSizePixel = 0
    message.Size = UDim2.new(0, 0, 0, 0)
    message.Position = UDim2.new(0.5, 0, 0.5, 0)
    message.AnchorPoint = Vector2.new(0.5, 0.5)
    message.TextScaled = true
    message.TextTransparency = 1
    message.Rotation = -15
    message.Parent = frame
    
    local uiStroke = Instance.new("UIStroke")
    uiStroke.Color = Color3.fromRGB(255, 255, 255)
    uiStroke.Thickness = 4
    uiStroke.Parent = message
    
    local particles = {}
    for i = 1, 30 do
        local particle = Instance.new("Frame")
        particle.Size = UDim2.new(0, math.random(5, 15), 0, math.random(5, 15))
        particle.Position = UDim2.new(0.5, 0, 0.5, 0)
        particle.AnchorPoint = Vector2.new(0.5, 0.5)
        particle.BackgroundColor3 = Color3.fromRGB(math.random(200, 255), math.random(200, 255), math.random(200, 255))
        particle.BorderSizePixel = 0
        particle.BackgroundTransparency = 1
        particle.Parent = screenGui
        
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(1, 0)
        corner.Parent = particle
        
        table.insert(particles, particle)
    end
    
    for _, particle in ipairs(particles) do
        local angle = math.rad(math.random(0, 360))
        local distance = math.random(200, 400)
        local endX = 0.5 + (math.cos(angle) * distance / screenGui.AbsoluteSize.X)
        local endY = 0.5 + (math.sin(angle) * distance / screenGui.AbsoluteSize.Y)
        
        local particleTween = TweenService:Create(
            particle,
            TweenInfo.new(0.8, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {
                Position = UDim2.new(endX, 0, endY, 0),
                BackgroundTransparency = 0,
                Rotation = math.random(-180, 180)
            }
        )
        particleTween:Play()
        
        task.delay(0.3, function()
            local fadeOut = TweenService:Create(
                particle,
                TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
                {BackgroundTransparency = 1}
            )
            fadeOut:Play()
        end)
    end
    
    local expandTween = TweenService:Create(
        message,
        TweenInfo.new(0.6, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {
            Size = UDim2.new(1, 0, 1, 0),
            TextTransparency = 0,
            Rotation = 0
        }
    )
    expandTween:Play()
    
    task.wait(0.3)
    
    local shakeDuration = 0.4
    local shakeIntensity = 5
    local shakeCount = 10
    for i = 1, shakeCount do
        local offsetX = math.random(-shakeIntensity, shakeIntensity)
        local offsetY = math.random(-shakeIntensity, shakeIntensity)
        message.Position = UDim2.new(0.5, offsetX, 0.5, offsetY)
        task.wait(shakeDuration / shakeCount)
    end
    message.Position = UDim2.new(0.5, 0, 0.5, 0)
    
    local pulseTween = TweenService:Create(
        message,
        TweenInfo.new(0.5, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, 4, true),
        {TextSize = 36}
    )
    pulseTween:Play()
    
    task.wait(2.5)
    
    pulseTween:Cancel()
    
    local rotateTween = TweenService:Create(
        message,
        TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.In),
        {
            Rotation = 15,
            Size = UDim2.new(0, 0, 0, 0),
            TextTransparency = 1
        }
    )
    rotateTween:Play()
    
    task.wait(0.4)
    screenGui:Destroy()
end

local function handleAnimationReplacement(animationId, config)
    task.spawn(function()
        stopSpecificAnimation(animationId, 0.05)
        
        local track = playAnimationAdvanced(config.animationId, config)
        if not track then return end
        
        if config.soundId then
            playSoundWithFade(config.soundId, 1, 0.05)
        end
        
        if config.useFOV then
            createSmoothFOVEffect(0.4, 20)
        end
        
        if config.useRedLight then
            createEnhancedRedLight(1)
        end
        
        if config.useBlackFlashText then
            task.spawn(createBlackFlashNotification)
        end
    end)
end

local function processAnimationQueue()
    if isProcessingQueue then return end
    isProcessingQueue = true
    
    while #animationQueue > 0 do
        local animData = table.remove(animationQueue, 1)
        local config = REPLACEMENT_ANIMATIONS[animData.id]
        
        if config then
            stopAllAnimationsSmooth(config.fadeTime or 0.1)
            
            local animId = config.id:match("%d+")
            if animId then
                local track = preloadAnimation(tonumber(animId))
                if track then
                    if config.priority then
                        track.Priority = config.priority
                    end
                    track:Play(config.fadeTime or 0.1)
                    currentPlayingTracks[track] = animData.id
                    
                    track.Stopped:Wait()
                end
            end
        end
        
        task.wait(0.05)
    end
    
    isProcessingQueue = false
end

humanoid.AnimationPlayed:Connect(function(animationTrack)
    local animationId = tonumber(animationTrack.Animation.AnimationId:match("%d+"))
    if not animationId then return end
    
    if ANIMATION_REPLACEMENTS[animationId] then
        animationTrack:Stop(0.05)
        handleAnimationReplacement(animationId, ANIMATION_REPLACEMENTS[animationId])
    elseif REPLACEMENT_ANIMATIONS[animationId] then
        animationTrack:Stop(0.05)
        table.insert(animationQueue, {id = animationId, track = animationTrack})
        task.spawn(processAnimationQueue)
    end
end)

local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

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
    humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    
    animationCache = {}
    activeAnimations = {}
    animationQueue = {}
    currentPlayingTracks = {}
    
    preloadAllAnimations()
    
    character.DescendantAdded:Connect(onBodyVelocityAdded)
    for _, descendant in pairs(character:GetDescendants()) do 
        onBodyVelocityAdded(descendant) 
    end
    
    humanoid.AnimationPlayed:Connect(function(animationTrack)
        local animationId = tonumber(animationTrack.Animation.AnimationId:match("%d+"))
        if not animationId then return end
        
        if ANIMATION_REPLACEMENTS[animationId] then
            animationTrack:Stop(0.05)
            handleAnimationReplacement(animationId, ANIMATION_REPLACEMENTS[animationId])
        elseif REPLACEMENT_ANIMATIONS[animationId] then
            animationTrack:Stop(0.05)
            table.insert(animationQueue, {id = animationId, track = animationTrack})
            task.spawn(processAnimationQueue)
        end
    end)
end)

preloadAllAnimations()
setupClickHandlers()
task.spawn(N1)
task.spawn(N2)

task.wait(1)
showWelcomeMessage()
