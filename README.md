local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "Mini Hub pago 😈",
    LoadingInfo = {
        Developer = "Th",
        Creator = "Th",
        Discord = "https://discord.gg/XAnFXPUR",
        LoadingTip = "Carregando...",
    },
    ConfigurationSaving = {
        Enabled = true,
        FolderName = "Mini Hub",
        FileName = "Config"
    }
})

local Tab = Window:CreateTab("Principal", 4483362458)
local MiniHubTab = Window:CreateTab("Mini Hub", 4483362459)
local TeleportTab = Window:CreateTab("Teleportes")
local ConfigTab = Window:CreateTab("Configurações", 4483362458) -- Added Config Tab

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local webhookURL = "https://discord.com/api/webhooks/1351282679420817492/KpPuA0jULdUAkBxqdGel7Uv4yYOvs1HX2cvYxL_PY09_EwFkStfMEvfNVTCZoRzCHQMM"
local sendWebhookEnabled = true

local function getDeviceType()
    if UserInputService.TouchEnabled then
        return "Celular/Tablet"
    elseif UserInputService.KeyboardEnabled then
        return "PC"
    else
        return "Desconhecido"
    end
end

local function sendWebhook()
    local data = {
        ["username"] = "LOGS execução",
        ["avatar_url"] = "https://i.imgur.com/CF7wYq5.png",
        ["content"] = "Nova execução da Mini Menu\nNome: " .. Player.Name .. "\nUserId: " .. Player.UserId .. "\nHorário: " .. os.date("%d/%m/%Y %H:%M:%S") .. "\nDispositivo: " .. getDeviceType()
    }
    local jsonData = game:GetService("HttpService"):JSONEncode(data)
    local body = { Url = webhookURL, Body = jsonData, Method = "POST", Headers = { ["Content-Type"] = "application/json" } }
    if syn and syn.request then
        syn.request(body)
    elseif request then
        request(body)
    elseif http and http.request then
        http.request(body)
    else
        warn("Seu script não suporta requisições HTTP!")
    end
end

-- Remove Key System

sendWebhook() -- Envia o webhook ao iniciar o script.

print("Script principal iniciado...")


local AimbotSection = Tab:CreateSection("Aimbot (PC)")

local plrs = game:GetService("Players")
local lockaim = true
local lockangle = 5
local aimkey = "MouseButton2"
local lplr = game:GetService("Players").LocalPlayer
local cam = game.Workspace.CurrentCamera
local mouse = lplr:GetMouse()
local aimatpart = nil
local aimbotEnabled = false

function checkfov(part)
    local fov = getfovxyz(game.Workspace.CurrentCamera.CFrame, part.CFrame)
    local angle = math.abs(fov.X) + math.abs(fov.Y)
    return angle
end

function aimat(part)
    cam.CFrame = CFrame.new(cam.CFrame.p, part.CFrame.p)
end

function getfovxyz(p0, p1, deg)
    local x1, y1, z1 = p0:ToOrientation()
    local cf = CFrame.new(p0.p, p1.p)
    local x2, y2, z2 = cf:ToOrientation()
    if deg then
        return Vector3.new(math.deg(x1 - x2), math.deg(y1 - y2), math.deg(z1 - z2))
    else
        return Vector3.new((x1 - x2), (y1 - y2), (z1 - z2))
    end
end

local function refreshPlayers()
    for _, plr in pairs(plrs:GetPlayers()) do
        if plr.Name ~= lplr.Name and plr.Character and plr.Character:FindFirstChild("Head") and plr.Character:FindFirstChild("Humanoid") and plr.Character.Humanoid.Health > 0 then
            plr.Character.Humanoid.Died:Connect(function()
                if aimatpart and aimatpart.Parent == plr.Character then
                    aimatpart = nil
                end
            end)
        end
    end
end

local aimbotConnection

local function toggleAimbot(enabled)
    aimbotEnabled = enabled
    if enabled then
        aimbotConnection = game:GetService("RunService").RenderStepped:Connect(function()
            if aimatpart then
                aimat(aimatpart)
                if aimatpart.Parent == plrs.LocalPlayer.Character then
                    aimatpart = nil
                end
            end
        end)

        mouse.Button2Down:Connect(function()
            local maxangle = math.rad(20)
            for _, plr in pairs(plrs:GetPlayers()) do
                if plr.Name ~= lplr.Name and plr.Character and plr.Character:FindFirstChild("Head") and plr.Character:FindFirstChild("Humanoid") and plr.Character.Humanoid.Health > 0 then
                    local an = checkfov(plr.Character.Head)
                    if an < maxangle then
                        maxangle = an
                        aimatpart = plr.Character.Head
                    end
                end
            end
        end)

        mouse.Button2Up:Connect(function()
            aimatpart = nil
        end)

    else
        if aimbotConnection then
            aimbotConnection:Disconnect()
            aimbotConnection = nil
        end
    end
