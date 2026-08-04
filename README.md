local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local normalSpeed = 16
local targetSpeed = 20

-- دالة لقراءة وقت القنبلة من اللعبة
local function getBombTime()
    -- بما أنك تستخدم Delta، يمكنك الوصول لواجهة المستخدم (PlayerGui) أو مساحة العمل (Workspace)
    -- هذا المسار افتراضي ويجب تعديله حسب اللعبة التي تلعبها
    
    -- مثال إذا كان العداد في واجهة الشاشة:
    -- pcall(function()
    --     local timerText = player.PlayerGui.BombUI.Timer.Text
    --     return tonumber(timerText)
    -- end)
    
    return -1 -- استبدل هذا الكود بالمسار الصحيح للعداد
end

-- التشغيل في كل إطار (Frame) لضمان الاستقرار
RunService.Heartbeat:Connect(function()
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
