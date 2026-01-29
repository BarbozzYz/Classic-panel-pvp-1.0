--[[
    ╔══════════════════════════════════════════╗
    ║   PANEL PVP CLASSIC MENU 1.0            ║
    ║   ESP HEALTH CORRIGIDO + COLORS FIXO    ║
    ╚══════════════════════════════════════════╝
]]

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local StarterGui = game:GetService("StarterGui")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

-- Notificação
StarterGui:SetCore("SendNotification", {
    Title = "CLASSIC PANEL PVP";
    Text = "✅ The best script was executed successfully!";
    Duration = 5;
})

-- Configurações
local Settings = {
    Combat = {
        Aimbot = false,
        AimbotType = "Ao mirar",
        AimbotKey = Enum.UserInputType.MouseButton2,
        FOV = false,
        FOVSize = 150,
        Hitbox = false,
        HitboxSize = 15,
        NoRecoil = false,
        NoHit = false
    },
    ESP = {
        Enabled = false,
        Team = false,
        Chams = false,
        ChamsTeam = false,
        Health = false,
        Name = false,
        Meters = false,
        Line = false,
        LinePlacement = "Top",
        Box = false
    },
    Colors = {
        General = Color3.fromRGB(255, 255, 255),
        PanelOnly = Color3.fromRGB(255, 255, 255),
        GeneralH = 0,
        GeneralS = 0,
        GeneralL = 100,
        PanelH = 0,
        PanelS = 0,
        PanelL = 100
    },
    Various = {
        NoClip = false,
        WalkSpeed = 16,
        JumpPower = 50,
        Fly = false,
        FlySpeed = 50,
        Macro = false,
        MacroSpeed = 0.05
    },
    SessionTime = 0
}

-- Variáveis Globais
local ESPObjects = {}
local FOVCircle = nil
local NoHitPart = nil
local FlyBodyVelocity = nil
local FlyBodyGyro = nil
local MacroConnection = nil
local AimbotConnection = nil
local NoClipConnection = nil
local WalkSpeedConnection = nil
local JumpPowerConnection = nil
local OriginalHitboxSizes = {}
local CurrentTarget = nil
local GUI = nil

-- Função de proteção
local function safeCall(func)
    local success, err = pcall(func)
    if not success then
        warn("⚠️ Erro:", err)
    end
    return success
end

-- FUNÇÃO HSL PARA RGB
local function HSLToRGB(h, s, l)
    h = h / 360
    s = s / 100
    l = l / 100
    
    local r, g, b
    
    if s == 0 then
        r, g, b = l, l, l
    else
        local function hue2rgb(p, q, t)
            if t < 0 then t = t + 1 end
            if t > 1 then t = t - 1 end
            if t < 1/6 then return p + (q - p) * 6 * t end
            if t < 1/2 then return q end
            if t < 2/3 then return p + (q - p) * (2/3 - t) * 6 end
            return p
        end
        
        local q = l < 0.5 and l * (1 + s) or l + s - l * s
        local p = 2 * l - q
        
        r = hue2rgb(p, q, h + 1/3)
        g = hue2rgb(p, q, h)
        b = hue2rgb(p, q, h - 1/3)
    end
    
    return Color3.fromRGB(math.floor(r * 255), math.floor(g * 255), math.floor(b * 255))
end

-- FUNÇÃO PARA APLICAR CORES
local function ApplyGeneralColor(color)
    if not GUI then return end
    
    -- Aplicar em TUDO (textos, bordas, backgrounds)
    for _, obj in pairs(GUI:GetDescendants()) do
        if obj:IsA("TextLabel") or obj:IsA("TextButton") then
            obj.TextColor3 = color
        end
        if obj:IsA("UIStroke") then
            obj.Color = color
        end
        if obj:IsA("Frame") and obj.Name ~= "MainFrame" and obj.BackgroundTransparency ~= 1 then
            obj.BackgroundColor3 = color
        end
    end
end

local function ApplyPanelOnlyColor(color)
    if not GUI then return end
    
    -- Aplicar APENAS em textos (não em backgrounds)
    for _, obj in pairs(GUI:GetDescendants()) do
        if obj:IsA("TextLabel") or obj:IsA("TextButton") then
            obj.TextColor3 = color
        end
        if obj:IsA("UIStroke") then
            obj.Color = color
        end
    end
end

-- ============================================
-- SISTEMAS FUNCIONAIS
-- ============================================

-- AIMBOT FUNCIONAL
local function GetClosestPlayerToCursor()
    if not Settings.Combat.Aimbot then return nil end
    
    local closestPlayer = nil
    local shortestDistance = Settings.Combat.FOVSize
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            local screenPoint, onScreen = Camera:WorldToViewportPoint(head.Position)
            
            if onScreen then
                local mouseLocation = UserInputService:GetMouseLocation()
                local distance = (Vector2.new(screenPoint.X, screenPoint.Y) - mouseLocation).Magnitude
                
                if distance < shortestDistance then
                    closestPlayer = player
                    shortestDistance = distance
                end
            end
        end
    end
    
    return closestPlayer
end

local function UpdateAimbot()
    if Settings.Combat.Aimbot then
        local isAiming = false
        
        if Settings.Combat.AimbotType == "Ao mirar" then
            isAiming = UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
        elseif Settings.Combat.AimbotType == "ToggleKey" then
            isAiming = UserInputService:IsKeyDown(Settings.Combat.AimbotKey) or 
                       UserInputService:IsMouseButtonPressed(Settings.Combat.AimbotKey)
        end
        
        if isAiming then
            CurrentTarget = GetClosestPlayerToCursor()
            if CurrentTarget and CurrentTarget.Character and CurrentTarget.Character:FindFirstChild("Head") then
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, CurrentTarget.Character.Head.Position)
            end
        end
    end
end

-- HITBOX FUNCIONAL
local function ApplyHitbox()
    safeCall(function()
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local hrp = player.Character:FindFirstChild("HumanoidRootPart")
                
                if hrp then
                    if Settings.Combat.Hitbox then
                        if not OriginalHitboxSizes[hrp] then
                            OriginalHitboxSizes[hrp] = hrp.Size
                        end
                        
                        hrp.Size = Vector3.new(Settings.Combat.HitboxSize, Settings.Combat.HitboxSize, Settings.Combat.HitboxSize)
                        hrp.Transparency = 0.7
                        hrp.Color = Color3.fromRGB(255, 0, 0)
                        hrp.Material = Enum.Material.Neon
                        hrp.CanCollide = false
                    else
                        if OriginalHitboxSizes[hrp] then
                            hrp.Size = OriginalHitboxSizes[hrp]
                            hrp.Transparency = 1
                            hrp.CanCollide = false
                        end
                    end
                end
            end
        end
    end)
end

-- NO RECOIL FUNCIONAL
local function ApplyNoRecoil()
    safeCall(function()
        if not Settings.Combat.NoRecoil then return end
        
        if LocalPlayer.Character then
            for _, tool in pairs(LocalPlayer.Character:GetChildren()) do
                if tool:IsA("Tool") then
                    for _, obj in pairs(tool:GetDescendants()) do
                        if obj:IsA("NumberValue") then
                            local name = obj.Name:lower()
                            if name:find("recoil") or name:find("spread") or name:find("kickback") then
                                obj.Value = 0
                            end
                        end
                    end
                end
            end
        end
        
        if LocalPlayer.Backpack then
            for _, tool in pairs(LocalPlayer.Backpack:GetChildren()) do
                if tool:IsA("Tool") then
                    for _, obj in pairs(tool:GetDescendants()) do
                        if obj:IsA("NumberValue") then
                            local name = obj.Name:lower()
                            if name:find("recoil") or name:find("spread") or name:find("kickback") then
                                obj.Value = 0
                            end
                        end
                    end
                end
            end
        end
    end)
