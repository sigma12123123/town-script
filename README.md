local OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/shlexware/Orion/main/source')))()

local Window = OrionLib:MakeWindow({Name = "💸HysteriaHubz💸", HidePremium = false, SaveConfig = true, ConfigFolder = "OnBd"})

local PlayerTab = Window:MakeTab({
    Name = "🕴️Player",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

local Section = PlayerTab:AddSection({
    Name = "Movement"
})

PlayerTab:AddSlider({
    Name = "Walkspeed",
    Min = 16,
    Max = 500,
    Default = 16,
    Color = Color3.fromRGB(255, 255, 255),
    Increment = 1,
    ValueName = "Ws",
    Callback = function(Value)
        game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = Value
    end    
})

PlayerTab:AddSlider({
    Name = "Jump Boost",
    Min = 50,
    Max = 500,
    Default = 50,
    Color = Color3.fromRGB(255, 255, 255),
    Increment = 1,
    ValueName = "Jump",
    Callback = function(Value)
        game.Players.LocalPlayer.Character.Humanoid.JumpPower = Value
    end    
})

-- Main Tab for Aimbot
local MainTab = Window:MakeTab({
    Name = "🎯Main",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

-- Aimbot Variables
local aimbotEnabled = false
local fovCircle
local fovRadius = 100
local smoothness = 0.1
local prediction = 0.1
local fovColor = Color3.fromRGB(255, 0, 0)  -- Default FOV color

local UserInputService = game:GetService("UserInputService")

local function createFOVCircle()
    fovCircle = Drawing.new("Circle")
    fovCircle.Visible = false
    fovCircle.Color = fovColor
    fovCircle.Thickness = 2
    fovCircle.NumSides = 100
    fovCircle.Filled = false
end

local function updateFOVCircle()
    if fovCircle then
        fovCircle.Radius = fovRadius
        fovCircle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X / 2, workspace.CurrentCamera.ViewportSize.Y / 2)
        fovCircle.Color = fovColor
    end
end

createFOVCircle()

-- Main Aimbot Logic
local function aimbot()
    if not aimbotEnabled or not UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then return end

    local player = game.Players.LocalPlayer
    local mouse = player:GetMouse()
    local closestPlayer = nil
    local closestMagnitude = fovRadius

    for _, target in pairs(game.Players:GetPlayers()) do
        if target ~= player and target.Character and target.Character:FindFirstChild("Head") then
            local targetPos = target.Character.Head.Position
            local screenPos, onScreen = workspace.CurrentCamera:WorldToScreenPoint(targetPos)

            if onScreen then
                local distance = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(mouse.X, mouse.Y)).Magnitude
                if distance < closestMagnitude then
                    closestMagnitude = distance
                    closestPlayer = target
                end
            end
        end
    end

    if closestPlayer then
        local targetPos = closestPlayer.Character.Head.Position
        local predictedPos = targetPos + (closestPlayer.Character.HumanoidRootPart.Velocity * prediction)

        local camera = workspace.CurrentCamera
        if smoothness == 0 then
            -- Instant aim lock
            camera.CFrame = CFrame.new(camera.CFrame.p, predictedPos)
        else
            -- Smooth aiming
            local direction = (predictedPos - camera.CFrame.p).unit
            local newCFrame = camera.CFrame:Lerp(CFrame.new(camera.CFrame.p, camera.CFrame.p + direction), smoothness)
            camera.CFrame = newCFrame
        end
    end
end

MainTab:AddToggle({
    Name = "Toggle Aimbot",
    Default = false,
    Callback = function(Value)
        aimbotEnabled = Value
        fovCircle.Visible = Value
    end
})

MainTab:AddSlider({
    Name = "FOV Radius",
    Min = 50,
    Max = 300,
    Default = 100,
    Color = Color3.fromRGB(255, 255, 255),
    Increment = 1,
    ValueName = "FOV",
    Callback = function(Value)
        fovRadius = Value
        updateFOVCircle()
    end    
})

MainTab:AddSlider({
    Name = "Smoothness",
    Min = 0,
    Max = 1,
    Default = 0.1,
    Color = Color3.fromRGB(255, 255, 255),
    Increment = 0.01,
    ValueName = "Smoothness",
    Callback = function(Value)
        smoothness = Value
    end    
})

MainTab:AddSlider({
    Name = "Prediction",
    Min = 0,
    Max = 1,
    Default = 0.1,
    Color = Color3.fromRGB(255, 255, 255),
    Increment = 0.01,
    ValueName = "Prediction",
    Callback = function(Value)
        prediction = Value
    end    
})

MainTab:AddSlider({
    Name = "FOV Color (R)",
    Min = 0,
    Max = 255,
    Default = 255,
    Color = Color3.fromRGB(255, 0, 0),
    Increment = 1,
    ValueName = "Red",
    Callback = function(Value)
        fovColor = Color3.fromRGB(Value, fovColor.G * 255, fovColor.B * 255)
        updateFOVCircle()
    end    
})

MainTab:AddSlider({
    Name = "FOV Color (G)",
    Min = 0,
    Max = 255,
    Default = 0,
    Color = Color3.fromRGB(0, 255, 0),
    Increment = 1,
    ValueName = "Green",
    Callback = function(Value)
        fovColor = Color3.fromRGB(fovColor.R * 255, Value, fovColor.B * 255)
        updateFOVCircle()
    end    
})

MainTab:AddSlider({
    Name = "FOV Color (B)",
    Min = 0,
    Max = 255,
    Default = 0,
    Color = Color3.fromRGB(0, 0, 255),
    Increment = 1,
    ValueName = "Blue",
    Callback = function(Value)
        fovColor = Color3.fromRGB(fovColor.R * 255, fovColor.G * 255, Value)
        updateFOVCircle()
    end    
})

-- Collision and Head Size Variables
_G.HeadSize = 5
_G.Disabled = true
_G.CollisionEnabled = false

MainTab:AddSlider({
    Name = "Head Size",
    Min = 1,
    Max = 10,
    Default = _G.HeadSize,
    Color = Color3.fromRGB(255, 255, 255),
    Increment = 0.1,
    ValueName = "Size",
    Callback = function(Value)
        _G.HeadSize = Value
    end    
})

MainTab:AddSlider({
    Name = "Head Transparency",
    Min = 0,
    Max = 1,
    Default = 0.6,
    Color = Color3.fromRGB(255, 255, 255),
    Increment = 0.01,
    ValueName = "Transparency",
    Callback = function(Value)
        _G.HeadTransparency = Value
    end    
})

MainTab:AddToggle({
    Name = "Enable Collision",
    Default = false,
    Callback = function(Value)
        _G.CollisionEnabled = Value
    end
})

-- RenderStepped function for head modifications
game:GetService('RunService').RenderStepped:Connect(function()
    if _G.Disabled then
        for _, player in pairs(game:GetService('Players'):GetPlayers()) do
            if player.Name ~= game:GetService('Players').LocalPlayer.Name then
                pcall(function()
                    local head = player.Character and player.Character:FindFirstChild("Head")
                    if head then
                        head.Size = Vector3.new(_G.HeadSize, _G.HeadSize, _G.HeadSize)
                        head.Transparency = _G.HeadTransparency or 0.6
                        head.BrickColor = BrickColor.new("Red")
                        head.Material = "Neon"
                        head.CanCollide = not _G.CollisionEnabled
                        head.Massless = true
                    end
                end)
            end
        end
    end
end)

game:GetService("RunService").RenderStepped:Connect(aimbot)

local EspTab = Window:MakeTab({
    Name = "👁️Esp",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

local highlightEnabled = false
local highlightColor = Color3.fromRGB(255, 255, 255)  -- Default color white

-- Function to create highlight for players
local function createHighlight(player)
    if player.Character then
        local highlight = Instance.new("Highlight")
        highlight.Adornee = player.Character
        highlight.FillColor = highlightColor
        highlight.OutlineColor = Color3.fromRGB(0, 0, 0)  -- Black outline
        highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        highlight.Parent = player.Character
        return highlight
    end
end

-- Function to update highlights
local function updateHighlights()
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character then
            if highlightEnabled then
                local highlight = player.Character:FindFirstChildOfClass("Highlight")
                if not highlight then
                    highlight = createHighlight(player)
                end
                highlight.FillColor = highlightColor  -- Set the highlight color
            else
                local highlight = player.Character:FindFirstChildOfClass("Highlight")
                if highlight then
                    highlight:Destroy()
                end
            end
        end
    end
end

-- Toggle for Highlight ESP
EspTab:AddToggle({
    Name = "Toggle Highlight ESP",
    Default = false,
    Callback = function(Value)
        highlightEnabled = Value
        updateHighlights()
    end
})

-- Sliders for RGB color customization
EspTab:AddSlider({
    Name = "Highlight Color (R)",
    Min = 0,
    Max = 255,
    Default = 255,
    Color = Color3.fromRGB(255, 0, 0),
    Increment = 1,
    ValueName = "Red",
    Callback = function(Value)
        highlightColor = Color3.fromRGB(Value, highlightColor.G * 255, highlightColor.B * 255)
        updateHighlights()
    end    
})

EspTab:AddSlider({
    Name = "Highlight Color (G)",
    Min = 0,
    Max = 255,
    Default = 255,
    Color = Color3.fromRGB(0, 255, 0),
    Increment = 1,
    ValueName = "Green",
    Callback = function(Value)
        highlightColor = Color3.fromRGB(highlightColor.R * 255, Value, highlightColor.B * 255)
        updateHighlights()
    end    
})

EspTab:AddSlider({
    Name = "Highlight Color (B)",
    Min = 0,
    Max = 255,
    Default = 255,
    Color = Color3.fromRGB(0, 0, 255),
    Increment = 1,
    ValueName = "Blue",
    Callback = function(Value)
        highlightColor = Color3.fromRGB(highlightColor.R * 255, highlightColor.G * 255, Value)
        updateHighlights()
    end    
})

-- Update highlights on player added/removed
game.Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        updateHighlights()
    end)
end)

game.Players.PlayerRemoving:Connect(function(player)
    if player.Character then
        local highlight = player.Character:FindFirstChildOfClass("Highlight")
        if highlight then
            highlight:Destroy()
        end
    end
end)

-- Call updateHighlights to initialize highlights for existing players
updateHighlights()

local MiscTab = Window:MakeTab({
    Name = "❓Misc",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

MiscTab:AddButton({
    Name = "Infinite Yield",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source"))()
    end
})

MiscTab:AddButton({
    Name = "Invisibility",
    Callback = function()
        loadstring(game:HttpGet('https://pastebin.com/raw/3Rnd9rHf'))()
    end
})

MiscTab:AddButton({
    Name = "Kill Interface",
    Callback = function()
        OrionLib:Destroy() -- This will remove the entire Orion interface.
    end
})
