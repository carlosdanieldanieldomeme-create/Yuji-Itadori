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
			skillName = "Downslam",
			animationId = 15954395010,
			speed = 1.1,
			timePos = 0
		},
		[12510170988] = {
			skillName = "Uppercut Skill",
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
			skillName = "Dash Foward",
			animationId = 132259592388175,
			speed = 1
		},
		[10503381238] = {
			skillName = "Uppercut M1",
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
	isAnimating = false,
	connections = {},
	activeEffects = {},
	isAlive = false
}

local function safeCall(func, ...)
	local success, result = pcall(func, ...)
	if not success then
		return false, result
	end
	return true, result
end

local function cleanupConnections()
	for _, connection in pairs(PlayerData.connections) do
		if connection and typeof(connection) == "RBXScriptConnection" then
			connection:Disconnect()
		end
	end
	PlayerData.connections = {}
end

local function cleanupEffects()
	for _, effect in pairs(PlayerData.activeEffects) do
		if effect and typeof(effect) == "Instance" then
			effect:Destroy()
		end
	end
	PlayerData.activeEffects = {}
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
	end)
	return success
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

function UIManager.getHotbarButton(index)
	if not validateCharacter() then return nil end

	local success, result = safeCall(function()
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
	local updateConnection
	updateConnection = Services.RunService.Heartbeat:Connect(function()
		if not validateCharacter() then
			if updateConnection then
				updateConnection:Disconnect()
			end
			return
		end

		safeCall(function()
			for i = 1, 4 do
				local baseButton = UIManager.getHotbarButton(i)
				if baseButton then
					local toolName = baseButton:FindFirstChild("ToolName")
					if toolName and toolName:IsA("TextLabel") then
						local currentText = toolName.Text

						for originalName, newName in pairs(CONFIG.ATTACK_NAMES) do
							if currentText == originalName then
								toolName.Text = newName
								FontManager.applyFont(toolName)
								break
							end
						end

						for originalName, newName in pairs(CONFIG.ULTIMATE_NAMES) do
							if currentText == originalName then
								toolName.Text = newName
								FontManager.applyFont(toolName)
								break
							end
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

		task.wait(0.1)
	end)

	table.insert(PlayerData.connections, updateConnection)
end

local NotificationManager = {}

function NotificationManager.createBlackFlash()
	if not validateCharacter() then return end

	task.spawn(function()
		safeCall(function()
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
			billboard.Size = UDim2.new(0, 0, 0, 0)
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

			local tweenIn = Services.TweenService:Create(
				billboard, 
				TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out), 
				{Size = UDim2.new(3.9, 0, 4, 0)}
			)
			tweenIn:Play()

			task.spawn(function()
				VFXManager.createCustomBlackFlash()
			end)

			tweenIn.Completed:Wait()
			task.wait(1.5)

			local tweenOut = Services.TweenService:Create(
				billboard, 
				TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.In), 
				{Size = UDim2.new(0, 0, 0, 0)}
			)
			tweenOut:Play()
			tweenOut.Completed:Wait()

			billboard:Destroy()
			table.insert(PlayerData.activeEffects, billboard)
		end)
	end)
end

local VFXManager = {}

