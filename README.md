local Aimbot = loadstring(game:HttpGet("https://raw.githubusercontent.com/Exunys/Aimbot-V3/main/src/Aimbot.lua"))()
Aimbot.Load()
--[[ 
Script de ESP (Extrasensory Perception) 
Exibe caixas, distâncias e linhas de rastreamento para jogadores no seu campo de visão
]]

-- CONFIGS INICIAIS
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local ESPObjects = {}

-- FUNÇÕES PARA CRIAR E REMOVER ESP
function CreateESP(player)
    local esp = {}
    esp.Box = Drawing.new("Square")
    esp.Tracer = Drawing.new("Line")
    esp.Distance = Drawing.new("Text")
    ESPObjects[player] = esp
end

function RemoveESP(player)
    if ESPObjects[player] then
        for _, v in pairs(ESPObjects[player]) do
            v:Remove()
        end
        ESPObjects[player] = nil
    end
end

-- ATUALIZAÇÃO DO ESP
function UpdateESP()
    for i, v in pairs(Players:GetPlayers()) do
        if v ~= LocalPlayer and v.Team ~= LocalPlayer.Team and v.Character and v.Character:FindFirstChild("HumanoidRootPart") and v.Character:FindFirstChild("Head") then
            local pos, onScreen = Camera:WorldToViewportPoint(v.Character.HumanoidRootPart.Position)
            if onScreen then
                if not ESPObjects[v] then
                    CreateESP(v)
                end
                local esp = ESPObjects[v]
                local distance = (LocalPlayer.Character.HumanoidRootPart.Position - v.Character.HumanoidRootPart.Position).Magnitude
                
                -- Box (caixa de esp)
                esp.Box.Size = Vector2.new(60, 100) / (distance / 50)
                esp.Box.Position = Vector2.new(pos.X - esp.Box.Size.X / 2, pos.Y - esp.Box.Size.Y / 2)
                esp.Box.Color = Color3.fromRGB(255, 0, 0) -- Vermelho (você pode mudar para qualquer cor)
                esp.Box.Thickness = 1.5
                esp.Box.Visible = true

                -- Tracer (linha de rastreamento)
                esp.Tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                esp.Tracer.To = Vector2.new(pos.X, pos.Y)
                esp.Tracer.Color = esp.Box.Color
                esp.Tracer.Thickness = 1
                esp.Tracer.Visible = true

                -- Distância
                esp.Distance.Text = math.floor(distance) .. "m"
                esp.Distance.Position = Vector2.new(pos.X, pos.Y + 30)
                esp.Distance.Color = esp.Box.Color
                esp.Distance.Size = 16
                esp.Distance.Visible = true
            else
                RemoveESP(v)
            end
        else
            RemoveESP(v)
        end
    end
end

-- LOOP PRINCIPAL
game:GetService("RunService").RenderStepped:Connect(function()
    UpdateESP()
end)
