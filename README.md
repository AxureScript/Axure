local OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/shlexware/Orion/main/source')))()
local Window = OrionLib:MakeWindow({Name = "Axure", HidePremium = false, SaveConfig = true, ConfigFolder = "OrionTest"})
local Animate = game.Players.LocalPlayer.Character.Animate

local Tab = Window:MakeTab({
    Name = "Aim Config",
    PremiumOnly = false
})

local Section = Tab:AddSection({
    Name = "Aim Settings"
})

local settings = {
    main = {
        Prediction = 0.19,
        Part = "Head",
        AirshotFunc = false,                
        ToggleKey = Enum.KeyCode.C,
    }
}

local function updatePrediction(pred)
    settings.main.Prediction = pred
end

local function updatePart(part)
    settings.main.Part = part
end

local function toggleAirshotFunc(Air)
    settings.main.AirshotFunc = Air
end

Section:AddTextbox({
	Name = "Prediction",
	Default = settings.main.Pred,
	TextDisappear = false,
	Callback = function(pred)
		updatePrediction(pred)
	end	  
})

Section:AddDropdown({
    Name = "Aim Part",
    Default = settings.main.Part,
    Options = {"Head", "HumanoidRootPart"},
    Callback = function(part)
        updatePart(part)
    end    
})

Section:AddToggle({
    Name = "Airshot Function",
    Default = settings.main.AirshotFunc,
    Callback = function(Air)
        toggleAirshotFunc(Air)
    end    
})

local function eliminarHerramientaDeInventario()
    local jugador = game.Players.LocalPlayer
    for _, item in pairs(jugador.Backpack:GetChildren()) do
        if item.Name == "[AIMLOCK]" then
            item:Destroy() 
        end
    end
end

local function toggleToolForMobile(value)
    if value then
        getgenv().keytoclick = "C"
tool = Instance.new("Tool")
tool.RequiresHandle = false
tool.Name = "[AIMLOCK]"
tool.Activated:connect(function()
    local vim = game:service("VirtualInputManager")
vim:SendKeyEvent(true, keytoclick, false, game)
end)
tool.Parent = game.Players.LocalPlayer.Backpack

local player = game.Players.LocalPlayer

local function connectCharacterAdded()
    player.CharacterAdded:Connect(onCharacterAdded)
end

connectCharacterAdded()

player.CharacterRemoving:Connect(function()
    tool.Parent = game.Players.LocalPlayer.Backpack
end)
    else
        eliminarHerramientaDeInventario()
    end
end

Section:AddToggle({
    Name = "Tool For Mobile",
    Default = false,
    Callback = function(Value)
        toggleToolForMobile(Value)
    end    
})

local Target = Window:MakeTab({
    Name = "Target",
    PremiumOnly = false
})

local TargetS = Target:AddSection({
    Name = "Target"
})

local playerDropdown

-- Función para obtener la lista de jugadores en el servidor, excluyendo al jugador local
local function getPlayers()
    local localPlayer = game:GetService("Players").LocalPlayer
    local players = game:GetService("Players"):GetPlayers()
    local playerNames = {}
    for _, player in pairs(players) do
        if player ~= localPlayer then
            table.insert(playerNames, player.Name)
        end
    end
    return playerNames
end

local function updatePlayerDropdown()
    if playerDropdown then
        local currentPlayers = getPlayers()
        playerDropdown:Refresh(currentPlayers, true)  -- Actualizar las opciones del dropdown
    else
        -- Si es la primera vez, crear el menú desplegable con todos los jugadores
        playerDropdown = TargetS:AddDropdown({
            Name = "Player",
            Default = "Select Player",
            Options = getPlayers(),
            Callback = function(value)
                print("Selected player:", value)
                getgenv().ppl = value  -- Guardar el nombre del jugador seleccionado en la variable global
            end
        })
    end
end

-- Llamar a la función de actualización al principio y luego cada cierto tiempo para mantener la lista actualizada
updatePlayerDropdown()
game:GetService("Players").PlayerAdded:Connect(updatePlayerDropdown)
game:GetService("Players").PlayerRemoving:Connect(updatePlayerDropdown)

local TargetB = Target:AddSection({
	Name = "Basic Options "
})


TargetB:AddButton({
    Name = "Goto Player",
    Callback = function()
        local playerName = getgenv().ppl  -- Obtener el nombre del jugador desde la variable global
        if playerName then
            local player = game:GetService("Players"):FindFirstChild(playerName)
            if player then
                local character = player.Character
                if character then
                    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
                    if humanoidRootPart then
                        local teleportCFrame = humanoidRootPart.CFrame
                        game.Players.LocalPlayer.Character:SetPrimaryPartCFrame(teleportCFrame)
                    else
                        print("Error: El jugador seleccionado no tiene un HumanoidRootPart.")
                    end
                else
                    print("Error: El jugador seleccionado no tiene un personaje.")
                end
            else
                print("Error: No se encontró al jugador seleccionado.")
            end
        else
            print("Error: No se ha seleccionado ningún jugador.")
        end
    end    
})

local isViewActive = false

