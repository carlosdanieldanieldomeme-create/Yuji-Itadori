local Services = {
    TweenService = game:GetService("TweenService"),
    Debris = game:GetService("Debris"),
    Lighting = game:GetService("Lighting"),
    ReplicatedStorage = game:GetService("ReplicatedStorage")
}

local CONFIG = {
    ANIMATION_REPLACEMENTS = {
        [10468665991] = {
            skillName = "NORMAL PUNCH",
            animationId = 17186602996,
            speed = 1,
            timePos = 0,
            soundId = 75307432501177,
            useFOV = true,
            useRedLight = true,
            useBlackFlashText = true,
            useCustomVFX = true,
            vfxDelay = 0.15
        },
        [10466974800] = {
            skillName = "CONSECUTIVE PUNCHES",
            animationId = 13560306510,
            speed = 3,
            timePos = 0,
            useFOV = false,
            useRedLight = false,
            useBlackFlashText = false,
            useAdvancedHandFire = true,
            handFireDuration = 1.6,
            includeClones = true
        },
        [10471336737] = {
            skillName = "SHOVE",
            animationId = 16944265635,
            speed = 1,
            timePos = 0
        },
        [10470104242] = {
            skillName = "UPPERCUT",
            animationId = 18179181663,
            speed = 1.1,
            timePos = 0
        },
        [12510170988] = {
            skillName = "SKILL 5",
            animationId = 18179181663,
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
    },
    
    FULL_REPLACEMENT_ANIMATIONS = {
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
    },
    
    ATTACK_NAMES = {
        ["Normal Punch"] = "Black Flash",
        ["Consecutive Punches"] = "Barragem Divergent",
        ["Shove"] = "Divergent Scan",
        ["Uppercut"] = "Storm Kick Black Flash"
    },
    
    ULTIMATE_NAMES = {},
    
    FONT = {
        Enabled = true,
        Font = Enum.Font.Sarpanch,
        Weight = Enum.FontWeight.Thin,
        Style = Enum.FontStyle.Normal
    },
    
    COLORS = {
        primary = Color3.fromRGB(0, 200, 255),
        secondary = Color3.fromRGB(0, 150, 255),
        bright = Color3.fromRGB(255, 255, 255),
        glow = Color3.fromRGB(150, 230, 255),
        dark = Color3.fromRGB(0, 0, 0),
        darkBlue = Color3.fromRGB(0, 50, 80),
        redDark = Color3.fromRGB(80, 0, 0),
        redMid = Color3.fromRGB(140, 0, 0),
        redLight = Color3.fromRGB(30, 0, 0)
    }
}

local PlayerData = {
    player = game.Players.LocalPlayer,
    character = nil,
    humanoid = nil,
    playerGui = nil,
    animationQueue = {},
    isAnimating = false
}

local function initializePlayerData()
    PlayerData.playerGui = PlayerData.player:WaitForChild("PlayerGui")
    PlayerData.character = PlayerData.player.Character or PlayerData.player.CharacterAdded:Wait()
    PlayerData.humanoid = PlayerData.character:WaitForChild("Humanoid")
end

local FontManager = {}

function FontManager.applyFont(label)
    if not CONFIG.FONT.Enabled or not label:IsA("TextLabel") then
        return
    end
    pcall(function()
        label.FontFace = Font.new(label.FontFace.Family, CONFIG.FONT.Weight, CONFIG.FONT.Style)
        label.Font = CONFIG.FONT.Font
    end)
end

local CursedFireManager = {}

function CursedFireManager.remove(buttonBase)
    if not buttonBase then return end
    local group = buttonBase:FindFirstChild("CursedFireEffectGroup")
    if group then group:Destroy() end
end

function CursedFireManager.add(buttonBase)
    if not buttonBase then return end
    CursedFireManager.remove(buttonBase)
    pcall(function()
        local hotbar = PlayerData.playerGui:FindFirstChild("Hotbar")
        if not hotbar then return end
        local backpack = hotbar:FindFirstChild("Backpack")
        if not backpack then return end
        local localScript = backpack:FindFirstChild("LocalScript")
        if not localScript then return end
        local kj = localScript:FindFirstChild("Flipbook")
        if not kj then return end
        local clone = kj:Clone()
        local group = Instance.new("Folder")
        group.Name = "CursedFireEffectGroup"
        group.Parent = buttonBase
        clone.Parent = group
        clone.LocalScript.Enabled = true
        for _, desc in pairs(clone:GetDescendants()) do
            if desc:IsA("ParticleEmitter") then
                desc.Color = ColorSequence.new({
                    ColorSequenceKeypoint.new(0, CONFIG.COLORS.redDark),
                    ColorSequenceKeypoint.new(0.5, CONFIG.COLORS.redMid),
                    ColorSequenceKeypoint.new(1, CONFIG.COLORS.redLight)
                })
            end
        end
    end)
end

local UIManager = {}

function UIManager.getHotbarButton(index)
    local success, result = pcall(function()
        local hotbar = PlayerData.playerGui:FindFirstChild("Hotbar")
        if not hotbar then return nil end
        local backpack = hotbar:FindFirstChild("Backpack")
        if not backpack then return nil end
        local hotbarFrame = backpack:FindFirstChild("Hotbar")
        if not hotbarFrame then return nil end
        local button = hotbarFrame:FindFirstChild(tostring(index))
        if not button then return nil end
        return button:FindFirstChild("Base")
    end)
    return success and result or nil
end

function UIManager.updateAttackNames()
    while PlayerData.character and PlayerData.humanoid and PlayerData.humanoid.Health > 0 do
        pcall(function()
            for i = 1, 4 do
                local baseButton = UIManager.getHotbarButton(i)
                if baseButton then
                    local toolName = baseButton:FindFirstChild("ToolName")
                    if toolName then
                        for originalName, newName in pairs(CONFIG.ATTACK_NAMES) do
                            if toolName.Text == originalName then
                                toolName.Text = newName
                                FontManager.applyFont(toolName)
                            end
                        end
                        for originalName, newName in pairs(CONFIG.ULTIMATE_NAMES) do
                            if toolName.Text == originalName then
                                toolName.Text = newName
                                FontManager.applyFont(toolName)
                            end
                        end
                        if toolName.Text == "Do not change here" then
                            CursedFireManager.add(baseButton)
                        else
                            CursedFireManager.remove(baseButton)
                        end
                    end
                end
            end
        end)
        task.wait(0.1)
    end
end

local NotificationManager = {}

function NotificationManager.createBlackFlash()
    pcall(function()
        local hrp = PlayerData.character:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        local sound = Instance.new("Sound")
        sound.SoundId = "rbxassetid://9086333748"
        sound.Volume = 0.5
        sound.Parent = hrp
        sound:Play()
        Services.Debris:AddItem(sound, 2)
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
        local tweenIn = Services.TweenService:Create(billboard, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(3.9, 0, 4, 0)})
        tweenIn:Play()
        task.wait(2)
        local tweenOut = Services.TweenService:Create(billboard, TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.In), {Size = UDim2.new(0, 0, 0, 0)})
        tweenOut:Play()
        Services.Debris:AddItem(billboard, 0.3)
    end)
