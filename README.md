-- Moons Look (fixo no player até desativar)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- GUI principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MoonsLookGui"
ScreenGui.Parent = game:GetService("CoreGui")

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 200, 0, 120)
MainFrame.Position = UDim2.new(0.4, 0, 0.2, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundColor3 = Color3.fromRGB(40, 0, 60)
Title.Text = "MOONS LOOK"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 18
Title.Parent = MainFrame

local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(1, -20, 0, 30)
ToggleButton.Position = UDim2.new(0, 10, 0, 40)
ToggleButton.BackgroundColor3 = Color3.fromRGB(60, 0, 100)
ToggleButton.Text = "Ativar Look"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.TextSize = 16
ToggleButton.Parent = MainFrame

local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 100, 0, 30)
CloseButton.Position = UDim2.new(0.25, 0, 0.8, 0)
CloseButton.BackgroundColor3 = Color3.fromRGB(150, 0, 50)
CloseButton.Text = "Fechar Menu"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.Font = Enum.Font.SourceSansBold
CloseButton.TextSize = 14
CloseButton.Active = true
CloseButton.Draggable = true
CloseButton.Parent = MainFrame

-- Variáveis
local lookEnabled = false
local targetPlayer = nil
local highlight = nil

-- Função para criar Highlight
local function createHighlight(character)
    local h = Instance.new("Highlight")
    h.Adornee = character
    h.FillColor = Color3.fromRGB(255, 0, 100)
    h.OutlineColor = Color3.fromRGB(255, 255, 255)
    h.FillTransparency = 0.5
    h.OutlineTransparency = 0
    h.Parent = character
    return h
end

-- Função para pegar player que a câmera está mirando (apenas na ativação)
local function getLookedPlayer()
    local middle = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    local unitRay = Camera:ViewportPointToRay(middle.X, middle.Y, 0)
    local raycastParams = RaycastParams.new()
    raycastParams.FilterDescendantsInstances = {LocalPlayer.Character}
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    local result = workspace:Raycast(unitRay.Origin, unitRay.Direction * 500, raycastParams)

    if result and result.Instance then
        local model = result.Instance:FindFirstAncestorOfClass("Model")
        if model and Players:GetPlayerFromCharacter(model) then
            return Players:GetPlayerFromCharacter(model)
        end
    end
    return nil
end

-- Loop do Look
RunService.RenderStepped:Connect(function()
    if lookEnabled and targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local targetPos = targetPlayer.Character.HumanoidRootPart.Position
        -- Mantém a câmera olhando pro player, mesmo com parede no meio
        Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPos)
    end
end)

-- Botão de ativar/desativar
ToggleButton.MouseButton1Click:Connect(function()
    if not lookEnabled then
        local newTarget = getLookedPlayer()
        if newTarget then
            targetPlayer = newTarget
            lookEnabled = true
            ToggleButton.Text = "Desativar Look"
            if highlight then highlight:Destroy() end
            if targetPlayer.Character then
                highlight = createHighlight(targetPlayer.Character)
            end
        else
            ToggleButton.Text = "Nenhum player mirando"
        end
    else
        lookEnabled = false
        ToggleButton.Text = "Ativar Look"
        if highlight then highlight:Destroy() highlight = nil end
        targetPlayer = nil
    end
end)

-- Botão de fechar
CloseButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
    local ReopenButton = Instance.new("TextButton")
    ReopenButton.Size = UDim2.new(0, 150, 0, 40)
    ReopenButton.Position = UDim2.new(0.5, -75, 0.9, 0)
    ReopenButton.BackgroundColor3 = Color3.fromRGB(50, 0, 120)
    ReopenButton.Text = "Reabrir Moons Look"
    ReopenButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    ReopenButton.Font = Enum.Font.SourceSansBold
    ReopenButton.TextSize = 16
    ReopenButton.Active = true
    ReopenButton.Draggable = true
    ReopenButton.Parent = ScreenGui

    ReopenButton.MouseButton1Click:Connect(function()
        MainFrame.Visible = true
        ReopenButton:Destroy()
    end)
end)
