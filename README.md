-- xX_Xx Hub - Universal Script (ESP Chams + Palitinhos + Aimbot Wall & Limpo)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Remove interface antiga se já existir
if CoreGui:FindFirstChild("xXxHubESP") then
    CoreGui.xXxHubESP:Destroy()
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "xXxHubESP"
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Botão Flutuante Arredondado e Arrastável com a imagem correta (ID: 116269572976664) e fundo preto
local FloatingButton = Instance.new("ImageButton")
FloatingButton.Name = "FloatingButton"
FloatingButton.Parent = ScreenGui
FloatingButton.BackgroundColor3 = Color3.fromRGB(0, 0, 0) -- Fundo Preto
FloatingButton.BorderColor3 = Color3.fromRGB(255, 42, 42)
FloatingButton.BorderSizePixel = 2
FloatingButton.Position = UDim2.new(0.05, 0, 0.2, 0)
FloatingButton.Size = UDim2.new(0, 60, 0, 60)
FloatingButton.Image = "rbxassetid://116269572976664"
FloatingButton.ScaleType = Enum.ScaleType.Crop
FloatingButton.Active = true
FloatingButton.Draggable = true

local UICornerBtn = Instance.new("UICorner")
UICornerBtn.CornerRadius = UDim.new(1, 0)
UICornerBtn.Parent = FloatingButton

-- Painel Principal do Menu
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
MainFrame.BorderColor3 = Color3.fromRGB(255, 42, 42)
MainFrame.BorderSizePixel = 2
MainFrame.Position = UDim2.new(0.5, -160, 0.5, -200)
MainFrame.Size = UDim2.new(0, 320, 0, 400)
MainFrame.Visible = false
MainFrame.Active = true
MainFrame.Draggable = true

local UICornerMain = Instance.new("UICorner")
UICornerMain.CornerRadius = UDim.new(0, 12)
UICornerMain.Parent = MainFrame

-- Título do Menu
local Title = Instance.new("TextLabel")
Title.Parent = MainFrame
Title.BackgroundTransparency = 1
Title.Position = UDim2.new(0, 0, 0, 10)
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Font = Enum.Font.SourceSansBold
Title.Text = "xX_Xx Hub - Combat"
Title.TextColor3 = Color3.fromRGB(255, 42, 42)
Title.TextSize = 22

-- Função para criar botões de toggle
local function createToggle(name, posY, callback)
    local btn = Instance.new("TextButton")
    btn.Parent = MainFrame
    btn.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
    btn.BorderColor3 = Color3.fromRGB(255, 42, 42)
    btn.Position = UDim2.new(0.1, 0, 0, posY)
    btn.Size = UDim2.new(0.8, 0, 0, 45)
    btn.Font = Enum.Font.SourceSansBold
    btn.Text = name .. ": [OFF]"
    btn.TextColor3 = Color3.fromRGB(200, 200, 200)
    btn.TextSize = 16
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = btn
    
    local state = false
    btn.MouseButton1Click:Connect(function()
        state = not state
        if state then
            btn.Text = name .. ": [ON]"
            btn.TextColor3 = Color3.fromRGB(42, 255, 42)
            btn.BackgroundColor3 = Color3.fromRGB(35, 15, 15)
        else
            btn.Text = name .. ": [OFF]"
            btn.TextColor3 = Color3.fromRGB(200, 200, 200)
            btn.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
        end
        callback(state)
    end)
end

-- Variáveis de controle
local espEnabled = false
local chamsEnabled = false
local aimbotWallEnabled = false
local aimbotLimpoEnabled = false

-- Abrir / Fechar Menu pelo Botão Flutuante
FloatingButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

-- Opções do Menu
createToggle("ESP Palitinhos, Vida & Linhas", 55, function(state)
    espEnabled = state
end)

createToggle("Chams (Vermelho Brilhante)", 110, function(state)
    chamsEnabled = state
end)

createToggle("Aimbot Wall", 165, function(state)
    aimbotWallEnabled = state
    if state then aimbotLimpoEnabled = false end
end)

createToggle("Aimbot Limpo", 220, function(state)
    aimbotLimpoEnabled = state
    if state then aimbotWallEnabled = false end
end)

-- Botão de Fechar Painel Interno
local CloseBtn = Instance.new("TextButton")
CloseBtn.Parent = MainFrame
CloseBtn.BackgroundColor3 = Color3.fromRGB(180, 20, 20)
CloseBtn.BorderSizePixel = 0
CloseBtn.Position = UDim2.new(0.1, 0, 0, 335)
CloseBtn.Size = UDim2.new(0.8, 0, 0, 35)
CloseBtn.Font = Enum.Font.SourceSansBold
CloseBtn.Text = "Fechar Tela"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 16
local CornerClose = Instance.new("UICorner")
CornerClose.CornerRadius = UDim.new(0, 8)
CornerClose.Parent = CloseBtn

CloseBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
end)

-- SISTEMA DE ESP E CHAMS
local espCache = {}

