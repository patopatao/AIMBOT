-- Configurações
local aimbot_ativo = true
local autoshot_ativo = true
local fov = 90

-- Desenhar Interface na tela
callbacks.Register("Draw", function()
    draw.Color(0, 255, 0, 255)
    draw.Text(10, 10, "Aimbot: " .. (aimbot_ativo and "Ativo" or "Desativado"))
    draw.Text(10, 25, "AutoShot: " .. (autoshot_ativo and "Ativo" or "Desativado"))
end)

-- Função para calcular ângulo entre dois vetores
function CalcAngle(from, to)
    local delta = to - from
    local pitch = math.atan2(-delta.z, math.sqrt(delta.x^2 + delta.y^2)) * 180 / math.pi
    local yaw = math.atan2(delta.y, delta.x) * 180 / math.pi
    return QAngle(pitch, yaw, 0)
end

-- Diferença entre ângulos
function AngleDiff(a, b)
    local diff = QAngle(a.pitch - b.pitch, a.yaw - b.yaw, 0)
    if diff.yaw > 180 then diff.yaw = diff.yaw - 360 end
    if diff.yaw < -180 then diff.yaw = diff.yaw + 360 end
    return diff
end

-- Verifica se está dentro do FOV
function dentroDoFOV(localPlayer, inimigo)
    local viewAngles = engine.GetViewAngles()
    local angulo = CalcAngle(localPlayer:GetEyePosition(), inimigo:GetHitboxPosition(0))
    local diff = AngleDiff(viewAngles, angulo)
    return math.abs(diff.yaw) <= fov / 2 and math.abs(diff.pitch) <= fov / 2
end

-- Seleciona o inimigo mais próximo
function encontrarAlvo()
    local localPlayer = entities.GetLocalPlayer()
    local melhorAlvo = nil
    local menorDist = math.huge

    for _, inimigo in pairs(entities.FindByClass("CCSPlayer")) do
        if inimigo:IsEnemy() and inimigo:IsAlive() and not inimigo:IsDormant() then
            if dentroDoFOV(localPlayer, inimigo) then
                local dist = (localPlayer:GetEyePosition() - inimigo:GetEyePosition()):Length()
                if dist < menorDist then
                    menorDist = dist
                    melhorAlvo = inimigo
                end
            end
        end
    end

    return melhorAlvo
end

-- Aimbot + AutoShot
callbacks.Register("CreateMove", function(cmd)
    if not aimbot_ativo then return end

    local localPlayer = entities.GetLocalPlayer()
    if not localPlayer or not localPlayer:IsAlive() then return end

    local alvo = encontrarAlvo()
    if alvo then
        local angulo = CalcAngle(localPlayer:GetEyePosition(), alvo:GetHitboxPosition(0))
        cmd:SetViewAngles(angulo)

        if autoshot_ativo then
            cmd.buttons = bit.bor(cmd.buttons, 1) -- Atira (IN_ATTACK = 1)
        end
    end
end)
