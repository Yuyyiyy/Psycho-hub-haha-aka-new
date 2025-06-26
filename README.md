-- SERVICES
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- COLORS
local darkNavyBlue = Color3.fromRGB(10, 20, 40)
local darkGold = Color3.fromRGB(180, 140, 30)
local defaultBtnColor = Color3.fromRGB(25, 25, 45)
local standardBtnColor = Color3.fromRGB(30, 30, 60)
local whiteText = Color3.fromRGB(255, 255, 255)

-- STATES
local toggleOpen = false
local loopList = {}
local playerButtons = {}
local speedBoostOn = false
local frontLoopMode = false
local loopbringAllActive = false
local attachModeOn = false
local twoSidePositionOn = false
local myDeaths = 0

-- GUI SETUP
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "PsychoLoopbring"
gui.ResetOnSpawn = false

local toggle = Instance.new("TextButton", gui)
toggle.Size = UDim2.new(0, 140, 0, 30)
toggle.Position = UDim2.new(0, 10, 0, 10)
toggle.BackgroundColor3 = darkNavyBlue
toggle.TextColor3 = darkGold
toggle.Text = "Psycho Loopbring"
toggle.Font = Enum.Font.GothamBold
toggle.TextSize = 14
toggle.BorderSizePixel = 0

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 240, 0, 380)
frame.Position = UDim2.new(0, 10, 0, 50)
frame.BackgroundColor3 = darkNavyBlue
frame.BorderSizePixel = 0
frame.Visible = false
frame.Active = true
frame.Draggable = true

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, -20, 0, 30)
title.Position = UDim2.new(0, 10, 0, 5)
title.BackgroundTransparency = 1
title.Text = "Psycho Loopbring"
title.TextColor3 = darkGold
title.TextScaled = true
title.Font = Enum.Font.FredokaOne
title.TextXAlignment = Enum.TextXAlignment.Left

local scroll = Instance.new("ScrollingFrame", frame)
scroll.Position = UDim2.new(0, 10, 0, 40)
scroll.Size = UDim2.new(1, -20, 1, -50)
scroll.BackgroundTransparency = 1
scroll.BorderSizePixel = 0
scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
scroll.ScrollBarThickness = 5
scroll.ClipsDescendants = true

local layout = Instance.new("UIListLayout", scroll)
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Padding = UDim.new(0, 3)

-- UTILITIES
local function tweenColor(object, color, duration)
    TweenService:Create(object, TweenInfo.new(duration or 0.25), {BackgroundColor3 = color}):Play()
end

local function setNoClip(character, state)
    for _, part in pairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanCollide = not state
        end
    end
end

-- TOGGLE UI
toggle.MouseButton1Click:Connect(function()
    toggleOpen = not toggleOpen
    frame.Visible = toggleOpen
end)

-- SPEED BOOST BUTTON
local speedButton = Instance.new("TextButton", scroll)
speedButton.Size = UDim2.new(1, 0, 0, 28)
speedButton.BackgroundColor3 = standardBtnColor
speedButton.TextColor3 = whiteText
speedButton.Font = Enum.Font.GothamBold
speedButton.TextSize = 13
speedButton.Text = "Speed Boost: OFF"
speedButton.LayoutOrder = 1

local function applySpeedBoost(state)
    local char = LocalPlayer.Character
    if char then
        local humanoid = char:FindFirstChildOfClass("Humanoid")
        if humanoid then
            if state then
                humanoid.WalkSpeed = 120
                humanoid.JumpPower = 120
                speedButton.Text = "Speed Boost: ON"
                tweenColor(speedButton, darkGold)
            else
                humanoid.WalkSpeed = 16
                humanoid.JumpPower = 50
                speedButton.Text = "Speed Boost: OFF"
                tweenColor(speedButton, standardBtnColor)
            end
        end
    end
end

speedButton.MouseButton1Click:Connect(function()
    speedBoostOn = not speedBoostOn
    applySpeedBoost(speedBoostOn)
end)

-- ATTACH MODE BUTTON
local attachModeButton = Instance.new("TextButton", scroll)
attachModeButton.Size = UDim2.new(1, 0, 0, 28)
attachModeButton.BackgroundColor3 = standardBtnColor
attachModeButton.TextColor3 = whiteText
attachModeButton.Font = Enum.Font.GothamBold
attachModeButton.TextSize = 13
attachModeButton.Text = "Attach Mode: OFF"
attachModeButton.LayoutOrder = 2

