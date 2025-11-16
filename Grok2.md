-- HUD DEFINITIVO GROKX v2.6 — MOBILE FRIENDLY (Opção B)
-- Integra BringMob (real) + FastClick (real) + controle completo + botão Start Script (loadstring)
-- ATENÇÃO: Teste APENAS em servidor privado. Verifique URLs antes de executar.

-- ===== CONFIG / FLAGS =====
getgenv().GrokX = getgenv().GrokX or {}
local G = getgenv().GrokX

G.AntiBan = G.AntiBan or false
G.StealthMode = G.StealthMode or false
G.AutoFarm = G.AutoFarm or false
G.AutoMastery = G.AutoMastery or false
G.BringMob = G.BringMob or false
G.BringDistance = G.BringDistance or 500
G.FastClick = G.FastClick or false
G.ClickSpeed = G.ClickSpeed or 100 -- ms
G.SelectedRaid = G.SelectedRaid or "Flame"

local TEST_IN_PRIVATE = false -- <<<< Mude para true antes de rodar loadstring remoto (safety check)

-- ===== Services & Locals =====
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local VirtualUser = game:GetService("VirtualUser")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local RootPart = Character:WaitForChild("HumanoidRootPart")

local CommF_ = ReplicatedStorage:FindFirstChild("Remotes") and ReplicatedStorage.Remotes:FindFirstChild("CommF_")

