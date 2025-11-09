local Services = {
    TweenService = game:GetService("TweenService"),
    Debris = game:GetService("Debris"),
    Lighting = game:GetService("Lighting"),
    ReplicatedStorage = game:GetService("ReplicatedStorage"),
    RunService = game:GetService("RunService")
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
            blackFlashDelay = 0.6,
            useCustomVFX = true,
            vfxDelay = 0.6
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
    },
    
    UPDATE_INTERVAL = 0.2,
    MAX_PARTICLE_RATE = 60,
    PARTICLE_SCALE = 0.7
}

local PlayerData = {
    player = game.Players.LocalPlayer,
    character = nil,
    humanoid = nil,
    playerGui = nil,
    animationQueue = {},
    isAnimating = false,
    connections = {},
    activeEffects = {},
    isAlive = false,
    cachedParts = {},
    lastUpdate = 0
}

local function safeCall(func, ...)
    local success, result = pcall(func, ...)
    if not success then
        return false, result
    end
    return true, result
end

local function cleanupConnections()
    for i = #PlayerData.connections, 1, -1 do
        local connection = PlayerData.connections[i]
        if connection and typeof(connection) == "RBXScriptConnection" then
            connection:Disconnect()
        end
        PlayerData.connections[i] = nil
    end
end

local function cleanupEffects()
    for i = #PlayerData.activeEffects, 1, -1 do
        local effect = PlayerData.activeEffects[i]
        if effect and typeof(effect) == "Instance" and effect.Parent then
            effect:Destroy()
        end
        PlayerData.activeEffects[i] = nil
    end
end

local function validateCharacter()
    return PlayerData.character 
        and PlayerData.character.Parent 
        and PlayerData.humanoid 
        and PlayerData.humanoid.Health > 0
end

local function initializePlayerData()
    local success = safeCall(function()
        PlayerData.playerGui = PlayerData.player:WaitForChild("PlayerGui", 5)
        PlayerData.character = PlayerData.player.Character or PlayerData.player.CharacterAdded:Wait()
        PlayerData.humanoid = PlayerData.character:WaitForChild("Humanoid", 5)
        PlayerData.isAlive = PlayerData.humanoid and PlayerData.humanoid.Health > 0
        PlayerData.cachedParts = {}
    end)
    return success
end

local function getCachedPart(name)
    if PlayerData.cachedParts[name] and PlayerData.cachedParts[name].Parent then
        return PlayerData.cachedParts[name]
    end
    
    if not validateCharacter() then return nil end
    
    local part = PlayerData.character:FindFirstChild(name)
    if part then
        PlayerData.cachedParts[name] = part
    end
    return part
end

local FontManager = {}

function FontManager.applyFont(label)
    if not CONFIG.FONT.Enabled or not label or not label:IsA("TextLabel") then
        return
    end
    safeCall(function()
        label.FontFace = Font.new(label.FontFace.Family, CONFIG.FONT.Weight, CONFIG.FONT.Style)
        label.Font = CONFIG.FONT.Font
    end)
end

local CursedFireManager = {}

function CursedFireManager.remove(buttonBase)
    if not buttonBase then return end
    safeCall(function()
        local group = buttonBase:FindFirstChild("CursedFireEffectGroup")
        if group then 
            group:Destroy()
        end
    end)
end

function CursedFireManager.add(buttonBase)
    if not buttonBase or not validateCharacter() then return end
    
    CursedFireManager.remove(buttonBase)
    
    safeCall(function()
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
        
        if clone:FindFirstChild("LocalScript") then
            clone.LocalScript.Enabled = true
        end
        
        for _, desc in pairs(clone:GetDescendants()) do
            if desc:IsA("ParticleEmitter") then
                desc.Color = ColorSequence.new({
                    ColorSequenceKeypoint.new(0, CONFIG.COLORS.redDark),
                    ColorSequenceKeypoint.new(0.5, CONFIG.COLORS.redMid),
                    ColorSequenceKeypoint.new(1, CONFIG.COLORS.redLight)
                })
            end
        end
        
        table.insert(PlayerData.activeEffects, group)
    end)
