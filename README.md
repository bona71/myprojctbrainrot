--[[ 🧠 AZARO HUB: Menu RGB Interativo com Três Abas + ESP Brain do Watch
Feito por: bona 💀 | Menu RGB e Tabs por ChatGPT | ESP Brain por bona71
]]

-- Serviços
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Hum = Char:WaitForChild("Humanoid")
local HRP = Char:WaitForChild("HumanoidRootPart")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")

-- Configurações
local SPEED = 35
local HIGH_JUMP = 100
local SKY_Y = 150
local NORMAL_SPEED = 16
local PlaceId = game.PlaceId

-- Estados
local toggles = { Speed = false, HighJump = false, Steal = false, ESP = false, BaseESP = false, ["ESP Brain do Watch"] = false }
local stealPlatform
local boosted = false
local speedMultiplier = 1.55

-- Variáveis do ESP Brain do Watch
local activeLockTimeEsp = false
local lteInstances = {}
local plotName = nil -- nome do plot do jogador

do -- Detecta o nome do plot do jogador, se existir
    local success, result = pcall(function()
        for _, plot in pairs(workspace.Plots:GetChildren()) do
            if plot.Owner and plot.Owner.Value == LocalPlayer then
                plotName = plot.Name
                break
            end
        end
    end)
end

-- Função do ESP Brain do Watch
local function updatelock()
    if not activeLockTimeEsp then
        for _, instance in pairs(lteInstances) do
            if instance then instance:Destroy() end
        end
        lteInstances = {}
        return
    end

    for _, plot in pairs(workspace.Plots:GetChildren()) do
        local billboardName = "LockTimeESP_" .. plot.Name
        local timeLabel = plot:FindFirstChild("Purchases", true)
            and plot.Purchases:FindFirstChild("PlotBlock", true)
            and plot.Purchases.PlotBlock.Main:FindFirstChild("BillboardGui", true)
            and plot.Purchases.PlotBlock.Main.BillboardGui:FindFirstChild("RemainingTime", true)

        if timeLabel and timeLabel:IsA("TextLabel") then
            local existing = lteInstances[plot.Name]
            local isUnlocked = timeLabel.Text == "0s"
            local displayText = isUnlocked and "Unlocked" or ("Lock: " .. timeLabel.Text)

            local color = Color3.fromRGB(255, 255, 0) -- amarelo padrão
            if plot.Name == plotName then
                color = Color3.fromRGB(0, 255, 0) -- verde para seu plot
            elseif isUnlocked then
                color = Color3.fromRGB(255, 0, 0) -- vermelho se desbloqueado
            end

            if not existing then
                local gui = Instance.new("BillboardGui")
                gui.Name = billboardName
                gui.Size = UDim2.new(0, 120, 0, 25)
                gui.StudsOffset = Vector3.new(0, 4, 0)
                gui.AlwaysOnTop = true
                gui.Adornee = plot.Purchases.PlotBlock.Main
                gui.Parent = plot

                local label = Instance.new("TextLabel")
                label.Size = UDim2.new(1, 0, 1, 0)
                label.BackgroundTransparency = 1
                label.TextScaled = true
                label.TextColor3 = color
                label.TextStrokeTransparency = 0
                label.TextStrokeColor3 = Color3.new(0, 0, 0)
                label.Font = Enum.Font.SourceSansBold
                label.Text = displayText
                label.Parent = gui

                lteInstances[plot.Name] = gui
            else
                local label = existing:FindFirstChildOfClass("TextLabel")
                if label then
                    label.Text = displayText
                    label.TextColor3 = color
                end
            end
        end
    end
end

-- GUI Principal
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "AzaroHubRGB"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 430, 0, 340)
frame.Position = UDim2.new(0, 50, 0, 80)
frame.BackgroundColor3 = Color3.fromRGB(35, 20, 60) -- início: roxo escuro
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true

local function createUICorner(parent, rad)
    local c = Instance.new("UICorner", parent)
    c.CornerRadius = UDim.new(0, rad or 10)
    return c
end

createUICorner(frame, 12)

