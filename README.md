--// Serviços
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

--// Configurações
local AimbotEnabled = false
local FovEnabled = false
local HoldingRightMouse = false
local FovRadius = 120

--// UI PRINCIPAL
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AimbotUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 260, 0, 160)
MainFrame.Position = UDim2.new(0.5, -130, 0, 20)
MainFrame.BackgroundColor3 = Color3.fromRGB(20,20,20)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner", MainFrame)
UICorner.CornerRadius = UDim.new(0, 12)

--// TÍTULO
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.Text = "AIMBOT"
Title.TextColor3 = Color3.fromRGB(255,255,255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 24
Title.Parent = MainFrame

--// FUNÇÃO DE BOTÃO
local function CreateButton(text, posY)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0, 220, 0, 32)
	btn.Position = UDim2.new(0.5, -110, 0, posY)
	btn.BackgroundColor3 = Color3.fromRGB(35,35,35)
	btn.TextColor3 = Color3.fromRGB(255,255,255)
	btn.Text = text
	btn.Font = Enum.Font.Gotham
	btn.TextSize = 14
	btn.Parent = MainFrame

	local c = Instance.new("UICorner", btn)
	c.CornerRadius = UDim.new(0, 8)

	return btn
end

--// BOTÕES
local AimbotButton = CreateButton("Aimbot: OFF", 50)
local FovButton = CreateButton("Círculo: OFF", 90)

--// BOTÕES + e -
local MinusButton = Instance.new("TextButton")
MinusButton.Size = UDim2.new(0, 40, 0, 32)
MinusButton.Position = UDim2.new(0, 20, 0, 130)
MinusButton.Text = "-"
MinusButton.TextSize = 20
MinusButton.BackgroundColor3 = Color3.fromRGB(35,35,35)
MinusButton.TextColor3 = Color3.new(1,1,1)
MinusButton.Parent = MainFrame
Instance.new("UICorner", MinusButton)

local PlusButton = MinusButton:Clone()
PlusButton.Text = "+"
PlusButton.Position = UDim2.new(1, -60, 0, 130)
PlusButton.Parent = MainFrame

--// CÍRCULO (FOV)
local FovCircle = Instance.new("Frame")
FovCircle.AnchorPoint = Vector2.new(0.5, 0.5)
FovCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
FovCircle.Size = UDim2.new(0, FovRadius*2, 0, FovRadius*2)
FovCircle.BackgroundTransparency = 1
FovCircle.Visible = false
FovCircle.Parent = ScreenGui

local Stroke = Instance.new("UIStroke", FovCircle)
Stroke.Thickness = 2
Stroke.Color = Color3.fromRGB(255,255,255)

local Corner = Instance.new("UICorner", FovCircle)
Corner.CornerRadius = UDim.new(1,0)

--// INPUT DO MOUSE
UIS.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		HoldingRightMouse = true
	end
end)

UIS.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		HoldingRightMouse = false
	end
end)

--// BOTÕES LÓGICA
AimbotButton.MouseButton1Click:Connect(function()
	AimbotEnabled = not AimbotEnabled
	AimbotButton.Text = "Aimbot: " .. (AimbotEnabled and "ON" or "OFF")
end)

FovButton.MouseButton1Click:Connect(function()
	FovEnabled = not FovEnabled
	FovCircle.Visible = FovEnabled
	FovButton.Text = "Círculo: " .. (FovEnabled and "ON" or "OFF")
end)

MinusButton.MouseButton1Click:Connect(function()
	FovRadius = math.clamp(FovRadius - 10, 50, 400)
	FovCircle.Size = UDim2.new(0, FovRadius*2, 0, FovRadius*2)
end)

PlusButton.MouseButton1Click:Connect(function()
	FovRadius = math.clamp(FovRadius + 10, 50, 400)
	FovCircle.Size = UDim2.new(0, FovRadius*2, 0, FovRadius*2)
end)

--// FUNÇÃO PARA PEGAR PLAYER MAIS PRÓXIMO DO CENTRO
local function GetClosestPlayer()
	local closest
	local shortest = math.huge
	local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
			local head = player.Character.Head
			local pos, visible = Camera:WorldToViewportPoint(head.Position)
			if visible then
				local dist = (Vector2.new(pos.X, pos.Y) - center).Magnitude
				if (not FovEnabled or dist <= FovRadius) and dist < shortest then
					shortest = dist
					closest = head
				end
			end
		end
	end

	return closest
end

--// LOOP DO AIMBOT
RunService.RenderStepped:Connect(function()
	if AimbotEnabled and HoldingRightMouse then
		local target = GetClosestPlayer()
		if target then
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Position)
		end
	end
end)