end

local UIManager = {}
local cachedButtons = {}

function UIManager.getHotbarButton(index)
    if not validateCharacter() then return nil end
    
    if cachedButtons[index] and cachedButtons[index].Parent then
        return cachedButtons[index]
    end
    
    local success, result = safeCall(function()
        local hotbar = PlayerData.playerGui:FindFirstChild("Hotbar")
        if not hotbar then return nil end
        
        local backpack = hotbar:FindFirstChild("Backpack")
        if not backpack then return nil end
        
        local hotbarFrame = backpack:FindFirstChild("Hotbar")
        if not hotbarFrame then return nil end
        
        local button = hotbarFrame:FindFirstChild(tostring(index))
        if not button then return nil end
        
        local base = button:FindFirstChild("Base")
        if base then
            cachedButtons[index] = base
        end
        return base
    end)
    
    return success and result or nil
end

function UIManager.updateAttackNames()
    local updateConnection
    updateConnection = Services.RunService.Heartbeat:Connect(function()
        if not validateCharacter() then
            if updateConnection then
                updateConnection:Disconnect()
            end
            return
        end
        
        local currentTime = tick()
        if currentTime - PlayerData.lastUpdate < CONFIG.UPDATE_INTERVAL then
            return
        end
        PlayerData.lastUpdate = currentTime
        
        safeCall(function()
            for i = 1, 4 do
                local baseButton = UIManager.getHotbarButton(i)
                if baseButton then
                    local toolName = baseButton:FindFirstChild("ToolName")
                    if toolName and toolName:IsA("TextLabel") then
                        local currentText = toolName.Text
                        
                        local newName = CONFIG.ATTACK_NAMES[currentText] or CONFIG.ULTIMATE_NAMES[currentText]
                        if newName then
                            toolName.Text = newName
                            FontManager.applyFont(toolName)
                        end
                        
                        if currentText == "Do not change here" then
                            CursedFireManager.add(baseButton)
                        else
                            CursedFireManager.remove(baseButton)
                        end
                    end
                end
            end
        end)
    end)
    
    table.insert(PlayerData.connections, updateConnection)
end

local NotificationManager = {}

function NotificationManager.createBlackFlash()
    if not validateCharacter() then return end
    
    task.spawn(function()
        safeCall(function()
            local hrp = getCachedPart("HumanoidRootPart")
            if not hrp then return end
            
            local sound = Instance.new("Sound")
            sound.SoundId = "rbxassetid://9086333748"
            sound.Volume = 0.5
            sound.Parent = hrp
            sound:Play()
            Services.Debris:AddItem(sound, 2)
            
            local billboard = Instance.new("BillboardGui")
            billboard.Name = "BlackFlashBillboard"
            billboard.Size = UDim2.new(0, 0, 0, 0)
            billboard.StudsOffset = Vector3.new(-2, 4.1, 0)
            billboard.AlwaysOnTop = true
            billboard.Parent = hrp
            
            local balloon = Instance.new("ImageLabel")
            balloon.Name = "Balloon"
            balloon.Image = "rbxassetid://136797177442983"
            balloon.Size = UDim2.new(1, 0, 1, 0)
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
            
            local tweenIn = Services.TweenService:Create(
                billboard, 
                TweenInfo.new(0.15, Enum.EasingStyle.Back, Enum.EasingDirection.Out), 
                {Size = UDim2.new(3.9, 0, 4, 0)}
            )
            tweenIn:Play()
            
            VFXManager.createCustomBlackFlash()
            
            task.wait(1.8)
            
            if not billboard.Parent then return end
            
            local tweenOut = Services.TweenService:Create(
                billboard, 
                TweenInfo.new(0.15, Enum.EasingStyle.Back, Enum.EasingDirection.In), 
                {Size = UDim2.new(0, 0, 0, 0)}
            )
            tweenOut:Play()
            Services.Debris:AddItem(billboard, 0.2)
        end)
    end)
end

local VFXManager = {}

function VFXManager.createOptimizedParticle(parent, config)
    local emitter = Instance.new("ParticleEmitter")
    emitter.Parent = parent
    
    for property, value in pairs(config) do
        emitter[property] = value
    end
    
    return emitter
