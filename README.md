local ANIMATION_REPLACEMENTS = {
    [10468665991] = {
        skillName = "NORMAL PUNCH",
        animationId = 17186602996,
        speed = 1,
        timePos = 0,
        soundId = 75307432501177,
        useFOV = true,
        useRedLight = true,
        useBlackFlashText = true,
        -- VFX de ataque
        useAttackVFX = true,
        vfxDelay = 0.15,
        vfxDuration = 0.8,
        vfxType = "blackFlash"
    },
    [10466974800] = {
        skillName = "CONSECUTIVE PUNCHES",
        animationId = 13560306510,
        speed = 3,
        timePos = 0,
        soundId = nil,
        useFOV = false,
        useRedLight = false,
        useBlackFlashText = false,
        -- NOVO: Fogo azul nas mãos durante o ataque
        useHandFire = true,
        handFireDuration = 2
    },
    [10471336737] = {
        skillName = "SHOVE",
        animationId = 18179181663,
        speed = 1,
        timePos = 0,
        soundId = nil,
        useFOV = true,
        useRedLight = false,
        useBlackFlashText = false,
        -- VFX de ataque
        useAttackVFX = true,
        vfxDelay = 0.2,
        vfxDuration = 1.2,
        vfxType = "shockwave"
    },
    [10470104242] = {
        skillName = "UPPERCUT",
        animationId = 17858997926,
        speed = 1.1,
        timePos = 0,
        soundId = nil,
        useFOV = false,
        useRedLight = false,
        useBlackFlashText = false,
        -- VFX de ataque
        useAttackVFX = true,
        vfxDelay = 0.25,
        vfxDuration = 1.5,
        vfxType = "uppercut"
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

-- NOVO: Fogo azul temporário nas mãos (só para CONSECUTIVE PUNCHES)
local function createTemporaryHandFire(duration)
    local leftHand = character:FindFirstChild("Left Arm") or character:FindFirstChild("LeftHand") or character:FindFirstChild("LeftLowerArm")
    local rightHand = character:FindFirstChild("Right Arm") or character:FindFirstChild("RightHand") or character:FindFirstChild("RightLowerArm")
    
    local attachments = {}
    
    local function createHandFire(hand)
        if not hand then return nil end
        
        local attachment = Instance.new("Attachment")
        attachment.Name = "TempHandFireAttachment"
        attachment.Position = Vector3.new(0, -0.5, 0)
        attachment.Parent = hand
        
        -- Fogo azul escuro
        local darkFire = Instance.new("ParticleEmitter")
        darkFire.Name = "DarkBlueFire"
        darkFire.Parent = attachment
        darkFire.Texture = "rbxassetid://11534281007"
        darkFire.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 0, 0)),
            ColorSequenceKeypoint.new(0.7, Color3.fromRGB(0, 50, 100)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 200, 255))
        })
        darkFire.Size = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0.8),
            NumberSequenceKeypoint.new(0.3, 2),
            NumberSequenceKeypoint.new(0.7, 1.5),
            NumberSequenceKeypoint.new(1, 0.4)
        })
        darkFire.Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0.3),
            NumberSequenceKeypoint.new(0.4, 0.5),
            NumberSequenceKeypoint.new(0.8, 0.8),
            NumberSequenceKeypoint.new(1, 1)
        })
        darkFire.Lifetime = NumberRange.new(0.6, 1)
        darkFire.Rate = 100
        darkFire.Speed = NumberRange.new(1, 2.5)
        darkFire.SpreadAngle = Vector2.new(25, 25)
        darkFire.Rotation = NumberRange.new(0, 360)
        darkFire.RotSpeed = NumberRange.new(-80, 80)
        darkFire.LightEmission = 0.3
        darkFire.Acceleration = Vector3.new(0, 2, 0)
        darkFire.Drag = 3
        darkFire.LockedToPart = true
        darkFire.Enabled = true
        
        -- Núcleo brilhante azul
        local core = Instance.new("ParticleEmitter")
        core.Name = "BlueCore"
        core.Parent = attachment
        core.Texture = "rbxassetid://11534281007"
        core.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
            ColorSequenceKeypoint.new(0.3, Color3.fromRGB(150, 230, 255)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 200, 255))
        })
        core.Size = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0.4),
            NumberSequenceKeypoint.new(0.2, 1.2),
            NumberSequenceKeypoint.new(0.6, 1),
            NumberSequenceKeypoint.new(1, 0.2)
        })
        core.Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0),
            NumberSequenceKeypoint.new(0.3, 0.1),
            NumberSequenceKeypoint.new(0.7, 0.5),
            NumberSequenceKeypoint.new(1, 1)
        })
        core.Lifetime = NumberRange.new(0.4, 0.7)
        core.Rate = 150
        core.Speed = NumberRange.new(2, 4)
        core.SpreadAngle = Vector2.new(15, 15)
        core.Rotation = NumberRange.new(0, 360)
        core.RotSpeed = NumberRange.new(-80, 80)
        core.LightEmission = 1
        core.Acceleration = Vector3.new(0, 3, 0)
        core.Drag = 2.5
        core.LockedToPart = true
        core.Enabled = true
        
        -- Partículas externas azuis
        local outer = Instance.new("ParticleEmitter")
        outer.Name = "BlueOuter"
        outer.Parent = attachment
        outer.Texture = "rbxassetid://11534281007"
        outer.Color = ColorSequence.new(Color3.fromRGB(0, 200, 255))
        outer.Size = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0.6),
            NumberSequenceKeypoint.new(0.3, 1.5),
            NumberSequenceKeypoint.new(0.7, 1.2),
            NumberSequenceKeypoint.new(1, 0)
        })
        outer.Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0.2),
            NumberSequenceKeypoint.new(0.4, 0.3),
            NumberSequenceKeypoint.new(0.8, 0.7),
            NumberSequenceKeypoint.new(1, 1)
        })
        outer.Lifetime = NumberRange.new(0.5, 0.9)
        outer.Rate = 120
        outer.Speed = NumberRange.new(1.5, 3)
        outer.SpreadAngle = Vector2.new(20, 20)
        outer.Rotation = NumberRange.new(0, 360)
        outer.RotSpeed = NumberRange.new(-100, 100)
        outer.LightEmission = 0.9
        outer.Acceleration = Vector3.new(0, 2.5, 0)
        outer.Drag = 2
        outer.LockedToPart = true
        outer.Enabled = true
        
        return attachment
    end
    
    if leftHand then
        table.insert(attachments, createHandFire(leftHand))
    end
    if rightHand then
        table.insert(attachments, createHandFire(rightHand))
    end
    
    task.wait(duration)
    
    for _, attachment in pairs(attachments) do
        if attachment then
            attachment:Destroy()
        end
    end
