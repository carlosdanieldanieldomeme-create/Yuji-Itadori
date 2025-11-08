local ANIMATION_REPLACEMENTS = {
    [10468665991] = {
        skillName = "NORMAL PUNCH",
        animationId = 17186602996,
        speed = 1,
        timePos = 0,
        soundId = 75307432501177,
        useFOV = true,
        useRedLight = true,
        useBlackFlashText = true
    },
    
    [10466974800] = {
        skillName = "CONSECUTIVE PUNCHES",
        animationId = 13560306510,
        speed = 3,
        timePos = 0,
        soundId = nil,
        useFOV = false,
        useRedLight = false,
        useBlackFlashText = false
    },
    
    [10471336737] = {
        skillName = "SHOVE",
        animationId = 18179181663,
        speed = 1,
        timePos = 0,
        soundId = nil,
        useFOV = true,
        useRedLight = false,
        useBlackFlashText = false
    },
    
    [10470104242] = {
        skillName = "UPPERCUT",
        animationId = 17858997926,
        speed = 1.1,
        timePos = 0,
        soundId = nil,
        useFOV = false,
        useRedLight = false,
        useBlackFlashText = false
    },
    
    [12510170988] = {
        skillName = "SKILL 5",
        animationId = 18897119503,
        speed = 1
    },
    
    [11343318134] = {
        skillName = "SKILL 6",
        animationId = 18450698238,
        speed = 1
    },
    
    [11365563255] = {
        skillName = "SKILL 7",
        animationId = 17861840167,
        speed = 0.3
    },
    
    [13927612951] = {
        skillName = "SKILL 8",
        animationId = 18459220516,
        speed = 1
    },
    
    [15955393872] = {
        skillName = "SKILL 9",
        animationId = 18459220516,
        speed = 1
    },
    
    [12983333733] = {
        skillName = "SKILL 10",
        animationId = 120001337057214,
        speed = 0.55
    },
    
    [12447707844] = {
        skillName = "SKILL 11",
        animationId = 106128760138039,
        speed = 1
    },
    
    [10479335397] = {
        skillName = "SKILL 12",
        animationId = 132259592388175,
        speed = 1
    },
    
    [10503381238] = {
        skillName = "SKILL 13",
        animationId = 14900168720,
        speed = 1
    }
}

local FULL_REPLACEMENT_ANIMATIONS = {
    [17859015788] = {
        skillName = "FULL REPLACEMENT 1",
        animationId = 12684185971,
        speed = 1
    },
    [10469493270] = {
        skillName = "FULL REPLACEMENT 2",
        animationId = 13491635433,
        speed = 1
    },
    [10469630950] = {
        skillName = "FULL REPLACEMENT 3",
        animationId = 13532600125,
        speed = 1
    },
    [10469639222] = {
        skillName = "FULL REPLACEMENT 4",
        animationId = 104895379416342,
        speed = 1
    },
    [10469643643] = {
        skillName = "FULL REPLACEMENT 5",
        animationId = 18181348446,
        speed = 1
    }
}

local ATTACK_NAMES = {
    ["Normal Punch"] = "Black Flash",
    ["Consecutive Punches"] = "Divergent Dam Combo",
    ["Shove"] = "Black Flash is expelled",
    ["Uppercut"] = "Divergent Punch"
}

local ULTIMATE_NAMES = {}

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