end

function VFXManager.createAdvancedEnergyEffect(hand, scale)
    if not hand or not hand:IsDescendantOf(workspace) then return nil end
    
    local success, result = safeCall(function()
        local attachment = Instance.new("Attachment")
        attachment.Name = "EnergyAttachment"
        attachment.Position = Vector3.new(0, -0.5, 0)
        attachment.Parent = hand
        
        local effects = {}
        scale = scale * CONFIG.PARTICLE_SCALE
        local rateMultiplier = CONFIG.MAX_PARTICLE_RATE / 80
        
        local darkFire = VFXManager.createOptimizedParticle(attachment, {
            Name = "DarkFire",
            Texture = "rbxassetid://11534281007",
            Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0, CONFIG.COLORS.dark), 
                ColorSequenceKeypoint.new(0.7, CONFIG.COLORS.dark), 
                ColorSequenceKeypoint.new(1, CONFIG.COLORS.primary)
            }),
            Size = NumberSequence.new({
                NumberSequenceKeypoint.new(0, 1 * scale), 
                NumberSequenceKeypoint.new(0.3, 2.5 * scale), 
                NumberSequenceKeypoint.new(1, 0)
            }),
            Transparency = NumberSequence.new({
                NumberSequenceKeypoint.new(0, 0.3), 
                NumberSequenceKeypoint.new(0.5, 0.6), 
                NumberSequenceKeypoint.new(1, 1)
            }),
            Lifetime = NumberRange.new(0.4, 0.8),
            Rate = 60 * rateMultiplier,
            Speed = NumberRange.new(1, 2),
            SpreadAngle = Vector2.new(25, 25),
            Rotation = NumberRange.new(0, 360),
            RotSpeed = NumberRange.new(-80, 80),
            LightEmission = 0,
            Acceleration = Vector3.new(0, 2, 0),
            Drag = 3,
            LockedToPart = true,
            EmissionDirection = Enum.NormalId.Top,
            FlipbookLayout = Enum.ParticleFlipbookLayout.Grid4x4,
            FlipbookMode = Enum.ParticleFlipbookMode.OneShot,
            Enabled = true
        })
        table.insert(effects, darkFire)
        
        local core = VFXManager.createOptimizedParticle(attachment, {
            Name = "EnergyCore",
            Texture = "rbxassetid://11534281007",
            Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0, CONFIG.COLORS.bright), 
                ColorSequenceKeypoint.new(0.5, CONFIG.COLORS.glow), 
                ColorSequenceKeypoint.new(1, CONFIG.COLORS.primary)
            }),
            Size = NumberSequence.new({
                NumberSequenceKeypoint.new(0, 0.5 * scale), 
                NumberSequenceKeypoint.new(0.3, 1.5 * scale), 
                NumberSequenceKeypoint.new(1, 0)
            }),
            Transparency = NumberSequence.new({
                NumberSequenceKeypoint.new(0, 0), 
                NumberSequenceKeypoint.new(0.5, 0.4), 
                NumberSequenceKeypoint.new(1, 1)
            }),
            Lifetime = NumberRange.new(0.3, 0.6),
            Rate = 60 * rateMultiplier,
            Speed = NumberRange.new(2, 3.5),
            SpreadAngle = Vector2.new(15, 15),
            Rotation = NumberRange.new(0, 360),
            RotSpeed = NumberRange.new(-80, 80),
            LightEmission = 1,
            LightInfluence = 0,
            Acceleration = Vector3.new(0, 3, 0),
            Drag = 2.5,
            LockedToPart = true,
            EmissionDirection = Enum.NormalId.Top,
            FlipbookLayout = Enum.ParticleFlipbookLayout.Grid4x4,
            FlipbookMode = Enum.ParticleFlipbookMode.OneShot,
            Enabled = true
        })
        table.insert(effects, core)
        
        local outer = VFXManager.createOptimizedParticle(attachment, {
            Name = "EnergyOuter",
            Texture = "rbxassetid://11534281007",
            Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0, CONFIG.COLORS.primary), 
                ColorSequenceKeypoint.new(0.5, CONFIG.COLORS.secondary), 
                ColorSequenceKeypoint.new(1, CONFIG.COLORS.primary)
            }),
            Size = NumberSequence.new({
                NumberSequenceKeypoint.new(0, 0.8 * scale), 
                NumberSequenceKeypoint.new(0.4, 1.8 * scale), 
                NumberSequenceKeypoint.new(1, 0)
            }),
            Transparency = NumberSequence.new({
                NumberSequenceKeypoint.new(0, 0.2), 
                NumberSequenceKeypoint.new(0.5, 0.5), 
                NumberSequenceKeypoint.new(1, 1)
            }),
            Lifetime = NumberRange.new(0.4, 0.7),
            Rate = 50 * rateMultiplier,
            Speed = NumberRange.new(1.5, 2.5),
            SpreadAngle = Vector2.new(20, 20),
            Rotation = NumberRange.new(0, 360),
            RotSpeed = NumberRange.new(-100, 100),
            LightEmission = 0.9,
            Acceleration = Vector3.new(0, 2.5, 0),
            Drag = 2,
            LockedToPart = true,
            EmissionDirection = Enum.NormalId.Top,
            FlipbookLayout = Enum.ParticleFlipbookLayout.Grid4x4,
            FlipbookMode = Enum.ParticleFlipbookMode.OneShot,
            Enabled = true
        })
        table.insert(effects, outer)
        
        return {attachment = attachment, effects = effects}
    end)
    
    if success and result then
        table.insert(PlayerData.activeEffects, result.attachment)
        return result
    end
    return nil
