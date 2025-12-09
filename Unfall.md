local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local rootPart = character:WaitForChild("HumanoidRootPart")

local unfallAtivo = false
local plataforma = nil

-- GUI (botão azul arrastável)
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

-- Arrastar
local dragging = false
local dragStart, startPos
botao.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.Touch or i.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true; dragStart = i.Position; startPos = botao.Position
    end
end)
botao.InputChanged:Connect(function(i)
    if dragging then
        local delta = i.Position - dragStart
        botao.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
botao.InputEnded:Connect(function() dragging = false end)

-- Clique no botão
botao.MouseButton1Click:Connect(function()
    unfallAtivo = not unfallAtivo

    if unfallAtivo then
        botao.Text = "UnFall: ON"
        botao.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

        -- Plataforma INVISÍVEL gigante, ANCORADA (nunca falha)
        plataforma = Instance.new("Part")
        plataforma.Size = Vector3.new(2, 1, 2)
        plataforma.Transparency = 1
        plataforma.Anchored = true          -- ← Aqui o segredo (zero delay)
        plataforma.CanCollide = true
        plataforma.Parent = workspace

        -- Segue o player no RenderStepped (o mais rápido possível)
        RunService.RenderStepped:Connect(function()
            if unfallAtivo and rootPart and rootPart.Parent and plataforma and plataforma.Parent then
                plataforma.CFrame = rootPart.CFrame * CFrame.new(0, -3.5, 0)
            end
        end)

    else
        botao.Text = "UnFall: OFF"
        botao.BackgroundColor3 = Color3.fromRGB(0, 120, 255)

        if plataforma then
            plataforma:Destroy()
            plataforma = nil
        end
    end
end)

-- Respawn
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    rootPart = character:WaitForChild("HumanoidRootPart")
end)

print("UnFall INFALÍVEL pronto! Toque no botão azul.")