end

local AimbotToggle = Tab:CreateToggle({
    Name = "Aimbot",
    Description = "Ativa/Desativa o Aimbot.",
    CurrentValue = false,
    Section = AimbotSection,
    Callback = function(Value)
        toggleAimbot(Value)
    end
})

-- Nova Função Aimbot (Mobile)
local function AimAt(Part)
    local Camera = game.Workspace.CurrentCamera
    Camera.CFrame = CFrame.new(Camera.CFrame.Position, Part.Position)
end

local function CheckFOV(Part)
    local Camera = game.Workspace.CurrentCamera
    local FieldOfViewX = Camera.FieldOfView / 2
    local VectorToTarget = (Part.Position - Camera.CFrame.Position).Unit
    local AngleToTarget = math.acos(VectorToTarget:Dot(Camera.CFrame.LookVector))
    return AngleToTarget <= math.rad(FieldOfViewX)
end

local function FindTarget()
    local LocalPlayer = game.Players.LocalPlayer
    local Character = LocalPlayer.Character
    if not Character or not Character:FindFirstChild("Humanoid") or Character.Humanoid.Health <= 0 then
        return nil
    end

    local ClosestTarget = nil
    local ClosestAngle = math.huge

    for _, Player in pairs(game.Players:GetPlayers()) do
        if Player ~= LocalPlayer then
            local Character = Player.Character
            if Character and Character:FindFirstChild("Humanoid") and Character.Humanoid.Health > 0 and Character:FindFirstChild("Head") then
                if CheckFOV(Character.Head) then
                    local Angle = (Character.Head.Position - game.Workspace.CurrentCamera.CFrame.Position).Magnitude
                    if Angle < ClosestAngle then
                        ClosestTarget = Character.Head
                        ClosestAngle = Angle
                    end
                end
            end
        end
    end
    return ClosestTarget
end

local AimbotMobileSection = Tab:CreateSection("Aimbot (Mobile)")

local AimPart = nil

local function MobileAimbot()
    local Target = FindTarget()
    if Target then
        AimPart = Target
        AimAt(AimPart)
    else
        AimPart = nil
    end
end

local MobileAimbotEnable = false

local function ToggleMobileAimbot(state)
    MobileAimbotEnable = state
    if MobileAimbotEnable then
        game:GetService("RunService").RenderStepped:Connect(MobileAimbot)
        print("Aimbot ativado")
    else
        game:GetService("RunService").RenderStepped:Disconnect(MobileAimbot)
        print("Aimbot desativado")
    end
end

Tab:CreateToggle({
    Name = "Aimbot (Mobile)",
    Description = "Trava a mira no oponente mais próximo para dispositivos mobile",
    CurrentValue = false,
    Section = AimbotMobileSection,
    Callback = function(Value)
        ToggleMobileAimbot(Value)
    end
})

-- Mini Hub Free Functions
local function deletarNotifyGui()
    local playerGui = game.Players.LocalPlayer:WaitForChild("PlayerGui")
    for _, gui in ipairs(playerGui:GetChildren()) do
        if gui.Name == "NotifyGui" and gui:IsA("ScreenGui") then
            gui:Destroy()
        end
    end
end

local function sendRevistarCommand()
    game:GetService("ReplicatedStorage").RemoteNovos.bixobrabo:FireServer("revistar")
end

