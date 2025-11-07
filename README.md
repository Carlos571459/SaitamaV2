local ATTACK_CONFIG = {
    ["Normal Punch"] = {
        newName = "Black Flash",
        animationId = 10468665991,
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
        animationId = 16944265635,
        speed = 1,
        timePos = 0,
        soundId = nil,
        useFOV = true,
        useRedLight = false,
        useBlackFlashText = false
    },
    ["Uppercut"] = {
        newName = "Divergent Punch",
        animationId = 18179181663,
        speed = 1,
        timePos = 0,
        soundId = nil,
        useFOV = false,
        useRedLight = false,
        useBlackFlashText = false
    }
}

local ANIMATION_REPLACEMENTS = {
    [10468665991] = "Normal Punch",
    [10466974800] = "Consecutive Punches",
    [10471336737] = "Shove",
    [10469597735] = "Uppercut",
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
        pcall(function()
            label.FontFace = Font.new(label.FontFace.Family, fontConfig.Weight, fontConfig.Style)
            label.Font = fontConfig.Font
        end)
    end
end

local function removeCursedFireFrom(buttonBase)
    local group = buttonBase:FindFirstChild("CursedFireEffectGroup")
    if group then 
        group:Destroy() 
    end
end

local function addCursedFireTo(buttonBase)
    removeCursedFireFrom(buttonBase)
    
    pcall(function()
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
                    ColorSequenceKeypoint.new(0, Color3.fromRGB(80, 0, 0)),
                    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(140, 0, 0)),
                    ColorSequenceKeypoint.new(1, Color3.fromRGB(30, 0, 0))
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
    Debris:AddItem(sound, 2)
    
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
    
    task.delay(2, function()
        pulseTween:Cancel()
        
        local tweenOut = TweenService:Create(
            message,
            TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.In),
            {TextSize = 0}
        )
        tweenOut:Play()
        
        Debris:AddItem(message, 0.3)
    end)
end

local function showWelcomeMessage()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "WelcomeMessage"
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.ResetOnSpawn = false
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

local function bindReplacement(animationId, replacementId, speed, timePos, soundId, useFOV, useRedLight, useBlackFlashText)
    humanoid.AnimationPlayed:Connect(function(animationTrack)
        if animationTrack.Animation.AnimationId == "rbxassetid://" .. animationId then
            task.spawn(function()
                task.wait()
                
                for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
                    if track.Animation.AnimationId == "rbxassetid://" .. animationId then
                        track:Stop()
                    end
                end
                
                local AnimAnim = Instance.new("Animation")
                AnimAnim.AnimationId = "rbxassetid://" .. replacementId
                local Anim = humanoid:LoadAnimation(AnimAnim)
                Anim:Play()
                Anim:AdjustSpeed(0)
                Anim.TimePosition = timePos or 0
                Anim:AdjustSpeed(speed or 1)
                
                if soundId and hrp then
                    local sound = Instance.new("Sound")
                    sound.SoundId = "rbxassetid://" .. soundId
                    sound.Volume = 1
                    sound.Parent = hrp
                    sound:Play()
                    Debris:AddItem(sound, 5)
                end
                
                if useFOV then
                    task.spawn(function()
                        local camera = workspace.CurrentCamera
                        local originalFOV = camera.FieldOfView
                        
                        local fovIncrease = TweenService:Create(
                            camera,
                            TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                            {FieldOfView = originalFOV + 20}
                        )
                        fovIncrease:Play()
                        
                        task.wait(0.3)
                        
                        local fovDecrease = TweenService:Create(
                            camera,
                            TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
                            {FieldOfView = originalFOV}
                        )
                        fovDecrease:Play()
                    end)
                end
                
                if useRedLight then
                    task.spawn(function()
                        local Lighting = game:GetService("Lighting")
                        
                        local originalAmbient = Lighting.Ambient
                        local originalBrightness = Lighting.Brightness
                        
                        local lightTween = TweenService:Create(
                            Lighting,
                            TweenInfo.new(0.5, Enum.EasingStyle.Cubic, Enum.EasingDirection.InOut),
                            {
                                Ambient = Color3.new(0.4, 0.05, 0.05),
                                Brightness = 1
                            }
                        )
                        lightTween:Play()
                        
                        task.wait(0.5)
                        
                        local lightReturn = TweenService:Create(
                            Lighting,
                            TweenInfo.new(0.5, Enum.EasingStyle.Cubic, Enum.EasingDirection.InOut),
                            {
                                Ambient = originalAmbient,
                                Brightness = originalBrightness
                            }
                        )
                        lightReturn:Play()
                    end)
                end
                
                if useBlackFlashText then
                    task.spawn(createBlackFlashNotification)
                end
            end)
        end
    end)
end

for originalAnim, attackData in pairs(ANIMATION_REPLACEMENTS) do
    if type(attackData) == "string" then
        local config = ATTACK_CONFIG[attackData]
        if config and config.animationId then
            bindReplacement(
                originalAnim,
                config.animationId,
                config.speed,
                config.timePos,
                config.soundId,
                config.useFOV,
                config.useRedLight,
                config.useBlackFlashText
            )
        end
    elseif type(attackData) == "table" then
        bindReplacement(
            originalAnim,
            attackData.animationId,
            attackData.speed or 1,
            0,
            nil,
            false,
            false,
            false
        )
    end
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
        Anim:Play()
        
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
            track:Stop() 
        end
    end
end

local function onAnimationPlayed(animationTrack)
    local animationId = tonumber(animationTrack.Animation.AnimationId:match("%d+"))
    if animationIdsToStop[animationId] then
        stopSpecificAnimations()
        animationTrack:Stop()
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
    
    character.DescendantAdded:Connect(onBodyVelocityAdded)
    for _, descendant in pairs(character:GetDescendants()) do 
        onBodyVelocityAdded(descendant) 
    end
    
    humanoid.AnimationPlayed:Connect(onAnimationPlayed)
    
    for originalAnim, attackData in pairs(ANIMATION_REPLACEMENTS) do
        if type(attackData) == "string" then
            local config = ATTACK_CONFIG[attackData]
            if config and config.animationId then
                bindReplacement(
                    originalAnim,
                    config.animationId,
                    config.speed,
                    config.timePos,
                    config.soundId,
                    config.useFOV,
                    config.useRedLight,
                    config.useBlackFlashText
                )
            end
        elseif type(attackData) == "table" then
            bindReplacement(
                originalAnim,
                attackData.animationId,
                attackData.speed or 1,
                0,
                nil,
                false,
                false,
                false
            )
        end
    end
    
    task.wait(0.5)
    setupClickHandlers()
end)

humanoid.Died:Connect(function()
    isAlive = false
    queue = {}
    isAnimating = false
end)

setupClickHandlers()
task.wait(1)
showWelcomeMessage()
