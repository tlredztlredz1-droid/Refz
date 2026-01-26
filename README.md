_G.FastAttack = true

if _G.FastAttack then
    local _ENV = (getgenv or getrenv or getfenv)()

    local function SafeWaitForChild(parent, childName)
        local success, result = pcall(function()
            return parent:WaitForChild(childName, 5) -- Timeout de 5s para não travar
        end)
        if not success or not result then
            warn("Não foi possível encontrar: " .. tostring(childName))
        end
        return result
    end

    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local RunService = game:GetService("RunService")
    local Players = game:GetService("Players")
    local Player = Players.LocalPlayer

    if not Player then return end

    -- Localizando Remotes e Módulos
    local Modules = SafeWaitForChild(ReplicatedStorage, "Modules")
    local Net = SafeWaitForChild(Modules, "Net")
    
    -- CORREÇÃO AQUI: Atribuindo os Remotes corretamente às variáveis
    local RegisterAttack = SafeWaitForChild(Net, "RE/RegisterAttack")
    local RegisterHit = SafeWaitForChild(Net, "RE/RegisterHit")

    local Settings = {
        AutoClick = true,
        ClickDelay = 0.1 -- Um delay de 0.1 é mais seguro contra kicks
    }

    local Module = {}

    Module.FastAttack = (function()
        if _ENV.rz_FastAttack then
            return _ENV.rz_FastAttack
        end

        local FastAttack = {
            Distance = 60, -- Distância recomendada para evitar detecção
            attackMobs = true,
            attackPlayers = true
        }

        local function IsAlive(character)
            return character and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0
        end

        local function ProcessEnemies(OthersEnemies, Folder)
            local BasePart = nil
            if not Folder then return nil end
            
            for _, Enemy in ipairs(Folder:GetChildren()) do
                local Head = Enemy:FindFirstChild("Head") or Enemy:FindFirstChild("HumanoidRootPart")
                if Head and IsAlive(Enemy) and Player:DistanceFromCharacter(Head.Position) < FastAttack.Distance then
                    if Enemy ~= Player.Character then
                        table.insert(OthersEnemies, Enemy) -- Enviando o objeto do inimigo
                        BasePart = Head
                    end
                end
            end
            return BasePart
        end

        function FastAttack:Attack(BasePart, OthersEnemies)
            if not BasePart or #OthersEnemies == 0 then return end
            -- Executa o ataque no servidor
            if RegisterAttack then RegisterAttack:FireServer(Settings.ClickDelay) end
            if RegisterHit then RegisterHit:FireServer(BasePart, OthersEnemies) end
        end

        function FastAttack:AttackNearest()
            local OthersEnemies = {}
            -- Verifica NPCs e outros Jogadores
            local Part1 = ProcessEnemies(OthersEnemies, workspace:FindFirstChild("Enemies"))
            local Part2 = ProcessEnemies(OthersEnemies, workspace:FindFirstChild("Characters"))

            local character = Player.Character
            if not character then return end
            
            local equippedWeapon = character:FindFirstChildOfClass("Tool")

            if equippedWeapon and equippedWeapon:FindFirstChild("LeftClickRemote") then
                -- Método alternativo para algumas armas específicas
                for _, enemy in ipairs(OthersEnemies) do
                    local hrp = enemy:FindFirstChild("HumanoidRootPart")
                    if hrp then
                        local direction = (hrp.Position - character:GetPivot().Position).Unit
                        pcall(function()
                            equippedWeapon.LeftClickRemote:FireServer(direction, 1)
                        end)
                    end
                end
            elseif #OthersEnemies > 0 then
                self:Attack(Part1 or Part2, OthersEnemies)
            end
        end

        function FastAttack:BladeHits()
            local character = Player.Character
            local Equipped = IsAlive(character) and character:FindFirstChildOfClass("Tool")
            -- Só ataca se estiver com arma na mão e não for arma de fogo
            if Equipped and Equipped:FindFirstChild("ToolTip") and Equipped.ToolTip ~= "Gun" then
                self:AttackNearest()
            elseif Equipped then
                self:AttackNearest() -- Tenta atacar mesmo sem ToolTip (alguns scripts não tem)
            end
        end

        -- Loop de execução
        task.spawn(function()
            while true do
                if Settings.AutoClick and _G.FastAttack then
                    FastAttack:BladeHits()
                end
                task.wait(Settings.ClickDelay)
            end
        end)

        _ENV.rz_FastAttack = FastAttack
        return FastAttack
    end)()
end
