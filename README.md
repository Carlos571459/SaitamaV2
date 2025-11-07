local og1, og2, og3, og4 = "Normal Punch", "Consecutive Punches", "Shove", "Uppercut"
local mn1, mn2, mn3, mn4 = "Black Flash", "Divergent Dam Combo", "Divergent Punch", "Black Flash is expelled"
local ultOG1, ultOG2, ultOG3, ultOG4 = "Death Counter", "Table Flip", "Serious Punch", "Omni Directional Punch"
local ult1, ult2, ult3, ult4 = "?", "?", "?", "?"

local fontConfig = {
    Enabled = true,
    Font = Enum.Font.Sarpanch,
    Weight = Enum.FontWeight.Thin,
    Style = Enum.FontStyle.Normal
}

local player = game.Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local character = player.Character or player.CharacterAdded:Wait()

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
            local toolName = baseButton:FindFirstChild("ToolName")

            if buttonGui and toolName then
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

setupClickHandlers()
coroutine.wrap(N1)()
coroutine.wrap(N2)()

-- Mostra mensagem de boas-vindas ao entrar no jogo
task.wait(1)
showWelcomeMessage()

local function createBlackFlashNotification()
    local player = game.Players.LocalPlayer
    local playerGui = player:WaitForChild("PlayerGui")
    
    -- Cria o ScreenGui se não existir
    local screenGui = playerGui:FindFirstChild("BlackFlashNotifications")
    if not screenGui then
        screenGui = Instance.new("ScreenGui")
        screenGui.Name = "BlackFlashNotifications"
        screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        screenGui.Parent = playerGui
    end
    
    -- Cria o Frame container se não existir
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
    
    -- Cria o som de notificação
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://9086333748"
    sound.Volume = 0.5
    sound.Parent = frame
    sound:Play()
    game.Debris:AddItem(sound, 2)
    
    -- Cria a mensagem SEM BACKGROUND
    local message = Instance.new("TextLabel")
    message.Text = "BLACK FLASH"
    message.Font = Enum.Font.GothamBold
    message.TextSize = 24
    message.TextColor3 = Color3.fromRGB(255, 255, 255)
    message.TextStrokeTransparency = 0.5
    message.TextStrokeColor3 = Color3.fromRGB(255, 0, 0)
    message.BackgroundTransparency = 1  -- SEM BACKGROUND
    message.BorderSizePixel = 0
    message.Size = UDim2.new(1, 0, 0, 40)
    message.TextScaled = false
    message.Parent = frame
    
    -- Adiciona brilho vermelho
    local stroke = Instance.new("UIStroke")
    stroke.Color = Color3.fromRGB(255, 0, 0)
    stroke.Thickness = 2
    stroke.Transparency = 0
    stroke.Parent = message
    
    -- Animação de entrada (crescendo)
    local origSize = message.TextSize
    message.TextSize = 0
    
    local tweenIn = game:GetService("TweenService"):Create(
        message,
        TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {TextSize = origSize}
    )
    tweenIn:Play()
    
    -- Efeito de pulso no stroke
    local pulseTween = game:GetService("TweenService"):Create(
        stroke,
        TweenInfo.new(0.3, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true),
        {Transparency = 0.5}
    )
    pulseTween:Play()
    
    -- Aguarda e remove
    task.wait(2)
    
    pulseTween:Cancel()
    
    -- Animação de saída (diminuindo até sumir)
    local tweenOut = game:GetService("TweenService"):Create(
        message,
        TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.In),
        {TextSize = 0}
    )
    tweenOut:Play()
    
    game:GetService("Debris"):AddItem(message, 0.3)
end

local function showWelcomeMessage()
    local player = game.Players.LocalPlayer
    local playerGui = player:WaitForChild("PlayerGui")
    
    -- Cria o ScreenGui
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "WelcomeMessage"
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Parent = playerGui
    
    -- Cria o Frame container
    local frame = Instance.new("Frame")
    frame.Name = "Frame"
    frame.Size = UDim2.new(0.4, 0, 0.15, 0)
    frame.Position = UDim2.new(0.3, 0, 0.425, 0)
    frame.BackgroundTransparency = 1
    frame.Parent = screenGui
    
    -- Cria a mensagem
    local message = Instance.new("TextLabel")
    message.Text = "Script By Golden_seasDeveloper"
    message.Font = Enum.Font.GothamBold
    message.TextSize = 28
    message.TextColor3 = Color3.fromRGB(255, 215, 0)
    message.TextStrokeTransparency = 0.5
    message.TextStrokeColor3 = Color3.fromRGB(255, 140, 0)
    message.BackgroundTransparency = 1
    message.BorderSizePixel = 0
    message.Size = UDim2.new(1, 0, 1, 0)
    message.TextScaled = true
    message.Parent = frame
    
    -- Adiciona brilho dourado
    local stroke = Instance.new("UIStroke")
    stroke.Color = Color3.fromRGB(255, 215, 0)
    stroke.Thickness = 2
    stroke.Transparency = 0
    stroke.Parent = message
    
    -- Animação de entrada (crescendo do centro)
    message.TextTransparency = 1
    stroke.Transparency = 1
    
    local tweenIn = game:GetService("TweenService"):Create(
        message,
        TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {TextTransparency = 0}
    )
    
    local strokeIn = game:GetService("TweenService"):Create(
        stroke,
        TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {Transparency = 0}
    )
    
    tweenIn:Play()
    strokeIn:Play()
    
    -- Efeito de pulso
    task.wait(0.5)
    local pulseTween = game:GetService("TweenService"):Create(
        stroke,
        TweenInfo.new(0.5, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, 4, true),
        {Transparency = 0.5}
    )
    pulseTween:Play()
    
    -- Aguarda e remove
    task.wait(3)
    
    pulseTween:Cancel()
    
    -- Animação de saída (diminuindo)
    local tweenOut = game:GetService("TweenService"):Create(
        message,
        TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.In),
        {TextTransparency = 1}
    )
    
    local strokeOut = game:GetService("TweenService"):Create(
        stroke,
        TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.In),
        {Transparency = 1}
    )
    
    tweenOut:Play()
    strokeOut:Play()
    
    task.wait(0.3)
    screenGui:Destroy()
