local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- CONFIGURATION
local ESP_ENABLED = false
local FRUIT_CHECK_INTERVAL = 0.5 -- How often to re-scan for fruits (in seconds)
local FRUIT_MODEL_NAME = "Blox Fruit" -- The name of the dropped fruit model in the Workspace

-- UI (SIMPLE TOGGLE)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "FruitESPGUI"
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local ToggleButton = Instance.new("TextButton")
ToggleButton.Name = "ESPToggleButton"
ToggleButton.Size = UDim2.new(0, 150, 0, 40)
ToggleButton.Position = UDim2.new(0.05, 0, 0.05, 0)
ToggleButton.Text = "Toggle Fruit ESP (OFF)"
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.TextSize = 18
ToggleButton.TextColor3 = Color3.new(1, 1, 1) -- White
ToggleButton.BackgroundColor3 = Color3.new(0.8, 0, 0) -- Red (OFF state)
ToggleButton.BorderSizePixel = 0
ToggleButton.Parent = ScreenGui

-- Internal table to keep track of the created ESP visual objects
local ActiveESPs = {}
local NextCheckTime = 0

-- Function to create the visual ESP for a single fruit
local function CreateFruitESP(fruit)
    -- Remove any old ESPs associated with this fruit, just in case
    if ActiveESPs[fruit] then
        ActiveESPs[fruit]:Destroy()
    end

    local BillboardGui = Instance.new("BillboardGui")
    BillboardGui.Name = "FruitESPVisual"
    BillboardGui.Adornee = fruit
    BillboardGui.Size = UDim2.new(0, 200, 0, 100)
    BillboardGui.AlwaysOnTop = true
    BillboardGui.ExtentsOffset = Vector3.new(0, 4, 0) -- Lift it slightly above the fruit
    BillboardGui.MaxDistance = 50000 -- Make it visible far away

    -- Text Label (Shows the fruit name)
    local TextLabel = Instance.new("TextLabel")
    TextLabel.Name = "FruitNameLabel"
    TextLabel.Size = UDim2.new(1, 0, 0.5, 0)
    TextLabel.Position = UDim2.new(0, 0, 0, 0)
    -- The fruit name is typically the third child (the Model's name is "Blox Fruit")
    local actualFruitName = "Unknown Fruit"
    local fruitTypePart = fruit:FindFirstChildOfClass("Part") -- Assuming a main part holds the name
    if fruitTypePart and fruitTypePart.Name then
        actualFruitName = fruitTypePart.Name:gsub("_", " "):gsub("^%l", string.upper) -- Clean up and capitalize
    end
    -- Fallback/Alternative way to get name:
    if fruit.Configuration and fruit.Configuration:FindFirstChild("FruitName") then
        actualFruitName = fruit.Configuration.FruitName.Value
    end

    TextLabel.Text = actualFruitName
    TextLabel.Font = Enum.Font.SourceSansBold
    TextLabel.TextSize = 20
    TextLabel.TextColor3 = Color3.new(1, 1, 1) -- White
    TextLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
    TextLabel.TextStrokeTransparency = 0
    TextLabel.BackgroundTransparency = 1
    TextLabel.Parent = BillboardGui

    -- Box (Simple Frame to act as a bounding box indicator)
    local BoxFrame = Instance.new("Frame")
    BoxFrame.Name = "BoxFrame"
    BoxFrame.Size = UDim2.new(1, 0, 1, 0)
    BoxFrame.Position = UDim2.new(0, 0, 0.5, 0)
    BoxFrame.BackgroundColor3 = Color3.new(0, 1, 0) -- Green
    BoxFrame.BackgroundTransparency = 0.8
    BoxFrame.BorderSizePixel = 2
    BoxFrame.BorderColor3 = Color3.new(0, 1, 0)
    BoxFrame.Parent = BillboardGui

    -- Store the BillboardGui to manage its lifecycle
    BillboardGui.Parent = BillboardGui.Adornee -- Attach to the fruit model
    ActiveESPs[fruit] = BillboardGui
end

-- Function to remove the ESP visual
local function RemoveFruitESP(fruit)
    if ActiveESPs[fruit] then
        ActiveESPs[fruit]:Destroy()
        ActiveESPs[fruit] = nil
    end
end

-- Core logic to scan and update the ESP
local function ScanForFruits()
    for _, child in ipairs(Workspace:GetChildren()) do
        if child.Name == FRUIT_MODEL_NAME and child:IsA("Model") then
            -- Check if this fruit already has an active ESP
            if not ActiveESPs[child] then
                CreateFruitESP(child)
            end
        end
    end

    -- Clean up ESPs for fruits that no longer exist
    for fruit, gui in pairs(ActiveESPs) do
        if not fruit.Parent or fruit.Parent ~= Workspace then
            RemoveFruitESP(fruit)
        end
    end
end

-- Main loop to manage ESP and timing
RunService.Heartbeat:Connect(function(dt)
    if ESP_ENABLED then
        NextCheckTime = NextCheckTime - dt
        if NextCheckTime <= 0 then
            ScanForFruits()
            NextCheckTime = FRUIT_CHECK_INTERVAL
        end
    end
end)

-- Function to handle fruit additions to the workspace (more reliable than just scanning)
local function OnWorkspaceChildAdded(child)
    if ESP_ENABLED and child.Name == FRUIT_MODEL_NAME and child:IsA("Model") then
        -- Wait a moment to ensure all parts/configuration values are loaded
        delay(0.1, function()
            if child.Parent == Workspace and not ActiveESPs[child] then
                CreateFruitESP(child)
            end
        end)
    end
end

-- Function to handle fruit removal from the workspace
local function OnWorkspaceChildRemoved(child)
    if child.Name == FRUIT_MODEL_NAME and ActiveESPs[child] then
        RemoveFruitESP(child)
    end
end

-- Connect listeners to automatically update when a fruit spawns or despawns
Workspace.ChildAdded:Connect(OnWorkspaceChildAdded)
Workspace.ChildRemoved:Connect(OnWorkspaceChildRemoved)

-- Toggle button logic
local function ToggleESP()
    ESP_ENABLED = not ESP_ENABLED

    if ESP_ENABLED then
        -- ON State
        ToggleButton.Text = "Toggle Fruit ESP (ON)"
        ToggleButton.BackgroundColor3 = Color3.new(0, 0.8, 0) -- Green
        ScanForFruits() -- Initial scan when turned on
    else
        -- OFF State
        ToggleButton.Text = "Toggle Fruit ESP (OFF)"
        ToggleButton.BackgroundColor3 = Color3.new(0.8, 0, 0) -- Red

        -- Clean up all existing ESP visuals
        for fruit in pairs(ActiveESPs) do
            RemoveFruitESP(fruit)
        end
    end
end

ToggleButton.MouseButton1Click:Connect(ToggleESP)

-- Cleanup function in case the script is disabled/destroyed
local function Cleanup()
    for fruit in pairs(ActiveESPs) do
        RemoveFruitESP(fruit)
    end
    ScreenGui:Destroy()
end
