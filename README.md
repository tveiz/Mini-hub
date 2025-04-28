-- Tenta carregar o Rayfield e lida com erros
local success, rayfield = pcall(function()
    return loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
end)

if not success then
    warn("Erro ao carregar o Rayfield Menu: " .. (rayfield or "Erro desconhecido"))
    -- Pode adicionar aqui uma mensagem no jogo (usando `game.StarterGui:SetCore("SendNotification",...)` se quiser)
    return -- Aborta a execução do script se o Rayfield não carregar
end

-- Cria o menu Rayfield
local Window = rayfield:CreateWindow({
    Name = "Menu Salvar Posição e Farm",
    LoadingInfo = {
        Percentage = 75,
        Text = "Carregando o Menu...",
        NeedPlayers = true
    },
    Discord = {
        Webhook = "", -- Removido
        Invite = "", -- Removido
        Remember = true
    },
    KeySystem = false, -- Desativado o sistema de chaves
    KeySettings = {
        Title = "Chave do Menu",
        Subtitle = "Insira a chave para acessar o menu:",
        Note = "A chave é fornecida no meu Discord.",
        FileName = "RayfieldKey",
        SaveKey = true,
        Creator = "SeuNome",
        Key = "" -- Deixado vazio
    }
})

-- Cria a aba "Salvar Posição e Farm"
local Tab = Window:CreateTab("Farm Peças", 47635) -- Pode mudar o ícone

-- Variáveis
local savedPosition = nil
local isSavingPosition = false
local isFarming = false

-- Funções auxiliares (extraídas do script da Luna)
local function fireAllProximityPrompts()
    local objetosMissao = workspace:FindFirstChild("MapaGeral")
    if objetosMissao then
        objetosMissao = objetosMissao:FindFirstChild("FavelaV2")
        if objetosMissao then
            objetosMissao = objetosMissao:FindFirstChild("objetosMissao")
            if objetosMissao then
                for _, prompt in ipairs(objetosMissao:GetDescendants()) do
                    if prompt:IsA("ProximityPrompt") then
                        prompt.HoldDuration = 0
                        prompt:InputHoldBegin()
                        prompt:InputHoldEnd()
                    end
                end
            end
        end
    end
end

local function findPlayerProximityPrompt()
    local playerName = game.Players.LocalPlayer.Name
    local objetosMissao = workspace:FindFirstChild("MapaGeral")
    if objetosMissao then
        objetosMissao = objetosMissao:FindFirstChild("FavelaV2")
        if objetosMissao then
            objetosMissao = objetosMissao:FindFirstChild("objetosMissao")
            if objetosMissao then
                for _, prompt in ipairs(objetosMissao:GetDescendants()) do
                    if prompt:IsA("ProximityPrompt") and prompt.Name == playerName then
                        return prompt
                    end
                end
            end
        end
    end
    return nil
end

-- Função para salvar a posição
local function SavePosition()
    if isSavingPosition and game.Players.LocalPlayer and game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        savedPosition = game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame
    end
end

-- Função para travar o jogador na posição salva
local function LockPlayerToPosition()
    if isSavingPosition and savedPosition and game.Players.LocalPlayer and game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = savedPosition
    end
end

-- Toggle para salvar posição
Tab:CreateToggle({
    Name = "Salvar Posição (Toggle)",
    Info = "Quando ativado, salva sua posição e te teleporta de volta a ela ao tentar se mover.",
    Callback = function(State)
        isSavingPosition = State
        if isSavingPosition then
            SavePosition() -- Salva a posição imediatamente ao ativar
        else
            savedPosition = nil -- Limpa a posição quando desativa
        end
    end,
})

-- Botão para Salvar a Posição (Adicionado)
Tab:CreateButton({
    Name = "Salvar Posição (Botão)",
    Description = "Salva a posição atual.",
    Callback = function()
        SavePosition()
    end,
})

-- Toggle para Iniciar/Parar o Farm de Peças
Tab:CreateToggle({
    Name = "Iniciar Farm de Peças",
    Info = "Ativa o farm automático de peças. Requer uma posição salva.",
    CurrentValue = false,
    Callback = function(Value)
        isFarming = Value

        if isFarming then
            -- Inicia o loop do farm
            spawn(function()
                while isFarming do
                    if not savedPosition then
                        warn("Farm de Peças: Nenhuma posição salva. Desativando o farm.")
                        isFarming = false
                        break
                    end

                    local player = game.Players.LocalPlayer
                    local character = player.Character
                    if not character or not character:FindFirstChild("HumanoidRootPart") then
                        warn("Farm de Peças: Personagem não encontrado. Desativando o farm.")
                        isFarming = false
                        break
                    end

                    fireAllProximityPrompts()
                    character.HumanoidRootPart.CFrame = savedPosition

                    task.wait(0.35)

                    local args = {
                        [1] = "missaoPECAS"
                    }
                    game:GetService("ReplicatedStorage"):WaitForChild("RemoteNovos"):WaitForChild("trabalhos"):FireServer(unpack(args))

                    local playerPrompt = findPlayerProximityPrompt()
                    if playerPrompt and playerPrompt.Parent then
                        character.HumanoidRootPart.CFrame = CFrame.new(playerPrompt.Parent.Position)
                    end

                    task.wait(0.2)
                end
            end)
        end
    end,
})

-- Conecta o heartbeat para manter o jogador na posição salva
game:GetService("RunService").Heartbeat:Connect(function()
    LockPlayerToPosition()
end)

-- Conecta o input para salvar a posição quando o botão for pressionado (opcional)
game:GetService("UserInputService").InputBegan:Connect(function(input, gameProcessedEvent)
    if not gameProcessedEvent and input.UserInputType == Enum.UserInputType.MouseButton1 then
        SavePosition()
    end
end)