end

-- Sistema de VFX por ataque (Black Flash, Shockwave, Uppercut)
local function createBlackFlashVFX(parent, duration)
    local attachment = Instance.new("Attachment")
    attachment.Name = "BlackFlashAttachment"
    attachment.Parent = parent
    
    -- Explosão vermelha intensa
    local explosion = Instance.new("ParticleEmitter")
    explosion.Name = "BlackFlashExplosion"
    explosion.Parent = attachment
    explosion.Texture = "rbxassetid://11534281007"
    explosion.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
        ColorSequenceKeypoint.new(0.3, Color3.fromRGB(255, 50, 50)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(100, 0, 0))
    })
    explosion.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(0.1, 4),
        NumberSequenceKeypoint.new(0.5, 3),
        NumberSequenceKeypoint.new(1, 0)
    })
    explosion.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(0.5, 0.5),
        NumberSequenceKeypoint.new(1, 1)
    })
    explosion.Lifetime = NumberRange.new(0.4, 0.6)
    explosion.Rate = 200
    explosion.Speed = NumberRange.new(8, 15)
    explosion.SpreadAngle = Vector2.new(180, 180)
    explosion.Rotation = NumberRange.new(0, 360)
    explosion.RotSpeed = NumberRange.new(-200, 200)
    explosion.LightEmission = 1
    explosion.ZOffset = 0.5
    explosion.Enabled = true
    
    -- Faíscas vermelhas
    local sparks = Instance.new("ParticleEmitter")
    sparks.Name = "BlackFlashSparks"
    sparks.Parent = attachment
    sparks.Texture = "rbxassetid://11841348746"
    sparks.Color = ColorSequence.new(Color3.fromRGB(255, 100, 100))
    sparks.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.3),
        NumberSequenceKeypoint.new(1, 0.1)
    })
    sparks.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.2),
        NumberSequenceKeypoint.new(1, 1)
    })
    sparks.Lifetime = NumberRange.new(0.3, 0.5)
    sparks.Rate = 150
    sparks.Speed = NumberRange.new(15, 25)
    sparks.SpreadAngle = Vector2.new(90, 90)
    sparks.LightEmission = 1
    sparks.Enabled = true
    
    task.wait(duration)
    attachment:Destroy()
end

