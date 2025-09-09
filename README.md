-- FPS Booster Automático para Roblox (local, leve e compatível com executores)
-- Ele aplica o booster sozinho ao detectar FPS baixo, sem travar e sem erros.

local ws = game:GetService("Workspace")
local lighting = game:GetService("Lighting")

-- Função para saber se é interface ou decal
local function isInterface(obj)
    local cn = obj.ClassName
    return cn == "BillboardGui" or cn == "SurfaceGui" or cn == "ScreenGui"
        or cn == "SelectionBox" or cn == "Handles" or cn == "BoxHandleAdornment"
        or cn == "Decal" or cn == "UIComponent" or cn == "Highlight"
        or cn == "TextLabel" or cn == "TextButton" or cn == "ImageLabel"
        or cn == "Frame" or cn == "ViewportFrame"
end

local function boost()
    for _, obj in ipairs(ws:GetDescendants()) do
        if not isInterface(obj) then
            if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Smoke")
            or obj:IsA("Sparkles") or obj:IsA("Fire") then
                pcall(function() obj.Enabled = false end)
            end
            if obj:IsA("Explosion") then
                pcall(function() obj.Visible = false end)
            end
            if obj:IsA("PointLight") or obj:IsA("SpotLight") or obj:IsA("SurfaceLight") then
                pcall(function() obj.Enabled = false obj.Brightness = 0 end)
            end
            if obj:IsA("BasePart") then
                pcall(function() obj.Material = Enum.Material.SmoothPlastic end)
            end
        end
    end

    for _, v in ipairs(lighting:GetChildren()) do
        if v:IsA("Sky") or v:IsA("Atmosphere") or v:IsA("Clouds")
        or v:IsA("BloomEffect") or v:IsA("SunRaysEffect") or v:IsA("ColorCorrectionEffect")
        or v:IsA("DepthOfFieldEffect") or v:IsA("BlurEffect") then
            pcall(function() v:Destroy() end)
        end
    end

    pcall(function()
        lighting.FogEnd = 1e10
        lighting.FogStart = 1e10
        lighting.Brightness = 1
        lighting.GlobalShadows = false
    end)

    local terrain = ws:FindFirstChildOfClass("Terrain")
    if terrain then
        pcall(function()
            terrain.WaterReflectance = 0
            terrain.WaterTransparency = 1
            terrain.WaterWaveSize = 0
            terrain.WaterWaveSpeed = 0
        end)
    end
end

-- Protege também objetos novos que nascerem
ws.DescendantAdded:Connect(function(obj)
    if not isInterface(obj) then
        if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Smoke")
        or obj:IsA("Sparkles") or obj:IsA("Fire") then
            pcall(function() obj.Enabled = false end)
        end
        if obj:IsA("Explosion") then
            pcall(function() obj.Visible = false end)
        end
        if obj:IsA("PointLight") or obj:IsA("SpotLight") or obj:IsA("SurfaceLight") then
            pcall(function() obj.Enabled = false obj.Brightness = 0 end)
        end
        if obj:IsA("BasePart") then
            pcall(function() obj.Material = Enum.Material.SmoothPlastic end)
        end
    end
end)

-- Função para checar FPS 
local function getFPS()
    if ws.GetRealPhysicsFPS then
        local ok, result = pcall(function() return ws:GetRealPhysicsFPS() end)
        if ok and type(result) == "number" then return result end
    end
    if stats and stats().Workspace and stats().Workspace.GetRealPhysicsFPS then
        local ok, result = pcall(function() return stats().Workspace:GetRealPhysicsFPS() end)
        if ok and type(result) == "number" then return result end
    end
    return 60 -- fallback
end

-- Loop leve e automático: aplica sempre que FPS cair
spawn(function()
    while true do
        local fps = getFPS()
        if fps < 55 then
            boost()
        end
        wait(2)
    end
end)

boost() -- aplica uma vez logo ao executar

-- Opcional: notificação simples
pcall(function()
    local plr = game:GetService("Players").LocalPlayer
    if plr then
        local pg = plr:FindFirstChildOfClass("PlayerGui")
        if pg then
            local gui = Instance.new("ScreenGui")
            gui.Name = "FPSBoosterAviso"
            gui.ResetOnSpawn = false
            gui.IgnoreGuiInset = true
            local frame = Instance.new("Frame")
            frame.Size = UDim2.new(0, 260, 0, 38)
            frame.Position = UDim2.new(1, -280, 1, -80)
            frame.BackgroundColor3 = Color3.fromRGB(40,180,40)
            frame.BackgroundTransparency = 0.18
            frame.BorderSizePixel = 0
            frame.Parent = gui
            local textoLabel = Instance.new("TextLabel")
            textoLabel.Size = UDim2.new(1, -14, 1, -14)
            textoLabel.Position = UDim2.new(0, 7, 0, 7)
            textoLabel.BackgroundTransparency = 1
            textoLabel.Text = "FPS Booster Automático ativo!"
            textoLabel.TextColor3 = Color3.new(1,1,1)
            textoLabel.TextScaled = true
            textoLabel.Font = Enum.Font.GothamBold
            textoLabel.TextStrokeTransparency = 0.5
            textoLabel.Parent = frame
            gui.Parent = pg
            spawn(function()
                wait(2.2)
                gui:Destroy()
            end)
        end
    end
end)
