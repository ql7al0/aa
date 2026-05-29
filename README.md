-- [[ SAADHUB OFFICIAL - FULL VERSION V102 (MODIFIED FOR BOTS ONLY) ]] --

local player = game.Players.LocalPlayer
local httpService = game:GetService("HttpService")
local tweenService = game:GetService("TweenService")
local runService = game:GetService("RunService")
local userInputService = game:GetService("UserInputService")
local starterGui = game:GetService("StarterGui")

-- [[ نظام الحفظ للإشعار الجديد ]] --
local updateFileName = "SaadHub_Safe_V102_Jitter.json"
local function shouldNotifyUpdate()
    local success, content = pcall(function() return readfile(updateFileName) end)
    if success and content == "done" then return false end
    pcall(function() writefile(updateFileName, "done") end)
    return true
end
local isFirstUpdateNotify = shouldNotifyUpdate()

-- [[ نظام الحفظ الأصلي ]] --
local fileName = "SaadHub_Global_Check.json"
local function shouldNotify()
    local success, content = pcall(function() return readfile(fileName) end)
    if success and content == "done" then return false end
    pcall(function() writefile(fileName, "done") end)
    return true
end
local isFirstTime = shouldNotify()

-- [[ 1. نظام العداد (Live Users) ]] --
local liveCount = "1"
task.spawn(function()
    pcall(function() 
        local response = game:HttpGet("https://api.counterapi.dev/v1/saadhub_official_unique/hits/up")
        local data = httpService:JSONDecode(response)
        if data and data.count then liveCount = tostring(data.count) end
    end)
end)

-- [[ إرسال الإشعار ]] --
task.spawn(function()
    if isFirstUpdateNotify then
        starterGui:SetCore("SendNotification", {
            Title = "SHIELD ACTIVE 🛡️",
            Text = "تم تفعيل السرعة القصوى مع الحماية!",
            Icon = "rbxassetid://13054812323",
            Duration = 6
        })
    end
end)

-- [[ 4. واجهة التحكم ]] --
local mainGui = Instance.new("ScreenGui", player.PlayerGui); mainGui.ResetOnSpawn = false
local toggle = Instance.new("TextButton", mainGui)
toggle.Size = UDim2.new(0, 140, 0, 45); toggle.Position = UDim2.new(0.05, 0, 0.4, 0); toggle.Text = "SAADHUB: ON"
toggle.BackgroundColor3 = Color3.fromRGB(170, 0, 0); toggle.TextColor3 = Color3.new(1, 1, 1); toggle.Font = Enum.Font.GothamBold; toggle.TextSize = 16; Instance.new("UICorner", toggle)
Instance.new("UIStroke", toggle).Color = Color3.new(1, 1, 1)

local dragCircle = Instance.new("Frame", toggle)
dragCircle.Size = UDim2.new(0, 25, 0, 25); dragCircle.Position = UDim2.new(0.5, -12.5, 0, -32)
dragCircle.BackgroundTransparency = 1; Instance.new("UICorner", dragCircle).CornerRadius = UDim.new(1, 0)
Instance.new("UIStroke", dragCircle).Transparency = 1

local dragging, dragStart, startPos
dragCircle.InputBegan:Connect(function(input) 
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then 
        dragging = true; dragStart = input.Position; startPos = toggle.Position 
    end 
end)
userInputService.InputChanged:Connect(function(input) 
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then 
        local delta = input.Position - dragStart; 
        toggle.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) 
    end 
end)
userInputService.InputEnded:Connect(function() dragging = false end)


-- [[ 6. منطق الالتصاق واللمس السريع ]] --
local active = true
local lockedTarget = nil

local function fastTouch(targetChar, tool)
    local handle = tool:FindFirstChild("Handle") or tool:FindFirstChildOfClass("Part")
    if handle and targetChar:FindFirstChild("HumanoidRootPart") then
        local dist = (player.Character.HumanoidRootPart.Position - targetChar.HumanoidRootPart.Position).Magnitude
        if dist < 3.8 then
            task.spawn(function()
                for i = 1, 15 do
                    firetouchinterest(targetChar.HumanoidRootPart, handle, 0)
                    firetouchinterest(targetChar.HumanoidRootPart, handle, 1)
                end
            end)
        end
    end
end

-- [[ وظيفة التغيير ]] --
local function toggleScript()
    active = not active
    toggle.Text = active and "SAADHUB: ON" or "SAADHUB: OFF"
    toggle.BackgroundColor3 = active and Color3.fromRGB(170, 0, 0) or Color3.fromRGB(40, 40, 40)
    if not active then lockedTarget = nil end
end

-- تفعيل/إيقاف عن طريق كليك الماوس اليسار
userInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.UserInputType == Enum.UserInputType.MouseButton1 then
        toggleScript()
    end
end)

-- الزر اليدوي أيضاً يعمل
toggle.MouseButton1Click:Connect(toggleScript)

runService.RenderStepped:Connect(function()
    if active and player.Character and player.Character:FindFirstChild("Humanoid") then
        local bpTool = player.Backpack:FindFirstChildOfClass("Tool")
        if bpTool then player.Character.Humanoid:EquipTool(bpTool) end

        local tool = player.Character:FindFirstChildOfClass("Tool")
        if tool then
            -- التحقق مما إذا كان الهدف الحالي لا يزال صالحاً (موجود، حي، وليس لاعباً حقيقياً)
            local isPlayer = lockedTarget and game.Players:GetPlayerFromCharacter(lockedTarget)
            if not lockedTarget or not lockedTarget.Parent or lockedTarget.Humanoid.Health <= 0 or isPlayer then
                local cDist = math.huge; lockedTarget = nil
                
                -- البحث في Workspace عن أي مودل غير اللاعبين
                for _, v in pairs(workspace:GetDescendants()) do
                    if v:IsA("Model") and v ~= player.Character and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") and v.Humanoid.Health > 0 then
                        -- هذا الشرط يتأكد أن الهدف ليس ضمن قائمة اللاعبين الحقيقيين (أي أنه بوت)
                        if not game.Players:GetPlayerFromCharacter(v) then
                            local d = (player.Character.HumanoidRootPart.Position - v.HumanoidRootPart.Position).Magnitude
                            if d < cDist then 
                                cDist = d
                                lockedTarget = v 
                            end
                        end
                    end
                end
            end
            
            if lockedTarget and lockedTarget:FindFirstChild("HumanoidRootPart") then
                local targetRoot = lockedTarget.HumanoidRootPart
                local myRoot = player.Character.HumanoidRootPart
                local dist = (targetRoot.Position - myRoot.Position).Magnitude
                
                -- [[ ميزة التذبذب الذكي المطلوبة: بين 2.5 و 3.5 ]] --
                local smartOffset = math.random(25, 35) / 10 
                
                if dist > smartOffset then
                    player.Character.Humanoid:Move((targetRoot.Position - myRoot.Position).Unit, false) 
                end

                fastTouch(lockedTarget, tool)
            end
        else 
            lockedTarget = nil 
        end
    end
end)

