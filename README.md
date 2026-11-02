-- Painel de testes para o SEU jogo (LocalScript)
-- StarterPlayer > StarterPlayerScripts. Toque no iOS e PC.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local player = Players.LocalPlayer
if not player then
	warn("[PainelTeste] Use um LocalScript em StarterPlayerScripts.")
	return
end
local camera = workspace.CurrentCamera
local playerGui = player:WaitForChild("PlayerGui")

local char, hum, root
local connections = {}
local function bind(c) table.insert(connections, c) return c end

local state = {
	esp = false, speed = false, speedValue = 50, jump = false, noclip = false,
	sky = false, skyRise = 8,
	tp = false, tpDist = 10, tpTime = 5,
	aim = false, aimPart = "Cabeça", fov = 120, lock = 25,
	fly = false, flySpeed = 60,
}

-- ================= INTERFACE =================
local gui = Instance.new("ScreenGui")
gui.Name = "PainelTeste"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.Parent = playerGui

local panel = Instance.new("Frame")
panel.Size = UDim2.fromOffset(220, 260)
panel.Position = UDim2.fromOffset(20, 60)
panel.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
panel.BorderSizePixel = 0
panel.Active = true
panel.Parent = gui
Instance.new("UICorner", panel).CornerRadius = UDim.new(0, 8)

local header = Instance.new("TextLabel")
header.Size = UDim2.new(1, 0, 0, 30)
header.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
header.Text = "  Painel de Teste"
header.TextXAlignment = Enum.TextXAlignment.Left
header.TextColor3 = Color3.new(1, 1, 1)
header.Font = Enum.Font.GothamBold
header.TextSize = 14
header.Active = true
header.Parent = panel
Instance.new("UICorner", header).CornerRadius = UDim.new(0, 8)

local minBtn = Instance.new("TextButton")
minBtn.Size = UDim2.fromOffset(30, 30)
minBtn.Position = UDim2.new(1, -30, 0, 0)
minBtn.BackgroundTransparency = 1
minBtn.Text = "–"
minBtn.TextColor3 = Color3.new(1, 1, 1)
minBtn.Font = Enum.Font.GothamBold
minBtn.TextSize = 20
minBtn.Parent = header

local list = Instance.new("ScrollingFrame")
list.Size = UDim2.new(1, -10, 1, -36)
list.Position = UDim2.fromOffset(5, 33)
list.BackgroundTransparency = 1
list.BorderSizePixel = 0
list.ScrollBarThickness = 4
list.CanvasSize = UDim2.new()
list.AutomaticCanvasSize = Enum.AutomaticSize.Y
list.ScrollingDirection = Enum.ScrollingDirection.Y
list.Parent = panel
local layout = Instance.new("UIListLayout", list)
layout.Padding = UDim.new(0, 4)
layout.SortOrder = Enum.SortOrder.LayoutOrder

local order = 0
local function nextOrder() order += 1 return order end

local function section(text)
	local l = Instance.new("TextLabel")
	l.Size = UDim2.new(1, -6, 0, 18)
	l.BackgroundTransparency = 1
	l.Text = text
	l.TextColor3 = Color3.fromRGB(150, 150, 170)
	l.Font = Enum.Font.GothamBold
	l.TextSize = 11
	l.TextXAlignment = Enum.TextXAlignment.Left
	l.LayoutOrder = nextOrder()
	l.Parent = list
end

local function toggle(name, key, onChange)
	local b = Instance.new("TextButton")
	b.Size = UDim2.new(1, -6, 0, 30)
	b.Font = Enum.Font.GothamBold
	b.TextSize = 13
	b.TextColor3 = Color3.new(1, 1, 1)
	b.AutoButtonColor = true
	b.LayoutOrder = nextOrder()
	b.Parent = list
	Instance.new("UICorner", b).CornerRadius = UDim.new(0, 6)
	local function refresh()
		b.Text = name .. (state[key] and ": ON" or ": OFF")
		b.BackgroundColor3 = state[key] and Color3.fromRGB(40, 160, 70) or Color3.fromRGB(170, 50, 50)
	end
	b.Activated:Connect(function()
		state[key] = not state[key]
		refresh()
		local ok, err = pcall(onChange, state[key])
		if not ok then warn("[PainelTeste] Erro em " .. name .. ": " .. tostring(err)) end
	end)
	refresh()
	return refresh
end