-- ===== Helpers =====
local function safeLog(...)
    local t = {...}
    local s = ""
    for i,v in ipairs(t) do s = s .. tostring(v) .. (i<#t and "\t" or "") end
    print("[GROKX] ", s)
end

-- Nice small tween helper
local function Tween(instance, props, time)
    pcall(function()
        local ti = TweenInfo.new(time or 0.18, Enum.EasingStyle.Sine, Enum.EasingDirection.Out)
        game:GetService("TweenService"):Create(instance, ti, props):Play()
    end)
end

-- ===== UI BUILD (Mobile Friendly) =====
-- remove old
if PlayerGui:FindFirstChild("Grokx_HUD_Mobile") then PlayerGui.Grokx_HUD_Mobile:Destroy() end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "Grokx_HUD_Mobile"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = PlayerGui

-- Root frame (centered, mobile size friendly)
local Root = Instance.new("Frame", ScreenGui)
Root.Size = UDim2.new(0.86, 0, 0.78, 0)
Root.Position = UDim2.new(0.07, 0, 0.11, 0)
Root.BackgroundColor3 = Color3.fromRGB(18,18,18)
Root.BorderSizePixel = 0
Root.Active = true
Root.Draggable = true
local rootCorner = Instance.new("UICorner", Root); rootCorner.CornerRadius = UDim.new(0,12)

-- Header
local Header = Instance.new("Frame", Root)
Header.Size = UDim2.new(1,0,0,52)
Header.Position = UDim2.new(0,0,0,0)
Header.BackgroundColor3 = Color3.fromRGB(26,26,26)
local hcorner = Instance.new("UICorner", Header); hcorner.CornerRadius = UDim.new(0,8)

local Title = Instance.new("TextLabel", Header)
Title.Text = "GROKX HUB v2.6 (Mobile)"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18
Title.TextColor3 = Color3.fromRGB(245,245,245)
Title.BackgroundTransparency = 1
Title.Size = UDim2.new(0.6,0,1,0)
Title.Position = UDim2.new(0.03,0,0,0)

local CloseBtn = Instance.new("TextButton", Header)
CloseBtn.Text = "X"
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 18
CloseBtn.Size = UDim2.new(0,48,0,36)
CloseBtn.Position = UDim2.new(0.86,0,0.08,0)
CloseBtn.BackgroundColor3 = Color3.fromRGB(40,40,40)
local closeCorner = Instance.new("UICorner", CloseBtn); closeCorner.CornerRadius = UDim.new(0,8)

CloseBtn.MouseButton1Click:Connect(function()
    Root.Visible = false
end)

-- Tabs area (horizontal scroll for mobile)
local TabStrip = Instance.new("Frame", Root)
TabStrip.Position = UDim2.new(0,0,0,52)
TabStrip.Size = UDim2.new(1,0,0,56)
TabStrip.BackgroundTransparency = 1

local TabScroll = Instance.new("ScrollingFrame", TabStrip)
TabScroll.Size = UDim2.new(1,0,1,0)
TabScroll.Position = UDim2.new(0,0,0,0)
TabScroll.CanvasSize = UDim2.new(0,0,0,0)
TabScroll.ScrollBarThickness = 6
TabScroll.HorizontalScrollBarInset = Enum.ScrollBarInset.ScrollBar
TabScroll.VerticalScrollBarInset = Enum.ScrollBarInset.None
TabScroll.BackgroundTransparency = 1
TabScroll.ClipsDescendants = false
TabScroll.AutomaticCanvasSize = Enum.AutomaticSize.X

local tabLayout = Instance.new("UIListLayout", TabScroll)
tabLayout.FillDirection = Enum.FillDirection.Horizontal
tabLayout.Padding = UDim.new(0,6)
tabLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left

-- Content container
local Content = Instance.new("Frame", Root)
Content.Position = UDim2.new(0,0,0,110)
Content.Size = UDim2.new(1,0,1,-110)
Content.BackgroundColor3 = Color3.fromRGB(14,14,14)
local contentCorner = Instance.new("UICorner", Content); contentCorner.CornerRadius = UDim.new(0,10)

local PagesFolder = Instance.new("Folder", Content)
PagesFolder.Name = "Pages"

local function MakeTab(name)
    local btn = Instance.new("TextButton", TabScroll)
    btn.Text = name
    btn.Font = Enum.Font.GothamSemibold
    btn.TextSize = 14
    btn.TextColor3 = Color3.new(1,1,1)
    btn.BackgroundColor3 = Color3.fromRGB(28,28,28)
    btn.AutoButtonColor = true
    btn.Size = UDim2.new(0,110,0,44)
    local bcorner = Instance.new("UICorner", btn); bcorner.CornerRadius = UDim.new(0,10)

    local page = Instance.new("ScrollingFrame", PagesFolder)
    page.Name = name
    page.Size = UDim2.new(1, -12, 1, -12)
    page.Position = UDim2.new(0,6,0,6)
    page.CanvasSize = UDim2.new(0,0,1,0)
    page.ScrollBarThickness = 8
    page.BackgroundTransparency = 1
    page.Visible = false

    local layout = Instance.new("UIListLayout", page)
    layout.Padding = UDim.new(0,8)
    layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    layout.SortOrder = Enum.SortOrder.LayoutOrder

    btn.MouseButton1Click:Connect(function()
        -- show page
        for _, v in pairs(PagesFolder:GetChildren()) do v.Visible = false end
        page.Visible = true
    end)

    return btn, page
end

-- Create tabs
local tabs = {}
tabs["Dashboard"], pagesDashboard = MakeTab("Dashboard")
tabs["Auto Farm"], pagesAutoFarm = MakeTab("Auto Farm")
tabs["Mastery"], pagesMastery = MakeTab("Mastery")
tabs["Raids"], pagesRaids = MakeTab("Raids")
tabs["Extras"], pagesExtras = MakeTab("Extras")
tabs["Visual"], pagesVisual = MakeTab("Visual")
tabs["Config"], pagesConfig = MakeTab("Config")

-- show default
pagesDashboard.Visible = true

-- ===== UI Helpers for controls (big buttons, toggles, input) =====
local function BigButton(parent, label, y)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(0.9,0,0,52)
    btn.Position = UDim2.new(0.05,0,0, y or 6)
    btn.BackgroundColor3 = Color3.fromRGB(36,36,36)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 16
    btn.TextColor3 = Color3.new(1,1,1)
    local c = Instance.new("UICorner", btn); c.CornerRadius = UDim.new(0,10)
    return btn
end

local function ToggleRow(parent, label, key)
    local frame = Instance.new("Frame", parent)
    frame.Size = UDim2.new(0.95,0,0,56)
    frame.BackgroundTransparency = 1
    local labelL = Instance.new("TextLabel", frame)
    labelL.Size = UDim2.new(0.6,0,1,0)
    labelL.Position = UDim2.new(0,8,0,0)
    labelL.BackgroundTransparency = 1
    labelL.Font = Enum.Font.Gotham
    labelL.TextSize = 15
    labelL.Text = label
    labelL.TextXAlignment = Enum.TextXAlignment.Left
    labelL.TextColor3 = Color3.new(1,1,1)

    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0.34,0,0.7,0)
    btn.Position = UDim2.new(0.62,0,0.15,0)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 14
    btn.Text = G[key] and "ON" or "OFF"
    btn.BackgroundColor3 = G[key] and Color3.fromRGB(28,160,40) or Color3.fromRGB(40,40,40)
    local bc = Instance.new("UICorner", btn); bc.CornerRadius = UDim.new(0,8)
    btn.MouseButton1Click:Connect(function()
        G[key] = not G[key]
        btn.Text = G[key] and "ON" or "OFF"
        btn.BackgroundColor3 = G[key] and Color3.fromRGB(28,160,40) or Color3.fromRGB(40,40,40)
    end)
    return frame, btn
