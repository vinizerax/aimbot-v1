# aimbot-v1

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local Camera = Workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local Drawing = Drawing or loadstring(game:HttpGet("https://raw.githubusercontent.com/CenteredSniper/Kenzen/master/Drawing.lua"))()

local aimbotEnabled = true
local fovRadius = 150
local lookingBack = false
local lastLookDirection = nil

-- FOV Circle
local fovCircle = Drawing.new("Circle")
fovCircle.Color = Color3.new(1, 0, 0)
fovCircle.Thickness = 1.5
fovCircle.Radius = fovRadius
fovCircle.Filled = false
fovCircle.Transparency = 1
fovCircle.Visible = true

-- Função: checar visibilidade
local function IsVisible(target)
	local origin = Camera.CFrame.Position
	local direction = (target.Position - origin).Unit * 500
	local rayParams = RaycastParams.new()
	rayParams.FilterDescendantsInstances = {LocalPlayer.Character}
	rayParams.FilterType = Enum.RaycastFilterType.Blacklist
	local result = Workspace:Raycast(origin, direction, rayParams)
	return result and result.Instance and target:IsDescendantOf(result.Instance.Parent)
end

-- Função: checar se dentro do FOV
local function IsInFOV(position)
	local screenPoint, onScreen = Camera:WorldToViewportPoint(position)
	if not onScreen then return false end
	local mousePos = UserInputService:GetMouseLocation()
	local distance = (Vector2.new(screenPoint.X, screenPoint.Y) - mousePos).Magnitude
	return distance <= fovRadius
end

-- Função: obter inimigos válidos no FOV
local function GetValidEnemiesInFOV()
	local enemies = {}
	for _, player in pairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Team ~= LocalPlayer.Team then
			local character = player.Character
			if character and character:FindFirstChild("Head") and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0 then
				local head = character.Head
				if IsVisible(head) and IsInFOV(head.Position) then
					table.insert(enemies, head)
				end
			end
		end
	end
	return enemies
end

-- Função: inimigo atrás
local function GetClosestEnemyBehind()
	local closestDistance = math.huge
	local closestEnemy = nil
	for _, player in pairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Team ~= LocalPlayer.Team then
			local character = player.Character
			if character and character:FindFirstChild("Head") and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0 then
				local head = character.Head
				local camForward = Camera.CFrame.LookVector
				local directionToHead = (head.Position - Camera.CFrame.Position).Unit
				local dot = camForward:Dot(directionToHead)
				if dot < -0.3 and IsVisible(head) then
					local distance = (Camera.CFrame.Position - head.Position).Magnitude
					if distance < closestDistance then
						closestDistance = distance
						closestEnemy = head
					end
				end
			end
		end
	end
	return closestEnemy
end

-- ESP
local espObjects = {}
local function UpdateESP()
	for _, obj in pairs(espObjects) do
		for _, drawing in pairs(obj) do
			drawing.Visible = false
		end
	end

	for _, player in pairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Team ~= LocalPlayer.Team then
			local character = player.Character
			if character and character:FindFirstChild("Head") and character:FindFirstChild("HumanoidRootPart") and character.Humanoid.Health > 0 then
				if not espObjects[player] then
					local box = Drawing.new("Square")
					box.Color = Color3.new(1, 0, 0)
					box.Thickness = 1
					box.Transparency = 1
					box.Filled = false

					local tracer = Drawing.new("Line")
					tracer.Color = Color3.new(1, 0, 0)
					tracer.Thickness = 1
					tracer.Transparency = 1

					local name = Drawing.new("Text")
					name.Color = Color3.new(1, 0, 0)
					name.Size = 14
					name.Center = true
					name.Outline = true

					local distance = Drawing.new("Text")
					distance.Color = Color3.new(1, 0, 0)
					distance.Size = 14
					distance.Center = true
					distance.Outline = true

					espObjects[player] = {box = box, tracer = tracer, name = name, distance = distance}
				end

				local screenPos, visible = Camera:WorldToViewportPoint(character.HumanoidRootPart.Position)
				if visible then
					local obj = espObjects[player]
					local headPos = Camera:WorldToViewportPoint(character.Head.Position)
					local rootPos = Camera:WorldToViewportPoint(character.HumanoidRootPart.Position)
					local distanceToPlayer = (Camera.CFrame.Position - character.HumanoidRootPart.Position).Magnitude

					-- Box
					obj.box.Size = Vector2.new(40, 60)
					obj.box.Position = Vector2.new(rootPos.X - 20, rootPos.Y - 30)
					obj.box.Visible = true

					-- Tracer
					local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
					obj.tracer.From = screenCenter
					obj.tracer.To = Vector2.new(rootPos.X, rootPos.Y)
					obj.tracer.Visible = true

					-- Nome
					obj.name.Position = Vector2.new(headPos.X, headPos.Y - 20)
					obj.name.Text = player.Name
					obj.name.Visible = true

					-- Distância
					obj.distance.Position = Vector2.new(headPos.X, headPos.Y)
					obj.distance.Text = string.format("%.1f studs", distanceToPlayer)
					obj.distance.Visible = true
				end
			end
		end
	end
end

-- Loop principal
RunService.RenderStepped:Connect(function()
	local mousePos = UserInputService:GetMouseLocation()
	fovCircle.Position = mousePos

	UpdateESP()

	-- Aimbot com botão direito
	if aimbotEnabled and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
		local enemies = GetValidEnemiesInFOV()
		if #enemies >= 1 then
			if #enemies >= 3 then
				local tickRate = 0.08
				for _, head in ipairs(enemies) do
					Camera.CFrame = CFrame.new(Camera.CFrame.Position, head.Position)
					task.wait(tickRate)
				end
			else
				local closest = enemies[1]
				Camera.CFrame = CFrame.new(Camera.CFrame.Position, closest.Position)
			end
		end
	end

	-- Mira atrás (botão esquerdo + inimigo visível atrás)
	if UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1)
		and not UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
		and not lookingBack then

		local enemyBehind = GetClosestEnemyBehind()
		if enemyBehind then
			lookingBack = true
			lastLookDirection = Camera.CFrame.LookVector
			local camPosition = Camera.CFrame.Position

			Camera.CFrame = CFrame.new(camPosition, enemyBehind.Position)

			task.delay(0.2, function()
				if lastLookDirection then
					Camera.CFrame = CFrame.new(camPosition, camPosition + lastLookDirection)
				end
				lookingBack = false
			end)
		end
	end
end)
