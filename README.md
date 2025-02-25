local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = game.Workspace.CurrentCamera
local AimbotEnabled = false
local UIS = game:GetService("UserInputService")
local Mouse = LocalPlayer:GetMouse()

-- Criar botão de ativação/desativação
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local ToggleButton = Instance.new("TextButton", ScreenGui)

ToggleButton.Size = UDim2.new(0, 100, 0, 50)
ToggleButton.Position = UDim2.new(0.1, 0, 0.1, 0)
ToggleButton.Text = "Aimbot OFF"
ToggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
ToggleButton.TextColor3 = Color3.new(1, 1, 1)
ToggleButton.BackgroundTransparency = 0.2  

ToggleButton.MouseButton1Click:Connect(function()
    AimbotEnabled = not AimbotEnabled
    if AimbotEnabled then
        ToggleButton.Text = "Aimbot ON"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    else
        ToggleButton.Text = "Aimbot OFF"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    end
end)

-- Função para encontrar o inimigo mais próximo dentro da distância certa
local function GetClosestEnemy()
    local ClosestEnemy = nil
    local ShortestDistance = math.huge

    for _, Player in pairs(Players:GetPlayers()) do
        if Player ~= LocalPlayer and Player.Character and Player.Character:FindFirstChild("Head") then
            -- Verificar se o inimigo é do time vermelho
            if Player.TeamColor == BrickColor.new("Bright red") then
                local Head = Player.Character.Head
                local DistanceFromPlayer = (Head.Position - LocalPlayer.Character.Head.Position).Magnitude
                
                -- Definir a meia distância (máximo de 150 studs)
                if DistanceFromPlayer < 150 then
                    local EnemyPos, OnScreen = Camera:WorldToViewportPoint(Head.Position)
                    if OnScreen then
                        local DistanceToCenter = (Vector2.new(EnemyPos.X, EnemyPos.Y) - UserInputService:GetMouseLocation()).Magnitude
                        if DistanceToCenter < ShortestDistance and DistanceToCenter < 100 then
                            ShortestDistance = DistanceToCenter
                            ClosestEnemy = Head
                        end
                    end
                end
            end
        end
    end
    return ClosestEnemy
end

-- Função de mira suavizada
local function SmoothAim(Target, Speed)
    if Target then
        local TargetPos = Target.Position
        local CurrentCFrame = Camera.CFrame
        local NewDirection = (TargetPos - CurrentCFrame.Position).unit

        -- Suaviza a transição para a mira
        local SmoothedCFrame = CFrame.new(CurrentCFrame.Position, CurrentCFrame.Position + (CurrentCFrame.LookVector:Lerp(NewDirection, Speed)))
        Camera.CFrame = SmoothedCFrame
    end
end

-- Ativar mira ao atirar com 80% de força
Mouse.Button1Down:Connect(function()
    if AimbotEnabled then
        local Target = GetClosestEnemy()
        if Target then
            SmoothAim(Target, 0.8) -- Ajuste de 80% da mira para o inimigo
        end
    end
end)

-- Função principal do Aimbot (mira suave sem travar)
local function Aimbot()
    while true do
        if AimbotEnabled then
            local Target = GetClosestEnemy()
            if Target then
                SmoothAim(Target, 0.2) -- 20% de ajuste enquanto mira normalmente
            end
        end
        task.wait(0.01)  
    end
end

-- Iniciar Aimbot
task.spawn(Aimbot)