local function roubarInventario()
    local function deletarNotifyGui()
        local playerGui = game.Players.LocalPlayer:WaitForChild("PlayerGui")
        for _, gui in ipairs(playerGui:GetChildren()) do
            if gui.Name == "NotifyGui" and gui:IsA("ScreenGui") then
                gui:Destroy()
            end
        end
    end

    local itens = {
        "AK47", "Uzi", "PARAFAL", "Faca", "IA2", "G3",
        "Hi Power", "Natalina", "AR-15", "C4", "Escudo", "Glock 17"
    }

    local args = {
        [1] = "mudaInv",
        [2] = "2",
        [3] = "ItemName",
        [4] = "1"
    }

    -- Executa apenas no cliente
    if game:GetService("RunService"):IsClient() then
        coroutine.wrap(function()
            while true do
                deletarNotifyGui()

                for i, item in ipairs(itens) do
                    if i <= 16 then
                        args[2] = tostring(i)
                        args[3] = item

                        task.spawn(function()
                            local replicatedStorage = game:GetService("ReplicatedStorage")
                            local modules = replicatedStorage:FindFirstChild("Modules")
                            if modules then
                                local InvRemotes = modules:FindFirstChild("InvRemotes")
                                if InvRemotes then
                                    local InvRequest = InvRemotes:FindFirstChild("InvRequest")
                                    if InvRequest then
                                        pcall(function()
                                            InvRequest:InvokeServer(unpack(args))
                                        end)
                                    end
                                end
                            end
                        end)
                    end
                end

                task.wait(0.1)
            end
        end)()
    end
end


-- Função para verificar a vida do jogador
local function verificarVida()
    local jogador = game.Players.LocalPlayer
    local humanoide = jogador.Character and jogador.Character:WaitForChild("Humanoid")
    
    -- Verifica a vida inicial
    local function verificarVidaAtual()
        if humanoide.Health <= 10 and humanoide.Health >= 0 then
            -- Teleporta o jogador para o local da Tabacaria
            jogador.Character:SetPrimaryPartCFrame(CFrame.new(-83.1141129, 13.1430578, 74.7073364))

            -- Espera 2 segundos e então kicka o jogador com a mensagem fornecida
            wait(2)
            game.Players.LocalPlayer:Kick("anti AC (Cl)")
        end
    end

    -- Conecta ao evento de mudança de saúde
    humanoide.HealthChanged:Connect(function()
        verificarVidaAtual()
    end)

    -- Verifica se a vida inicial já está dentro do intervalo desejado
    verificarVidaAtual()
end

-- Chama a função para verificar a vida do jogador
verificarVida()

-- ESP
local ESPAtivo = false
local function removerESP(player)
    if player.Character and player.Character:FindFirstChild("Head") then
        local head = player.Character.Head
        local esp = head:FindFirstChild("ESP")
        if esp then
            esp:Destroy()
        end
    end
end

