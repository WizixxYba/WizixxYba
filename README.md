## Hi there 👋
Roblox ESP Script
```local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer

local ESP_ENABLED = true
local ESP_SCALE = 1

-- ================== GUI ==================
local screenGui = Instance.new("ScreenGui")
screenGui.ResetOnSpawn = false
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- ================== STATUS ==================
local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.fromOffset(200, 50)
statusLabel.AnchorPoint = Vector2.new(0.5, 0.5)
statusLabel.Position = UDim2.fromScale(0.5, 0.5)
statusLabel.BackgroundTransparency = 1
statusLabel.TextScaled = true
statusLabel.Font = Enum.Font.GothamBold
statusLabel.TextStrokeTransparency = 0
statusLabel.Visible = false
statusLabel.Parent = screenGui

local function showStatus(text, color)
	statusLabel.Text = text
	statusLabel.TextColor3 = color
	statusLabel.TextTransparency = 0
	statusLabel.TextStrokeTransparency = 0
	statusLabel.Visible = true

	TweenService:Create(
		statusLabel,
		TweenInfo.new(2.5),
		{TextTransparency = 1, TextStrokeTransparency = 1}
	):Play()
end

-- ================== WATERMARK ==================
local watermark = Instance.new("TextLabel")
watermark.Size = UDim2.fromOffset(120, 20)
watermark.Position = UDim2.fromScale(1, 1)
watermark.AnchorPoint = Vector2.new(1, 1)
watermark.BackgroundTransparency = 1
watermark.Text = "WizixxYba"
watermark.Font = Enum.Font.GothamBold
watermark.TextScaled = true
watermark.TextColor3 = Color3.fromRGB(180,180,180)
watermark.Parent = screenGui

-- ================== FRIEND CHECK ==================
local function isFriend(player)
	local ok, res = pcall(function()
		return LocalPlayer:IsFriendsWith(player.UserId)
	end)
	return ok and res
end

-- ================== CREATE ESP ==================
local function createESP(player, character)
	if not ESP_ENABLED then return end
	if character:FindFirstChild("FullESP") then return end

	local humanoid = character:WaitForChild("Humanoid")
	local head = character:WaitForChild("Head")
	local root = character:WaitForChild("HumanoidRootPart")

	local folder = Instance.new("Folder")
	folder.Name = "FullESP"
	folder.Parent = character

	local highlight = Instance.new("Highlight")
	highlight.Adornee = character
	highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
	highlight.FillTransparency = 0.45
	highlight.OutlineColor = Color3.new(1,1,1)
	highlight.FillColor = isFriend(player)
		and Color3.fromRGB(0,170,255)
		or Color3.fromRGB(255,80,80)
	highlight.Parent = folder

	-- NAME
	local nameGui = Instance.new("BillboardGui")
	nameGui.Name = "NameGui"
	nameGui.Size = UDim2.fromOffset(120 * ESP_SCALE, 22 * ESP_SCALE)
	nameGui.StudsOffset = Vector3.new(0, 1.5, 0)
	nameGui.AlwaysOnTop = true
	nameGui.Adornee = head
	nameGui.Parent = folder

	local nameLabel = Instance.new("TextLabel")
	nameLabel.Size = UDim2.fromScale(1,1)
	nameLabel.BackgroundTransparency = 1
	nameLabel.Text = player.Name
	nameLabel.TextScaled = true
	nameLabel.Font = Enum.Font.GothamBold
	nameLabel.TextStrokeTransparency = 0.3
	nameLabel.TextColor3 = isFriend(player)
		and Color3.fromRGB(0,170,255)
		or Color3.new(1,1,1)
	nameLabel.Parent = nameGui

	-- HP
	local hpGui = Instance.new("BillboardGui")
	hpGui.Name = "HpGui"
	hpGui.Size = UDim2.fromOffset(6 * ESP_SCALE, 40 * ESP_SCALE)
	hpGui.StudsOffset = Vector3.new(1.9, 0.2, 0)
	hpGui.AlwaysOnTop = true
	hpGui.Adornee = root
	hpGui.Parent = folder

	local bg = Instance.new("Frame")
	bg.Size = UDim2.fromScale(1,1)
	bg.BackgroundColor3 = Color3.fromRGB(30,30,30)
	bg.BorderSizePixel = 0
	bg.Parent = hpGui

	local bar = Instance.new("Frame")
	bar.AnchorPoint = Vector2.new(0,1)
	bar.Position = UDim2.fromScale(0,1)
	bar.BackgroundColor3 = Color3.fromRGB(0,255,0)
	bar.BorderSizePixel = 0
	bar.Parent = bg

	local function updateHP()
		local p = math.clamp(humanoid.Health / humanoid.MaxHealth,0,1)
		bar.Size = UDim2.fromScale(1,p)
	end
	updateHP()
	humanoid.HealthChanged:Connect(updateHP)
end

-- ================== REMOVE ==================
local function removeESP()
	for _,p in ipairs(Players:GetPlayers()) do
		if p.Character then
			local e = p.Character:FindFirstChild("FullESP")
			if e then e:Destroy() end
		end
	end
end

-- ================== SIZE UPDATE ==================
local function updateSize()
	for _,p in ipairs(Players:GetPlayers()) do
		if p.Character then
			local esp = p.Character:FindFirstChild("FullESP")
			if esp then
				local n = esp:FindFirstChild("NameGui")
				local h = esp:FindFirstChild("HpGui")
				if n then n.Size = UDim2.fromOffset(120*ESP_SCALE,22*ESP_SCALE) end
				if h then h.Size = UDim2.fromOffset(6*ESP_SCALE,40*ESP_SCALE) end
			end
		end
	end
end

-- ================== PLAYERS ==================
local function onPlayer(p)
	if p == LocalPlayer then return end
	if p.Character then createESP(p,p.Character) end
	p.CharacterAdded:Connect(function(c)
		task.wait(0.5)
		createESP(p,c)
	end)
end

for _,p in ipairs(Players:GetPlayers()) do onPlayer(p) end
Players.PlayerAdded:Connect(onPlayer)

-- ================== TOGGLE L ==================
UserInputService.InputBegan:Connect(function(i,gp)
	if gp then return end
	if i.KeyCode == Enum.KeyCode.L then
		ESP_ENABLED = not ESP_ENABLED
		if ESP_ENABLED then
			for _,p in ipairs(Players:GetPlayers()) do
				if p~=LocalPlayer and p.Character then
					createESP(p,p.Character)
				end
			end
			showStatus("ON🟢 | L to off ESP", Color3.fromRGB(80,220,80))
		else
			removeESP()
			showStatus("OFF🔴 | L to on ESP", Color3.fromRGB(220,80,80))
		end
	end
end)

-- ================== SIZE SLIDER ==================
local sizeText = Instance.new("TextLabel")
sizeText.Text = "Size"
sizeText.Size = UDim2.fromOffset(80,20)
sizeText.Position = UDim2.fromScale(0.5,0.86)
sizeText.AnchorPoint = Vector2.new(0.5,0)
sizeText.BackgroundTransparency = 1
sizeText.Font = Enum.Font.GothamBold
sizeText.TextScaled = true
sizeText.TextColor3 = Color3.new(1,1,1)
sizeText.Parent = screenGui

local slider = Instance.new("Frame")
slider.Size = UDim2.fromOffset(220,6)
slider.Position = UDim2.fromScale(0.5,0.9)
slider.AnchorPoint = Vector2.new(0.5,0)
slider.BackgroundColor3 = Color3.fromRGB(80,80,80)
slider.BorderSizePixel = 0
slider.Parent = screenGui

local knob = Instance.new("Frame")
knob.Size = UDim2.fromOffset(12,14)
knob.AnchorPoint = Vector2.new(0.5,0.5)
knob.Position = UDim2.fromScale(0.5,0.5)
knob.BackgroundColor3 = Color3.fromRGB(200,200,200)
knob.Parent = slider

local dragging = false

knob.InputBegan:Connect(function(i)
	if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true end
end)
UserInputService.InputEnded:Connect(function(i)
	if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

UserInputService.InputChanged:Connect(function(i)
	if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
		local x = math.clamp(
			(i.Position.X - slider.AbsolutePosition.X) / slider.AbsoluteSize.X,
			0.6,1.4
		)
		ESP_SCALE = x
		knob.Position = UDim2.fromScale(x/1.4,0.5)
		updateSize()
	end
end)

-- старт
showStatus("ON🟢 | L to off ESP", Color3.fromRGB(80,220,80))
```

<!--
**WizixxYba/WizixxYba** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
