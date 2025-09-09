-- FPS Booster Ultra Automático (super agressivo, leve e seguro)
-- Créditos: @PepsiMannumero1  # Yotube

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local PlayerGui = player:WaitForChild("PlayerGui")
local LocalizationService = game:GetService("LocalizationService")

-- Traduções para "Fps Booster ativado"
local translations_fps = {
    ["en"] = "Fps Booster activated",
    ["es"] = "Fps Booster activado",
    ["fr"] = "Fps Booster activé",
    ["de"] = "Fps Booster aktiviert",
    ["ru"] = "Fps Booster активирован",
    ["it"] = "Fps Booster attivato",
    ["tr"] = "Fps Güçlendirici etkinleştirildi",
    ["pt"] = "Fps Booster ativado",
    ["ja"] = "FPSブースターが有効化されました",
    ["ko"] = "Fps 부스터가 활성화되었습니다",
    ["zh"] = "Fps加速器已激活",
}

local translations_mom = {
    ["en"] = "Your mom",
    ["es"] = "Tu mamá",
    ["fr"] = "Ta mère",
    ["de"] = "Deine Mutter",
    ["ru"] = "Твоя мама",
    ["it"] = "Tua madre",
    ["tr"] = "Annen",
    ["pt"] = "Sua mãe",
    ["ja"] = "あなたのお母さん",
    ["ko"] = "너희 엄마",
    ["zh"] = "你妈",
}

local function getLocaleText(translations)
    local locale = LocalizationService.RobloxLocaleId or "en"
    locale = locale:sub(1,2):lower()
    return translations[locale] or translations["en"]
end

local function showDialog(text, color, duration)
    local old = PlayerGui:FindFirstChild("FPSBoosterMsgGui")
    if old then old:Destroy() end

    local gui = Instance.new("ScreenGui")
    gui.Name = "FPSBoosterMsgGui"
    gui.ResetOnSpawn = false
    gui.IgnoreGuiInset = true

    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0.5, 0, 0.12, 0)
    frame.Position = UDim2.new(0.5, 0, 0.5, 0)
    frame.AnchorPoint = Vector2.new(0.5, 0.5)
    frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
    frame.BorderSizePixel = 0
    frame.Parent = gui

    local uicorner = Instance.new("UICorner")
    uicorner.CornerRadius = UDim.new(0.15,0)
    uicorner.Parent = frame

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1,0,1,0)
    label.Position = UDim2.new(0,0,0,0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = color or Color3.fromRGB(0, 174, 255)
    label.Font = Enum.Font.GothamBlack
    label.TextScaled = true
    label.TextWrapped = true
    label.Parent = frame

    gui.Parent = PlayerGui

    wait(duration or 1.5)
    gui:Destroy()
end

-- FPS Booster ativado (traduzido)
showDialog(getLocaleText(translations_fps), Color3.fromRGB(0,174,255), 1.5)
wait(1.5)
-- Créditos
showDialog("@PepsiMannumero1  # Yotube", Color3.fromRGB(0,174,255), 1.5)
wait(1.5)
-- "Sua mãe" traduzido
showDialog(getLocaleText(translations_mom), Color3.fromRGB(255,255,255), 1.5)

-- FPS BOOSTER (inalterado)
local ws = game:GetService("Workspace")
local lighting = game:GetService("Lighting")

local function cleanPart(obj)
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
    for _, obj in ipairs(ws:GetDescendants()) do
        cleanPart(obj)
    end

    for _, v in ipairs(lighting:GetChildren()) do
        if v:IsA("Sky") or v:IsA("Atmosphere") or v:IsA("Clouds")
        or v:IsA("BloomEffect") or v:IsA("SunRaysEffect") or v:IsA("ColorCorrectionEffect")
        or v:IsA("DepthOfFieldEffect") or v:IsA("BlurEffect") then
            pcall(function() v:Destroy() end)
        end
    end

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

boost()
ws.DescendantAdded:Connect(cleanPart)
while true do
    wait(10)
    boost()
end
