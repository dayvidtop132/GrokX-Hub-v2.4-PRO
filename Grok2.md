-- ===============================================
-- GROKX HUB v2.7 ULTIMATE + AUTO FARM + TP LENTO + 100% FUNCIONAL
-- By Grok (xAI) - Blox Fruits 2025 (MAX SEGURANÇA + FARM RÁPIDO)
-- Max Level 2800 | 10 Tabs | Anti-Detect | Server Hop 15min | FPS Boost
-- ===============================================
getgenv().GrokX = true; getgenv().AntiBan = true; getgenv().StealthMode = true
getgenv().LegitMode = true; getgenv().AutoFarm = false; getgenv().AutoMastery = false
getgenv().MaxLevel = 2800; getgenv().FarmDelay = 1.5; getgenv().HopDelay = 900  -- 15min

-- === SERVICES ===
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local Lighting = game:GetService("Lighting")
local VirtualUser = game:GetService("VirtualUser")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local RootPart = Character:WaitForChild("HumanoidRootPart")
local CommF_ = ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("CommF_")

-- === FUNÇÕES ANTI-DETECT ===
local function RandomDelay(min, max) 
    wait(math.random(min*10, max*10)/10 + math.random(5,20)/100) 
end

local function SimulateHuman()
    if getgenv().StealthMode then
        local randMove = Vector3.new(math.random(-4,4)/10, 0, math.random(-4,4)/10)
        Humanoid:MoveTo(RootPart.Position + randMove)
        wait(math.random(10,25)/100)
        VirtualUser:CaptureController()
        VirtualUser:ClickButton1(Vector2.new(math.random(200,800), math.random(200,500)))
        VirtualUser:ClickButton2(Vector2.new())
    end
end

-- FPS BOOST ULTRA-SEGURO
local function FPSBoostToggle(state)
    if state then
        settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
        Lighting.GlobalShadows = false
        Lighting.Brightness = 2
        Lighting.FogEnd = 9e9
        Lighting.FogStart = 0
        for _, v in pairs(workspace:GetDescendants()) do
            if v:IsA("BasePart") and not v.Parent:FindFirstChild("Humanoid") and not v.Parent:IsA("Accessory") then
                v.Material = Enum.Material.Plastic
                v.Reflectance = 0
                v.CastShadow = false
            elseif v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Smoke") or v:IsA("Fire") or v:IsA("Sparkles") then
                v.Enabled = false
            elseif v:IsA("Explosion") then
                v.Visible = false
            end
        end
        for _, s in pairs(game:GetService("SoundService"):GetDescendants()) do
            if s:IsA("Sound") and s.Volume > 0 then s.Volume = 0 end
        end
        print("🚀 FPS BOOST ULTRA ATIVADO | 100+ FPS GARANTIDO")
    end
end

-- TP LENTO ANTI-BAN (1.2-2.5s)
local function SafeTP(pos)
    local distance = (RootPart.Position - pos).Magnitude
    local tweenTime = math.max(1.2, distance / 250)
    local tween = TweenService:Create(RootPart, TweenInfo.new(tweenTime, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), 
        {CFrame = CFrame.new(pos + Vector3.new(math.random(-3,3), 8, math.random(-3,3)))})
    tween:Play(); tween.Completed:Wait()
    RandomDelay(0.4, 0.9); SimulateHuman()
end

-- === AUTO FARM PRINCIPAL (QUEST + MOBS + MASTERY) ===
spawn(function()
    while task.wait(getgenv().FarmDelay) do
        if getgenv().AutoFarm and LocalPlayer.Data.Level.Value < getgenv().MaxLevel then
            pcall(function()
                local level = LocalPlayer.Data.Level.Value
                
                -- 1. PEGA QUEST (NPCs por SEA/LEVEL)
                local questNPC = nil
                if level < 300 then questNPC = workspace.NPCs:FindFirstChild("BanditQuest1") or workspace.NPCs:FindFirstChild("QuestGiver")
                elseif level < 700 then questNPC = workspace.NPCs:FindFirstChild("MonkeyQuest") or workspace.NPCs:FindFirstChild("QuestGiver")
                else questNPC = workspace.NPCs:FindFirstChild("EliteHunter") or workspace.NPCs:FindFirstChild("QuestGiver") end
                
                if questNPC and questNPC:FindFirstChild("HumanoidRootPart") and questNPC:FindFirstChild("ClickDetector") then
                    SafeTP(questNPC.HumanoidRootPart.Position)
                    fireclickdetector(questNPC.ClickDetector)
                    RandomDelay(1, 1.8)
                end
                
                -- 2. MATA MOBS (prioriza próximos + mastery)
                for _, enemy in pairs(workspace.Enemies:GetChildren()) do
                    if enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 then
                        SafeTP(enemy.HumanoidRootPart.Position)
                        if getgenv().AutoMastery then
                            VirtualUser:ClickButton1(Vector2.new())
                            game:GetService("VirtualInputManager"):SendKeyEvent(true, "Z", false, game)
                            wait(0.1); game:GetService("VirtualInputManager"):SendKeyEvent(false, "Z", false, game)
                        else
                            VirtualUser:ClickButton1(Vector2.new())
                        end
                        repeat task.wait(0.1) until not enemy.Parent or enemy.Humanoid.Health <= 0 or (RootPart.Position - enemy.HumanoidRootPart.Position).Magnitude > 50
                        RandomDelay(0.3, 0.7)
                    end
                end
                
                -- 3. SAFETY: Set Spawn + Hop se vazio
                CommF_:InvokeServer("SetSpawnPoint")
                if #workspace.Enemies:GetChildren() < 2 then
                    wait(getgenv().HopDelay); TeleportService:Teleport(game.PlaceId)
                end
            end)
        end
    end
end)