local function adjust(name, key, step, minV, maxV, onChange)
	local row = Instance.new("Frame")
	row.Size = UDim2.new(1, -6, 0, 28)
	row.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
	row.LayoutOrder = nextOrder()
	row.Parent = list
	Instance.new("UICorner", row).CornerRadius = UDim.new(0, 6)
	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(1, -64, 1, 0)
	label.Position = UDim2.fromOffset(32, 0)
	label.BackgroundTransparency = 1
	label.TextColor3 = Color3.new(1, 1, 1)
	label.Font = Enum.Font.Gotham
	label.TextSize = 12
	label.Parent = row
	local function mk(txt, x, delta)
		local b = Instance.new("TextButton")
		b.Size = UDim2.fromOffset(28, 24)
		b.Position = x
		b.Text = txt
		b.BackgroundColor3 = Color3.fromRGB(70, 70, 90)
		b.TextColor3 = Color3.new(1, 1, 1)
		b.Font = Enum.Font.GothamBold
		b.TextSize = 16
		b.Parent = row
		Instance.new("UICorner", b).CornerRadius = UDim.new(0, 5)
		b.Activated:Connect(function()
			state[key] = math.clamp(state[key] + delta, minV, maxV)
			label.Text = name .. ": " .. state[key]
			if onChange then pcall(onChange, state[key]) end
		end)
	end
	mk("−", UDim2.fromOffset(2, 2), -step)
	mk("+", UDim2.new(1, -30, 0, 2), step)
	label.Text = name .. ": " .. state[key]
end

local function cycle(name, key, options)
	local b = Instance.new("TextButton")
	b.Size = UDim2.new(1, -6, 0, 28)
	b.BackgroundColor3 = Color3.fromRGB(60, 60, 110)
	b.TextColor3 = Color3.new(1, 1, 1)
	b.Font = Enum.Font.GothamBold
	b.TextSize = 12
	b.LayoutOrder = nextOrder()
	b.Parent = list
	Instance.new("UICorner", b).CornerRadius = UDim.new(0, 6)
	b.Text = name .. ": " .. state[key]
	b.Activated:Connect(function()
		local i = table.find(options, state[key]) or 0
		state[key] = options[i % #options + 1]
		b.Text = name .. ": " .. state[key]
	end)
end

-- Minimizar
local minimized = false
minBtn.Activated:Connect(function()
	minimized = not minimized
	list.Visible = not minimized
	panel.Size = minimized and UDim2.fromOffset(220, 30) or UDim2.fromOffset(220, 260)
	minBtn.Text = minimized and "+" or "–"
end)

-- Arrastar pelo título
do
	local dragging, startPos, startInput
	header.InputBegan:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.Touch or i.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging, startInput, startPos = true, i.Position, panel.Position
			i.Changed:Connect(function()
				if i.UserInputState == Enum.UserInputState.End then dragging = false end
			end)
		end
	end)
	bind(UIS.InputChanged:Connect(function(i)
		if dragging and (i.UserInputType == Enum.UserInputType.Touch or i.UserInputType == Enum.UserInputType.MouseMovement) then
			local d = i.Position - startInput
			panel.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
		end
	end))
end

-- ================= MOVIMENTO =================
local function applyMovement()
	if not hum then return end
	hum.WalkSpeed = state.speed and state.speedValue or 16
	hum.UseJumpPower = true
	hum.JumpPower = state.jump and 120 or 50
end

-- ================= ESP =================
local espGui = Instance.new("ScreenGui")
espGui.Name = "ESPTeste"
espGui.ResetOnSpawn = false
espGui.IgnoreGuiInset = true
espGui.Parent = playerGui

local espItems = {}
local function isAlly(p)
	return player.Team ~= nil and not player.Neutral and p.Team == player.Team
end

local function removeESP(p)
	local item = espItems[p]
	if item then
		if item.hl then item.hl:Destroy() end
		item.box:Destroy()
		item.label:Destroy()
		espItems[p] = nil
	end
end

local function clearESP()
	for p in pairs(espItems) do removeESP(p) end
end

local function getESP(p)
	local item = espItems[p]
	if not item then
		local box = Instance.new("Frame")
		box.BackgroundTransparency = 1
		box.BorderSizePixel = 0
		box.Parent = espGui
		local stroke = Instance.new("UIStroke", box)
		stroke.Color = Color3.new(1, 1, 1)
		stroke.Thickness = 1
		local label = Instance.new("TextLabel")
		label.BackgroundTransparency = 1
		label.TextColor3 = Color3.new(1, 1, 1)
		label.TextStrokeTransparency = 0.4
		label.Font = Enum.Font.GothamBold
		label.TextSize = 12
		label.Size = UDim2.fromOffset(200, 16)
		label.AnchorPoint = Vector2.new(0.5, 1)
		label.Parent = espGui
		item = { box = box, label = label }
		espItems[p] = item
	end
	if p.Character and (not item.hl or item.hl.Parent ~= p.Character) then
		if item.hl then item.hl:Destroy() end
		local hl = Instance.new("Highlight")
		hl.FillColor = Color3.fromRGB(255, 50, 50)
		hl.OutlineColor = Color3.new(1, 1, 1)
		hl.FillTransparency = 0.5
		hl.Parent = p.Character
		item.hl = hl
	end
	return item
