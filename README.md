--[[
	WARNING: Heads up! This script has not been verified by ScriptBlox. Use at your own risk!
]]
local player=game.Players.LocalPlayer
local workspace=game.Workspace
local lighting=game.Lighting
local RunService=game:GetService("RunService")
local camera=workspace.CurrentCamera
local LOW_QUALITY=true
local VOXEL_SIZE=1.2
local BLUR_INTENSITY=10
local MAX_PARTS=8000
local DISABLE_SHAD=true
local FLAT_COLORS=true
lighting.GlobalShadows=false
lighting.Brightness=0.5
lighting.Ambient=Color3.new(0.4,0.4,0.4)
lighting.OutdoorAmbient=Color3.new(0.3,0.3,0.3)
lighting.ShadowSoftness=0
lighting.EnvironmentDiffuseScale=0
lighting.EnvironmentSpecularScale=0
lighting.Technology=Enum.Technology.Compatibility
for _,light in pairs(lighting:GetChildren())do if light:IsA("Light")then light.Enabled=false end end
for _,obj in pairs(workspace:GetDescendants())do
if obj:IsA("Light")then obj.Enabled=false end
if obj:IsA("Atmosphere")then obj.Enabled=false end
if obj:IsA("BloomEffect")or obj:IsA("BlurEffect")or obj:IsA("ColorCorrectionEffect")or obj:IsA("DepthOfFieldEffect")or obj:IsA("SunRaysEffect")then obj.Enabled=false end
end
for _,obj in pairs(workspace:GetDescendants())do
if obj:IsA("BasePart")then
obj.Material=Enum.Material.Plastic
for _,child in pairs(obj:GetChildren())do
if child:IsA("Texture")or child:IsA("Decal")or child:IsA("SurfaceAppearance")then child:Destroy()end
end
if FLAT_COLORS then
obj.Color=Color3.new(math.random(20,80)/100,math.random(20,80)/100,math.random(20,80)/100)
end
end
if obj:IsA("ParticleEmitter")or obj:IsA("Trail")or obj:IsA("Beam")or obj:IsA("Smoke")or obj:IsA("Fire")then
obj.Enabled=false
obj.Rate=0
end
end
for _,obj in pairs(workspace:GetDescendants())do
if obj:IsA("MeshPart")or obj:IsA("UnionOperation")then
obj.RenderFidelity=Enum.RenderFidelity.Performance
end
if obj:IsA("SpecialMesh")then obj:Destroy()end
end
local partCount=0
local function potatoVoxel(part)
if partCount>MAX_PARTS or part.Parent==player.Character or not LOW_QUALITY then return end
partCount+=1
local size=part.Size
local cf=part.CFrame
local steps=Vector3.new(1,1,1)/VOXEL_SIZE
for x=1,math.ceil(size.X*steps.X)do
for y=1,math.ceil(size.Y*steps.Y)do
for z=1,math.ceil(size.Z*steps.Z)do
if partCount>MAX_PARTS then return end
local voxel=Instance.new("Part")
voxel.Name="PotatoVoxel"
voxel.Size=Vector3.new(VOXEL_SIZE,VOXEL_SIZE,VOXEL_SIZE)
voxel.Material=Enum.Material.Plastic
voxel.Color=part.Color
voxel.Anchored=part.Anchored
voxel.CanCollide=false
voxel.TopSurface=Enum.SurfaceType.SmoothNoOutlines
local offset=Vector3.new((x-0.5)*VOXEL_SIZE-size.X/2,(y-0.5)*VOXEL_SIZE-size.Y/2,(z-0.5)*VOXEL_SIZE-size.Z/2)
voxel.CFrame=cf+cf:VectorToWorldSpace(offset)
voxel.Parent=part.Parent
end
end
end
part:Destroy()
end
spawn(function()
for _,part in pairs(workspace:GetDescendants())do
if part:IsA("BasePart")and not part.Parent:FindFirstChild("Humanoid")then
spawn(function()potatoVoxel(part)end)
end
wait(0.001)
end
end)
workspace.DescendantAdded:Connect(function(obj)
wait(0.05)
if obj:IsA("BasePart")then
obj.Material=Enum.Material.Plastic
obj.CastShadow=false
for _,child in pairs(obj:GetChildren())do
if child:IsA("Texture")or child:IsA("Decal")then child:Destroy()end
end
if LOW_QUALITY then spawn(function()potatoVoxel(obj)end)end
end
end)
local blur=Instance.new("BlurEffect")
blur.Size=BLUR_INTENSITY
blur.Parent=camera
local terrain=workspace.Terrain
terrain.WaterTransparency=1
terrain.WaterReflectance=0
terrain:Clear()