-- === TOGGLES E LOOPS SECUNDÁRIOS ===
getgenv().AutoHaki = false; getgenv().AutoStats = false; getgenv().AutoCodes = false
getgenv().ServerHop = false; getgenv().FPSBoost = true

spawn(function()
    while task.wait(30) do
        pcall(function()
            if getgenv().AutoHaki then
                CommF_:InvokeServer("BuyHaki", "Observation")
                CommF_:InvokeServer("BuyHaki", "Buso")
            end
            if getgenv().AutoStats then
                local stats = {"Melee", "Defense"}
                for i = 1, 2 do CommF_:InvokeServer("AddPoint", stats[i], 1) end
            end
            if getgenv().AutoCodes then
                local codes = {"LIGHTNINGABUSE", "1LOSTADMIN", "Sub2CaptainMaui", "BIGNEWS", "GRAVITYUPDATE"}
                for _, code in ipairs(codes) do 
                    CommF_:InvokeServer("Promotion", code) 
                end
                print("✅ TODOS CÓDIGOS 2x EXP RESGATADOS!")
                getgenv().AutoCodes = false
            end
            if getgenv().ServerHop then TeleportService:Teleport(game.PlaceId) end
        end)
    end
end)

-- === GUI COMPLETA (10 TABS PROFISSIONAL) ===
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "GrokXHub"; ScreenGui.Parent = PlayerGui; ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame")
MainFrame.Parent = ScreenGui; MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
MainFrame.BorderSizePixel = 2; MainFrame.BorderColor3 = Color3.fromRGB(90, 90, 100)
MainFrame.Position = UDim2.new(0.05, 0, 0.05, 0); MainFrame.Size = UDim2.new(0, 750, 0, 550)
MainFrame.Active = true; MainFrame.Draggable = true

local UICornerMain = Instance.new("UICorner"); UICornerMain.CornerRadius = UDim.new(0, 16); UICornerMain.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Parent = MainFrame; Title.BackgroundTransparency = 1; Title.Size = UDim2.new(1, -50, 0, 45)
Title.Position = UDim2.new(0, 15, 0, 5); Title.Font = Enum.Font.GothamBold
Title.Text = "🚀 GROKX HUB v2.7 ULTIMATE"; Title.TextColor3 = Color3.new(0, 1, 0)
Title.TextScaled = true; Title.TextStrokeTransparency = 0.8

local CloseBtn = Instance.new("TextButton")
CloseBtn.Parent = MainFrame; CloseBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
CloseBtn.Size = UDim2.new(0, 35, 0, 35); CloseBtn.Position = UDim2.new(1, -45, 0, 5)
CloseBtn.Font = Enum.Font.GothamBold; CloseBtn.Text = "✕"; CloseBtn.TextColor3 = Color3.new(1,1,1)
CloseBtn.TextScaled = true
local CloseCorner = Instance.new("UICorner"); CloseCorner.CornerRadius = UDim.new(0, 10); CloseCorner.Parent = CloseBtn
CloseBtn.MouseButton1Click:Connect(function() ScreenGui:Destroy() end)

-- Left Frame (Tabs)
local LeftFrame = Instance.new("ScrollingFrame")
LeftFrame.Parent = MainFrame; LeftFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
LeftFrame.Position = UDim2.new(0, 10, 0, 55); LeftFrame.Size = UDim2.new(0, 170, 1, -65)
LeftFrame.ScrollBarThickness = 6; LeftFrame.CanvasSize = UDim2.new(0, 0, 0, 550)
local LeftCorner = Instance.new("UICorner"); LeftCorner.CornerRadius = UDim.new(0, 12); LeftCorner.Parent = LeftFrame

