# Freecam
Freecam para as lives da CopaTCS New Gen!
loadstring([[
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local StarterGui = game:GetService("StarterGui")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local playerGui = player:WaitForChild("PlayerGui")
local controls = require(player.PlayerScripts:WaitForChild("PlayerModule")):GetControls()

local isEnabled = false
local camRot = Vector2.new(0, 0)
local currentZoom = 70
local hiddenGuis = {}

local screenGui = Instance.new("ScreenGui", playerGui)
screenGui.Name = "CinemaCam_Freecam_Clean"
screenGui.IgnoreGuiInset = true
screenGui.ResetOnSpawn = false
screenGui.DisplayOrder = 9999999

local liveBtn = Instance.new("TextButton", screenGui)
liveBtn.Size = UDim2.new(0, 90, 0, 35)
liveBtn.Position = UDim2.new(0.5, -45, 0.9, 0)
liveBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
liveBtn.Text = "LIVE"
liveBtn.Font = Enum.Font.GothamBold
liveBtn.TextColor3 = Color3.new(1, 1, 1)
liveBtn.ZIndex = 9999999
Instance.new("UICorner", liveBtn).CornerRadius = UDim.new(0, 8)

local ghostBtn = Instance.new("TextButton", screenGui)
ghostBtn.Size = UDim2.new(0, 70, 0, 70)
ghostBtn.Position = UDim2.new(0, 60, 0, 0)
ghostBtn.BackgroundTransparency = 1
ghostBtn.Text = ""
ghostBtn.ZIndex = 9999999

ghostBtn.MouseButton1Click:Connect(function()
    liveBtn.Visible = not liveBtn.Visible
end)

local function makeDraggable(gui)
    local dragging, dragInput, dragStart, startPos
    gui.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = gui.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)
    gui.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            gui.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

makeDraggable(liveBtn)

local function setEverythingVisible(visible)
    pcall(function()
        StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.All, visible)
        StarterGui:SetCore("TopbarEnabled", visible)
    end)
    local touchGui = playerGui:FindFirstChild("TouchGui")
    if touchGui then touchGui.Enabled = visible end
    
    local character = player.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        character.HumanoidRootPart.Anchored = not visible
    end
    
    if not visible then
        for _, gui in ipairs(playerGui:GetChildren()) do
            if gui:IsA("ScreenGui") and gui ~= screenGui and not gui.Name:find("Delta") then
                if gui.Enabled then table.insert(hiddenGuis, gui) gui.Enabled = false end
            end
        end
    else
        for _, gui in ipairs(hiddenGuis) do if gui and gui.Parent then gui.Enabled = true end end
        hiddenGuis = {}
    end
end

liveBtn.MouseButton1Click:Connect(function()
    isEnabled = not isEnabled
    camera.CameraType = isEnabled and Enum.CameraType.Scriptable or Enum.CameraType.Custom
    setEverythingVisible(not isEnabled)
    liveBtn.BackgroundColor3 = isEnabled and Color3.fromRGB(255, 0, 0) or Color3.fromRGB(0, 120, 255)
    if not isEnabled then 
        camera.FieldOfView = 70 
        currentZoom = 70
    end
end)

RunService.RenderStepped:Connect(function()
    if isEnabled then
        camera.FieldOfView = camera.FieldOfView + (currentZoom - camera.FieldOfView) * 0.12
        local targetRot = CFrame.Angles(0, math.rad(camRot.Y), 0) * CFrame.Angles(math.rad(camRot.X), 0, 0)
        local moveInput = controls:GetMoveVector()
        local moveVector = targetRot:VectorToWorldSpace(Vector3.new(moveInput.X, 0, moveInput.Z) * 7)
        camera.CFrame = camera.CFrame:Lerp(CFrame.new(camera.CFrame.Position + moveVector) * targetRot, 0.12)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if isEnabled and input.UserInputType == Enum.UserInputType.Touch then
        camRot = camRot - Vector2.new(input.Delta.Y * 0.25, input.Delta.X * 0.25)
        camRot = Vector2.new(math.clamp(camRot.X, -85, 85), camRot.Y)
    end
end)

UserInputService.TouchPinch:Connect(function(_, scale)
    if isEnabled then
        if scale > 1 then currentZoom = math.max(1, currentZoom - 4)
        elseif scale < 1 then currentZoom = math.min(120, currentZoom + 4) end
    end
end)
]])()
