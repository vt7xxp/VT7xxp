local aimRadius = 200.0
local aimKey = 25 -- Botão direito do mouse (mirar)

Citizen.CreateThread(function()
    while true do
        Citizen.Wait(0)

        if IsControlPressed(0, aimKey) then
            local playerPed = PlayerPedId()
            local playerCoords = GetEntityCoords(playerPed)
            local closestPlayer, closestDistance = nil, aimRadius

            for _, playerId in ipairs(GetActivePlayers()) do
                local targetPed = GetPlayerPed(playerId)
                if targetPed ~= playerPed and not IsPedDeadOrDying(targetPed) then
                    local targetCoords = GetEntityCoords(targetPed)
                    local distance = #(playerCoords - targetCoords)

                    if distance < closestDistance and HasEntityClearLosToEntity(playerPed, targetPed, 17) then
                        closestDistance = distance
                        closestPlayer = targetPed
                    end
                end
            end

            if closestPlayer then
                local boneIndex = GetPedBoneIndex(closestPlayer, 31086) -- Cabeça
                local headCoords = GetWorldPositionOfEntityBone(closestPlayer, boneIndex)
                local _, screenX, screenY = World3dToScreen2d(headCoords.x, headCoords.y, headCoords.z)

                -- Aqui a mágica acontece: coloca a mira exatamente na posição
                SetPedShootsAtCoord(playerPed, headCoords.x, headCoords.y, headCoords.z, true)
                SetGameplayCamRelativeHeading(0.0)

                -- Opcional: vira o player também pra mirar reto
                local dir = GetEntityForwardVector(playerPed)
                local angle = GetHeadingFromVector_2d(headCoords.x - playerCoords.x, headCoords.y - playerCoords.y)
                SetEntityHeading(playerPed, angle)
            end
        end
    end
end)
function love.load()
    -- Configurações iniciais
    love.window.setTitle("Painel de Login")
    larguraJanela, alturaJanela = 800, 600
    love.window.setMode(larguraJanela, alturaJanela)
    
    -- Variáveis do login
    usuario = ""
    senha = ""
    foco = "usuario" -- Campo focado no momento
    
    carregando = false -- Indica se está carregando
end

function love.textinput(tecla)
    -- Digitação nos campos de usuário e senha
    if foco == "usuario" then
        usuario = usuario .. tecla
    elseif foco == "senha" then
        senha = senha .. tecla
    end
end

function love.keypressed(tecla)
    -- Apagar caracteres
    if tecla == "backspace" then
        if foco == "usuario" and #usuario > 0 then
            usuario = usuario:sub(1, -2)
        elseif foco == "senha" and #senha > 0 then
            senha = senha:sub(1, -2)
        end
    end

    -- Alternar foco com Tab
    if tecla == "tab" then
        if foco == "usuario" then
            foco = "senha"
        else
            foco = "usuario"
        end
    end

    -- Tela de carregamento ao pressionar Enter
    if tecla == "return" then
        carregando = true
    end
end

function love.draw()
    if carregando then
        -- Tela de carregamento
        love.graphics.printf("Carregando...", 0, alturaJanela / 2 - 10, larguraJanela, "center")
    else
        -- Título do painel
        love.graphics.printf("Painel de Login", 0, 50, larguraJanela, "center")
        
        -- Campo de Usuário
        love.graphics.printf("Usuário:", 200, 200, 400, "left")
        love.graphics.rectangle("line", 200, 230, 400, 30)
        love.graphics.printf(usuario, 210, 235, 380, "left")
        
        -- Campo de Senha
        love.graphics.printf("Senha:", 200, 300, 400, "left")
        love.graphics.rectangle("line", 200, 330, 400, 30)
        love.graphics.printf(string.rep("*", #senha), 210, 335, 380, "left")
        
        -- Indicação do campo focado
        if foco == "usuario" then
            love.graphics.rectangle("line", 195, 225, 410, 40)
        elseif foco == "senha" then
            love.graphics.rectangle("line", 195, 325, 410, 40)
        end
    end
end
