local player = game.Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local remote = ReplicatedStorage:WaitForChild("ModMenuEvent")

local gui = script.Parent

-- BOTÃO LATERAL
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0,120,0,40)
toggleButton.Position = UDim2.new(0,20,0.5,-20)
toggleButton.Text = "MENU TESTE"
toggleButton.BackgroundColor3 = Color3.fromRGB(0,170,255)
toggleButton.TextColor3 = Color3.new(1,1,1)
toggleButton.Parent = gui

-- FRAME DO MENU
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0,220,0,260)
frame.Position = UDim2.new(0,-250,0.5,-130)
frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
frame.Parent = gui

local layout = Instance.new("UIListLayout")
layout.Parent = frame

-- ANIMAÇÃO
local open = false

local function toggleMenu()
	open = not open
	
	local goal = {}
	if open then
		goal.Position = UDim2.new(0,20,0.5,-130)
	else
		goal.Position = UDim2.new(0,-250,0.5,-130)
	end
	
	local tween = TweenService:Create(
		frame,
		TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
		goal
	)
	tween:Play()
end

toggleButton.MouseButton1Click:Connect(toggleMenu)

-- TECLA F1
UserInputService.InputBegan:Connect(function(input, processed)
	if processed then return end
	
	if input.KeyCode == Enum.KeyCode.F1 then
		toggleMenu()
	end
end)

-- FUNÇÃO BOTÃO
local function createButton(text, callback)
	local button = Instance.new("TextButton")
	button.Size = UDim2.new(1,0,0,50)
	button.Text = text .. " OFF"
	button.BackgroundColor3 = Color3.fromRGB(60,60,60)
	button.TextColor3 = Color3.new(1,1,1)
	button.Parent = frame
	
	local active = false
	
	button.MouseButton1Click:Connect(function()
		active = not active
		
		if active then
			button.Text = text .. " ON"
			button.BackgroundColor3 = Color3.fromRGB(0,170,0)
		else
			button.Text = text .. " OFF"
			button.BackgroundColor3 = Color3.fromRGB(60,60,60)
		end
		
		callback(active)
	end)
end

-- =================
-- FLY LIVRE
-- =================

local flying = false
local bodyVelocity
local bodyGyro

createButton("Fly", function(state)
	flying = state
	
	local character = player.Character
	if not character then return end
	
	local root = character:FindFirstChild("HumanoidRootPart")
	local humanoid = character:FindFirstChild("Humanoid")
	
	if state then
		humanoid:ChangeState(Enum.HumanoidStateType.Physics)
		
		bodyVelocity = Instance.new("BodyVelocity")
		bodyVelocity.MaxForce = Vector3.new(400000,400000,400000)
		bodyVelocity.Parent = root
		
		bodyGyro = Instance.new("BodyGyro")
		bodyGyro.MaxTorque = Vector3.new(400000,400000,400000)
		bodyGyro.Parent = root
	else
		humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
		if bodyVelocity then bodyVelocity:Destroy() end
		if bodyGyro then bodyGyro:Destroy() end
	end
end)

UserInputService.InputBegan:Connect(function(input)
	if not flying then return end
	
	local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
	if not root then return end
	
	if input.KeyCode == Enum.KeyCode.W then
		bodyVelocity.Velocity = root.CFrame.LookVector * 60
	elseif input.KeyCode == Enum.KeyCode.S then
		bodyVelocity.Velocity = -root.CFrame.LookVector * 60
	elseif input.KeyCode == Enum.KeyCode.A then
		bodyVelocity.Velocity = -root.CFrame.RightVector * 60
	elseif input.KeyCode == Enum.KeyCode.D then
		bodyVelocity.Velocity = root.CFrame.RightVector * 60
	elseif input.KeyCode == Enum.KeyCode.Space then
		bodyVelocity.Velocity = Vector3.new(0,60,0)
	elseif input.KeyCode == Enum.KeyCode.LeftControl then
		bodyVelocity.Velocity = Vector3.new(0,-60,0)
	end
end)

-- =================
-- HITBOX
-- =================
createButton("Hitbox", function(state)
	remote:FireServer("Hitbox", state)
end)

-- =================
-- NAMES
-- =================
createButton("Names", function(state)
	remote:FireServer("Names", state)
end)
