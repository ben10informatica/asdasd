-- SERVIÇOS
local PathfindingService = game:GetService("PathfindingService")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local Root = Character:WaitForChild("HumanoidRootPart")

local IsActive = false

-- --- LÓGICA DE MOVIMENTO ---
local function WalkTo(targetPos)
    IsActive = true
    local path = PathfindingService:CreatePath({AgentRadius = 2, AgentCanJump = true})
    path:ComputeAsync(Root.Position, targetPos)
    
    if path.Status == Enum.PathStatus.Success then
        local waypoints = path:GetWaypoints()
        for i = 1, #waypoints do
            if not IsActive then break end
            Humanoid:MoveTo(waypoints[i].Position)
            local timedOut = not Humanoid.MoveToFinished:Wait(2)
            if timedOut then Humanoid.Jump = true end
        end
    end
    IsActive = false
end

-- --- INTERFACE VISUAL (UI) ---
local ScreenGui = Instance.new("ScreenGui", Player.PlayerGui)
ScreenGui.Name = "GeminiHub"
ScreenGui.ResetOnSpawn = false

-- Botão de Minimizar (Flutuante)
local OpenBtn = Instance.new("TextButton", ScreenGui)
OpenBtn.Name = "OpenBtn"
OpenBtn.Size = UDim2.new(0, 50, 0, 50)
OpenBtn.Position = UDim2.new(0.02, 0, 0.45, 0)
OpenBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
OpenBtn.Text = "G"
OpenBtn.TextColor3 = Color3.fromRGB(0, 170, 255)
OpenBtn.Font = Enum.Font.GothamBold
OpenBtn.TextSize = 25
OpenBtn.Visible = false -- Inicia escondido
local BtnCorner = Instance.new("UICorner", OpenBtn)
BtnCorner.CornerRadius = UDim.new(0, 12)

-- Frame Principal
local Main = Instance.new("Frame", ScreenGui)
Main.Name = "Main"
Main.Size = UDim2.new(0, 260, 0, 360)
Main.Position = UDim2.new(0.5, -130, 0.5, -180)
Main.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Main.BorderSizePixel = 0
Main.ClipsDescendants = true
local MainCorner = Instance.new("UICorner", Main)

-- Banner de Topo
local TopBar = Instance.new("Frame", Main)
TopBar.Size = UDim2.new(1, 0, 0, 50)
TopBar.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
local TopCorner = Instance.new("UICorner", TopBar)

local Title = Instance.new("TextLabel", TopBar)
Title.Size = UDim2.new(1, -50, 1, 0)
Title.Position = UDim2.new(0, 15, 0, 0)
Title.Text = "GEMINI HUB"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.BackgroundTransparency = 1

local CloseBtn = Instance.new("TextButton", TopBar)
CloseBtn.Size = UDim2.new(0, 40, 0, 40)
CloseBtn.Position = UDim2.new(1, -45, 0, 5)
CloseBtn.Text = "-"
CloseBtn.TextColor3 = Color3.new(1, 1, 1)
CloseBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
CloseBtn.Font = Enum.Font.GothamBold
local CloseCorner = Instance.new("UICorner", CloseBtn)

-- Conteúdo
local Content = Instance.new("Frame", Main)
Content.Size = UDim2.new(1, 0, 1, -60)
Content.Position = UDim2.new(0, 0, 0, 60)
Content.BackgroundTransparency = 1

local Layout = Instance.new("UIListLayout", Content)
Layout.Padding = UDim.new(0, 12)
Layout.HorizontalAlignment = Enum.HorizontalAlignment.Center

-- Função para criar botões modernos
local function AddMenuButton(txt, color, func)
    local b = Instance.new("TextButton", Content)
    b.Size = UDim2.new(0.9, 0, 0, 45)
    b.Text = txt
    b.BackgroundColor3 = color
    b.TextColor3 = Color3.new(1, 1, 1)
    b.Font = Enum.Font.GothamMedium
    b.TextSize = 14
    b.AutoButtonColor = true
    local c = Instance.new("UICorner", b)
    c.CornerRadius = UDim.new(0, 8)
    
    b.MouseButton1Click:Connect(func)
    return b
end

-- --- BOTÕES E AÇÕES ---

AddMenuButton("📦 PEGAR PIZZA / MOTO", Color3.fromRGB(60, 60, 60), function()
    IsActive = true
    print("Indo para Pizzaria...")
    WalkTo(Vector3.new(454, 13, 282)) -- Bancada
    task.wait(1)
    print("Indo para as Motos...")
    WalkTo(Vector3.new(454, 13, 260)) -- Motos
end)

AddMenuButton("🍕 ENTREGAR (SIGA A SETA)", Color3.fromRGB(0, 140, 255), function()
    IsActive = true
    local target = nil
    for _, v in pairs(workspace:GetDescendants()) do
        if v.Name == "JobWayPoint" or (v:IsA("BillboardGui") and v.Enabled) then
            target = v.Adornee or v.Parent
            break
        end
    end
    if target then WalkTo(target.Position) else print("Seta não encontrada!") end
end)

AddMenuButton("🏠 AUTO-MOOD (CASA)", Color3.fromRGB(46, 204, 113), function()
    IsActive = true
    for _, plot in pairs(workspace.Plots:GetChildren()) do
        if plot:FindFirstChild("Owner") and plot.Owner.Value == Player.Name then
            WalkTo(plot.PrimaryPart.Position)
        end
    end
end)

AddMenuButton("🛑 PARAR TUDO", Color3.fromRGB(180, 0, 0), function()
    IsActive = false
    Humanoid:MoveTo(Root.Position)
end)

-- --- SISTEMA DE MINIMIZAR ---
local function ToggleUI()
    local visible = Main.Visible
    if visible then
        Main:TweenPosition(UDim2.new(0.5, -130, 1, 50), "Out", "Quart", 0.5, true)
        task.wait(0.5)
        Main.Visible = false
        OpenBtn.Visible = true
    else
        Main.Visible = true
        OpenBtn.Visible = false
        Main:TweenPosition(UDim2.new(0.5, -130, 0.5, -180), "Out", "Back", 0.5, true)
    end
end

CloseBtn.MouseButton1Click:Connect(ToggleUI)
OpenBtn.MouseButton1Click:Connect(ToggleUI)

-- Arrastar Menu
local dragging, dragInput, dragStart, startPos
Main.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = Main.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

print("Gemini Hub carregado. Clique no '-' para minimizar.")
