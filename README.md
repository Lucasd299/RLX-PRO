-- RLX PRO - SCRIPT COMPLETO Blox Fruits
-- Suporte: 3 mundos, AutoFarm, Skills, ESP, Eventos, Teleporte

-- Serviços principais
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer

-- Evento principal de comunicação do jogo
local CommF_ = ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("CommF_")
local AttackEvent = ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("Attack")
local SkillEvent = ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("SkillEvent")

-- Configurações iniciais (pode expandir com menu depois)
local settings = {
    AutoFarm = true,
    AutoSkills = true,
    SkillKeys = {"Z", "X", "C", "V", "F"},
    AutoTPToNextIsland = false,
    AutoUseHaki = true,
}

-- 💥 Auto Uso de Skills (Z, X, C, etc)
spawn(function()
    while settings.AutoSkills do
        for _, skill in pairs(settings.SkillKeys) do
            pcall(function()
                SkillEvent:FireServer(skill)
            end)
            wait(1.5)
        end
        wait(1)
    end
end)

-- 🥋 Auto Haki (Buso Haki)
spawn(function()
    while settings.AutoUseHaki do
        pcall(function()
            LocalPlayer.Character:FindFirstChild("Humanoid"):UnequipTools()
            ReplicatedStorage.Remotes.CommF_:InvokeServer("Buso")
        end)
        wait(10)
    end
end)

-- 🗺️ Teleport para Ilhas ou Posição
function teleportTo(pos)
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(pos)
    end
end

-- 👁️ ESP Básico para Players
function createESP(target, color)
    local box = Instance.new("BoxHandleAdornment")
    box.Size = Vector3.new(4,6,4)
    box.Color3 = color
    box.AlwaysOnTop = true
    box.Adornee = target
    box.ZIndex = 5
    box.Transparency = 0.5
    box.Parent = target
end

-- ⚔️ Auto Farm de NPC
function getClosestEnemy()
    local enemies = Workspace.Enemies:GetChildren()
    local closest = nil
    local shortest = math.huge

    for _, enemy in pairs(enemies) do
        if enemy:FindFirstChild("HumanoidRootPart") and enemy:FindFirstChild("Humanoid") then
            local distance = (LocalPlayer.Character.HumanoidRootPart.Position - enemy.HumanoidRootPart.Position).Magnitude
            if distance < shortest and enemy.Humanoid.Health > 0 then
                closest = enemy
                shortest = distance
            end
        end
    end
    return closest
end

spawn(function()
    while settings.AutoFarm do
        local target = getClosestEnemy()
        if target then
            teleportTo(target.HumanoidRootPart.Position + Vector3.new(0,5,0))
            AttackEvent:FireServer(target.HumanoidRootPart.Position)
        end
        wait(0.8)
    end
end)

-- 🌍 Troca de Mundo (exemplo: teleportar ou solicitar porta)
function goToWorld(worldName)
    local worldIDs = {
        ["First Sea"] = 1,
        ["Second Sea"] = 2,
        ["Third Sea"] = 3,
    }
    if worldIDs[worldName] then
        CommF_:InvokeServer("TravelMain", worldIDs[worldName])
    else
        warn("Nome de mundo inválido!")
    end
end

-- Exemplo de chamada:
-- goToWorld("Second Sea")-- Função para pegar o nome da missão certa com base no nível
function getQuestInfo()
    local level = LocalPlayer.Data.Level.Value
    if level <= 9 then
        return "BanditQuest1", "Bandit", CFrame.new(1060, 16, 1547)
    elseif level <= 14 then
        return "BanditQuest2", "Monkey", CFrame.new(-1603, 8, 145)
    elseif level <= 29 then
        return "JungleQuest", "Gorilla", CFrame.new(-1599, 36, -31)
    elseif level <= 39 then
        return "BuggyQuest1", "Pirate", CFrame.new(-1144, 5, 3825)
    elseif level <= 59 then
        return "BuggyQuest2", "Brute", CFrame.new(-1191, 6, 4295)
    -- (continua com todos os NPCs até o terceiro mundo...)
    else
        return nil, nil, nil -- fora do suporte inicial
    end
end

