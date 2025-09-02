-- LocalScript dentro de StarterPlayerScripts
local player = game.Players.LocalPlayer
local camera = workspace.CurrentCamera
local runService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

-- Variáveis de controle
local aimbotEnabled = false
local espEnabled = false
local walkTPEnabled = false
local walkTPTool
local walkSpeedValue = 16
local jumpPowerValue = 50

-- GUI Principal
local screenGui = Instance.new("ScreenGui", player.PlayerGui)
screenGui.Name = "ZARALHO V4"

-- Painel
local panel = Instance.new("Frame", screenGui)
panel.Size = UDim2.new(0, 420, 0, 360)
panel.Position = UDim2.new(0.5, -210, 0.5, -180)
panel.BackgroundColor3 = Color3.fromRGB(20,20,20)
panel.BorderSizePixel = 0

-- Borda RGB
local uiStroke = Instance.new("UIStroke", panel)
uiStroke.Thickness = 3
uiStroke.Color = Color3.new(1,0,0)

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

-- Botão Minimizar
local buttonMinimize = Instance.new("TextButton", panel)
buttonMinimize.Size = UDim2.new(0,40,0,40)
buttonMinimize.Position = UDim2.new(1,-45,0,5)
buttonMinimize.Text = "-"
buttonMinimize.TextColor3 = Color3.new(1,1,1)
buttonMinimize.BackgroundColor3 = Color3.fromRGB(255,0,0)
buttonMinimize.Font = Enum.Font.GothamBold
buttonMinimize.TextSize = 22

-- Bolinha de Minimizar
local ball = Instance.new("Frame", screenGui)
ball.Size = UDim2.new(0,60,0,60)
ball.Position = UDim2.new(0.1,0,0.5,0)
ball.AnchorPoint = Vector2.new(0.5,0.5)
ball.BackgroundColor3 = Color3.new(0,0,0)
ball.Visible = false
ball.ZIndex = 5
ball.BorderSizePixel = 0
ball.Name = "ZaralhaBall"

local ballText = Instance.new("TextLabel", ball)
ballText.Size = UDim2.new(1,0,1,0)
ballText.BackgroundTransparency = 1
ballText.Text = "Z"
ballText.TextColor3 = Color3.new(1,0,0)
ballText.Font = Enum.Font.GothamBold
ballText.TextSize = 40

-- Minimizar/Abrir
buttonMinimize.MouseButton1Click:Connect(function()
    panel.Visible = false
    ball.Visible = true
end)

-- Arrastar e clique da bolinha
local dragging = false
local dragStart, startPos
local moved = false

ball.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        moved = false
        dragStart = UIS:GetMouseLocation()
        startPos = ball.Position
    end
end)

UIS.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = UIS:GetMouseLocation() - dragStart
        if math.abs(delta.X) > 2 or math.abs(delta.Y) > 2 then
            moved = true
        end
        ball.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
        if not moved then
            panel.Visible = true
            ball.Visible = false
        end
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

-- Função criar input numérico
local function createInput(placeholder, default, order)
    local box = Instance.new("TextBox", panel)
    box.Size = UDim2.new(0,180,0,40)
    box.Position = UDim2.new(0,20 + (order*190),0,280)
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
local btnWalkTP = createButton("WalkTeleport: OFF",4)

-- Inputs WalkSpeed/JumpPower
local walkInput = createInput("WalkSpeed",16,0)
local jumpInput = createInput("JumpPower",50,1)

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
                local hl = Instance.new("Highlight")
                hl.FillTransparency = 1
                hl.OutlineColor = Color3.new(1,0,0)
                hl.Parent = plr.Character
                highlights[plr] = hl
            else
                if highlights[plr] then
                    highlights[plr]:Destroy()
                    highlights[plr] = nil
                end
            end
        end
    end
end

-- WalkTeleport Tool
local function toggleWalkTP(state)
    if state then
        walkTPTool = Instance.new("Tool")
        walkTPTool.RequiresHandle = false
        walkTPTool.Name = "WalkTeleport"
        walkTPTool.Parent = player.Backpack

        walkTPTool.Activated:Connect(function()
            local mouse = player:GetMouse()
            if mouse and mouse.Hit then
                local pos = mouse.Hit.p
                if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                    player.Character.HumanoidRootPart.CFrame = CFrame.new(pos + Vector3.new(0,3,0))
                end
            end
        end)
    else
        if walkTPTool then
            walkTPTool:Destroy()
            walkTPTool = nil
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
    walkTPEnabled = not walkTPEnabled
    btnWalkTP.Text = "WalkTeleport: " .. (walkTPEnabled and "ON" or "OFF")
    toggleWalkTP(walkTPEnabled)
end)

-- Aplicar valores WS/JP
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
