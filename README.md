-- 🌈 ZARALHO V6 - GUI UNIVERSAL (PC/Mobile/Delta)

-- Serviços
local player = game.Players.LocalPlayer
local runService = game:GetService("RunService")
local camera = game.Workspace.CurrentCamera
local uis = game:GetService("UserInputService")

-- GUI Principal
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "ZaralhoV6"
gui.ResetOnSpawn = false

local panel = Instance.new("Frame", gui)
panel.Size = UDim2.new(0, 500, 0, 420)
panel.Position = UDim2.new(0.5, -250, 0.5, -210)
panel.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
panel.Active = true
panel.Draggable = true
panel.Visible = true

local stroke = Instance.new("UIStroke", panel)
stroke.Thickness = 3
task.spawn(function()
    local hue = 0
    while task.wait() do
        hue = (hue + 0.01) % 1
        stroke.Color = Color3.fromHSV(hue, 1, 1)
    end
end)

local title = Instance.new("TextLabel", panel)
title.Size = UDim2.new(1, 0, 0, 40)
title.Text = "ZARALHO V6"
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 26

local tabs = Instance.new("Frame", panel)
tabs.Size = UDim2.new(1, 0, 0, 40)
tabs.Position = UDim2.new(0, 0, 0, 40)
tabs.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

local pages = {}
local currentPage = nil

local function createPage(name)
    local page = Instance.new("Frame", panel)
    page.Name = name
    page.Size = UDim2.new(1, -20, 1, -100)
    page.Position = UDim2.new(0, 10, 0, 90)
    page.BackgroundTransparency = 1
    page.Visible = false
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
    btn.Size = UDim2.new(0.25, 0, 1, 0)
    btn.Position = UDim2.new(0.25 * index, 0, 0, 0)
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

-- Criar abas
createTab("⚙️ Configurações", "Config", 0)
createTab("🎯 Combate", "Combat", 1)
createTab("🛠️ Ferramentas", "Tools", 2)
createTab("📂 Scripts", "Scripts", 3)

switchPage("Config")

-- Funções de criação
local function createButton(parent, text, order)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(0, 380, 0, 40)
    btn.Position = UDim2.new(0, 20, 0, order * 45)
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Text = text
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 20
    return btn
end

local function createInput(parent, placeholder, default, order)
    local box = Instance.new("TextBox", parent)
    box.Size = UDim2.new(0, 180, 0, 40)
    box.Position = UDim2.new(0, 20, 0, order * 45)
    box.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    box.TextColor3 = Color3.new(1, 1, 1)
    box.PlaceholderText = placeholder
    box.Text = tostring(default)
    box.Font = Enum.Font.GothamBold
    box.TextSize = 18
    return box
end

-- ⚙️ CONFIG
local walkInput = createInput(pages["Config"], "WalkSpeed", 16, 0)
local jumpInput = createInput(pages["Config"], "JumpPower", 50, 1)
local flySpeedInput = createInput(pages["Config"], "FlySpeed", 50, 2)

-- 🎯 COMBATE
local btnAimbot = createButton(pages["Combat"], "Aimbot: OFF", 0)
local btnESP = createButton(pages["Combat"], "ESP: OFF", 1)

-- 🛠️ FERRAMENTAS
local btnTP = createButton(pages["Tools"], "WalkTeleport Tool", 0)
local btnInvisible = createButton(pages["Tools"], "Invisibilidade: OFF", 1)
local btnFly = createButton(pages["Tools"], "Fly: OFF", 2)
local btnFarm = createButton(pages["Tools"], "AutoFarm: OFF", 3)
local btnCollect = createButton(pages["Tools"], "AutoColeta: OFF", 4)
local btnNoclip = createButton(pages["Tools"], "Noclip: OFF", 5)

-- 📂 SCRIPTS
local btnYield = createButton(pages["Scripts"], "Executar Infinite Yield", 0)
local btnPedro = createButton(pages["Scripts"], "Executar Pedroxz Menu", 1)

-- Variáveis de controle
local aimbotEnabled, espEnabled, invisibleEnabled = false, false, false
local autoFarmEnabled, autoCollectEnabled, flyEnabled, noclipEnabled = false, false, false, false
local flySpeed = 50
local flyingConn, noclipConn

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
        if plr ~= player and plr.Character then
            if state then
                local hl = Instance.new("Highlight", plr.Character)
                hl.FillTransparency = 1
                hl.OutlineColor = Color3.new(1, 0, 0)
            else
                for _, obj in ipairs(plr.Character:GetChildren()) do
                    if obj:IsA("Highlight") then obj:Destroy() end
                end
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

