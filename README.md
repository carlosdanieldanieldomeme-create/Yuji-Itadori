local Services = {
    TweenService = game:GetService("TweenService"),
    Debris = game:GetService("Debris"),
    Lighting = game:GetService("Lighting")
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
            useFOV = false,
            useRedLight = false,
            useBlackFlashText = false
        },
        [10471336737] = {
            skillName = "SHOVE",
            animationId = 18179181663,
            speed = 1,
            timePos = 0
        },
        [10470104242] = {
            skillName = "UPPERCUT",
            animationId = 17858997926,
            speed = 1.1,
            timePos = 0
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
        ["Consecutive Punches"] = "Divergent Dam Combo",
        ["Shove"] = "Black Flash is expelled",
        ["Uppercut"] = "Divergent Punch"
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
    local success = pcall(function()
        local kj = PlayerData.playerGui.Hotbar.Backpack.LocalScript:FindFirstChild("Flipbook")
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

function VFXManager.createBlackFlash(parent, duration)
    pcall(function()
        local attachment = Instance.new("Attachment")
        attachment.Name = "BlackFlashAttachment"
        attachment.Parent = parent
        local explosion = Instance.new("ParticleEmitter")
        explosion.Name = "BlackFlashExplosion"
        explosion.Parent = attachment
        explosion.Texture = "rbxassetid://11534281007"
        explosion.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)), ColorSequenceKeypoint.new(0.3, Color3.fromRGB(255, 50, 50)), ColorSequenceKeypoint.new(1, Color3.fromRGB(100, 0, 0))})
        explosion.Size = NumberSequence.new({NumberSequenceKeypoint.new(0, 0), NumberSequenceKeypoint.new(0.1, 4), NumberSequenceKeypoint.new(0.5, 3), NumberSequenceKeypoint.new(1, 0)})
        explosion.Transparency = NumberSequence.new({NumberSequenceKeypoint.new(0, 0), NumberSequenceKeypoint.new(0.5, 0.5), NumberSequenceKeypoint.new(1, 1)})
        explosion.Lifetime = NumberRange.new(0.4, 0.6)
        explosion.Rate = 200
        explosion.Speed = NumberRange.new(8, 15)
        explosion.SpreadAngle = Vector2.new(180, 180)
        explosion.Rotation = NumberRange.new(0, 360)
        explosion.RotSpeed = NumberRange.new(-200, 200)
        explosion.LightEmission = 1
        explosion.ZOffset = 0.5
        explosion.Enabled = true
        local sparks = Instance.new("ParticleEmitter")
        sparks.Name = "BlackFlashSparks"
        sparks.Parent = attachment
        sparks.Texture = "rbxassetid://11841348746"
        sparks.Color = ColorSequence.new(Color3.fromRGB(255, 100, 100))
        sparks.Size = NumberSequence.new({NumberSequenceKeypoint.new(0, 0.3), NumberSequenceKeypoint.new(1, 0.1)})
        sparks.Transparency = NumberSequence.new({NumberSequenceKeypoint.new(0, 0.2), NumberSequenceKeypoint.new(1, 1)})
        sparks.Lifetime = NumberRange.new(0.3, 0.5)
        sparks.Rate = 150
        sparks.Speed = NumberRange.new(15, 25)
        sparks.SpreadAngle = Vector2.new(90, 90)
        sparks.LightEmission = 1
        sparks.Enabled = true
        task.wait(duration)
        attachment:Destroy()
    end)
end

function VFXManager.trigger(vfxType, duration)
    pcall(function()
        local hrp = PlayerData.character:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        if vfxType == "blackFlash" then
            VFXManager.createBlackFlash(hrp, duration)
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
            if config.useAttackVFX and config.vfxType then
                task.spawn(function()
                    task.wait(config.vfxDelay or 0.2)
                    VFXManager.trigger(config.vfxType, config.vfxDuration or 1)
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