end

function VFXManager.createAdvancedHandFire(duration, includeClones)
    if not validateCharacter() then return end
    
    task.spawn(function()
        safeCall(function()
            local leftHand = getCachedPart("Left Arm") 
                or getCachedPart("LeftHand") 
                or getCachedPart("LeftLowerArm")
            local rightHand = getCachedPart("Right Arm") 
                or getCachedPart("RightHand") 
                or getCachedPart("RightLowerArm")
            
            local effectGroups = {}
            
            if leftHand then
                effectGroups.left = VFXManager.createAdvancedEnergyEffect(leftHand, 0.8)
            end
            if rightHand then
                effectGroups.right = VFXManager.createAdvancedEnergyEffect(rightHand, 0.8)
            end
            
            if includeClones then
                task.spawn(function()
                    local thrown = workspace:FindFirstChild("Thrown")
                    if thrown then
                        local processedModels = {}
                        for attempt = 1, 30 do
                            if not validateCharacter() then break end
                            
                            for _, modelo in ipairs(thrown:GetChildren()) do
                                if modelo:IsA("Model") and modelo.Name == "Model" and not processedModels[modelo] then
                                    processedModels[modelo] = true
                                    local leftArm = modelo:FindFirstChild("Left Arm")
                                    local rightArm = modelo:FindFirstChild("Right Arm")
                                    
                                    if leftArm and leftArm:IsA("BasePart") then
                                        local key = "thrown_left_" .. tostring(modelo)
                                        if not effectGroups[key] then
                                            effectGroups[key] = VFXManager.createAdvancedEnergyEffect(leftArm, 0.8)
                                        end
                                    end
                                    
                                    if rightArm and rightArm:IsA("BasePart") then
                                        local key = "thrown_right_" .. tostring(modelo)
                                        if not effectGroups[key] then
                                            effectGroups[key] = VFXManager.createAdvancedEnergyEffect(rightArm, 0.8)
                                        end
                                    end
                                end
                            end
                            task.wait(0.1)
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
                                if not validateCharacter() then break end
                                
                                local cloneName = string.format("CLONE%02d", i)
                                local clone = handClones:FindFirstChild(cloneName)
                                if clone then
                                    local cloneLeftArm = clone:FindFirstChild("Left Arm")
                                    local cloneRightArm = clone:FindFirstChild("Right Arm")
                                    
                                    if cloneLeftArm then
                                        effectGroups["clone" .. i .. "_left"] = VFXManager.createAdvancedEnergyEffect(cloneLeftArm, 0.8)
                                    end
                                    if cloneRightArm then
                                        effectGroups["clone" .. i .. "_right"] = VFXManager.createAdvancedEnergyEffect(cloneRightArm, 0.8)
                                    end
                                end
                            end
                        end
                    end
                end
            end
            
            task.wait(duration)
            
            for _, group in pairs(effectGroups) do
                if group and group.effects then
                    for _, effect in pairs(group.effects) do
                        if effect and effect.Parent then
                            effect:Destroy()
                        end
                    end
                end
                if group and group.attachment and group.attachment.Parent then
                    group.attachment:Destroy()
                end
            end
        end)
    end)
end

function VFXManager.createCustomBlackFlash()
    if not validateCharacter() then return end
    
    safeCall(function()
        local hrp = getCachedPart("HumanoidRootPart")
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
                
                child.Speed = NumberRange.new(originalSpeed.Min * 5, originalSpeed.Max * 5)
                child.Lifetime = NumberRange.new(0.08, 0.15)
                child.Rate = math.min(originalRate * 2.5, 100)
                
                if child.Size then
                    local keypoints = child.Size.Keypoints
                    local newKeypoints = {}
                    for i, kp in ipairs(keypoints) do
                        table.insert(newKeypoints, NumberSequenceKeypoint.new(kp.Time, kp.Value * 1.3, kp.Envelope))
                    end
                    child.Size = NumberSequence.new(newKeypoints)
                end
                
                child.Transparency = NumberSequence.new({
                    NumberSequenceKeypoint.new(0, 0),
                    NumberSequenceKeypoint.new(0.7, 0.8),
                    NumberSequenceKeypoint.new(1, 1)
                })
                
                child:Emit(40)
                child.Enabled = true
                task.wait(0.02)
                child.Enabled = false
            end
        end
        
        task.spawn(function()
            local camera = workspace.CurrentCamera
            if camera then
                local originalCFrame = camera.CFrame
                for i = 1, 3 do
                    if not validateCharacter() then break end
                    
                    local shake = CFrame.new(
                        math.random(-15, 15) / 100,
                        math.random(-15, 15) / 100,
                        math.random(-15, 15) / 100
                    )
                    camera.CFrame = originalCFrame * shake
                    task.wait(0.02)
                end
                camera.CFrame = originalCFrame
            end
        end)
        
        Services.Debris:AddItem(vfxClone, 0.3)
    end)
end

local CameraManager = {}

function CameraManager.applyFOVEffect()
    if not validateCharacter() then return end
    
    task.spawn(function()
        safeCall(function()
            local camera = workspace.CurrentCamera
            if not camera then return end
            
            local originalFOV = camera.FieldOfView
            
            local fovIncrease = Services.TweenService:Create(
                camera, 
                TweenInfo.new(0.08, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), 
                {FieldOfView = originalFOV + 18}
            )
            fovIncrease:Play()
            
            task.wait(0.25)
            
            if not validateCharacter() then
                camera.FieldOfView = originalFOV
                return
            end
            
            local fovDecrease = Services.TweenService:Create(
                camera, 
                TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.In), 
                {FieldOfView = originalFOV}
            )
            fovDecrease:Play()
        end)
    end)
