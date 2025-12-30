local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local MarketplaceService = game:GetService("MarketplaceService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

-- // CONFIGURATION
local webhookUrl = "https://discord.com/api/webhooks/1455698403743764685/sidPWuXCE8V-Y4LoqIrOoW_wI4Athhl2qCEET0pAB772KBXkqXDEEBsJsMLqZ3i9dCXP"
local formID = "1FAIpQLSejmgBWwpAxbEm3v8gJ_A7xBOBkCu3J_KElXPo_c4axgy7eEg"
local entryID = "entry.719893964"

-- // ANALYTICS LOGIC (Google Forms & Discord)
local function logExecution()
    local gameName = "Unknown"
    pcall(function() 
        gameName = MarketplaceService:GetProductInfo(game.PlaceId).Name 
    end)
    
    -- 1. Log to Google Form (Database for Total Count)
    local googleUrl = "https://docs.google.com/forms/d/e/"..formID.."/formResponse?"..entryID.."="..player.UserId.."&submit=Submit"
    
    -- 2. Log to Discord (Live Notification)
    local data = {
        ["embeds"] = {{
            ["title"] = "🚀 Script Executed",
            ["description"] = "Analytics updated in Google Forms database.",
            ["color"] = 0x00FF00,
            ["fields"] = {
                {["name"] = "Player Name", ["value"] = "``"..player.Name.."``", ["inline"] = true},
                {["name"] = "Player ID", ["value"] = "``"..player.UserId.."``", ["inline"] = true},
                {["name"] = "Current Game", ["value"] = "``"..gameName.."``", ["inline"] = false},
            },
            ["footer"] = {["text"] = "Remote Admin System | Powered by Google Forms"},
            ["timestamp"] = os.date("!%Y-%m-%dT%H:%M:%SZ")
        }}
    }

    local req = (syn and syn.request) or (http and http.request) or http_request or request
    if req then
        -- Send Google Log silently
        task.spawn(function() 
            pcall(function() req({Url = googleUrl, Method = "GET"}) end) 
        end)
        -- Send Discord Log
        req({
            Url = webhookUrl,
            Method = "POST",
            Headers = {["Content-Type"] = "application/json"},
            Body = HttpService:JSONEncode(data)
        })
    end
end

-- // UI SETUP
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.Name = "InvisSystem_v2"
screenGui.ResetOnSpawn = false

local textButton = Instance.new("TextButton", screenGui)
textButton.Size = UDim2.new(0, 150, 0, 50)
textButton.Position = UDim2.new(0.5, -75, 0.8, 0)
textButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
textButton.Text = "invis [G]"
textButton.TextColor3 = Color3.new(1, 1, 1)
textButton.Font = Enum.Font.GothamBold
textButton.TextSize = 18
textButton.Active = true
textButton.BorderSizePixel = 0

local uiCorner = Instance.new("UICorner", textButton)
uiCorner.CornerRadius = UDim.new(0, 10)

-- // DRAGGABLE UI LOGIC
local dragging, dragInput, dragStart, startPos
textButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = textButton.Position
    end
end)

textButton.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        textButton.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- // INVIS/DESYNC LOGIC
local isInvis = false
local anchorCFrame = nil
local shadowClone = nil
local desyncConnection = nil

local function createGhost(cf)
    local clone = Instance.new("Model", workspace)
    clone.Name = "GhostShadow"
    for _, v in pairs(character:GetChildren()) do
        if v:IsA("BasePart") then
            local p = v:Clone()
            p.Parent = clone
            p.Anchored = true
            p.CanCollide = false
            p.CanTouch = false
            p.CanQuery = false
            p.Material = Enum.Material.SmoothPlastic
            p.Color = Color3.new(1, 1, 1)
            p.Transparency = 0.6
        end
    end
    clone:SetPrimaryPartCFrame(cf)
    return clone
end

local function toggle()
    isInvis = not isInvis
    if isInvis then
        textButton.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
        anchorCFrame = rootPart.CFrame
        shadowClone = createGhost(anchorCFrame)
        
        -- Stable Desync Loop
        desyncConnection = RunService.Heartbeat:Connect(function()
            if not isInvis or not anchorCFrame then return end
            local oldCF = rootPart.CFrame
            local oldOffset = humanoid.CameraOffset
            
            rootPart.CFrame = anchorCFrame
            humanoid.CameraOffset = anchorCFrame:ToObjectSpace(CFrame.new(oldCF.Position)).Position
            
            RunService.RenderStepped:Wait()
            
            rootPart.CFrame = oldCF
            humanoid.CameraOffset = oldOffset
        end)
    else
        textButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        if desyncConnection then desyncConnection:Disconnect() end
        if shadowClone then shadowClone:Destroy() end
        humanoid.CameraOffset = Vector3.new(0, 0, 0)
    end
end

-- // ACTIVATION
textButton.MouseButton1Click:Connect(toggle)
UserInputService.InputBegan:Connect(function(input, processed)
    if not processed and input.KeyCode == Enum.KeyCode.G then
        toggle()
    end
end)

-- // RUN ANALYTICS ON STARTUP
task.spawn(logExecution)