local function toggleView()
    isViewActive = not isViewActive
    
    if isViewActive then
        local player = game.Players:FindFirstChild(getgenv().ppl)
        if player then
            game.Workspace.CurrentCamera.CameraSubject = player.Character
        end
    else
        game.Workspace.CurrentCamera.CameraSubject = game.Players.LocalPlayer.Character
    end
end

TargetB:AddToggle({
    Name = "View",
    Default = false,
    Callback = function(View)
        if View then
            toggleView()
        else
            isViewActive = false
            game.Workspace.CurrentCamera.CameraSubject = game.Players.LocalPlayer.Character
        end
    end    
})

local TargetK = Target:AddSection({
	Name = "Kill Options"
})


local isFunctionActive = false -- Variable para rastrear si alguna función está activa
local toolName = "Combat" -- Nombre de la herramienta

-- Función para matar al jugador
local function killPlayer()
    local playerToKill = getgenv().ppl -- Obtener el jugador almacenado
    if playerToKill then
        local player = game.Players.LocalPlayer
        local character = player.Character
        if character then
            local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
            if humanoidRootPart then
                local targetPlayer = game.Players:FindFirstChild(playerToKill)
                if targetPlayer then
                    local targetCharacter = targetPlayer.Character
                    if targetCharacter then
                        local targetHumanoidRootPart = targetCharacter:FindFirstChild("HumanoidRootPart")
                        if targetHumanoidRootPart then
                            humanoidRootPart.CFrame = targetHumanoidRootPart.CFrame
                        else
                            print("Error: El jugador seleccionado no tiene un HumanoidRootPart.")
                        end
                    else
                        print("Error: El jugador seleccionado no tiene un personaje.")
                    end
                else
                    print("Error: No se encontró al jugador seleccionado.")
                end
            end
        end
    end
end

-- Función para activar o desactivar la función de matar jugadores y la herramienta
local function toggleFunction()
    isFunctionActive = not isFunctionActive
    if isFunctionActive then
        -- Activar la herramienta y los bucles
        local tool = game.Players.LocalPlayer.Backpack:FindFirstChild(toolName)
        if tool then
            tool.Parent = game.Players.LocalPlayer.Character
            while isFunctionActive do
                tool:Activate()                
                killPlayer()
                wait(0.01) -- Esperar un segundo antes de volver a activar
            end
        end
    else
        -- Desactivar la función de matar jugadores
        isKillPlayerActive = false
    end
end

TargetK:AddToggle({
    Name = "Knock Player(Beta)",
    Default = false,
    Callback = function(IsEnabled)
        if IsEnabled then
            -- Activar la función de matar jugadores y la herramienta
            toggleFunction()
        else
            -- Desactivar la función de matar jugadores y la herramienta
            isFunctionActive = false
        end
    end    
})

local Move = Window:MakeTab({
    Name = "Other options",
    PremiumOnly = false
})

local MoveS = Move:AddSection({
    Name = "Movements"
})

local Multiplier = 4
local SpeedEnabled = false

local ToggleCFrameSpeed = MoveS:AddToggle({
    Name = "Cframe Speed",
    Default = false,
    Callback = function(Value)
        SpeedEnabled = Value
        print("CFrame Speed is now:", SpeedEnabled)
    end    
})

local L_136_ = game:GetService('UserInputService')
local RunService = game:GetService('RunService')

local function MoveCharacter()
    if SpeedEnabled then
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame + game.Players.LocalPlayer.Character.Humanoid.MoveDirection * Multiplier
    end
end

L_136_.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.LeftBracket then
        Multiplier = Multiplier + 0.01
        print(Multiplier)
    elseif input.KeyCode == Enum.KeyCode.RightBracket then
        Multiplier = Multiplier - 0.01
        print(Multiplier)
    end
end)

RunService.Heartbeat:Connect(function()
    MoveCharacter()
end)

local AutoBuy = Window:MakeTab({
	Name = "Auto Buy",	
	PremiumOnly = false
})

local BuyG = AutoBuy:AddSection({
	Name = "Auto Buy Guns"
})

BuyG:AddButton({
	Name = "[Revolver] - $1379",
	Callback = function()
local TweenService = game:GetService("TweenService")
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Guardar la posición original del jugador
local originalPosition = rootPart.CFrame

-- CFrame al que deseas teletransportar al jugador
local teleportCFrame = CFrame.new(-642.210022, 18.8500004, -119.634995, 1, 0, 0, 0, 1, 0, 0, 0, 1)

-- Teletransportar al jugador al nuevo CFrame
rootPart.CFrame = teleportCFrame

-- Esperar 1 segundo
wait(1)

-- Teletransportar al jugador de vuelta a la posición original
rootPart.CFrame = originalPosition      		
  	end    
})

BuyG:AddButton({
	Name = "[TacticalShotgun] - $1857",
	Callback = function()
local TweenService = game:GetService("TweenService")
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Guardar la posición original del jugador
local originalPosition = rootPart.CFrame

-- CFrame al que deseas teletransportar al jugador
local teleportCFrame = CFrame.new(470.877533, 45.1272316, -620.630676, 1, 0, 0, 0, 1, 0, 0, 0, 1)

-- Teletransportar al jugador al nuevo CFrame
rootPart.CFrame = teleportCFrame

-- Esperar 1 segundo
wait(1)

-- Teletransportar al jugador de vuelta a la posición original
rootPart.CFrame = originalPosition
  	end    
})



