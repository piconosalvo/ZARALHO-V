-- ZARALHO V6 FINAL
-- GUI UNIVERSAL PARA DELTA
-- by ChatGPT & Você 🔥

local player = game.Players.LocalPlayer
local runService = game:GetService("RunService")
local camera = workspace.CurrentCamera

-- GUI PRINCIPAL
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "ZaralhoV6"
gui.ResetOnSpawn = false

-- Painel principal
local panel = Instance.new("Frame", gui)
panel.Size = UDim2.new(0, 500, 0, 420)
panel.Position = UDim2.new(0.5, -250, 0.5, -210)
panel.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
panel.Active = true
panel.Draggable = true

-- Fundo com imagem
local bg = Instance.new("ImageLabel", panel)
bg.Size = UDim2.new(1, 0, 1, 0)
bg.Image = "rbxassetid://7129155278"
bg.ImageTransparency = 0.2
bg.BackgroundTransparency = 1

-- Stroke RGB
local stroke = Instance.new("UIStroke", panel)
stroke.Thickness = 3
task.spawn(function()
    local hue = 0
    while task.wait() do
        hue = (hue + 0.01) % 1
        stroke.Color = Color3.fromHSV(hue, 1, 1)
    end
end)

-- Título
local title = Instance.new("TextLabel", panel)
title.Size = UDim2.new(1, 0, 0, 40)
title.Text = "🔥 ZARALHO V6 🔥"
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 28

-- Barra de abas
local tabs = Instance.new("Frame", panel)
tabs.Size = UDim2.new(1, 0, 0, 40)
tabs.Position = UDim2.new(0, 0, 0, 40)
tabs.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

-- Páginas
local pages = {}
local currentPage = nil

local function createPage(name)
    local page = Instance.new("ScrollingFrame", panel)
    page.Name = name
    page.Size = UDim2.new(1, -20, 1, -100)
    page.Position = UDim2.new(0, 10, 0, 90)
    page.BackgroundTransparency = 1
    page.Visible = false
    page.ScrollBarThickness = 6
    page.CanvasSize = UDim2.new(0,0,2,0)
    pages[name] = page
    return page
end

local function switchPage(name)
    for pageName, pageFrame in pairs(pages) do
        pageFrame.Visible = (pageName == name)
    end
    currentPage = pages[name]
end

local function createTab(label, pageName, index)
    local btn = Instance.new("TextButton", tabs)
    btn.Size = UDim2.new(0.2, 0, 1, 0)
    btn.Position = UDim2.new(0.2 * index, 0, 0, 0)
    btn.Text = label
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 18
    btn.MouseButton1Click:Connect(function()
        switchPage(pageName)
    end)
end

-- Criar páginas
local config = createPage("Config")
local combat = createPage("Combat")
local tools = createPage("Tools")
local scripts = createPage("Scripts")
local hub = createPage("ScriptHub")

-- Criar abas
createTab("⚙️ Config", "Config", 0)
createTab("🎯 Combat", "Combat", 1)
createTab("🛠️ Tools", "Tools", 2)
createTab("📂 Scripts", "Scripts", 3)
createTab("🌐 Hub", "ScriptHub", 4)

switchPage("Config")

-- Criação de botões e inputs
local function createButton(parent, text, order, callback)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(0, 380, 0, 40)
    btn.Position = UDim2.new(0, 20, 0, order * 45)
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Text = text
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 20
    btn.MouseButton1Click:Connect(callback)
    return btn
end

local function createInput(parent, placeholder, default, order, callback)
    local box = Instance.new("TextBox", parent)
    box.Size = UDim2.new(0, 180, 0, 40)
    box.Position = UDim2.new(0, 20, 0, order * 45)
    box.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    box.TextColor3 = Color3.new(1, 1, 1)
    box.PlaceholderText = placeholder
    box.Text = tostring(default)
    box.Font = Enum.Font.GothamBold
    box.TextSize = 18
    box.FocusLost:Connect(function()
        callback(tonumber(box.Text))
    end)
    return box
end
-- CONFIG: WalkSpeed e JumpPower
local walkInput = createInput(config, "WalkSpeed", 16, 0, function(val)
    if val and player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid.WalkSpeed = val
    end
end)

local jumpInput = createInput(config, "JumpPower", 50, 1, function(val)
    if val and player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid.JumpPower = val
    end
end)

-- Puxa Armas (Beta)
createButton(config, "📦 Puxar Armas (BETA)", 2, function()
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("Tool") then
            v.Parent = player.Backpack
        end
    end
end)

-- Variáveis de controle
local aimbotEnabled, espEnabled, invisibleEnabled = false, false, false
local autoFarmEnabled, autoCollectEnabled = false, false
local noclipEnabled, flyEnabled = false, false