-- Fly
local function toggleFly()
    if flyEnabled then
        flyEnabled = false
        btnFly.Text = "Fly: OFF"
        if flyingConn then flyingConn:Disconnect() end
        if player.Character and player.Character:FindFirstChild("Humanoid") then
            player.Character.Humanoid.PlatformStand = false
        end
    else
        flyEnabled = true
        btnFly.Text = "Fly: ON"
        if player.Character and player.Character:FindFirstChild("Humanoid") then
            player.Character.Humanoid.PlatformStand = true
        end
        flyingConn = runService.RenderStepped:Connect(function()
            if flyEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = player.Character.HumanoidRootPart
                local moveDir = Vector3.zero
                if uis:IsKeyDown(Enum.KeyCode.W) then moveDir = moveDir + camera.CFrame.LookVector end
                if uis:IsKeyDown(Enum.KeyCode.S) then moveDir = moveDir - camera.CFrame.LookVector end
                if uis:IsKeyDown(Enum.KeyCode.A) then moveDir = moveDir - camera.CFrame.RightVector end
                if uis:IsKeyDown(Enum.KeyCode.D) then moveDir = moveDir + camera.CFrame.RightVector end
                if uis:IsKeyDown(Enum.KeyCode.Space) then moveDir = moveDir + Vector3.new(0,1,0) end
                if uis:IsKeyDown(Enum.KeyCode.LeftShift) then moveDir = moveDir - Vector3.new(0,1,0) end
                hrp.Velocity = moveDir.Unit * flySpeed
            end
        end)
    end
end

-- Noclip
local function toggleNoclip()
    if noclipEnabled then
        noclipEnabled = false
        btnNoclip.Text = "Noclip: OFF"
        if noclipConn then noclipConn:Disconnect() end
    else
        noclipEnabled = true
        btnNoclip.Text = "Noclip: ON"
        noclipConn = runService.Stepped:Connect(function()
            if player.Character then
                for _, part in ipairs(player.Character:GetDescendants()) do
                    if part:IsA("BasePart") and part.CanCollide == true then
                        part.CanCollide = false
                    end
                end
            end
        end)
    end
end

-- Teleporte Tool
btnTP.MouseButton1Click:Connect(function()
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

-- AutoFarm
runService.RenderStepped:Connect(function()
    if autoFarmEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local closest, dist = nil, math.huge
        for _, npc in ipairs(workspace:GetChildren()) do
            if npc.Name == "FarmNPC" and npc:FindFirstChild("HumanoidRootPart") then
                local mag = (npc.HumanoidRootPart.Position - player.Character.HumanoidRootPart.Position).Magnitude
                if mag < dist then
                    dist = mag
                    closest = npc
                end
            end
        end
        if closest then
            player.Character:MoveTo(closest.HumanoidRootPart.Position + Vector3.new(2, 0, 2))
        end
    end
end)

-- AutoColeta
runService.RenderStepped:Connect(function()
    if autoCollectEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        for _, obj in ipairs(workspace:GetChildren()) do
            if obj.Name == "Coletavel" and obj:IsA("BasePart") then
                local mag = (obj.Position - player.Character.HumanoidRootPart.Position).Magnitude
                if mag < 10 then
                    firetouchinterest(player.Character.HumanoidRootPart, obj, 0)
                    firetouchinterest(player.Character.HumanoidRootPart, obj, 1)
                end
            end
        end
    end
end)

-- Botões
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
    btnInvisible.Text = "Invisibilidade: " .. (invisibleEnabled and "ON" or "OFF")
    toggleInvisible(invisibleEnabled)
end)

btnFly.MouseButton1Click:Connect(toggleFly)
btnNoclip.MouseButton1Click:Connect(toggleNoclip)

btnFarm.MouseButton1Click:Connect(function()
    autoFarmEnabled = not autoFarmEnabled
    btnFarm.Text = "AutoFarm: " .. (autoFarmEnabled and "ON" or "OFF")
end)

btnCollect.MouseButton1Click:Connect(function()
    autoCollectEnabled = not autoCollectEnabled
    btnCollect.Text = "AutoColeta: " .. (autoCollectEnabled and "ON" or "OFF")
end)

btnYield.MouseButton1Click:Connect(function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source"))()
end)

btnPedro.MouseButton1Click:Connect(function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/Pedroxz63/PedroxzMenuAtualizado/refs/heads/main/pedroxzmenuv.2.0.2.md"))()
end)

-- Inputs
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

flySpeedInput.FocusLost:Connect(function()
    local val = tonumber(flySpeedInput.Text)
    if val then
        flySpeed = val
    end
end)

-- Bolinha RGB "Z"
local miniBtn = Instance.new("TextButton", gui)
miniBtn.Size = UDim2.new(0, 50, 0, 50)
miniBtn.Position = UDim2.new(0, 20, 0.5, -25)
miniBtn.Text = "Z"
miniBtn.Font = Enum.Font.GothamBold
miniBtn.TextSize = 24
miniBtn.TextColor3 = Color3.new(1,1,1)
miniBtn.BackgroundColor3 = Color3.fromRGB(30,30,30)
miniBtn.Active = true
miniBtn.Draggable = true

local miniStroke = Instance.new("UIStroke", miniBtn)
miniStroke.Thickness = 3
task.spawn(function()
    local hue = 0
    while task.wait() do
        hue = (hue + 0.01) % 1
        miniStroke.Color = Color3.fromHSV(hue,1,1)
    end
end)

miniBtn.MouseButton1Click:Connect(function()
    panel.Visible = not panel.Visible
end)

-- Anti-Kick básico
local mt = getrawmetatable(game)
local old = mt.__namecall
setreadonly(mt,false)
mt.__namecall = newcclosure(function(self,...)
    local method = getnamecallmethod()
    if method == "Kick" or method == "kick" then
        return nil
    end
    return old(self,...)
end)
setreadonly(mt,true)

end
end
