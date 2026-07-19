local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local localPlayer = Players.LocalPlayer

-- Configurações globais do Hub
local isActive = false
local hitboxSize = 5 -- Tamanho padrão
local hubVisible = true

-- ==========================================
-- 1. CRIAÇÃO DA INTERFACE (GUI)
-- ==========================================
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BHB_HitboxHub"
screenGui.ResetOnSpawn = false
screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 250, 0, 160)
mainFrame.Position = UDim2.new(0.5, -125, 0.5, -80)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
mainFrame.BorderSizePixel = 0
mainFrame.Visible = hubVisible
mainFrame.Parent = screenGui

local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 8)
uiCorner.Parent = mainFrame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundTransparency = 1
title.Text = "BHB Local Hitbox"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 16
title.Parent = mainFrame

local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0.8, 0, 0, 35)
toggleBtn.Position = UDim2.new(0.1, 0, 0.25, 0)
toggleBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
toggleBtn.Text = "Status: DESATIVADO"
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextSize = 14
toggleBtn.Parent = mainFrame

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 6)
btnCorner.Parent = toggleBtn

local sizeInput = Instance.new("TextBox")
sizeInput.Size = UDim2.new(0.8, 0, 0, 35)
sizeInput.Position = UDim2.new(0.1, 0, 0.55, 0)
sizeInput.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
sizeInput.Text = tostring(hitboxSize)
sizeInput.PlaceholderText = "Tamanho (ex: 5, 10)"
sizeInput.TextColor3 = Color3.fromRGB(255, 255, 255)
sizeInput.Font = Enum.Font.Gotham
sizeInput.TextSize = 14
sizeInput.Parent = mainFrame

local inputCorner = Instance.new("UICorner")
inputCorner.CornerRadius = UDim.new(0, 6)
inputCorner.Parent = sizeInput

local infoText = Instance.new("TextLabel")
infoText.Size = UDim2.new(1, 0, 0, 20)
infoText.Position = UDim2.new(0, 0, 1, -25)
infoText.BackgroundTransparency = 1
infoText.Text = "Pressione [L] para ocultar/mostrar"
infoText.TextColor3 = Color3.fromRGB(150, 150, 150)
infoText.Font = Enum.Font.Gotham
infoText.TextSize = 11
infoText.Parent = mainFrame

-- ==========================================
-- 2. LÓGICA DA HITBOX DO JOGADOR LOCAL
-- ==========================================
local function atualizarHitboxLocal()
    local char = localPlayer.Character
    if not char then return end
    
    local torso = char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso")
    if not torso then return end

    -- Remove a hitbox visual anterior, se existir
    local hitboxAntiga = char:FindFirstChild("HitboxVisualExpandida")
    if hitboxAntiga then
        hitboxAntiga:Destroy()
    end

    if isActive then
        -- Cria a nova peça que servirá como hitbox expandida e visualizador
        local novaHitbox = Instance.new("Part")
        novaHitbox.Name = "HitboxVisualExpandida"
        novaHitbox.Size = Vector3.new(hitboxSize, hitboxSize, hitboxSize)
        
        -- Configurações visuais (Material ForceField para ver através dela facilmente)
        novaHitbox.Material = Enum.Material.ForceField
        novaHitbox.Color = Color3.fromRGB(0, 255, 150) -- Verde ciano
        novaHitbox.Transparency = 0.2
        
        -- Configurações físicas cruciais para não bugar o personagem
        novaHitbox.CanCollide = false 
        novaHitbox.Massless = true
        novaHitbox.CFrame = torso.CFrame
        
        -- Solda a hitbox ao tronco
        local weld = Instance.new("WeldConstraint")
        weld.Part0 = torso
        weld.Part1 = novaHitbox
        weld.Parent = novaHitbox
        
        novaHitbox.Parent = char
    end
end

-- ==========================================
-- 3. CONEXÕES DOS BOTÕES E EVENTOS
-- ==========================================
toggleBtn.MouseButton1Click:Connect(function()
    isActive = not isActive
    if isActive then
        toggleBtn.Text = "Status: ATIVADO"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
    else
        toggleBtn.Text = "Status: DESATIVADO"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    end
    atualizarHitboxLocal()
end)

sizeInput.FocusLost:Connect(function()
    local novoTamanho = tonumber(sizeInput.Text)
    if novoTamanho and novoTamanho > 0 then
        hitboxSize = novoTamanho
        if isActive then
            atualizarHitboxLocal()
        end
    else
        sizeInput.Text = tostring(hitboxSize)
    end
end)

-- Garante que a hitbox volte se o seu personagem morrer/resetar
localPlayer.CharacterAdded:Connect(function()
    task.wait(0.5)
    if isActive then
        atualizarHitboxLocal()
    end
end)

-- ==========================================
-- 4. SISTEMA DE OCULTAR MENU (TECLA L)
-- ==========================================
UserInputService.InputBegan:Connect(function(input, isProcessed)
    if isProcessed and not sizeInput:IsFocused() then return end

    if input.KeyCode == Enum.KeyCode.L then
        hubVisible = not hubVisible
        mainFrame.Visible = hubVisible
    end
end)