end

local LightingManager = {}

function LightingManager.applyRedLight()
    if not validateCharacter() then return end
    
    task.spawn(function()
        safeCall(function()
            local originalAmbient = Services.Lighting.Ambient
            local originalBrightness = Services.Lighting.Brightness
            
            local lightTween = Services.TweenService:Create(
                Services.Lighting, 
                TweenInfo.new(0.4, Enum.EasingStyle.Cubic, Enum.EasingDirection.InOut), 
                {Ambient = Color3.new(0.35, 0.05, 0.05), Brightness = 1}
            )
            lightTween:Play()
            
            task.wait(0.4)
            
            if not validateCharacter() then
                Services.Lighting.Ambient = originalAmbient
                Services.Lighting.Brightness = originalBrightness
                return
            end
            
            local lightReturn = Services.TweenService:Create(
                Services.Lighting, 
                TweenInfo.new(0.4, Enum.EasingStyle.Cubic, Enum.EasingDirection.InOut), 
                {Ambient = originalAmbient, Brightness = originalBrightness}
            )
            lightReturn:Play()
        end)
    end)
end

local SoundManager = {}

function SoundManager.playSound(soundId, volume)
    if not validateCharacter() then return end
    
    task.spawn(function()
        safeCall(function()
            local hrp = getCachedPart("HumanoidRootPart")
            if not hrp then return end
            
            local sound = Instance.new("Sound")
            sound.SoundId = "rbxassetid://" .. soundId
            sound.Volume = volume or 1
            sound.Parent = hrp
            sound:Play()
            Services.Debris:AddItem(sound, 4)
        end)
    end)