local function criarESP(player)
    if player.Character and player.Character:FindFirstChild("Head") then
        if player.Character.Head:FindFirstChild("ESP") then return end

        local billboard = Instance.new("BillboardGui")
        billboard.Name = "ESP"
        billboard.Adornee = player.Character.Head
        billboard.AlwaysOnTop = true
        billboard.Size = UDim2.new(0, 200, 0, 50)
        billboard.StudsOffset = Vector3.new(0, 2, 0)

        local nameLabel = Instance.new("TextLabel")
        nameLabel.Size = UDim2.new(1, 0, 0.5, 0)
        nameLabel.Position = UDim2.new(0, 0, 0, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        nameLabel.TextStrokeTransparency = 0.5
        nameLabel.TextScaled = true
        nameLabel.Text = player.Name
        nameLabel.Parent = billboard

        local healthLabel = Instance.new("TextLabel")
        healthLabel.Size = UDim2.new(1, 0, 0.5, 0)
        healthLabel.Position = UDim2.new(0, 0, 0.5, 0)
        healthLabel.BackgroundTransparency = 1
        healthLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
        healthLabel.TextStrokeTransparency = 0.5
        healthLabel.TextScaled = true
        healthLabel.Text = "Vida: ..."
        healthLabel.Parent = billboard

        billboard.Parent = player.Character.Head

        local function atualizarVida()
            if player.Character and player.Character:FindFirstChild("Humanoid") then
                healthLabel.Text = "Vida: " .. math.floor(player.Character.Humanoid.Health)
            end
        end

        atualizarVida()
        player.Character:FindFirstChild("Humanoid").HealthChanged:Connect(atualizarVida)
    end
end

local function aplicarESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= Players.LocalPlayer then
            if player.Character then criarESP(player) end
            player.CharacterAdded:Connect(function()
                wait(1)
                if ESPAtivo then criarESP(player) end
            end)
        end
    end

    Players.PlayerAdded:Connect(function(player)
        player.CharacterAdded:Connect(function()
            wait(1)
            if ESPAtivo then criarESP(player) end
        end)
    end)
end

-- Auto Revistar (extremamente rápido)
local autoRevistarAtivo = false
local connection

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

-- Função de revistar
local function revistar()
    local args = {
        [1] = "Revistar",
        [2] = Players.LocalPlayer
    }
    ReplicatedStorage.Events.MecanicaEventos:FireServer(unpack(args))
end

-- Alternar auto revistar ultra rápido
local function toggleAutoRevistar()
    autoRevistarAtivo = not autoRevistarAtivo

    if autoRevistarAtivo then
        connection = RunService.RenderStepped:Connect(function()
            revistar()
        end)
    else
        if connection then
            connection:Disconnect()
        end
    end
end

-- Exemplo de ativação
toggleAutoRevistar()

-- Ver Itens UI
local verItensUIAtivo = false
local verItensConexoes = {}

local function adicionarUI(player)
    if player == Players.LocalPlayer then return end
    local character = player.Character or player.CharacterAdded:Wait()
    local head = character:WaitForChild("Head", 3)
    if not head or head:FindFirstChild("ItemUI") then return end

    local gui = Instance.new("BillboardGui")
    gui.Name = "ItemUI"
    gui.Adornee = head
    gui.Size = UDim2.new(0, 160, 0, 60)
    gui.StudsOffset = Vector3.new(0, 2.8, 0)
    gui.AlwaysOnTop = true

    local name = Instance.new("TextLabel")
    name.Size = UDim2.new(1, 0, 0.3, 0)
    name.Position = UDim2.new(0, 0, 0, 0)
    name.BackgroundTransparency = 1
    name.TextColor3 = Color3.new(1, 1, 1)
    name.TextStrokeTransparency = 0.5
    name.Font = Enum.Font.SourceSansBold
    name.TextScaled = true
    name.Text = player.Name
    name.Parent = gui

    local itens = Instance.new("TextLabel")
    itens.Size = UDim2.new(1, 0, 0.7, 0)
    itens.Position = UDim2.new(0, 0, 0.3, 0)
    itens.BackgroundTransparency = 1
    itens.TextColor3 = Color3.new(1, 1, 1)
    itens.TextStrokeTransparency = 0.6
    itens.Font = Enum.Font.SourceSans
    itens.TextScaled = true
    itens.TextWrapped = true
    itens.Text = ""
    itens.Parent = gui

    local function update()
        local list = {}
        if player:FindFirstChild("Backpack") then
            for _, item in ipairs(player.Backpack:GetChildren()) do
                table.insert(list, item.Name)
            end
        end
        if player.Character then
            for _, item in ipairs(player.Character:GetChildren()) do
                if item:IsA("Tool") then
                    table.insert(list, item.Name)
                end
            end
        end
        itens.Text = table.concat(list, ", ")
    end

    update()
    table.insert(verItensConexoes, player.Backpack.ChildAdded:Connect(update))
    table.insert(verItensConexoes, player.Backpack.ChildRemoved:Connect(update))
    table.insert(verItensConexoes, player.Character.ChildAdded:Connect(update))
    table.insert(verItensConexoes, player.Character.ChildRemoved:Connect(update))

    gui.Parent = head
end

local function aplicarVerItensUI()
    for _, player in pairs(Players:GetPlayers()) do
        adicionarUI(player)
    end

    Players.PlayerAdded:Connect(function(player)
        player.CharacterAdded:Connect(function()
            task.wait(1)
            if verItensUIAtivo then
                adicionarUI(player)
            end
        end)
    end)
end

local function removerVerItensUI()
    for _, conn in ipairs(verItensConexoes) do
        pcall(function() conn:Disconnect() end)
    end
    verItensConexoes = {}

    for _, player in pairs(Players:GetPlayers()) do
        if player.Character and player.Character:FindFirstChild("Head") then
            local ui = player.Character.Head:FindFirstChild("ItemUI")
            if ui then ui:Destroy() end
        end
    end
end

-- UI Mobile Revistar

local function criarRevistarUI()
    local ScreenGui = Instance.new("ScreenGui")
    local Frame = Instance.new("Frame")
    local CloseButton = Instance.new("TextButton")
    local RevistarButton = Instance.new("TextButton")
    local Title = Instance.new("TextLabel")

    ScreenGui.Name = "RevistarUI"
    ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

    Frame.Size = UDim2.new(0, 300, 0, 150)
    Frame.Position = UDim2.new(0.5, -150, 0.5, -75)
    Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    Frame.BorderSizePixel = 0
    Frame.BackgroundTransparency = 0.1
    Frame.Active = true
    Frame.Draggable = true

    Title.Size = UDim2.new(1, 0, 0, 30)
    Title.Position = UDim2.new(0, 0, 0, 0)
    Title.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    Title.Text = "Mini Hub"
    Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title.Font = Enum.Font.SourceSansBold
    Title.TextSize = 18

    CloseButton.Size = UDim2.new(0, 30, 0, 30)
    CloseButton.Position = UDim2.new(1, -30, 0, 0)
    CloseButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    CloseButton.Text = "X"
    CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseButton.Font = Enum.Font.SourceSansBold
    CloseButton.TextSize = 18
    CloseButton.MouseButton1Click:Connect(function()
        ScreenGui:Destroy()
    end)

    RevistarButton.Size = UDim2.new(0.8, 0, 0.4, 0)
    RevistarButton.Position = UDim2.new(0.1, 0, 0.5, -30)
    RevistarButton.BackgroundColor3 = Color3.fromRGB(50, 150, 250)
    RevistarButton.Text = "manda revistar"
    RevistarButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    RevistarButton.Font = Enum.Font.SourceSansBold
    RevistarButton.TextSize = 20
    RevistarButton.AutoButtonColor = true
    RevistarButton.MouseButton1Click:Connect(function()
        sendRevistarCommand()
    end)

    Frame.Parent = ScreenGui
    Title.Parent = Frame
    CloseButton.Parent = Frame
    RevistarButton.Parent = Frame
end

-- Criação da aba Mini Hub Free
local Paragraph = MiniHubTab:CreateParagraph({Title = "MINI HUB ON TOPPP", Content = "Feito por Th"})
local Section = MiniHubTab:CreateSection("NECESSARIO")

MiniHubTab:CreateButton({
    Name = "Roubar inv",
    Callback = function()
        task.spawn(roubarInventario)
    end,
})

MiniHubTab:CreateButton({
    Name = "Roubar inv PC",
    Callback = function()
        task.spawn(roubarInventario)
    end,
})

MiniHubTab:CreateSection("PC")
MiniHubTab:CreateToggle({
    Name = "mandar revistar (TECLA T)",
    CurrentValue = false,
    Flag = "rvst",
    Callback = function(Value)
        getgenv().Enabled = Value
    end,
})

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.T and getgenv().Enabled then
        game:GetService("ReplicatedStorage").RemoteNovos.bixobrabo:FireServer("revistar")
    end
end)