local function createShockwaveVFX(parent, duration)
    local attachment = Instance.new("Attachment")
    attachment.Name = "ShockwaveAttachment"
    attachment.Parent = parent
    
    -- Onda de choque azul
    local wave = Instance.new("ParticleEmitter")
    wave.Name = "ShockwaveRing"
    wave.Parent = attachment
    wave.Texture = "rbxassetid://11841348746"
    wave.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(100, 200, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 100, 200))
    })
    wave.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.5),
        NumberSequenceKeypoint.new(0.3, 5),
        NumberSequenceKeypoint.new(1, 8)
    })
    wave.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(0.5, 0.3),
        NumberSequenceKeypoint.new(1, 1)
    })
    wave.Lifetime = NumberRange.new(0.8, 1.2)
    wave.Rate = 80
    wave.Speed = NumberRange.new(10, 15)
    wave.SpreadAngle = Vector2.new(5, 5)
    wave.Rotation = NumberRange.new(0, 360)
    wave.RotSpeed = NumberRange.new(-100, 100)
    wave.LightEmission = 0.8
    wave.Acceleration = Vector3.new(0, -5, 0)
    wave.Enabled = true
    
    -- Partículas de impacto
    local impact = Instance.new("ParticleEmitter")
    impact.Name = "ShockwaveImpact"
    impact.Parent = attachment
    impact.Texture = "rbxassetid://11534281007"
    impact.Color = ColorSequence.new(Color3.fromRGB(150, 230, 255))
    impact.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 1),
        NumberSequenceKeypoint.new(0.5, 2),
        NumberSequenceKeypoint.new(1, 0.5)
    })
    impact.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.3),
        NumberSequenceKeypoint.new(1, 1)
    })
    impact.Lifetime = NumberRange.new(0.5, 0.8)
    impact.Rate = 100
    impact.Speed = NumberRange.new(5, 10)
    impact.SpreadAngle = Vector2.new(30, 30)
    impact.LightEmission = 0.9
    impact.Enabled = true
    
    task.wait(duration)
    attachment:Destroy()
end

local function createUppercutVFX(parent, duration)
    local attachment = Instance.new("Attachment")
    attachment.Name = "UppercutAttachment"
    attachment.Parent = parent
    
    -- Espiral dourada subindo
    local spiral = Instance.new("ParticleEmitter")
    spiral.Name = "UppercutSpiral"
    spiral.Parent = attachment
    spiral.Texture = "rbxassetid://11534281007"
    spiral.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 200)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 200, 50)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 150, 0))
    })
    spiral.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 1),
        NumberSequenceKeypoint.new(0.5, 2.5),
        NumberSequenceKeypoint.new(1, 0.5)
    })
    spiral.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.2),
        NumberSequenceKeypoint.new(0.7, 0.6),
        NumberSequenceKeypoint.new(1, 1)
    })
    spiral.Lifetime = NumberRange.new(1, 1.5)
    spiral.Rate = 120
    spiral.Speed = NumberRange.new(10, 20)
    spiral.SpreadAngle = Vector2.new(15, 15)
    spiral.Rotation = NumberRange.new(0, 360)
    spiral.RotSpeed = NumberRange.new(-300, 300)
    spiral.LightEmission = 1
    spiral.Acceleration = Vector3.new(0, 15, 0)
    spiral.Drag = 1
    spiral.Enabled = true
    
    -- Rastro dourado
    local trail = Instance.new("ParticleEmitter")
    trail.Name = "UppercutTrail"
    trail.Parent = attachment
    trail.Texture = "rbxassetid://11841348746"
    trail.Color = ColorSequence.new(Color3.fromRGB(255, 220, 100))
    trail.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.8),
        NumberSequenceKeypoint.new(1, 0.3)
    })
    trail.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.3),
        NumberSequenceKeypoint.new(1, 1)
    })
    trail.Lifetime = NumberRange.new(0.6, 1)
    trail.Rate = 150
    trail.Speed = NumberRange.new(8, 15)
    trail.SpreadAngle = Vector2.new(20, 20)
    trail.LightEmission = 1
    trail.Acceleration = Vector3.new(0, 10, 0)
    trail.Enabled = true
    
    task.wait(duration)
    attachment:Destroy()
end

local function triggerAttackVFX(vfxType, duration)
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    if vfxType == "blackFlash" then
        createBlackFlashVFX(hrp, duration)
    elseif vfxType == "shockwave" then
        createShockwaveVFX(hrp, duration)
    elseif vfxType == "uppercut" then
        createUppercutVFX(hrp, duration)
    end
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
            
            -- NOVO: Fogo azul nas mãos (só para CONSECUTIVE PUNCHES)
            if config.useHandFire then
                task.spawn(function()
                    createTemporaryHandFire(config.handFireDuration or 2)
                end)
            end
            
            -- VFX de ataque (Black Flash, Shockwave, Uppercut)
            if config.useAttackVFX and config.vfxType then
                task.spawn(function()
                    task.wait(config.vfxDelay or 0.2)
                    triggerAttackVFX(config.vfxType, config.vfxDuration or 1)
                end)
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
    -- Limpeza não é mais necessária pois não há efeitos permanentes
end)

player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoid = character:WaitForChild("Humanoid")
    task.wait(1)
    
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