end

-- ESP FUNCIONAL COM HEALTH BAR CORRIGIDA
local function CreateESP(player)
    if player == LocalPlayer then return end
    
    local function UpdateESP()
        safeCall(function()
            if not Settings.ESP.Enabled then
                if ESPObjects[player] then
                    for _, obj in pairs(ESPObjects[player]) do
                        if obj then
                            if typeof(obj) == "Instance" then
                                obj:Destroy()
                            elseif typeof(obj) == "table" and obj.Remove then
                                obj:Remove()
                            end
                        end
                    end
                    ESPObjects[player] = nil
                end
                return
            end
            
            if not player.Parent or not player.Character then return end
            
            ESPObjects[player] = ESPObjects[player] or {}
            
            -- Highlight ESP
            if not ESPObjects[player].Highlight then
                local highlight = Instance.new("Highlight")
                highlight.Adornee = player.Character
                highlight.FillColor = Color3.fromRGB(128, 128, 128)
                highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                highlight.FillTransparency = 0.5
                highlight.OutlineTransparency = 0
                highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                highlight.Parent = player.Character
                ESPObjects[player].Highlight = highlight
            end
            
            -- ESP Team Colors
            if Settings.ESP.Team and player.Team then
                ESPObjects[player].Highlight.FillColor = player.Team.TeamColor.Color
                ESPObjects[player].Highlight.OutlineColor = player.Team.TeamColor.Color
            end
            
            -- ESP Health Bar CORRIGIDA (Tamanho Fixo)
            if Settings.ESP.Health and player.Character:FindFirstChild("Head") and player.Character:FindFirstChild("Humanoid") then
                if not ESPObjects[player].HealthBar then
                    local billboard = Instance.new("BillboardGui")
                    billboard.Adornee = player.Character.Head
                    billboard.Size = UDim2.new(0, 4, 0, 5) -- TAMANHO FIXO PEQUENO
                    billboard.StudsOffset = Vector3.new(-2, 0, 0)
                    billboard.AlwaysOnTop = true
                    billboard.Parent = player.Character.Head
                    
                    local frame = Instance.new("Frame")
                    frame.Size = UDim2.new(1, 0, 1, 0)
                    frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
                    frame.BorderSizePixel = 1
                    frame.BorderColor3 = Color3.fromRGB(0, 0, 0)
                    frame.Parent = billboard
                    
                    local bar = Instance.new("Frame")
                    bar.Size = UDim2.new(1, 0, 1, 0)
                    bar.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
                    bar.BorderSizePixel = 0
                    bar.Parent = frame
                    
                    ESPObjects[player].HealthBar = {Billboard = billboard, Bar = bar}
                end
                
                local humanoid = player.Character.Humanoid
                local healthPercent = humanoid.Health / humanoid.MaxHealth
                
                -- Atualizar tamanho da barra verticalmente
                ESPObjects[player].HealthBar.Bar.Size = UDim2.new(1, 0, healthPercent, 0)
                ESPObjects[player].HealthBar.Bar.Position = UDim2.new(0, 0, 1 - healthPercent, 0)
                
                -- Cores baseadas na vida
                if healthPercent > 0.75 then
                    ESPObjects[player].HealthBar.Bar.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
                elseif healthPercent > 0.5 then
                    ESPObjects[player].HealthBar.Bar.BackgroundColor3 = Color3.fromRGB(255, 255, 0)
                elseif healthPercent > 0.25 then
                    ESPObjects[player].HealthBar.Bar.BackgroundColor3 = Color3.fromRGB(255, 165, 0)
                else
                    ESPObjects[player].HealthBar.Bar.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
                end
            elseif not Settings.ESP.Health and ESPObjects[player].HealthBar then
                ESPObjects[player].HealthBar.Billboard:Destroy()
                ESPObjects[player].HealthBar = nil
            end
            
            -- ESP Name
            if Settings.ESP.Name and player.Character:FindFirstChild("Head") then
                if not ESPObjects[player].Name then
                    local billboard = Instance.new("BillboardGui")
                    billboard.Adornee = player.Character.Head
                    billboard.Size = UDim2.new(0, 200, 0, 50)
                    billboard.StudsOffset = Vector3.new(0, 2, 0)
                    billboard.AlwaysOnTop = true
                    billboard.Parent = player.Character.Head
                    
                    local label = Instance.new("TextLabel")
                    label.Size = UDim2.new(1, 0, 1, 0)
                    label.BackgroundTransparency = 1
                    label.Text = player.Name
                    label.TextColor3 = Color3.fromRGB(255, 255, 255)
                    label.Font = Enum.Font.SourceSansBold
                    label.TextSize = 16
                    label.TextStrokeTransparency = 0
                    label.Parent = billboard
                    
                    ESPObjects[player].Name = billboard
                end
            elseif not Settings.ESP.Name and ESPObjects[player].Name then
                ESPObjects[player].Name:Destroy()
                ESPObjects[player].Name = nil
            end
            
            -- ESP Meters
            if Settings.ESP.Meters and player.Character:FindFirstChild("Head") and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                if not ESPObjects[player].Meters then
                    local billboard = Instance.new("BillboardGui")
                    billboard.Adornee = player.Character.Head
                    billboard.Size = UDim2.new(0, 200, 0, 50)
                    billboard.StudsOffset = Vector3.new(0, 3, 0)
                    billboard.AlwaysOnTop = true
                    billboard.Parent = player.Character.Head
                    
                    local label = Instance.new("TextLabel")
                    label.Size = UDim2.new(1, 0, 1, 0)
                    label.BackgroundTransparency = 1
                    label.TextColor3 = Color3.fromRGB(255, 255, 255)
                    label.Font = Enum.Font.SourceSansBold
                    label.TextSize = 14
                    label.TextStrokeTransparency = 0
                    label.Parent = billboard
                    
                    ESPObjects[player].Meters = label
                end
                
                local distance = (player.Character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
                ESPObjects[player].Meters.Text = string.format("%.0fm", distance)
            elseif not Settings.ESP.Meters and ESPObjects[player].Meters then
                ESPObjects[player].Meters.Parent:Destroy()
                ESPObjects[player].Meters = nil
            end
            
            -- ESP Line
            if Settings.ESP.Line and player.Character:FindFirstChild("HumanoidRootPart") then
                if not ESPObjects[player].Line then
                    local line = Drawing.new("Line")
                    line.Visible = true
                    line.Color = Color3.new(1, 1, 1)
                    line.Thickness = 1
                    line.Transparency = 1
                    ESPObjects[player].Line = line
                end
                
                local hrp = player.Character.HumanoidRootPart
                local screenPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                
                if onScreen then
                    local startY = Settings.ESP.LinePlacement == "Top" and 0 or Camera.ViewportSize.Y
                    ESPObjects[player].Line.From = Vector2.new(Camera.ViewportSize.X / 2, startY)
                    ESPObjects[player].Line.To = Vector2.new(screenPos.X, screenPos.Y)
                    ESPObjects[player].Line.Visible = true
                else
                    ESPObjects[player].Line.Visible = false
                end
            elseif not Settings.ESP.Line and ESPObjects[player].Line then
                ESPObjects[player].Line:Remove()
                ESPObjects[player].Line = nil
            end
            
            -- ESP Box
            if Settings.ESP.Box and player.Character:FindFirstChild("HumanoidRootPart") then
                if not ESPObjects[player].Box then
                    local box = Drawing.new("Square")
                    box.Visible = true
                    box.Color = Color3.new(1, 1, 1)
                    box.Thickness = 1
                    box.Transparency = 1
                    box.Filled = false
                    ESPObjects[player].Box = box
                end
                
                local hrp = player.Character.HumanoidRootPart
                local screenPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                
                if onScreen then
                    local topPos = Camera:WorldToViewportPoint((hrp.CFrame * CFrame.new(0, 3, 0)).Position)
                    local bottomPos = Camera:WorldToViewportPoint((hrp.CFrame * CFrame.new(0, -3, 0)).Position)
                    
                    local height = math.abs(topPos.Y - bottomPos.Y)
                    local width = height / 2
                    
                    ESPObjects[player].Box.Size = Vector2.new(width, height)
                    ESPObjects[player].Box.Position = Vector2.new(screenPos.X - width / 2, screenPos.Y - height / 2)
                    ESPObjects[player].Box.Visible = true
                else
                    ESPObjects[player].Box.Visible = false
                end
            elseif not Settings.ESP.Box and ESPObjects[player].Box then
                ESPObjects[player].Box:Remove()
                ESPObjects[player].Box = nil
            end
            
            -- Chams
            if Settings.ESP.Chams and player.Character then
                if not ESPObjects[player].Chams then
                    ESPObjects[player].Chams = {}
                    
                    for _, part in pairs(player.Character:GetDescendants()) do
                        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                            local box = Instance.new("BoxHandleAdornment")
                            box.Size = part.Size
                            box.Adornee = part
                            box.AlwaysOnTop = true
                            box.ZIndex = 10
                            box.Transparency = 0.5
                            box.Color3 = Color3.fromRGB(128, 128, 128)
                            box.Parent = part
                            
                            table.insert(ESPObjects[player].Chams, box)
                        end
                    end
                end
                
                if Settings.ESP.ChamsTeam and player.Team then
                    for _, box in pairs(ESPObjects[player].Chams) do
                        box.Color3 = player.Team.TeamColor.Color
                    end
                end
            elseif not Settings.ESP.Chams and ESPObjects[player].Chams then
                for _, box in pairs(ESPObjects[player].Chams) do
                    box:Destroy()
                end
                ESPObjects[player].Chams = nil
            end
        end)
    end
    
    player.CharacterAdded:Connect(function()
        task.wait(0.5)
        UpdateESP()
    end)
    
    RunService.RenderStepped:Connect(UpdateESP)
end

-- FLY FUNCIONAL
local function ToggleFly(state)
    if state then
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = LocalPlayer.Character.HumanoidRootPart
            
            FlyBodyVelocity = Instance.new("BodyVelocity")
            FlyBodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
            FlyBodyVelocity.Velocity = Vector3.new(0, 0, 0)
            FlyBodyVelocity.Parent = hrp
            
            FlyBodyGyro = Instance.new("BodyGyro")
            FlyBodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
            FlyBodyGyro.P = 9e9
            FlyBodyGyro.CFrame = hrp.CFrame
            FlyBodyGyro.Parent = hrp
            
            RunService.RenderStepped:Connect(function()
                if Settings.Various.Fly and FlyBodyVelocity and FlyBodyGyro then
                    local cam = workspace.CurrentCamera
                    local velocity = Vector3.new(0, 0, 0)
                    
                    if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                        velocity = velocity + (cam.CFrame.LookVector * Settings.Various.FlySpeed)
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                        velocity = velocity - (cam.CFrame.LookVector * Settings.Various.FlySpeed)
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                        velocity = velocity - (cam.CFrame.RightVector * Settings.Various.FlySpeed)
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                        velocity = velocity + (cam.CFrame.RightVector * Settings.Various.FlySpeed)
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                        velocity = velocity + Vector3.new(0, Settings.Various.FlySpeed, 0)
                    end
                    if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
                        velocity = velocity - Vector3.new(0, Settings.Various.FlySpeed, 0)
                    end
                    
                    FlyBodyVelocity.Velocity = velocity
                    FlyBodyGyro.CFrame = cam.CFrame
                end
            end)
        end
    else
        if FlyBodyVelocity then
            FlyBodyVelocity:Destroy()
            FlyBodyVelocity = nil
        end
        if FlyBodyGyro then
            FlyBodyGyro:Destroy()
            FlyBodyGyro = nil
        end
    end
end

-- MACRO FUNCIONAL
local function ToggleMacro(state)
    if state then
        MacroConnection = RunService.Heartbeat:Connect(function()
            if Settings.Various.Macro and LocalPlayer.Backpack then
                local toolCount = #LocalPlayer.Backpack:GetChildren()
                for i = 1, math.min(toolCount, 9) do
                    task.wait(Settings.Various.MacroSpeed)
                    local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid")
                    if humanoid then
                        local tool = LocalPlayer.Backpack:FindFirstChildOfClass("Tool")
                        if tool then
                            humanoid:EquipTool(tool)
                        end
                    end
                end
            end
        end)
    else
        if MacroConnection then
            MacroConnection:Disconnect()
            MacroConnection = nil
        end
    end
end

-- ============================================
-- CRIAR PAINEL COM COLORS CORRETO
-- ============================================
local function CreatePanel()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "ClassicPVPPanel"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    GUI = ScreenGui
    
    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "MainFrame"
    MainFrame.Size = UDim2.new(0, 900, 0, 600)
    MainFrame.Position = UDim2.new(0.5, -450, 0.5, -300)
    MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    MainFrame.BorderSizePixel = 0
    MainFrame.Active = true
    MainFrame.Draggable = true
    MainFrame.Parent = ScreenGui
    
    local MainStroke = Instance.new("UIStroke")
    MainStroke.Color = Color3.fromRGB(200, 200, 200)
    MainStroke.Thickness = 3
    MainStroke.Parent = MainFrame
    
    local TitleBar = Instance.new("Frame")
    TitleBar.Size = UDim2.new(1, 0, 0, 50)
    TitleBar.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    TitleBar.BorderSizePixel = 0
    TitleBar.Parent = MainFrame
    
    local TitleStroke = Instance.new("UIStroke")
    TitleStroke.Color = Color3.fromRGB(200, 200, 200)
    TitleStroke.Thickness = 2
    TitleStroke.Parent = TitleBar
    
    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(1, -120, 1, 0)
    Title.Position = UDim2.new(0, 15, 0, 0)
    Title.BackgroundTransparency = 1
    Title.Text = "PANEL PVP CLASSIC MENU 1.0"
    Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title.Font = Enum.Font.Code
    Title.TextSize = 18
    Title.TextXAlignment = Enum.TextXAlignment.Left
    Title.Parent = TitleBar
    
    local MinButton = Instance.new("TextButton")
    MinButton.Size = UDim2.new(0, 45, 0, 40)
    MinButton.Position = UDim2.new(1, -95, 0, 5)
    MinButton.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    MinButton.Text = "━"
    MinButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    MinButton.Font = Enum.Font.Code
    MinButton.TextSize = 20
    MinButton.BorderSizePixel = 1
    MinButton.BorderColor3 = Color3.fromRGB(200, 200, 200)
    MinButton.Parent = TitleBar
    
    local CloseButton = Instance.new("TextButton")
    CloseButton.Size = UDim2.new(0, 45, 0, 40)
    CloseButton.Position = UDim2.new(1, -47.5, 0, 5)
    CloseButton.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    CloseButton.Text = "X"
    CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseButton.Font = Enum.Font.Code
    CloseButton.TextSize = 20
    CloseButton.BorderSizePixel = 1
    CloseButton.BorderColor3 = Color3.fromRGB(200, 200, 200)
    CloseButton.Parent = TitleBar
    
    local Sidebar = Instance.new("Frame")
    Sidebar.Size = UDim2.new(0, 200, 1, -50)
    Sidebar.Position = UDim2.new(0, 0, 0, 50)
    Sidebar.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    Sidebar.BorderSizePixel = 0
    Sidebar.Parent = MainFrame
    
    local SidebarStroke = Instance.new("UIStroke")
    SidebarStroke.Color = Color3.fromRGB(200, 200, 200)
    SidebarStroke.Thickness = 2
    SidebarStroke.Parent = Sidebar
    
    local SidebarLayout = Instance.new("UIListLayout")
    SidebarLayout.Padding = UDim.new(0, 3)
    SidebarLayout.Parent = Sidebar
    
    local ContentFrame = Instance.new("ScrollingFrame")
    ContentFrame.Size = UDim2.new(0, 698, 1, -50)
    ContentFrame.Position = UDim2.new(0, 202, 0, 50)
    ContentFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    ContentFrame.BorderSizePixel = 0
    ContentFrame.ScrollBarThickness = 6
    ContentFrame.ScrollBarImageColor3 = Color3.fromRGB(200, 200, 200)
    ContentFrame.Visible = false
    ContentFrame.Parent = MainFrame
    
    -- FUNÇÕES DE UI
    local function CreateCategoryButton(name, icon, callback)
        local Button = Instance.new("TextButton")
        Button.Size = UDim2.new(1, 0, 0, 50)
        Button.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
        Button.BorderSizePixel = 1
        Button.BorderColor3 = Color3.fromRGB(50, 50, 50)
        Button.Text = ""
        Button.Parent = Sidebar
        
        local Icon = Instance.new("TextLabel")
        Icon.Size = UDim2.new(0, 30, 1, 0)
        Icon.Position = UDim2.new(0, 10, 0, 0)
        Icon.BackgroundTransparency = 1
        Icon.Text = icon
        Icon.TextColor3 = Color3.fromRGB(255, 255, 255)
        Icon.Font = Enum.Font.Code
        Icon.TextSize = 20
        Icon.Parent = Button
        
        local Label = Instance.new("TextLabel")
        Label.Size = UDim2.new(1, -45, 1, 0)
        Label.Position = UDim2.new(0, 43, 0, 0)
        Label.BackgroundTransparency = 1
        Label.Text = name
        Label.TextColor3 = Color3.fromRGB(255, 255, 255)
        Label.Font = Enum.Font.Code
        Label.TextSize = 16
        Label.TextXAlignment = Enum.TextXAlignment.Left
        Label.Parent = Button
        
        Button.MouseButton1Click:Connect(function()
            ContentFrame.Visible = true
            for _, child in pairs(ContentFrame:GetChildren()) do
                if child:IsA("Frame") and child.Name == "Container" then
                    child:Destroy()
                end
            end
            if callback then
                callback()
            end
        end)
        
        return Button
    end
    
    local function CreateCheckbox(parent, text, settingPath, callback)
        local currentValue = Settings
        for _, key in ipairs(settingPath) do
            currentValue = currentValue[key]
        end
        
        local Frame = Instance.new("Frame")
        Frame.Size = UDim2.new(1, -10, 0, 35)
        Frame.BackgroundTransparency = 1
        Frame.Parent = parent
        
        local Checkbox = Instance.new("TextButton")
        Checkbox.Size = UDim2.new(0, 25, 0, 25)
        Checkbox.Position = UDim2.new(0, 5, 0, 5)
        Checkbox.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
        Checkbox.BorderSizePixel = 2
        Checkbox.BorderColor3 = Color3.fromRGB(200, 200, 200)
        Checkbox.Text = currentValue and "X" or ""
        Checkbox.TextColor3 = Color3.fromRGB(0, 255, 0)
        Checkbox.Font = Enum.Font.Code
        Checkbox.TextSize = 18
        Checkbox.Parent = Frame
        
        local Label = Instance.new("TextLabel")
        Label.Size = UDim2.new(1, -35, 1, 0)
        Label.Position = UDim2.new(0, 35, 0, 0)
        Label.BackgroundTransparency = 1
        Label.Text = text
        Label.TextColor3 = Color3.fromRGB(255, 255, 255)
        Label.Font = Enum.Font.Code
        Label.TextSize = 14
        Label.TextXAlignment = Enum.TextXAlignment.Left
        Label.Parent = Frame
        
        Checkbox.MouseButton1Click:Connect(function()
            local setting = Settings
            for i = 1, #settingPath - 1 do
                setting = setting[settingPath[i]]
            end
            setting[settingPath[#settingPath]] = not setting[settingPath[#settingPath]]
            
            local newValue = setting[settingPath[#settingPath]]
            Checkbox.Text = newValue and "X" or ""
            
            if callback then
                callback(newValue)
            end
        end)
        
        return Frame
    end
    
    local function CreateSlider(parent, text, min, max, settingPath, callback)
        local currentValue = Settings
        for _, key in ipairs(settingPath) do
            currentValue = currentValue[key]
        end
        
        local Frame = Instance.new("Frame")
        Frame.Size = UDim2.new(1, -10, 0, 55)
        Frame.BackgroundTransparency = 1
        Frame.Parent = parent
        
        local Label = Instance.new("TextLabel")
        Label.Size = UDim2.new(1, 0, 0, 20)
        Label.Position = UDim2.new(0, 5, 0, 0)
        Label.BackgroundTransparency = 1
        Label.Text = text .. ": " .. tostring(currentValue)
        Label.TextColor3 = Color3.fromRGB(255, 255, 255)
        Label.Font = Enum.Font.Code
        Label.TextSize = 14
        Label.TextXAlignment = Enum.TextXAlignment.Left
        Label.Parent = Frame
        
        local SliderBG = Instance.new("Frame")
        SliderBG.Size = UDim2.new(1, -10, 0, 8)
        SliderBG.Position = UDim2.new(0, 5, 0, 28)
        SliderBG.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        SliderBG.BorderSizePixel = 2
        SliderBG.BorderColor3 = Color3.fromRGB(200, 200, 200)
        SliderBG.Parent = Frame
        
        local SliderFill = Instance.new("Frame")
        SliderFill.Size = UDim2.new((currentValue - min) / (max - min), 0, 1, 0)
        SliderFill.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
        SliderFill.BorderSizePixel = 0
        SliderFill.Parent = SliderBG
        
        local SliderButton = Instance.new("TextButton")
        SliderButton.Size = UDim2.new(0, 16, 0, 16)
        SliderButton.Position = UDim2.new((currentValue - min) / (max - min), -8, 0.5, -8)
        SliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        SliderButton.BorderSizePixel = 2
        SliderButton.BorderColor3 = Color3.fromRGB(200, 200, 200)
        SliderButton.Text = ""
        SliderButton.Parent = SliderBG
        
        local dragging = false
        
        SliderButton.MouseButton1Down:Connect(function()
            dragging = true
        end)
        
        UserInputService.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                dragging = false
            end
        end)
        
        RunService.RenderStepped:Connect(function()
            if dragging then
                local mouse = UserInputService:GetMouseLocation()
                local relativeX = mouse.X - SliderBG.AbsolutePosition.X
                local percentage = math.clamp(relativeX / SliderBG.AbsoluteSize.X, 0, 1)
                local value = math.floor(min + (max - min) * percentage)
                
                local setting = Settings
                for i = 1, #settingPath - 1 do
                    setting = setting[settingPath[i]]
                end
                setting[settingPath[#settingPath]] = value
                
                SliderFill.Size = UDim2.new(percentage, 0, 1, 0)
                SliderButton.Position = UDim2.new(percentage, -8, 0.5, -8)
                Label.Text = text .. ": " .. tostring(value)
                
                if callback then
                    callback(value)
                end
            end
        end)
        
        return Frame
    end
    
    local function CreateDropdown(parent, text, options, settingPath, callback)
        local currentValue = Settings
        for _, key in ipairs(settingPath) do
            currentValue = currentValue[key]
        end
        
        local Frame = Instance.new("Frame")
        Frame.Size = UDim2.new(1, -10, 0, 35)
        Frame.BackgroundTransparency = 1
        Frame.Parent = parent
        
        local Label = Instance.new("TextLabel")
        Label.Size = UDim2.new(0.5, 0, 1, 0)
        Label.Position = UDim2.new(0, 5, 0, 0)
        Label.BackgroundTransparency = 1
        Label.Text = text
        Label.TextColor3 = Color3.fromRGB(255, 255, 255)
        Label.Font = Enum.Font.Code
        Label.TextSize = 14
        Label.TextXAlignment = Enum.TextXAlignment.Left
        Label.Parent = Frame
        
        local DropButton = Instance.new("TextButton")
        DropButton.Size = UDim2.new(0.48, 0, 0, 28)
        DropButton.Position = UDim2.new(0.52, 0, 0, 3.5)
        DropButton.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
        DropButton.BorderSizePixel = 2
        DropButton.BorderColor3 = Color3.fromRGB(200, 200, 200)
        DropButton.Text = currentValue
        DropButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        DropButton.Font = Enum.Font.Code
        DropButton.TextSize = 12
        DropButton.Parent = Frame
        
        local OptionsFrame = Instance.new("Frame")
        OptionsFrame.Size = UDim2.new(0.48, 0, 0, #options * 30)
        OptionsFrame.Position = UDim2.new(0.52, 0, 1, 3)
        OptionsFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
        OptionsFrame.BorderSizePixel = 2
        OptionsFrame.BorderColor3 = Color3.fromRGB(200, 200, 200)
        OptionsFrame.Visible = false
        OptionsFrame.ZIndex = 10
        OptionsFrame.Parent = Frame
        
        local OptLayout = Instance.new("UIListLayout")
        OptLayout.Parent = OptionsFrame
        
        for _, option in ipairs(options) do
            local OptButton = Instance.new("TextButton")
            OptButton.Size = UDim2.new(1, 0, 0, 28)
            OptButton.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
            OptButton.BorderSizePixel = 0
            OptButton.Text = option
            OptButton.TextColor3 = Color3.fromRGB(255, 255, 255)
            OptButton.Font = Enum.Font.Code
            OptButton.TextSize = 12
            OptButton.Parent = OptionsFrame
            
            OptButton.MouseButton1Click:Connect(function()
                local setting = Settings
                for i = 1, #settingPath - 1 do
                    setting = setting[settingPath[i]]
                end
                setting[settingPath[#settingPath]] = option
                
                DropButton.Text = option
                OptionsFrame.Visible = false
                if callback then
                    callback(option)
                end
            end)
        end
        
        DropButton.MouseButton1Click:Connect(function()
            OptionsFrame.Visible = not OptionsFrame.Visible
        end)
        
        return Frame
    end
    
    local function CreateHSLSlider(parent, text, min, max, settingPath, callback)
        local currentValue = Settings
        for _, key in ipairs(settingPath) do
            currentValue = currentValue[key]
        end
        
        local Frame = Instance.new("Frame")
        Frame.Size = UDim2.new(1, -10, 0, 55)
        Frame.BackgroundTransparency = 1
        Frame.Parent = parent
        
        local Label = Instance.new("TextLabel")
        Label.Size = UDim2.new(1, 0, 0, 20)
        Label.Position = UDim2.new(0, 5, 0, 0)
        Label.BackgroundTransparency = 1
        Label.Text = text .. ": " .. tostring(currentValue)
        Label.TextColor3 = Color3.fromRGB(255, 255, 255)
        Label.Font = Enum.Font.Code
        Label.TextSize = 13
        Label.TextXAlignment = Enum.TextXAlignment.Left
        Label.Parent = Frame
        
        local SliderBG = Instance.new("Frame")
        SliderBG.Size = UDim2.new(1, -10, 0, 8)
        SliderBG.Position = UDim2.new(0, 5, 0, 28)
        SliderBG.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        SliderBG.BorderSizePixel = 2
        SliderBG.BorderColor3 = Color3.fromRGB(200, 200, 200)
        SliderBG.Parent = Frame
        
        local SliderFill = Instance.new("Frame")
        SliderFill.Size = UDim2.new((currentValue - min) / (max - min), 0, 1, 0)
        SliderFill.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
        SliderFill.BorderSizePixel = 0
        SliderFill.Parent = SliderBG
        
        local SliderButton = Instance.new("TextButton")
        SliderButton.Size = UDim2.new(0, 16, 0, 16)
        SliderButton.Position = UDim2.new((currentValue - min) / (max - min), -8, 0.5, -8)
        SliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        SliderButton.BorderSizePixel = 2
        SliderButton.BorderColor3 = Color3.fromRGB(200, 200, 200)
        SliderButton.Text = ""
        SliderButton.Parent = SliderBG
        
        local dragging = false
        
        SliderButton.MouseButton1Down:Connect(function()
            dragging = true
        end)
        
        UserInputService.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                dragging = false
            end
        end)
        
        RunService.RenderStepped:Connect(function()
            if dragging then
                local mouse = UserInputService:GetMouseLocation()
                local relativeX = mouse.X - SliderBG.AbsolutePosition.X
                local percentage = math.clamp(relativeX / SliderBG.AbsoluteSize.X, 0, 1)
                local value = math.floor(min + (max - min) * percentage)
                
                local setting = Settings
                for i = 1, #settingPath - 1 do
                    setting = setting[settingPath[i]]
                end
                setting[settingPath[#settingPath]] = value
                
                SliderFill.Size = UDim2.new(percentage, 0, 1, 0)
                SliderButton.Position = UDim2.new(percentage, -8, 0.5, -8)
                Label.Text = text .. ": " .. tostring(value)
                
                if callback then
                    callback(value)
                end
            end
        end)
        
        return Frame
    end
    
    -- ===== CATEGORIAS =====
    
    -- COMBAT (mesmo código do anterior)
    CreateCategoryButton("Combat", "⚔", function()
        local Container = Instance.new("Frame")
        Container.Name = "Container"
        Container.Size = UDim2.new(1, 0, 0, 0)
        Container.BackgroundTransparency = 1
        Container.Parent = ContentFrame
        
        local Layout = Instance.new("UIListLayout")
        Layout.Padding = UDim.new(0, 8)
        Layout.Parent = Container
        
        local Padding = Instance.new("UIPadding")
        Padding.PaddingTop = UDim.new(0, 15)
        Padding.PaddingLeft = UDim.new(0, 10)
        Padding.Parent = Container
        
        CreateCheckbox(Container, "Aimbot", {"Combat", "Aimbot"}, function(state) end)
        CreateDropdown(Container, "Type Of Aimbot", {"Ao mirar", "ToggleKey"}, {"Combat", "AimbotType"}, function(option) end)
        CreateCheckbox(Container, "FOV", {"Combat", "FOV"}, function(state)
            if FOVCircle then
                FOVCircle.Visible = state
            end
        end)
        CreateSlider(Container, "FOV Size", 50, 500, {"Combat", "FOVSize"}, function(value)
            if FOVCircle then
                FOVCircle.Radius = value
            end
        end)
        CreateCheckbox(Container, "Hitbox", {"Combat", "Hitbox"}, function(state) end)
        CreateSlider(Container, "Hitbox Size", 5, 50, {"Combat", "HitboxSize"}, function(value) end)
        CreateCheckbox(Container, "No Recoil", {"Combat", "NoRecoil"}, function(state) end)
        CreateCheckbox(Container, "No Hit", {"Combat", "NoHit"}, function(state)
            if state then
                if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                    if not NoHitPart then
                        NoHitPart = Instance.new("Part")
                        NoHitPart.Size = Vector3.new(10, 10, 10)
                        NoHitPart.Transparency = 1
                        NoHitPart.CanCollide = true
                        NoHitPart.Anchored = false
                        NoHitPart.CFrame = LocalPlayer.Character.HumanoidRootPart.CFrame
                        NoHitPart.Parent = workspace
                        
                        local Weld = Instance.new("Weld")
                        Weld.Part0 = NoHitPart
                        Weld.Part1 = LocalPlayer.Character.HumanoidRootPart
                        Weld.C0 = CFrame.new(0, 0, 0)
                        Weld.Parent = NoHitPart
                    end
                end
            else
                if NoHitPart then
                    NoHitPart:Destroy()
                    NoHitPart = nil
                end
            end
        end)
        
        Container.Size = UDim2.new(1, 0, 0, Layout.AbsoluteContentSize.Y)
        ContentFrame.CanvasSize = UDim2.new(0, 0, 0, Layout.AbsoluteContentSize.Y + 30)
    end)
    
    -- ESP (mesmo código do anterior)
    CreateCategoryButton("ESP", "👁", function()
        local Container = Instance.new("Frame")
        Container.Name = "Container"
        Container.Size = UDim2.new(1, 0, 0, 0)
        Container.BackgroundTransparency = 1
        Container.Parent = ContentFrame
        
        local Layout = Instance.new("UIListLayout")
        Layout.Padding = UDim.new(0, 8)
        Layout.Parent = Container
        
        local Padding = Instance.new("UIPadding")
        Padding.PaddingTop = UDim.new(0, 15)
        Padding.PaddingLeft = UDim.new(0, 10)
        Padding.Parent = Container
        
        CreateCheckbox(Container, "ESP", {"ESP", "Enabled"}, function(state) end)
        CreateCheckbox(Container, "ESP Team", {"ESP", "Team"}, function(state) end)
        CreateCheckbox(Container, "Chams", {"ESP", "Chams"}, function(state) end)
        CreateCheckbox(Container, "Chams Team", {"ESP", "ChamsTeam"}, function(state) end)
        CreateCheckbox(Container, "ESP Health", {"ESP", "Health"}, function(state) end)
        CreateCheckbox(Container, "ESP Name", {"ESP", "Name"}, function(state) end)
        CreateCheckbox(Container, "ESP Meters", {"ESP", "Meters"}, function(state) end)
        CreateCheckbox(Container, "ESP Line", {"ESP", "Line"}, function(state) end)
        CreateDropdown(Container, "Place Of Line", {"Top", "Low"}, {"ESP", "LinePlacement"}, function(option) end)
        CreateCheckbox(Container, "ESP Box", {"ESP", "Box"}, function(state) end)
        
        Container.Size = UDim2.new(1, 0, 0, Layout.AbsoluteContentSize.Y)
        ContentFrame.CanvasSize = UDim2.new(0, 0, 0, Layout.AbsoluteContentSize.Y + 30)
    end)
    
    -- COLORS COM HSL CORRETO
    CreateCategoryButton("Colors", "🎨", function()
        local Container = Instance.new("Frame")
        Container.Name = "Container"
        Container.Size = UDim2.new(1, 0, 0, 0)
        Container.BackgroundTransparency = 1
        Container.Parent = ContentFrame
        
        local Layout = Instance.new("UIListLayout")
        Layout.Padding = UDim.new(0, 8)
        Layout.Parent = Container
        
        local Padding = Instance.new("UIPadding")
        Padding.PaddingTop = UDim.new(0, 15)
        Padding.PaddingLeft = UDim.new(0, 10)
        Padding.Parent = Container
        
        -- Geral Color Of Panel
        local GeneralTitle = Instance.new("TextLabel")
        GeneralTitle.Size = UDim2.new(1, -10, 0, 25)
        GeneralTitle.BackgroundTransparency = 1
        GeneralTitle.Text = "Geral Color Of Panel"
        GeneralTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
        GeneralTitle.Font = Enum.Font.Code
        GeneralTitle.TextSize = 16
        GeneralTitle.TextXAlignment = Enum.TextXAlignment.Left
        GeneralTitle.Parent = Container
        
        local GeneralPreview = Instance.new("Frame")
        GeneralPreview.Size = UDim2.new(0, 40, 0, 40)
        GeneralPreview.Position = UDim2.new(1, -50, 0, 0)
        GeneralPreview.BackgroundColor3 = Settings.Colors.General
        GeneralPreview.BorderSizePixel = 2
        GeneralPreview.BorderColor3 = Color3.fromRGB(200, 200, 200)
        GeneralPreview.Parent = GeneralTitle
        
        local GeneralHex = Instance.new("TextLabel")
        GeneralHex.Size = UDim2.new(1, -10, 0, 20)
        GeneralHex.BackgroundTransparency = 1
        GeneralHex.Text = "#FFFFFF"
        GeneralHex.TextColor3 = Color3.fromRGB(200, 200, 200)
        GeneralHex.Font = Enum.Font.Code
        GeneralHex.TextSize = 12
        GeneralHex.TextXAlignment = Enum.TextXAlignment.Left
        GeneralHex.Parent = Container
        
        CreateHSLSlider(Container, "Hue", 0, 360, {"Colors", "GeneralH"}, function(value)
            local color = HSLToRGB(value, Settings.Colors.GeneralS, Settings.Colors.GeneralL)
            Settings.Colors.General = color
            GeneralPreview.BackgroundColor3 = color
            GeneralHex.Text = string.format("#%02X%02X%02X", math.floor(color.R * 255), math.floor(color.G * 255), math.floor(color.B * 255))
            ApplyGeneralColor(color)
        end)
        
        CreateHSLSlider(Container, "Saturation", 0, 100, {"Colors", "GeneralS"}, function(value)
            local color = HSLToRGB(Settings.Colors.GeneralH, value, Settings.Colors.GeneralL)
            Settings.Colors.General = color
            GeneralPreview.BackgroundColor3 = color
            GeneralHex.Text = string.format("#%02X%02X%02X", math.floor(color.R * 255), math.floor(color.G * 255), math.floor(color.B * 255))
            ApplyGeneralColor(color)
        end)
        
        CreateHSLSlider(Container, "Lightness", 0, 100, {"Colors", "GeneralL"}, function(value)
            local color = HSLToRGB(Settings.Colors.GeneralH, Settings.Colors.GeneralS, value)
            Settings.Colors.General = color
            GeneralPreview.BackgroundColor3 = color
            GeneralHex.Text = string.format("#%02X%02X%02X", math.floor(color.R * 255), math.floor(color.G * 255), math.floor(color.B * 255))
            ApplyGeneralColor(color)
        end)
        
        -- Separator
        local Sep = Instance.new("Frame")
        Sep.Size = UDim2.new(1, -10, 0, 2)
        Sep.BackgroundColor3 = Color3.fromRGB(200, 200, 200)
        Sep.BorderSizePixel = 0
        Sep.Parent = Container
        
        -- Panel Color Only
        local PanelTitle = Instance.new("TextLabel")
        PanelTitle.Size = UDim2.new(1, -10, 0, 25)
        PanelTitle.BackgroundTransparency = 1
        PanelTitle.Text = "Panel Color Only"
        PanelTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
        PanelTitle.Font = Enum.Font.Code
        PanelTitle.TextSize = 16
        PanelTitle.TextXAlignment = Enum.TextXAlignment.Left
        PanelTitle.Parent = Container
        
        local PanelPreview = Instance.new("Frame")
        PanelPreview.Size = UDim2.new(0, 40, 0, 40)
        PanelPreview.Position = UDim2.new(1, -50, 0, 0)
        PanelPreview.BackgroundColor3 = Settings.Colors.PanelOnly
        PanelPreview.BorderSizePixel = 2
        PanelPreview.BorderColor3 = Color3.fromRGB(200, 200, 200)
        PanelPreview.Parent = PanelTitle
        
        local PanelHex = Instance.new("TextLabel")
        PanelHex.Size = UDim2.new(1, -10, 0, 20)
        PanelHex.BackgroundTransparency = 1
        PanelHex.Text = "#FFFFFF"
        PanelHex.TextColor3 = Color3.fromRGB(200, 200, 200)
        PanelHex.Font = Enum.Font.Code
        PanelHex.TextSize = 12
        PanelHex.TextXAlignment = Enum.TextXAlignment.Left
        PanelHex.Parent = Container
        
        CreateHSLSlider(Container, "Hue", 0, 360, {"Colors", "PanelH"}, function(value)
            local color = HSLToRGB(value, Settings.Colors.PanelS, Settings.Colors.PanelL)
            Settings.Colors.PanelOnly = color
            PanelPreview.BackgroundColor3 = color
            PanelHex.Text = string.format("#%02X%02X%02X", math.floor(color.R * 255), math.floor(color.G * 255), math.floor(color.B * 255))
            ApplyPanelOnlyColor(color)
        end)
        
        CreateHSLSlider(Container, "Saturation", 0, 100, {"Colors", "PanelS"}, function(value)
            local color = HSLToRGB(Settings.Colors.PanelH, value, Settings.Colors.PanelL)
            Settings.Colors.PanelOnly = color
            PanelPreview.BackgroundColor3 = color
            PanelHex.Text = string.format("#%02X%02X%02X", math.floor(color.R * 255), math.floor(color.G * 255), math.floor(color.B * 255))
            ApplyPanelOnlyColor(color)
        end)
        
        CreateHSLSlider(Container, "Lightness", 0, 100, {"Colors", "PanelL"}, function(value)
            local color = HSLToRGB(Settings.Colors.PanelH, Settings.Colors.PanelS, value)
            Settings.Colors.PanelOnly = color
            PanelPreview.BackgroundColor3 = color
            PanelHex.Text = string.format("#%02X%02X%02X", math.floor(color.R * 255), math.floor(color.G * 255), math.floor(color.B * 255))
            ApplyPanelOnlyColor(color)
        end)
        
        Container.Size = UDim2.new(1, 0, 0, Layout.AbsoluteContentSize.Y)
        ContentFrame.CanvasSize = UDim2.new(0, 0, 0, Layout.AbsoluteContentSize.Y + 30)
    end)
    
    -- VARIOUS (mesmo código do anterior)
    CreateCategoryButton("Various", "📦", function()
        local Container = Instance.new("Frame")
        Container.Name = "Container"
        Container.Size = UDim2.new(1, 0, 0, 0)
        Container.BackgroundTransparency = 1
        Container.Parent = ContentFrame
        
        local Layout = Instance.new("UIListLayout")
        Layout.Padding = UDim.new(0, 8)
        Layout.Parent = Container
        
        local Padding = Instance.new("UIPadding")
        Padding.PaddingTop = UDim.new(0, 15)
        Padding.PaddingLeft = UDim.new(0, 10)
        Padding.Parent = Container
        
        CreateCheckbox(Container, "No Clip", {"Various", "NoClip"}, function(state)
            if state then
                NoClipConnection = RunService.Stepped:Connect(function()
                    if Settings.Various.NoClip and LocalPlayer.Character then
                        for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
                            if part:IsA("BasePart") then
                                part.CanCollide = false
                            end
                        end
                    end
                end)
            else
                if NoClipConnection then
                    NoClipConnection:Disconnect()
                    NoClipConnection = nil
                end
            end
        end)
        
        CreateSlider(Container, "WalkSpeed", 16, 200, {"Various", "WalkSpeed"}, function(value)
            WalkSpeedConnection = RunService.Heartbeat:Connect(function()
                if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
                    LocalPlayer.Character.Humanoid.WalkSpeed = Settings.Various.WalkSpeed
                end
            end)
        end)
        
        CreateSlider(Container, "JumpPower", 50, 200, {"Various", "JumpPower"}, function(value)
            JumpPowerConnection = RunService.Heartbeat:Connect(function()
                if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
                    LocalPlayer.Character.Humanoid.JumpPower = Settings.Various.JumpPower
                end
            end)
        end)
        
        CreateCheckbox(Container, "Fly", {"Various", "Fly"}, function(state)
            ToggleFly(state)
        end)
        
        CreateSlider(Container, "Fly Speed", 10, 200, {"Various", "FlySpeed"}, function(value) end)
        
        CreateCheckbox(Container, "Macro", {"Various", "Macro"}, function(state)
            ToggleMacro(state)
        end)
        
        CreateSlider(Container, "Macro Speed", 0.01, 1, {"Various", "MacroSpeed"}, function(value) end)
        
        Container.Size = UDim2.new(1, 0, 0, Layout.AbsoluteContentSize.Y)
        ContentFrame.CanvasSize = UDim2.new(0, 0, 0, Layout.AbsoluteContentSize.Y + 30)
    end)
    
    -- INFO (mesmo código do anterior)
    CreateCategoryButton("Info", "ℹ", function()
        local Container = Instance.new("Frame")
        Container.Name = "Container"
        Container.Size = UDim2.new(1, 0, 0, 0)
        Container.BackgroundTransparency = 1
        Container.Parent = ContentFrame
        
        local Layout = Instance.new("UIListLayout")
        Layout.Padding = UDim.new(0, 12)
        Layout.Parent = Container
        
        local Padding = Instance.new("UIPadding")
        Padding.PaddingTop = UDim.new(0, 15)
        Padding.PaddingLeft = UDim.new(0, 15)
        Padding.Parent = Container
        
        local Thumbnail = Instance.new("ImageLabel")
        Thumbnail.Size = UDim2.new(0, 150, 0, 150)
        Thumbnail.Position = UDim2.new(0.5, -75, 0, 0)
        Thumbnail.BackgroundTransparency = 1
        Thumbnail.Image = "https://www.roblox.com/headshot-thumbnail/image?userId=" .. LocalPlayer.UserId .. "&width=150&height=150&format=png"
        Thumbnail.Parent = Container
        
        local Username = Instance.new("TextLabel")
        Username.Size = UDim2.new(1, -20, 0, 25)
        Username.BackgroundTransparency = 1
        Username.Text = "Username: " .. LocalPlayer.Name
        Username.TextColor3 = Color3.fromRGB(255, 255, 255)
        Username.Font = Enum.Font.Code
        Username.TextSize = 14
        Username.TextXAlignment = Enum.TextXAlignment.Left
        Username.Parent = Container
        
        local DisplayName = Instance.new("TextLabel")
        DisplayName.Size = UDim2.new(1, -20, 0, 25)
        DisplayName.BackgroundTransparency = 1
        DisplayName.Text = "Display: " .. LocalPlayer.DisplayName
        DisplayName.TextColor3 = Color3.fromRGB(255, 255, 255)
        DisplayName.Font = Enum.Font.Code
        DisplayName.TextSize = 14
        DisplayName.TextXAlignment = Enum.TextXAlignment.Left
        DisplayName.Parent = Container
        
        local UserID = Instance.new("TextLabel")
        UserID.Size = UDim2.new(1, -20, 0, 25)
        UserID.BackgroundTransparency = 1
        UserID.Text = "UserID: " .. LocalPlayer.UserId
        UserID.TextColor3 = Color3.fromRGB(255, 255, 255)
        UserID.Font = Enum.Font.Code
        UserID.TextSize = 14
        UserID.TextXAlignment = Enum.TextXAlignment.Left
        UserID.Parent = Container
        
        local SessionTime = Instance.new("TextLabel")
        SessionTime.Size = UDim2.new(1, -20, 0, 25)
        SessionTime.BackgroundTransparency = 1
        SessionTime.Text = "Time Using: 00:00:00"
        SessionTime.TextColor3 = Color3.fromRGB(255, 255, 255)
        SessionTime.Font = Enum.Font.Code
        SessionTime.TextSize = 14
        SessionTime.TextXAlignment = Enum.TextXAlignment.Left
        SessionTime.Parent = Container
        
        local Status = Instance.new("TextLabel")
        Status.Size = UDim2.new(1, -20, 0, 25)
        Status.BackgroundTransparency = 1
        Status.Text = "Status: Active"
        Status.TextColor3 = Color3.fromRGB(0, 255, 0)
        Status.Font = Enum.Font.Code
        Status.TextSize = 14
        Status.TextXAlignment = Enum.TextXAlignment.Left
        Status.Parent = Container
        
        local Separator = Instance.new("Frame")
        Separator.Size = UDim2.new(1, -20, 0, 2)
        Separator.BackgroundColor3 = Color3.fromRGB(200, 200, 200)
        Separator.BorderSizePixel = 0
        Separator.Parent = Container
        
        local Made = Instance.new("TextLabel")
        Made.Size = UDim2.new(1, -20, 0, 25)
        Made.BackgroundTransparency = 1
        Made.Text = "Made by Barboza;"
        Made.TextColor3 = Color3.fromRGB(255, 255, 255)
        Made.Font = Enum.Font.Code
        Made.TextSize = 14
        Made.TextXAlignment = Enum.TextXAlignment.Left
        Made.Parent = Container
        
        task.spawn(function()
            while task.wait(1) do
                if SessionTime and SessionTime.Parent then
                    Settings.SessionTime = Settings.SessionTime + 1
                    local h = math.floor(Settings.SessionTime / 3600)
                    local m = math.floor((Settings.SessionTime % 3600) / 60)
                    local s = Settings.SessionTime % 60
                    SessionTime.Text = string.format("Time Using: %02d:%02d:%02d", h, m, s)
                else
                    break
                end
            end
        end)
        
        Container.Size = UDim2.new(1, 0, 0, Layout.AbsoluteContentSize.Y + 170)
        ContentFrame.CanvasSize = UDim2.new(0, 0, 0, Layout.AbsoluteContentSize.Y + 200)
    end)
    
    -- HANDLERS
    CloseButton.MouseButton1Click:Connect(function()
        MainFrame.Visible = false
    end)
    
    local minimized = false
    MinButton.MouseButton1Click:Connect(function()
        minimized = not minimized
        if minimized then
            MainFrame.Size = UDim2.new(0, 900, 0, 50)
        else
            MainFrame.Size = UDim2.new(0, 900, 0, 600)
        end
    end)
    
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    return ScreenGui
end

-- ============================================
-- INICIALIZAÇÃO
-- ============================================

-- Criar FOV Circle
pcall(function()
    FOVCircle = Drawing.new("Circle")
    FOVCircle.Thickness = 1
    FOVCircle.NumSides = 50
    FOVCircle.Radius = Settings.Combat.FOVSize
    FOVCircle.Filled = false
    FOVCircle.Visible = false
    FOVCircle.Color = Color3.new(1, 1, 1)
    FOVCircle.Transparency = 1
    
    RunService.RenderStepped:Connect(function()
        if FOVCircle then
            FOVCircle.Visible = Settings.Combat.FOV
            FOVCircle.Radius = Settings.Combat.FOVSize
            FOVCircle.Position = Vector2.new(Mouse.X, Mouse.Y + 36)
        end
    end)
end)

-- Criar ESP para todos
for _, player in pairs(Players:GetPlayers()) do
    CreateESP(player)
end

Players.PlayerAdded:Connect(function(player)
    CreateESP(player)
end)

Players.PlayerRemoving:Connect(function(player)
    if ESPObjects[player] then
        for _, obj in pairs(ESPObjects[player]) do
            if typeof(obj) == "Instance" then
                obj:Destroy()
            elseif typeof(obj) == "table" and obj.Remove then
                obj:Remove()
            end
        end
        ESPObjects[player] = nil
    end
end)

-- Loops principais
RunService.Heartbeat:Connect(function()
    safeCall(UpdateAimbot)
    safeCall(ApplyHitbox)
    safeCall(ApplyNoRecoil)
end)

-- Criar painel
GUI = CreatePanel()

-- Toggle Shift + Z
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.Z and UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
        local MainFrame = GUI:FindFirstChild("MainFrame")
        if MainFrame then
            MainFrame.Visible = not MainFrame.Visible
        end
    end
end)

print("╔══════════════════════════════════════════╗")
print("║  PANEL PVP CLASSIC MENU 1.0             ║")
print("║  ESP HEALTH CORRIGIDA + COLORS FIXO     ║")
print("╠══════════════════════════════════════════╣")
print("║  SHIFT + Z: Toggle Panel                ║")
print("║  Made by Barboza                         ║")
print("╚══════════════════════════════════════════╝")
