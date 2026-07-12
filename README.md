local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "BringUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = PlayerGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 320, 0, 220)
MainFrame.Position = UDim2.new(0.5, -160, 0.35, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 16)
UICorner.Parent = MainFrame

local UIStroke = Instance.new("UIStroke")
UIStroke.Color = Color3.fromRGB(60, 60, 60)
UIStroke.Thickness = 1.5
UIStroke.Parent = MainFrame

local dragging, dragInput, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = MainFrame.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

MainFrame.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		dragInput = input
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and input == dragInput then
		local delta = input.Position - dragStart
		MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
end)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.Text = "🔹 BRING & FOLLOW"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18
Title.Parent = MainFrame

local TextBox = Instance.new("TextBox")
TextBox.Size = UDim2.new(0.9, 0, 0, 36)
TextBox.Position = UDim2.new(0.05, 0, 0.22, 0)
TextBox.PlaceholderText = "Username or DisplayName"
TextBox.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
TextBox.TextColor3 = Color3.new(1, 1, 1)
TextBox.Font = Enum.Font.Gotham
TextBox.TextSize = 15
TextBox.ClearTextOnFocus = false
local tbCorner = Instance.new("UICorner")
tbCorner.CornerRadius = UDim.new(0, 10)
tbCorner.Parent = TextBox
TextBox.Parent = MainFrame

local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(0.9, 0, 0, 42)
ToggleButton.Position = UDim2.new(0.05, 0, 0.42, 0)
ToggleButton.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
ToggleButton.Text = "❌ DISABLED"
ToggleButton.TextColor3 = Color3.new(1, 1, 1)
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.TextSize = 16
local togCorner = Instance.new("UICorner")
togCorner.CornerRadius = UDim.new(0, 10)
togCorner.Parent = ToggleButton
ToggleButton.Parent = MainFrame

local Status = Instance.new("TextLabel")
Status.Size = UDim2.new(0.9, 0, 0, 28)
Status.Position = UDim2.new(0.05, 0, 0.65, 0)
Status.BackgroundTransparency = 1
Status.Text = "👤 Target: None"
Status.TextColor3 = Color3.fromRGB(180, 180, 180)
Status.Font = Enum.Font.Gotham
Status.TextSize = 14
Status.Parent = MainFrame

local RangeLabel = Instance.new("TextLabel")
RangeLabel.Size = UDim2.new(0.45, 0, 0, 20)
RangeLabel.Position = UDim2.new(0.05, 0, 0.8, 0)
RangeLabel.BackgroundTransparency = 1
RangeLabel.Text = "Distance: 6"
RangeLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
RangeLabel.Font = Enum.Font.Gotham
RangeLabel.TextSize = 13
RangeLabel.TextXAlignment = Enum.TextXAlignment.Left
RangeLabel.Parent = MainFrame

local RangeSlider = Instance.new("TextButton")
RangeSlider.Size = UDim2.new(0.45, 0, 0, 20)
RangeSlider.Position = UDim2.new(0.52, 0, 0.8, 0)
RangeSlider.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
RangeSlider.Text = ""
local rsCorner = Instance.new("UICorner")
rsCorner.CornerRadius = UDim.new(0, 8)
rsCorner.Parent = RangeSlider
RangeSlider.Parent = MainFrame

local targets = {}
local connections = {}
local visuals = {}
local rangeValue = 6
local sliderDragging = false

local function createVisualizer(plr)
	if visuals[plr] then visuals[plr]:Destroy() end
	local char = plr.Character
	if not char then return end
	local head = char:FindFirstChild("Head")
	if not head then return end
	
	local billboard = Instance.new("BillboardGui")
	billboard.Adornee = head
	billboard.Size = UDim2.new(0, 80, 0, 80)
	billboard.StudsOffset = Vector3.new(0, 2.5, 0)
	billboard.AlwaysOnTop = true
	billboard.Parent = char
	
	local xLabel = Instance.new("TextLabel")
	xLabel.Size = UDim2.new(1, 0, 1, 0)
	xLabel.BackgroundTransparency = 1
	xLabel.Text = "❌"
	xLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
	xLabel.TextScaled = true
	xLabel.Font = Enum.Font.GothamBold
	xLabel.Parent = billboard
	
	visuals[plr] = billboard
