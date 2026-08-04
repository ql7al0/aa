-- الانتظار لمدة ثانيتين قبل بدء السكربت
task.wait(2)

local player = game.Players.LocalPlayer

-- إنشاء واجهة المستخدم (GUI)
local screenGui = Instance.new("ScreenGui")
-- وضع الواجهة في واجهة اللاعب لتكون آمنة وتعمل بشكل سليم
screenGui.Parent = player:WaitForChild("PlayerGui") 

-- إنشاء زر التشغيل والإيقاف
local toggleButton = Instance.new("TextButton")
toggleButton.Parent = screenGui
toggleButton.Size = UDim2.new(0, 150, 0, 50)
toggleButton.Position = UDim2.new(0.5, -75, 0.1, 0) -- وضع الزر في أعلى منتصف الشاشة
toggleButton.Text = "Speed: OFF"
toggleButton.TextScaled = true
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50) -- لون أحمر افتراضي (مغلق)

local isSpeedOn = false

-- وظيفة الزر عند الضغط عليه
toggleButton.MouseButton1Click:Connect(function()
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:FindFirstChildOfClass("Humanoid")

    if humanoid then
        isSpeedOn = not isSpeedOn
        if isSpeedOn then
            -- تشغيل السرعة إلى 20
            humanoid.WalkSpeed = 20
            toggleButton.Text = "Speed: ON"
            toggleButton.BackgroundColor3 = Color3.fromRGB(50, 255, 50) -- تحويل لون الزر للأخضر
        else
            -- إرجاع السرعة للافتراضية (16)
            humanoid.WalkSpeed = 16 
            toggleButton.Text = "Speed: OFF"
            toggleButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50) -- تحويل لون الزر للأحمر
        end
    end
end)
