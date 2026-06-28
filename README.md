-- [[ OMNI-FIX-LAG MASTER V3.7 - ANTI-CRASH & SMOOTH ]] --
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local LightEnabled = false
local MaxRenderDistance = 120 -- Khoảng cách kết xuất (thay vì xóa vĩnh viễn vật thể)

-- Xác định nơi chèn UI an toàn
local TargetGui
local success, _ = pcall(function() local a = CoreGui.Name end)
if success then TargetGui = CoreGui else TargetGui = LocalPlayer:WaitForChild("PlayerGui") end

if TargetGui:FindFirstChild("OmniLagGui") then TargetGui.OmniLagGui:Destroy() end

-- 1. TẠO UI SIÊU MINI
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local UIListLayout = Instance.new("UIListLayout")

ScreenGui.Name = "OmniLagGui"
ScreenGui.Parent = TargetGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

MainFrame.Name = "MainFrame"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
MainFrame.BackgroundTransparency = 0.4
MainFrame.Position = UDim2.new(0, 10, 0, 10)
MainFrame.Size = UDim2.new(0, 105, 0, 0)
MainFrame.AutomaticSize = Enum.AutomaticSize.Y
MainFrame.BorderSizePixel = 0

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 4)
UICorner.Parent = MainFrame

UIListLayout.Parent = MainFrame
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Padding = UDim.new(0, 3)

local function CreateLabel(name, text, color)
    local Label = Instance.new("TextLabel")
    Label.Name = name
    Label.Parent = MainFrame
    Label.BackgroundTransparency = 1
    Label.Size = UDim2.new(1, 0, 0, 14)
    Label.Font = Enum.Font.SourceSansBold
    Label.Text = text
    Label.TextColor3 = color
    Label.TextSize = 11
    Label.TextXAlignment = Enum.TextXAlignment.Left
    
    local UIPadding = Instance.new("UIPadding")
    UIPadding.PaddingLeft = UDim.new(0, 6)
    UIPadding.Parent = Label
    return Label
end

local FpsLabel = CreateLabel("FpsLabel", "FPS: --", Color3.fromRGB(0, 255, 100))
local PingLabel = CreateLabel("PingLabel", "Ping: -- ms", Color3.fromRGB(0, 220, 255))

-- SỬA UI NÚT BẤM: Đảm bảo phân cấp chuẩn để mượt mà hơn
local LightBtnFrame = Instance.new("Frame")
LightBtnFrame.Name = "LightBtnFrame"
LightBtnFrame.Parent = MainFrame
LightBtnFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
LightBtnFrame.Size = UDim2.new(0, 95, 0, 22)

local BtnCorner = Instance.new("UICorner")
BtnCorner.CornerRadius = UDim.new(0, 4)
BtnCorner.Parent = LightBtnFrame

local BtnPadding = Instance.new("UIPadding")
BtnPadding.PaddingLeft = UDim.new(0, 5)
BtnPadding.Parent = MainFrame

local TextButton = Instance.new("TextButton")
TextButton.Size = UDim2.new(1, 0, 1, 0)
TextButton.BackgroundTransparency = 1
TextButton.Text = "LIGHT: OFF"
TextButton.Font = Enum.Font.SourceSansBold
TextButton.TextColor3 = Color3.fromRGB(255, 165, 0)
TextButton.TextSize = 11
TextButton.Parent = LightBtnFrame

-- Cập nhật FPS & Ping tối ưu
local FrameCount = 0
local LastUpdate = os.clock()

RunService.RenderStepped:Connect(function()
    FrameCount = FrameCount + 1
    local Now = os.clock()
    if Now - LastUpdate >= 1 then
        FpsLabel.Text = "FPS: " .. tostring(FrameCount)
        FrameCount = 0
        LastUpdate = Now
        
        local PrecisePing = math.floor(LocalPlayer:GetNetworkPing() * 1000)
        PingLabel.Text = "Ping: " .. tostring(PrecisePing) .. " ms"
    end
end)

-- 2. TỐI ƯU HÓA ÁNH SÁNG BAN ĐẦU
local function CleanLighting()
    Lighting:ClearAllChildren()
    Lighting.FogStart = 999999
    Lighting.FogEnd = 999999
    Lighting.Ambient = Color3.fromRGB(128, 128, 128)
    Lighting.OutdoorAmbient = Color3.fromRGB(128, 128, 128)