end

local function createBlackFlashEffect()
    local player = game.Players.LocalPlayer
    local playerGui = player:WaitForChild("PlayerGui")
    
    -- Cria a tela de efeito
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "BlackFlashEffect"
    screenGui.Parent = playerGui
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    -- Fundo vermelho
    local background = Instance.new("Frame")
    background.Size = UDim2.new(1, 0, 1, 0)
    background.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    background.BackgroundTransparency = 1
    background.Parent = screenGui
    
    -- Texto "Black Flash"
    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(0.8, 0, 0.3, 0)
    textLabel.Position = UDim2.new(0.1, 0, 0.35, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.Text = "BLACK FLASH"
    textLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
    textLabel.TextStrokeTransparency = 0
    textLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    textLabel.Font = Enum.Font.GothamBold
    textLabel.TextScaled = true
    textLabel.TextTransparency = 1
    textLabel.Parent = screenGui
    
    -- Animação do fundo
    local bgTween = game:GetService("TweenService"):Create(
        background,
        TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {BackgroundTransparency = 0.7}
    )
    bgTween:Play()
    
    task.wait(0.1)
    
    local bgFade = game:GetService("TweenService"):Create(
        background,
        TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
        {BackgroundTransparency = 1}
    )
    bgFade:Play()
    
    -- Animação do texto
    local textAppear = game:GetService("TweenService"):Create(
        textLabel,
        TweenInfo.new(0.15, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
        {TextTransparency = 0, Size = UDim2.new(1, 0, 0.4, 0), Position = UDim2.new(0, 0, 0.3, 0)}
    )
    textAppear:Play()
    
    task.wait(0.5)
    
    local textFade = game:GetService("TweenService"):Create(
        textLabel,
        TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
        {TextTransparency = 1}
    )
    textFade:Play()
    
    task.wait(0.3)
    screenGui:Destroy()
end

local function bindReplacement(animationId, replacementId, speed, timePos, soundId, useFOV, useRedLight, useBlackFlashText)
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:WaitForChild("Humanoid")
    humanoid.AnimationPlayed:Connect(function(animationTrack)
        if animationTrack.Animation.AnimationId == "rbxassetid://" .. animationId then
            local Humanoid = player.Character:WaitForChild("Humanoid")
            for _, animTrack in pairs(Humanoid:GetPlayingAnimationTracks()) do animTrack:Stop() end
            local AnimAnim = Instance.new("Animation")
            AnimAnim.AnimationId = "rbxassetid://" .. replacementId
            local Anim = Humanoid:LoadAnimation(AnimAnim)
            Anim:Play()
            Anim:AdjustSpeed(0)
            Anim.TimePosition = timePos or 0
            Anim:AdjustSpeed(speed or 1)
            
            -- Toca o som se fornecido
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
            
            -- Efeito de FOV
            if useFOV then
                local camera = workspace.CurrentCamera
                local originalFOV = camera.FieldOfView
                
                -- Aumenta o FOV
                local fovIncrease = game:GetService("TweenService"):Create(
                    camera,
                    TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                    {FieldOfView = originalFOV + 20}
                )
                fovIncrease:Play()
                
                task.wait(0.3)
                
                -- Volta ao FOV normal
                local fovDecrease = game:GetService("TweenService"):Create(
                    camera,
                    TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
                    {FieldOfView = originalFOV}
                )
                fovDecrease:Play()
            end
            
            -- Luz vermelha (iluminação ambiente)
            if useRedLight then
                local Lighting = game:GetService("Lighting")
                local TweenService = game:GetService("TweenService")
                
                -- Salva a iluminação original
                local originalAmbient = Lighting.Ambient
                local originalBrightness = Lighting.Brightness
                
                -- Cria iluminação vermelha suave
                local lightTween = TweenService:Create(
                    Lighting,
                    TweenInfo.new(0.5, Enum.EasingStyle.Cubic, Enum.EasingDirection.InOut),
                    {
                        Ambient = Color3.new(0.4, 0.05, 0.05), -- Vermelho suave
                        Brightness = 1
                    }
                )
                lightTween:Play()
                
                -- Aguarda e volta ao normal
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
            
            -- Texto "Black Flash" (notificação estilo do seu código)
            if useBlackFlashText then
                coroutine.wrap(createBlackFlashNotification)()
            end
        end
    end)
end

-- Normal Punch com TODOS os efeitos (FOV, Luz, Texto)
bindReplacement(10468665991, 17186602996, 1, nil, 75307432501177, true, true, true)
-- Consecutive Punches (sem efeitos)
bindReplacement(10466974800, 13560306510, 3)
-- Uppercut SEM efeitos de luz e texto, apenas FOV
bindReplacement(10471336737, 18182425133, 1, nil, nil, true, false, false)
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

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

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
    humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    character.DescendantAdded:Connect(onBodyVelocityAdded)
    for _, descendant in pairs(character:GetDescendants()) do onBodyVelocityAdded(descendant) end
end)
