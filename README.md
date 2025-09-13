-- Aniquilador de Lag v5.1 – Agresivo Total c/ UI preservada
-- Créditos: Sua mãe, @PepsiMannumero1, @Lr-Scripts & @Bing IA Scripts

local Players           = game:GetService("Players")
local RunService        = game:GetService("RunService")
local Workspace         = game:GetService("Workspace")
local Lighting          = game:GetService("Lighting")
local CollectionService = game:GetService("CollectionService")

-- parâmetros de batch dinâmico
local BATCH_MIN    = 20
local BATCH_MAX    = 200
local SAMPLE_COUNT = 30
local FPS_LOW      = 30
local FPS_HIGH     = 60

local fpsBuffer     = {}
local optimizeQueue = {}

-- exibe créditos
local function showCredits()
    local player  = Players.LocalPlayer
    local guiRoot = player and player:WaitForChild("PlayerGui", 5)
    if not guiRoot then return end
    for _, txt in ipairs({
        "Aniquilador de Lag Turbo ativado!",
        "@PepsiMannumero1 & @Lr-Scripts",
        "Sua mãe",
        "@Bing IA Scripts"
    }) do
        local gui = Instance.new("ScreenGui")
        gui.Name, gui.ResetOnSpawn, gui.Parent = "AL_Credits", false, guiRoot
        local frame = Instance.new("Frame", gui)
        frame.Size             = UDim2.new(0.4,0,0.1,0)
        frame.Position         = UDim2.new(0.5,0,0.8,0)
        frame.AnchorPoint      = Vector2.new(0.5,0.5)
        frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
        frame.BorderSizePixel  = 0
        Instance.new("UICorner", frame).CornerRadius = UDim.new(0.2,0)
        local label = Instance.new("TextLabel", frame)
        label.Size                   = UDim2.new(1,0,1,0)
        label.BackgroundTransparency = 1
        label.Font                   = Enum.Font.GothamBold
        label.TextScaled             = true
        label.TextWrapped            = true
        label.TextColor3             = Color3.fromRGB(0,174,255)
        label.Text                   = txt
        task.delay(1.4, function() if gui.Parent then gui:Destroy() end end)
        task.wait(1.4)
    end
end
task.spawn(function() pcall(showCredits) end)

-- ajusta configurações gráficas mínimas
pcall(function()
    local r = settings().Rendering
    r.QualityLevel            = 1
    r.MeshPartDetailLevel     = Enum.MeshPartDetailLevel.Low
    r.EagerBulkExecution      = false
    r.InterpolationThrottling = true
    Lighting.GlobalShadows    = false
    Lighting.Ambient          = Color3.fromRGB(80,80,80)
    Lighting.OutdoorAmbient   = Color3.fromRGB(80,80,80)
    Lighting.FogStart         = 0
    Lighting.FogEnd           = 1e6
end)

-- escapa instâncias de UI e interações
local function shouldPreserve(inst)
    if not inst.Parent then return false end

    -- tudo em PlayerGui ou StarterGui
    if inst:IsDescendantOf(Players.LocalPlayer:WaitForChild("PlayerGui")) then
        return true
    end
    -- interações 3D no Workspace
    if inst:IsA("ProximityPrompt")
    or inst:IsA("BillboardGui")
    or inst:IsA("SurfaceGui")
    or inst:IsA("ClickDetector") then
        return true
    end
    return false
end

-- identifica se pertence a um personagem (preservar)
local function isInCharacter(inst)
    for _, plr in ipairs(Players:GetPlayers()) do
        local char = plr.Character
        if char and inst:IsDescendantOf(char) then
            return true
        end
    end
    return false
end

-- otimização agressiva
local function optimizeInstance(inst)
    if CollectionService:HasTag(inst, "AL_Optimized") then return end
    if shouldPreserve(inst) then return end
    if isInCharacter(inst)   then return end

    pcall(function()
        CollectionService:AddTag(inst, "AL_Optimized")
        if inst:IsA("Sky")
        or inst:IsA("Atmosphere")
        or inst:IsA("Clouds") then
            inst:Destroy()
        elseif inst:IsA("PostEffect") then
            inst.Enabled = false
        elseif inst:IsA("Terrain") then
            inst.WaterWaveSize     = 0
            inst.WaterWaveSpeed    = 0
            inst.WaterReflectance  = 0
            inst.WaterTransparency = 1
        elseif inst:IsA("ParticleEmitter")
           or inst:IsA("Trail")
           or inst:IsA("Smoke")
           or inst:IsA("Sparkles")
           or inst:IsA("Fire")
           or inst:IsA("Explosion") then
            inst.Enabled = false
        elseif inst:IsA("Light") then
            inst.Shadows    = false
            inst.Brightness = (inst.Brightness or 1) * 0.5
        elseif inst:IsA("Decal")
           or inst:IsA("Texture")
           or inst:IsA("SurfaceAppearance") then
            inst:Destroy()
        elseif inst:IsA("Constraint") then
            inst:Destroy()
        elseif inst:IsA("BasePart") then
            inst.Material    = Enum.Material.SmoothPlastic
            inst.CastShadow  = false
            inst.Reflectance = 0
        end
    end)
end

-- enfileira para otimizar
local function enqueue(inst)
    if inst and inst.Parent
    and not CollectionService:HasTag(inst, "AL_Optimized") then
        optimizeQueue[#optimizeQueue+1] = inst
    end
end

-- fila inicial
for _, inst in ipairs(Workspace:GetDescendants()) do enqueue(inst) end
for _, inst in ipairs(Lighting:GetDescendants())   do enqueue(inst) end

-- responde a mudanças no jogo
Workspace.DescendantAdded:Connect(enqueue)
Lighting.DescendantAdded:Connect(enqueue)
Workspace.ChildRemoved:Connect(function()
    task.delay(1, function()
        for _, inst in ipairs(Workspace:GetDescendants()) do enqueue(inst) end
        for _, inst in ipairs(Lighting:GetDescendants())   do enqueue(inst) end
    end)
end)
Players.LocalPlayer.CharacterAdded:Connect(function()
    fpsBuffer     = {}
    optimizeQueue = {}
    task.delay(1, function()
        for _, inst in ipairs(Workspace:GetDescendants()) do enqueue(inst) end
        for _, inst in ipairs(Lighting:GetDescendants())   do enqueue(inst) end
    end)
end)

-- loop principal: batch dinâmico conforme FPS
RunService.Heartbeat:Connect(function(dt)
    local fps = 1/ math.max(dt,1e-4)
    fpsBuffer[#fpsBuffer+1] = fps
    if #fpsBuffer > SAMPLE_COUNT then
        table.remove(fpsBuffer,1)
    end

    local sum = 0
    for _,v in ipairs(fpsBuffer) do sum = sum + v end
    local avg = sum / #fpsBuffer

    local batch
    if avg >= FPS_HIGH then
        batch = BATCH_MAX
    elseif avg <= FPS_LOW then
        batch = BATCH_MIN
    else
        local t = (avg - FPS_LOW)/(FPS_HIGH - FPS_LOW)
        batch = math.floor(BATCH_MIN + t*(BATCH_MAX-BATCH_MIN))
    end

    for i=1,batch do
        local inst = table.remove(optimizeQueue,1)
        if not inst then break end
        optimizeInstance(inst)
    end
end)
