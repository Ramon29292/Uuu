--[[
 .____                  ________ ___.    _____                           __                
 |    |    __ _______   \_____  \\_ |___/ ____\_ __  ______ ____ _____ _/  |_  ___________ 
 |    |   |  |  \__  \   /   |   \| __ \   __\  |  \/  ___// ___\\__  \\   __\/  _ \_  __ \
 |    |___|  |  // __ \_/    |    \ \_\ \  | |  |  /\___ \\  \___ / __ \|  | (  <_> )  | \/
 |_______ \____/(____  /\_______  /___  /__| |____//____  >\___  >____  /__|  \____/|__|   
         \/          \/         \/    \/                \/     \/     \/                   
          \_Welcome to LuaObfuscator.com   (Alpha 0.10.9) ~  Much Love, Ferib 
]]

--[[
  Script de Vuelo con GUI Arcoíris para Roblox (Android y PC)
  Mejorado por ChatGPT
  Instrucciones:
    1. Coloca este script en StarterGui > StarterGuiScripts.
    2. El GUI aparecerá en pantalla y podrás activar/desactivar el vuelo tocando el botón.
    3. El botón tendrá un efecto arcoíris animado.
    4. Para Android: vuela hacia donde mira la cámara (mueve la cámara para dirigir el vuelo).
       Para PC: puedes controlar vuelo con WASD/teclas de movimiento.
]]

-- Variables básicas
local player = game.Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local RS = game:GetService("RunService")

local flying = false
local flightConn -- para desconectar RenderStepped
local cam = workspace.CurrentCamera

-- Crear GUI (proteger de duplicados)
if player.PlayerGui:FindFirstChild("RainbowFlightGUI") then
    player.PlayerGui.RainbowFlightGUI:Destroy()
end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "RainbowFlightGUI"
screenGui.Parent = player:WaitForChild("PlayerGui")
screenGui.ResetOnSpawn = false

local flightButton = Instance.new("TextButton")
flightButton.Size = UDim2.new(0, 180, 0, 60)
flightButton.Position = UDim2.new(0.5, -90, 0.9, 0)
flightButton.Text = "✈️ VOLAR"
flightButton.Font = Enum.Font.GothamBold
flightButton.TextSize = 28
flightButton.TextColor3 = Color3.new(1, 1, 1)
flightButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
flightButton.BorderSizePixel = 0
flightButton.AutoButtonColor = false
flightButton.Parent = screenGui
flightButton.Active = true
flightButton.Draggable = true -- Permite mover el botón

-- Sombra elegante
local uiCorner = Instance.new("UICorner", flightButton)
uiCorner.CornerRadius = UDim.new(0, 18)
local uiStroke = Instance.new("UIStroke", flightButton)
uiStroke.Thickness = 2
uiStroke.Color = Color3.fromRGB(255,255,255)
uiStroke.Transparency = 0.4

-- Efecto arcoíris para el botón
spawn(function()
    local t = 0
    while flightButton.Parent do
        t = t + 0.03
        local r = math.sin(t) * 0.5 + 0.5
        local g = math.sin(t + 2) * 0.5 + 0.5
        local b = math.sin(t + 4) * 0.5 + 0.5
        flightButton.BackgroundColor3 = Color3.new(r, g, b)
        uiStroke.Color = Color3.new(b, r, g)
        wait(0.02)
    end
end)

-- Función de vuelo
local function fly()
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end

    -- Eliminar cuerpos anteriores (prevención de bugs)
    for _, obj in ipairs(character.HumanoidRootPart:GetChildren()) do
        if obj:IsA("BodyPosition") or obj:IsA("BodyGyro") then
            obj:Destroy()
        end
    end

    local bp = Instance.new("BodyPosition")
    bp.MaxForce = Vector3.new(1e6, 1e6, 1e6)
    bp.P = 2e4
    bp.D = 1000
    bp.Position = character.HumanoidRootPart.Position
    bp.Parent = character.HumanoidRootPart

    local bg = Instance.new("BodyGyro")
    bg.MaxTorque = Vector3.new(1e6, 1e6, 1e6)
    bg.P = 2e4
    bg.CFrame = character.HumanoidRootPart.CFrame
    bg.Parent = character.HumanoidRootPart

    local flySpeed = 65

    flying = true

    -- Movimiento
    if flightConn then flightConn:Disconnect() end
    flightConn = RS.RenderStepped:Connect(function()
        if not flying or not character or not character:FindFirstChild("HumanoidRootPart") then
            bp:Destroy()
            bg:Destroy()
            if flightConn then flightConn:Disconnect() end
            return
        end

        local moveDirection = Vector3.new()
        if UIS.TouchEnabled then
            moveDirection = cam.CFrame.LookVector
        else
            moveDirection = humanoid.MoveDirection.Magnitude > 0 and humanoid.MoveDirection or Vector3.new()
        end
        -- Permite flotar si no se mueve (Android)
        bp.Position = character.HumanoidRootPart.Position + moveDirection * flySpeed
        bg.CFrame = cam.CFrame
    end)
end

local function stopFly()
    flying = false
    local character = player.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        for _, obj in ipairs(character.HumanoidRootPart:GetChildren()) do
            if obj:IsA("BodyPosition") or obj:IsA("BodyGyro") then
                obj:Destroy()
            end
        end
    end
    if flightConn then flightConn:Disconnect() end
end

-- Mejor feedback visual del botón y protección
flightButton.MouseButton1Click:Connect(function()
    if not flying then
        fly()
        flightButton.Text = "🛑 DETENER VUELO"
        flightButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    else
        stopFly()
        flightButton.Text = "✈️ VOLAR"
        flightButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    end
end)

-- Detener vuelo al morir, respawn o cambio de personaje
local function onChar()
    stopFly()
    flightButton.Text = "✈️ VOLAR"
end
player.CharacterAdded:Connect(onChar)
if player.Character then onChar() end

-- Extra: Bloquear salto mientras vuelas (opcional)
local oldJump
UIS.JumpRequest:Connect(function()
    if flying then
        local humanoid = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            oldJump = humanoid.JumpPower
            humanoid.JumpPower = 0
            wait(0.1)
            humanoid.JumpPower = oldJump
        end
    end
end)

-- (Opcional) Oculta el botón si abres el menú de escape en PC
if UIS.TouchEnabled == false then
    UIS.InputBegan:Connect(function(input, processed)
        if input.KeyCode == Enum.KeyCode.Escape then
            flightButton.Visible = false
        end
    end)
    UIS.InputEnded:Connect(function(input, processed)
        if input.KeyCode == Enum.KeyCode.Escape then
            flightButton.Visible = true
        end
    end)
end
