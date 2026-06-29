local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

-- [ الإعدادات ] --
local isEnabled = false
local smoothing = 0.04 -- سرعة السحب (كلما كان الرقم أقل، كان الايم أخف وأكثر طبيعية)
local fovRadius = 150 -- مساحة البحث عن الخصم حول مؤشر الماوس

-- [ بناء واجهة SaadHub ] --
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local Title = Instance.new("TextLabel")
local ToggleButton = Instance.new("TextButton")
local UIStroke = Instance.new("UIStroke")
local UICorner = Instance.new("UICorner")

-- حماية الواجهة من اكتشاف اللعبة (Anti-Cheat Bypass)
local success, err = pcall(function()
    ScreenGui.Parent = game:GetService("CoreGui")
end)
if not success then
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

ScreenGui.Name = "SaadHub_NeuralAim"

MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15) -- أسود داكن
MainFrame.Position = UDim2.new(0.8, 0, 0.5, 0)
MainFrame.Size = UDim2.new(0, 180, 0, 80)
MainFrame.Active = true
MainFrame.Draggable = true -- قابل للسحب بالشاشة

UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

UIStroke.Color = Color3.fromRGB(138, 43, 226) -- بنفسجي نيون (Neon Purple)
UIStroke.Thickness = 2
UIStroke.Parent = MainFrame

Title.Parent = MainFrame
Title.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Title.BackgroundTransparency = 1
Title.Size = UDim2.new(1, 0, 0.4, 0)
Title.Font = Enum.Font.GothamBold
Title.Text = "SaadHub Aim"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 14

ToggleButton.Parent = MainFrame
ToggleButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
ToggleButton.Position = UDim2.new(0.1, 0, 0.45, 0)
ToggleButton.Size = UDim2.new(0.8, 0, 0.4, 0)
ToggleButton.Font = Enum.Font.GothamSemibold
ToggleButton.Text = "OFF"
ToggleButton.TextColor3 = Color3.fromRGB(255, 50, 50) -- أحمر عند الإيقاف
ToggleButton.TextSize = 14

local ButtonCorner = Instance.new("UICorner")
ButtonCorner.CornerRadius = UDim.new(0, 6)
ButtonCorner.Parent = ToggleButton

-- [ نظام التخفي العصبي - Neural Stealth Logic ] --
local function getClosestPlayer()
    local closestTarget = nil
    local shortestDistance = fovRadius

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid").Health > 0 then
            -- التأكد من أن الخصم ليس خلف جدار (اختياري لزيادة الأمان)
            local targetPos, onScreen = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
            
            if onScreen then
                local mousePos = Vector2.new(Mouse.X, Mouse.Y)
                local distance = (Vector2.new(targetPos.X, targetPos.Y) - mousePos).Magnitude

                if distance < shortestDistance then
                    shortestDistance = distance
                    closestTarget = player.Character.HumanoidRootPart
                end
            end
        end
    end
    return closestTarget
end

-- زر التفعيل والإيقاف
ToggleButton.MouseButton1Click:Connect(function()
    isEnabled = not isEnabled
    if isEnabled then
        ToggleButton.Text = "ON"
        ToggleButton.TextColor3 = Color3.fromRGB(50, 255, 50) -- أخضر عند التشغيل
        UIStroke.Color = Color3.fromRGB(255, 215, 0) -- يتغير لذهبي عند التفعيل (Gold)
    else
        ToggleButton.Text = "OFF"
        ToggleButton.TextColor3 = Color3.fromRGB(255, 50, 50)
        UIStroke.Color = Color3.fromRGB(138, 43, 226) -- يرجع بنفسجي نيون
    end
end)

-- حلقة التوجيه السلسة
RunService.RenderStepped:Connect(function()
    if isEnabled then
        local target = getClosestPlayer()
        if target then
            -- تم إضافة إزاحة (Vector3.new) لرفع مستوى التصويب إلى الرأس/الصدر لرؤية الخصم بوضوح
            local aimPosition = target.Position + Vector3.new(0, 1.5, 0)
            
            -- استخدام CFrame.Lerp لانتقال سلس جداً يخدع المراقبة
            local targetCFrame = CFrame.new(Camera.CFrame.Position, aimPosition)
            Camera.CFrame = Camera.CFrame:Lerp(targetCFrame, smoothing)
        end
    end
end)