end

local VFXManager = {}

function VFXManager.createAdvancedEnergyEffect(hand, scale)
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
    darkFire.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, CONFIG.COLORS.dark), ColorSequenceKeypoint.new(0.7, CONFIG.COLORS.dark), ColorSequenceKeypoint.new(1, CONFIG.COLORS.primary)})
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
    darkFire.Enabled = true
    table.insert(effects, darkFire)
    local core = Instance.new("ParticleEmitter")
    core.Name = "EnergyCore"
    core.Parent = attachment
    core.Texture = "rbxassetid://11534281007"
    core.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, CONFIG.COLORS.bright), ColorSequenceKeypoint.new(0.3, CONFIG.COLORS.glow), ColorSequenceKeypoint.new(1, CONFIG.COLORS.primary)})
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
    core.Enabled = true
    table.insert(effects, core)
    local outer = Instance.new("ParticleEmitter")
    outer.Name = "EnergyOuter"
    outer.Parent = attachment
    outer.Texture = "rbxassetid://11534281007"
    outer.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, CONFIG.COLORS.primary), ColorSequenceKeypoint.new(0.5, CONFIG.COLORS.secondary), ColorSequenceKeypoint.new(1, CONFIG.COLORS.primary)})
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
    outer.Enabled = true
    table.insert(effects, outer)
    local baseGlow = Instance.new("ParticleEmitter")
    baseGlow.Name = "BaseGlow"
    baseGlow.Parent = attachment
    baseGlow.Texture = "rbxassetid://11841348746"
    baseGlow.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, CONFIG.COLORS.bright), ColorSequenceKeypoint.new(0.5, CONFIG.COLORS.glow), ColorSequenceKeypoint.new(1, CONFIG.COLORS.primary)})
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
    baseGlow.Enabled = true
    table.insert(effects, baseGlow)
    local wisps = Instance.new("ParticleEmitter")
    wisps.Name = "EnergyWisps"
    wisps.Parent = attachment
    wisps.Texture = "rbxassetid://11841348746"
    wisps.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, CONFIG.COLORS.glow), ColorSequenceKeypoint.new(1, CONFIG.COLORS.secondary)})
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
    wisps.Enabled = true
    table.insert(effects, wisps)
    return {attachment = attachment, effects = effects}