-- AIMBOT
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

-- ESP melhorado
local function applyESP(plr)
    if plr ~= player and plr.Character then
        if not plr.Character:FindFirstChild("ZARALHO_ESP") then
            local hl = Instance.new("Highlight")
            hl.Name = "ZARALHO_ESP"
            hl.FillTransparency = 1
            hl.OutlineColor = Color3.fromRGB(255, 0, 0)
            hl.OutlineTransparency = 0
            hl.Parent = plr.Character
        end
    end
end

local function toggleESP(state)
    espEnabled = state
    if espEnabled then
        for _, plr in ipairs(game.Players:GetPlayers()) do
            applyESP(plr)
        end
        game.Players.PlayerAdded:Connect(applyESP)
    else
        for _, plr in ipairs(game.Players:GetPlayers()) do
            if plr.Character and plr.Character:FindFirstChild("ZARALHO_ESP") then
                plr.Character.ZARALHO_ESP:Destroy()
            end
        end
    end
end

-- Invisibilidade
local function toggleInvisible(state)
    if player.Character then
        for _, part in ipairs(player.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.LocalTransparencyModifier = state and 1 or 0
            end
        end
    end
end

-- Noclip
runService.Stepped:Connect(function()
    if noclipEnabled and player.Character then
        for _, part in ipairs(player.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end)

-- Fly (BETA by Me_OzoneYT adaptado)
local flySpeed = 3
local flyControl = {w=false,a=false,s=false,d=false,q=false,e=false}
local bodyVel, bodyGyro

local function startFly()
    local hrp = player.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    bodyVel = Instance.new("BodyVelocity")
    bodyVel.Velocity = Vector3.zero
    bodyVel.MaxForce = Vector3.new(4000,4000,4000)
    bodyVel.Parent = hrp

    bodyGyro = Instance.new("BodyGyro")
    bodyGyro.MaxTorque = Vector3.new(4000,4000,4000)
    bodyGyro.CFrame = hrp.CFrame
    bodyGyro.P = 10000
    bodyGyro.Parent = hrp

    flyEnabled = true
end

local function stopFly()
    flyEnabled = false
    if bodyVel then bodyVel:Destroy() end
    if bodyGyro then bodyGyro:Destroy() end
end

game:GetService("UserInputService").InputBegan:Connect(function(input,gpe)
    if gpe then return end
    local key = input.KeyCode
    if key == Enum.KeyCode.W then flyControl.w=true end
    if key == Enum.KeyCode.S then flyControl.s=true end
    if key == Enum.KeyCode.A then flyControl.a=true end
    if key == Enum.KeyCode.D then flyControl.d=true end
    if key == Enum.KeyCode.E then flyControl.e=true end
    if key == Enum.KeyCode.Q then flyControl.q=true end
end)

game:GetService("UserInputService").InputEnded:Connect(function(input,gpe)
    local key = input.KeyCode
    if key == Enum.KeyCode.W then flyControl.w=false end
    if key == Enum.KeyCode.S then flyControl.s=false end
    if key == Enum.KeyCode.A then flyControl.a=false end
    if key == Enum.KeyCode.D then flyControl.d=false end
    if key == Enum.KeyCode.E then flyControl.e=false end
    if key == Enum.KeyCode.Q then flyControl.q=false end
end)

runService.RenderStepped:Connect(function()
    if flyEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local hrp = player.Character.HumanoidRootPart
        local camCF = workspace.CurrentCamera.CFrame
        local move = Vector3.zero
        if flyControl.w then move = move + camCF.LookVector end
        if flyControl.s then move = move - camCF.LookVector end
        if flyControl.a then move = move - camCF.RightVector end
        if flyControl.d then move = move + camCF.RightVector end
        if flyControl.e then move = move + Vector3.new(0,1,0) end
        if flyControl.q then move = move - Vector3.new(0,1,0) end
        bodyVel.Velocity = move * flySpeed
        bodyGyro.CFrame = camCF
    end
end)
-- BOTÕES DE COMBATE
local btnAimbot = createButton(combat, "🎯 Aimbot: OFF", 0, function()
    aimbotEnabled = not aimbotEnabled
    btnAimbot.Text = "🎯 Aimbot: " .. (aimbotEnabled and "ON" or "OFF")
end)

local btnESP = createButton(combat, "👁️ ESP: OFF", 1, function()
    toggleESP(not espEnabled)
    btnESP.Text = "👁️ ESP: " .. (espEnabled and "ON" or "OFF")
end)

-- BOTÕES DE TOOLS
local btnInvisible = createButton(tools, "👻 Invisibilidade: OFF", 0, function()
    invisibleEnabled = not invisibleEnabled
    toggleInvisible(invisibleEnabled)
    btnInvisible.Text = "👻 Invisibilidade: " .. (invisibleEnabled and "ON" or "OFF")
end)

local btnFarm = createButton(tools, "🌾 AutoFarm: OFF", 1, function()
    autoFarmEnabled = not autoFarmEnabled
    btnFarm.Text = "🌾 AutoFarm: " .. (autoFarmEnabled and "ON" or "OFF")
end)

local btnCollect = createButton(tools, "📦 AutoColeta: OFF", 2, function()
    autoCollectEnabled = not autoCollectEnabled
    btnCollect.Text = "📦 AutoColeta: " .. (autoCollectEnabled and "ON" or "OFF")
end)

local btnNoclip = createButton(tools, "🚪 Noclip: OFF", 3, function()
    noclipEnabled = not noclipEnabled
    btnNoclip.Text = "🚪 Noclip: " .. (noclipEnabled and "ON" or "OFF")
end)

local btnFly = createButton(tools, "✈️ FLY (BETA): OFF", 4, function()
    if flyEnabled then
        stopFly()
        btnFly.Text = "✈️ FLY (BETA): OFF"
    else
        startFly()
        btnFly.Text = "✈️ FLY (BETA): ON"
    end
end)

-- BOTÕES DE SCRIPTS
createButton(scripts, "⚡ Infinite Yield", 0, function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source"))()
end)

createButton(scripts, "🛠️ Pedroxz Menu", 1, function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/Pedroxz63/PedroxzMenuAtualizado/refs/heads/main/pedroxzmenuv.2.0.2.md"))()
end)

-- SCRIPT HUB (Brookhaven, Blox Fruits, Ink, Fly especial)
createButton(hub, "🏡 Brookhaven GUI 1", 0, function()
    loadstring(game:HttpGet("https://pastebin.com/raw/0xTtC3uK"))()
end)

createButton(hub, "🏡 Brookhaven GUI 2", 1, function()
    loadstring(game:HttpGet("https://pastebin.com/raw/gb8rG6LV"))()
end)

createButton(hub, "🏡 Brookhaven GUI 3", 2, function()
    loadstring(game:HttpGet("https://pastebin.com/raw/QJdQ6G6U"))()
end)

createButton(hub, "🍏 Blox Fruits GUI 1", 3, function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/Project-BloxFruit/Scripts/main/BF1.lua"))()
end)

createButton(hub, "🍏 Blox Fruits GUI 2", 4, function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/Project-BloxFruit/Scripts/main/BF2.lua"))()
end)

createButton(hub, "🍏 Blox Fruits GUI 3", 5, function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/Project-BloxFruit/Scripts/main/BF3.lua"))()
end)

createButton(hub, "🖌️ Ink Game GUI 1", 6, function()
    loadstring(game:HttpGet("https://pastebin.com/raw/ZXcvInk1"))()
end)

createButton(hub, "🖌️ Ink Game GUI 2", 7, function()
    loadstring(game:HttpGet("https://pastebin.com/raw/ZXcvInk2"))()
end)

createButton(hub, "🖌️ Ink Game GUI 3", 8, function()
    loadstring(game:HttpGet("https://pastebin.com/raw/ZXcvInk3"))()
end)

createButton(hub, "✈️ Fly GUI Especial", 9, function()
    loadstring(game:HttpGet("https://pastebin.com/raw/1FLYozoneYT"))()
end)

-- ANTI-KICK
for _, v in pairs(getconnections(player.Idled)) do
    v:Disable()
end
player.CharacterAdded:Connect(function(char)
    char:WaitForChild("Humanoid").Changed:Connect(function()
        pcall(function()
            player.Character.Humanoid:ChangeState(11)
        end)
    end)
end)

-- BOTÃO DE MINIMIZAR
local miniBtn = Instance.new("TextButton", gui)
miniBtn.Size = UDim2.new(0,50,0,50)
miniBtn.Position = UDim2.new(0,20,0.5,-25)
miniBtn.BackgroundColor3 = Color3.fromRGB(30,30,30)
miniBtn.Text = "Z"
miniBtn.Font = Enum.Font.GothamBold
miniBtn.TextSize = 22
miniBtn.TextColor3 = Color3.new(1,1,1)
miniBtn.Active = true
miniBtn.Draggable = true

-- Stroke RGB na bolinha
local miniStroke = Instance.new("UIStroke", miniBtn)
miniStroke.Thickness = 2
task.spawn(function()
    local hue = 0
    while task.wait() do
        hue = (hue + 0.01) % 1
        miniStroke.Color = Color3.fromHSV(hue, 1, 1)
    end
end)

local minimized = false
miniBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    panel.Visible = not minimized
end)
