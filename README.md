-- Configuração de variáveis
local aimbot_ativo = true
local autoshot_ativo = true
local fov = 90 -- Campo de visão para mirar
local alvo = nil

-- Interface simples
function desenharInterface()
    draw.Text(10, 10, "Aimbot: " .. (aimbot_ativo and "ON" or "OFF"))
    draw.Text(10, 25, "AutoShot: " .. (autoshot_ativo and "ON" or "OFF"))
end

-- Verifica se o inimigo está dentro do FOV
function dentroDoFOV(jogador)
    local angulo_para_inimigo = CalcAngle(localPlayer:GetEyePosition(), jogador:GetEyePosition())
    local diferenca = AngleDiff(localPlayer:GetViewAngles(), angulo_para_inimigo)
    return math.abs(diferenca.yaw) <= fov / 2 and math.abs(diferenca.pitch) <= fov / 2
end

-- Seleciona o inimigo mais próximo do centro da mira
function encontrarAlvo()
    local melhorDistancia = math.huge
    local melhorAlvo = nil

    for i, jogador in pairs(entities.FindByClass("CCSPlayer")) do
        if jogador:IsEnemy() and jogador:IsAlive() and not jogador:IsDormant() then
            if dentroDoFOV(jogador) then
                local dist = (jogador:GetEyePosition() - localPlayer:GetEyePosition()):Length()
                if dist < melhorDistancia then
                    melhorDistancia = dist
                    melhorAlvo = jogador
                end
            end
        end
    end

    return melhorAlvo
end

-- Ajusta a mira para o inimigo
function mirarNoAlvo(inimigo)
    local angulo = CalcAngle(localPlayer:GetEyePosition(), inimigo:GetHitboxPosition(0)) -- Cabeça
    engine.SetViewAngles(angulo)
end

-- Dispara automaticamente
function autoshot()
    if input.IsButtonDown(MOUSE_LEFT) == false then
        input.SetMouseDown(MOUSE_LEFT)
    end
end

-- Callback principal
callbacks.Register("Draw", function()
    desenharInterface()
end)

callbacks.Register("CreateMove", function(cmd)
    if not aimbot_ativo then return end
    localPlayer = entities.GetLocalPlayer()
    if not localPlayer or not localPlayer:IsAlive() then return end

    alvo = encontrarAlvo()
    if alvo then
        mirarNoAlvo(alvo)
        if autoshot_ativo then
            autoshot()
        end
    end
end)
