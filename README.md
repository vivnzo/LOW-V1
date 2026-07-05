-- Safe Cosmic Max Anti-Lag & Texture Stripper
-- يحسن الأداء الأقصى مع حماية كاملة لشخصية اللاعب وأدوات القتال لمنع التجمد

local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local Players = game:GetService("Players")
local Terrain = Workspace:FindFirstChildOfClass("Terrain")
local LocalPlayer = Players.LocalPlayer

-- تعطيل إعدادات الإضاءة المسببة للاق وإزالة الظلال
Lighting.GlobalShadows = false
Lighting.FogEnd = 9e9
Lighting.Brightness = 1

-- إزالة تأثيرات ما بعد المعالجة البصرية
for _, effect in ipairs(Lighting:GetChildren()) do
    if effect:IsA("PostEffect") or effect:IsA("BloomEffect") or effect:IsA("BlurEffect") or effect:IsA("ColorCorrectionEffect") or effect:IsA("SunRaysEffect") or effect:IsA("DepthOfFieldEffect") then
        effect.Enabled = false
    end
end

-- تحسين إعدادات التضاريس والمياه
if Terrain then
    Terrain.WaterWaveSize = 0
    Terrain.WaterWaveSpeed = 0
    Terrain.WaterReflectance = 0
    Terrain.WaterTransparency = 0
end

-- دالة الفحص لحماية اللاعب وأدوات القتال من الأعطال
local function IsSafeToModify(object)
    -- استثناء أدوات القتال والأسلحة (Tools) ومحتوياتها تماماً
    if object:IsA("Tool") or object:FindFirstAncestorOfClass("Tool") then
        return false
    end
    
    -- استثناء شخصية اللاعب الحالي لمنع تجمد الحركة والقفز
    if LocalPlayer.Character and (object == LocalPlayer.Character or object:IsAncestorOf(LocalPlayer.Character)) then
        return false
    end
    
    -- استثناء شخصيات اللاعبين الآخرين لضمان استقرار نظام اللعبة وعدم كسر التفاعل
    local possibleModel = object:FindFirstAncestorOfClass("Model")
    if possibleModel and Players:GetPlayerFromCharacter(possibleModel) then
        return false
    end

    return true
end

-- دالة تحسين العناصر وتنعيم الأسطح
local function OptimizeObject(object)
    -- التحقق من أمان العنصر قبل التعديل عليه
    if not IsSafeToModify(object) then return end

    -- إزالة الملصقات والأنسجة تماماً من الخريطة
    if object:IsA("Decal") or object:IsA("Texture") then
        object:Destroy()
    -- تعطيل الجسيمات والتأثيرات الحركية
    elseif object:IsA("ParticleEmitter") or object:IsA("Trail") or object:IsA("Beam") then
        object.Enabled = false
    elseif object:IsA("Fire") or object:IsA("Smoke") or object:IsA("Sparkles") then
        object.Enabled = false
    -- تحويل أسطح الماب إلى بلاستيك ناعم بلون صافي خالي من الانعكاسات
    elseif object:IsA("BasePart") or object:IsA("MeshPart") then
        object.Material = Enum.Material.SmoothPlastic
        object.Reflectance = 0
    -- إزالة الملابس غير الضرورية من مجسمات الماب البيئية
    elseif object:IsA("Shirt") or object:IsA("Pants") or object:IsA("ShirtGraphic") or object:IsA("Clothing") then
        object:Destroy()
    end
end

-- تطبيق التعديل الآمن على عناصر الخريطة الحالية
for _, object in ipairs(Workspace:GetDescendants()) do
    pcall(OptimizeObject, object)
end

-- مراقبة وتصفية العناصر الجديدة المضافة ديناميكياً مع استمرار الحماية
Workspace.DescendantAdded:Connect(function(object)
    pcall(OptimizeObject, object)
end)