-- 🧭 Auto Missão
function autoGetQuest()
    local questName, enemyName, questCFrame = getQuestInfo()
    if questName then
        teleportTo(questCFrame.Position + Vector3.new(0, 5, 0))
        wait(1)
        CommF_:InvokeServer("StartQuest", questName, 1)
    end
end

-- 🗡️ Auto Equipar Espada ou Fruta
function autoEquip(weaponName)
    local backpack = LocalPlayer.Backpack
    if backpack:FindFirstChild(weaponName) then
        LocalPlayer.Character.Humanoid:EquipTool(backpack[weaponName])
    end
end

-- ⚔️ Auto Farm com Missão Inteligente
spawn(function()
    while settings.AutoFarm do
        local questName, enemyName = getQuestInfo()
        if not LocalPlayer.PlayerGui.Main.Quest.Visible then
            autoGetQuest()
        else
            local target = getClosestEnemy()
            if target and target.Name:find(enemyName) then
                teleportTo(target.HumanoidRootPart.Position + Vector3.new(0, 5, 0))
                autoEquip("Dark Blade") -- Altere para sua arma/fruta
                AttackEvent:FireServer(target.HumanoidRootPart.Position)
            end
        end
        wait(0.6)
    end
end)
-- ⚙️ ESP Configuration
local ESPSettings = {
    FruitESP = true,
    RealFruitESP = true,
    ChestESP = true,
    FlowerESP = true,
    PlayerESP = true,
    BossESP = true,
}

-- 🎨 Função de criação do ESP visual
function addESP(obj, color)
    local box = Instance.new("BoxHandleAdornment", obj)
    box.Size = obj.Size + Vector3.new(0.5, 0.5, 0.5)
    box.Adornee = obj
    box.AlwaysOnTop = true
    box.ZIndex = 10
    box.Color3 = color
    box.Transparency = 0.4
    box.Name = "RLX_ESP"
end

-- 🛰️ Função que verifica e adiciona ESPs
spawn(function()
    while true do
        if ESPSettings.FruitESP then
            for _, v in pairs(game.Workspace:GetChildren()) do
                if v:IsA("Tool") and v:FindFirstChild("Handle") and not v.Handle:FindFirstChild("RLX_ESP") then
                    addESP(v.Handle, Color3.fromRGB(255, 85, 0))
                end
            end
        end
        if ESPSettings.RealFruitESP then
            for _, v in pairs(game.ReplicatedStorage:GetChildren()) do
                if v:IsA("Tool") and v:FindFirstChild("Handle") and not v.Handle:FindFirstChild("RLX_ESP") then
                    addESP(v.Handle, Color3.fromRGB(255, 0, 255))
                end
            end
        end
        if ESPSettings.ChestESP and Workspace:FindFirstChild("Chest1") then
            for _, v in pairs(Workspace:GetChildren()) do
                if string.find(v.Name, "Chest") and v:IsA("Model") and v:FindFirstChild("Chest") then
                    if not v.Chest:FindFirstChild("RLX_ESP") then
                        addESP(v.Chest, Color3.fromRGB(255, 255, 0))
                    end
                end
            end
        end
        if ESPSettings.FlowerESP and Workspace:FindFirstChild("Flower2") then
            for _, v in pairs(Workspace:GetChildren()) do
                if v.Name:find("Flower") and v:IsA("Model") and v:FindFirstChild("TouchInterest") then
                    if not v:FindFirstChild("RLX_ESP") then
                        addESP(v, Color3.fromRGB(0, 255, 127))
                    end
                end
            end
        end
        if ESPSettings.PlayerESP then
            for _, plr in pairs(Players:GetPlayers()) do
                if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and not plr.Character.HumanoidRootPart:FindFirstChild("RLX_ESP") then
                    addESP(plr.Character.HumanoidRootPart, Color3.fromRGB(0, 180, 255))
                end
            end
        end
        wait(2)
    end
end)

-- 🍍 Auto pegar frutas caídas
spawn(function()
    while true do
        for _, v in pairs(game.Workspace:GetChildren()) do
            if v:IsA("Tool") and v:FindFirstChild("Handle") and (v.Name:find("Fruit") or v.Name:find("Bomb")) then
                pcall(function()
                    LocalPlayer.Character.HumanoidRootPart.CFrame = v.Handle.CFrame
                end)
                wait(1)
            end
        end
        wait(10)
    end
end)
-- GUI Simples para Teleporte
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local OpenButton = Instance.new("TextButton", ScreenGui)
local MainFrame = Instance.new("Frame", ScreenGui)

