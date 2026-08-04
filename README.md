local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer

-- ==========================================
-- 1. إعدادات إخفاء وإظهار الواجهة (GUI)
-- ==========================================

-- ملاحظة مهمة: استبدل "ScreenGui" باسم الواجهة الخاصة بك أو مسارها الصحيح
local myGui = game.CoreGui:FindFirstChild("ScreenGui") 

local isCtrlHeld = false

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    -- التحقق إذا الزر المضغوط هو Ctrl (يمين أو يسار)
    if input.KeyCode == Enum.KeyCode.LeftControl or input.KeyCode == Enum.KeyCode.RightControl then
        isCtrlHeld = true
        task.wait(3) -- الانتظار 3 ثواني
        
        -- إذا لا زال الزر مضغوط بعد 3 ثواني
        if isCtrlHeld then
            if myGui then
                myGui.Enabled = not myGui.Enabled -- يعكس الحالة (إخفاء/إظهار)
            end
        end
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.LeftControl or input.KeyCode == Enum.KeyCode.RightControl then
        isCtrlHeld = false -- إلغاء الضغطة إذا شال يده عن الزر
    end
end)


-- ==========================================
-- 2. إعدادات سرعة المشي والقنبلة
-- ==========================================

local isBombCycleRunning = false

local function startBombSpeedCycle()
    if isBombCycleRunning then return end
    isBombCycleRunning = true
    
    -- يشتغل على ثانية 2
    task.wait(2)
    
    local character = player.Character
    if character then
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = 20
        end
    end
    
    -- ينتظر 8 ثواني عشان يصير المجموع 10 ثواني من وقت ظهور القنبلة
    task.wait(8)
    
    -- يطفى على ثانية 10 ويرجع السرعة الطبيعية
    if character then
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = 16
        end
    end
    
    isBombCycleRunning = false
end

-- البحث عن القنبلة عند ظهورها في الماب
-- إذا كان اسم القنبلة مختلف في المود، تقدر تغير كلمة "bomb" للاسم الفعلي
game.Workspace.DescendantAdded:Connect(function(descendant)
    if string.find(string.lower(descendant.Name), "bomb") then
        task.spawn(startBombSpeedCycle)
    end
end)