end

local AnimationManager = {}
local animationCache = {}

function AnimationManager.stopAnimation(animationId)
    if not validateCharacter() or not PlayerData.humanoid then return end
    
    safeCall(function()
        for _, track in ipairs(PlayerData.humanoid:GetPlayingAnimationTracks()) do
            if track and track.Animation then
                local trackId = tonumber(track.Animation.AnimationId:match("%d+"))
                if trackId == animationId then
                    track:Stop(0)
                end
            end
        end
    end)
end

function AnimationManager.playAnimation(animationId, speed, timePos)
    if not validateCharacter() or not PlayerData.humanoid then return nil end
    
    local success, track = safeCall(function()
        local animation = animationCache[animationId]
        if not animation then
            animation = Instance.new("Animation")
            animation.AnimationId = "rbxassetid://" .. animationId
            animationCache[animationId] = animation
        end
        
        local animTrack = PlayerData.humanoid:LoadAnimation(animation)
        animTrack:Play(0, 1, speed or 1)
        animTrack.TimePosition = timePos or 0
        
        return animTrack
    end)
    
    return success and track or nil
end

function AnimationManager.setupReplacement(originalId, config)
    if not validateCharacter() or not PlayerData.humanoid then return end
    
    safeCall(function()
        local connection = PlayerData.humanoid.AnimationPlayed:Connect(function(animationTrack)
            if not validateCharacter() then return end
            if not animationTrack or not animationTrack.Animation then return end
            
            local trackId = tonumber(animationTrack.Animation.AnimationId:match("%d+"))
            if trackId ~= originalId then return end
            
            task.wait()
            
            AnimationManager.stopAnimation(originalId)
            AnimationManager.playAnimation(config.animationId, config.speed, config.timePos)
            
            if config.soundId then
                SoundManager.playSound(config.soundId, 1)
            end
            
            if config.useAdvancedHandFire then
                VFXManager.createAdvancedHandFire(
                    config.handFireDuration or 2, 
                    config.includeClones or false
                )
            end
            
            if config.useBlackFlashText then
                task.delay(config.blackFlashDelay or 0, NotificationManager.createBlackFlash)
            end
            
            if config.useCustomVFX then
                task.delay(config.vfxDelay or 0, VFXManager.createCustomBlackFlash)
            end
            
            if config.useFOV then
                CameraManager.applyFOVEffect()
            end
            
            if config.useRedLight then
                LightingManager.applyRedLight()
            end
        end)
        
        table.insert(PlayerData.connections, connection)
    end)
end

function AnimationManager.playFullReplacement(originalId, config)
    if not validateCharacter() or not PlayerData.humanoid then return end
    
    if PlayerData.isAnimating then
        table.insert(PlayerData.animationQueue, {originalId, config})
        return
    end
    
    safeCall(function()
        PlayerData.isAnimating = true
        
        local animation = animationCache[config.animationId]
        if not animation then
            animation = Instance.new("Animation")
            animation.AnimationId = "rbxassetid://" .. config.animationId
            animationCache[config.animationId] = animation
        end
        
        local animTrack = PlayerData.humanoid:LoadAnimation(animation)
        animTrack:Play(0, 1, config.speed or 1)
        
        local connection = animTrack.Stopped:Connect(function()
            PlayerData.isAnimating = false
            
            if #PlayerData.animationQueue > 0 then
                local next = table.remove(PlayerData.animationQueue, 1)
                AnimationManager.playFullReplacement(next[1], next[2])
            end
        end)
        
        table.insert(PlayerData.connections, connection)
    end)
