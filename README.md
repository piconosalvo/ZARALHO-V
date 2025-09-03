-- [Zaralho V6 Final Estilizado]

local player = game.Players.LocalPlayer
local runService = game:GetService("RunService")
local camera = game.Workspace.CurrentCamera

-- Anti-Kick
local mt = getrawmetatable(game)
local oldNamecall = mt.__namecall
setreadonly(mt, false)
mt.__namecall = newcclosure(function(self, ...)
    if getnamecallmethod() == "Kick" then
        return nil
    end
    return oldNamecall(self, ...)
end)
setreadonly(mt, true)

-- GUI Principal
local gui = Instance.new("ScreenGui")
gui.Name = "ZaralhoV6"
gui.ResetOnSpawn = false
if syn and syn.protect_gui then
    syn.protect_gui(gui)
    gui.Parent = game.CoreGui
else
    gui.Parent = player:WaitForChild("PlayerGui")
end

-- Painel
local panel = Instance.new("Frame", gui)
panel.Size = UDim2.new(0, 550, 0, 450)
panel.Position = UDim2.new(0.5, -275, 0.5, -225)
panel.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
panel.Active = true
panel.Draggable = true
panel.Visible = true

local bg = Instance.new("ImageLabel", panel)
bg.Size = UDim2.new(1, 0, 1, 0)
bg.Image = "rbxassetid://7129155278"
bg.ImageTransparency = 0.25
bg.BackgroundTransparency = 1

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
title.Text = "⚡ ZARALHO V6 ⚡"
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 28

-- Botão Minimizar
local minimizeBtn = Instance.new("TextButton", gui)
minimizeBtn.Size = UDim2.new(0, 50, 0, 50)
minimizeBtn.Position = UDim2.new(0, 20, 0.5, -25)
minimizeBtn.Text = "Z"
minimizeBtn.TextSize = 24
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.TextColor3 = Color3.new(1, 1, 1)
minimizeBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

local minStroke = Instance.new("UIStroke", minimizeBtn)
minStroke.Thickness = 3
task.spawn(function()
    local hue = 0
    while task.wait() do
        hue = (hue + 0.01) % 1
        minStroke.Color = Color3.fromHSV(hue, 1, 1)
    end
end)

local minimized = false
minimizeBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    panel.Visible = not minimized
end)

-- Abas
local tabs = Instance.new("Frame", panel)
tabs.Size = UDim2.new(1, 0, 0, 40)
tabs.Position = UDim2.new(0, 0, 0, 40)
tabs.BackgroundColor3 = Color3.fromRGB(25, 25, 25)

local pages = {}
local function createPage(name)
    local page = Instance.new("Frame", panel)
    page.Size = UDim2.new(1, -20, 1, -100)
    page.Position = UDim2.new(0, 10, 0, 90)
    page.BackgroundTransparency = 1
    page.Visible = false
    pages[name] = page
    return page
end
local function switchPage(name)
    for p, f in pairs(pages) do
        f.Visible = (p == name)
    end
end
local function createTab(label, name, index)
    local btn = Instance.new("TextButton", tabs)
    btn.Size = UDim2.new(0.25, 0, 1, 0)
    btn.Position = UDim2.new(0.25 * index, 0, 0, 0)
    btn.Text = label
    btn.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 18

    local bStroke = Instance.new("UIStroke", btn)
    bStroke.Thickness = 2
    task.spawn(function()
        local hue = 0
        while task.wait() do
            hue = (hue + 0.01) % 1
            bStroke.Color = Color3.fromHSV(hue, 1, 1)
        end
    end)

    btn.MouseButton1Click:Connect(function()
        switchPage(name)
    end)
end

-- Páginas
local config = createPage("Config")
local combat = createPage("Combat")
local tools = createPage("Tools")
local hub = createPage("Hub")

createTab("⚙️ Configurações", "Config", 0)
createTab("🎯 Combate", "Combat", 1)
createTab("🛠️ Ferramentas", "Tools", 2)
createTab("🌐 Script Hub", "Hub", 3)
switchPage("Config")

-- Funções Config
local function createButton(parent, text, order, callback)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(0, 400, 0, 40)
    btn.Position = UDim2.new(0, 20, 0, order * 50)
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Text = text
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 18
    btn.MouseButton1Click:Connect(callback)
    return btn
end
local function createInput(parent, placeholder, default, order, callback)
    local box = Instance.new("TextBox", parent)
    box.Size = UDim2.new(0, 180, 0, 40)
    box.Position = UDim2.new(0, 20, 0, order * 50)
    box.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    box.TextColor3 = Color3.new(1, 1, 1)
    box.PlaceholderText = placeholder
    box.Text = tostring(default)
    box.Font = Enum.Font.GothamBold
    box.TextSize = 18
    box.FocusLost:Connect(function() callback(box.Text) end)
    return box
