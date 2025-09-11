-- FPS Booster Adaptativo Sem Travamentos
-- Créditos fixos em português (“Sua mãe”)
-- (@PepsiMannumero1 & @Lr-Scripts no YouTube)

local RunService        = game:GetService("RunService")
local Workspace         = game:GetService("Workspace")
local Players           = game:GetService("Players")
local Lighting          = game:GetService("Lighting")
local CollectionService = game:GetService("CollectionService")

local player    = Players.LocalPlayer
local PlayerGui = player:WaitForChild("PlayerGui")

-- Mensagens de crédito
local credits = {
    "FPS Booster ativado!",
    "@PepsiMannumero1 & @Lr-Scripts no YouTube",
    "Sua mãe"
}
local creditTime    = 1.4
local BATCH_SIZE    = 20
local FPS_THRESHOLD = 75
local SAMPLE_SIZE   = 30

-- Estado do booster (será zerado em cada respawn)
local optimizeQueue = {}
local fpsBuffer     = {}
local boosterActive = false

-- Exibe um crédito temporário na tela
local function showCredit(txt)
    local gui = Instance.new("ScreenGui", PlayerGui)
    gui.ResetOnSpawn = false

    local frame = Instance.new("Frame", gui)
    frame.Size              = UDim2.new(0.4, 0, 0.1, 0)
    frame.Position          = UDim2.new(0.5, 0, 0.8, 0)
    frame.AnchorPoint       = Vector2.new(0.5, 0.5)
    frame.BackgroundColor3  = Color3.fromRGB(30, 30, 30)
    frame.BorderSizePixel   = 0
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0.2, 0)

    local label = Instance.new("TextLabel", frame)
    label.Size                   = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Font                   = Enum.Font.GothamBold
    label.TextScaled             = true
    label.TextWrapped            = true
    label.TextColor3             = Color3.fromRGB(0, 174, 255)
    label.Text                   = txt

    task.delay(creditTime, function()
        gui:Destroy()
    end)
end

-- Exibe todos os créditos em sequência (uma única vez)
local function playCredits()
    task.spawn(function()
        for _, txt in ipairs(credits) do
            showCredit(txt)
            task.wait(creditTime)
        end
    end)
end

-- Chama a sequência de créditos apenas no carregamento inicial
playCredits()

-- Ajustes de iluminação e pós-processamento (vai rodar só uma vez)
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

-- Aplica otimizações dependendo do tipo de objeto
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
    table.insert(optimizeQueue, obj)
end

-- Monitora FPS e processa lotes de otimização
RunService.Heartbeat:Connect(function(dt)
    -- Atualiza buffer
    table.insert(fpsBuffer, 1/dt)
    if #fpsBuffer > SAMPLE_SIZE then
        table.remove(fpsBuffer, 1)
    end

    -- Calcula média móvel
    local sum = 0
    for _, v in ipairs(fpsBuffer) do sum += v end
    local avgFPS = sum / #fpsBuffer

    -- Quando FPS cai, ativa booster e enfileira tudo
    if not boosterActive and avgFPS < FPS_THRESHOLD then
        boosterActive = true
        for _, obj in ipairs(Workspace:GetDescendants()) do
            enqueue(obj)
        end
    end

    -- Processa um lote de otimizações
    for i = 1, BATCH_SIZE do
        local obj = table.remove(optimizeQueue, 1)
        if not obj then break end
        optimize(obj)
    end
end)

-- Novos objetos entram na fila se o booster já estiver ativo
Workspace.DescendantAdded:Connect(function(obj)
    if boosterActive then
        enqueue(obj)
    end
end)

-- Função que reinicia fila, buffer e flag a cada respawn
local function resetState()
    optimizeQueue = {}
    fpsBuffer     = {}
    boosterActive = false
end

-- Conecta e executa o reset no primeiro Character
player.CharacterAdded:Connect(resetState)
if player.Character then
    resetState()
end