local function updateAttackNames()
    while character.Humanoid.Health > 0 do
        local hotbar = playerGui:FindFirstChild("Hotbar")
        if hotbar then
            local backpack = hotbar:FindFirstChild("Backpack")
            if backpack then
                local hotbarFrame = backpack:FindFirstChild("Hotbar")
                if hotbarFrame then
                    for i = 1, 4 do
                        local baseButton = hotbarFrame:FindFirstChild(tostring(i))
                        baseButton = baseButton and baseButton:FindFirstChild("Base")
                        if baseButton then
                            local toolName = baseButton:FindFirstChild("ToolName")
                            if toolName then
                                for originalName, newName in pairs(ATTACK_NAMES) do
                                    if toolName.Text == originalName then
                                        toolName.Text = newName
                                        applyFont(toolName)
                                    end
                                end
                                
                                for originalName, newName in pairs(ULTIMATE_NAMES) do
                                    if toolName.Text == originalName then
                                        toolName.Text = newName
                                        applyFont(toolName)
                                    end
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
    local ts = game:GetService("TweenService")
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then return end
    
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://9086333748"
    sound.Volume = 0.5
    sound.Parent = humanoidRootPart
    sound:Play()
    game.Debris:AddItem(sound, 2)
    
    local Dialogue = Instance.new("BillboardGui")
    local Chat1 = Instance.new("Frame")
    local Sub = Instance.new("TextLabel")
    
    Dialogue.Active = true
    Dialogue.Size = UDim2.new(15, 0, 15, 0)
    Dialogue.StudsOffset = Vector3.new(0, 0, 2)
    Dialogue.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    Dialogue.Name = "Dialogue"
    Dialogue.Parent = humanoidRootPart
    
    Chat1.AnchorPoint = Vector2.new(0.5, 0.5)
    Chat1.BackgroundColor3 = Color3.new(1, 1, 1)
    Chat1.BorderColor3 = Color3.new(0, 0, 0)
    Chat1.BorderSizePixel = 2
    Chat1.Position = UDim2.new(0.6, 0, -0.2, 0)
    Chat1.Size = UDim2.new(0.1, 0, 0.2, 0)
    Chat1.Name = "Chat1"
    Chat1.BackgroundTransparency = 1
    Chat1.Parent = Dialogue
    
    Sub.FontFace = Font.new("rbxassetid://12187375716", Enum.FontWeight.Bold, Enum.FontStyle.Italic)
    Sub.Text = "BLACK FLASH"
    Sub.TextColor3 = Color3.new(0, 0, 0)
    Sub.TextScaled = true
    Sub.TextSize = 14
    Sub.TextWrapped = true
    Sub.AnchorPoint = Vector2.new(0.5, 0.5)
    Sub.BackgroundColor3 = Color3.new(1, 1, 1)
    Sub.TextTransparency = 1
    Sub.BorderColor3 = Color3.new(0, 0, 0)
    Sub.BorderSizePixel = 0
    Sub.Position = UDim2.new(0.5, 0, 0.5, 0)
    Sub.Size = UDim2.new(0.85, 0, 0.35, 0)
    Sub.Name = "Sub"
    Sub.Parent = Chat1
    
    game.Debris:AddItem(Dialogue, 6)
    
    local function tweenProperty(object, properties, time)
        local tweenInfo = TweenInfo.new(time, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
        local tween = ts:Create(object, tweenInfo, properties)
        tween:Play()
        return tween
    end
    
    tweenProperty(Chat1, {BackgroundTransparency = 0}, 1)
    tweenProperty(Sub, {TextTransparency = 0}, 1)
    tweenProperty(Chat1, {Position = UDim2.new(0.6, 0, 0.4, 0)}, 1)
    task.wait(4)
    tweenProperty(Chat1, {BackgroundTransparency = 1}, 2)
    tweenProperty(Sub, {TextTransparency = 1}, 2)
end

local function setupAnimationReplacement(originalId, config)
    humanoid.AnimationPlayed:Connect(function(animationTrack)
        if animationTrack.Animation.AnimationId == "rbxassetid://" .. originalId then
            task.wait()
            
            for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
                if track.Animation.AnimationId == "rbxassetid://" .. originalId then
                    track:Stop()
                end
            end
            
            local AnimAnim = Instance.new("Animation")
            AnimAnim.AnimationId = "rbxassetid://" .. config.animationId
            local Anim = humanoid:LoadAnimation(AnimAnim)
            Anim:Play()
            Anim:AdjustSpeed(0)
            Anim.TimePosition = config.timePos or 0
            Anim:AdjustSpeed(config.speed or 1)
            
            if config.soundId then
                local hrp = character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    local sound = Instance.new("Sound")
                    sound.SoundId = "rbxassetid://" .. config.soundId
                    sound.Volume = 1
                    sound.Parent = hrp
                    sound:Play()
                    game.Debris:AddItem(sound, 5)
                end
            end
            
            if config.useFOV then
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
            
            if config.useRedLight then
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
            
            if config.useBlackFlashText then
                coroutine.wrap(createBlackFlashNotification)()
            end
        end
    end)
end

local queue, isAnimating = {}, false

local function playFullReplacement(originalId, config)
    if isAnimating then
        table.insert(queue, {originalId, config})
        return
    end
    isAnimating = true
    
    local AnimAnim = Instance.new("Animation")
    AnimAnim.AnimationId = "rbxassetid://" .. config.animationId
    local Anim = humanoid:LoadAnimation(AnimAnim)
    Anim:Play()
    Anim:AdjustSpeed(config.speed or 1)
    
    Anim.Stopped:Connect(function()
        isAnimating = false
        if #queue > 0 then
            local next = table.remove(queue, 1)
            playFullReplacement(next[1], next[2])
        end
    end)
end

local function setupFullReplacement(originalId, config)
    humanoid.AnimationPlayed:Connect(function(animationTrack)
        local animId = tonumber(animationTrack.Animation.AnimationId:match("%d+"))
        if animId == originalId then
            for _, track in ipairs(humanoid:GetPlayingAnimationTracks()) do
                local trackId = tonumber(track.Animation.AnimationId:match("%d+"))
                if trackId == originalId then
                    track:Stop()
                end
            end
            animationTrack:Stop()
            playFullReplacement(originalId, config)
        end
    end)
end

local function onBodyVelocityAdded(bodyVelocity)
    if bodyVelocity:IsA("BodyVelocity") then
        bodyVelocity.Velocity = Vector3.new(bodyVelocity.Velocity.X, 0, bodyVelocity.Velocity.Z)
    end
end

for originalId, config in pairs(ANIMATION_REPLACEMENTS) do
    setupAnimationReplacement(originalId, config)
end

for originalId, config in pairs(FULL_REPLACEMENT_ANIMATIONS) do
    setupFullReplacement(originalId, config)
end

coroutine.wrap(updateAttackNames)()

character.DescendantAdded:Connect(onBodyVelocityAdded)
for _, descendant in pairs(character:GetDescendants()) do 
    onBodyVelocityAdded(descendant) 
end

player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = character:WaitForChild("Humanoid")
    
    for originalId, config in pairs(ANIMATION_REPLACEMENTS) do
        setupAnimationReplacement(originalId, config)
    end
    
    for originalId, config in pairs(FULL_REPLACEMENT_ANIMATIONS) do
        setupFullReplacement(originalId, config)
    end
    
    coroutine.wrap(updateAttackNames)()
    
    character.DescendantAdded:Connect(onBodyVelocityAdded)
    for _, descendant in pairs(character:GetDescendants()) do 
        onBodyVelocityAdded(descendant) 
    end
end)
