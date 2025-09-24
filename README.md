local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local UIS = game:GetService("UserInputService")

-- GUI Setup
local ScreenGui = Instance.new("ScreenGui", player.PlayerGui)
ScreenGui.Name = "FlySpeedJumpGui"

local Frame = Instance.new("Frame", ScreenGui)
Frame.Position = UDim2.new(0.02,0,0.3,0)
Frame.Size = UDim2.new(0,220,0,200)
Frame.BackgroundColor3 = Color3.fromRGB(40,40,40)
Frame.BorderSizePixel = 0

local title = Instance.new("TextLabel", Frame)
title.Size = UDim2.new(1,0,0,30)
title.BackgroundTransparency = 1
title.Text = "Fly, Speed & Jump GUI"
title.TextColor3 = Color3.fromRGB(255,255,255)
title.Font = Enum.Font.GothamBold
title.TextSize = 18

-- Fly Button
local flyButton = Instance.new("TextButton", Frame)
flyButton.Position = UDim2.new(0,10,0,40)
flyButton.Size = UDim2.new(0,200,0,30)
flyButton.Text = "Toggle Fly (F)"
flyButton.BackgroundColor3 = Color3.fromRGB(60,60,60)
flyButton.TextColor3 = Color3.fromRGB(255,255,255)
flyButton.Font = Enum.Font.Gotham
flyButton.TextSize = 16

-- Speed
local speedLabel = Instance.new("TextLabel", Frame)
speedLabel.Position = UDim2.new(0,10,0,80)
speedLabel.Size = UDim2.new(0,60,0,30)
speedLabel.Text = "Speed:"
speedLabel.BackgroundTransparency = 1
speedLabel.TextColor3 = Color3.fromRGB(255,255,255)
speedLabel.Font = Enum.Font.Gotham
speedLabel.TextSize = 16

local speedBox = Instance.new("TextBox", Frame)
speedBox.Position = UDim2.new(0,80,0,80)
speedBox.Size = UDim2.new(0,80,0,30)
speedBox.Text = tostring(humanoid.WalkSpeed)
speedBox.BackgroundColor3 = Color3.fromRGB(60,60,60)
speedBox.TextColor3 = Color3.fromRGB(255,255,255)
speedBox.Font = Enum.Font.Gotham
speedBox.TextSize = 16

local speedSetButton = Instance.new("TextButton", Frame)
speedSetButton.Position = UDim2.new(0,170,0,80)
speedSetButton.Size = UDim2.new(0,40,0,30)
speedSetButton.Text = "Set"
speedSetButton.BackgroundColor3 = Color3.fromRGB(80,80,80)
speedSetButton.TextColor3 = Color3.fromRGB(255,255,255)
speedSetButton.Font = Enum.Font.Gotham
speedSetButton.TextSize = 14

-- Jump
local jumpLabel = Instance.new("TextLabel", Frame)
jumpLabel.Position = UDim2.new(0,10,0,120)
jumpLabel.Size = UDim2.new(0,60,0,30)
jumpLabel.Text = "Jump:"
jumpLabel.BackgroundTransparency = 1
jumpLabel.TextColor3 = Color3.fromRGB(255,255,255)
jumpLabel.Font = Enum.Font.Gotham
jumpLabel.TextSize = 16

local jumpBox = Instance.new("TextBox", Frame)
jumpBox.Position = UDim2.new(0,80,0,120)
jumpBox.Size = UDim2.new(0,80,0,30)
jumpBox.Text = tostring(humanoid.JumpPower)
jumpBox.BackgroundColor3 = Color3.fromRGB(60,60,60)
jumpBox.TextColor3 = Color3.fromRGB(255,255,255)
jumpBox.Font = Enum.Font.Gotham
jumpBox.TextSize = 16

local jumpSetButton = Instance.new("TextButton", Frame)
jumpSetButton.Position = UDim2.new(0,170,0,120)
jumpSetButton.Size = UDim2.new(0,40,0,30)
jumpSetButton.Text = "Set"
jumpSetButton.BackgroundColor3 = Color3.fromRGB(80,80,80)
jumpSetButton.TextColor3 = Color3.fromRGB(255,255,255)
jumpSetButton.Font = Enum.Font.Gotham
jumpSetButton.TextSize = 14

-- Fly logic
local flying = false
local flySpeed = 50
local flyConn = nil
local bv, bg = nil, nil

function startFly()
    local root = character:FindFirstChild("HumanoidRootPart")
    if not root then return end
    flying = true
    bv = Instance.new("BodyVelocity")
    bv.Velocity = Vector3.new(0,0,0)
    bv.MaxForce = Vector3.new(1,1,1) * 1e5
    bv.Parent = root

    bg = Instance.new("BodyGyro")
    bg.CFrame = root.CFrame
    bg.D = 10
    bg.MaxTorque = Vector3.new(1,1,1) * 1e5
    bg.Parent = root

    flyConn = game:GetService("RunService").RenderStepped:Connect(function()
        local cam = workspace.CurrentCamera
        local move = Vector3.new()
        if UIS:IsKeyDown(Enum.KeyCode.W) then move = move + cam.CFrame.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.S) then move = move - cam.CFrame.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.A) then move = move - cam.CFrame.RightVector end
        if UIS:IsKeyDown(Enum.KeyCode.D) then move = move + cam.CFrame.RightVector end
        bv.Velocity = move.Unit * flySpeed
        if move.Magnitude == 0 then
            bv.Velocity = Vector3.new(0,0,0)
        end
        bg.CFrame = cam.CFrame
    end)
end

function stopFly()
    flying = false
    if flyConn then flyConn:Disconnect() end
    if bv then bv:Destroy() end
    if bg then bg:Destroy() end
end

-- Button events
flyButton.MouseButton1Click:Connect(function()
    if flying then
        stopFly()
        flyButton.Text = "Toggle Fly (F)"
    else
        startFly()
        flyButton.Text = "Stop Fly (F)"
    end
end)

speedSetButton.MouseButton1Click:Connect(function()
    local val = tonumber(speedBox.Text)
    if val and val > 0 then
        humanoid.WalkSpeed = val
    end
end)

jumpSetButton.MouseButton1Click:Connect(function()
    local val = tonumber(jumpBox.Text)
    if val and val > 0 then
        humanoid.JumpPower = val
    end
end)

-- Hotkey toggle (F)
UIS.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.F then
        flyButton:Activate()
    end
end)

-- Update fields on respawn
player.CharacterAdded:Connect(function(char)
    character = char
    humanoid = character:WaitForChild("Humanoid")
    speedBox.Text = tostring(humanoid.WalkSpeed)
    jumpBox.Text = tostring(humanoid.JumpPower)
end)
