-- SERVIÇOS
local PathfindingService = game:GetService("PathfindingService")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local Root = Character:WaitForChild("HumanoidRootPart")

local IsActive = false

-- --- LOCALIZADOR DE DESTINOS DINÂMICO ---
local function GetLocation(type)
    if type == "Pizzaria" then
        -- Tenta achar pelo nome do local no mapa do Bloxburg
        local shop = workspace:FindFirstChild("Pizza Planet", true) or workspace:FindFirstChild("Pizzaria", true)
        if shop then
            return shop:IsA("BasePart") and shop.Position or shop:FindFirstChildWhichIsA("BasePart", true).Position
        end
        return Vector3.new(454, 13, 282) -- Fallback se não achar
    elseif type == "Moto" then
        return Vector3.new(454, 13, 260) -- Área das motos
    end
end

-- --- FUNÇÃO DE CAMINHADA (COM CHECAGEM DE ERRO) ---
local function WalkTo(targetPos)
    IsActive = true
    local path = PathfindingService:CreatePath({
        AgentRadius = 3, 
        AgentCanJump = true,
        AgentHeight = 5
    })
    
    local success, _ = pcall(function()
        path:ComputeAsync(Root.Position, targetPos)
    end)
    
    if success and path.Status == Enum.PathStatus.Success then
        local waypoints = path:GetWaypoints()
        for i, waypoint in ipairs(waypoints) do
            if not IsActive then break end
            Humanoid:MoveTo(waypoint.Position)
            
            -- Se o personagem ficar parado mais de 2 segundos, ele pula para desbugar
            local arrived = Humanoid.MoveToFinished:Wait(2)
            if not arrived then Humanoid.Jump = true end
        end
    else
        -- Se o Pathfinding falhar, ele tenta ir em linha reta (Tween curto) por segurança
        print("Caminho complexo falhou, usando aproximação direta...")
        Humanoid:MoveTo(targetPos)
    end
    IsActive = false
end

-- --- INTERFACE (UI) ---
local ScreenGui = Instance.new("ScreenGui", Player.PlayerGui)
ScreenGui.Name = "GeminiFixed"

local Main = Instance.new("Frame", ScreenGui)
Main.Size = UDim2.new(0, 240, 0, 320)
Main.Position = UDim2.new(0.5, -120, 0.4, 0)
Main.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
Instance.new("UICorner", Main)

-- Botão de Minimizar (G)
local Mini = Instance.new("TextButton", ScreenGui)
Mini.Size = UDim2.new(0, 45, 0, 45)
Mini.Position = UDim2.new(0.02, 0, 0.45, 0)
Mini.Text = "G"
Mini.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
Mini.TextColor3 = Color3.new(1, 1, 1)
Mini.Visible = false
Instance.new("UICorner", Mini).CornerRadius = UDim.new(1, 0)

local function CreateBtn(txt, col, func)
    local b = Instance.new("TextButton", Main)
    b.Size = UDim2.new(0.9, 0, 0, 40)
    b.Text = txt
    b.BackgroundColor3 = col
    b.TextColor3 = Color3.new(1, 1, 1)
    b.Font = Enum.Font.GothamBold
    Instance.new("UICorner", b)
    b.MouseButton1Click:Connect(func)
end

Instance.new("UIListLayout", Main).Padding = UDim.new(0, 10)
Instance.new("UIListLayout", Main).HorizontalAlignment = Enum.HorizontalAlignment.Center
Instance.new("UIPadding", Main).PaddingTop = UDim.new(0, 15)

-- BOTÕES
CreateBtn("📦 IR PARA PIZZARIA", Color3.fromRGB(60, 60, 60), function()
    local pos = GetLocation("Pizzaria")
    print("Indo para: ", pos)
    WalkTo(pos)
end)

CreateBtn("🛵 IR PARA MOTOS", Color3.fromRGB(100, 100, 100), function()
    WalkTo(GetLocation("Moto"))
end)

CreateBtn("🍕 ENTREGAR (SETA)", Color3.fromRGB(0, 140, 255), function()
    local target = nil
    for _, v in pairs(workspace:GetDescendants()) do
        if v.Name == "JobWayPoint" or (v:IsA("BillboardGui") and v.Enabled) then
            target = v.Adornee or v.Parent
            break
        end
    end
    if target then WalkTo(target.Position) else warn("Pegue a pizza primeiro!") end
end)

CreateBtn("🛑 PARAR TUDO", Color3.fromRGB(200, 50, 50), function()
    IsActive = false
    Humanoid:MoveTo(Root.Position)
end)

CreateBtn("MINIMIZAR", Color3.fromRGB(40, 40, 40), function()
    Main.Visible = false
    Mini.Visible = true
end)

Mini.MouseButton1Click:Connect(function()
    Main.Visible = true
    Mini.Visible = false
end)

-- Draggable
local dragging = false
Main.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true end end)
UserInputService.InputChanged:Connect(function(i) if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then Main.Position = UDim2.new(0, i.Position.X - 120, 0, i.Position.Y - 20) end end)
UserInputService.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
