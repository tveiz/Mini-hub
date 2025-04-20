-- Configurações da Key e HWID
local KEY = "teste"  -- Chave padrão
local KEY_EXPIRATION_DAYS = 7 -- Duração da chave em dias

-- Funções de armazenamento e leitura de dados (usando DataStore)
local DataStoreService = game:GetService("DataStoreService")
local keyDataStore = DataStoreService:GetDataStore("MiniCityKeyData")

local function saveData(key, data)
    xpcall(function()
        keyDataStore:SetAsync(key, data)
    end, function(err)
        warn("Erro ao salvar dados:", err)
    end)
end

local function loadData(key)
    local data = nil
    xpcall(function()
        keyDataStore:GetAsync(key)
    end, function(err)
        warn("Erro ao carregar dados:", err)
    end)
    return data
end

-- Função para obter o HWID (pode precisar de ajustes para melhor segurança)
local function getHWID()
    local hwid = nil
    if game:GetService("RunService"):IsStudio() then
        hwid = "STUDIO_MODE" -- Para testes no Roblox Studio
    else
       hwid = game:GetService("RbxAnalyticsService"):GetClientId()
	end
    return hwid
end

local hwid = getHWID()
print("HWID:", hwid)

local function verificarKey(keyDigitada)
    if keyDigitada == KEY then
       -- Verificar se a key já foi usada antes
        local keyData = loadData(KEY)
		print("keyData = loadData(KEY) ",keyData)
        if keyData then
            -- Key já foi usada
            if keyData.HWID == hwid then
                -- Mesmo dispositivo, verificar expiração
                local expirationDate = keyData.ExpirationDate
                if expirationDate and expirationDate > os.time() then
                    -- Key válida e não expirada
                    return true, "Key válida e não expirada."
                else
                    -- Key expirada
                    return false, "Key expirada."
                end
            else
                -- Key usada em outro dispositivo
                return false, "Key já utilizada em outro dispositivo."
            end
        else
            -- Key não foi usada ainda, registrar uso
            local expirationDate = os.time() + (KEY_EXPIRATION_DAYS * 24 * 60 * 60)
            local newData = {
                HWID = hwid,
                ExpirationDate = expirationDate
            }
            saveData(KEY, newData)
            return true, "Key ativada com sucesso."
        end
    else
        return false, "Key inválida."
    end
end

-- Adicione esse sistema de Key nesse scritp e faça que só irá abrir o menu se a Key estiver certa e quando colocar a Key certa e o menu abrir a UI de Key irá sumir

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local StarterGui = game:GetService("StarterGui") -- Get StarterGui service
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Create Key Input UI
local KeyGui = Instance.new("ScreenGui")
KeyGui.Name = "KeyGui"
KeyGui.Parent = StarterGui
KeyGui.ResetOnSpawn = false

local KeyFrame = Instance.new("Frame")
KeyFrame.Size = UDim2.new(0.3, 0, 0.2, 0)
KeyFrame.Position = UDim2.new(0.5, -0.15, 0.5, -0.1)
KeyFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
KeyFrame.BackgroundTransparency = 0.2
KeyFrame.AnchorPoint = Vector2.new(0.5, 0.5)
KeyFrame.Parent = KeyGui
Instance.new("UICorner", KeyFrame).CornerRadius = UDim.new(0, 12)

local KeyImage = Instance.new("ImageLabel")
KeyImage.Size = UDim2.new(0, 80, 0, 80)
KeyImage.Position = UDim2.new(0.5, -40, 0, 10)
KeyImage.BackgroundTransparency = 1
KeyImage.Image = "rbxassetid://79308101207172"
KeyImage.Parent = KeyFrame

local KeyBox = Instance.new("TextBox")
KeyBox.PlaceholderText = "Digite a KEY..."
KeyBox.Size = UDim2.new(0.8, 0, 0, 40)
KeyBox.Position = UDim2.new(0.1, 0, 0.65, 0)
KeyBox.Text = ""
KeyBox.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
KeyBox.TextColor3 = Color3.fromRGB(255, 255, 255)
KeyBox.Font = Enum.Font.Gotham
KeyBox.TextSize = 16
KeyBox.Parent = KeyFrame
Instance.new("UICorner", KeyBox).CornerRadius = UDim.new(0, 8)

