local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

local humanoid
local rootPart

local unfallAtivo = false
local plataforma = nil
local conexao = nil

-- Atualiza personagem (R6 e R15)
local function atualizarPersonagem(char)
	character = char
	humanoid = character:WaitForChild("Humanoid")
	rootPart = character:WaitForChild("HumanoidRootPart")
end

atualizarPersonagem(character)

-- GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "UnFallGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

local botao = Instance.new("TextButton")
botao.Size = UDim2.new(0, 180, 0, 60)
botao.Position = UDim2.new(0, 20, 0, 20)
botao.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
botao.Text = "UnFall: OFF"
botao.TextColor3 = Color3.new(1,1,1)
botao.Font = Enum.Font.GothamBold
botao.TextScaled = true
botao.BorderSizePixel = 0
botao.Parent = screenGui

-- Arrastar botão
local dragging = false
local dragStart
local startPos

botao.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
	or input.UserInputType == Enum.UserInputType.Touch then
		
		dragging = true
		dragStart = input.Position
		startPos = botao.Position
	end
end)

botao.InputChanged:Connect(function(input)
	if dragging then
		local delta = input.Position - dragStart

		botao.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end)

botao.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
	or input.UserInputType == Enum.UserInputType.Touch then
		
		dragging = false
	end
end)

-- Distância baseada no tipo do rig
local function pegarOffset()
	if humanoid.RigType == Enum.HumanoidRigType.R15 then
		return -3.5
	else
		return -3
	end
end

-- Liga / desliga
botao.MouseButton1Click:Connect(function()

	unfallAtivo = not unfallAtivo

	if unfallAtivo then

		botao.Text = "UnFall: ON"
		botao.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

		-- Criar plataforma
		plataforma = Instance.new("Part")
		plataforma.Name = "UnFallPlatform"
		plataforma.Size = Vector3.new(5, 1, 5)
		plataforma.Transparency = 1
		plataforma.Anchored = true
		plataforma.CanCollide = true
		plataforma.Parent = workspace

		-- Atualizar plataforma
		conexao = RunService.RenderStepped:Connect(function()

			if not unfallAtivo then
				return
			end

			if rootPart
			and rootPart.Parent
			and plataforma
			and plataforma.Parent then

				local offset = pegarOffset()

				plataforma.CFrame =
					rootPart.CFrame * CFrame.new(0, offset, 0)
			end
		end)

	else

		botao.Text = "UnFall: OFF"
		botao.BackgroundColor3 = Color3.fromRGB(0, 120, 255)

		if conexao then
			conexao:Disconnect()
			conexao = nil
		end

		if plataforma then
			plataforma:Destroy()
			plataforma = nil
		end
	end
end)

-- Respawn
player.CharacterAdded:Connect(function(newChar)
	atualizarPersonagem(newChar)
end)
