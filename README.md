local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local normalSpeed = 16
local targetSpeed = 20
local active = true -- حالة الزر (يعمل افتراضياً عند التشغيل)

-- [[ 1. تصميم واجهة الزر (من السكربت الثاني) ]] --
local mainGui = Instance.new("ScreenGui", player.PlayerGui)
mainGui.ResetOnSpawn = false

local toggle = Instance.new("TextButton", mainGui)
toggle.Size = UDim2.new(0, 140, 0, 45)
toggle.Position = UDim2.new(0.05, 0, 0.4, 0)
toggle.Text = "SAADHUB: ON"
toggle.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
toggle.TextColor3 = Color3.new(1, 1, 1)
toggle.Font = Enum.Font.GothamBold
toggle.TextSize = 16
Instance.new("UICorner", toggle)

local stroke = Instance.new("UIStroke", toggle)
stroke.Color = Color3.new(1, 1, 1)

-- [[ 2. جعل الزر قابل للسحب (Drag) ]] --
local dragCircle = Instance.new("Frame", toggle)
dragCircle.Size = UDim2.new(0, 25, 0, 25)
dragCircle.Position = UDim2.new(0.5, -12.5, 0, -32)
dragCircle.BackgroundTransparency = 1
local dragCorner = Instance.new("UICorner", dragCircle)
dragCorner.CornerRadius = UDim.new(1, 0)
local dragStroke = Instance.new("UIStroke", dragCircle)
dragStroke.Transparency = 1

local dragging, dragStart, startPos
dragCircle.InputBegan:Connect(function(input) 
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then 
        dragging = true
        dragStart = input.Position
        startPos = toggle.Position 
    end 
end)
UserInputService.InputChanged:Connect(function(input) 
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then 
        local delta = input.Position - dragStart
        toggle.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) 
    end 
end)
UserInputService.InputEnded:Connect(function() 
    dragging = false 
end)

-- [[ 3. نظام التشغيل والإيقاف للزر ]] --
local function toggleScript()
    active = not active
    toggle.Text = active and "SAADHUB: ON" or "SAADHUB: OFF"
    toggle.BackgroundColor3 = active and Color3.fromRGB(170, 0, 0) or Color3.fromRGB(40, 40, 40)
    
    -- إذا تم إطفاء الزر، قم بإرجاع السرعة للطبيعي فوراً
    if not active and player.Character then
        local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = normalSpeed
        end
    end
end
-- ربط الضغط على الزر بتغيير الحالة
toggle.MouseButton1Click:Connect(toggleScript)


-- [[ 4. السكربت الأول (سكربت السرعة والقنبلة) ]] --

-- دالة لقراءة وقت القنبلة من اللعبة
local function getBombTime()
    -- استبدل هذا الكود بالمسار الصحيح للعداد
    return -1 
end

-- التشغيل في كل إطار (Frame)
RunService.Heartbeat:Connect(function()
    -- إذا كان الزر على حالة OFF، نوقف عمل السكربت نهائياً هنا
    if not active then return end 

    local character = player.Character
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")
    
    -- التحقق من أن الشخصية موجودة وحية
    if humanoid and humanoid.Health > 0 then
        local bombTime = getBombTime()
        
        -- إذا وصل وقت القنبلة إلى 2
        if bombTime == 2 then
            if humanoid.WalkSpeed ~= targetSpeed then
                humanoid.WalkSpeed = targetSpeed
            end
        else
            -- إذا كان الوقت مختلفاً، ترجع السرعة إلى 16
            if humanoid.WalkSpeed ~= normalSpeed then
                humanoid.WalkSpeed = normalSpeed
            end
        end
    end
end)

-- [[ 5. إضافة اختصار الكيبورد (زر 0) ]] --
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    -- gameProcessed يمنع تفعيل السكربت إذا كنت تكتب في الشات
    if not gameProcessed then
        -- التحقق مما إذا كان الزر المضغوط هو 0
        if input.KeyCode == Enum.KeyCode.Zero then
            toggleScript() -- استدعاء نفس الدالة المسؤولة عن التشغيل والإيقاف
        end
    end
end)
