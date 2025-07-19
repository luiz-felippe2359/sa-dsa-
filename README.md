local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Cria a GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "DrillControlGui"
screenGui.Parent = playerGui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 250, 0, 140)
frame.Position = UDim2.new(0.5, -125, 0.5, -70)
frame.AnchorPoint = Vector2.new(0.5, 0.5)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
frame.BorderSizePixel = 0
frame.Parent = screenGui

-- Adiciona cantos arredondados
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = frame

-- Adiciona sombra/borda
local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(70, 130, 255)
stroke.Thickness = 2
stroke.Parent = frame

-- Header da GUI (área para arrastar)
local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 35)
header.Position = UDim2.new(0, 0, 0, 0)
header.BackgroundColor3 = Color3.fromRGB(70, 130, 255)
header.BorderSizePixel = 0
header.Parent = frame

local headerCorner = Instance.new("UICorner")
headerCorner.CornerRadius = UDim.new(0, 12)
headerCorner.Parent = header

-- Corrige os cantos do header
local headerFix = Instance.new("Frame")
headerFix.Size = UDim2.new(1, 0, 0, 12)
headerFix.Position = UDim2.new(0, 0, 1, -12)
headerFix.BackgroundColor3 = Color3.fromRGB(70, 130, 255)
headerFix.BorderSizePixel = 0
headerFix.Parent = header

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -10, 1, 0)
title.Position = UDim2.new(0, 10, 0, 0)
title.Text = "Drill Control"
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextScaled = true
title.Font = Enum.Font.GothamBold
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = header

-- Botão principal
local button = Instance.new("TextButton")
button.Size = UDim2.new(0.85, 0, 0, 45)
button.Position = UDim2.new(0.075, 0, 0.4, 0)
button.Text = "Iniciar Drilling"
button.BackgroundColor3 = Color3.fromRGB(40, 180, 70)
button.TextColor3 = Color3.fromRGB(255, 255, 255)
button.BorderSizePixel = 0
button.Font = Enum.Font.GothamBold
button.TextScaled = true
button.Parent = frame

-- Cantos arredondados do botão
local buttonCorner = Instance.new("UICorner")
buttonCorner.CornerRadius = UDim.new(0, 8)
buttonCorner.Parent = button

-- Status label
local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(0.85, 0, 0, 20)
statusLabel.Position = UDim2.new(0.075, 0, 0.75, 0)
statusLabel.Text = "Status: Inativo"
statusLabel.BackgroundTransparency = 1
statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
statusLabel.TextScaled = true
statusLabel.Font = Enum.Font.Gotham
statusLabel.Parent = frame

-- Sistema de arrastar
local dragging = false
local dragStart = nil
local startPos = nil

local function updateInput(input)
    local delta = input.Position - dragStart
    local position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    frame.Position = position
end

header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

header.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        if dragging then
            updateInput(input)
        end
    end
end)

-- Efeitos visuais do botão
local function animateButton(targetColor, targetSize)
    local colorTween = TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = targetColor})
    local sizeTween = TweenService:Create(button, TweenInfo.new(0.1), {Size = targetSize})
    colorTween:Play()
    sizeTween:Play()
end

button.MouseEnter:Connect(function()
    if not isDrilling then
        animateButton(Color3.fromRGB(50, 190, 80), UDim2.new(0.87, 0, 0, 47))
    end
end)

button.MouseLeave:Connect(function()
    if not isDrilling then
        animateButton(Color3.fromRGB(40, 180, 70), UDim2.new(0.85, 0, 0, 45))
    else
        animateButton(Color3.fromRGB(200, 50, 50), UDim2.new(0.85, 0, 0, 45))
    end
end)

-- Variáveis de controle (não modificadas)
local isDrilling = false
local drillCycle = nil

-- Função para enviar o sinal de início de drilling (não modificada)
local function startDrilling()
    local args = {
        [1] = {
            ["clickedX"] = 1.2763991045138483,
            ["kind"] = "drillRequest"
        }
    }
    
    ReplicatedStorage.Packages.Knit.Services.TowerService.RE.remote:FireServer(unpack(args))
end

-- Função para enviar o sinal de parada de drilling (não modificada)
local function stopDrilling()
    local args = {
        [1] = {
            ["kind"] = "drillStopped"
        }
    }
    
    ReplicatedStorage.Packages.Knit.Services.TowerService.RE.remote:FireServer(unpack(args))
end

-- Função que controla o ciclo de drilling (não modificada)
local function runDrillCycle()
    if not isDrilling then return end
    
    startDrilling()
    
    -- Espera 10 segundos e para o drilling
    task.wait(10)
    if not isDrilling then return end
    
    stopDrilling()
    
    -- Espera 5 segundos e reinicia o ciclo
    task.wait(5)
    if isDrilling then
        runDrillCycle()
    end
end

-- Manipulador de clique do botão (modificado apenas para incluir efeitos visuais)
button.MouseButton1Click:Connect(function()
    isDrilling = not isDrilling
    
    if isDrilling then
        button.Text = "Parar Drilling"
        button.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        statusLabel.Text = "Status: Ativo - Perfurando..."
        statusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
        runDrillCycle()
    else
        button.Text = "Iniciar Drilling"
        button.BackgroundColor3 = Color3.fromRGB(40, 180, 70)
        statusLabel.Text = "Status: Inativo"
        statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
        stopDrilling() -- Garante que o drilling é parado imediatamente
    end
end)