BuyG:AddButton({
	Name = "[Double-Barrel SG] - $1485",
	Callback = function()
     local TweenService = game:GetService("TweenService")
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Guardar la posición original del jugador
local originalPosition = rootPart.CFrame

-- CFrame al que deseas teletransportar al jugador
local teleportCFrame = CFrame.new(-1039.59985, 18.8513641, -256.449951, -1, 0, 0, 0, 1, 0, 0, 0, -1)

-- Teletransportar al jugador al nuevo CFrame
rootPart.CFrame = teleportCFrame

-- Esperar 1 segundo
wait(1)

-- Teletransportar al jugador de vuelta a la posición original
rootPart.CFrame = originalPosition
  	end    
})

local BuyA = AutoBuy:AddSection({
	Name = "Auto Buy Ammo"
})

BuyA:AddButton({
	Name = "[Revolver Ammo] - $80",
	Callback = function()
    local TweenService = game:GetService("TweenService")
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Guardar la posición original del jugador
local originalPosition = rootPart.CFrame

-- CFrame al que deseas teletransportar al jugador
local teleportCFrame = CFrame.new(-635.77002, 18.8555126, -119.345001, 1, 0, 0, 0, 1, 0, 0, 0, 1)

-- Teletransportar al jugador al nuevo CFrame
rootPart.CFrame = teleportCFrame

-- Esperar 1 segundo
wait(1)

-- Teletransportar al jugador de vuelta a la posición original
rootPart.CFrame = originalPosition
  	end    
})

BuyA:AddButton({
	Name = "[TacticalShotgun Ammo] - $64",
	Callback = function()
    local TweenService = game:GetService("TweenService")
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Guardar la posición original del jugador
local originalPosition = rootPart.CFrame

-- CFrame al que deseas teletransportar al jugador
local teleportCFrame = CFrame.new(492.877991, 45.1124992, -620.43103, 1, 0, 0, 0, 1, 0, 0, 0, 1)

-- Teletransportar al jugador al nuevo CFrame
rootPart.CFrame = teleportCFrame

-- Esperar 1 segundo
wait(1)

-- Teletransportar al jugador de vuelta a la posición original
rootPart.CFrame = originalPosition
  	end    
})



BuyA:AddButton({
	Name = "[Double-Barrel SG Ammo] - $64",
	Callback = function()
     local TweenService = game:GetService("TweenService")
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Guardar la posición original del jugador
local originalPosition = rootPart.CFrame

-- CFrame al que deseas teletransportar al jugador
local teleportCFrame = CFrame.new(-1046.20032, 18.8513641, -256.449951, -1, 0, 0, 0, 1, 0, 0, 0, -1)

-- Teletransportar al jugador al nuevo CFrame
rootPart.CFrame = teleportCFrame

-- Esperar 1 segundo
wait(1)

-- Teletransportar al jugador de vuelta a la posición original
rootPart.CFrame = originalPosition
  	end    
})


local BuyO = AutoBuy:AddSection({
	Name = "Auto Buy Others"
})

BuyO:AddButton({
	Name = "[Knife] - $159",
	Callback = function()
      local TweenService = game:GetService("TweenService")
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Guardar la posición original del jugador
local originalPosition = rootPart.CFrame

-- CFrame al que deseas teletransportar al jugador
local teleportCFrame = CFrame.new(-277.649994, 18.8493004, -236, 0, 0, -1, 0, 1, 0, 1, 0, 0)

-- Teletransportar al jugador al nuevo CFrame
rootPart.CFrame = teleportCFrame

-- Esperar 1 segundo
wait(1)

-- Teletransportar al jugador de vuelta a la posición original
rootPart.CFrame = originalPosition
  	end    
})

BuyO:AddButton({
	Name = "[Medium Armor] - $1697",
	Callback = function()
     local TweenService = game:GetService("TweenService")
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- Guardar la posición original del jugador
local originalPosition = rootPart.CFrame

-- CFrame al que deseas teletransportar al jugador
local teleportCFrame = CFrame.new(-934.025024, -28.149826, 570.549683)

-- Teletransportar al jugador al nuevo CFrame
rootPart.CFrame = teleportCFrame

-- Esperar 1 segundo
wait(1)

-- Teletransportar al jugador de vuelta a la posición original
rootPart.CFrame = originalPosition
  	end    
})

local Esp = Window:MakeTab({
	Name = "Esp",
	PremiumOnly = false
})

local EspS = Esp:AddSection({
	Name = "Esp"
})

