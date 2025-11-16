-- GUI helper seguro e melhorado (somente UI / cliente)
-- Usa UIListLayout para posicionamento automático e salva preferências em getgenv()

-- função utilitária para salvar configurações (apenas em getgenv)
local function SaveSetting(key, value)
    getgenv().GrokX_Settings = getgenv().GrokX_Settings or {}
    getgenv().GrokX_Settings[key] = value
end
local function LoadSetting(key, default)
    getgenv().GrokX_Settings = getgenv().GrokX_Settings or {}
    if getgenv().GrokX_Settings[key] ~= nil then return getgenv().GrokX_Settings[key] end
    return default
end

-- substitui posicionamento manual por UIListLayout
local function PrepareContainer(frame)
    for _, child in ipairs(frame:GetChildren()) do
        if child:IsA("UIListLayout") then child:Destroy() end
    end
    local list = Instance.new("UIListLayout")
    list.Parent = frame
    list.SortOrder = Enum.SortOrder.LayoutOrder
    list.Padding = UDim.new(0, 8)
    frame.AutomaticCanvasSize = Enum.AutomaticSize.Y
    frame.ClipsDescendants = true
end

-- Cria um item base (frame + label)
local function CreateBaseRow(parent, height)
    local frame = Instance.new("Frame")
    frame.Parent = parent
    frame.BackgroundColor3 = Color3.fromRGB(45,45,60)
    frame.Size = UDim2.new(1, -20, 0, height or 40)
    frame.AnchorPoint = Vector2.new(0,0)
    frame.BorderSizePixel = 0
    local corner = Instance.new("UICorner"); corner.CornerRadius = UDim.new(0,8); corner.Parent = frame
    return frame
end

-- CreateToggle melhorado
local function CreateToggle(parent, key, text, callback)
    local frame = CreateBaseRow(parent, 40)
    frame.LayoutOrder = #parent:GetChildren() + 1
    local label = Instance.new("TextLabel")
    label.Parent = frame; label.BackgroundTransparency = 1
    label.Size = UDim2.new(0.7, 0, 1, 0)
    label.Font = Enum.Font.Gotham; label.Text = text
    label.TextColor3 = Color3.new(1,1,1); label.TextXAlignment = Enum.TextXAlignment.Left
    local btn = Instance.new("TextButton")
    btn.Parent = frame; btn.Size = UDim2.new(0, 70, 0, 30)
    btn.Position = UDim2.new(1, -80, 0.5, -15)
    btn.Font = Enum.Font.GothamBold; btn.TextColor3 = Color3.new(1,1,1)
    local bCorner = Instance.new("UICorner"); bCorner.CornerRadius = UDim.new(0,16); bCorner.Parent = btn

    local state = LoadSetting(key, false)
    local function updateButton()
        btn.Text = state and "ON" or "OFF"
        btn.BackgroundColor3 = state and Color3.fromRGB(60,255,60) or Color3.fromRGB(255,60,60)
    end
    updateButton()
    -- persist e callback local
    btn.MouseButton1Click:Connect(function()
        state = not state
        SaveSetting(key, state)
        updateButton()
        pcall(callback, state)
    end)
    -- chamar callback inicial se quiser
    pcall(callback, state)
    return frame
end