end

-- Variáveis
local aimbotEnabled, espEnabled, invisibleEnabled = false, false, false
local flyEnabled, noclipEnabled = false, false
local autoFarmEnabled, autoCollectEnabled = false, false

-- Inputs
createInput(config, "WalkSpeed", 16, 0, function(val)
    local n = tonumber(val)
    if n and player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid.WalkSpeed = n
    end
end)
createInput(config, "JumpPower", 50, 1, function(val)
    local n = tonumber(val)
    if n and player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid.JumpPower = n
    end
end)
createButton(config, "Puxa Armas (BETA)", 2, function()
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("Tool") then
            v.Parent = player.Backpack
        end
    end
end)

-- Combate
createButton(combat, "Aimbot: OFF", 0, function(btn)
    aimbotEnabled = not aimbotEnabled
    btn.Text = "Aimbot: " .. (aimbotEnabled and "ON" or "OFF")
end)
createButton(combat, "ESP: OFF", 1, function(btn)
    espEnabled = not espEnabled
    btn.Text = "ESP: " .. (espEnabled and "ON" or "OFF")
end)

-- Ferramentas
createButton(tools, "WalkTeleport Tool", 0, function()
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
createButton(tools, "Fly (BETA): OFF", 1, function(btn)
    flyEnabled = not flyEnabled
    btn.Text = "Fly (BETA): " .. (flyEnabled and "ON" or "OFF")
end)
createButton(tools, "Noclip: OFF", 2, function(btn)
    noclipEnabled = not noclipEnabled
    btn.Text = "Noclip: " .. (noclipEnabled and "ON" or "OFF")
end)
createButton(tools, "AutoFarm: OFF", 3, function(btn)
    autoFarmEnabled = not autoFarmEnabled
    btn.Text = "AutoFarm: " .. (autoFarmEnabled and "ON" or "OFF")
end)
createButton(tools, "AutoColeta: OFF", 4, function(btn)
    autoCollectEnabled = not autoCollectEnabled
    btn.Text = "AutoColeta: " .. (autoCollectEnabled and "ON" or "OFF")
end)
createButton(tools, "Invisibilidade: OFF", 5, function(btn)
    invisibleEnabled = not invisibleEnabled
    btn.Text = "Invisibilidade: " .. (invisibleEnabled and "ON" or "OFF")
end)

-- Script Hub
local function createHubButton(parent, text, url, order)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(0, 400, 0, 40)
    btn.Position = UDim2.new(0, 20, 0, order * 50)
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Text = text
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 18
    btn.MouseButton1Click:Connect(function()
        loadstring(game:HttpGet(url))()
    end)
end

local brookPage = Instance.new("Frame", hub)
brookPage.Size = UDim2.new(1, -20, 1, -100)
brookPage.Position = UDim2.new(0, 10, 0, 50)
brookPage.BackgroundTransparency = 1

createHubButton(brookPage, "Brookhaven Script 1", "https://pastebin.com/raw/xxxxx1", 0)
-- repete até 7 scripts...

local inkPage = Instance.new("Frame", hub)
inkPage.Size = UDim2.new(1, -20, 1, -100)
inkPage.Position = UDim2.new(0, 10, 0, 50)
inkPage.BackgroundTransparency = 1
inkPage.Visible = false

local bloxPage = Instance.new("Frame", hub)
bloxPage.Size = UDim2.new(1, -20, 1, -100)
bloxPage.Position = UDim2.new(0, 10, 0, 50)
bloxPage.BackgroundTransparency = 1
bloxPage.Visible = false

local hubTabs = Instance.new("Frame", hub)
hubTabs.Size = UDim2.new(1, 0, 0, 40)
hubTabs.Position = UDim2.new(0, 0, 0, 0)
hubTabs.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

local function createHubTab(label, pageRef, index)
    local btn = Instance.new("TextButton", hubTabs)
    btn.Size = UDim2.new(0.33, 0, 1, 0)
    btn.Position = UDim2.new(0.33 * index, 0, 0, 0)
    btn.Text = label
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 16
    btn.MouseButton1Click:Connect(function()
        brookPage.Visible, inkPage.Visible, bloxPage.Visible = false, false, false
        pageRef.Visible = true
    end)
end

createHubTab("Brookhaven", brookPage, 0)
createHubTab("Ink Game", inkPage, 1)
createHubTab("Blox Fruits", bloxPage, 2)

print("✅ Zaralho V6 Final Estilizado carregado com sucesso!")
