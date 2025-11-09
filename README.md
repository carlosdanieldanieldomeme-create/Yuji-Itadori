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

local COLORS = {
    primary = Color3.fromRGB(0, 200, 255),
    secondary = Color3.fromRGB(0, 150, 255),
    bright = Color3.fromRGB(255, 255, 255),
    glow = Color3.fromRGB(150, 230, 255),
    dark = Color3.fromRGB(0, 0, 0),
    darkBlue = Color3.fromRGB(0, 50, 80)
}

local player = game.Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

local effectGroups = {}

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
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://9086333748"
    sound.Volume = 0.5
    sound.Parent = hrp
    sound:Play()
    game.Debris:AddItem(sound, 2)
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "BlackFlashBillboard"
    billboard.Size = UDim2.new(3.9, 0, 4, 0)
    billboard.StudsOffset = Vector3.new(-2, 4.1, 0)
    billboard.AlwaysOnTop = true
    billboard.Parent = hrp
    local balloon = Instance.new("ImageLabel")
    balloon.Name = "Balloon"
    balloon.Image = "rbxassetid://136797177442983"
    balloon.Size = UDim2.new(1, 0, 1, 0)
    balloon.Position = UDim2.new(0, 0, 0, 0)
    balloon.BackgroundTransparency = 1
    balloon.Parent = billboard
    local blackFlashText = Instance.new("ImageLabel")
    blackFlashText.Name = "BlackFlashText"
    blackFlashText.Image = "rbxassetid://17702987052"
    blackFlashText.Size = UDim2.new(0.613, 0, 0.6, 0)
    blackFlashText.Position = UDim2.new(0.519, 0, 0.5, 0)
    blackFlashText.AnchorPoint = Vector2.new(0.5, 0.5)
    blackFlashText.BackgroundTransparency = 1
    blackFlashText.Parent = balloon
    billboard.Size = UDim2.new(0, 0, 0, 0)
    local tweenIn = game:GetService("TweenService"):Create(billboard, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(3.9, 0, 4, 0)})
    tweenIn:Play()
    task.wait(2)
    local tweenOut = game:GetService("TweenService"):Create(billboard, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0)})
    tweenOut:Play()
    game:GetService("Debris"):AddItem(billboard, 0.3)
end

