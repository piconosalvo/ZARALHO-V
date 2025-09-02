-- LocalScript dentro de StarterPlayerScripts
local player = game.Players.LocalPlayer
local runService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- Variáveis
local aimbotEnabled, espEnabled, invisibleEnabled = false, false, false
local highlights = {}
local screenGui, panel, ball

-- Função recriar interface
local function createUI()
	if screenGui then screenGui:Destroy() end

	screenGui = Instance.new("ScreenGui")
	screenGui.Name = "ZaralhoV4"
	screenGui.ResetOnSpawn = false
	screenGui.Parent = player:WaitForChild("PlayerGui")

	-- Painel
	panel = Instance.new("Frame", screenGui)
	panel.Size = UDim2.new(0, 420, 0, 400)
	panel.Position = UDim2.new(0.5, -210, 0.5, -200)
	panel.BackgroundColor3 = Color3.fromRGB(20,20,20)
	panel.BorderSizePixel = 0
	panel.Visible = true

	local corner = Instance.new("UICorner", panel)
	corner.CornerRadius = UDim.new(0, 12)

	local uiStroke = Instance.new("UIStroke", panel)
	uiStroke.Thickness = 3
	task.spawn(function()
		local hue = 0
		while task.wait() do
			hue += 0.01
			if hue > 1 then hue = 0 end
			uiStroke.Color = Color3.fromHSV(hue,1,1)
		end
	end)

	-- Título
	local title = Instance.new("TextLabel", panel)
	title.Size = UDim2.new(1,0,0,40)
	title.Text = "ZARALHO V4"
	title.TextColor3 = Color3.new(1,1,1)
	title.BackgroundTransparency = 1
	title.Font = Enum.Font.GothamBold
	title.TextSize = 24

	-- Minimizar
	local buttonMinimize = Instance.new("TextButton", panel)
	buttonMinimize.Size = UDim2.new(0,40,0,40)
	buttonMinimize.Position = UDim2.new(1,-45,0,5)
	buttonMinimize.Text = "-"
	buttonMinimize.TextColor3 = Color3.new(1,1,1)
	buttonMinimize.BackgroundColor3 = Color3.fromRGB(255,0,0)
	buttonMinimize.Font = Enum.Font.GothamBold
	buttonMinimize.TextSize = 22

	-- Bolinha
	ball = Instance.new("Frame", screenGui)
	ball.Size = UDim2.new(0,60,0,60)
	ball.Position = UDim2.new(0.1,0,0.5,0)
	ball.AnchorPoint = Vector2.new(0.5,0.5)
	ball.BackgroundColor3 = Color3.fromRGB(30,30,30)
	ball.Visible = false
	ball.Active = true
	ball.BorderSizePixel = 0

	local ballCorner = Instance.new("UICorner", ball)
	ballCorner.CornerRadius = UDim.new(1,0)

	local ballText = Instance.new("TextLabel", ball)
	ballText.Size = UDim2.new(1,0,1,0)
	ballText.BackgroundTransparency = 1
	ballText.Text = "Z"
	ballText.TextColor3 = Color3.new(1,0,0)
	ballText.Font = Enum.Font.GothamBold
	ballText.TextSize = 40

	-- Drag bolinha
	local dragging, dragInput, dragStart, startPos
	local function update(input)
		local delta = input.Position - dragStart
		ball.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
	ball.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			dragStart = input.Position
			startPos = ball.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then dragging = false end
			end)
		end
	end)
	ball.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
			dragInput = input
		end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if input == dragInput and dragging then update(input) end
	end)

	-- Minimizar
	buttonMinimize.MouseButton1Click:Connect(function()
		panel.Visible = false
		ball.Visible = true
	end)
	ball.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			panel.Visible = true
			ball.Visible = false
		end
	end)

	-- Layout para botões
	local layout = Instance.new("UIListLayout", panel)
	layout.Padding = UDim.new(0,6)
	layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
	layout.SortOrder = Enum.SortOrder.LayoutOrder

	local function createButton(text, callback)
		local btn = Instance.new("TextButton", panel)
		btn.Size = UDim2.new(0,380,0,40)
		btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
		btn.TextColor3 = Color3.new(1,1,1)
		btn.Text = text
		btn.Font = Enum.Font.GothamBold
		btn.TextSize = 20
		Instance.new("UICorner", btn).CornerRadius = UDim.new(0,8)
		btn.MouseButton1Click:Connect(callback)
		return btn
	end

	-- Botões
	local btnAimbot = createButton("Aimbot: OFF", function()
		aimbotEnabled = not aimbotEnabled
		btnAimbot.Text = "Aimbot: " .. (aimbotEnabled and "ON" or "OFF")
	end)

	local btnESP = createButton("ESP: OFF", function()
		espEnabled = not espEnabled
		btnESP.Text = "ESP: " .. (espEnabled and "ON" or "OFF")
		if not espEnabled then
			for _, hl in pairs(highlights) do hl:Destroy() end
			highlights = {}
		end
	end)

	local btnInvisible = createButton("Invisible: OFF", function()
		invisibleEnabled = not invisibleEnabled
		btnInvisible.Text = "Invisible: " .. (invisibleEnabled and "ON" or "OFF")
		if player.Character then
			for _, part in ipairs(player.Character:GetDescendants()) do
				if part:IsA("BasePart") then
					part.LocalTransparencyModifier = invisibleEnabled and 1 or 0
				end
			end
		end
	end)
end

-- ESP handler
local function applyESP(plr)
	if not espEnabled then return end
	if plr.Character and not highlights[plr] then
		local hl = Instance.new("Highlight")
		hl.FillTransparency = 1
		hl.OutlineColor = Color3.new(1,0,0)
		hl.Parent = plr.Character
		highlights[plr] = hl
	end
end

game.Players.PlayerAdded:Connect(function(plr)
	plr.CharacterAdded:Connect(function()
		task.wait(1)
		applyESP(plr)
	end)
end)

-- Aimbot
runService.RenderStepped:Connect(function()
	if aimbotEnabled and player.Character and player.Character:FindFirstChild("Head") then
		local cam = workspace.CurrentCamera
		local closest, dist = nil, math.huge
		for _, plr in ipairs(game.Players:GetPlayers()) do
			if plr ~= player and plr.Character and plr.Character:FindFirstChild("Head") then
				local mag = (plr.Character.Head.Position - player.Character.Head.Position).Magnitude
				if mag < dist then
					dist, closest = mag, plr
				end
			end
		end
		if closest then
			cam.CFrame = CFrame.new(cam.CFrame.Position, closest.Character.Head.Position)
		end
	end
end)

-- Sempre recriar UI ao respawn
player.CharacterAdded:Connect(function()
	task.wait(1)
	createUI()
end)

-- Primeira criação
createUI()
