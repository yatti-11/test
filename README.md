# test
-- yatti fly gui (Mobile / Local only)

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local humanoid = char:WaitForChild("Humanoid")

-- ===== 状態 =====
local flying = false
local flySpeed = 50
local upHeld = false
local downHeld = false
local bodyGyro, bodyVel

-- ===== GUI =====
local gui = Instance.new("ScreenGui")
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local main = Instance.new("Frame")
main.Size = UDim2.fromOffset(300, 160) -- 🔽 小さく
main.Position = UDim2.fromScale(0.5, 0.5)
main.AnchorPoint = Vector2.new(0.5, 0.5)
main.BackgroundColor3 = Color3.fromRGB(40,40,40)
main.Parent = gui

-- ===== タイトル（ここだけドラッグ可）=====
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30) -- 🔽
title.BackgroundColor3 = Color3.fromRGB(60,60,60)
title.Text = "yatti fly gui"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 16 -- 🔽
title.Parent = main

-- ドラッグ
do
	local dragging, dragStart, startPos
	title.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			dragStart = input.Position
			startPos = main.Position
		end
	end)

	title.InputEnded:Connect(function()
		dragging = false
	end)

	UIS.InputChanged:Connect(function(input)
		if dragging and input.UserInputType == Enum.UserInputType.Touch then
			local delta = input.Position - dragStart
			main.Position = UDim2.new(
				startPos.X.Scale,
				startPos.X.Offset + delta.X,
				startPos.Y.Scale,
				startPos.Y.Offset + delta.Y
			)
		end
	end)
end

-- ===== 閉じる & 最小化 =====
local close = Instance.new("TextButton")
close.Size = UDim2.fromOffset(30,30) -- 🔽
close.Position = UDim2.fromOffset(0,0)
close.Text = "X"
close.BackgroundColor3 = Color3.fromRGB(200,60,60)
close.TextColor3 = Color3.new(1,1,1)
close.Parent = title

local mini = Instance.new("TextButton")
mini.Size = UDim2.fromOffset(30,30) -- 🔽
mini.Position = UDim2.fromOffset(270,0) -- main幅-30
mini.Text = "-"
mini.BackgroundColor3 = Color3.fromRGB(180,140,220)
mini.TextColor3 = Color3.new(0,0,0)
mini.Parent = title

local minimized = false
mini.MouseButton1Click:Connect(function()
	minimized = not minimized
	for _,v in ipairs(main:GetChildren()) do
		if v ~= title then
			v.Visible = not minimized
		end
	end
end)

close.MouseButton1Click:Connect(function()
	gui:Destroy()
end)

-- ===== ボタン生成関数 =====
local function makeBtn(text, pos, color)
	local b = Instance.new("TextButton")
	b.Size = UDim2.fromOffset(60,45) -- 🔽
	b.Position = pos
	b.Text = text
	b.BackgroundColor3 = color
	b.TextColor3 = Color3.new(0,0,0)
	b.Font = Enum.Font.SourceSansBold
	b.TextSize = 16 -- 🔽
	b.Parent = main
	return b
end

local upBtn    = makeBtn("アップ", UDim2.fromOffset(5,35),   Color3.fromRGB(150,255,150))
local downBtn  = makeBtn("ダウン", UDim2.fromOffset(5,85),   Color3.fromRGB(150,200,255))
local plusBtn  = makeBtn("+",     UDim2.fromOffset(85,35),  Color3.fromRGB(150,150,255))
local minusBtn = makeBtn("-",     UDim2.fromOffset(85,85),  Color3.fromRGB(150,255,255))
local speedLbl = makeBtn("1",     UDim2.fromOffset(155,85), Color3.fromRGB(255,120,120))
local flyBtn   = makeBtn("fly",   UDim2.fromOffset(215,85), Color3.fromRGB(255,255,120))

-- ===== Fly処理 =====
local function startFly()
	if flying then return end
	flying = true

	bodyGyro = Instance.new("BodyGyro")
	bodyGyro.P = 9e4
	bodyGyro.MaxTorque = Vector3.new(9e9,9e9,9e9)
	bodyGyro.CFrame = hrp.CFrame
	bodyGyro.Parent = hrp

	bodyVel = Instance.new("BodyVelocity")
	bodyVel.MaxForce = Vector3.new(9e9,9e9,9e9)
	bodyVel.Parent = hrp

	RunService.RenderStepped:Connect(function()
		if not flying then return end

		local cam = workspace.CurrentCamera
		bodyGyro.CFrame = cam.CFrame

		local move = humanoid.MoveDirection * flySpeed
		local y = 0
		if upHeld then y = flySpeed end
		if downHeld then y = -flySpeed end

		bodyVel.Velocity = Vector3.new(move.X, y, move.Z)
	end)
end

local function stopFly()
	flying = false
	if bodyGyro then bodyGyro:Destroy() end
	if bodyVel then bodyVel:Destroy() end
end

flyBtn.MouseButton1Click:Connect(function()
	if flying then stopFly() else startFly() end
end)

-- ===== 押しっぱなし対応 =====
local function hold(btn, onStart, onEnd)
	btn.MouseButton1Down:Connect(onStart)
	btn.MouseButton1Up:Connect(onEnd)
	btn.TouchLongPress:Connect(function(_, state)
		if state == Enum.LongPressState.Begin then onStart() end
		if state == Enum.LongPressState.End then onEnd() end
	end)
end

hold(upBtn, function() upHeld = true end, function() upHeld = false end)
hold(downBtn, function() downHeld = true end, function() downHeld = false end)

plusBtn.MouseButton1Click:Connect(function()
	flySpeed += 10
	speedLbl.Text = tostring(math.floor(flySpeed/10))
end)

minusBtn.MouseButton1Click:Connect(function()
	flySpeed = math.max(10, flySpeed - 10)
	speedLbl.Text = tostring(math.floor(flySpeed/10))
end)