end

bind(Players.PlayerRemoving:Connect(removeESP))

bind(RunService.RenderStepped:Connect(function()
	if not state.esp then return end
	for _, p in ipairs(Players:GetPlayers()) do
		local c = p.Character
		local h = c and c:FindFirstChildOfClass("Humanoid")
		local r = c and c:FindFirstChild("HumanoidRootPart")
		if p == player or isAlly(p) or not (h and r and h.Health > 0) then
			removeESP(p)
		else
			local item = getESP(p)
			local cf, size = c:GetBoundingBox()
			local minX, minY, maxX, maxY = math.huge, math.huge, -math.huge, -math.huge
			local visible = false
			for _, x in ipairs({-0.5, 0.5}) do
				for _, y in ipairs({-0.5, 0.5}) do
					for _, z in ipairs({-0.5, 0.5}) do
						local pt, on = camera:WorldToViewportPoint((cf * CFrame.new(size.X * x, size.Y * y, size.Z * z)).Position)
						if pt.Z > 0 then
							visible = visible or on
							minX, maxX = math.min(minX, pt.X), math.max(maxX, pt.X)
							minY, maxY = math.min(minY, pt.Y), math.max(maxY, pt.Y)
						end
					end
				end
			end
			item.box.Visible = visible
			item.label.Visible = visible
			if visible then
				item.box.Position = UDim2.fromOffset(minX, minY)
				item.box.Size = UDim2.fromOffset(maxX - minX, maxY - minY)
				local dist = root and math.floor((root.Position - r.Position).Magnitude) or 0
				item.label.Text = p.DisplayName .. " | " .. dist .. "m"
				item.label.Position = UDim2.fromOffset((minX + maxX) / 2, minY - 2)
			end
		end
	end
end))

-- ================= NOCLIP =================
local savedCollide = {}
local function restoreCollisions()
	for part, v in pairs(savedCollide) do
		if part.Parent then part.CanCollide = v end
	end
	savedCollide = {}
end

bind(RunService.Stepped:Connect(function()
	if not (state.noclip and char) then return end
	for _, part in ipairs(char:GetDescendants()) do
		if part:IsA("BasePart") and part.CanCollide then
			if savedCollide[part] == nil then savedCollide[part] = true end
			part.CanCollide = false
		end
	end
end))

-- ================= SKYWALK (sobe) =================
local skyPart, skyY
local refreshSky
local function stopSky()
	if skyPart then skyPart:Destroy() skyPart = nil end
	skyY = nil
end
local function startSky()
	stopSky()
	if not (root and hum) then return end
	skyPart = Instance.new("Part")
	skyPart.Name = "SkyWalkTeste"
	skyPart.Size = Vector3.new(8, 1, 8)
	skyPart.Anchored = true
	skyPart.Transparency = 0.5
	skyPart.Color = Color3.fromRGB(120, 200, 255)
	skyPart.Material = Enum.Material.Neon
	skyPart.Parent = workspace
	skyY = root.Position.Y - root.Size.Y / 2 - hum.HipHeight - 0.5
end

bind(RunService.Heartbeat:Connect(function(dt)
	if not (state.sky and skyPart and root and hum) then return end
	local feetY = root.Position.Y - root.Size.Y / 2 - hum.HipHeight - 0.5
	-- Sobe sozinho na velocidade escolhida
	skyY += state.skyRise * dt
	-- Se pular mais alto, a plataforma acompanha (não deixa cair)
	if feetY > skyY + 0.3 and root.AssemblyLinearVelocity.Y <= 0 then
		skyY = feetY
	end
	skyPart.CFrame = CFrame.new(root.Position.X, skyY, root.Position.Z)
	-- Empurra o jogador junto quando a plataforma sobe
	if feetY < skyY + 0.2 then
		root.CFrame += Vector3.new(0, (skyY + 0.2) - feetY, 0)
		local v = root.AssemblyLinearVelocity
		if v.Y < 0 then root.AssemblyLinearVelocity = Vector3.new(v.X, 0, v.Z) end
	end
end))