-- Function to create and open the Rayfield menu
local function openRayfieldMenu()
    -- Your Rayfield menu code starts here
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
    local MiniHubTab = Window:CreateTab("Mini Hub Free", 4483362459)

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

    -- Hitbox Section (INSERIDO AQUI)
    local HitboxSection = Tab:CreateSection("Hitbox (PC e Mobile)")

    local hitboxEnabled = false
    local originalSizes = {}
    local hitboxSize = Vector3.new(10, 10, 10) -- Tamanho expandido da Hitbox

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

    local function processPlayer(player)
        if player ~= Player and player.Character and player.Character:FindFirstChild("Head") then
            if hitboxEnabled then
                setHitbox(player, hitboxSize)
            else
                resetHitbox(player)
            end
        end
    end

    local function toggleHitbox(enabled)
        hitboxEnabled = enabled

        if enabled then
           -- Processar jogadores existentes
            for _, player in pairs(game:GetService("Players"):GetPlayers()) do
                processPlayer(player)
            end
        else
            -- Resetar Hitbox de todos os jogadores
            for _, player in pairs(game:GetService("Players"):GetPlayers()) do
                resetHitbox(player)
            end
            originalSizes = {}
        end
    end

    --Conectar ao evento PlayerAdded para novos jogadores
    game:GetService("Players").PlayerAdded:Connect(function(player)
        player.CharacterAdded:Connect(function(character)
            if hitboxEnabled then
                -- Aguardar o personagem ser adicionado e a cabeça estar presente
                character.AncestryChanged:Connect(function()
                    if player.Character:FindFirstChild("Head") then
                        processPlayer(player)
                    end
                end)
                if player.Character:FindFirstChild("Head") then
                   processPlayer(player)
                end
            end
        end)
    end)

    Tab:CreateToggle({
        Name = "Hitbox Expander",
        Description = "Aumenta o tamanho da cabeça dos outros jogadores.",
        CurrentValue = false,
        Section = HitboxSection,
        Callback = function(Value)
            toggleHitbox(Value)
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

    -- Roubar Inventário
    local function roubarInventario()
        local itens = {"AK47", "Uzi", "PARAFAL", "Faca", "IA2", "G3", "Escudo", "C4", "AR-15", "Hi Power", "Natalina", "Tratamento", "Planta Limpa", "Planta Suja", "ArmaÃƒÂ§ÃƒÂ£o de Arma", "PeÃƒÂ§a de Arma", "Glock 17", "Skate" }
        local args = { [1] = "mudaInv", [2] = "2", [4] = "1" }

        while true do
            deletarNotifyGui()
            for i, item in ipairs(itens) do
                if i <= 16 then
                    args[3] = item
                    args[2] = tostring(i)
                    task.spawn(function()
                        game:GetService("ReplicatedStorage"):WaitForChild("Modules"):WaitForChild("InvRemotes"):WaitForChild("InvRequest"):InvokeServer(unpack(args))
                    end)
                end
            end
            wait(0)
        end
    end

    -- Anti Revistar
    local function ativarAntiRevistar()
        local player = game.Players.LocalPlayer
        local character = player.Character or player.CharacterAdded:Wait()
        local humanoid = character:WaitForChild("Humanoid")

        humanoid.HealthChanged:Connect(function(health)
            if health <= 5 then
                player:Kick("ANTI SER REVISTADO")
            end
        end)
    end

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

    -- Auto Revistar
    local autoRevistarAtivo = false
    local autoRevTask

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

    MiniHubTab:CreateSection("Anti Revistar")
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
    -- Your Rayfield menu code ends here
end

-- Connect the Key validation to the FocusLost event of the TextBox
KeyBox.FocusLost:Connect(function(enter)
    if enter then
        local keyDigitada = KeyBox.Text
        local isValid, message = verificarKey(keyDigitada)

        if isValid then
            -- Key is valid: Show a notification, destroy KeyGui, and open the menu
            StarterGui:SetCore("SendNotification", {
                Title = "Acesso Liberado",
                Text = message .. " Bem-vindo, " .. Player.Name,
                Duration = 5
            })

            -- Destroy KeyGui
            KeyGui:Destroy()

            -- Open Rayfield menu
            openRayfieldMenu()
        else
            -- Key is invalid: Show an error notification
            StarterGui:SetCore("SendNotification", {
                Title = "Erro de Acesso",
                Text = message,
                Duration = 5
            })

            -- Clear the KeyBox text
            KeyBox.Text = ""
        end
    end
end)