end

-- ===== Populate Dashboard =====
do
    local p = pagesDashboard
    local status = Instance.new("TextLabel", p)
    status.Size = UDim2.new(0.9,0,0,44)
    status.Position = UDim2.new(0.05,0,0,6)
    status.BackgroundTransparency = 1
    status.Font = Enum.Font.GothamBold
    status.TextSize = 16
    status.TextColor3 = Color3.new(1,1,1)
    status.Text = "Status: idle. Test in private before executing remote script."
end

-- ===== Populate Auto Farm =====
do
    local p = pagesAutoFarm
    local toggleFrame, toggleBtn = ToggleRow(p, "Auto Farm (simulated)", "AutoFarm")
    toggleFrame.Position = UDim2.new(0.025,0,0,6)

    local startBtn = BigButton(p, "Start AutoFarm (simulated)")
    startBtn.Position = UDim2.new(0.05,0,0,80)
    startBtn.MouseButton1Click:Connect(function()
        G.AutoFarm = not G.AutoFarm
        startBtn.Text = G.AutoFarm and "Stop AutoFarm (simulated)" or "Start AutoFarm (simulated)"
        safeLog("AutoFarm ->", G.AutoFarm)
    end)

    -- BringMob button (real)
    local bringBtn = BigButton(p, "Toggle BringMob (REAL)")
    bringBtn.Position = UDim2.new(0.05,0,0,150)
    local bringStateLabel = Instance.new("TextLabel", bringBtn)
    bringStateLabel.Size = UDim2.new(1,0,0.35,0)
    bringStateLabel.Position = UDim2.new(0,0,0.02,0)
    bringStateLabel.BackgroundTransparency = 1
    bringStateLabel.Font = Enum.Font.Gotham
    bringStateLabel.TextSize = 13
    bringStateLabel.TextColor3 = Color3.new(1,1,1)
    bringStateLabel.Text = "Bring Distance: " .. tostring(G.BringDistance)

    local bringToggle = Instance.new("TextButton", bringBtn)
    bringToggle.Size = UDim2.new(0.6,0,0.42,0)
    bringToggle.Position = UDim2.new(0.2,0,0.52,0)
    bringToggle.Font = Enum.Font.GothamBold
    bringToggle.TextSize = 14
    bringToggle.Text = G.BringMob and "ON" or "OFF"
    bringToggle.BackgroundColor3 = G.BringMob and Color3.fromRGB(28,160,40) or Color3.fromRGB(44,44,44)
    local bc = Instance.new("UICorner", bringToggle); bc.CornerRadius = UDim.new(0,8)

    bringToggle.MouseButton1Click:Connect(function()
        G.BringMob = not G.BringMob
        bringToggle.Text = G.BringMob and "ON" or "OFF"
        bringToggle.BackgroundColor3 = G.BringMob and Color3.fromRGB(28,160,40) or Color3.fromRGB(44,44,44)
        safeLog("BringMob ->", G.BringMob)
    end)

    -- FastClick button (real)
    local fastBtn = BigButton(p, "Toggle Fast Attack (REAL)")
    fastBtn.Position = UDim2.new(0.05,0,0,220)
    local fastToggle = Instance.new("TextButton", fastBtn)
    fastToggle.Size = UDim2.new(0.6,0,0.42,0)
    fastToggle.Position = UDim2.new(0.2,0,0.52,0)
    fastToggle.Font = Enum.Font.GothamBold
    fastToggle.TextSize = 14
    fastToggle.Text = G.FastClick and "ON" or "OFF"
    fastToggle.BackgroundColor3 = G.FastClick and Color3.fromRGB(28,160,40) or Color3.fromRGB(44,44,44)
    local fc = Instance.new("UICorner", fastToggle); fc.CornerRadius = UDim.new(0,8)

    fastToggle.MouseButton1Click:Connect(function()
        G.FastClick = not G.FastClick
        fastToggle.Text = G.FastClick and "ON" or "OFF"
        fastToggle.BackgroundColor3 = G.FastClick and Color3.fromRGB(28,160,40) or Color3.fromRGB(44,44,44)
        safeLog("FastClick ->", G.FastClick)
    end)

    -- ClickSpeed input
    local csLabel = Instance.new("TextLabel", p)
    csLabel.Size = UDim2.new(0.9,0,0,26)
    csLabel.Position = UDim2.new(0.05,0,0,292)
    csLabel.BackgroundTransparency = 1
    csLabel.Font = Enum.Font.Gotham
    csLabel.TextSize = 13
    csLabel.TextColor3 = Color3.new(1,1,1)
    csLabel.Text = "Click Speed (ms):"

    local csBox = Instance.new("TextBox", p)
    csBox.Size = UDim2.new(0.9,0,0,36)
    csBox.Position = UDim2.new(0.05,0,0,320)
    csBox.Text = tostring(G.ClickSpeed)
    csBox.Font = Enum.Font.Gotham
    csBox.TextSize = 14
    local csCorner = Instance.new("UICorner", csBox); csCorner.CornerRadius = UDim.new(0,8)
    csBox.ClearTextOnFocus = false
    csBox.FocusLost:Connect(function(enter)
        local n = tonumber(csBox.Text)
        if n then G.ClickSpeed = n; safeLog("ClickSpeed ->", n) else csBox.Text = tostring(G.ClickSpeed) end
    end)