attachModeButton.MouseButton1Click:Connect(function()
    attachModeOn = not attachModeOn
    attachModeButton.Text = attachModeOn and "Attach Mode: ON" or "Attach Mode: OFF"
    tweenColor(attachModeButton, attachModeOn and darkGold or standardBtnColor)
end)

-- LOOPBRING ALL BUTTON
local loopbringAllButton = Instance.new("TextButton", scroll)
loopbringAllButton.Size = UDim2.new(1, 0, 0, 28)
loopbringAllButton.BackgroundColor3 = standardBtnColor
loopbringAllButton.TextColor3 = whiteText
loopbringAllButton.Font = Enum.Font.GothamBold
loopbringAllButton.TextSize = 13
loopbringAllButton.Text = "Loopbring All: OFF"
loopbringAllButton.LayoutOrder = 3

loopbringAllButton.MouseButton1Click:Connect(function()
    loopbringAllActive = not loopbringAllActive
    loopbringAllButton.Text = loopbringAllActive and "Loopbring All: ON" or "Loopbring All: OFF"
    tweenColor(loopbringAllButton, loopbringAllActive and darkGold or standardBtnColor)
    
    for name, btn in pairs(playerButtons) do
        if Players:FindFirstChild(name) and name ~= LocalPlayer.Name then
            loopList[name] = loopbringAllActive or nil
            tweenColor(btn, loopbringAllActive and darkGold or defaultBtnColor)
        end
    end
end)

-- MY DEATHS LABEL
local myDeathLabel = Instance.new("TextLabel", scroll)
myDeathLabel.Size = UDim2.new(1, 0, 0, 28)
myDeathLabel.BackgroundColor3 = standardBtnColor
myDeathLabel.Text = "My Deaths: 0"
myDeathLabel.TextColor3 = Color3.new(1, 0.2, 0.2)
myDeathLabel.TextScaled = true
myDeathLabel.Font = Enum.Font.GothamBold
myDeathLabel.LayoutOrder = 4

-- TWO SIDE POSITION BUTTON
local twoSidePositionButton = Instance.new("TextButton", scroll)
twoSidePositionButton.Size = UDim2.new(1, 0, 0, 28)
twoSidePositionButton.BackgroundColor3 = standardBtnColor
twoSidePositionButton.TextColor3 = whiteText
twoSidePositionButton.Font = Enum.Font.GothamBold
twoSidePositionButton.TextSize = 13
twoSidePositionButton.Text = "2 Side Position: OFF"
twoSidePositionButton.LayoutOrder = 5

twoSidePositionButton.MouseButton1Click:Connect(function()
    twoSidePositionOn = not twoSidePositionOn
    twoSidePositionButton.Text = twoSidePositionOn and "2 Side Position: ON" or "2 Side Position: OFF"
    tweenColor(twoSidePositionButton, twoSidePositionOn and darkGold or standardBtnColor)
end)

-- FRONT POSITION BUTTON
local frontPositionButton = Instance.new("TextButton", scroll)
frontPositionButton.Size = UDim2.new(1, 0, 0, 28)
frontPositionButton.BackgroundColor3 = standardBtnColor
frontPositionButton.TextColor3 = whiteText
frontPositionButton.Font = Enum.Font.GothamBold
frontPositionButton.TextSize = 13
frontPositionButton.Text = "Front Position: OFF"
frontPositionButton.LayoutOrder = 6

frontPositionButton.MouseButton1Click:Connect(function()
    frontLoopMode = not frontLoopMode
    frontPositionButton.Text = frontLoopMode and "Front Position: ON" or "Front Position: OFF"
    tweenColor(frontPositionButton, frontLoopMode and darkGold or standardBtnColor)
end)

-- LOOPBRING FUNCTIONS
local function loopbringTwoSidePosition(myHRP, activeTargets)
    for i, target in ipairs(activeTargets) do
        local char = target.Character
        if char then
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hrp then
                setNoClip(char, true)
                hrp.Velocity = Vector3.new(0, 0, 0)
                hrp.RotVelocity = Vector3.new(0, 0, 0)

                local side = (i % 2 == 1) and -1 or 1
                local sideOffset = 0.5  -- Adjusted to 0.5 studs (to make it 1 stud distance between two targets)
                local forwardOffset = 1  -- Adjusted to 1 stud distance to make it 1 stud apart

                local targetPos = myHRP.Position + 
                    myHRP.CFrame.RightVector * (side * sideOffset) + 
                    myHRP.CFrame.LookVector * forwardOffset

                targetPos = Vector3.new(targetPos.X, myHRP.Position.Y, targetPos.Z)
                hrp.CFrame = CFrame.new(targetPos) * CFrame.Angles(0, math.rad(myHRP.Orientation.Y), 0)

                -- Kill anyone touching the tool
                local humanoid = hrp.Parent:FindFirstChild("Humanoid")
                if humanoid then
                    humanoid.Health = 0 -- Kill the player
                end
            end
        end
    end