-- Botão para abrir
OpenButton.Size = UDim2.new(0, 100, 0, 30)
OpenButton.Position = UDim2.new(0, 10, 0, 100)
OpenButton.Text = "Teleport"
OpenButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
OpenButton.TextColor3 = Color3.new(1,1,1)
OpenButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

-- Janela principal
MainFrame.Size = UDim2.new(0, 250, 0, 350)
MainFrame.Position = UDim2.new(0, 120, 0, 100)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.Visible = false

-- Lista de teleportes
local islands = {
    ["First Sea"] = {
        ["Starter Island"] = CFrame.new(1098, 16, 1433),
        ["Jungle"] = CFrame.new(-1614, 36, 146),
        ["Pirate Village"] = CFrame.new(-1144, 5, 3825),
    },
    ["Second Sea"] = {
        ["Kingdom of Rose"] = CFrame.new(-390, 358, 1400),
        ["Green Zone"] = CFrame.new(-2365, 72, -3130),
        ["Snow Mountain"] = CFrame.new(1400, 449, -6200),
    },
    ["Third Sea"] = {
        ["Port Town"] = CFrame.new(-293, 19, 5465),
        ["Hydra Island"] = CFrame.new(5225, 25, -110),
        ["Great Tree"] = CFrame.new(2260, 25, -6700),
        ["Castle on the Sea"] = CFrame.new(-5500, 300, -2900),
    }
}

-- Criar botões para cada ilha
local yPos = 10
for sea, list in pairs(islands) do
    local title = Instance.new("TextLabel", MainFrame)
    title.Size = UDim2.new(1, 0, 0, 20)
    title.Position = UDim2.new(0, 0, 0, yPos)
    title.Text = "🌊 "..sea
    title.TextColor3 = Color3.new(1,1,0.6)
    title.BackgroundTransparency = 1
    title.TextScaled = true
    yPos += 25

    for name, cframe in pairs(list) do
        local btn = Instance.new("TextButton", MainFrame)
        btn.Size = UDim2.new(1, -20, 0, 25)
        btn.Position = UDim2.new(0, 10, 0, yPos)
        btn.Text = name
        btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        btn.TextColor3 = Color3.new(1,1,1)
        btn.MouseButton1Click:Connect(function()
            LocalPlayer.Character.HumanoidRootPart.CFrame = cframe + Vector3.new(0, 5, 0)
        end)
        yPos += 30
    end
end
-- Mirage Island Detector
spawn(function()
    while wait(10) do
        for _, v in pairs(game.Workspace:GetChildren()) do
            if v.Name == "Mirage Island" then
                print("[RLX HUB] Mirage Island detectada!")
                game.StarterGui:SetCore("SendNotification", {
                    Title = "🚨 Mirage Detectada!",
                    Text = "A Mirage Island apareceu!",
                    Duration = 10
                })
            end
        end
    end
end)

-- Detector do Boss Longma (Evento Tushita)
spawn(function()
    while wait(15) do
        if game.Workspace.Enemies:FindFirstChild("Longma") then
            print("[RLX HUB] Boss Longma detectado!")
            game.StarterGui:SetCore("SendNotification", {
                Title = "⚔️ Longma!",
                Text = "O Boss da Tushita apareceu!",
                Duration = 8
            })
        end
    end
end)

-- Detector de Fire Essence (para Raça Ghoul)
spawn(function()
    while wait(8) do
        local essence = game.Workspace:FindFirstChild("Fire Essence")
        if essence then
            game.StarterGui:SetCore("SendNotification", {
                Title = "🔥 Fire Essence!",
                Text = "Drop raro de Ghoul V3 detectado.",
                Duration = 6
            })
        end
    end
end)

-- Detector de Holy Torch (para Angel V3)
spawn(function()
    while wait(8) do
        for _, v in pairs(game.Workspace:GetChildren()) do
            if v.Name:find("Holy Torch") then
                game.StarterGui:SetCore("SendNotification", {
                    Title = "🕯️ Holy Torch!",
                    Text = "Drop raro para Angel detectado.",
                    Duration = 6
                })
            end
        end
    end
end)