end

-- ===== Mastery tab (simple toggles) =====
do
    local p = pagesMastery
    ToggleRow(p, "Auto Mastery (sim)", "AutoMastery")
    local drop = Instance.new("TextBox", p)
    drop.Size = UDim2.new(0.9,0,0,36)
    drop.Position = UDim2.new(0.05,0,0,80)
    drop.Text = "Mastery Type: " .. tostring(G.MasteryType or "Sword")
    drop.ClearTextOnFocus = false
    drop.FocusLost:Connect(function()
        local txt = drop.Text
        for _,opt in ipairs({"Sword","FightingStyle","DevilFruit","Gun"}) do
            if string.find(txt, opt) then G.MasteryType = opt; safeLog("MasteryType ->", opt); break end
        end
    end)
end

-- ===== Raids/Extras/Visual/Config minimal content =====
do
    local p = pagesRaids
    ToggleRow(p, "AutoRaid (sim)", "AutoRaid")
    local p2 = pagesExtras
    ToggleRow(p2, "Fruit Sniper (sim)", "FruitSniper")
    local p3 = pagesVisual
    ToggleRow(p3, "ESP (sim)", "ESP")
    local p4 = pagesConfig
    local tframe,tbtn = ToggleRow(p4, "Stealth Mode", "StealthMode")
    tframe.Position = UDim2.new(0,0,0,6)
    local privateBox = Instance.new("TextBox", p4)
    privateBox.Size = UDim2.new(0.9,0,0,36)
    privateBox.Position = UDim2.new(0.05,0,0,80)
    privateBox.Text = "TEST_IN_PRIVATE = false (click to toggle)"
    privateBox.ClearTextOnFocus = false
    privateBox.FocusLost:Connect(function()
        -- noop
    end)
    local privateBtn = Instance.new("TextButton", p4)
    privateBtn.Size = UDim2.new(0.9,0,0,46)
    privateBtn.Position = UDim2.new(0.05,0,0,122)
    privateBtn.Text = "I confirm I'm testing in private: OFF"
    privateBtn.Font = Enum.Font.GothamBold
    privateBtn.TextSize = 14
    local pbCorner = Instance.new("UICorner", privateBtn); pbCorner.CornerRadius = UDim.new(0,8)
    privateBtn.BackgroundColor3 = Color3.fromRGB(90,40,40)
    privateBtn.MouseButton1Click:Connect(function()
        TEST_IN_PRIVATE = not TEST_IN_PRIVATE
        privateBtn.Text = "I confirm I'm testing in private: " .. (TEST_IN_PRIVATE and "ON" or "OFF")
        privateBtn.BackgroundColor3 = TEST_IN_PRIVATE and Color3.fromRGB(28,160,40) or Color3.fromRGB(90,40,40)
    end)
