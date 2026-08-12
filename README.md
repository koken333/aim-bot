
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Universal Combat & ESP",
   LoadingTitle = "Gohken Script",
   LoadingSubtitle = "333 จัดให้",
   ConfigurationSaving = {
      Enabled = false
   },
   KeySystem = false
})

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Variables & Settings
local Settings = {
    AimbotEnabled = false,
    AimPart = "Head",
    Smoothness = 0.2,
    TeamCheck = true,
    
    ShowFOV = false,
    FOVRadius = 100,
    FOVColor = Color3.fromRGB(255, 255, 255),
    
    ESPEnabled = false,
    ESPBoxes = false,
    ESPNames = false,
    ESPTracers = false,
    ESPTeamCheck = true
}

-- FOV Circle Drawing Creation
local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 1.5
FOVCircle.NumSides = 60
FOVCircle.Filled = false
FOVCircle.Transparency = 1
FOVCircle.Visible = false
FOVCircle.Color = Settings.FOVColor

-- Helper Functions
local function isEnemy(player)
    if not Settings.TeamCheck then return true end
    if player == LocalPlayer then return false end
    if LocalPlayer.Team and player.Team then
        return LocalPlayer.Team ~= player.Team
    end
    return true
end

local function getClosestEnemyInFOV()
    local closestPlayer = nil
    local shortestDistance = Settings.FOVRadius
    local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and isEnemy(player) then
            local character = player.Character
            if character and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0 then
                local targetPart = character:FindFirstChild(Settings.AimPart)
                if targetPart then
                    local screenPosition, onScreen = Camera:WorldToViewportPoint(targetPart.Position)
                    if onScreen then
                        local targetVector = Vector2.new(screenPosition.X, screenPosition.Y)
                        local distanceFromCenter = (targetVector - screenCenter).Magnitude

                        if distanceFromCenter <= shortestDistance then
                            closestPlayer = character
                            shortestDistance = distanceFromCenter
                        end
                    end
                end
            end
        end
    end
    return closestPlayer
end

-- ESP System Storage
local ESPHolders = {}

local function createESP(player)
    if player == LocalPlayer then return end
    
    local drawings = {
        Box = Drawing.new("Square"),
        Name = Drawing.new("Text"),
        Tracer = Drawing.new("Line")
    }

    drawings.Box.Thickness = 1
    drawings.Box.Filled = false
    drawings.Box.Color = Color3.fromRGB(255, 50, 50)

    drawings.Name.Size = 14
    drawings.Name.Center = true
    drawings.Name.Outline = true
    drawings.Name.Color = Color3.fromRGB(255, 255, 255)

    drawings.Tracer.Thickness = 1
    drawings.Tracer.Color = Color3.fromRGB(255, 50, 50)

    ESPHolders[player] = drawings
end

local function removeESP(player)
    if ESPHolders[player] then
        for _, drawing in pairs(ESPHolders[player]) do
            drawing:Remove()
        end
        ESPHolders[player] = nil
    end
end

for _, p in pairs(Players:GetPlayers()) do createESP(p) end
Players.PlayerAdded:Connect(createESP)
Players.PlayerRemoving:Connect(removeESP)

-- UI Tabs
local MainTab = Window:CreateTab("Aimbot", 4483362458)
local ESPTab = Window:CreateTab("ESP Visuals", 4483362458)

-- Aimbot Controls
MainTab:CreateToggle({
   Name = "Enable Aimbot",
   CurrentValue = false,
   Callback = function(Value)
      Settings.AimbotEnabled = Value
   end,
})

MainTab:CreateToggle({
   Name = "Team Check (Enemies Only)",
   CurrentValue = true,
   Callback = function(Value)
      Settings.TeamCheck = Value
   end,
})

MainTab:CreateDropdown({
   Name = "Aim Target Part",
   Options = {"Head", "HumanoidRootPart"},
   CurrentOption = {"Head"},
   MultipleOptions = false,
   Callback = function(Option)
      Settings.AimPart = Option[1]
   end,
})

