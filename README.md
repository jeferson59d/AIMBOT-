-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- Variáveis
local aimbotActive = false
local fovActive = false
local fovRadius = 120
local holdingRightClick = false
local menuVisible = true

-- GUI
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.ResetOnSpawn = false

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0,220,0,150)
Frame.Position = UDim2.new(0.5,-110,0.5,-75)
Frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Draggable = true

local Title = Instance.new("TextLabel", Frame)
Title.Size = UDim2.new(1,0,0,25)
Title.Text = "JEFIN Menu"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16
Title.TextColor3 = Color3.fromRGB(255,255,255)
Title.BackgroundTransparency = 1

-- Botão AIMBOT
local AimbotButton = Instance.new("TextButton", Frame)
AimbotButton.Size = UDim2.new(0,200,0,25)
AimbotButton.Position = UDim2.new(0,10,0,35)
AimbotButton.Text = "AIMBOT"
AimbotButton.Font = Enum.Font.Gotham
AimbotButton.TextSize = 14
AimbotButton.TextColor3 = Color3.fromRGB(255,255,255)
AimbotButton.BackgroundColor3 = Color3.fromRGB(0,200,0)

AimbotButton.MouseButton1Click:Connect(function()
	aimbotActive = not aimbotActive
	AimbotButton.BackgroundColor3 = aimbotActive and Color3.fromRGB(200,0,0) or Color3.fromRGB(0,200,0)
end)

-- Botão FOV
local FovButton = Instance.new("TextButton", Frame)
FovButton.Size = UDim2.new(0,200,0,25)
FovButton.Position = UDim2.new(0,10,0,65)
FovButton.Text = "FOV"
FovButton.Font = Enum.Font.Gotham
FovButton.TextSize = 14
FovButton.TextColor3 = Color3.fromRGB(255,255,255)
FovButton.BackgroundColor3 = Color3.fromRGB(0,200,0)

FovButton.MouseButton1Click:Connect(function()
	fovActive = not fovActive
	FovButton.BackgroundColor3 = fovActive and Color3.fromRGB(200,0,0) or Color3.fromRGB(0,200,0)
end)

-- Slider do FOV
local SliderBack = Instance.new("Frame", Frame)
SliderBack.Size = UDim2.new(0,200,0,6)
SliderBack.Position = UDim2.new(0,10,0,100)
SliderBack.BackgroundColor3 = Color3.fromRGB(60,60,60)

local SliderFill = Instance.new("Frame", SliderBack)
SliderFill.Size = UDim2.new(0.4,0,1,0)
SliderFill.BackgroundColor3 = Color3.fromRGB(200,0,0)

local FovValue = Instance.new("TextLabel", Frame)
FovValue.Size = UDim2.new(0,200,0,20)
FovValue.Position = UDim2.new(0,10,0,110)
FovValue.Text = "FOV: "..fovRadius
FovValue.Font = Enum.Font.Gotham
FovValue.TextSize = 14
FovValue.TextColor3 = Color3.fromRGB(255,255,255)
FovValue.BackgroundTransparency = 1

-- Slider funcional
local draggingSlider = false
SliderBack.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		draggingSlider = true
	end
end)
UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		draggingSlider = false
	end
end)
RunService.RenderStepped:Connect(function()
	if draggingSlider then
		local mouseX = UserInputService:GetMouseLocation().X
		local relativeX = math.clamp(mouseX - SliderBack.AbsolutePosition.X, 0, SliderBack.AbsoluteSize.X)
		SliderFill.Size = UDim2.new(0, relativeX, 1, 0)
		fovRadius = math.floor(50 + (relativeX / SliderBack.AbsoluteSize.X) * 250)
		FovValue.Text = "FOV: "..fovRadius
	end
end)

-- Círculo FOV
local DrawingCircle = Drawing.new("Circle")
DrawingCircle.Visible = false
DrawingCircle.Thickness = 1.5
DrawingCircle.Color = Color3.fromRGB(255,0,0)
DrawingCircle.NumSides = 64
DrawingCircle.Radius = fovRadius
DrawingCircle.Filled = false

-- Texto FOV na tela
local DrawingText = Drawing.new("Text")
DrawingText.Text = tostring(fovRadius)
DrawingText.Size = 18
DrawingText.Center = true
DrawingText.Color = Color3.fromRGB(255,0,0)
DrawingText.Visible = true

-- Input botão direito
UserInputService.InputBegan:Connect(function(input, gp)
	if not gp and input.UserInputType == Enum.UserInputType.MouseButton2 then
		holdingRightClick = true
	end
end)
UserInputService.InputEnded:Connect(function(input, gp)
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		holdingRightClick = false
	end
end)

-- Tecla H para abrir/fechar menu
UserInputService.InputBegan:Connect(function(input, gp)
	if not gp and input.KeyCode == Enum.KeyCode.H then
		menuVisible = not menuVisible
		Frame.Visible = menuVisible
	end
end)

-- Loop principal AIMBOT + FOV
RunService.RenderStepped:Connect(function()
	local cam = workspace.CurrentCamera
	local viewport = cam.ViewportSize
	local center = Vector2.new(viewport.X/2, viewport.Y/2)
	
	-- Atualiza círculo
	DrawingCircle.Position = center
	DrawingCircle.Radius = fovRadius
	DrawingCircle.Visible = fovActive
	
	-- Atualiza texto FOV
	DrawingText.Position = center
	DrawingText.Text = "FOV: "..fovRadius
	
	-- AIMBOT
	if aimbotActive and holdingRightClick then
		local closestDist = math.huge
		local target
		for _,player in ipairs(Players:GetPlayers()) do
			if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
				local part = player.Character.Head
				local screenPos, onScreen = cam:WorldToViewportPoint(part.Position)
				if onScreen then
					local dist = (Vector2.new(screenPos.X, screenPos.Y) - center).Magnitude
					if (not fovActive or dist <= fovRadius) and dist < closestDist then
						closestDist = dist
						target = part
					end
				end
			end
		end
		if target then
			cam.CFrame = CFrame.new(cam.CFrame.Position, target.Position)
		end
	end
end)
