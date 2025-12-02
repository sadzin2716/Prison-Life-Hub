--// CONFIGURAÇÕES
local highlightColor = Color3.fromRGB(255, 255, 255) -- Branco
local lineThickness = 2

--// SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

--// CRIA HIGHLIGHT
local function createHighlight(character)
    if character:FindFirstChild("EnemyHighlight") then return end

    local highlight = Instance.new("Highlight")
    highlight.Name = "EnemyHighlight"
    highlight.FillColor = highlightColor
    highlight.OutlineColor = highlightColor
    highlight.Parent = character
end

--// REMOVE HIGHLIGHT
local function removeHighlight(character)
    local hl = character:FindFirstChild("EnemyHighlight")
    if hl then hl:Destroy() end
end

--// VERIFICAR SE É INIMIGO
local function isEnemy(player)
    if #game.Teams:GetTeams() == 0 then
        return player ~= LocalPlayer
    else
        return player.Team ~= LocalPlayer.Team
    end
end

--// GET ROOT PART
local function getRoot(character)
    return character:FindFirstChild("HumanoidRootPart")
end

--// CRIA LINHA (TRACER)
local camera = workspace.CurrentCamera

local function createTracer(player)
    if not player.Character then return end
    if player.Character:FindFirstChild("Tracer") then return end

    local line = Drawing.new("Line")
    line.Thickness = lineThickness
    line.Color = Color3.new(1,1,1)
    line.Visible = true

    local tracerObj = Instance.new("BoolValue")
    tracerObj.Name = "Tracer"
    tracerObj.Parent = player.Character

    RunService.RenderStepped:Connect(function()
        if not player.Character or not getRoot(player.Character) then
            line.Visible = false
            return
        end

        local root = getRoot(player.Character)
        local screenPos, onScreen = camera:WorldToViewportPoint(root.Position)

        if onScreen then
            line.From = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y)
            line.To = Vector2.new(screenPos.X, screenPos.Y)
            line.Visible = true
        else
            line.Visible = false
        end
    end)
end

--// MONITORA JOGADOR
local function monitorPlayer(player)
    player.CharacterAdded:Connect(function(character)
        task.wait(1)

        if isEnemy(player) then
            createHighlight(character)
            createTracer(player)
        else
            removeHighlight(character)
        end
    end)

    if player.Character then
        if isEnemy(player) then
            createHighlight(player.Character)
            createTracer(player)
        end
    end
end

--// MONITORAR TODOS
for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        monitorPlayer(player)
    end
end

Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        monitorPlayer(player)
    end
end)

Players.PlayerRemoving:Connect(function(player)
    if player.Character then
        removeHighlight(player.Character)
    end
end)