EspS:AddButton({
	Name = "Esp Box",
	Callback = function()
     local plr = game.Players.LocalPlayer
local camera = game.Workspace.CurrentCamera

for i, v in pairs(game.Players:GetChildren()) do
    local Top = Drawing.new("Line")
    Top.Visible = false
    Top.From = Vector2.new(0, 0)
    Top.To = Vector2.new(200, 200)
    Top.Color = Color3.fromRGB(255, 0, 0)
    Top.Thickness = 2
    Top.Transparency = 1

    local Bottom = Drawing.new("Line")
    Bottom.Visible = false
    Bottom.From = Vector2.new(0, 0)
    Bottom.To = Vector2.new(200, 200)
    Bottom.Color = Color3.fromRGB(255, 0, 0)
    Bottom.Thickness = 2
    Bottom.Transparency = 1

    local Left = Drawing.new("Line")
    Left.Visible = false
    Left.From = Vector2.new(0, 0)
    Left.To = Vector2.new(200, 200)
    Left.Color = Color3.fromRGB(255, 0, 0)
    Left.Thickness = 2
    Left.Transparency = 1

    local Right = Drawing.new("Line")
    Right.Visible = false
    Right.From = Vector2.new(0, 0)
    Right.To = Vector2.new(200, 200)
    Right.Color = Color3.fromRGB(255, 0, 0)
    Right.Thickness = 2
    Right.Transparency = 1

    function ESP()
        local connection
        connection = game:GetService("RunService").RenderStepped:Connect(function()
            if v.Character ~= nil and v.Character:FindFirstChild("HumanoidRootPart") ~= nil and v.Name ~= plr.Name  and v.Character.Humanoid.Health > 0 then 
                local ScreenPos, OnScreen = camera:WorldToViewportPoint(v.Character.HumanoidRootPart.Position)
                if OnScreen then
                    local Scale = v.Character.Head.Size.Y/2
                    local Size = Vector3.new(2, 3, 0) * (Scale * 2)
                    local humpos = camera:WorldToViewportPoint(v.Character.HumanoidRootPart.Position)
                    local TL = camera:WorldToViewportPoint((v.Character.HumanoidRootPart.CFrame * CFrame.new(Size.X, Size.Y, 0)).p)
                    local TR = camera:WorldToViewportPoint((v.Character.HumanoidRootPart.CFrame * CFrame.new(-Size.X, Size.Y, 0)).p)
                    local BL = camera:WorldToViewportPoint((v.Character.HumanoidRootPart.CFrame * CFrame.new(Size.X, -Size.Y, 0)).p)
                    local BR = camera:WorldToViewportPoint((v.Character.HumanoidRootPart.CFrame * CFrame.new(-Size.X, -Size.Y, 0)).p)

                    Top.From = Vector2.new(TL.X, TL.Y)
                    Top.To = Vector2.new(TR.X, TR.Y)

                    Left.From = Vector2.new(TL.X, TL.Y)
                    Left.To = Vector2.new(BL.X, BL.Y)

                    Right.From = Vector2.new(TR.X, TR.Y)
                    Right.To = Vector2.new(BR.X, BR.Y)

                    Bottom.From = Vector2.new(BL.X, BL.Y)
                    Bottom.To = Vector2.new(BR.X, BR.Y)

                    if v.TeamColor == plr.TeamColor then
                        Top.Color = Color3.fromRGB(0, 255, 0)
                        Left.Color = Color3.fromRGB(0, 255, 0)
                        Bottom.Color = Color3.fromRGB(0, 255, 0)
                        Right.Color = Color3.fromRGB(0, 255, 0)
                    else 
                        Top.Color = Color3.fromRGB(255, 0, 0)
                        Left.Color = Color3.fromRGB(255, 0, 0)
                        Bottom.Color = Color3.fromRGB(255, 0, 0)
                        Right.Color = Color3.fromRGB(255, 0, 0)
                    end

                    Top.Visible = true
                    Left.Visible = true
                    Bottom.Visible = true
                    Right.Visible = true
                else 
                    Top.Visible = false
                    Left.Visible = false
                    Bottom.Visible = false
                    Right.Visible = false
                end
            else 
                Top.Visible = false
                Left.Visible = false
                Bottom.Visible = false
                Right.Visible = false
                if game.Players:FindFirstChild(v.Name) == nil then
                    connection:Disconnect()
                end
            end
        end)
    end
    coroutine.wrap(ESP)()
end

game.Players.PlayerAdded:Connect(function(newplr) --Parameter gets the new player that has been added
    local Top = Drawing.new("Line")
    Top.Visible = false
    Top.From = Vector2.new(0, 0)
    Top.To = Vector2.new(200, 200)
    Top.Color = Color3.fromRGB(255, 0, 0)
    Top.Thickness = 2
    Top.Transparency = 1

    local Bottom = Drawing.new("Line")
    Bottom.Visible = false
    Bottom.From = Vector2.new(0, 0)
    Bottom.To = Vector2.new(200, 200)
    Bottom.Color = Color3.fromRGB(255, 0, 0)
    Bottom.Thickness = 2
    Bottom.Transparency = 1

    local Left = Drawing.new("Line")
    Left.Visible = false
    Left.From = Vector2.new(0, 0)
    Left.To = Vector2.new(200, 200)
    Left.Color = Color3.fromRGB(255, 0, 0)
    Left.Thickness = 2
    Left.Transparency = 1

    local Right = Drawing.new("Line")
    Right.Visible = false
    Right.From = Vector2.new(0, 0)
    Right.To = Vector2.new(200, 200)
    Right.Color = Color3.fromRGB(255, 0, 0)
    Right.Thickness = 2
    Right.Transparency = 1

    function ESP()
        local connection
        connection = game:GetService("RunService").RenderStepped:Connect(function()
            if newplr.Character ~= nil and newplr.Character:FindFirstChild("HumanoidRootPart") ~= nil and newplr.Name ~= plr.Name  and newplr.Character.Humanoid.Health > 0 then
                local ScreenPos, OnScreen = camera:WorldToViewportPoint(newplr.Character.HumanoidRootPart.Position)
                if OnScreen then
                    local Scale = newplr.Character.Head.Size.Y/2
                    local Size = Vector3.new(2, 3, 0) * (Scale * 2)
                    local humpos = camera:WorldToViewportPoint(newplr.Character.HumanoidRootPart.Position)
                    local TL = camera:WorldToViewportPoint((newplr.Character.HumanoidRootPart.CFrame * CFrame.new(Size.X, Size.Y, 0)).p)
                    local TR = camera:WorldToViewportPoint((newplr.Character.HumanoidRootPart.CFrame * CFrame.new(-Size.X, Size.Y, 0)).p)
                    local BL = camera:WorldToViewportPoint((newplr.Character.HumanoidRootPart.CFrame * CFrame.new(Size.X, -Size.Y, 0)).p)
                    local BR = camera:WorldToViewportPoint((newplr.Character.HumanoidRootPart.CFrame * CFrame.new(-Size.X, -Size.Y, 0)).p)

                    Top.From = Vector2.new(TL.X, TL.Y)
                    Top.To = Vector2.new(TR.X, TR.Y)

                    Left.From = Vector2.new(TL.X, TL.Y)
                    Left.To = Vector2.new(BL.X, BL.Y)

                    Right.From = Vector2.new(TR.X, TR.Y)
                    Right.To = Vector2.new(BR.X, BR.Y)

                    Bottom.From = Vector2.new(BL.X, BL.Y)
                    Bottom.To = Vector2.new(BR.X, BR.Y)

                    if newplr.TeamColor == plr.TeamColor then
                        Top.Color = Color3.fromRGB(0, 255, 0)
                        Left.Color = Color3.fromRGB(0, 255, 0)
                        Bottom.Color = Color3.fromRGB(0, 255, 0)
                        Right.Color = Color3.fromRGB(0, 255, 0)
                    else 
                        Top.Color = Color3.fromRGB(255, 0, 0)
                        Left.Color = Color3.fromRGB(255, 0, 0)
                        Bottom.Color = Color3.fromRGB(255, 0, 0)
                        Right.Color = Color3.fromRGB(255, 0, 0)
                    end

                    Top.Visible = true
                    Left.Visible = true
                    Bottom.Visible = true
                    Right.Visible = true
                else 
                    Top.Visible = false
                    Left.Visible = false
                    Bottom.Visible = false
                    Right.Visible = false
                end
            else 
                Top.Visible = false
                Left.Visible = false
                Bottom.Visible = false
                Right.Visible = false
                if game.Players:FindFirstChild(newplr.Name) == nil then
                    connection:Disconnect()
                end
            end
        end)
    end
    coroutine.wrap(ESP)()
end)
  	end    
})