-- RGB Effect com degrade mais bonito
local rgb = 0
RunService.RenderStepped:Connect(function()
    rgb = (rgb + 0.8) % 360
    local color = Color3.fromHSV(rgb/360, 0.8, 1)
    local baseColor = Color3.fromRGB(35, 20, 60)
    frame.BackgroundColor3 = color:lerp(baseColor, 0.5)
end)

-- TopBar com Título e Botão Fechar
local topBar = Instance.new("Frame", frame)
topBar.Size = UDim2.new(1, 0, 0, 50)
topBar.BackgroundTransparency = 1

local title = Instance.new("TextLabel", topBar)
title.Size = UDim2.new(1, -60, 1, 0)
title.Position = UDim2.new(0, 20, 0, 0)
title.BackgroundTransparency = 1
title.Text = "🧠 Azaro Hub"
title.Font = Enum.Font.FredokaOne
title.TextSize = 28
title.TextColor3 = Color3.new(1,1,1)

local closeBtn = Instance.new("TextButton", topBar)
closeBtn.Size = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -40, 0, 10)
closeBtn.Text = "_"
closeBtn.Font = Enum.Font.SourceSansBold
closeBtn.TextSize = 20
closeBtn.BackgroundTransparency = 1
closeBtn.TextColor3 = Color3.fromRGB(255, 100, 100)

local minimized = false
closeBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    for _, v in ipairs(frame:GetChildren()) do
        if v ~= topBar then
            v.Visible = not minimized
        end
    end
    frame.Size = minimized and UDim2.new(0,430,0,60) or UDim2.new(0,430,0,340)
end)

-- Tabs
local tabNames = { "Funções", "Visual", "Servidor" }
local tabs = {}
local selectedTab = 1

local tabBar = Instance.new("Frame", frame)
tabBar.Size = UDim2.new(1, -40, 0, 38)
tabBar.Position = UDim2.new(0, 20, 0, 50)
tabBar.BackgroundTransparency = 1

for i, name in ipairs(tabNames) do
    local tab = Instance.new("TextButton", tabBar)
    tab.Size = UDim2.new(0, 120, 0, 32)
    tab.Position = UDim2.new(0, (i-1)*130, 0, 0)
    tab.Text = name
    tab.Font = Enum.Font.FredokaOne
    tab.TextSize = 20
    tab.BackgroundColor3 = (i==1) and Color3.fromRGB(70,40,120) or Color3.fromRGB(40,30,60)
    tab.TextColor3 = Color3.new(1,1,1)
    tab.AutoButtonColor = false
    createUICorner(tab, 8)
    tabs[i] = tab
end

-- Container das Abas
local tabFrames = {}
for i=1,#tabNames do
    local f = Instance.new("Frame", frame)
    f.Size = UDim2.new(1, -40, 1, -108)
    f.Position = UDim2.new(0, 20, 0, 98)
    f.BackgroundTransparency = 1
    f.Visible = (i==1)
    tabFrames[i] = f
end

-- Função para trocar de aba
local function selectTab(idx)
    for i=1,#tabFrames do
        tabFrames[i].Visible = (i==idx)
        tabs[i].BackgroundColor3 = (i==idx) and Color3.fromRGB(70,40,120) or Color3.fromRGB(40,30,60)
    end
    selectedTab = idx
end

for i,tab in ipairs(tabs) do
    tab.MouseButton1Click:Connect(function() selectTab(i) end)
end

-- Aba 1: Funções
local scroll1 = Instance.new("ScrollingFrame", tabFrames[1])
scroll1.Size = UDim2.new(1, 0, 1, 0)
scroll1.CanvasSize = UDim2.new(0,0,0,500)
scroll1.ScrollBarThickness = 8
createUICorner(scroll1, 8)