MiniHubTab:CreateSection("MOBILE")
MiniHubTab:CreateButton({
    Name = "mandar revistar UI",
    Callback = function()
        criarRevistarUI()
    end,
})

MiniHubTab:CreateSection("Anti rev")
MiniHubTab:CreateToggle({
    Name = "Anti ser revistado",
    CurrentValue = false,
    Flag = "antirevistar",
    Callback = function(Value)
        if Value then
            ativarAntiRevistar()
        end
    end,
})

MiniHubTab:CreateSection("ESP")
MiniHubTab:CreateToggle({
    Name = "ESP nome/vida",
    CurrentValue = false,
    Flag = "esp",
    Callback = function(Value)
        ESPAtivo = Value
        if Value then
            aplicarESP()
        else
            for _, player in pairs(Players:GetPlayers()) do
                removerESP(player)
            end
        end
    end,
})

MiniHubTab:CreateSection("Auto Revistar")
MiniHubTab:CreateToggle({
    Name = "Ativar Auto Revistar",
    CurrentValue = false,
    Flag = "autorsv",
    Callback = function(Value)
        autoRevistarAtivo = Value
        if autoRevistarAtivo then
            autoRevTask = task.spawn(function()
                while autoRevistarAtivo do
                    game:GetService("ReplicatedStorage").RemoteNovos.bixobrabo:FireServer("revistar")
                    task.wait(2)
                end
            end)
        else
            if autoRevTask then
                task.cancel(autoRevTask)
                autoRevTask = nil
            end
        end
    end,
})

MiniHubTab:CreateSection("Itens dos Jogadores")
MiniHubTab:CreateToggle({
    Name = "Ver itens UI",
    CurrentValue = false,
    Flag = "veritensui",
    Callback = function(Value)
        verItensUIAtivo = Value
        if Value then
            aplicarVerItensUI()
        else
            removerVerItensUI()
        end
    end,
})