end

-- ===== START SCRIPT PANEL (loadstring) =====
do
    local p = pagesExtras
    local label = Instance.new("TextLabel", p)
    label.Size = UDim2.new(0.9,0,0,26)
    label.Position = UDim2.new(0.05,0,0,380)
    label.BackgroundTransparency = 1
    label.Font = Enum.Font.Gotham
    label.TextSize = 13
    label.Text = "Enter RAW URL to load (https://.../raw)"

    local urlBox = Instance.new("TextBox", p)
    urlBox.Size = UDim2.new(0.9,0,0,36)
    urlBox.Position = UDim2.new(0.05,0,0,410)
    urlBox.Text = "https://"
    urlBox.Font = Enum.Font.Gotham
    urlBox.TextSize = 14
    local urlCorner = Instance.new("UICorner", urlBox); urlCorner.CornerRadius = UDim.new(0,8)

    local runBtn = Instance.new("TextButton", p)
    runBtn.Size = UDim2.new(0.9,0,0,46)
    runBtn.Position = UDim2.new(0.05,0,0,460)
    runBtn.Text = "Start Script (loadstring)"
    runBtn.Font = Enum.Font.GothamBold
    runBtn.TextSize = 15
    local rcorner = Instance.new("UICorner", runBtn); rcorner.CornerRadius = UDim.new(0,8)
    runBtn.BackgroundColor3 = Color3.fromRGB(28,120,220)

    runBtn.MouseButton1Click:Connect(function()
        local url = tostring(urlBox.Text or "")
        if not TEST_IN_PRIVATE then
            safeLog("Refused to run remote script: TEST_IN_PRIVATE is not ON.")
            return
        end
        if url == "" or not string.find(url, "http") then
            safeLog("Invalid URL.")
            return
        end
        -- Optional: simple confirmation dialog
        local confirmed = true
        if confirmed then
            safeLog("Loading remote script from:", url)
            local ok, res = pcall(function() return loadstring(game:HttpGet(url))() end)
            if not ok then safeLog("Remote script error:", res) else safeLog("Remote script executed.") end
        end
    end)
end

-- ===== REAL BEHAVIORS (BringMob & FastClick) =====
-- BringMob (real) - moves enemies' HumanoidRootPart behind player within distance
spawn(function()
    while task.wait(0.12) do
        if G.BringMob then
            pcall(function()
                if workspace:FindFirstChild("Enemies") then
                    for _,enemy in pairs(workspace.Enemies:GetChildren()) do
                        if enemy and enemy:FindFirstChild("HumanoidRootPart") and enemy:FindFirstChild("Humanoid") then
                            local hrp = enemy.HumanoidRootPart
                            local hum = enemy.Humanoid
                            if hum.Health > 0 then
                                local dist = (hrp.Position - RootPart.Position).Magnitude
                                if dist <= (G.BringDistance or 500) then
                                    -- Move enemy behind player
                                    hrp.CFrame = RootPart.CFrame * CFrame.new(0, 0, -6)
                                    -- try to 'stun' to prevent movement
                                    pcall(function()
                                        if hum then
                                            hum.WalkSpeed = 0
                                            hum.JumpPower = 0
                                        end
                                    end)
                                end
                            end
                        end
                    end
                end
            end)
        end
    end
end)

-- FastClick (real)
spawn(function()
    while task.wait(math.max(0.001, (G.ClickSpeed or 100)/1000)) do
        if G.FastClick then
            pcall(function()
                -- Use VirtualUser to emulate clicks
                VirtualUser:ClickButton1(Vector2.new(math.random(200,900), math.random(200,900)))
            end)
        end
    end
end)

-- ===== Cleanup on GUI close/unload =====
ScreenGui.Destroying:Connect(function()
    -- ensure flags off to avoid stray loops acting unexpectedly
    G.BringMob = false
    G.FastClick = false
    safeLog("Grokx HUD destroyed - flags reset.")
end)

-- ===== Final info =====
safeLog("GROKX Mobile HUD loaded. Ensure TEST_IN_PRIVATE = true before using loadstring.")
