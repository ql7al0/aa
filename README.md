-- LocalScript - Speed Toggle UI with Bomb Detection
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

local isEnabled = false
local normalSpeed = 16
local targetSpeed = 20

-- 1. إنشاء واجهة المستخدم (GUI) والزر
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "SpeedModGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local toggleBtn = Instance.new("TextButton")
toggleBtn.Name = "ToggleSpeed"
toggleBtn.Size = UDim2.new(0, 130, 0, 45)
toggleBtn.Position = UDim2.new(0.05, 0, 0.35, 0)
toggleBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50) -- أحمر عند الإيقاف
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.Text = "Speed: OFF"
toggleBtn.TextSize = 18
toggleBtn.Font = Enum.Font.SourceSansBold
toggleBtn.Active = true
toggleBtn.Draggable = true -- يمكنك سحب الزر لأي مكان في الشاشة
toggleBtn.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = toggleBtn

-- 2. وظيفة التبديل (ON / OFF)
toggleBtn.MouseButton1Click:Connect(function()
    isEnabled = not isEnabled
    if isEnabled then
        toggleBtn.Text = "Speed: ON"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(50, 200, 50) -- أخضر عند التشغيل
    else
        toggleBtn.Text = "Speed: OFF"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
        -- إرجاع السرعة للطبيعي عند الإيقاف
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            LocalPlayer.Character.Humanoid.WalkSpeed = normalSpeed
        end
    end
end)

-- 3. دالة البحث عن القنبلة أو عداد الوقت (2 ثانية)
local function checkBombTimer()
    -- البحث في واجهة اللاعب أو الماب عن نص العداد "2" أو "00:02"
    local playerGui = LocalPlayer:FindFirstChild("PlayerGui")
    if not playerGui then return false end

    -- يفحص جميع النصوص المعروضة في الشاشة للبحث عن رقم 2 أو القنبلة
    for _, gui in pairs(playerGui:GetDescendants()) do
        if gui:IsA("TextLabel") and gui.Visible then
            local text = tostring(gui.Text)
            if text == "2" or text == "02" or text == "00:02" or text:find("2s") then
                return true
            end
        end
    end
    return false
end

-- 4. الحلقة الأساسية لتطبيق السرعة
RunService.Heartbeat:Connect(function()
    local char = LocalPlayer.Character
    if not char then return end
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end

    if isEnabled then
        -- إذا تحقق شرط العداد عند الثانية 2 (أو أثناء تفعيل المود)
        if checkBombTimer() or isEnabled then
            humanoid.WalkSpeed = targetSpeed
        end
    else
        if humanoid.WalkSpeed == targetSpeed then
            humanoid.WalkSpeed = normalSpeed
        end
    end
end)