local function createAdvancedEnergyEffect(hand, scale)
    if not hand then return nil end
    local attachment = Instance.new("Attachment")
    attachment.Name = "EnergyAttachment"
    attachment.Position = Vector3.new(0, -0.5, 0)
    attachment.Parent = hand
    local effects = {}
    scale = scale or 1
    local darkFire = Instance.new("ParticleEmitter")
    darkFire.Name = "DarkFire"
    darkFire.Parent = attachment
    darkFire.Texture = "rbxassetid://11534281007"
    darkFire.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, COLORS.dark), ColorSequenceKeypoint.new(0.7, COLORS.dark), ColorSequenceKeypoint.new(1, COLORS.primary)})
    darkFire.Size = NumberSequence.new({NumberSequenceKeypoint.new(0, 1 * scale), NumberSequenceKeypoint.new(0.3, 2.5 * scale), NumberSequenceKeypoint.new(0.7, 2 * scale), NumberSequenceKeypoint.new(1, 0.5 * scale)})
    darkFire.Transparency = NumberSequence.new({NumberSequenceKeypoint.new(0, 0.3), NumberSequenceKeypoint.new(0.4, 0.5), NumberSequenceKeypoint.new(0.8, 0.8), NumberSequenceKeypoint.new(1, 1)})
    darkFire.Lifetime = NumberRange.new(0.6, 1)
    darkFire.Rate = 80 * scale
    darkFire.Speed = NumberRange.new(1, 2.5)
    darkFire.SpreadAngle = Vector2.new(25, 25)
    darkFire.Rotation = NumberRange.new(0, 360)
    darkFire.RotSpeed = NumberRange.new(-80, 80)
    darkFire.LightEmission = 0
    darkFire.ZOffset = -0.1
    darkFire.Acceleration = Vector3.new(0, 2, 0)
    darkFire.Drag = 3
    darkFire.VelocityInheritance = 0
    darkFire.LockedToPart = true
    darkFire.TimeScale = 1
    darkFire.EmissionDirection = Enum.NormalId.Top
    darkFire.WindAffectsDrag = false
    darkFire.FlipbookLayout = Enum.ParticleFlipbookLayout.Grid4x4
    darkFire.FlipbookMode = Enum.ParticleFlipbookMode.OneShot
    darkFire.FlipbookStartRandom = false
    table.insert(effects, darkFire)
    local core = Instance.new("ParticleEmitter")
    core.Name = "EnergyCore"
    core.Parent = attachment
    core.Texture = "rbxassetid://11534281007"
    core.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, COLORS.bright), ColorSequenceKeypoint.new(0.3, COLORS.glow), ColorSequenceKeypoint.new(1, COLORS.primary)})
    core.Size = NumberSequence.new({NumberSequenceKeypoint.new(0, 0.5 * scale), NumberSequenceKeypoint.new(0.2, 1.5 * scale), NumberSequenceKeypoint.new(0.6, 1.2 * scale), NumberSequenceKeypoint.new(1, 0.3 * scale)})
    core.Transparency = NumberSequence.new({NumberSequenceKeypoint.new(0, 0), NumberSequenceKeypoint.new(0.3, 0.1), NumberSequenceKeypoint.new(0.7, 0.5), NumberSequenceKeypoint.new(1, 1)})
    core.Lifetime = NumberRange.new(0.4, 0.7)
    core.Rate = 120 * scale
    core.Speed = NumberRange.new(2, 4)
    core.SpreadAngle = Vector2.new(15, 15)
    core.Rotation = NumberRange.new(0, 360)
    core.RotSpeed = NumberRange.new(-80, 80)
    core.LightEmission = 1
    core.LightInfluence = 0
    core.ZOffset = 0.3
    core.Acceleration = Vector3.new(0, 3, 0)
    core.Drag = 2.5
    core.VelocityInheritance = 0
    core.LockedToPart = true
    core.TimeScale = 1
    core.WindAffectsDrag = false
    core.EmissionDirection = Enum.NormalId.Top
    core.FlipbookLayout = Enum.ParticleFlipbookLayout.Grid4x4
    core.FlipbookMode = Enum.ParticleFlipbookMode.OneShot
    core.FlipbookStartRandom = false
    table.insert(effects, core)
    local outer = Instance.new("ParticleEmitter")
    outer.Name = "EnergyOuter"
    outer.Parent = attachment
    outer.Texture = "rbxassetid://11534281007"
    outer.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, COLORS.primary), ColorSequenceKeypoint.new(0.5, COLORS.secondary), ColorSequenceKeypoint.new(1, COLORS.primary)})
    outer.Size = NumberSequence.new({NumberSequenceKeypoint.new(0, 0.8 * scale), NumberSequenceKeypoint.new(0.3, 2 * scale), NumberSequenceKeypoint.new(0.7, 1.5 * scale), NumberSequenceKeypoint.new(1, 0)})
    outer.Transparency = NumberSequence.new({NumberSequenceKeypoint.new(0, 0.2), NumberSequenceKeypoint.new(0.4, 0.3), NumberSequenceKeypoint.new(0.8, 0.7), NumberSequenceKeypoint.new(1, 1)})
    outer.Lifetime = NumberRange.new(0.5, 0.9)
    outer.Rate = 100 * scale
    outer.Speed = NumberRange.new(1.5, 3)
    outer.SpreadAngle = Vector2.new(20, 20)
    outer.Rotation = NumberRange.new(0, 360)
    outer.RotSpeed = NumberRange.new(-100, 100)
    outer.LightEmission = 0.9
    outer.ZOffset = 0.2
    outer.Acceleration = Vector3.new(0, 2.5, 0)
    outer.Drag = 2
    outer.VelocityInheritance = 0
    outer.LockedToPart = true
    outer.TimeScale = 1
    outer.EmissionDirection = Enum.NormalId.Top
    outer.WindAffectsDrag = false
    outer.FlipbookLayout = Enum.ParticleFlipbookLayout.Grid4x4
    outer.FlipbookMode = Enum.ParticleFlipbookMode.OneShot
    outer.FlipbookStartRandom = false
    table.insert(effects, outer)
    local baseGlow = Instance.new("ParticleEmitter")
    baseGlow.Name = "BaseGlow"
    baseGlow.Parent = attachment
    baseGlow.Texture = "rbxassetid://11841348746"
    baseGlow.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, COLORS.bright), ColorSequenceKeypoint.new(0.5, COLORS.glow), ColorSequenceKeypoint.new(1, COLORS.primary)})
    baseGlow.Size = NumberSequence.new({NumberSequenceKeypoint.new(0, 1.2 * scale), NumberSequenceKeypoint.new(0.4, 1.8 * scale), NumberSequenceKeypoint.new(1, 0)})
    baseGlow.Transparency = NumberSequence.new({NumberSequenceKeypoint.new(0, 0.3), NumberSequenceKeypoint.new(0.5, 0.6), NumberSequenceKeypoint.new(1, 1)})
    baseGlow.Lifetime = NumberRange.new(0.2, 0.4)
    baseGlow.Rate = 80 * scale
    baseGlow.Speed = NumberRange.new(0.5, 1.5)
    baseGlow.SpreadAngle = Vector2.new(25, 25)
    baseGlow.Rotation = NumberRange.new(0, 360)
    baseGlow.RotSpeed = NumberRange.new(-150, 150)
    baseGlow.LightEmission = 1
    baseGlow.ZOffset = 0.1
    baseGlow.Acceleration = Vector3.new(0, 2, 0)
    baseGlow.Drag = 1
    baseGlow.VelocityInheritance = 0
    baseGlow.LockedToPart = true
    baseGlow.TimeScale = 1
    baseGlow.EmissionDirection = Enum.NormalId.Top
    baseGlow.WindAffectsDrag = false
    baseGlow.FlipbookLayout = Enum.ParticleFlipbookLayout.Grid4x4
    baseGlow.FlipbookMode = Enum.ParticleFlipbookMode.OneShot
    baseGlow.FlipbookStartRandom = false
    table.insert(effects, baseGlow)
    local wisps = Instance.new("ParticleEmitter")
    wisps.Name = "EnergyWisps"
    wisps.Parent = attachment
    wisps.Texture = "rbxassetid://11841348746"
    wisps.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, COLORS.glow), ColorSequenceKeypoint.new(1, COLORS.secondary)})
    wisps.Size = NumberSequence.new({NumberSequenceKeypoint.new(0, 0.3 * scale), NumberSequenceKeypoint.new(0.5, 0.6 * scale), NumberSequenceKeypoint.new(1, 0.2 * scale)})
    wisps.Transparency = NumberSequence.new({NumberSequenceKeypoint.new(0, 0.2), NumberSequenceKeypoint.new(0.6, 0.6), NumberSequenceKeypoint.new(1, 1)})
    wisps.Lifetime = NumberRange.new(0.6, 1)
    wisps.Rate = 60 * scale
    wisps.Speed = NumberRange.new(3, 5)
    wisps.SpreadAngle = Vector2.new(12, 12)
    wisps.Rotation = NumberRange.new(0, 360)
    wisps.RotSpeed = NumberRange.new(-200, 200)
    wisps.LightEmission = 1
    wisps.ZOffset = 0.4
    wisps.Acceleration = Vector3.new(0, 4, 0)
    wisps.Drag = 1.8
    wisps.VelocityInheritance = 0
    wisps.LockedToPart = true
    wisps.TimeScale = 1
    wisps.EmissionDirection = Enum.NormalId.Top
    wisps.WindAffectsDrag = false
    wisps.FlipbookLayout = Enum.ParticleFlipbookLayout.Grid4x4
    wisps.FlipbookMode = Enum.ParticleFlipbookMode.OneShot
    wisps.FlipbookStartRandom = false
    table.insert(effects, wisps)
    return {attachment = attachment, effects = effects}