-- ================= TELEPORTE AO ANDAR =================
local walkTime = 0
bind(RunService.Heartbeat:Connect(function(dt)
	if not (state.tp and root and hum and hum.Health > 0) then walkTime = 0 return end
	if hum.MoveDirection.Magnitude > 0.1 then
		walkTime += dt
		if walkTime >= state.tpTime then
			walkTime = 0
			local dir = hum.MoveDirection.Unit
			root.CFrame += Vector3.new(dir.X, 0, dir.Z) * state.tpDist
		end
	else
		walkTime = 0
	end
end))

-- ================= FLY =================
local flyAtt, flyVel
local upHeld, downHeld = false, false

local flyButtons = Instance.new("Frame")
flyButtons.Size = UDim2.fromOffset(60, 130)
flyButtons.Position = UDim2.new(1, -80, 0.5, -65)
flyButtons.BackgroundTransparency = 1
flyButtons.Visible = false
flyButtons.Parent = gui

local function holdButton(txt, y, setter)
	local b = Instance.new("TextButton")
	b.Size = UDim2.fromOffset(60, 60)
	b.Position = UDim2.fromOffset(0, y)
	b.Text = txt
	b.TextSize = 26
	b.Font = Enum.Font.GothamBold
	b.TextColor3 = Color3.new(1, 1, 1)
	b.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
	b.BackgroundTransparency = 0.3
	b.Parent = flyButtons
	Instance.new("UICorner", b).CornerRadius = UDim.new(1, 0)
	b.InputBegan:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.Touch or i.UserInputType == Enum.UserInputType.MouseButton1 then
			setter(true)
		end
	end)
	b.InputEnded:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.Touch or i.UserInputType == Enum.UserInputType.MouseButton1 then
			setter(false)
		end
	end)
end
holdButton("▲", 0, function(v) upHeld = v end)
holdButton("▼", 70, function(v) downHeld = v end)

local function stopFly()
	if flyVel then flyVel:Destroy() flyVel = nil end
	if flyAtt then flyAtt:Destroy() flyAtt = nil end
	flyButtons.Visible = false
	upHeld, downHeld = false, false
	if hum then hum:ChangeState(Enum.HumanoidStateType.Freefall) end
end
local function startFly()
	stopFly()
	if not root then return end
	flyAtt = Instance.new("Attachment", root)
	flyVel = Instance.new("LinearVelocity")
	flyVel.Attachment0 = flyAtt
	flyVel.MaxForce = math.huge
	flyVel.VelocityConstraintMode = Enum.VelocityConstraintMode.Vector
	flyVel.RelativeTo = Enum.ActuatorRelativeTo.World
	flyVel.VectorVelocity = Vector3.zero
	flyVel.Parent = root
	flyButtons.Visible = true
end

bind(RunService.Heartbeat:Connect(function()
	if not (state.fly and flyVel and hum) then return end
	local move = hum.MoveDirection -- analógico do iPhone / WASD
	local vertical = 0
	if move.Magnitude > 0 then
		-- Segue a inclinação da câmera ao andar para frente
		local look = camera.CFrame.LookVector
		vertical = look.Y * move:Dot(Vector3.new(look.X, 0, look.Z).Unit)
	end
	if upHeld or UIS:IsKeyDown(Enum.KeyCode.Space) then vertical = 1 end
	if downHeld or UIS:IsKeyDown(Enum.KeyCode.LeftControl) then vertical = -1 end
	flyVel.VectorVelocity = (move + Vector3.new(0, vertical, 0)) * state.flySpeed
	hum:ChangeState(Enum.HumanoidStateType.Physics)
end))

-- ================= AIMBOT =================
local fovCircle = Instance.new("Frame")
fovCircle.AnchorPoint = Vector2.new(0.5, 0.5)
fovCircle.Position = UDim2.fromScale(0.5, 0.5)
fovCircle.BackgroundTransparency = 1
fovCircle.Visible = false
fovCircle.Parent = gui
Instance.new("UICorner", fovCircle).CornerRadius = UDim.new(1, 0)
local fovStroke = Instance.new("UIStroke", fovCircle)
fovStroke.Color = Color3.new(1, 1, 1)
fovStroke.Thickness = 1.5

local function updateFov()
	fovCircle.Size = UDim2.fromOffset(state.fov * 2, state.fov * 2)
end
updateFov()

