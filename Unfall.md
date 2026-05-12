local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local rootPart = character:WaitForChild("HumanoidRootPart")

local unfallAtivo = false
local plataforma = nil
local conexao = nil

-- GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player:WaitForChild("PlayerGui")
screenGui.ResetOnSpawn = false

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

botao.InputBegan:Connect(function(i)
	if i.UserInputType == Enum.UserInputType.Touch
	or i.UserInputType == Enum.UserInputType.MouseButton1 then
		
		dragging = true
		dragStart = i.Position
		startPos = botao.Position
	end
end)

botao.InputChanged:Connect(function(i)
	if dragging then
		local delta = i.Position - dragStart
		
		botao.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end)

botao.InputEnded:Connect(function()
	dragging = false
end)

-- Ativar / desativar
botao.MouseButton1Click:Connect(function()

	unfallAtivo = not unfallAtivo

	if unfallAtivo then

		botao.Text = "UnFall: ON"
		botao.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

		-- Criar plataforma
		plataforma = Instance.new("Part")
		plataforma.Size = Vector3.new(5, 1, 5)
		plataforma.Transparency = 1
		plataforma.Anchored = true
		plataforma.CanCollide = true
		plataforma.Parent = workspace

		-- Seguir player
		conexao = RunService.RenderStepped:Connect(function()

			if unfallAtivo
			and rootPart
			and rootPart.Parent
			and plataforma
			and plataforma.Parent then

				-- MAIS EMBAIXO pra não empurrar pra cima
				plataforma.CFrame =
					rootPart.CFrame * CFrame.new(0, -3.5, 0)
			end
		end)

	else

		botao.Text = "UnFall: OFF"
		botao.BackgroundColor3 = Color3.fromRGB(0, 120, 255)

		-- Remover plataforma
		if plataforma then
			plataforma:Destroy()
			plataforma = nil
		end

		-- Desconectar loop
		if conexao then
			conexao:Disconnect()
			conexao = nil
		end
	end
end)

-- Respawn
player.CharacterAdded:Connect(function(newChar)
	character = newChar
	rootPart = character:WaitForChild("HumanoidRootPart")
end)
