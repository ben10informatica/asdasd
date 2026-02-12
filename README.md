local PathfindingService = game:GetService("PathfindingService")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local Root = Character:WaitForChild("HumanoidRootPart")

local IsActive = false

-- 1. FUNÇÃO DE CAMINHADA REAL (Pathfinding)
local function WalkTo(targetPos)
    IsActive = true
    local path = PathfindingService:CreatePath({
        AgentRadius = 3,
        AgentCanJump = true,
        WaypointSpacing = 4
    })

    local success, errorMessage = pcall(function()
        path:ComputeAsync(Root.Position, targetPos)
    end)

    if success and path.Status == Enum.PathStatus.Success then
        local waypoints = path:GetWaypoints()
        
        for i, waypoint in ipairs(waypoints) do
            if not IsActive then break end
            
            -- Move o personagem para o próximo ponto do chão
            Humanoid:MoveTo(waypoint.Position)
            
            -- Espera chegar no ponto antes de ir para o próximo
            local reached = Humanoid.MoveToFinished:Wait(1) 
            
            -- Se demorar demais para chegar, pode estar travado
            if not reached then
                Humanoid.Jump = true
            end
        end
    else
        warn("Caminho não encontrado ou bloqueado.")
    end
    IsActive = false
end

-- 2. INTERFACE (GUI REFORMULADA)
local ScreenGui = Instance.new("ScreenGui", Player.PlayerGui)
local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 220, 0, 300)
Frame.Position = UDim2.new(0.5, -110, 0.4, 0)
Frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Instance.new("UICorner", Frame)

local Title = Instance.new("TextLabel", Frame)
Title.Size = UDim2.new(1, 0, 0, 40)
Title.Text = "Bloxburg Ground Walker"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.BackgroundColor3 = Color3.fromRGB(0, 150, 200)

local Layout = Instance.new("UIListLayout", Frame)
Layout.Padding = UDim.new(0, 5)
Layout.HorizontalAlignment = Enum.HorizontalAlignment.Center

-- Função para criar os botões
local function AddBtn(text, color, callback)
    local b = Instance.new("TextButton", Frame)
    b.Size = UDim2.new(0.9, 0, 0, 40)
    b.Text = text
    b.BackgroundColor3 = color
    b.TextColor3 = Color3.new(1, 1, 1)
    Instance.new("UICorner", b)
    b.MouseButton1Click:Connect(callback)
end

-- BOTÕES
AddBtn("Auto-Pizza (Andar)", Color3.fromRGB(230, 126, 34), function()
    local target = nil
    for _, v in pairs(workspace:GetDescendants()) do
        if (v:IsA("BillboardGui") and v.Enabled) or v.Name == "JobWayPoint" then
            target = v.Adornee or v.Parent
            break
        end
    end
    if target then WalkTo(target.Position) else warn("Seta não encontrada!") end
end)

AddBtn("🛑 PARAR AGORA", Color3.fromRGB(200, 0, 0), function()
    IsActive = false
    Humanoid:MoveTo(Root.Position) -- Faz o personagem parar onde está
end)

AddBtn("Auto-Mood (Home)", Color3.fromRGB(46, 204, 113), function()
    for _, plot in pairs(workspace.Plots:GetChildren()) do
        if plot:FindFirstChild("Owner") and plot.Owner.Value == Player.Name then
            WalkTo(plot.PrimaryPart.Position)
            return
        end
    end
end)

-- Mostrar/Esconder
UserInputService.InputBegan:Connect(function(i, g)
    if i.KeyCode == Enum.KeyCode.Insert and not g then
        Frame.Visible = not Frame.Visible
    end
end)