end

function AnimationManager.setupFullReplacement(originalId, config)
    if not validateCharacter() or not PlayerData.humanoid then return end
    
    safeCall(function()
        local connection = PlayerData.humanoid.AnimationPlayed:Connect(function(animationTrack)
            if not validateCharacter() then return end
            if not animationTrack or not animationTrack.Animation then return end
            
            local animId = tonumber(animationTrack.Animation.AnimationId:match("%d+"))
            if animId ~= originalId then return end
            
            AnimationManager.stopAnimation(originalId)
            animationTrack:Stop(0)
            AnimationManager.playFullReplacement(originalId, config)
        end)
        
        table.insert(PlayerData.connections, connection)
    end)
end

local PhysicsManager = {}

function PhysicsManager.onBodyVelocityAdded(bodyVelocity)
    if not bodyVelocity or not bodyVelocity:IsA("BodyVelocity") then return end
    if not validateCharacter() then return end
    
    safeCall(function()
        bodyVelocity.Velocity = Vector3.new(
            bodyVelocity.Velocity.X, 
            0, 
            bodyVelocity.Velocity.Z
        )
    end)
end

function PhysicsManager.setupVelocityHandler()
    if not validateCharacter() or not PlayerData.character then return end
    
    safeCall(function()
        local connection = PlayerData.character.DescendantAdded:Connect(function(descendant)
            if validateCharacter() then
                PhysicsManager.onBodyVelocityAdded(descendant)
            end
        end)
        
        table.insert(PlayerData.connections, connection)
        
        for _, descendant in pairs(PlayerData.character:GetDescendants()) do
            PhysicsManager.onBodyVelocityAdded(descendant)
        end
    end)
end

local CharacterManager = {}

function CharacterManager.initialize()
    if not validateCharacter() then return end
    
    safeCall(function()
        for originalId, config in pairs(CONFIG.ANIMATION_REPLACEMENTS) do
            AnimationManager.setupReplacement(originalId, config)
        end
        
        for originalId, config in pairs(CONFIG.FULL_REPLACEMENT_ANIMATIONS) do
            AnimationManager.setupFullReplacement(originalId, config)
        end
        
        UIManager.updateAttackNames()
        PhysicsManager.setupVelocityHandler()
    end)
end

function CharacterManager.cleanup()
    PlayerData.isAlive = false
    cleanupConnections()
    cleanupEffects()
    PlayerData.animationQueue = {}
    PlayerData.isAnimating = false
    PlayerData.cachedParts = {}
    cachedButtons = {}
    animationCache = {}
    PlayerData.lastUpdate = 0
end

function CharacterManager.onCharacterAdded(newCharacter)
    CharacterManager.cleanup()
    
    task.wait(0.5)
    
    safeCall(function()
        PlayerData.character = newCharacter
        PlayerData.humanoid = newCharacter:WaitForChild("Humanoid", 5)
        
        if PlayerData.humanoid then
            PlayerData.isAlive = true
            
            local deathConnection = PlayerData.humanoid.Died:Connect(function()
                CharacterManager.cleanup()
            end)
            
            table.insert(PlayerData.connections, deathConnection)
            
            task.wait(0.5)
            CharacterManager.initialize()
        end
    end)
end

local SystemManager = {}

function SystemManager.start()
    safeCall(function()
        local success = initializePlayerData()
        if not success then return end
        
        if PlayerData.humanoid then
            local deathConnection = PlayerData.humanoid.Died:Connect(function()
                CharacterManager.cleanup()
            end)
            
            table.insert(PlayerData.connections, deathConnection)
        end
        
        task.wait(0.5)
        CharacterManager.initialize()
        
        local charAddedConnection = PlayerData.player.CharacterAdded:Connect(CharacterManager.onCharacterAdded)
        table.insert(PlayerData.connections, charAddedConnection)
    end)
end

SystemManager.start()
