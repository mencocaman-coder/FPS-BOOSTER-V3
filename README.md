-- FPS Booster Ultra AutomÃ¡tico para Roblox (Super agressivo, leve e seguro)
-- Aplica as otimizaÃ§Ãµes mÃ¡ximas assim que executado

local ws = game:GetService("Workspace")
local lighting = game:GetService("Lighting")

local function cleanPart(obj)
    -- Remove partÃ­culas e efeitos de partes
    pcall(function()
        if obj:IsA("BasePart") then
            obj.Material = Enum.Material.SmoothPlastic
            obj.Reflectance = 0
            obj.CastShadow = false
        end
    end)
    if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Smoke")
    or obj:IsA("Sparkles") or obj:IsA("Fire") or obj:IsA("Explosion") then
        pcall(function() obj.Enabled = false end)
        pcall(function() obj.Visible = false end)
        pcall(function() obj.Lifetime = NumberRange.new(0) end)
    end
    if obj:IsA("Decal") or obj:IsA("Texture") then
        pcall(function() obj.Transparency = 1 end)
    end
    if obj:IsA("PointLight") or obj:IsA("SpotLight") or obj:IsA("SurfaceLight") then
        pcall(function()
            obj.Enabled = false
            obj.Brightness = 0
            obj.Range = 0
        end)
    end
end

local function boost()
    -- Otimiza todas as partes do workspace
    for _, obj in ipairs(ws:GetDescendants()) do
        cleanPart(obj)
    end

    -- Remove todos os efeitos de iluminaÃ§Ã£o
    for _, v in ipairs(lighting:GetChildren()) do
        if v:IsA("Sky") or v:IsA("Atmosphere") or v:IsA("Clouds")
        or v:IsA("BloomEffect") or v:IsA("SunRaysEffect") or v:IsA("ColorCorrectionEffect")
        or v:IsA("DepthOfFieldEffect") or v:IsA("BlurEffect") then
            pcall(function() v:Destroy() end)
        end
    end

    -- Configura iluminaÃ§Ã£o para o mÃ­nimo possÃ­vel
    pcall(function()
        lighting.GlobalShadows = false
        lighting.FogEnd = 1e10
        lighting.FogStart = 1e10
        lighting.Brightness = 1
        lighting.OutdoorAmbient = Color3.new(0.5, 0.5, 0.5)
        lighting.Ambient = Color3.new(0.5, 0.5, 0.5)
        lighting.EnvironmentDiffuseScale = 0
        lighting.EnvironmentSpecularScale = 0
    end)

    -- Configura o terreno
    local terrain = ws:FindFirstChildOfClass("Terrain")
    if terrain then
        pcall(function()
            terrain.WaterReflectance = 0
            terrain.WaterTransparency = 1
            terrain.WaterWaveSize = 0
            terrain.WaterWaveSpeed = 0
            terrain.Decorations = false
        end)
    end
end

-- Aplica boost em tudo assim que executar
boost()

-- Aplica nas novas partes que aparecerem no jogo
ws.DescendantAdded:Connect(cleanPart)

-- Reaplica boost a cada 10 segundos (caso algo volte)
while true do
    wait(10)
    boost()
end