local Animations = Window:MakeTab({
	Name = "Animations",
	PremiumOnly = false
})

local Ani = Animations:AddSection({
	Name = "Animations"
})

Ani:AddButton({
	Name = "Trayhard",
	Callback = function()
Animate.idle.Animation1.AnimationId = "http://www.roblox.com/asset/?id=2510196951"
Animate.idle.Animation2.AnimationId = "http://www.roblox.com/asset/?id=2510197257"
Animate.walk.WalkAnim.AnimationId = "http://www.roblox.com/asset/?id=616168032"
Animate.run.RunAnim.AnimationId = "http://www.roblox.com/asset/?id=616163682"
Animate.jump.JumpAnim.AnimationId = "http://www.roblox.com/asset/?id=656117878"
Animate.climb.ClimbAnim.AnimationId = "http://www.roblox.com/asset/?id=656114359"
Animate.fall.FallAnim.AnimationId = "http://www.roblox.com/asset/?id=656115606"
game.Players.LocalPlayer.Character.Humanoid.Jump = true
  	end    
})

Ani:AddButton({
	Name = "Ninja",
	Callback = function()
Animate.idle.Animation1.AnimationId = "http://www.roblox.com/asset/?id=656117400"
Animate.idle.Animation2.AnimationId = "http://www.roblox.com/asset/?id=656118341"
Animate.walk.WalkAnim.AnimationId = "http://www.roblox.com/asset/?id=656121766"
Animate.run.RunAnim.AnimationId = "http://www.roblox.com/asset/?id=656118852"
Animate.jump.JumpAnim.AnimationId = "http://www.roblox.com/asset/?id=656117878"
Animate.climb.ClimbAnim.AnimationId = "http://www.roblox.com/asset/?id=656114359"
Animate.fall.FallAnim.AnimationId = "http://www.roblox.com/asset/?id=656115606"
game.Players.LocalPlayer.Character.Humanoid.Jump = true
  	end    
})

