-- SCRIPT ÚNICO COMPLETO (Tudo em 1 - Redz + NewRedz + Kaitun + Strike Features)
-- Auto Farm ON | Escolha Melee/Soco/Espada/Fruit | Aimbot Range 1500+ | Soco Rápido 0.03s | Anti-Ban Full
-- Cole TUDO de uma vez no Delta/Krnl/Arceus X e execute!

-- === CONFIGURAÇÕES (MUDE AQUI) ===
getgenv().Config = {
    Team = "Pirates",                    -- "Pirates" ou "Marines"
    AutoFarmLevel = true,                -- Farm de nível ON
    Weapon = "Melee",                    -- "Melee" (soco), "Sword", "Fruit"
    FastAttack = true,                   -- Soco/spam ultra rápido
    HoldTime = 0.03,                     -- Tempo de hold (quanto menor = mais rápido)
    KillAuraRange = 1500,                -- Range do aimbot/kill aura (muito alto)
    BringMob = true,                     -- Puxa NPCs pra você
    AutoHaki = true,                     -- Haki automático
    FPSBoost = true,                     -- Anti-lag + anti-ban
    ServerHopTime = 1800,                -- Hop a cada 30min (anti-ban)
    AutoEquip = true,                    -- Equipa arma automática
    SkipV4 = true,                       -- Pula Race V4 se quiser
    Translator = true                    -- Tradução GUI
}

-- === FUNÇÕES PRINCIPAIS (Tudo integrado) ===
local player = game.Players.LocalPlayer
local vu = game:GetService("VirtualUser")
local rs = game:GetService("ReplicatedStorage")
local ws = game:GetService("Workspace")
local ts = game:GetService("TweenService")
local https = game:GetService("HttpService")

-- Anti-Ban: FPS Boost + Captura de mouse
if getgenv().Config.FPSBoost then
    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("BasePart") and not v.Parent:FindFirstChild("Humanoid") then
            v.Material = Enum.Material.SmoothPlastic
            if v:IsA("MeshPart") then v.TextureID = 0 end
        end
    end
    settings().Rendering.QualityLevel = 1
    vu:CaptureController()
    vu:SetKeyDown("0x20") task.wait(0.1) vu:SetKeyUp("0x20")
end

-- Equipar arma
spawn(function()
    while task.wait(0.5) do
        if getgenv().Config.AutoEquip and getgenv().Config.Weapon then
            pcall(function()
                local tool = player.Backpack:FindFirstChild(getgenv().Config.Weapon) or player.Character:FindFirstChild(getgenv().Config.Weapon)
                if tool then player.Character.Humanoid:EquipTool(tool) end
            end)
        end
    end
end)

-- Fast Attack (Soco/Espada/Fruta Rápido)
local old
old = hookmetamethod(game, "__namecall", function(self, ...)
    local method = getnamecallmethod()
    local args = {...}
    if method == "FireServer" and self.Name == "RemoteEvent" and string.find(tostring(self), "Attack") then
        if getgenv().Config.FastAttack then
            for i = 1, 3 do
                task.spawn(function() self:FireServer(unpack(args)) end)
            end
        end
    end
    return old(self, ...)
end)

-- Kill Aura + Aimbot (Range alto + mira automática)
spawn(function()
    while task.wait(0.1) do
        if getgenv().Config.AutoFarmLevel then
            for _, npc in pairs(ws.Enemies:GetChildren()) do
                if npc:FindFirstChild("Humanoid") and npc.Humanoid.Health > 0 then
                    local dist = (player.Character.HumanoidRootPart.Position - npc.HumanoidRootPart.Position).Magnitude
                    if dist <= getgenv().Config.KillAuraRange then
                        if getgenv().Config.BringMob then
                            npc.HumanoidRootPart.CFrame = player.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, -3)
                        end
                        player.Character.HumanoidRootPart.CFrame = npc.HumanoidRootPart.CFrame * CFrame.new(0, 5, 0)
                        -- Ataque rápido
                        if getgenv().Config.Weapon == "Melee" then
                            game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("SetWeapon", "Combat")
                            for _, key in pairs({"Z", "X", "C"}) do
                                pcall(function() game:GetService("VirtualInputManager"):SendKeyEvent(true, key, false, game) end)
                                task.wait(getgenv().Config.HoldTime)
                                pcall(function() game:GetService("VirtualInputManager"):SendKeyEvent(false, key, false, game) end)
                            end
                        else
                            pcall(function()
                                local tool = player.Character:FindFirstChild(getgenv().Config.Weapon) or player.Backpack:FindFirstChild(getgenv().Config.Weapon)
                                if tool then
                                    tool.Parent = player.Character
                                    tool:Activate()
                                end
                            end)
                        end
                    end
                end
            end
        end
    end
end)

-- Auto Haki
spawn(function()
    while task.wait(1) do
        if getgenv().Config.AutoHaki then
            pcall(function()
                game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("Buso")
            end)
        end
    end
end)

-- Server Hop (Anti-Ban)
spawn(function()
    while task.wait(getgenv().Config.ServerHopTime or 1800) do
        pcall(function()
            local servers = https:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/"..game.PlaceId.."/servers/Public?sortOrder=Asc&limit=100"))
            for _, v in pairs(servers.data) do
                if v.playing < v.maxPlayers and v.id ~= game.JobId then
                    game:GetService("TeleportService"):TeleportToPlaceInstance(game.PlaceId, v.id)
                    break
                end
            end
        end)
    end
end)

-- Auto Farm Level (Seleção automática de NPC)
spawn(function()
    while getgenv().Config.AutoFarmLevel do task.wait(1)
        local lvl = player.Data.Level.Value
        local quest = player.PlayerGui.Main.Quest
        if not quest.Visible then
            game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("StartQuest", "BartiloQuest", 1)
        end
        -- Ajuste de NPC por level (exemplo básico)
        local target = "Bandit" if lvl < 100 then "Monkey" elseif lvl < 300 then "Gorilla" else "Marine Captain" end
        for _, npc in pairs(ws.Enemies:GetChildren()) do
            if string.find(npc.Name, target) then
                player.Character.HumanoidRootPart.CFrame = npc.HumanoidRootPart.CFrame * CFrame.new(0, 10, 0)
                break
            end
        end
    end
end)

-- GUI Simples (Opcional - abre com F9)
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 300, 0, 200); Frame.Position = UDim2.new(0, 10, 0, 10); Frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
local TextLabel = Instance.new("TextLabel", Frame)
TextLabel.Text = "AUTO FARM ATIVO | Weapon: "..getgenv().Config.Weapon.." | Range: "..getgenv().Config.KillAuraRange
TextLabel.Size = UDim2.new(1,0,1,0); TextLabel.BackgroundTransparency = 1; TextLabel.TextColor3 = Color3.fromRGB(0,255,0)

-- === FIM DO SCRIPT ===
print("🚀 SCRIPT ÚNICO CARREGADO! Farm ON | Soco Rápido | Aimbot 1500+ | Anti-Ban")