end

function VFXManager.createAdvancedHandFire(duration, includeClones)
    pcall(function()
        local leftHand = PlayerData.character:FindFirstChild("Left Arm") or PlayerData.character:FindFirstChild("LeftHand") or PlayerData.character:FindFirstChild("LeftLowerArm")
        local rightHand = PlayerData.character:FindFirstChild("Right Arm") or PlayerData.character:FindFirstChild("RightHand") or PlayerData.character:FindFirstChild("RightLowerArm")
        local effectGroups = {}
        
        if leftHand then
            effectGroups.left = VFXManager.createAdvancedEnergyEffect(leftHand, 0.8)
        end
        if rightHand then
            effectGroups.right = VFXManager.createAdvancedEnergyEffect(rightHand, 0.8)
        end
        
        if includeClones then
            task.spawn(function()
                local barrageBind = PlayerData.character:WaitForChild("Barrage", 5)
                if barrageBind then
                    local thrown = workspace:WaitForChild("Thrown", 5)
                    if thrown then
                        for attempt = 1, 30 do
                            for _, modelo in ipairs(thrown:GetChildren()) do
                                if modelo:IsA("Model") and modelo.Name == "Model" then
                                    local leftArm = modelo:FindFirstChild("Left Arm")
                                    local rightArm = modelo:FindFirstChild("Right Arm")
                                    
                                    if leftArm and leftArm:IsA("BasePart") then
                                        local key = string.format("thrown_left_%s_%d", modelo:GetDebugId(), tick())
                                        if not effectGroups[key] then
                                            effectGroups[key] = VFXManager.createAdvancedEnergyEffect(leftArm, 0.8)
                                        end
                                    end
                                    
                                    if rightArm and rightArm:IsA("BasePart") then
                                        local key = string.format("thrown_right_%s_%d", modelo:GetDebugId(), tick())
                                        if not effectGroups[key] then
                                            effectGroups[key] = VFXManager.createAdvancedEnergyEffect(rightArm, 0.8)
                                        end
                                    end
                                end
                            end
                            task.wait(0.05)
                        end
                    end
                end
            end)
            
            local repStorage = Services.ReplicatedStorage
            local resources = repStorage:FindFirstChild("Resources")
            if resources then
                local finisher = resources:FindFirstChild("Consecutive Punches Finisher")
                if finisher then
                    local handClones = finisher:FindFirstChild("HandClones")
                    if handClones then
                        for i = 1, 5 do
                            local cloneName = string.format("CLONE%02d", i)
                            local clone = handClones:FindFirstChild(cloneName)
                            if clone then
                                local cloneLeftArm = clone:FindFirstChild("Left Arm")
                                local cloneRightArm = clone:FindFirstChild("Right Arm")
                                
                                if cloneLeftArm then
                                    local key = "clone" .. i .. "_left"
                                    effectGroups[key] = VFXManager.createAdvancedEnergyEffect(cloneLeftArm, 0.8)
                                end
                                if cloneRightArm then
                                    local key = "clone" .. i .. "_right"
                                    effectGroups[key] = VFXManager.createAdvancedEnergyEffect(cloneRightArm, 0.8)
                                end
                            end
                        end
                    end
                end
            end
        end
        
        task.wait(duration)
        for _, group in pairs(effectGroups) do
            if group then
                for _, effect in pairs(group.effects) do
                    if effect then effect:Destroy() end
                end
                if group.attachment then group.attachment:Destroy() end
            end
        end
    end)
