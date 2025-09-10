-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer

-- Variáveis
local platform
local platformSize = Vector3.new(10, 1, 10)
local followOffset = Vector3.new(0, -3, 0)
local floating = false
local imortal = false

local currentCharacter = player.Character or player.CharacterAdded:Wait()
local rootPart = currentCharacter:WaitForChild("HumanoidRootPart")
local humanoid = currentCharacter:WaitForChild("Humanoid")

-- Função para aplicar imortalidade
local function enableImortalidade()
    imortal = true
    spawn(function()
        while imortal do
            local character = player.Character
            if character and character:FindFirstChild("Humanoid") then
                local humanoid = character.Humanoid
                if humanoid.Health < humanoid.MaxHealth then
                    humanoid.Health = humanoid.MaxHealth
                end
            end
            wait(0.1)
        end
    end)
end

-- Função para atualizar referências após reset
local function updateCharacterRefs(character)
    rootPart = character:WaitForChild("HumanoidRootPart")
    humanoid = character:WaitForChild("Humanoid")
    if imortal then
        enableImortalidade()
    end
end

-- Detecta reset do personagem
player.CharacterAdded:Connect(function(character)
    currentCharacter = character
    updateCharacterRefs(character)
end)

-- GUI Setup
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 200, 0, 100)
frame.Position = UDim2.new(0.5, -100, 0.5, -50)
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
frame.Active = true
frame.Draggable = true

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 30)
title.Position = UDim2.new(0, 0, 0, 0)
title.Text = "TIKTOK:EPPX_HUB"
title.TextScaled = true
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(255, 0, 0)

local button = Instance.new("TextButton", frame)
button.Size = UDim2.new(0.8, 0, 0, 25)
button.Position = UDim2.new(0.1, 0, 0.5, -12)
button.Text = "Escript"
button.TextScaled = true
button.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
button.TextColor3 = Color3.fromRGB(255, 255, 255)

-- Tween título
spawn(function()
    while true do
        local randomColor = Color3.fromHSV(math.random(), 1, 1)
        TweenService:Create(title, TweenInfo.new(0.6, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut), {TextColor3 = randomColor}):Play()
        wait(0.6)
    end
end)

-- Tween plataforma
local function tweenPlatformColor(part)
    spawn(function()
        while part and part.Parent do
            local randomColor = Color3.fromHSV(math.random(), 1, 1)
            TweenService:Create(part, TweenInfo.new(1, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut), {Color = randomColor}):Play()
            wait(1)
        end
    end)
end

-- Criar plataforma
local function createPlatform()
    platform = Instance.new("Part")
    platform.Size = platformSize
    platform.Anchored = true
    platform.CanCollide = true
    platform.Position = rootPart.Position + followOffset
    platform.Parent = workspace

    tweenPlatformColor(platform)
    floating = true
    imortal = true
    enableImortalidade()

    spawn(function()
        while floating and platform and platform.Parent do
            local character = player.Character
            if character and character:FindFirstChild("HumanoidRootPart") then
                local rp = character.HumanoidRootPart
                platform.CFrame = CFrame.new(rp.Position + followOffset)

                local desiredHeight = platform.Position.Y + platform.Size.Y/2 + 2
                local currentPos = rp.Position
                local newY = currentPos.Y + (desiredHeight - currentPos.Y) * 0.1
                rp.CFrame = CFrame.new(currentPos.X, newY, currentPos.Z)
            end
            wait(0.03)
        end
    end)
end

-- Remover plataforma
local function removePlatform()
    if platform and platform.Parent then
        floating = false
        imortal = false
        platform:Destroy()
        platform = nil
    end
end

-- Botão alterna entre criar/remover plataforma
button.MouseButton1Click:Connect(function()
    if platform and platform.Parent then
        removePlatform()
    else
        createPlatform()
    end
end)