function VFXManager.createAdvancedEnergyEffect(hand, scale)
	if not hand or not hand:IsDescendantOf(workspace) then return nil end

	local success, result = safeCall(function()
		local attachment = Instance.new("Attachment")
		attachment.Name = "EnergyAttachment"
		attachment.Position = Vector3.new(0, -0.5, 0)
		attachment.Parent = hand

		local effects = {}
		scale = scale or 1

		local emitterConfigs = {
			{
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
					NumberSequenceKeypoint.new(0.7, 2 * scale), 
					NumberSequenceKeypoint.new(1, 0.5 * scale)
				}),
				Transparency = NumberSequence.new({
					NumberSequenceKeypoint.new(0, 0.3), 
					NumberSequenceKeypoint.new(0.4, 0.5), 
					NumberSequenceKeypoint.new(0.8, 0.8), 
					NumberSequenceKeypoint.new(1, 1)
				}),
				Lifetime = NumberRange.new(0.6, 1),
				Rate = 80 * scale,
				Speed = NumberRange.new(1, 2.5),
				SpreadAngle = Vector2.new(25, 25),
				LightEmission = 0,
				ZOffset = -0.1,
				Acceleration = Vector3.new(0, 2, 0),
				Drag = 3
			},
			{
				Name = "EnergyCore",
				Texture = "rbxassetid://11534281007",
				Color = ColorSequence.new({
					ColorSequenceKeypoint.new(0, CONFIG.COLORS.bright), 
					ColorSequenceKeypoint.new(0.3, CONFIG.COLORS.glow), 
					ColorSequenceKeypoint.new(1, CONFIG.COLORS.primary)
				}),
				Size = NumberSequence.new({
					NumberSequenceKeypoint.new(0, 0.5 * scale), 
					NumberSequenceKeypoint.new(0.2, 1.5 * scale), 
					NumberSequenceKeypoint.new(0.6, 1.2 * scale), 
					NumberSequenceKeypoint.new(1, 0.3 * scale)
				}),
				Transparency = NumberSequence.new({
					NumberSequenceKeypoint.new(0, 0), 
					NumberSequenceKeypoint.new(0.3, 0.1), 
					NumberSequenceKeypoint.new(0.7, 0.5), 
					NumberSequenceKeypoint.new(1, 1)
				}),
				Lifetime = NumberRange.new(0.4, 0.7),
				Rate = 120 * scale,
				Speed = NumberRange.new(2, 4),
				SpreadAngle = Vector2.new(15, 15),
				LightEmission = 1,
				LightInfluence = 0,
				ZOffset = 0.3,
				Acceleration = Vector3.new(0, 3, 0),
				Drag = 2.5
			},
			{
				Name = "EnergyOuter",
				Texture = "rbxassetid://11534281007",
				Color = ColorSequence.new({
					ColorSequenceKeypoint.new(0, CONFIG.COLORS.primary), 
					ColorSequenceKeypoint.new(0.5, CONFIG.COLORS.secondary), 
					ColorSequenceKeypoint.new(1, CONFIG.COLORS.primary)
				}),
				Size = NumberSequence.new({
					NumberSequenceKeypoint.new(0, 0.8 * scale), 
					NumberSequenceKeypoint.new(0.3, 2 * scale), 
					NumberSequenceKeypoint.new(0.7, 1.5 * scale), 
					NumberSequenceKeypoint.new(1, 0)
				}),
				Transparency = NumberSequence.new({
					NumberSequenceKeypoint.new(0, 0.2), 
					NumberSequenceKeypoint.new(0.4, 0.3), 
					NumberSequenceKeypoint.new(0.8, 0.7), 
					NumberSequenceKeypoint.new(1, 1)
				}),
				Lifetime = NumberRange.new(0.5, 0.9),
				Rate = 100 * scale,
				Speed = NumberRange.new(1.5, 3),
				SpreadAngle = Vector2.new(20, 20),
				LightEmission = 0.9,
				ZOffset = 0.2,
				Acceleration = Vector3.new(0, 2.5, 0),
				Drag = 2
			},
			{
				Name = "BaseGlow",
				Texture = "rbxassetid://11841348746",
				Color = ColorSequence.new({
					ColorSequenceKeypoint.new(0, CONFIG.COLORS.bright), 
					ColorSequenceKeypoint.new(0.5, CONFIG.COLORS.glow), 
					ColorSequenceKeypoint.new(1, CONFIG.COLORS.primary)
				}),
				Size = NumberSequence.new({
					NumberSequenceKeypoint.new(0, 1.2 * scale), 
					NumberSequenceKeypoint.new(0.4, 1.8 * scale), 
					NumberSequenceKeypoint.new(1, 0)
				}),
				Transparency = NumberSequence.new({
					NumberSequenceKeypoint.new(0, 0.3), 
					NumberSequenceKeypoint.new(0.5, 0.6), 
					NumberSequenceKeypoint.new(1, 1)
				}),
				Lifetime = NumberRange.new(0.2, 0.4),
				Rate = 80 * scale,
				Speed = NumberRange.new(0.5, 1.5),
				SpreadAngle = Vector2.new(25, 25),
				LightEmission = 1,
				ZOffset = 0.1,
				Acceleration = Vector3.new(0, 2, 0),
				Drag = 1
			},
			{
				Name = "EnergyWisps",
				Texture = "rbxassetid://11841348746",
				Color = ColorSequence.new({
					ColorSequenceKeypoint.new(0, CONFIG.COLORS.glow), 
					ColorSequenceKeypoint.new(1, CONFIG.COLORS.secondary)
				}),
				Size = NumberSequence.new({
					NumberSequenceKeypoint.new(0, 0.3 * scale), 
					NumberSequenceKeypoint.new(0.5, 0.6 * scale), 
					NumberSequenceKeypoint.new(1, 0.2 * scale)
				}),
				Transparency = NumberSequence.new({
					NumberSequenceKeypoint.new(0, 0.2), 
					NumberSequenceKeypoint.new(0.6, 0.6), 
					NumberSequenceKeypoint.new(1, 1)
				}),
				Lifetime = NumberRange.new(0.6, 1),
				Rate = 60 * scale,
				Speed = NumberRange.new(3, 5),
				SpreadAngle = Vector2.new(12, 12),
				LightEmission = 1,
				ZOffset = 0.4,
				Acceleration = Vector3.new(0, 4, 0),
				Drag = 1.8
			}
		}

		for _, config in ipairs(emitterConfigs) do
			local emitter = Instance.new("ParticleEmitter")
			emitter.Name = config.Name
			emitter.Parent = attachment
			emitter.Texture = config.Texture
			emitter.Color = config.Color
			emitter.Size = config.Size
			emitter.Transparency = config.Transparency
			emitter.Lifetime = config.Lifetime
			emitter.Rate = config.Rate
			emitter.Speed = config.Speed
			emitter.SpreadAngle = config.SpreadAngle
			emitter.Rotation = NumberRange.new(0, 360)
			emitter.RotSpeed = NumberRange.new(-150, 150)
			emitter.LightEmission = config.LightEmission or 0
			emitter.LightInfluence = config.LightInfluence or 0
			emitter.ZOffset = config.ZOffset
			emitter.Acceleration = config.Acceleration
			emitter.Drag = config.Drag
			emitter.VelocityInheritance = 0
			emitter.LockedToPart = true
			emitter.TimeScale = 1
			emitter.EmissionDirection = Enum.NormalId.Top
			emitter.WindAffectsDrag = false
			emitter.FlipbookLayout = Enum.ParticleFlipbookLayout.Grid4x4
			emitter.FlipbookMode = Enum.ParticleFlipbookMode.OneShot
			emitter.FlipbookStartRandom = false
			emitter.Enabled = true
			table.insert(effects, emitter)
		end

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
			local leftHand = PlayerData.character:FindFirstChild("Left Arm") 
				or PlayerData.character:FindFirstChild("LeftHand") 
				or PlayerData.character:FindFirstChild("LeftLowerArm")
			local rightHand = PlayerData.character:FindFirstChild("Right Arm") 
				or PlayerData.character:FindFirstChild("RightHand") 
				or PlayerData.character:FindFirstChild("RightLowerArm")

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
						for attempt = 1, 30 do
							if not validateCharacter() then break end

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
				end)

				task.spawn(function()
					local repStorage = Services.ReplicatedStorage
					local resources = repStorage:FindFirstChild("Resources")
					if resources then
						local finisher = resources:FindFirstChild("Consecutive Punches Finisher")
						if finisher then
							local handClones = finisher:FindFirstChild("HandClones")
							if handClones then
								for attempt = 1, 40 do
									if not validateCharacter() then break end

									for i = 1, 5 do
										local cloneName = string.format("CLONE%02d", i)
										local clone = handClones:FindFirstChild(cloneName)
										if clone and clone:IsDescendantOf(workspace) then
											local cloneLeftArm = clone:FindFirstChild("Left Arm")
											local cloneRightArm = clone:FindFirstChild("Right Arm")

											if cloneLeftArm and cloneLeftArm:IsA("BasePart") then
												local key = "clone" .. i .. "_left_" .. tick()
												if not effectGroups[key] then
													effectGroups[key] = VFXManager.createAdvancedEnergyEffect(cloneLeftArm, 0.8)
												end
											end
											if cloneRightArm and cloneRightArm:IsA("BasePart") then
												local key = "clone" .. i .. "_right_" .. tick()
												if not effectGroups[key] then
													effectGroups[key] = VFXManager.createAdvancedEnergyEffect(cloneRightArm, 0.8)
												end
											end
										end
									end
									task.wait(0.05)
								end
							end
						end
					end
				end)
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

	task.spawn(function()
		safeCall(function()
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
						if not validateCharacter() then break end

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
			table.insert(PlayerData.activeEffects, vfxClone)
		end)
	end)