end
function VFXManager.createCustomBlackFlash()
    pcall(function()
        local hrp = PlayerData.character:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        
        local vfxSource = Services.ReplicatedStorage:FindFirstChild("Emotes")
        if not vfxSource then return end
        vfxSource = vfxSource:FindFirstChild("VFX")
        if not vfxSource then return end
        vfxSource = vfxSource:FindFirstChild("VfxMods")
        if not vfxSource then return end
        vfxSource = vfxSource:FindFirstChild("Flasher")
        if not vfxSource then return end
        vfxSource = vfxSource:FindFirstChild("vfx")
        if not vfxSource then return end
        vfxSource = vfxSource:FindFirstChild("LastImpactFx")
        if not vfxSource then return end
        vfxSource = vfxSource:FindFirstChild("Attachment")
        if not vfxSource then return end
        
        local vfxClone = vfxSource:Clone()
        vfxClone.Parent = hrp
        
        if vfxClone:IsA("Attachment") then
            vfxClone.Position = Vector3.new(0, 1, -2)
            vfxClone.Orientation = Vector3.new(0, 0, 0)
        end
        
        for _, child in ipairs(vfxClone:GetChildren()) do
            if child:IsA("ParticleEmitter") and (child.Name == "Lightning" or child.Name == "Glow") then
                local originalSpeed = child.Speed
                local originalRate = child.Rate
                
                child.Speed = NumberRange.new(originalSpeed.Min * 6, originalSpeed.Max * 6)
                child.Lifetime = NumberRange.new(0.08, 0.2)
                child.Rate = originalRate * 3
                
                if child.Size then
                    local keypoints = child.Size.Keypoints
                    local newKeypoints = {}
                    for i, kp in ipairs(keypoints) do
                        table.insert(newKeypoints, NumberSequenceKeypoint.new(kp.Time, kp.Value * 1.5, kp.Envelope))
                    end
                    child.Size = NumberSequence.new(newKeypoints)
                end
                
                child.Transparency = NumberSequence.new({
                    NumberSequenceKeypoint.new(0, 0),
                    NumberSequenceKeypoint.new(0.7, 0.8),
                    NumberSequenceKeypoint.new(1, 1)
                })
                
                child:Emit(60)
                child.Enabled = true
                task.wait(0.02)
                child.Enabled = false
            end
        end
        
        task.spawn(function()
            local camera = workspace.CurrentCamera
            if camera then
                local originalCFrame = camera.CFrame
                for i = 1, 5 do
                    local shake = CFrame.new(
                        math.random(-20, 20) / 100,
                        math.random(-20, 20) / 100,
                        math.random(-20, 20) / 100
                    )
                    camera.CFrame = originalCFrame * shake
                    task.wait(0.02)
                end
                camera.CFrame = originalCFrame
            end
        end)
        
        Services.Debris:AddItem(vfxClone, 0.35)
    end)
end

function VFXManager.trigger(vfxType, duration)
    pcall(function()
        local hrp = PlayerData.character:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        if vfxType == "blackFlash" then
            VFXManager.createCustomBlackFlash()
        end
    end)
end

local CameraManager = {}

function CameraManager.applyFOVEffect()
    pcall(function()
        local camera = workspace.CurrentCamera
        if not camera then return end
        local originalFOV = camera.FieldOfView
        local fovIncrease = Services.TweenService:Create(camera, TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {FieldOfView = originalFOV + 20})
        fovIncrease:Play()
        task.wait(0.3)
        local fovDecrease = Services.TweenService:Create(camera, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {FieldOfView = originalFOV})
        fovDecrease:Play()
    end)
end

local LightingManager = {}

function LightingManager.applyRedLight()
    pcall(function()
        local originalAmbient = Services.Lighting.Ambient
        local originalBrightness = Services.Lighting.Brightness
        local lightTween = Services.TweenService:Create(Services.Lighting, TweenInfo.new(0.5, Enum.EasingStyle.Cubic, Enum.EasingDirection.InOut), {Ambient = Color3.new(0.4, 0.05, 0.05), Brightness = 1})
        lightTween:Play()
        task.wait(0.5)
        local lightReturn = Services.TweenService:Create(Services.Lighting, TweenInfo.new(0.5, Enum.EasingStyle.Cubic, Enum.EasingDirection.InOut), {Ambient = originalAmbient, Brightness = originalBrightness})
        lightReturn:Play()
    end)
end

local SoundManager = {}

function SoundManager.playSound(soundId, volume)
    pcall(function()
        local hrp = PlayerData.character:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        local sound = Instance.new("Sound")
        sound.SoundId = "rbxassetid://" .. soundId
        sound.Volume = volume or 1
        sound.Parent = hrp
        sound:Play()
        Services.Debris:AddItem(sound, 5)
    end)
end

local AnimationManager = {}

function AnimationManager.stopAnimation(animationId)
    pcall(function()
        if not PlayerData.humanoid then return end
        for _, track in ipairs(PlayerData.humanoid:GetPlayingAnimationTracks()) do
            local trackId = tonumber(track.Animation.AnimationId:match("%d+"))
            if trackId == animationId then
                track:Stop()
            end
        end
    end)
end

function AnimationManager.playAnimation(animationId, speed, timePos)
    local success, track = pcall(function()
        if not PlayerData.humanoid then return nil end
        local animation = Instance.new("Animation")
        animation.AnimationId = "rbxassetid://" .. animationId
        local animTrack = PlayerData.humanoid:LoadAnimation(animation)
        animTrack:Play()
        animTrack:AdjustSpeed(0)
        animTrack.TimePosition = timePos or 0
        animTrack:AdjustSpeed(speed or 1)
        return animTrack
    end)
    if not success then return nil end
    return track
