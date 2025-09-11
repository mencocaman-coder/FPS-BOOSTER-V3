-- FPS Booster Ultimate v1.0
-- Créditos: Sua mãe, @PepsiMannumero1 & @Lr-Scripts
-- Melhorias: qualidade mínima, streaming, delay no lobby, sem acumular listeners

local Players           = game:GetService("Players")
local RunService        = game:GetService("RunService")
local Workspace         = game:GetService("Workspace")
local Lighting          = game:GetService("Lighting")
local CollectionService = game:GetService("CollectionService")

local player    = Players.LocalPlayer
local PlayerGui = player:WaitForChild("PlayerGui")

-- Configurações gráficas mínimas
pcall(function()
    settings().Rendering.QualityLevel            = 1
    settings().Rendering.MeshPartDetailLevel     = Enum.MeshPartDetailLevel.Low
    settings().Rendering.EagerBulkExecution       = false
    settings().Rendering.InterpolationThrottling = true
end)

-- Ativa streaming para reduzir carga do lobby
Workspace.StreamingEnabled = true

-- Parâmetros configuráveis
local BATCH_SIZE    = 20
local FPS_THRESHOLD = 75
local SAMPLE_SIZE   = 30
local LOBBY_DELAY   = 10     -- segundos para evitar otimização imediata no lobby
local creditTime    = 1.4

-- Créditos na tela (uma única vez)
local credits = {
    "FPS Booster ativado!",
    "@PepsiMannumero1 & @Lr-Scripts",
    "Sua mãe"
}

-- Estado interno
local optimizeQueue = {}
local fpsBuffer     = {}
local boosterActive = false
local inLobby       = true

-- Função para exibir um crédito
local function showCredit(text)
    local gui = Instance.new("ScreenGui")
    gui.Name             = "FPSBoosterCredits"
    gui.ResetOnSpawn     = false
    gui.Parent           = PlayerGui

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

    task.delay(creditTime, function()
        gui:Destroy()
    end)
end

-- Exibe os créditos em sequência
task.spawn(function()
    for _, text in ipairs(credits) do
        showCredit(text)
        task.wait(creditTime)
    end
end)

-- Ajustes de iluminação e pós-processamento (apenas uma vez)
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

-- Função que aplica otimizações a um objeto
local function optimize(obj)
    if CollectionService:HasTag(obj, "FB_Optimized") then return end
    CollectionService:AddTag(obj, "FB_Optimized")

    if obj:IsA("ParticleEmitter") or obj:IsA("Trail")
    or obj:IsA("Smoke") or obj:IsA("Sparkles")
    or obj:IsA("Fire") or obj:IsA("Explosion") then
        pcall(function()
            obj.Enabled  = false
            obj.Rate     = 0
            obj.Lifetime = NumberRange.new(0)
        end)
    elseif obj:IsA("BasePart") then
        pcall(function()
            obj.Material    = Enum.Material.SmoothPlastic
            obj.CastShadow  = false
            obj.Reflectance = 0
        end)
    elseif obj:IsA("PointLight")
       or obj:IsA("SpotLight")
       or obj:IsA("SurfaceLight") then
        pcall(function()
            obj.Brightness = math.clamp(obj.Brightness or 1, 0.3, 1)
            obj.Range      = math.clamp(obj.Range      or 8, 6, 16)
        end)
    end
end

-- Coloca objeto na fila de otimização
local function enqueue(obj)
    optimizeQueue[#optimizeQueue + 1] = obj
end

-- Reseta apenas as variáveis de estado (respawn e início de lobby)
local function resetState()
    optimizeQueue = {}
    fpsBuffer     = {}
    boosterActive = false
    inLobby       = true
    task.delay(LOBBY_DELAY, function()
        inLobby = false
    end)
end

-- Conexão única ao Heartbeat para processar FPS e otimizações
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

-- Conexão única a DescendantAdded para enfileirar novos objetos
Workspace.DescendantAdded:Connect(function(obj)
    if boosterActive then
        enqueue(obj)
    end
end)

-- Reseta estado no respawn do personagem
player.CharacterAdded:Connect(resetState)
resetState()