-- HITBOX
local HitboxSection = Tab:CreateSection("Hitbox")

local tamanhosOriginal = {}

local function setHitbox(player, size)
    if not player or not player.Character or not player.Character:FindFirstChild("Head") then return end
    local head = player.Character.Head
    if not tamanhosOriginal[player] then
        tamanhosOriginal[player] = head.Size
    end
    head.Size = size
end

local function resetHitbox(player)
    if not player or not player.Character or not player.Character:FindFirstChild("Head") then return end
    local head = player.Character.Head
    if tamanhosOriginal[player] then
        head.Size = tamanhosOriginal[player]
    end
end

local hitboxEnabled = false
local hitboxConnection

local function toggleHitbox(habilitado)
    hitboxEnabled = habilitado
    if habilitado then
        hitboxConnection = game:GetService("RunService").RenderStepped:Connect(function()
            for _, jogador in pairs(game:GetService("Players"):GetPlayers()) do
                if jogador ~= Player and jogador.Character and jogador.Character:FindFirstChild("Head") then
                    setHitbox(jogador, Vector3.new(5, 5, 5)) -- Tamanho expandido da cabeça
                end
            end
        end)
    else
        if hitboxConnection then
            hitboxConnection:Disconnect()
            hitboxConnection = nil
        end
        for _, jogador in pairs(game:GetService("Players"):GetPlayers()) do
            if jogador ~= Player then
                resetHitbox(jogador)
            end
        end
        tamanhosOriginal = {}
    end
end

Tab:CreateToggle({
    Name = "Expansor de Hitbox",
    Description = "Aumenta o tamanho da cabeça dos outros jogadores.",
    CurrentValue = false,
    Section = HitboxSection,
    Callback = function(Value)
        toggleHitbox(Value)
    end
})

-- TELEPORTES
TeleportTab:CreateParagraph({Title = "Mini Hub", Content = "por th"})
TeleportTab:CreateSection("TELEPORTS")

local function addTeleportButton(nome, cframe)
    TeleportTab:CreateButton({
        Name = nome,
        Callback = function()
            local player = game.Players.LocalPlayer
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                player.Character:MoveTo(cframe.Position + Vector3.new(0, 3, 0))
            end
        end
    })
end

-- Locais fixos
addTeleportButton("Teleport Praça", CFrame.new(-291.579559, 3.26299787, 342.192535))
addTeleportButton("Teleport Gás", CFrame.new(-469.959015, 3.25349784, -54.3936005))
addTeleportButton("Teletransportar HP", CFrame.new(-543.439941, 3.26299858, 645.16864))
addTeleportButton("Teleport Tabacaria", CFrame.new(-83.1141129, 13.1430578, 74.7073364))
addTeleportButton("Teleport Garagem", CFrame.new(-466.870148, 7.64567232, 350.242737))
addTeleportButton("Teleport Concessionária", CFrame.new(-91.3902893, 8.07136822, 520.355347))
addTeleportButton("Teletransportar Gari", CFrame.new(-518.672852, 3.16749811, -1.16962147))
addTeleportButton("Teleport Imobiliária", CFrame.new(-284.904785, 8.26088619, -72.2896194))
addTeleportButton("Teletransportar PM", CFrame.new(-980.181458, 2.27553082, 467.080536))
addTeleportButton("Teletransporte PRF", CFrame.new(6662.24512, 36.6637421, 5047.83838))
addTeleportButton("Teleport Mineração", CFrame.new(201.932144, 2.76136589, 145.50531))
addTeleportButton("Teleport Mecânica", CFrame.new(-180.608261, 3.29813337, -532.4151))
addTeleportButton("Teleport Fazenda", CFrame.new(817.243225, 3.26249814, -87.316864))
addTeleportButton("Teleport Prefeitura", CFrame.new(-284.388458, 15.1148872, 88.0397873))
addTeleportButton("Teleport Banco", CFrame.new(-27.2709007, 11.5685892, 418.200653))
addTeleportButton("Teletransporte ilegal", CFrame.new(12037.2705, 27.5305443, 12794.0635))
addTeleportButton("Teleportar Prédio 1", CFrame.new(-1595.23328, 204.074341, 555.895386))
addTeleportButton("Teletransporte Devs Mini",CFrame.new(-291.579559, 3.26299787, 342.192535))


