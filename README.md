-- FPS Booster Automático com Créditos (@PepsiMannumero1 #Youtube)
-- Cole este script no executor como LocalScript

local Players    = game:GetService("Players")
local RunService = game:GetService("RunService")
local Lighting   = game:GetService("Lighting")
local Workspace  = game:GetService("Workspace")

local player    = Players.LocalPlayer
local PlayerGui = player:WaitForChild("PlayerGui")

-- CONFIGURAÇÃO INTERNA
local Config = {
    fpsOnThreshold   = 50,   -- abaixo disso ativa o booster
    fpsOffThreshold  = 75,   -- acima disso desativa o booster
    sampleSize       = 30,   -- frames para média móvel de FPS
    messageDuration  = 1.4,  -- duração de cada crédito na tela
    creditMessages   = {
        "FPS Booster ativado!",
        "@PepsiMannumero1  #Youtube",
        "Sua mãe"
    }
}

-- Armazena valores originais de Lighting e efeitos
local originalLighting = {
    Brightness               = Lighting.Brightness,
    GlobalShadows            = Lighting.GlobalShadows,
    OutdoorAmbient           = Lighting.OutdoorAmbient,
    Ambient                  = Lighting.Ambient,
    FogEnd                   = Lighting.FogEnd,
    FogStart                 = Lighting.FogStart,
    EnvironmentDiffuseScale  = Lighting.EnvironmentDiffuseScale,
    EnvironmentSpecularScale = Lighting.EnvironmentSpecularScale,
    Effects                  = {}
}
for _, eff in ipairs(Lighting:GetDescendants()) do
    if eff:IsA("PostEffect") then
        originalLighting.Effects[eff] = eff.Enabled
    end
end

local boosterEnabled = false
local fpsBuffer      = {}

-- Exibe apenas as mensagens de crédito no spawn
local function showCredit(text)
    local gui = Instance.new("ScreenGui")
    gui.Name         = "FPSBoosterCredit"
    gui.ResetOnSpawn = false
    gui.Parent       = PlayerGui

    local frame = Instance.new("Frame", gui)
    frame.Size               = UDim2.new(0.4,0,0.1,0)
    frame.Position           = UDim2.new(0.5,0,0.8,0)
    frame.AnchorPoint        = Vector2.new(0.5,0.5)
    frame.BackgroundColor3   = Color3.fromRGB(30,30,30)
    frame.BorderSizePixel    = 0

    local corner = Instance.new("UICorner", frame)
    corner.CornerRadius      = UDim.new(0.2, 0)

    local label = Instance.new("TextLabel", frame)
    label.Size               = UDim2.new(1,0,1,0)
    label.BackgroundTransparency = 1
    label.Font               = Enum.Font.GothamBold
    label.TextScaled         = true
    label.TextWrapped        = true
    label.TextColor3         = Color3.fromRGB(0,174,255)
    label.Text               = text

    task.delay(Config.messageDuration, function()
        gui:Destroy()
    end)
end

-- Sequência de créditos
task.spawn(function()
    for _, msg in ipairs(Config.creditMessages) do
        showCredit(msg)
        task.wait(Config.messageDuration)
    end
end)

-- Otimização de partículas, partes e luzes
local function optimizeObj(obj)
    if obj:IsA("ParticleEmitter") or obj:IsA("Trail")
    or obj:IsA("Smoke") or obj:IsA("Sparkles")
    or obj:IsA("Fire") or obj:IsA("Explosion") then
        pcall(function()
            obj.Enabled  = false
            obj.Lifetime = NumberRange.new(0)
            obj.Rate     = 0
        end)
    elseif obj:IsA("BasePart") then
        pcall(function()
            obj.Material    = Enum.Material.SmoothPlastic
            obj.CastShadow  = false
            obj.Reflectance = 0
        end)
    elseif obj:IsA("PointLight") or obj:IsA("SpotLight")
       or obj:IsA("SurfaceLight") then
        pcall(function()
            obj.Brightness = math.clamp(obj.Brightness or 1, 0.3, 1)
            obj.Range      = math.clamp(obj.Range or 8, 6, 16)
        end)
    end
end

-- Aplica ajustes de Lighting
local function applyLighting()
    for eff, _ in pairs(originalLighting.Effects) do
        pcall(function() eff.Enabled = false end)
    end
    Lighting.Brightness               = math.max(1.5, Lighting.Brightness)
    Lighting.GlobalShadows            = false
    Lighting.OutdoorAmbient           = Color3.new(0.8,0.8,0.8)
    Lighting.Ambient                  = Color3.new(0.8,0.8,0.8)
    Lighting.FogEnd                   = 1e5
    Lighting.FogStart                 = 99999
    Lighting.EnvironmentDiffuseScale  = 0
    Lighting.EnvironmentSpecularScale = 0
end

-- Restaura iluminação original
local function restoreLighting()
    local o = originalLighting
    Lighting.Brightness               = o.Brightness
    Lighting.GlobalShadows            = o.GlobalShadows
    Lighting.OutdoorAmbient           = o.OutdoorAmbient
    Lighting.Ambient                  = o.Ambient
    Lighting.FogEnd                   = o.FogEnd
    Lighting.FogStart                 = o.FogStart
    Lighting.EnvironmentDiffuseScale  = o.EnvironmentDiffuseScale
    Lighting.EnvironmentSpecularScale = o.EnvironmentSpecularScale
    for eff, enabled in pairs(o.Effects) do
        pcall(function() eff.Enabled = enabled end)
    end
end

-- Otimiza novos objetos apenas quando booster ativo
Workspace.DescendantAdded:Connect(function(obj)
    if boosterEnabled then
        optimizeObj(obj)
    end
end)

-- Loop de Heartbeat: coleta FPS e ativa/desativa automaticamente
RunService.Heartbeat:Connect(function(dt)
    table.insert(fpsBuffer, 1/dt)
    if #fpsBuffer > Config.sampleSize then
        table.remove(fpsBuffer, 1)
    end

    local sum = 0
    for _, v in ipairs(fpsBuffer) do sum += v end
    local avgFPS = sum / #fpsBuffer

    if not boosterEnabled and avgFPS < Config.fpsOnThreshold then
        boosterEnabled = true
        for _, obj in ipairs(Workspace:GetDescendants()) do
            optimizeObj(obj)
        end
        applyLighting()

    elseif boosterEnabled and avgFPS > Config.fpsOffThreshold then
        boosterEnabled = false
        restoreLighting()
    end
end)