MainTab:CreateSlider({
   Name = "Aim Smoothness",
   Range = {0.05, 1},
   Increment = 0.05,
   Suffix = "",
   CurrentValue = 0.2,
   Callback = function(Value)
      Settings.Smoothness = Value
   end,
})

MainTab:CreateSection("FOV Settings")

MainTab:CreateToggle({
   Name = "Show FOV Circle",
   CurrentValue = false,
   Callback = function(Value)
      Settings.ShowFOV = Value
   end,
})

MainTab:CreateSlider({
   Name = "FOV Radius Size",
   Range = {30, 500},
   Increment = 5,
   Suffix = "px",
   CurrentValue = 100,
   Callback = function(Value)
      Settings.FOVRadius = Value
   end,
})

-- ESP Controls
ESPTab:CreateToggle({
   Name = "Enable ESP Master Toggle",
   CurrentValue = false,
   Callback = function(Value)
      Settings.ESPEnabled = Value
   end,
})

ESPTab:CreateToggle({
   Name = "ESP Boxes",
   CurrentValue = false,
   Callback = function(Value)
      Settings.ESPBoxes = Value
   end,
})

ESPTab:CreateToggle({
   Name = "ESP Names",
   CurrentValue = false,
   Callback = function(Value)
      Settings.ESPNames = Value
   end,
})

ESPTab:CreateToggle({
   Name = "ESP Tracers",
   CurrentValue = false,
   Callback = function(Value)
      Settings.ESPTracers = Value
   end,
})

-- Main Loop (Aimbot, FOV, and ESP Rendering)
RunService.RenderStepped:Connect(function()
    -- 1. FOV Update
    local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    FOVCircle.Position = screenCenter
    FOVCircle.Radius = Settings.FOVRadius
    FOVCircle.Visible = Settings.ShowFOV

    -- 2. Aimbot Logic
    if Settings.AimbotEnabled then
        local targetCharacter = getClosestEnemyInFOV()
        if targetCharacter and targetCharacter:FindFirstChild(Settings.AimPart) then
            local targetPosition = targetCharacter[Settings.AimPart].Position
            local currentCFrame = Camera.CFrame
            local targetCFrame = CFrame.new(currentCFrame.Position, targetPosition)
            
            Camera.CFrame = currentCFrame:Lerp(targetCFrame, Settings.Smoothness)
        end
    end

    -- 3. ESP Rendering Logic
    for player, drawings in pairs(ESPHolders) do
        local character = player.Character
        local shouldShow = Settings.ESPEnabled and isEnemy(player) and character and character:FindFirstChild("HumanoidRootPart") and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0

        if shouldShow then
            local hrp = character.HumanoidRootPart
            local vector, onScreen = Camera:WorldToViewportPoint(hrp.Position)

            if onScreen then
                local head = character:FindFirstChild("Head")
                local headPos = head and Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0)) or vector
                local legPos = Camera:WorldToViewportPoint(hrp.Position - Vector3.new(0, 3, 0))
                local height = math.abs(headPos.Y - legPos.Y)
                local width = height / 1.8

                -- Box ESP
                if Settings.ESPBoxes then
                    drawings.Box.Size = Vector2.new(width, height)
                    drawings.Box.Position = Vector2.new(vector.X - width / 2, vector.Y - height / 2)
                    drawings.Box.Visible = true
                else
                    drawings.Box.Visible = false
                end

                -- Name ESP
                if Settings.ESPNames then
                    drawings.Name.Text = player.Name
                    drawings.Name.Position = Vector2.new(vector.X, vector.Y - height / 2 - 16)
                    drawings.Name.Visible = true
                else
                    drawings.Name.Visible = false
                end

                -- Tracer ESP
                if Settings.ESPTracers then
                    drawings.Tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                    drawings.Tracer.To = Vector2.new(vector.X, vector.Y)
                    drawings.Tracer.Visible = true
                else
                    drawings.Tracer.Visible = false
                end
            else
                drawings.Box.Visible = false
                drawings.Name.Visible = false
                drawings.Tracer.Visible = false
            end
        else
            drawings.Box.Visible = false
            drawings.Name.Visible = false
            drawings.Tracer.Visible = false
        end
    end
end)