local function createDrawingObjects()
    local objs = {}
    objs.tracer = Drawing.new("Line")
    objs.tracer.Visible = false
    objs.tracer.Color = Color3.fromRGB(255, 0, 0)
    objs.tracer.Thickness = 1.5

    objs.bones = {
        Drawing.new("Line"), -- Cabeça ao Tronco
        Drawing.new("Line"), -- Tronco a Membros
        Drawing.new("Line")
    }
    for _, bone in ipairs(objs.bones) do
        bone.Visible = false
        bone.Color = Color3.fromRGB(255, 0, 0)
        bone.Thickness = 2
    end

    objs.healthBar = Drawing.new("Line")
    objs.healthBar.Visible = false
    objs.healthBar.Thickness = 3
    
    return objs
end

Players.PlayerRemoving:Connect(function(player)
    if espCache[player] then
        for _, obj in pairs(espCache[player].bones) do obj:Remove() end
        espCache[player].tracer:Remove()
        espCache[player].healthBar:Remove()
        espCache[player] = nil
    end
end)

-- LOOP DE RENDERIZAÇÃO
RunService.RenderStepped:Connect(function()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") then
            if not espCache[player] then
                espCache[player] = createDrawingObjects()
            end
            
            local cache = espCache[player]
            local char = player.Character
            local rootPart = char.HumanoidRootPart
            local head = char:FindFirstChild("Head")
            local humanoid = char.Humanoid
            
            -- GERENCIAMENTO DE CHAMS (Highlight vermelho brilhante nos bonecos)
            local highlight = char:FindFirstChild("xXxChams")
            if chamsEnabled and humanoid.Health > 0 then
                if not highlight then
                    highlight = Instance.new("Highlight")
                    highlight.Name = "xXxChams"
                    highlight.Adornee = char
                    highlight.FillColor = Color3.fromRGB(255, 0, 0)
                    highlight.FillTransparency = 0.5
                    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                    highlight.OutlineTransparency = 0
                    highlight.Parent = char
                end
            else
                if highlight then
                    highlight:Destroy()
                end
            end
            
            local rootPos, onScreen = Camera:WorldToViewportPoint(rootPart.Position)
            
            if espEnabled and onScreen and humanoid.Health > 0 then
                -- 1. Linha da tela até o boneco
                cache.tracer.Visible = true
                cache.tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                cache.tracer.To = Vector2.new(rootPos.X, rootPos.Y)
                
                -- 2. Palitinhos vermelhos (Esqueleto)
                if head and (char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso")) then
                    local torso = char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso")
                    local headPos = Camera:WorldToViewportPoint(head.Position)
                    local torsoPos = Camera:WorldToViewportPoint(torso.Position)
                    
                    cache.bones[1].Visible = true
                    cache.bones[1].From = Vector2.new(headPos.X, headPos.Y)
                    cache.bones[1].To = Vector2.new(torsoPos.X, torsoPos.Y)
                    
                    cache.bones[2].Visible = true
                    cache.bones[2].From = Vector2.new(torsoPos.X, torsoPos.Y)
                    cache.bones[2].To = Vector2.new(rootPos.X, rootPos.Y)
                else
                    cache.bones[1].Visible = false
                    cache.bones[2].Visible = false
                end

                -- 3. Barra de Vida
                local healthPercent = humanoid.Health / humanoid.MaxHealth
                cache.healthBar.Visible = true
                cache.healthBar.From = Vector2.new(rootPos.X - 25, rootPos.Y + 30)
                cache.healthBar.To = Vector2.new(rootPos.X - 25, (rootPos.Y + 30) - (60 * healthPercent))
                cache.healthBar.Color = Color3.fromRGB(255 - (healthPercent * 255), healthPercent * 255, 0)
            else
                cache.tracer.Visible = false
                cache.bones[1].Visible = false
                cache.bones[2].Visible = false
                cache.healthBar.Visible = false
            end
        end
    end

    -- SISTEMA DE AIMBOT (Wall & Limpo)
    if aimbotWallEnabled or aimbotLimpoEnabled then
        local closestTarget = nil
        local shortestDistance = math.huge
        
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") and player.Character:FindFirstChild("Humanoid") then
                local humanoid = player.Character.Humanoid
                if humanoid.Health > 0 then
                    local head = player.Character.Head
                    local headPos, onScreen = Camera:WorldToViewportPoint(head.Position)
                    
                    local isValid = false
                    if aimbotWallEnabled and onScreen then
                        isValid = true
                    elseif aimbotLimpoEnabled and onScreen then
                        -- Checa se está visível na tela sem paredes na frente (Aimbot Limpo)
                        local rayParams = RaycastParams.new()
                        rayParams.FilterDescendantsInstances = {LocalPlayer.Character, player.Character}
                        rayParams.FilterType = Enum.RaycastFilterType.Exclude
                        local result = workspace:Raycast(Camera.CFrame.Position, (head.Position - Camera.CFrame.Position), rayParams)
                        if not result then
                            isValid = true
                        end
                    end
                    
                    if isValid then
                        local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
                        local distanceToCenter = (Vector2.new(headPos.X, headPos.Y) - screenCenter).Magnitude
                        
                        if distanceToCenter < shortestDistance then
                            shortestDistance = distanceToCenter
                            closestTarget = head
                        end
                    end
                end
            end
        end
        
        if closestTarget then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, closestTarget.Position)
        end
    end
end)