end

function AnimationManager.setupReplacement(originalId, config)
    pcall(function()
        if not PlayerData.humanoid then return end
        PlayerData.humanoid.AnimationPlayed:Connect(function(animationTrack)
            local trackId = tonumber(animationTrack.Animation.AnimationId:match("%d+"))
            if trackId ~= originalId then return end
            task.wait()
            AnimationManager.stopAnimation(originalId)
            AnimationManager.playAnimation(config.animationId, config.speed, config.timePos)
            if config.soundId then
                SoundManager.playSound(config.soundId, 1)
            end
            if config.useAdvancedHandFire then
                task.spawn(function()
                    VFXManager.createAdvancedHandFire(config.handFireDuration or 2, config.includeClones or false)
                end)
            end
            if config.useCustomVFX then
                task.spawn(function()
                    task.wait(config.vfxDelay or 0.2)
                    VFXManager.trigger("blackFlash", 1)
                end)
            end
            if config.useFOV then
                task.spawn(CameraManager.applyFOVEffect)
            end
            if config.useRedLight then
                task.spawn(LightingManager.applyRedLight)
            end
            if config.useBlackFlashText then
                task.spawn(NotificationManager.createBlackFlash)
            end
        end)
    end)
end

function AnimationManager.playFullReplacement(originalId, config)
    if PlayerData.isAnimating then
        table.insert(PlayerData.animationQueue, {originalId, config})
        return
    end
    pcall(function()
        PlayerData.isAnimating = true
        local animation = Instance.new("Animation")
        animation.AnimationId = "rbxassetid://" .. config.animationId
        local animTrack = PlayerData.humanoid:LoadAnimation(animation)
        animTrack:Play()
        animTrack:AdjustSpeed(config.speed or 1)
        animTrack.Stopped:Connect(function()
            PlayerData.isAnimating = false
            if #PlayerData.animationQueue > 0 then
                local next = table.remove(PlayerData.animationQueue, 1)
                AnimationManager.playFullReplacement(next[1], next[2])
            end
        end)
    end)
end

function AnimationManager.setupFullReplacement(originalId, config)
    pcall(function()
        if not PlayerData.humanoid then return end
        PlayerData.humanoid.AnimationPlayed:Connect(function(animationTrack)
            local animId = tonumber(animationTrack.Animation.AnimationId:match("%d+"))
            if animId ~= originalId then return end
            AnimationManager.stopAnimation(originalId)
            animationTrack:Stop()
            AnimationManager.playFullReplacement(originalId, config)
        end)
    end)
end

local PhysicsManager = {}

function PhysicsManager.onBodyVelocityAdded(bodyVelocity)
    if not bodyVelocity:IsA("BodyVelocity") then return end
    pcall(function()
        bodyVelocity.Velocity = Vector3.new(bodyVelocity.Velocity.X, 0, bodyVelocity.Velocity.Z)
    end)
end

function PhysicsManager.setupVelocityHandler()
    pcall(function()
        if not PlayerData.character then return end
        PlayerData.character.DescendantAdded:Connect(PhysicsManager.onBodyVelocityAdded)
        for _, descendant in pairs(PlayerData.character:GetDescendants()) do
            PhysicsManager.onBodyVelocityAdded(descendant)
        end
    end)
end

local CharacterManager = {}

function CharacterManager.initialize()
    pcall(function()
        for originalId, config in pairs(CONFIG.ANIMATION_REPLACEMENTS) do
            AnimationManager.setupReplacement(originalId, config)
        end
        for originalId, config in pairs(CONFIG.FULL_REPLACEMENT_ANIMATIONS) do
            AnimationManager.setupFullReplacement(originalId, config)
        end
        task.spawn(UIManager.updateAttackNames)
        PhysicsManager.setupVelocityHandler()
    end)
end

function CharacterManager.onCharacterAdded(newCharacter)
    pcall(function()
        PlayerData.character = newCharacter
        PlayerData.humanoid = newCharacter:WaitForChild("Humanoid")
        task.wait(1)
        CharacterManager.initialize()
    end)
end

local SystemManager = {}

function SystemManager.start()
    pcall(function()
        initializePlayerData()
        task.wait(1)
        CharacterManager.initialize()
        PlayerData.player.CharacterAdded:Connect(CharacterManager.onCharacterAdded)
    end)
end

SystemManager.start()