local function getAimPart(c)
	if state.aimPart == "Cabeça" then
		return c:FindFirstChild("Head")
	elseif state.aimPart == "Torso" then
		return c:FindFirstChild("UpperTorso") or c:FindFirstChild("Torso") or c:FindFirstChild("HumanoidRootPart")
	else
		return c:FindFirstChild("RightFoot") or c:FindFirstChild("LeftFoot") or c:FindFirstChild("Right Leg") or c:FindFirstChild("HumanoidRootPart")
	end
end

local function isVisible(part, targetChar)
	local params = RaycastParams.new()
	params.FilterType = Enum.RaycastFilterType.Exclude
	params.FilterDescendantsInstances = { char, skyPart }
	local origin = camera.CFrame.Position
	local result = workspace:Raycast(origin, part.Position - origin, params)
	return result == nil or result.Instance:IsDescendantOf(targetChar)
end

local function validTarget(p)
	if not p or p == player or isAlly(p) then return nil end
	local c = p.Character
	local h = c and c:FindFirstChildOfClass("Humanoid")
	if not (h and h.Health > 0) then return nil end
	local part = getAimPart(c)
	if not part then return nil end
	local pt, on = camera:WorldToViewportPoint(part.Position)
	if not on or pt.Z <= 0 then return nil end
	if not isVisible(part, c) then return nil end
	return part, Vector2.new(pt.X, pt.Y)
end

local lockedPlayer
RunService:BindToRenderStep("AimTeste", Enum.RenderPriority.Camera.Value + 1, function(dt)
	fovCircle.Visible = state.aim
	if not state.aim then lockedPlayer = nil return end
	local center = camera.ViewportSize / 2
	local part = lockedPlayer and validTarget(lockedPlayer)
	if not part then
		lockedPlayer = nil
		local best = state.fov
		for _, p in ipairs(Players:GetPlayers()) do
			local pp, screen = validTarget(p)
			if pp then
				local d = (screen - center).Magnitude
				if d <= best then best, lockedPlayer, part = d, p, pp end
			end
		end
	end
	if part then
		local desired = CFrame.lookAt(camera.CFrame.Position, part.Position)
		local alpha = 1 - math.exp(-state.lock * dt)
		camera.CFrame = camera.CFrame:Lerp(desired, alpha)
	end
end)

-- ================= BOTÕES =================
section("JOGADOR")
toggle("ESP Player", "esp", function(on) if not on then clearESP() end end)
toggle("Speed", "speed", applyMovement)
adjust("Velocidade", "speedValue", 10, 16, 500, applyMovement)
toggle("Super Pulo", "jump", applyMovement)
toggle("Noclip", "noclip", function(on) if not on then restoreCollisions() end end)

section("MOVIMENTO")
refreshSky = toggle("SkyWalk", "sky", function(on) if on then startSky() else stopSky() end end)
adjust("Subida", "skyRise", 2, 0, 60)
local refreshTp = toggle("Teleporte", "tp", function() walkTime = 0 end)
adjust("Distância TP", "tpDist", 5, 5, 200)
adjust("Tempo TP (s)", "tpTime", 1, 1, 30)
local refreshFly = toggle("Fly", "fly", function(on) if on then startFly() else stopFly() end end)
adjust("Vel. Fly", "flySpeed", 10, 10, 300)

section("MIRA")
toggle("Aimbot", "aim", function() lockedPlayer = nil end)
cycle("Parte", "aimPart", { "Cabeça", "Torso", "Pé" })
adjust("FOV", "fov", 10, 30, 400, updateFov)
adjust("Trava", "lock", 5, 5, 60)

-- ================= PERSONAGEM =================
local function onCharacter(c)
	char = c
	hum = c:WaitForChild("Humanoid")
	root = c:WaitForChild("HumanoidRootPart")
	savedCollide = {}
	applyMovement()
	if state.fly then state.fly = false stopFly() refreshFly() end
	if state.sky then state.sky = false stopSky() refreshSky() end
	hum.Died:Connect(function()
		if state.fly then state.fly = false stopFly() refreshFly() end
		if state.sky then state.sky = false stopSky() refreshSky() end
	end)
end
if player.Character then task.spawn(onCharacter, player.Character) end
bind(player.CharacterAdded:Connect(onCharacter))

gui.Destroying:Connect(function()
	RunService:UnbindFromRenderStep("AimTeste")
	for _, c in ipairs(connections) do c:Disconnect() end
	stopFly() stopSky() restoreCollisions() clearESP()
	espGui:Destroy()
end)

print("[PainelTeste] Painel carregado (SkyWalk que sobe + Teleporte).")