local function addToggleLine(parent, name, order, callback)
    local container = Instance.new("Frame", parent)
    container.LayoutOrder = order
    container.Size = UDim2.new(1, -20, 0, 44)
    container.Position = UDim2.new(0, 10, 0, (order-1)*50)
    container.BackgroundColor3 = Color3.fromRGB(40, 30, 60)
    createUICorner(container, 6)

    local label = Instance.new("TextLabel", container)
    label.Size = UDim2.new(0.75, 0, 1, 0)
    label.Position = UDim2.new(0, 14, 0, 0)
    label.BackgroundTransparency = 1
    label.Font = Enum.Font.SourceSans
    label.TextSize = 20
    label.TextColor3 = Color3.new(1,1,1)
    label.Text = name

    local box = Instance.new("TextButton", container)
    box.Size = UDim2.new(0, 32, 0, 32)
    box.Position = UDim2.new(1, -48, 0.5, -16)
    box.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
    box.Text = ""
    box.AutoButtonColor = false
    createUICorner(box, 6)

    box.MouseButton1Click:Connect(function()
        toggles[name] = not toggles[name]
        box.BackgroundColor3 = toggles[name] and Color3.fromRGB(80, 255, 80) or Color3.fromRGB(100, 100, 100)
        callback(toggles[name])
    end)
    return container
end

-- Funções
addToggleLine(scroll1, "HighJump",  1, function(v) end)
addToggleLine(scroll1, "Steal",     2, function(v)
    if v then
        HRP.CFrame = CFrame.new(HRP.Position.X, SKY_Y, HRP.Position.Z)
        stealPlatform = Instance.new("Part", workspace)
        stealPlatform.Name = "SkyPlatform"
        stealPlatform.Size = Vector3.new(30,1,30)
        stealPlatform.Position = HRP.Position - Vector3.new(0,3,0)
        stealPlatform.Anchored = true
        stealPlatform.CanCollide = true
        stealPlatform.Transparency = 1
    else
        if stealPlatform then stealPlatform:Destroy() end
        HRP.Anchored = true
        task.spawn(function()
            for i = 1, 20 do
                HRP.CFrame = HRP.CFrame - Vector3.new(0, 5, 0)
                task.wait(0.05)
            end
            HRP.Anchored = false
        end)
    end
end)
addToggleLine(scroll1, "Speed", 3, function(v)
    toggles.Speed = v
    if v then
        boosted = true
        fakeWalk()
    else
        boosted = false
    end
end)

-- Aba 2: Visual
local scroll2 = Instance.new("ScrollingFrame", tabFrames[2])
scroll2.Size = UDim2.new(1, 0, 1, 0)
scroll2.CanvasSize = UDim2.new(0,0,0,600)
scroll2.ScrollBarThickness = 8
createUICorner(scroll2, 8)

addToggleLine(scroll2, "ESP", 1, function(v)
    for _, plr in pairs(Players:GetPlayers()) do
        local head = plr.Character and plr.Character:FindFirstChild("Head")
        if head then
            local existing = head:FindFirstChild("ESP")
            if v and not existing then
                local g = Instance.new("BillboardGui", head)
                g.Name = "ESP"
                g.Size = UDim2.new(0,100,0,30)
                g.AlwaysOnTop = true
                g.Adornee = head
                local t = Instance.new("TextLabel", g)
                t.Size = UDim2.new(1,0,1,0)
                t.BackgroundTransparency = 1
                t.TextColor3 = Color3.fromRGB(0, 0, 0)
                t.TextScaled = true
                t.Text = plr.Name
            elseif not v and existing then
                existing:Destroy()
            end
        end
    end
end)

addToggleLine(scroll2, "BaseESP", 2, function(v)
    if not v then
        for _, gui in pairs(workspace:GetDescendants()) do
            if gui:IsA("BillboardGui") and gui.Name == "BaseTimerESP" then
                gui:Destroy()
            end
        end
    else
        for _, part in pairs(workspace:GetDescendants()) do
            if part:IsA("BasePart") and part:FindFirstChild("UnlockTime") then
                local g = Instance.new("BillboardGui", part)
                g.Name = "BaseTimerESP"
                g.Size = UDim2.new(0,140,0,30)
                g.Adornee = part
                g.AlwaysOnTop = true
                local l = Instance.new("TextLabel", g)
                l.Size = UDim2.new(1,0,1,0)
                l.BackgroundTransparency = 1
                l.TextScaled = true
                task.spawn(function()
                    while part and g and toggles.BaseESP do
                        local diff = part.UnlockTime.Value - os.time()
                        l.Text = diff>0 and ("⏳ "..diff.."s") or "🔓 Liberada!"
                        l.TextColor3 = Color3.fromRGB(0, 0, 0)
                        task.wait(1)
                    end
                    if g then g:Destroy() end
                end)
            end
        end
    end
end)