end

function VFXManager.trigger(vfxType, duration)
	if not validateCharacter() then return end

	task.spawn(function()
		safeCall(function()
			local hrp = PlayerData.character:FindFirstChild("HumanoidRootPart")
			if not hrp then return end

			if vfxType == "blackFlash" then
				VFXManager.createCustomBlackFlash()
			end
		end)
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
				TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), 
				{FieldOfView = originalFOV + 20}
			)
			fovIncrease:Play()

			task.wait(0.3)

			if not validateCharacter() then
				camera.FieldOfView = originalFOV
				return
			end

			local fovDecrease = Services.TweenService:Create(
				camera, 
				TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In), 
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
				TweenInfo.new(0.5, Enum.EasingStyle.Cubic, Enum.EasingDirection.InOut), 
				{Ambient = Color3.new(0.4, 0.05, 0.05), Brightness = 1}
			)
			lightTween:Play()

			task.wait(0.5)

			if not validateCharacter() then
				Services.Lighting.Ambient = originalAmbient
				Services.Lighting.Brightness = originalBrightness
				return
			end

			local lightReturn = Services.TweenService:Create(
				Services.Lighting, 
				TweenInfo.new(0.5, Enum.EasingStyle.Cubic, Enum.EasingDirection.InOut), 
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
			local hrp = PlayerData.character:FindFirstChild("HumanoidRootPart")
			if not hrp then return end

			local sound = Instance.new("Sound")
			sound.SoundId = "rbxassetid://" .. soundId
			sound.Volume = volume or 1
			sound.Parent = hrp
			sound:Play()
			Services.Debris:AddItem(sound, 5)

			table.insert(PlayerData.activeEffects, sound)
		end)
	end)