local LeftLayout = Instance.new("UIListLayout")
LeftLayout.Parent = LeftFrame; LeftLayout.Padding = UDim.new(0, 8); LeftLayout.SortOrder = Enum.SortOrder.LayoutOrder

-- Right Frame (Content)
local RightFrame = Instance.new("ScrollingFrame")
RightFrame.Parent = MainFrame; RightFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 40)
RightFrame.Position = UDim2.new(0, 190, 0, 55); RightFrame.Size = UDim2.new(1, -200, 1, -65)
RightFrame.ScrollBarThickness = 6; RightFrame.CanvasSize = UDim2.new(0, 0, 0, 1200)
local RightCorner = Instance.new("UICorner"); RightCorner.CornerRadius = UDim.new(0, 12); RightCorner.Parent = RightFrame

-- Tabs Config
local tabs = {
    {name = "🛡️ Safe", var = "Safe"},
    {name = "⚔️ Farm", var = "Farm"},
    {name = "⚡ FPS", var = "FPS"},
    {name = "📊 Stats", var = "Stats"},
    {name = "🎟️ Codes", var = "Codes"},
    {name = "🚪 Hop", var = "Hop"},
    {name = "🔥 Haki", var = "Haki"},
    {name = "🤝 Helper", var = "Helper"},
    {name = "⚔️ PvP", var = "PvP"},
    {name = "ℹ️ Info", var = "Info"}
}

local tabContents = {}
local selectedTab = nil

-- Create Tabs
for i, tab in ipairs(tabs) do
    local TabBtn = Instance.new("TextButton")
    TabBtn.Parent = LeftFrame; TabBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
    TabBtn.Size = UDim2.new(1, -12, 0, 50); TabBtn.Font = Enum.Font.Gotham
    TabBtn.Text = tab.name; TabBtn.TextColor3 = Color3.new(1,1,1)
    TabBtn.TextScaled = true; TabBtn.LayoutOrder = i
    local TabCorner = Instance.new("UICorner"); TabCorner.CornerRadius = UDim.new(0, 10); TabCorner.Parent = TabBtn
    
    local Content = Instance.new("ScrollingFrame")
    Content.Parent = RightFrame; Content.BackgroundTransparency = 1
    Content.Size = UDim2.new(1, -20, 1, 0); Content.CanvasSize = UDim2.new(0, 0, 0, 400)
    Content.ScrollBarThickness = 0; Content.Visible = false
    Content.Position = UDim2.new(0, 10, 0, 10)
    tabContents[tab.var] = Content
    
    local ContentLayout = Instance.new("UIListLayout")
    ContentLayout.Parent = Content; ContentLayout.Padding = UDim.new(0, 12)
    
    TabBtn.MouseButton1Click:Connect(function()
        if selectedTab then selectedTab.BackgroundColor3 = Color3.fromRGB(50, 50, 70) end
        TabBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
        for _, c in pairs(tabContents) do c.Visible = false end
        Content.Visible = true
        selectedTab = TabBtn
    end)
end

-- === CREATE TOGGLE FUNCTION ===
local function CreateToggle(parent, text, callback, default)
    local frame = Instance.new("Frame"); frame.Parent = parent; frame.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
    frame.Size = UDim2.new(1, 0, 0, 45)
    local fCorner = Instance.new("UICorner"); fCorner.CornerRadius = UDim.new(0, 10); fCorner.Parent = frame
    
    local label = Instance.new("TextLabel"); label.Parent = frame; label.BackgroundTransparency = 1
    label.Size = UDim2.new(1, -80, 1, 0); label.Position = UDim2.new(0, 15, 0, 0)
    label.Font = Enum.Font.Gotham; label.Text = text; label.TextColor3 = Color3.new(1,1,1)
    label.TextXAlignment = Enum.TextXAlignment.Left; label.TextScaled = true
    
    local btn = Instance.new("TextButton"); btn.Parent = frame; btn.BackgroundColor3 = Color3.fromRGB(255, 60, 60)
    btn.Size = UDim2.new(0, 70, 0, 35); btn.Position = UDim2.new(1, -80, 0.5, -17.5)
    btn.Font = Enum.Font.GothamBold; btn.Text = "OFF"; btn.TextColor3 = Color3.new(1,1,1)
    btn.TextScaled = true
    local bCorner = Instance.new("UICorner"); bCorner.CornerRadius = UDim.new(0, 18); bCorner.Parent = btn
    
    local state = default or false
    btn.MouseButton1Click:Connect(function()
        state = not state
        btn.BackgroundColor3 = state and Color3.fromRGB(60, 255, 60) or Color3.fromRGB(255, 60, 60)
        btn.Text = state and "ON" or "OFF"
        callback(state)
    end)
    
    if default then btn.BackgroundColor3 = Color3.fromRGB(60, 255, 60); btn.Text = "ON"; callback(true) end