Ani:AddButton({
	Name = "Bubbly",
	Callback = function()
Animate.idle.Animation1.AnimationId = "http://www.roblox.com/asset/?id=910004836"
Animate.idle.Animation2.AnimationId = "http://www.roblox.com/asset/?id=910009958"
Animate.walk.WalkAnim.AnimationId = "http://www.roblox.com/asset/?id=910034870"
Animate.run.RunAnim.AnimationId = "http://www.roblox.com/asset/?id=910025107"
Animate.jump.JumpAnim.AnimationId = "http://www.roblox.com/asset/?id=910016857"
Animate.fall.FallAnim.AnimationId = "http://www.roblox.com/asset/?id=910001910"
Animate.swimidle.SwimIdle.AnimationId = "http://www.roblox.com/asset/?id=910030921"
Animate.swim.Swim.AnimationId = "http://www.roblox.com/asset/?id=910028158"
game.Players.LocalPlayer.Character.Humanoid.Jump = true
  	end    
})

Ani:AddButton({
	Name = "Wolf",
	Callback = function()
Animate.idle.Animation1.AnimationId = "http://www.roblox.com/asset/?id=1083195517"
Animate.idle.Animation2.AnimationId = "http://www.roblox.com/asset/?id=1083214717"
Animate.walk.WalkAnim.AnimationId = "http://www.roblox.com/asset/?id=1083178339"
Animate.run.RunAnim.AnimationId = "http://www.roblox.com/asset/?id=1083216690"
Animate.jump.JumpAnim.AnimationId = "http://www.roblox.com/asset/?id=1083218792"
Animate.climb.ClimbAnim.AnimationId = "http://www.roblox.com/asset/?id=1083182000"
Animate.fall.FallAnim.AnimationId = "http://www.roblox.com/asset/?id=1083189019"
game.Players.LocalPlayer.Character.Humanoid.Jump = true
  	end    
})

Ani:AddButton({
	Name = "Stylish",
	Callback = function()
Animate.idle.Animation1.AnimationId = "http://www.roblox.com/asset/?id=616136790"
Animate.idle.Animation2.AnimationId = "http://www.roblox.com/asset/?id=616138447"
Animate.walk.WalkAnim.AnimationId = "http://www.roblox.com/asset/?id=616146177"
Animate.run.RunAnim.AnimationId = "http://www.roblox.com/asset/?id=616140816"
Animate.jump.JumpAnim.AnimationId = "http://www.roblox.com/asset/?id=616139451"
Animate.climb.ClimbAnim.AnimationId = "http://www.roblox.com/asset/?id=616133594"
Animate.fall.FallAnim.AnimationId = "http://www.roblox.com/asset/?id=616134815"
game.Players.LocalPlayer.Character.Humanoid.Jump = true
  	end    
})

Ani:AddButton({
	Name = "Toy",
	Callback = function()
Animate.idle.Animation1.AnimationId = "http://www.roblox.com/asset/?id=782841498"
Animate.idle.Animation2.AnimationId = "http://www.roblox.com/asset/?id=782845736"
Animate.walk.WalkAnim.AnimationId = "http://www.roblox.com/asset/?id=782843345"
Animate.run.RunAnim.AnimationId = "http://www.roblox.com/asset/?id=782842708"
Animate.jump.JumpAnim.AnimationId = "http://www.roblox.com/asset/?id=782847020"
Animate.climb.ClimbAnim.AnimationId = "http://www.roblox.com/asset/?id=782843869"
Animate.fall.FallAnim.AnimationId = "http://www.roblox.com/asset/?id=782846423"
game.Players.LocalPlayer.Character.Humanoid.Jump = true
  	end    
})

Ani:AddButton({
	Name = "Vampire",
	Callback = function()
Animate.idle.Animation1.AnimationId = "http://www.roblox.com/asset/?id=1083445855"
Animate.idle.Animation2.AnimationId = "http://www.roblox.com/asset/?id=1083450166"
Animate.walk.WalkAnim.AnimationId = "http://www.roblox.com/asset/?id=1083473930"
Animate.run.RunAnim.AnimationId = "http://www.roblox.com/asset/?id=1083462077"
Animate.jump.JumpAnim.AnimationId = "http://www.roblox.com/asset/?id=1083455352"
Animate.climb.ClimbAnim.AnimationId = "http://www.roblox.com/asset/?id=1083439238"
Animate.fall.FallAnim.AnimationId = "http://www.roblox.com/asset/?id=1083443587"
game.Players.LocalPlayer.Character.Humanoid.Jump = true
  	end    
})

Ani:AddButton({
	Name = "Zombie",
	Callback = function()
Animate.idle.Animation1.AnimationId = "http://www.roblox.com/asset/?id=616158929"
Animate.idle.Animation2.AnimationId = "http://www.roblox.com/asset/?id=616160636"
Animate.walk.WalkAnim.AnimationId = "http://www.roblox.com/asset/?id=616168032"
Animate.run.RunAnim.AnimationId = "http://www.roblox.com/asset/?id=616163682"
Animate.jump.JumpAnim.AnimationId = "http://www.roblox.com/asset/?id=616161997"
Animate.climb.ClimbAnim.AnimationId = "http://www.roblox.com/asset/?id=616156119"
Animate.fall.FallAnim.AnimationId = "http://www.roblox.com/asset/?id=616157476"
game.Players.LocalPlayer.Character.Humanoid.Jump = true
  	end    
})