end

local function loopbringStandard(myHRP, name, target)
    if target and target.Character then
        local hrp = target.Character:FindFirstChild("HumanoidRootPart")
        if hrp then
            setNoClip(target.Character, true)
            hrp.Velocity = Vector3.new(0, 0, 0)
            hrp.RotVelocity = Vector3.new(0, 0, 0)
            
            local offset
            if frontLoopMode then
                offset = myHRP.CFrame.LookVector * 3 + Vector3.new(0, 1, 0)
            else
                offset = myHRP.CFrame.RightVector * 3 + myHRP.CFrame.LookVector + Vector3.new(0, 1, 0)
            end
            
            hrp.CFrame = myHRP.CFrame + offset
        end
    end
end

-- ADD PLAYER BUTTONS
local function addPlayerButton(targetPlayer)
    if targetPlayer == LocalPlayer then return end

    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, 0, 0, 28)
    button.BackgroundColor3 = defaultBtnColor
    button.TextColor3 = whiteText
    button.Font = Enum.Font.GothamBold
    button.TextSize = 13
    button.Text = targetPlayer.Name
    button.LayoutOrder = 100
    button.Parent = scroll

    playerButtons[targetPlayer.Name] = button

    -- Kill tracker label
    local killLabel = Instance.new("TextLabel", button)
    killLabel.Size = UDim2.new(0, 60, 1, 0)
    killLabel.Position = UDim2.new(1, -65, 0, 0)
    killLabel.BackgroundTransparency = 1
    killLabel.TextColor3 = Color3.new(1, 0.2, 0.2)
    killLabel.Font = Enum.Font.GothamBold
    killLabel.TextSize = 12
    killLabel.TextXAlignment = Enum.TextXAlignment.Right
    killLabel.Text = "Kills: 0"

    local kills = 0

    -- Track kills
    local function onCharacterAdded(char)
        local humanoid = char:WaitForChild("Humanoid", 10)
        if humanoid then
            humanoid.Died:Connect(function()
                local killerTool = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Tool")
                if killerTool then
                    kills = kills + 1
                    killLabel.Text = "Kills: " .. kills
                end
            end)
        end
    end

    if targetPlayer.Character then
        onCharacterAdded(targetPlayer.Character)
    end
    targetPlayer.CharacterAdded:Connect(onCharacterAdded)

    button.MouseButton1Click:Connect(function()
        local name = targetPlayer.Name
        loopList[name] = not loopList[name]
        tweenColor(button, loopList[name] and darkGold or defaultBtnColor)
    end)
end

-- Initialize player buttons
for _, player in ipairs(Players:GetPlayers()) do
    addPlayerButton(player)
end
Players.PlayerAdded:Connect(addPlayerButton)

-- Track local player deaths
LocalPlayer.CharacterAdded:Connect(function(char)
    local humanoid = char:WaitForChild("Humanoid", 10)
    if humanoid then
        humanoid.Died:Connect(function()
            myDeaths = myDeaths + 1
            myDeathLabel.Text = "My Deaths: " .. myDeaths
        end)
    end
    
    -- Reapply speed boost if it was on
    if speedBoostOn then
        applySpeedBoost(true)
    end
end)

-- MAIN LOOPBRING EXECUTION WITH ATTACH MODE
task.spawn(function()
    while true do
        local myHRP = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if myHRP then
            -- Get active targets
            local activeTargets = {}
            for name, active in pairs(loopList) do
                if active then
                    local target = Players:FindFirstChild(name)
                    if target then
                        table.insert(activeTargets, target)
                    end
                end
            end
            
            -- Execute based on mode priority
            if attachModeOn then
                -- ATTACH MODE: Move local player to target
                attachModeExecution(myHRP, activeTargets)
            elseif twoSidePositionOn then
                -- TWO SIDE POSITION MODE
                loopbringTwoSidePosition(myHRP, activeTargets)
            else
                -- STANDARD LOOPBRING MODE
                for name, active in pairs(loopList) do
                    if active then
                        local target = Players:FindFirstChild(name)
                        loopbringStandard(myHRP, name, target)
                    end
                end
            end
        end
        
        task.wait(0.1)
    end
end)