end

local function setupEnergyEffects()
    for _, group in pairs(effectGroups) do
        if group then
            for _, effect in pairs(group.effects) do
                if typeof(effect) == "Instance" then effect:Destroy() end
            end
            if group.attachment then group.attachment:Destroy() end
        end
    end
    effectGroups = {}
    local leftHand = character:FindFirstChild("Left Arm") or character:FindFirstChild("LeftHand") or character:FindFirstChild("LeftLowerArm")
    local rightHand = character:FindFirstChild("Right Arm") or character:FindFirstChild("RightHand") or character:FindFirstChild("RightLowerArm")
    if leftHand then effectGroups.left = createAdvancedEnergyEffect(leftHand, 0.8) end
    if rightHand then effectGroups.right = createAdvancedEnergyEffect(rightHand, 0.8) end
end

local function setupAnimationReplacement(originalId, config)
    humanoid.AnimationPlayed:Connect(function(animationTrack)
        if animationTrack.Animation.AnimationId == "rbxassetid://" .. originalId then
            task.wait()
            for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
                if track.Animation.AnimationId == "rbxassetid://" .. originalId then track:Stop() end
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
                local fovIncrease = game:GetService("TweenService"):Create(camera, TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {FieldOfView = originalFOV + 20})
                fovIncrease:Play()
                task.wait(0.3)
                local fovDecrease = game:GetService("TweenService"):Create(camera, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {FieldOfView = originalFOV})
                fovDecrease:Play()
            end
            if config.useRedLight then
                local Lighting = game:GetService("Lighting")
                local TweenService = game:GetService("TweenService")
                local originalAmbient = Lighting.Ambient
                local originalBrightness = Lighting.Brightness
                local lightTween = TweenService:Create(Lighting, TweenInfo.new(0.5, Enum.EasingStyle.Cubic, Enum.EasingDirection.InOut), {Ambient = Color3.new(0.4, 0.05, 0.05), Brightness = 1})
                lightTween:Play()
                task.wait(0.5)
                local lightReturn = TweenService:Create(Lighting, TweenInfo.new(0.5, Enum.EasingStyle.Cubic, Enum.EasingDirection.InOut), {Ambient = originalAmbient, Brightness = originalBrightness})
                lightReturn:Play()
            end
            if config.useBlackFlashText then coroutine.wrap(createBlackFlashNotification)() end
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
                if trackId == originalId then track:Stop() end
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

task.wait(1)
setupEnergyEffects()

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

humanoid.Died:Connect(function()
    for _, group in pairs(effectGroups) do
        if group then
            for _, effect in pairs(group.effects) do
                if typeof(effect) == "Instance" then effect:Destroy()
                elseif typeof(effect) == "RBXScriptConnection" then effect:Disconnect() end
            end
            if group.attachment then group.attachment:Destroy() end
        end
    end
end)

player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = character:WaitForChild("Humanoid")
    task.wait(1)
    setupEnergyEffects()
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
    humanoid.Died:Connect(function()
        for _, group in pairs(effectGroups) do
            if group then
                for _, effect in pairs(group.effects) do
                    if typeof(effect) == "Instance" then effect:Destroy()
                    elseif typeof(effect) == "RBXScriptConnection" then effect:Disconnect() end
                end
                if group.attachment then group.attachment:Destroy() end
            end
        end
    end)
end)
