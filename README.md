-- LocalScript dentro de StarterPlayerScripts
local player = game.Players.LocalPlayer
local camera = workspace.CurrentCamera
local runService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- Variáveis
local aimbotEnabled, espEnabled, invisibleEnabled = false, false, false
local highlights = {}

-- GUI Principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ZaralhoV5"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Painel
local panel = Instance.new("Frame", screenGui)
panel.Size = UDim2.new(0, 460, 0, 420)
panel.Position = UDim2.new(0.5, -230, 0.5, -210)
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

-- Bolinha (abre/fecha)
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

-- Arrastar a bolinha
local dragging, dragInput, dragStart, startPos = false
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
UserInputService.InputBegan:Connect(function(input, gpe)
    if not gpe and input.KeyCode == Enum.KeyCode.Z then
        panel.Visible = not panel.Visible
        ball.Visible = not panel.Visible
    end
end)

-- Criador de botões
local function createButton(tab, text, order)
    local btn = Instance.new("TextButton", tab)
    btn.Size = UDim2.new(0,400,0,40)
    btn.Position = UDim2.new(0,30,0,20 + (order*45))
    btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Text = text
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 20
    return btn
end

-- Criador de inputs
local function createInput(tab, placeholder, default, order)
    local box = Instance.new("TextBox", tab)
    box.Size = UDim2.new(0,180,0,40)
    box.Position = UDim2.new(0,30,0,20 + (order*45))
    box.BackgroundColor3 = Color3.fromRGB(40,40,40)
    box.TextColor3 = Color3.new(1,1,1)
    box.PlaceholderText = placeholder
    box.Text = tostring(default)
    box.Font = Enum.Font.GothamBold
    box.TextSize = 18
    return box
end

-- Criando Tabs
local tabsFrame = Instance.new("Frame", panel)
tabsFrame.Size = UDim2.new(1,0,0,30)
tabsFrame.Position = UDim2.new(0,0,0,40)
tabsFrame.BackgroundTransparency = 1

local tabButtons = {}
local currentTab = nil
local tabPages = {}

local function createTab(name, order)
    local tabBtn = Instance.new("TextButton", tabsFrame)
    tabBtn.Size = UDim2.new(0,140,1,0)
    tabBtn.Position = UDim2.new(0,(order*150),0,0)
    tabBtn.Text = name
    tabBtn.BackgroundColor3 = Color3.fromRGB(30,30,30)
    tabBtn.TextColor3 = Color3.new(1,1,1)
    tabBtn.Font = Enum.Font.GothamBold
    tabBtn.TextSize = 18

    local page = Instance.new("Frame", panel)
    page.Size = UDim2.new(1,0,1,-70)
    page.Position = UDim2.new(0,0,0,70)
    page.BackgroundTransparency = 1
    page.Visible = false

    tabBtn.MouseButton1Click:Connect(function()
        if currentTab then currentTab.Visible = false end
        page.Visible = true
        currentTab = page
    end)

    if not currentTab then
        page.Visible = true
        currentTab = page
    end

    table.insert(tabPages,page)
    return page
end

-- Abas
local tabFuncoes = createTab("⚡ Funções",0)
local tabUtils = createTab("🛠️ Utilitários",1)
local tabPlayer = createTab("🎮 Player",2)

-------------------------
-- ⚡ Funções
-------------------------
local btnAimbot = createButton(tabFuncoes,"Aimbot: OFF",0)
local btnESP = createButton(tabFuncoes,"ESP: OFF",1)
local btnInvisible = createButton(tabFuncoes,"WalkInvisible: OFF",2)
local btnWalkTP = createButton(tabFuncoes,"WalkTeleport Tool",3)

-------------------------
-- 🛠️ Utilitários
-------------------------
local btnInfinite = createButton(tabUtils,"Executar Infinite Yield",0)
local btnPedro = createButton(tabUtils,"Executar Pedroxz Menu",1)

-------------------------
-- 🎮 Player
-------------------------
local walkInput = createInput(tabPlayer,"WalkSpeed",16,0)
local jumpInput = createInput(tabPlayer,"JumpPower",50,1)

----------------------------------------------------------------
-- 📌 Funções
----------------------------------------------------------------
-- Aimbot
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

----------------------------------------------------------------
-- 📌 Eventos Botões
----------------------------------------------------------------
btnAimbot.MouseButton1Click:Connect(function()
    aimbotEnabled = not aimbotEnabled
    btnAimbot.Text = "Aimbot: " .. (aimbotEnabled and "ON" or "OFF")
end)

btnESP.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    btnESP.Text = "ESP: " .. (espEnabled and "ON" or "OFF")
    toggleESP(espEnabled)
end)

btnInvisible.MouseButton1Click:Connect(function()
    invisibleEnabled = not invisibleEnabled
    btnInvisible.Text = "WalkInvisible: " .. (invisibleEnabled and "ON" or "OFF")
    toggleInvisible(invisibleEnabled)
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

btnInfinite.MouseButton1Click:Connect(function()
    loadstring(game:HttpGet('https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source'))()
end)

btnPedro.MouseButton1Click:Connect(function()
    loadstring(game:HttpGet('https://raw.githubusercontent.com/Pedroxz63/PedroxzMenuAtualizado/refs/heads/main/pedroxzmenuv.2.0.2.md'))()
end)

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