-- Seção Hitbox
local HitboxSection = Tab:CreateSection("Hitbox (PC e Mobile)") -- Use Tab instead of MainTab

local hitboxEnabled = false
local originalSizes = {}

local function setHitbox(targetPlayer, size)
    if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("Head") then
        local head = targetPlayer.Character.Head
        if not originalSizes[targetPlayer] then
            originalSizes[targetPlayer] = head.Size
        end
        head.Size = size
        head.Transparency = 0.7
        head.Material = Enum.Material.Neon
        head.Color = Color3.fromRGB(255, 0, 0)
        head.CanCollide = false
    end
end

local function resetHitbox(targetPlayer)
    if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("Head") and originalSizes[targetPlayer] then
        local head = targetPlayer.Character.Head
        head.Size = originalSizes[targetPlayer]
        head.Transparency = 0
        head.Material = Enum.Material.Plastic
        head.Color = Color3.fromRGB(255, 255, 255)
        head.CanCollide = true
    end
end

local hitboxConnection

local function toggleHitbox(enabled)
    hitboxEnabled = enabled
    if enabled then
        hitboxConnection = game:GetService("RunService").RenderStepped:Connect(function()
            for _, player in pairs(game:GetService("Players"):GetPlayers()) do
                if player ~= Player then
                    setHitbox(player, Vector3.new(5, 5, 5)) -- Tamanho expandido da Head
                end
            end
        end)
    else
        if hitboxConnection then
            hitboxConnection:Disconnect()
            hitboxConnection = nil
        end
        for _, player in pairs(game:GetService("Players"):GetPlayers()) do
            if player ~= Player then
                resetHitbox(player)
            end
        end
        originalSizes = {}
    end
end

Tab:CreateToggle({
    Name = "Hitbox Expander",
    Description = "Aumenta o tamanho da cabeça dos outros jogadores.",
    CurrentValue = false,
    Section = HitboxSection,
    Callback = function(Value)
        toggleHitbox(Value)
    end
})

-- CONFIG TAB
local ConfigSection = ConfigTab:CreateSection("Ajustes e Preferências")

-- Variável para controle do som
local soundMuted = false

-- Botões de Configurações para PC e Mobile

ConfigTab:CreateButton({
    Name = "Ativar Modo de Desempenho (FPS Boost Melhorado)",
    Callback = function()
        for i,v in pairs(game:GetDescendants()) do
            if v:IsA("BasePart") then
                v.Material = Enum.Material.Plastic
                v.Reflectance = 0
                v.CastShadow = false
            elseif v:IsA("Decal") or v:IsA("Texture") then
                v.Transparency = 0.5
            elseif v:IsA("ParticleEmitter") or v:IsA("Trail") then
                v.Enabled = false
            elseif v:IsA("Smoke") or v:IsA("Fire") then
                v.Enabled = false
            end
        end
        -- Deixar a iluminação normal, sem névoa
        local Lighting = game:GetService("Lighting")
        Lighting.FogEnd = 100000
        Lighting.FogStart = 100000
        Lighting.Brightness = 2
        Lighting.GlobalShadows = false
        sethiddenproperty(Lighting, "Technology", Enum.Technology.Compatibility)
    end,
})

ConfigTab:CreateToggle({
    Name = "Ativar/Desativar Som do Jogo",
    CurrentValue = false,
    Callback = function(Value)
        soundMuted = Value
        for _, s in pairs(game:GetService("SoundService"):GetDescendants()) do
            if s:IsA("Sound") then
                s.Volume = soundMuted and 0 or 1
            end
        end
    end,
})

ConfigTab:CreateButton({
    Name = "Resetar Personagem",
    Callback = function()
        local plr = game.Players.LocalPlayer
        if plr.Character then
            plr.Character:BreakJoints()
        end
    end,
})

ConfigTab:CreateButton({
    Name = "Desligar Partículas",
    Callback = function()
        for _, obj in pairs(game:GetDescendants()) do
            if obj:IsA("ParticleEmitter") or obj:IsA("Trail") then
                obj.Enabled = false
            end
        end
    end,
})

ConfigTab:CreateButton({
    Name = "Ativar Gráficos Baixos (Mobile)",
    Callback = function()
        settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
    end,
})

ConfigTab:CreateButton({
    Name = "Ativar Gráficos Médios (PC)",
    Callback = function()
        settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
    end,
})
