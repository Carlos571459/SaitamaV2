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
    [10468665991] = "Normal Punch",
    [10466974800] = "Consecutive Punches",
    [10471336737] = "Shove",
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
    
    local tweenIn = game:GetService("TweenService"):Create(
        message,
        TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {TextSize = origSize}
    )
    tweenIn:Play()
    
    local pulseTween = game:GetService("TweenService"):Create(
        stroke,
        TweenInfo.new(0.3, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true),
        {Transparency = 0.5}
    )
    pulseTween:Play()
    
    task.wait(2)
    
    pulseTween:Cancel()
    
    local tweenOut = game:GetService("TweenService"):Create(
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
    
    local TweenService = game:GetService("TweenService")
    
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
            
            if soundId then
                local hrp = character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    local sound = Instance.new("Sound")
                    sound.SoundId = "rbxassetid://" .. soundId
                    sound.Volume = 1
                    sound.Parent = hrp
                    sound:Play()
                    game.Debris:AddItem(sound, 5)
                end
            end
            
            if useFOV then
                local camera = workspace.CurrentCamera
                local originalFOV = camera.FieldOfView
                
                local fovIncrease = game:GetService("TweenService"):Create(
                    camera,
                    TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                    {FieldOfView = originalFOV + 20}
                )
                fovIncrease:Play()
                
                task.wait(0.3)
                
                local fovDecrease = game:GetService("TweenService"):Create(
                    camera,
                    TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
                    {FieldOfView = originalFOV}
                )
                fovDecrease:Play()
            end
            
            if useRedLight then
                local Lighting = game:GetService("Lighting")
                local TweenService = game:GetService("TweenService")
                
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
            end
            
            if useBlackFlashText then
                coroutine.wrap(createBlackFlashNotification)()
            end
        end
    end)
end

bindReplacement(10468665991, 17186602996, 1, nil, 75307432501177, true, true, true)
bindReplacement(10466974800, 13560306510, 3)
bindReplacement(10471336737, 18179181663, 1, nil, nil, true, false, false)
bindReplacement(12510170988, 18897119503, 1)
bindReplacement(11343318134, 18450698238, 1)
bindReplacement(11365563255, 17861840167, 0.3)
bindReplacement(13927612951, 18459220516, 1)
bindReplacement(15955393872, 18459220516, 1)
bindReplacement(12983333733, 120001337057214, 0.55)
bindReplacement(12447707844, 106128760138039, 1)
bindReplacement(10479335397, 132259592388175, 1)
bindReplacement(10503381238, 14900168720, 1)
bindReplacement(10470104242, 17858997926, 1.1)

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

local queue, isAnimating = {}, false

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
        if animationIdsToStop[animationId] then track:Stop() end
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

local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

local function onBodyVelocityAdded(bodyVelocity)
    if bodyVelocity:IsA("BodyVelocity") then
        bodyVelocity.Velocity = Vector3.new(bodyVelocity.Velocity.X, 0, bodyVelocity.Velocity.Z)
    end
end

character.DescendantAdded:Connect(onBodyVelocityAdded)
for _, descendant in pairs(character:GetDescendants()) do onBodyVelocityAdded(descendant) end

player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = character:WaitForChild("Humanoid")
    humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    character.DescendantAdded:Connect(onBodyVelocityAdded)
    for _, descendant in pairs(character:GetDescendants()) do onBodyVelocityAdded(descendant) end
end)

setupClickHandlers()
coroutine.wrap(N1)()
coroutine.wrap(N2)()

task.wait(1)
showWelcomeMessage()
