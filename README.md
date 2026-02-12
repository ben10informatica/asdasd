--[[ 
    SCRIPT COMPLETO: TRABALHO + HUMOR + BUILD + MENU
    Instruções:
    - Pressione 'Insert' para ocultar/mostrar o menu.
    - O Auto-Pizza busca a seta amarela automaticamente.
]]

local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Root = Character:WaitForChild("HumanoidRootPart")

-- CONFIGURAÇÕES
local WalkingSpeed = 30 -- Velocidade segura
local IsMoving = false
local CurrentTween = nil

-- 1. FUNÇÃO DE ALTURA INTELIGENTE (Raycast)
local function GetGroundHeight(pos)
    local rayOrigin = pos + Vector3.new(0, 50, 0)
    local rayDirection = Vector3.new(0, -100, 0)
    local params = RaycastParams.new()
    params.FilterDescendantsInstances = {Character}
    local result = workspace:Raycast(rayOrigin, rayDirection, params)
    return result and (result.Position.Y + 3.5) or pos.Y
end

-- 2. FUNÇÃO DE MOVIMENTO (Tween)
local function SmartMove(targetPos)
    if CurrentTween then CurrentTween:Cancel() end
    IsMoving = true
    local finalPos = Vector3.new(targetPos.X, GetGroundHeight(targetPos), targetPos.Z)
    local distance = (Root.Position - finalPos).Magnitude
    local duration = distance / WalkingSpeed
    
    CurrentTween = TweenService:Create(Root, TweenInfo.new(duration, Enum.EasingStyle.Linear), {CFrame = CFrame.new(finalPos)})
    CurrentTween:Play()
    CurrentTween.Completed:Connect(function() IsMoving = false end)
end

-- 3. LÓGICA AUTO-PIZZA (Busca o Marcador)
local function AutoPizza()
    local target = nil
    for _, v in pairs(workspace:GetDescendants()) do
        if (v:IsA("BillboardGui") and v.Enabled) or v.Name == "JobWayPoint" then
            target = v.Adornee or v.Parent
            break
        end
    end
    
    if target then
        print("Seta encontrada! Indo para o cliente...")
        SmartMove(target.Position)
    else
        warn("Sem destino. Indo buscar pizza na bancada...")
        SmartMove(Vector3.new(455, 13, 275)) -- Coordenada aproximada da Pizzaria
    end
end

-- 4. INTERFACE GRÁFICA
local ScreenGui = Instance.new("ScreenGui", Player.PlayerGui)
ScreenGui.Name = "BloxburgHub"

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 220, 0, 350)
MainFrame.Position = UDim2.new(0.5, -110, 0.3, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true

local Corner = Instance.new("UICorner", MainFrame)

local Title = Instance.new("TextLabel", MainFrame)
Title.Size = UDim2.new(1, 0, 0, 40)
Title.Text = "BLOXBURG MASTER"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
local TitleCorner = Instance.new("UICorner", Title)

local Layout = Instance.new("UIListLayout", MainFrame)
Layout.Padding = UDim.new(0, 8)
Layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
local Pad = Instance.new("UIPadding", MainFrame)
Pad.PaddingTop = UDim.new(0, 50)

-- Criador de Botões
local function AddButton(text, color, callback)
    local btn = Instance.new("TextButton", MainFrame)
    btn.Text = text
    btn.Size = UDim2.new(0, 190, 0, 40)
    btn.BackgroundColor3 = color
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.GothamMedium
    Instance.new("UICorner", btn)
    btn.MouseButton1Click:Connect(callback)
end

-- ADICIONANDO AS FUNÇÕES AO MENU
AddButton("Auto-Pizza (Ir até Seta)", Color3.fromRGB(230, 126, 34), function()
    AutoPizza()
end)

AddButton("Auto-Mood (Ir p/ Casa)", Color3.fromRGB(46, 204, 113), function()
    print("Indo recuperar humor...")
    -- O script tenta achar o terreno (Plot) do jogador
    for _, plot in pairs(workspace.Plots:GetChildren()) do
        if plot:FindFirstChild("Owner") and plot.Owner.Value == Player.Name then
            SmartMove(plot.PrimaryPart.Position + Vector3.new(0, 5, 0))
            return
        end
    end
    warn("Não encontrei seu terreno automaticamente.")
end)

local noclip = false
AddButton("No-Collision (Build)", Color3.fromRGB(155, 89, 182), function()
    noclip = not noclip
    print("No-Clip: " .. tostring(noclip))
    RunService.Stepped:Connect(function()
        if noclip and Character then
            for _, v in pairs(Character:GetDescendants()) do
                if v:IsA("BasePart") then v.CanCollide = false end
            end
        end
    end)
end)

AddButton("Esconder Menu (Insert)", Color3.fromRGB(80, 80, 80), function()
    MainFrame.Visible = false
end)

-- Mostrar/Esconder com tecla
UserInputService.InputBegan:Connect(function(input, gpe)
    if not gpe and input.KeyCode == Enum.KeyCode.Insert then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

-- ANTI-AFK
local VirtualUser = game:GetService("VirtualUser")
Player.Idled:Connect(function()
    VirtualUser:CaptureController()
    VirtualUser:ClickButton2(Vector2.new())
end)

print("Script completo carregado. Pressione INSERT para abrir/fechar.")
