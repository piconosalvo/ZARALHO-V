-- LocalScript dentro de StarterPlayerScripts
local player = game.Players.LocalPlayer
local camera = workspace.CurrentCamera
local runService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- Variáveis de controle
local aimbotEnabled = false
local espEnabled = false
local invisibleEnabled = false

-- GUI Principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ZaralhoV5"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Painel
local panel = Instance.new("Frame", screenGui)
panel.Size = UDim2.new(0, 420, 0, 400)
panel.Position = UDim2.new(0.5, -210, 0.5, -200)
panel.BackgroundColor3 = Color3.fromRGB(20,20,20)
panel.BorderSizePixel = 0
panel.Visible = true
panel.Active = true
panel.Draggable = true

-- Borda RGB
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
title.Text = "ZARALHO V5"
title.TextColor3 = Color3.new(1,1,1)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 24

-- Botão Minimizar
local buttonMinimize = Instance.new("TextButton", panel)
buttonMinimize.Size = UDim2.new(0,40,0,40)
buttonMinimize.Position = UDim2.new(1,-45,0,5)
buttonMinimize.Text = "-"
buttonMinimize.TextColor3 = Color3.new(1,1,1)
buttonMinimize.BackgroundColor3 = Color3.fromRGB(255,0,0)
buttonMinimize.Font = Enum.Font.GothamBold
buttonMinimize.TextSize = 22
buttonMinimize.ZIndex = 10

-- Bolinha
local ball = Instance.new("Frame", screenGui)
ball.Size = UDim2.new(0,60,0,60)
ball.Position = UDim2.new(0.1,0,0.5,0)
ball.AnchorPoint = Vector2.new(0.5,0.5)
ball.BackgroundColor3 = Color3.new(0,0,0)
ball.Visible = false
ball.ZIndex = 20
ball.Active = true
ball.BorderSizePixel = 0
ball.Name = "ZaralhaBall"

local ballText = Instance.new("TextLabel", ball)
ballText.Size = UDim2.new(1,0,1,0)
ballText.BackgroundTransparency = 1
ballText.Text = "Z"
ballText.TextColor3 = Color3.new(1,0,0)
ballText.Font = Enum.Font.GothamBold
ballText.TextSize = 40
ballText.ZIndex = 21

-- Drag da bolinha
local dragging = false
local dragInput, dragStart, startPos
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
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)
ball.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        update(input)
    end
end)

-- Minimizar/Abrir
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

-- Atalho PC: tecla Z
UserInputService.InputBegan:Connect(function(input, gpe)
    if not gpe and input.KeyCode == Enum.KeyCode.Z then
        panel.Visible = not panel.Visible
        ball.Visible = not panel.Visible
    end
end)

-- Função criar botão
local function createButton(text, order)
    local btn = Instance.new("TextButton", panel)
    btn.Size = UDim2.new(0,380,0,40)
    btn.Position = UDim2.new(0,20,0,50 + (order*45))
    btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Text = text
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 20
    return btn
end

-- Função criar input
local function createInput(placeholder, default, order)
    local box = Instance.new("TextBox", panel)
    box.Size = UDim2.new(0,180,0,40)
    box.Position = UDim2.new(0,20,0,50 + (order*45))
    box.BackgroundColor3 = Color3.fromRGB(40,40,40)
    box.TextColor3 = Color3.new(1,1,1)
    box.PlaceholderText = placeholder
    box.Text = tostring(default)
    box.Font = Enum.Font.GothamBold
    box.TextSize = 18
    return box
end

-- Botões
local btnAimbot = createButton("Aimbot: OFF",0)
local btnESP = createButton("ESP: OFF",1)
local btnInfinite = createButton("Executar Infinite Yield",2)
local btnPedro = createButton("Executar Pedroxz Menu",3)
local btnWalkTP = createButton("WalkTeleport Tool",4)
local btnInvisible = createButton("WalkInvisible: OFF",5)

-- Inputs WalkSpeed / JumpPower
local walkInput = createInput("WalkSpeed",16,6)
local jumpInput = createInput("JumpPower",50,7)

-- Aimbot inteligente
runService.RenderStepped:Connect(function()
    if aimbotEnabled and player.Character and player.Character:FindFirstChild("Head") then
        local closest, dist = nil, math.huge
        for _, plr in ipairs(game.Players:GetPlayers()) do
            if plr ~= player and plr.Character and plr.Character:FindFirstChild("Head") then
                local mag = (plr.Character.Head.Position - player.Character.Head.Position).Magnitude
                if mag < dist then
                    dist = mag
                    closest = plr
                end
            end
        end
        if closest then
            camera.CFrame = CFrame.new(camera.CFrame.Position, closest.Character.Head.Position)
        end
    end
end)

-- ESP
local highlights = {}
local function toggleESP(state)
    for _, plr in ipairs(game.Players:GetPlayers()) do
        if plr ~= player then
            if state then
                if not highlights[plr] then
                    local hl = Instance.new("Highlight")
                    hl.FillTransparency = 1
                    hl.OutlineColor = Color3.new(1,0,0)
                    hl.Parent = plr.Character
                    highlights[plr] = hl
                end
            else
                if highlights[plr] then
                    highlights[plr]:Destroy()
                    highlights[plr] = nil
                end
            end
        end
    end
end
game.Players.PlayerAdded:Connect(function(plr)
    if espEnabled then
        plr.CharacterAdded:Connect(function(char)
            local hl = Instance.new("Highlight")
            hl.FillTransparency = 1
            hl.OutlineColor = Color3.new(1,0,0)
            hl.Parent = char
            highlights[plr] = hl
        end)
    end
end)

-- Invisível
local function toggleInvisible(state)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        for _, part in ipairs(player.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.LocalTransparencyModifier = state and 1 or 0
            end
        end
    end
end

-- Botões eventos
btnAimbot.MouseButton1Click:Connect(function()
    aimbotEnabled = not aimbotEnabled
    btnAimbot.Text = "Aimbot: " .. (aimbotEnabled and "ON" or "OFF")
end)
btnESP.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    btnESP.Text = "ESP: " .. (espEnabled and "ON" or "OFF")
    toggleESP(espEnabled)
end)
btnInfinite.MouseButton1Click:Connect(function()
    loadstring(game:HttpGet('https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source'))()
end)
btnPedro.MouseButton1Click:Connect(function()
    loadstring(game:HttpGet('https://raw.githubusercontent.com/Pedroxz63/PedroxzMenuAtualizado/refs/heads/main/pedroxzmenuv.2.0.2.md'))()
end)
btnWalkTP.MouseButton1Click:Connect(function()
    local tool = Instance.new("Tool")
    tool.RequiresHandle = false
    tool.Name = "WalkTeleport"
    tool.Parent = player.Backpack
    tool.Activated:Connect(function()
        local mouse = player:GetMouse()
        if mouse.Hit then
            player.Character:MoveTo(mouse.Hit.p)
        end
    end)
end)
btnInvisible.MouseButton1Click:Connect(function()
    invisibleEnabled = not invisibleEnabled
    btnInvisible.Text = "WalkInvisible: " .. (invisibleEnabled and "ON" or "OFF")
    toggleInvisible(invisibleEnabled)
end)

-- WalkSpeed / JumpPower aplicar
walkInput.FocusLost:Connect(function()
    local val = tonumber(walkInput.Text)
    if val and player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid.WalkSpeed = val
    end
end)
jumpInput.FocusLost:Connect(function()
    local val = tonumber(jumpInput.Text)
    if val and player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid.JumpPower = val
    end
end)
