local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Teams = game:GetService("Teams")

-- Criando a UI (Interface Gráfica)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.CoreGui

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 200, 0, 50)
Frame.Position = UDim2.new(0.5, -100, 0, 50)
Frame.BackgroundColor3 = Color3.new(0, 0, 0)
Frame.BackgroundTransparency = 0.5
Frame.Parent = ScreenGui
Frame.Active = true
Frame.Draggable = true

local Button = Instance.new("TextButton")
Button.Size = UDim2.new(1, 0, 1, 0)
Button.Position = UDim2.new(0, 0, 0, 0)
Button.Text = "Ativar ESP"
Button.BackgroundColor3 = Color3.new(1, 1, 1)
Button.Parent = Frame

local espAtivo = false

-- Função para verificar se o jogador é do mesmo time
local function ehAliado(jogador)
    if jogador.Team and LocalPlayer.Team then
        return jogador.Team == LocalPlayer.Team
    end
    return false
end

-- Criar ESP para um jogador
local function criarESP(jogador)
    if jogador == LocalPlayer or not jogador.Character then return end  

    local character = jogador.Character
    local head = character:FindFirstChild("Head")
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not head or not humanoidRootPart then return end

    -- Criar BillboardGui para exibir nome e time
    local billGui = Instance.new("BillboardGui")
    billGui.Size = UDim2.new(0, 200, 0, 50)
    billGui.Adornee = head
    billGui.StudsOffset = Vector3.new(0, 2, 0)
    billGui.Parent = head

    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.TextScaled = true
    textLabel.TextStrokeTransparency = 0
    textLabel.Parent = billGui

    -- Criar linha de trace (Beam)
    local attachment1 = Instance.new("Attachment", head)
    local attachment2 = Instance.new("Attachment", LocalPlayer.Character.Head)

    local beam = Instance.new("Beam")
    beam.Attachment0 = attachment1
    beam.Attachment1 = attachment2
    beam.Width0 = 0.1
    beam.Width1 = 0.1
    beam.Parent = head

    -- Criar Caixa (Bounding Box)
    local box = Instance.new("BoxHandleAdornment")
    box.Size = humanoidRootPart.Size + Vector3.new(1, 2, 1)
    box.Adornee = humanoidRootPart
    box.AlwaysOnTop = true
    box.ZIndex = 5
    box.Transparency = 0.3
    box.Parent = humanoidRootPart

    -- Definir cores baseadas no time correto
    if ehAliado(jogador) then
        textLabel.Text = "[Aliado] " .. jogador.Name
        textLabel.TextColor3 = Color3.new(0, 1, 0) -- Verde
        beam.Color = ColorSequence.new(Color3.new(0, 1, 0)) -- Verde para aliados
        box.Color3 = Color3.new(0, 1, 0) -- Caixa verde para aliados
    else
        textLabel.Text = "[Inimigo] " .. jogador.Name
        textLabel.TextColor3 = Color3.new(1, 0, 0) -- Vermelho
        beam.Color = ColorSequence.new(Color3.new(1, 0, 0)) -- Vermelho para inimigos
        box.Color3 = Color3.new(1, 0, 0) -- Caixa vermelha para inimigos
    end

    return billGui, beam, box
end

-- Ativar/desativar ESP
local function ativarESP()
    if espAtivo then
        -- Desativar ESP
        for _, player in pairs(Players:GetPlayers()) do
            if player.Character then
                local head = player.Character:FindFirstChild("Head")
                local humanoidRootPart = player.Character:FindFirstChild("HumanoidRootPart")
                if head then
                    for _, obj in pairs(head:GetChildren()) do
                        if obj:IsA("BillboardGui") or obj:IsA("Beam") or obj:IsA("Attachment") then
                            obj:Destroy()
                        end
                    end
                end
                if humanoidRootPart then
                    for _, obj in pairs(humanoidRootPart:GetChildren()) do
                        if obj:IsA("BoxHandleAdornment") then
                            obj:Destroy()
                        end
                    end
                end
            end
        end
        Button.Text = "Ativar ESP"
        espAtivo = false
    else
        -- Ativar ESP
        for _, player in pairs(Players:GetPlayers()) do
            criarESP(player)
        end
        Button.Text = "Desativar ESP"
        espAtivo = true
    end
end

Button.MouseButton1Click:Connect(ativarESP)

-- Atualizar ESP quando novos jogadores entrarem
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        if espAtivo then
            criarESP(player)
        end
    end)
end)