-- NOVA FUNÇÃO: ESP Brain do Watch
addToggleLine(scroll2, "ESP Brain do Watch", 3, function(v)
    activeLockTimeEsp = v
    if v then
        -- Atualiza sempre que ativo
        task.spawn(function()
            while activeLockTimeEsp do
                updatelock()
                task.wait(1)
            end
        end)
    else
        updatelock()
    end
end)

-- Aba 3: Servidor
local scroll3 = Instance.new("Frame", tabFrames[3])
scroll3.Size = UDim2.new(1, 0, 1, 0)
scroll3.BackgroundTransparency = 1

local hopBtn = Instance.new("TextButton", scroll3)
hopBtn.Size = UDim2.new(0, 260, 0, 44)
hopBtn.Position = UDim2.new(0.5, -130, 0.2, 0)
hopBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 100)
hopBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
hopBtn.TextSize = 22
hopBtn.Font = Enum.Font.FredokaOne
hopBtn.Text = "Server Hop"
createUICorner(hopBtn, 8)

local function serverHop()
    local req = syn and syn.request or http_request or request
    if not req then
        print("Executor não suporta HTTP request")
        return
    end
    local ok, response = pcall(function()
        return req({
            Url = "https://games.roblox.com/v1/games/" .. PlaceId .. "/servers/Public?sortOrder=Asc&limit=100"
        })
    end)
    if not ok or not response then
        print("Erro ao buscar servidores.")
        return
    end
    local success, body = pcall(HttpService.JSONDecode, HttpService, response.Body)
    if not success or not body or not body.data then
        print("Falha ao decodificar resposta.")
        return
    end
    local servers = {}
    for _, v in ipairs(body.data) do
        if v.playing < v.maxPlayers and v.id ~= game.JobId then
            table.insert(servers, v.id)
        end
    end
    if #servers > 0 then
        local serverId = servers[math.random(#servers)]
        print("Server Hop para:", serverId)
        TeleportService:TeleportToPlaceInstance(PlaceId, serverId, LocalPlayer)
    else
        print("Nenhum servidor disponível.")
    end
end
hopBtn.MouseButton1Click:Connect(serverHop)

-- Função Speed/FakeWalk
function fakeWalk()
    local char = LocalPlayer.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    local conn
    conn = RunService.Heartbeat:Connect(function()
        if not boosted then conn:Disconnect() return end
        local moveDir = humanoid.MoveDirection
        if moveDir.Magnitude > 0 then
            hrp.CFrame = hrp.CFrame + moveDir.Unit * speedMultiplier * 0.2
        end
    end)
end

-- HighJump
UIS.InputBegan:Connect(function(i)
    if i.KeyCode == Enum.KeyCode.Space and toggles.HighJump then
        HRP.Velocity = Vector3.new(0, HIGH_JUMP, 0)
    end
end)

-- Anti-fall
Hum:GetPropertyChangedSignal("FloorMaterial"):Connect(function()
    if Hum.FloorMaterial == Enum.Material.Air and HRP.Position.Y < -20 then
        HRP.Velocity = Vector3.new(0,100,0)
    end
end)

-- Speed reset ao respawn
LocalPlayer.CharacterAdded:Connect(function()
    boosted = false
end)

-- Layouts
for _, scroll in ipairs({scroll1, scroll2}) do
    local layout = Instance.new("UIListLayout", scroll)
    layout.Padding = UDim.new(0, 10)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
end

topBar.Parent = frame
tabBar.Parent = frame
for _, f in ipairs(tabFrames) do f.Parent = frame end

-- Fim do script
