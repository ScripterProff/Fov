-- Vision Hub (Mobile Edition)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LP = Players.LocalPlayer

-- Configurações do Aimbot
local Settings = {
    Enabled = false,
    AimPart = "Head",
    FOV = 100,
    Smoothness = 0.15,
    ShowFOV = true,
    Minimized = false
}

-- Função para identificar inimigos
local function IsEnemy(player)
    if LP.Team and player.Team and LP.Team == player.Team then
        return false
    end
    return true
end

-- Função para buscar alvo mais próximo do centro da tela
local function GetClosest()
    local closest, dist = nil, Settings.FOV
    local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and IsEnemy(plr) and plr.Character and plr.Character:FindFirstChild(Settings.AimPart) then
            local pos = plr.Character[Settings.AimPart].Position
            local screen, onScreen = Camera:WorldToViewportPoint(pos)
            if onScreen then
                local magnitude = (Vector2.new(screen.X, screen.Y) - screenCenter).Magnitude
                if magnitude < dist then
                    dist = magnitude
                    closest = plr
                end
            end
        end
    end
    return closest
end

-- Loop do Aimbot
RunService.RenderStepped:Connect(function()
    if not Settings.Enabled then return end
    local target = GetClosest()
    if target and target.Character and target.Character:FindFirstChild(Settings.AimPart) then
        local pos = target.Character[Settings.AimPart].Position
        local camPos = Camera.CFrame.Position
        local direction = (pos - camPos).Unit
        local cf = CFrame.new(camPos, camPos + direction)
        Camera.CFrame = Camera.CFrame:Lerp(cf, Settings.Smoothness)
    end
end)

-- Círculo de FOV
local circle = Drawing.new("Circle")
circle.Color = Color3.new(1, 1, 1)
circle.Thickness = 1.5
circle.NumSides = 64
circle.Filled = false
circle.Visible = true

RunService.RenderStepped:Connect(function()
    circle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    circle.Radius = Settings.FOV
    circle.Visible = Settings.ShowFOV
end)

-- GUI (interface)
local gui = Instance.new("ScreenGui", LP:WaitForChild("PlayerGui"))
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 220, 0, 230)
frame.Position = UDim2.new(0.02, 0, 0.2, 0)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true

-- Título
local title = Instance.new("TextLabel", frame)
title.Text = "Vision Hub (Mobile)"
title.Size = UDim2.new(1, -40, 0, 30)
title.Position = UDim2.new(0, 5, 0, 0)
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundTransparency = 1
title.Font = Enum.Font.SourceSansBold
title.TextSize = 18
title.TextXAlignment = Enum.TextXAlignment.Left

-- Botão Fechar
local close = Instance.new("TextButton", frame)
close.Text = "X"
close.Size = UDim2.new(0, 30, 0, 30)
close.Position = UDim2.new(1, -35, 0, 0)
close.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
close.TextColor3 = Color3.new(1,1,1)
close.MouseButton1Click:Connect(function()
    gui:Destroy()
    circle:Remove()
end)

-- Botão Minimizar
local minimize = Instance.new("TextButton", frame)
minimize.Text = "-"
minimize.Size = UDim2.new(0, 30, 0, 30)
minimize.Position = UDim2.new(1, -70, 0, 0)
minimize.BackgroundColor3 = Color3.fromRGB(255, 165, 0)
minimize.TextColor3 = Color3.new(1,1,1)
minimize.MouseButton1Click:Connect(function()
    Settings.Minimized = not Settings.Minimized
    for _, v in ipairs(frame:GetChildren()) do
        if v:IsA("TextButton") and v ~= close and v ~= minimize then
            v.Visible = not Settings.Minimized
        end
    end
end)

-- Função para botões
local function createButton(text, posY, callback)
    local btn = Instance.new("TextButton", frame)
    btn.Text = text
    btn.Size = UDim2.new(1, -10, 0, 30)
    btn.Position = UDim2.new(0, 5, 0, posY)
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.SourceSans
    btn.TextSize = 16
    btn.MouseButton1Click:Connect(callback)
    return btn
end

-- Botões do menu
local toggle = createButton("Aimbot: OFF", 40, function()
    Settings.Enabled = not Settings.Enabled
    toggle.Text = "Aimbot: " .. (Settings.Enabled and "ON" or "OFF")
end)

local partBtn = createButton("Parte: Head", 75, function()
    Settings.AimPart = Settings.AimPart == "Head" and "HumanoidRootPart" or "Head"
    partBtn.Text = "Parte: " .. Settings.AimPart
end)

local fovBtn = createButton("FOV: 100", 110, function()
    Settings.FOV = Settings.FOV + 20
    if Settings.FOV > 200 then Settings.FOV = 40 end
    fovBtn.Text = "FOV: " .. Settings.FOV
end)

local smoothBtn = createButton("Smooth: 0.15", 145, function()
    Settings.Smoothness = Settings.Smoothness + 0.05
    if Settings.Smoothness > 0.5 then Settings.Smoothness = 0.05 end
    smoothBtn.Text = "Smooth: " .. string.format("%.2f", Settings.Smoothness)
end)