Ani:AddButton({
	Name = "Super Hero",
	Callback = function()
Animate.idle.Animation1.AnimationId = "http://www.roblox.com/asset/?id=616111295"
Animate.idle.Animation2.AnimationId = "http://www.roblox.com/asset/?id=616113536"
Animate.walk.WalkAnim.AnimationId = "http://www.roblox.com/asset/?id=616122287"
Animate.run.RunAnim.AnimationId = "http://www.roblox.com/asset/?id=616117076"
Animate.jump.JumpAnim.AnimationId = "http://www.roblox.com/asset/?id=616115533"
Animate.climb.ClimbAnim.AnimationId = "http://www.roblox.com/asset/?id=616104706"
Animate.fall.FallAnim.AnimationId = "http://www.roblox.com/asset/?id=616108001"
game.Players.LocalPlayer.Character.Humanoid.Jump = true
  	end    
})

Ani:AddButton({
	Name = "Pirata",
	Callback = function()
Animate.idle.Animation1.AnimationId = "http://www.roblox.com/asset/?id=750781874"
Animate.idle.Animation2.AnimationId = "http://www.roblox.com/asset/?id=750782770"
Animate.walk.WalkAnim.AnimationId = "http://www.roblox.com/asset/?id=750785693"
Animate.run.RunAnim.AnimationId = "http://www.roblox.com/asset/?id=750783738"
Animate.jump.JumpAnim.AnimationId = "http://www.roblox.com/asset/?id=750782230"
Animate.climb.ClimbAnim.AnimationId = "http://www.roblox.com/asset/?id=750779899"
Animate.fall.FallAnim.AnimationId = "http://www.roblox.com/asset/?id=750780242"
game.Players.LocalPlayer.Character.Humanoid.Jump = true
  	end    
})

Ani:AddButton({
	Name = "Mage",
	Callback = function()
Animate.idle.Animation1.AnimationId = "http://www.roblox.com/asset/?id=707742142"
Animate.idle.Animation2.AnimationId = "http://www.roblox.com/asset/?id=707855907"
Animate.walk.WalkAnim.AnimationId = "http://www.roblox.com/asset/?id=707897309"
Animate.run.RunAnim.AnimationId = "http://www.roblox.com/asset/?id=707861613"
Animate.jump.JumpAnim.AnimationId = "http://www.roblox.com/asset/?id=707853694"
Animate.climb.ClimbAnim.AnimationId = "http://www.roblox.com/asset/?id=707826056"
Animate.fall.FallAnim.AnimationId = "http://www.roblox.com/asset/?id=707829716"
game.Players.LocalPlayer.Character.Humanoid.Jump = true
  	end    
})

Ani:AddButton({
	Name = "Levitation",
	Callback = function()
Animate.idle.Animation1.AnimationId = "http://www.roblox.com/asset/?id=616006778"
Animate.idle.Animation2.AnimationId = "http://www.roblox.com/asset/?id=616008087"
Animate.walk.WalkAnim.AnimationId = "http://www.roblox.com/asset/?id=616013216"
Animate.run.RunAnim.AnimationId = "http://www.roblox.com/asset/?id=616010382"
Animate.jump.JumpAnim.AnimationId = "http://www.roblox.com/asset/?id=616008936"
Animate.climb.ClimbAnim.AnimationId = "http://www.roblox.com/asset/?id=616003713"
Animate.fall.FallAnim.AnimationId = "http://www.roblox.com/asset/?id=616005863"
game.Players.LocalPlayer.Character.Humanoid.Jump = true
  	end    
})

Ani:AddButton({
	Name = "Cartoony",
	Callback = function()
Animate.idle.Animation1.AnimationId = "http://www.roblox.com/asset/?id=742637544"
Animate.idle.Animation2.AnimationId = "http://www.roblox.com/asset/?id=742638445"
Animate.walk.WalkAnim.AnimationId = "http://www.roblox.com/asset/?id=742640026"
Animate.run.RunAnim.AnimationId = "http://www.roblox.com/asset/?id=742638842"
Animate.jump.JumpAnim.AnimationId = "http://www.roblox.com/asset/?id=742637942"
Animate.climb.ClimbAnim.AnimationId = "http://www.roblox.com/asset/?id=742636889"
Animate.fall.FallAnim.AnimationId = "http://www.roblox.com/asset/?id=742637151"
game.Players.LocalPlayer.Character.Humanoid.Jump = true
  	end    
})

Ani:AddButton({
	Name = "Elder",
	Callback = function()
Animate.idle.Animation1.AnimationId = "http://www.roblox.com/asset/?id=845397899"
Animate.idle.Animation2.AnimationId = "http://www.roblox.com/asset/?id=845400520"
Animate.walk.WalkAnim.AnimationId = "http://www.roblox.com/asset/?id=845403856"
Animate.run.RunAnim.AnimationId = "http://www.roblox.com/asset/?id=845386501"
Animate.jump.JumpAnim.AnimationId = "http://www.roblox.com/asset/?id=845398858"
Animate.climb.ClimbAnim.AnimationId = "http://www.roblox.com/asset/?id=845392038"
Animate.fall.FallAnim.AnimationId = "http://www.roblox.com/asset/?id=845396048"
game.Players.LocalPlayer.Character.Humanoid.Jump = true
  	end    
})