end

local AnimationManager = {}

function AnimationManager.stopAnimation(animationId)
	if not validateCharacter() then return end

	safeCall(function()
		if not PlayerData.humanoid then return end

		for _, track in ipairs(PlayerData.humanoid:GetPlayingAnimationTracks()) do
			if track and track.Animation then
				local trackId = tonumber(track.Animation.AnimationId:match("%d+"))
				if trackId == animationId then
					track:Stop()
				end
			end
		end
	end)
end

function AnimationManager.playAnimation(animationId, speed, timePos)
	if not validateCharacter() then return nil end

	local success, track = safeCall(function()
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
	if not validateCharacter() then return end

	safeCall(function()
		if not PlayerData.humanoid then return end

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
				task.spawn(function()
					VFXManager.createAdvancedHandFire(
						config.handFireDuration or 2, 
						config.includeClones or false
					)
				end)
			end

			if config.useBlackFlashText then
				task.spawn(function()
					task.wait(config.blackFlashDelay or 0)
					NotificationManager.createBlackFlash()
				end)
			end

			if config.useCustomVFX then
				task.spawn(function()
					task.wait(config.vfxDelay or 0)
					VFXManager.trigger("blackFlash", 1)
				end)
			end

			if config.useFOV then
				task.spawn(CameraManager.applyFOVEffect)
			end

			if config.useRedLight then
				task.spawn(LightingManager.applyRedLight)
			end
		end)

		table.insert(PlayerData.connections, connection)
	end)
end

function AnimationManager.playFullReplacement(originalId, config)
	if not validateCharacter() then return end

	if PlayerData.isAnimating then
		table.insert(PlayerData.animationQueue, {originalId, config})
		return
	end

	safeCall(function()
		PlayerData.isAnimating = true

		local animation = Instance.new("Animation")
		animation.AnimationId = "rbxassetid://" .. config.animationId

		local animTrack = PlayerData.humanoid:LoadAnimation(animation)
		animTrack:Play()
		animTrack:AdjustSpeed(config.speed or 1)

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
	if not validateCharacter() then return end

	safeCall(function()
		if not PlayerData.humanoid then return end

		local connection = PlayerData.humanoid.AnimationPlayed:Connect(function(animationTrack)
			if not validateCharacter() then return end
			if not animationTrack or not animationTrack.Animation then return end

			local animId = tonumber(animationTrack.Animation.AnimationId:match("%d+"))
			if animId ~= originalId then return end

			AnimationManager.stopAnimation(originalId)
			animationTrack:Stop()
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
	if not validateCharacter() then return end

	safeCall(function()
		if not PlayerData.character then return end

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

		task.spawn(UIManager.updateAttackNames)
		PhysicsManager.setupVelocityHandler()
	end)
end

function CharacterManager.cleanup()
	PlayerData.isAlive = false
	cleanupConnections()
	cleanupEffects()
	PlayerData.animationQueue = {}
	PlayerData.isAnimating = false
end

function CharacterManager.onCharacterAdded(newCharacter)
	CharacterManager.cleanup()

	safeCall(function()
		PlayerData.character = newCharacter
		PlayerData.humanoid = newCharacter:WaitForChild("Humanoid", 5)

		if PlayerData.humanoid then
			PlayerData.isAlive = true

			local deathConnection = PlayerData.humanoid.Died:Connect(function()
				CharacterManager.cleanup()
			end)

			table.insert(PlayerData.connections, deathConnection)

			task.wait(1)
			CharacterManager.initialize()
		end
	end)
end

local SystemManager = {}

function SystemManager.start()
	safeCall(function()
		local success = initializePlayerData()
		if not success then
			return
		end

		if PlayerData.humanoid then
			local deathConnection = PlayerData.humanoid.Died:Connect(function()
				CharacterManager.cleanup()
			end)

			table.insert(PlayerData.connections, deathConnection)
		end

		task.wait(1)
		CharacterManager.initialize()

		local charAddedConnection = PlayerData.player.CharacterAdded:Connect(CharacterManager.onCharacterAdded)
		table.insert(PlayerData.connections, charAddedConnection)
	end)
end

SystemManager.start()
