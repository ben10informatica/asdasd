-- SERVIÇOS
local Players = game:GetService("Players")
local PathfindingService = game:GetService("PathfindingService")
local UserInputService = game:GetService("UserInputService")

local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local Root = Character:WaitForChild("HumanoidRootPart")

local IsActive = false

-- --- FUNÇÃO DE CAMINHADA MELHORADA ---
local function WalkTo(targetPos)
    IsActive = true
    local path = PathfindingService:CreatePath({AgentRadius = 2, AgentCanJump = true})
    
    -- Tenta calcular o caminho 3 vezes antes de desistir
    local success, _
    for i = 1, 3 do
        success, _ = pcall(function() path:ComputeAsync(Root.Position, targetPos) end)
        if success and path.Status == Enum.PathStatus.Success then break end
        task.wait(0.2)
    end

    if success and path.Status == Enum.PathStatus.Success then
        local waypoints = path:GetWaypoints()
        for _, waypoint in ipairs(waypoints) do
            if not IsActive then break end
            Humanoid:MoveTo(waypoint.Position)
            local arrived = Humanoid.MoveToFinished:Wait(1.5)
            if not arrived then Humanoid.Jump = true end -- Pula se travar
        end
    else
        -- Se falhar o caminho, ele usa a inteligência básica do Roblox
        Humanoid:MoveTo(targetPos)
    end
    IsActive = false
end

-- --- LOCALIZADORES REAIS DO BLOXBURG ---
local function GetPizzaCounter()
    -- Procura a bancada de trabalho na Pizzaria
    local p = workspace:FindFirstChild("Pizza Planet", true) 
    if p then
        local counter = p:FindFirstChild("WorkPlace", true) or p:FindFirstChild("Counter", true)
        if counter then return counter.Position end
    end
    return Vector3.new(454, 13, 282) -- Coordenada padrão atualizada
end

local function GetMoped()
    -- Procura uma moto de entrega disponível
    for _, v in pairs(workspace:GetDescendants()) do
        if v.Name == "Moped" and v:FindFirstChild("VehicleSeat") then
            -- Verifica se a moto está perto da pizzaria
            if (v.PrimaryPart.Position - Vector3.new(454, 13, 260)).Magnitude < 50 then
                return v.PrimaryPart.Position
            end
        end
    end
    return Vector3.new(454, 13, 260) -- Área das motos
end

-- --- INTERFACE ---
local ScreenGui = Instance.new("ScreenGui", Player.PlayerGui)
local Main = Instance.new("Frame", ScreenGui)
Main.Size = UDim2.new(0, 240, 0, 340)
Main.Position = UDim2.new(0.5, -120, 0.4, 0)
Main.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Instance.new("UICorner", Main)

local function AddBtn(txt, col, func)
    local b = Instance.new("TextButton", Main)
    b.Size = UDim2.new(0, 200, 0, 45)
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

-- BOTÕES COM LÓGICA CORRIGIDA
AddBtn("1. IR PEGAR PIZZA", Color3.fromRGB(60, 60, 60), function()
    WalkTo(GetPizzaCounter())
end)

AddBtn("2. IR PARA MOTOS", Color3.fromRGB(100, 100, 100), function()
    WalkTo(GetMoped())
end)

AddBtn("3. ENTREGAR (NA MOTO)", Color3.fromRGB(0, 140, 255), function()
    -- Verificação de segurança: Você está sentado?
    if Humanoid.Sit == false then
        warn("SENTA NA MOTO PRIMEIRO!")
        return
    end
    
    local target = nil
    for _, v in pairs(workspace:GetDescendants()) do
        if v.Name == "JobWayPoint" or (v:IsA("BillboardGui") and v.Enabled) then
            target = v.Adornee or v.Parent
            break
        end
    end
    
    if target then 
        WalkTo(target.Position) 
    else 
        print("Seta não encontrada. Pegue a pizza!") 
    end
end)

AddBtn("🛑 PARAR TUDO", Color3.fromRGB(200, 0, 0), function()
    IsActive = false
    Humanoid:MoveTo(Root.Position)
end)

-- Arrastar e Minimizar (Simplificado)
local mini = false
UserInputService.InputBegan:Connect(function(i, g)
    if i.KeyCode == Enum.KeyCode.Insert and not g then
        mini = not mini
        Main.Visible = not mini
    end
end)