Ani:AddButton({
	Name = "Robot",
	Callback = function()
Animate.idle.Animation1.AnimationId = "http://www.roblox.com/asset/?id=616088211"
Animate.idle.Animation2.AnimationId = "http://www.roblox.com/asset/?id=616089559"
Animate.walk.WalkAnim.AnimationId = "http://www.roblox.com/asset/?id=616095330"
Animate.run.RunAnim.AnimationId = "http://www.roblox.com/asset/?id=616091570"
Animate.jump.JumpAnim.AnimationId = "http://www.roblox.com/asset/?id=616090535"
Animate.climb.ClimbAnim.AnimationId = "http://www.roblox.com/asset/?id=616086039"
Animate.fall.FallAnim.AnimationId = "http://www.roblox.com/asset/?id=616087089"
game.Players.LocalPlayer.Character.Humanoid.Jump = true
  	end    
})

OrionLib:Init()

local Mouse = game.Players.LocalPlayer:GetMouse()
local Plr = game.Players.LocalPlayer
local CurrentCamera = game.Workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local selectionBillboard = nil 

local function createImageLabel(player)
    if not selectionBillboard then
        local billboardGui = Instance.new("BillboardGui")
        billboardGui.Name = "SelectionBillboard"
        billboardGui.AlwaysOnTop = true
        billboardGui.Size = UDim2.new(4, 4, 4, 4)
        billboardGui.StudsOffset = Vector3.new(0, 3, 0)

        billboardGui.Adornee = player.Character.Head

        local imageLabel = Instance.new("ImageLabel")
        imageLabel.Name = "SelectionImage"
        imageLabel.Parent = billboardGui
        imageLabel.Size = UDim2.new(1, 0, 1, 0)
        imageLabel.BackgroundTransparency = 1
        imageLabel.Image = "rbxassetid://17062639197"
        imageLabel.ImageTransparency = 0

        billboardGui.Parent = game.Workspace

        selectionBillboard = billboardGui
    end
end

local function removeImageLabel()
    if selectionBillboard then
        selectionBillboard:Destroy()
        selectionBillboard = nil
    end
end

local function toggleLockMode()
    settings.main.DotEnabled = not settings.main.DotEnabled
    
    if settings.main.DotEnabled then
        Plr = FindClosestUser()
        if settings.main.Notifications then
            game.StarterGui:SetCore("SendNotification", {
                Title = "Aim Status",
                Text = "Locked on to: " .. tostring(Plr.Character.Humanoid.DisplayName)
            })
        end
        createImageLabel(Plr)
    else
        if settings.main.Notifications then
            game.StarterGui:SetCore("SendNotification", {
                Title = "Aim Status",
                Text = "No longer locked on"
            })
        end
        removeImageLabel()
    end
end

UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == settings.main.ToggleKey then
        toggleLockMode()
    end
end)

function FindClosestUser()
    local closestPlayer
    local shortestDistance = math.huge

    for i, v in pairs(game.Players:GetPlayers()) do
        if v ~= game.Players.LocalPlayer and v.Character and v.Character:FindFirstChild("Humanoid") and
            v.Character.Humanoid.Health ~= 0 and v.Character:FindFirstChild("HumanoidRootPart") then
            local pos = CurrentCamera:WorldToViewportPoint(v.Character.PrimaryPart.Position)
            local magnitude = (Vector2.new(pos.X, pos.Y) - Vector2.new(Mouse.X, Mouse.Y)).magnitude
            if magnitude < shortestDistance then
                closestPlayer = v
                shortestDistance = magnitude
            end
        end
    end
    return closestPlayer
end

RunService.Stepped:Connect(function()
    if settings.main.DotEnabled and Plr.Character and Plr.Character:FindFirstChild(settings.main.Part) then

    end
end)

local mt = getrawmetatable(game)
local old = mt.__namecall
setreadonly(mt, false)
mt.__namecall = newcclosure(function(...)
    local args = {...}
    if settings.main.DotEnabled and getnamecallmethod() == "FireServer" and args[2] == "UpdateMousePos" then
        args[3] = Plr.Character[settings.main.Part].Position +
                      (Plr.Character[settings.main.Part].Velocity * settings.main.Prediction)
        return old(unpack(args))
    end
    return old(...)
end)

if settings.main.AirshotFunc then
    if Plr.Character.Humanoid.Jump == true and Plr.Character.Humanoid.FloorMaterial == Enum.Material.Air then
        settings.main.Part = "RightFoot"
    else
        Plr.Character:WaitForChild("Humanoid").StateChanged:Connect(function(old,new)
            if new == Enum.HumanoidStateType.Freefall then
                settings.main.Part = "RightFoot"
            else
                settings.main.Part = part
            end
        end)
    end
end