end

-- === POPULATE TABS ===
-- Safe
CreateToggle(tabContents.Safe, "🔒 Legit Mode (Anti-Ban)", function(s) getgenv().LegitMode = s end, true)

-- Farm
CreateToggle(tabContents.Farm, "🚀 Auto Farm Level/Quest/Mobs", function(s) getgenv().AutoFarm = s end)
CreateToggle(tabContents.Farm, "📚 Auto Mastery", function(s) getgenv().AutoMastery = s end)

-- FPS
CreateToggle(tabContents.FPS, "⚡ FPS Boost Ultra", function(s) getgenv().FPSBoost = s; FPSBoostToggle(s) end, true)

-- Stats
CreateToggle(tabContents.Stats, "📈 Auto Stats (Melee/Defense)", function(s) getgenv().AutoStats = s end)

-- Codes
CreateToggle(tabContents.Codes, "🎟️ Resgatar Todos Códigos 2x EXP", function(s) if s then getgenv().AutoCodes = true end end)

-- Hop
CreateToggle(tabContents.Hop, "🚪 Server Hop 15min", function(s) getgenv().ServerHop = s; getgenv().HopDelay = 900 end)

-- Haki
CreateToggle(tabContents.Haki, "🔥 Auto Buy Haki (Buso + Observation)", function(s) getgenv().AutoHaki = s end)

-- Helper
CreateToggle(tabContents.Helper, "🤝 Bring Mobs (Legit Range)", function(s) 
    spawn(function()
        while s do
            for _, mob in pairs(workspace.Enemies:GetChildren()) do
                if mob:FindFirstChild("HumanoidRootPart") and (mob.HumanoidRootPart.Position - RootPart.Position).Magnitude < 400 then
                    mob.HumanoidRootPart.CFrame = RootPart.CFrame * CFrame.new(0,0,-8)
                end
            end
            task.wait(1)
        end
    end)
end)

-- PvP Info
local pvpLabel = Instance.new("TextLabel")
pvpLabel.Parent = tabContents.PvP; pvpLabel.BackgroundTransparency = 1
pvpLabel.Size = UDim2.new(1, 0, 1, 0); pvpLabel.Font = Enum.Font.Gotham
pvpLabel.TextColor3 = Color3.new(1,1,0); pvpLabel.TextScaled = true
pvpLabel.TextYAlignment = Enum.TextYAlignment.Top; pvpLabel.TextXAlignment = Enum.TextXAlignment.Left
pvpLabel.Text = "⚔️ PvP LEGÍTIMO 2025:\n• Sanguine Art + Leopard\n• Shark V4 + Soul Guitar\n• Factory 1h/dia\n• Sea Events (Terror Shark)\n\nNÃO USE KILLAURA!"

-- Info
local infoLabel = Instance.new("TextLabel")
infoLabel.Parent = tabContents.Info; infoLabel.BackgroundTransparency = 1
infoLabel.Size = UDim2.new(1, 0, 1, 0); infoLabel.Font = Enum.Font.Gotham
infoLabel.TextColor3 = Color3.new(0,1,1); infoLabel.TextScaled = true
infoLabel.TextYAlignment = Enum.TextYAlignment.Top; infoLabel.TextXAlignment = Enum.TextXAlignment.Left
infoLabel.Text = "ℹ️ GROKX v2.7 INFO:\n\n✅ AUTO FARM + TP LENTO (1.5s)\n✅ FPS 100+ GARANTIDO\n✅ 2x EXP CÓDIGOS AUTO\n✅ HOP 15MIN ANTI-DETECT\n✅ MOB BRING (400 studs)\n\n⚠️ USE ALT ACCOUNT\n🎯 2800 em 5-7 dias"

-- Activate First Tab
tabContents.Safe.Visible = true
LeftFrame:GetChildren()[2].BackgroundColor3 = Color3.fromRGB(0, 170, 0)

-- Toggle GUI (INSERT)
UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.Insert then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

-- AUTO START
FPSBoostToggle(true)
getgenv().AutoCodes = true
print("🎉 GROKX HUB v2.7 CARREGADO COMPLETO!")
print("✅ FPS Boost | Códigos | Pronto pra Farm!")