end

local function removeVisualizer(plr)
	if visuals[plr] then
		visuals[plr]:Destroy()
		visuals[plr] = nil
	end
end

local function findPlayerByPartial(query)
	if query == "" then return nil end
	query = query:lower()
	
	local bestMatch = nil
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer then
			local nameLower = plr.Name:lower()
			local displayLower = plr.DisplayName:lower()
			
			-- Priority: exact prefix match on username, then display name
			if nameLower:sub(1, #query) == query then
				return plr
			elseif displayLower:sub(1, #query) == query then
				bestMatch = plr
			elseif nameLower:find(query) then
				if not bestMatch then bestMatch = plr end
			elseif displayLower:find(query) then
				if not bestMatch then bestMatch = plr end
			end
		end
	end
	return bestMatch
end

local function getRoot(char)
	return char and char:FindFirstChild("HumanoidRootPart")
end

local function savePosition(plr)
	local root = getRoot(plr.Character)
	if root then
		if not targets[plr] then targets[plr] = {} end
		targets[plr].savedCFrame = root.CFrame
	end
end

local function restorePosition(plr)
	local data = targets[plr]
	if data and data.savedCFrame then
		local root = getRoot(plr.Character)
		if root then root.CFrame = data.savedCFrame end
	end
end

local function applyBring(plr)
	local myRoot = getRoot(LocalPlayer.Character)
	local tRoot = getRoot(plr.Character)
	if not myRoot or not tRoot then return end
	savePosition(plr)
	local goal = myRoot.CFrame * CFrame.new(0, 0, -rangeValue)
	tRoot.CFrame = goal
end

local function updateFollow(plr)
	local myRoot = getRoot(LocalPlayer.Character)
	local tRoot = getRoot(plr.Character)
	if not myRoot or not tRoot then return end
	local goal = myRoot.CFrame * CFrame.new(0, 0, -rangeValue)
	tRoot.CFrame = goal
end

local function cleanupTarget(plr)
	if connections[plr] then
		connections[plr]:Disconnect()
		connections[plr] = nil
	end
	restorePosition(plr)
	removeVisualizer(plr)
	targets[plr] = nil
end

local function setupCharacter(plr)
	createVisualizer(plr)
	plr.CharacterAdded:Connect(function()
		task.wait(0.6)
		if targets[plr] then
			applyBring(plr)
			createVisualizer(plr)
		end
	end)
	plr.CharacterRemoving:Connect(function()
		removeVisualizer(plr)
	end)
end

local function addTarget(plr)
	if targets[plr] then return end
	targets[plr] = {}
	setupCharacter(plr)
	applyBring(plr)
	
	local conn = RunService.Heartbeat:Connect(function()
		if targets[plr] then updateFollow(plr) end
	end)
	connections[plr] = conn
end

local function removeTarget(plr)
	cleanupTarget(plr)
end

local function toggleLogic()
	local input = TextBox.Text
	if input == "" then
		Status.Text = "⚠️ Enter username or display name"
		return
	end
	
	local found = findPlayerByPartial(input)
	if not found then
		Status.Text = "❌ Player not found"
		return
	end
	
	if targets[found] then
		removeTarget(found)
		Status.Text = "👤 Removed: " .. found.Name .. " (" .. found.DisplayName .. ")"
	else
		addTarget(found)
		Status.Text = "👤 Following: " .. found.Name .. " (" .. found.DisplayName .. ")"
	end
end

ToggleButton.MouseButton1Click:Connect(toggleLogic)

RangeSlider.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		sliderDragging = true
	end
end)

RangeSlider.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		sliderDragging = false
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if sliderDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
		local sliderPos = RangeSlider.AbsolutePosition
		local sliderSize = RangeSlider.AbsoluteSize
		local relX = math.clamp((input.Position.X - sliderPos.X) / sliderSize.X, 0, 1)
		rangeValue = math.floor(4 + relX * 4)
		RangeLabel.Text = "Distance: " .. rangeValue
	end
end)

TextBox.FocusLost:Connect(function(enter)
	if enter then toggleLogic() end
end)

Players.PlayerRemoving:Connect(function(plr)
	if targets[plr] then cleanupTarget(plr) end
end)

Status.Text = "Ready • Supports Username & DisplayName"
