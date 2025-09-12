-- FPS Booster v2.1 – Efeitos Extras Desativados
-- Créditos: Sua mãe, @PepsiMannumero1 & @Lr-Scripts

local Players           = game:GetService("Players")
local RunService        = game:GetService("RunService")
local Workspace         = game:GetService("Workspace")
local Lighting          = game:GetService("Lighting")
local CollectionService = game:GetService("CollectionService")

local player    = Players.LocalPlayer
local PlayerGui = player:WaitForChild("PlayerGui")

-- FORÇAR QUALIDADE GRÁFICA MÍNIMA
pcall(function()
    settings().Rendering.QualityLevel            = 1
    settings().Rendering.MeshPartDetailLevel     = Enum.MeshPartDetailLevel.Low
    settings().Rendering.EagerBulkExecution       = false
    settings().Rendering.InterpolationThrottling = true
end)

-- ATIVAR STREAMING PARA ALÍVIO NO LOBBY
Workspace.StreamingEnabled = true

-- OTIMIZAÇÃO DO TERRAIN (ÁGUA)
local terrain = Workspace:FindFirstChildOfClass("Terrain")
if terrain then
    terrain.WaterWaveSize     = 0
    terrain.WaterWaveSpeed    = 0
    terrain.WaterReflectance  = 0
    terrain.WaterTransparency = 1
end

-- AJUSTES DE ILUMINAÇÃO E PÓS-PROCESSAMENTO
Lighting.FogStart      = 0
Lighting.FogEnd        = 1e5
Lighting.GlobalShadows = false

for _, eff in ipairs(Lighting:GetDescendants()) do
    if eff:IsA("PostEffect") then
        eff.Enabled = false
        eff:GetPropertyChangedSignal("Enabled"):Connect(function()
            eff.Enabled = false
        end)
    end
end

Lighting.DescendantAdded:Connect(function(eff)
    if eff:IsA("PostEffect") then
        eff.Enabled = false
    end
end)

-- PARÂMETROS
local BATCH_SIZE    = 20
local FPS_THRESHOLD = 75
local SAMPLE_SIZE   = 30
local LOBBY_DELAY   = 10    -- segundos antes de iniciar otimização
local CREDIT_TIME   = 1.4

-- CRÉDITOS (MOSTRAR UMA VEZ)
local credits = {
    "FPS Booster ativado!",
    "@PepsiMannumero1 & @Lr-Scripts no YouTube",
    "Sua mãe"
}

local function showCredit(text)
    local gui = Instance.new("ScreenGui")
    gui.Name         = "FPSBoosterCredits"
    gui.ResetOnSpawn = false
    gui.Parent       = PlayerGui

    local frame = Instance.new("Frame", gui)
    frame.Size            = UDim2.new(0.4, 0, 0.1, 0)
    frame.Position        = UDim2.new(0.5, 0, 0.8, 0)
    frame.AnchorPoint     = Vector2.new(0.5, 0.5)
    frame.BackgroundColor3= Color3.fromRGB(30,30,30)
    frame.BorderSizePixel = 0
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0.2, 0)

    local label = Instance.new("TextLabel", frame)
    label.Size                   = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Font                   = Enum.Font.GothamBold
    label.TextScaled             = true
    label.TextWrapped            = true
    label.TextColor3             = Color3.fromRGB(0,174,255)
    label.Text                   = text

    task.delay(CREDIT_TIME, function()
        gui:Destroy()
    end)
end

task.spawn(function()
    for _, txt in ipairs(credits) do
        showCredit(txt)
        task.wait(CREDIT_TIME)
    end
end)

-- ESTADO INTERNO
local optimizeQueue = {}
local fpsBuffer     = {}
local boosterActive = false
local inLobby       = true

-- FUNÇÃO DE OTIMIZAÇÃO
local function optimize(obj)
    if CollectionService:HasTag(obj, "FB_Optimized") then
        return
    end
    CollectionService:AddTag(obj, "FB_Optimized")

    if obj:IsA("ParticleEmitter")
    or obj:IsA("Trail")
    or obj:IsA("Smoke")
    or obj:IsA("Sparkles")
    or obj:IsA("Fire")
    or obj:IsA("Explosion") then
        pcall(function()
            obj.Enabled  = false
            if obj.Rate     then obj.Rate     = 0 end
            if obj.Lifetime then obj.Lifetime = NumberRange.new(0) end
        end)

    elseif obj:IsA("Decal") or obj:IsA("Texture") then
        pcall(function()
            obj.Transparency = 1
        end)

    elseif obj:IsA("BasePart") then
        pcall(function()
            if obj.Material == Enum.Material.Water then
                obj.Transparency = 1
                obj.CastShadow  = false
            else
                obj.Material    = Enum.Material.SmoothPlastic
                obj.CastShadow  = false
                obj.Reflectance = 0
            end
        end)

    elseif obj:IsA("PointLight")
       or obj:IsA("SpotLight")
       or obj:IsA("SurfaceLight") then
        pcall(function()
            obj.Brightness = math.clamp(obj.Brightness or 1, 0.3, 1)
            obj.Range      = math.clamp(obj.Range or 8, 6, 16)
        end)
    end
end

-- ENFILEIRA OBJETO PARA OTIMIZAÇÃO
local function enqueue(obj)
    optimizeQueue[#optimizeQueue + 1] = obj
end

-- DESATIVA IMEDIATO DE EFEITOS AO CRIAREM-SE
Workspace.DescendantAdded:Connect(function(obj)
    if obj:IsA("ParticleEmitter")
    or obj:IsA("Trail")
    or obj:IsA("Smoke")
    or obj:IsA("Sparkles")
    or obj:IsA("Fire")
    or obj:IsA("Explosion")
    or obj:IsA("Decal")
    or obj:IsA("Texture") then
        optimize(obj)
    elseif boosterActive then
        enqueue(obj)
    end
end)

-- PASSE INICIAL EM TUDO
for _, obj in ipairs(Workspace:GetDescendants()) do
    if obj:IsA("ParticleEmitter")
    or obj:IsA("Trail")
    or obj:IsA("Smoke")
    or obj:IsA("Sparkles")
    or obj:IsA("Fire")
    or obj:IsA("Explosion")
    or obj:IsA("Decal")
    or obj:IsA("Texture") then
        optimize(obj)
    end
end

-- RESETA ESTADO NO RESPAWN
local function resetState()
    optimizeQueue = {}
    fpsBuffer     = {}
    boosterActive = false
    inLobby       = true
    task.delay(LOBBY_DELAY, function()
        inLobby = false
    end)
end

player.CharacterAdded:Connect(resetState)
resetState()

-- MONITORA FPS E PROCESSA OTIMIZAÇÃO
RunService.Heartbeat:Connect(function(dt)
    if inLobby then return end

    fpsBuffer[#fpsBuffer + 1] = 1/dt
    if #fpsBuffer > SAMPLE_SIZE then
        table.remove(fpsBuffer, 1)
    end

    local sum = 0
    for _, v in ipairs(fpsBuffer) do sum += v end
    local avgFPS = sum / #fpsBuffer

    if not boosterActive and avgFPS < FPS_THRESHOLD then
        boosterActive = true
        for _, obj in ipairs(Workspace:GetDescendants()) do
            enqueue(obj)
        end
    end

    for i = 1, BATCH_SIZE do
        local obj = table.remove(optimizeQueue, 1)
        if not obj then break end
        optimize(obj)
    end
end)