end
CleanLighting()

-- 3. LÀM NHỰA BỀ MẶT (CHỈ CHẠY 1 LẦN KHI LOAD VÀ KHI THÊM MỚI)
local function OptimizeObject(Obj)
    if Obj:IsA("ParticleEmitter") or Obj:IsA("Smoke") or Obj:IsA("Fire") or Obj:IsA("Sparkles") or Obj:IsA("Decal") or Obj:IsA("Texture") then
        Obj:Destroy()
    elseif Obj:IsA("BasePart") and not Obj:IsA("Terrain") then
        Obj.Material = Enum.Material.SmoothPlastic
        Obj.CastShadow = false
        if Obj:IsA("MeshPart") then
            Obj.RenderFidelity = Enum.RenderFidelity.Performance
        end
    elseif Obj:IsA("SpecialMesh") and Obj.MeshType == Enum.MeshType.FileMesh then
        Obj.Scale = Obj.Scale * 0.9
    end
end

for _, v in ipairs(Workspace:GetDescendants()) do OptimizeObject(v) end
Workspace.DescendantAdded:Connect(function(v) task.wait() OptimizeObject(v) end)

-- 4. BỘ ĐIỀU KHIỂN ÁNH SÁNG (SỬA LỖI: CHỈ CẬP NHẬT KHI CÓ SỰ KIỆN CLICK)
local function UpdateLightingProperties()
    if LightEnabled then
        Lighting.Brightness = 3
        Lighting.ClockTime = 14
        Lighting.GlobalShadows = false
        TextButton.Text = "LIGHT: ON"
        TextButton.TextColor3 = Color3.fromRGB(0, 255, 100)
        LightBtnFrame.BackgroundColor3 = Color3.fromRGB(0, 60, 0)
    else
        Lighting.Brightness = 1
        Lighting.ClockTime = 12
        Lighting.GlobalShadows = true
        TextButton.Text = "LIGHT: OFF"
        TextButton.TextColor3 = Color3.fromRGB(255, 165, 0)
        LightBtnFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    end
end

local function ToggleLighting()
    LightEnabled = not LightEnabled
    UpdateLightingProperties()
end

-- Gọi khởi tạo ban đầu, không dùng vòng lặp vô hạn gây lag CPU
UpdateLightingProperties()

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.L then
        ToggleLighting()
    end
end)

TextButton.MouseButton1Click:Connect(ToggleLighting)

-- 5. CƠ CHẾ ĐIỀU CHỈNH TẦM NHÌN THÔNG MINH (SỬA LỖI: KHÔNG XÓA MAP)
-- Cơ chế này ẩn các object ở quá xa (ngoài 120 studs) để GPU nhẹ hơn, và tự hiện lại khi bạn đi tới gần.
task.spawn(function()
    while true do
        local Character = LocalPlayer.Character
        if Character and Character:FindFirstChild("HumanoidRootPart") then
            local RootPos = Character.HumanoidRootPart.Position
            for _, Obj in ipairs(Workspace:GetDescendants()) do
                if Obj:IsA("BasePart") and not Obj:IsA("Terrain") and not Obj:IsDescendantOf(Character) then
                    local NameLower = string.lower(Obj.Name)
                    -- Bảo vệ các phần quan trọng
                    if not string.find(NameLower, "baseplate") and 
                       not string.find(NameLower, "floor") and 
                       not string.find(NameLower, "ground") and
                       not Obj.Parent:FindFirstChild("Humanoid") then
                        
                        local Distance = (Obj.Position - RootPos).Magnitude
                        if Distance > MaxRenderDistance then
                            -- Nếu quá xa, ẩn đi để tăng FPS
                            if not Obj:GetAttribute("OriginalTransparency") then
                                Obj:SetAttribute("OriginalTransparency", Obj.Transparency)
                            end
                            Obj.Transparency = 1
                            Obj.CanCollide = false
                        else
                            -- Nếu lại gần, hiện lại như cũ để không hỏng map
                            local orig = Obj:GetAttribute("OriginalTransparency")
                            if orig then
                                Obj.Transparency = orig
                                Obj.CanCollide = true
                            end
                        end
                    end
                end
            end
        end
        task.wait(1.5) -- Tăng thời gian chờ lên 1.5s để giảm tải cho CPU
    end
end)