-- CreateDropdown melhorado
local function CreateDropdown(parent, key, text, options, default, callback)
    local frame = CreateBaseRow(parent, 40)
    frame.LayoutOrder = #parent:GetChildren() + 1
    local label = Instance.new("TextLabel")
    label.Parent = frame; label.BackgroundTransparency = 1
    label.Size = UDim2.new(0.4, 0, 1, 0)
    label.Font = Enum.Font.Gotham; label.Text = text
    label.TextColor3 = Color3.new(1,1,1); label.TextXAlignment = Enum.TextXAlignment.Left

    local dropdown = Instance.new("TextButton")
    dropdown.Parent = frame
    dropdown.Size = UDim2.new(0.55, -10, 0, 28)
    dropdown.Position = UDim2.new(0.45, 5, 0.5, -14)
    dropdown.Font = Enum.Font.Gotham
    local stored = LoadSetting(key, default)
    dropdown.Text = stored or default

    local dCorner = Instance.new("UICorner"); dCorner.CornerRadius = UDim.new(0,6); dCorner.Parent = dropdown

    local menuOpen = false
    local menuFrame = nil
    dropdown.MouseButton1Click:Connect(function()
        if menuOpen then
            if menuFrame then menuFrame:Destroy() end
            menuOpen = false
            return
        end
        menuOpen = true
        menuFrame = Instance.new("Frame")
        menuFrame.Parent = frame
        menuFrame.Size = UDim2.new(0.55, -10, 0, #options * 28)
        menuFrame.Position = UDim2.new(0.45, 5, 1, 6)
        menuFrame.BackgroundColor3 = Color3.fromRGB(50,50,70)
        local mCorner = Instance.new("UICorner"); mCorner.CornerRadius = UDim.new(0,6); mCorner.Parent = menuFrame
        for i,opt in ipairs(options) do
            local optBtn = Instance.new("TextButton"); optBtn.Parent = menuFrame
            optBtn.BackgroundTransparency = 1
            optBtn.Size = UDim2.new(1, 0, 0, 28)
            optBtn.Position = UDim2.new(0,0,0,(i-1)*28)
            optBtn.Font = Enum.Font.Gotham; optBtn.Text = opt; optBtn.TextColor3 = Color3.new(1,1,1)
            optBtn.MouseButton1Click:Connect(function()
                dropdown.Text = opt
                SaveSetting(key, opt)
                pcall(callback, opt)
                menuFrame:Destroy(); menuOpen = false
            end)
        end
    end)
    -- callback inicial
    pcall(callback, stored)
    return frame
end

-- CreateTextbox melhorado
local function CreateTextbox(parent, key, labelText, placeholder, callback)
    local frame = CreateBaseRow(parent, 40)
    frame.LayoutOrder = #parent:GetChildren() + 1
    local label = Instance.new("TextLabel")
    label.Parent = frame; label.BackgroundTransparency = 1
    label.Size = UDim2.new(0.4, 0, 1, 0)
    label.Font = Enum.Font.Gotham; label.Text = labelText
    label.TextColor3 = Color3.new(1,1,1); label.TextXAlignment = Enum.TextXAlignment.Left

    local box = Instance.new("TextBox")
    box.Parent = frame; box.Size = UDim2.new(0.55, -10, 0, 28)
    box.Position = UDim2.new(0.45, 5, 0.5, -14)
    box.Font = Enum.Font.Gotham; box.PlaceholderText = placeholder
    local stored = LoadSetting(key, "")
    box.Text = stored
    local bCorner = Instance.new("UICorner"); bCorner.CornerRadius = UDim.new(0,6); bCorner.Parent = box

    box.FocusLost:Connect(function(enter)
        if enter then
            SaveSetting(key, box.Text)
            pcall(callback, box.Text)
        end
    end)
    -- callback inicial
    pcall(callback, stored)
    return frame
end

-- Exemplo de uso:
-- prepare os contêineres
PrepareContainer(Farms)
PrepareContainer(Raids)
-- criar toggles usando chaves para persistência
CreateToggle(Farms, "auto_farm_level", "Auto Farm Level/Money (safe placeholder)", function(s)
    -- Atenção: aqui você deve implementar apenas lógica cliente-legítima, sem invocar remotes para atacar/teleportar/etc.
    print("AutoFarm (modo local):", s)
end)

CreateDropdown(Raids, "selected_raid", "Select Raid (local)", {"Flame","Ice","Sand"}, "Flame", function(opt)
    print("Raid selecionado (local):", opt)
end)

CreateTextbox(Farms, "spawn_fruit_local", "Spawn Fruit (apenas label)", "Leopard", function(txt)
    print("Spawn fruit (valor salvo local):", txt)
end)
