-- [[ Ultimate Time Bomb Auto-Parry Script for Mobile ]] --
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local parryEnabled = false
local parryRange = 15 -- المسافة الافتراضية للصد (يمكنك تعديلها من الواجهة)

-- دالة للبحث عن أقرب قنبلة قادمة نحوك
local function getClosestBomb()
    local character = LocalPlayer.Character
    if not character then return nil end
    local root = character:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    
    local closestBomb = nil
    local shortestDistance = math.huge
    
    -- البحث في مساحة العمل (Workspace) لتجنب اللاغ
    for _, child in ipairs(workspace:GetChildren()) do
        if child:IsA("BasePart") and (child.Name == "Bomb" or child.Name:match("Bomb")) then
            -- التأكد أن القنبلة ليست بيدك حالياً
            if not child:IsDescendantOf(character) then
                local dist = (child.Position - root.Position).Magnitude
                if dist < shortestDistance then
                    closestBomb = child
                    shortestDistance = dist
                end
            end
        elseif child:IsA("Model") and (child.Name == "Bomb" or child.Name:match("Bomb")) then
            local primary = child.PrimaryPart or child:FindFirstChildWhichIsA("BasePart")
            if primary and not child:IsDescendantOf(character) then
                local dist = (primary.Position - root.Position).Magnitude
                if dist < shortestDistance then
                    closestBomb = primary
                    shortestDistance = dist
                end
            end
        end
    end
    return closestBomb, shortestDistance
end

-- حلقة التفعيل السريع والصد التلقائي
task.spawn(function()
    while true do
        task.wait() -- فحص مستمر فائق السرعة
        if parryEnabled then
            local bomb, distance = getClosestBomb()
            if bomb and distance <= parryRange then
                local character = LocalPlayer.Character
                if character then
                    local backpack = LocalPlayer:FindFirstChild("Backpack")
                    local tool = character:FindFirstChildOfClass("Tool") or (backpack and backpack:FindFirstChildOfClass("Tool"))
                    
                    if tool then
                        -- إذا كان السلاح في الحقيبة، يتم تجهيزه تلقائياً
                        if tool.Parent == backpack then
                            character:WaitForChild("Humanoid"):EquipTool(tool)
                        end
                        -- تنفيذ ضربة الصد (Parry)
                        tool:Activate()
                    end
                end
            end
        end
    end
end)

-- === تصميم واجهة الجوال (GUI) ===
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local UICorner = Instance.new("UICorner")
local Title = Instance.new("TextLabel")
local ToggleButton = Instance.new("TextButton")
local ButtonCorner = Instance.new("UICorner")
local RangeInput = Instance.new("TextBox")
local RangeCorner = Instance.new("UICorner")

ScreenGui.Name = "TimeBombParryGui"
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false

-- الإطار الرئيسي
MainFrame.Name = "MainFrame"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
MainFrame.Position = UDim2.new(0.1, 0, 0.3, 0)
MainFrame.Size = UDim2.new(0, 150, 0, 180)
MainFrame.Active = true
MainFrame.Draggable = true -- تتيح لك سحب الواجهة بيدك في أي مكان على الشاشة

UICorner.CornerRadius = UDim.new(0, 12)
UICorner.Parent = MainFrame

-- العنوان
Title.Parent = MainFrame
Title.BackgroundTransparency = 1
Title.Size = UDim2.new(1, 0, 0, 40)
Title.Font = Enum.Font.SourceSansBold
Title.Text = "AUTO PARRY"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 18

-- زر التشغيل والإطفاء
ToggleButton.Parent = MainFrame
ToggleButton.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
ToggleButton.Position = UDim2.new(0.1, 0, 0.25, 0)
ToggleButton.Size = UDim2.new(0.8, 0, 0, 40)
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.Text = "OFF"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.TextSize = 16

ButtonCorner.CornerRadius = UDim.new(0, 8)
ButtonCorner.Parent = ToggleButton

-- خانة تعديل المسافة (Range)
RangeInput.Parent = MainFrame
RangeInput.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
RangeInput.Position = UDim2.new(0.1, 0, 0.55, 0)
RangeInput.Size = UDim2.new(0.8, 0, 0, 40)
RangeInput.Font = Enum.Font.SourceSans
RangeInput.Text = "Range: 15"
RangeInput.TextColor3 = Color3.fromRGB(200, 200, 200)
RangeInput.TextSize = 14

RangeCorner.CornerRadius = UDim.new(0, 8)
RangeCorner.Parent = RangeInput

-- برمجة زر التشغيل
ToggleButton.MouseButton1Click:Connect(function()
    parryEnabled = not parryEnabled
    if parryEnabled then
        ToggleButton.Text = "ON"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(40, 180, 40)
    else
        ToggleButton.Text = "OFF"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
    end
end)

-- برمجة خانة تعديل المسافة
RangeInput.FocusLost:Connect(function()
    local text = RangeInput.Text:gsub("Range: ", "")
    local num = tonumber(text)
    if num then
        parryRange = num
        RangeInput.Text = "Range: " .. tostring(num)
    else
        RangeInput.Text = "Range: " .. tostring(parryRange)
    end
end)

-- === الإضافة الجديدة: إخفاء/إظهار الواجهة بضغطة مطولة على Ctrl ===
local holdDuration = 0.5 -- المدة المطلوبة بالثواني (نصف ثانية)
local isHoldingCtrl = false
local holdStartTime = 0

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    -- تجاهل الضغطة إذا كان اللاعب يكتب في الشات مثلاً
    if gameProcessed then return end
    
    -- التحقق من الضغط على Ctrl (اليسار أو اليمين)
    if input.KeyCode == Enum.KeyCode.LeftControl or input.KeyCode == Enum.KeyCode.RightControl then
        isHoldingCtrl = true
        holdStartTime = os.clock()
        
        -- انتظار المدة المحددة للضغطة المطولة
        task.delay(holdDuration, function()
            -- التأكد أن اللاعب لا يزال ضاغطاً على الزر وأن الوقت قد مر
            if isHoldingCtrl and (os.clock() - holdStartTime) >= holdDuration then
                ScreenGui.Enabled = not ScreenGui.Enabled -- يعكس حالة الواجهة (إخفاء/إظهار)
            end
        end)
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
    -- عند إفلات زر Ctrl، يتم إلغاء حالة "الضغط المستمر"
    if input.KeyCode == Enum.KeyCode.LeftControl or input.KeyCode == Enum.KeyCode.RightControl then
        isHoldingCtrl = false
    end
end)
