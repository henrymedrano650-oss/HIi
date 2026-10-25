-------------------------------------------------
-- SERVICES
-------------------------------------------------
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local MarketplaceService = game:GetService("MarketplaceService")
local Lighting = game:GetService("Lighting")
local TeleportService = game:GetService("TeleportService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-------------------------------------------------
-- CONFIG
-------------------------------------------------
local CORRECT_KEY = "Hallowen#1"
local GET_KEY_LINK = "https://work.ink/2inB/83465860-b047-49a3-b03e-a4f4b917b1d6"
local SAVE_FOLDER = "HalloweenHub"
local KEY_FILE = SAVE_FOLDER .. "/key.txt"

local DEFAULT_WALKSPEED = 16
local DEFAULT_JUMPPOWER = 50
local DEFAULT_FOV = 70
local DEFAULT_GRAVITY = workspace.Gravity

local originalLighting = {
    Brightness = Lighting.Brightness,
    ClockTime = Lighting.ClockTime,
    Ambient = Lighting.Ambient,
    OutdoorAmbient = Lighting.OutdoorAmbient,
}

-- State
local walkSpeed = DEFAULT_WALKSPEED
local jumpPower = DEFAULT_JUMPPOWER
local flySpeed = 50
local flying = false
local noclip = false
local espEnabled = false

-------------------------------------------------
-- HELPERS
-------------------------------------------------
local function trim(s)
    return (s:match("^%s*(.-)%s*$"))
end

local function getCharacter()
    return player.Character or player.CharacterAdded:Wait()
end

local function getHumanoid()
    return getCharacter():WaitForChild("Humanoid")
end

local function getRoot()
    return getCharacter():WaitForChild("HumanoidRootPart")
end

local function applyMovement()
    local humanoid = getHumanoid()
    humanoid.WalkSpeed = walkSpeed
    humanoid.UseJumpPower = true
    humanoid.JumpPower = jumpPower
end

-------------------------------------------------
-- KEY SAVING
-- Checks the session cache first, then the file.
-- Needs an executor with isfile/readfile/writefile.
-------------------------------------------------
local function sessionFlag()
    return getgenv and getgenv().HalloweenHubVerified
end

local function hasSavedKey()
    if sessionFlag() then return true end
    local ok, saved = pcall(function()
        if isfile and readfile and isfile(KEY_FILE) then
            return readfile(KEY_FILE)
        end
    end)
    return ok and saved ~= nil and trim(saved) == CORRECT_KEY
end

local function saveKey()
    if getgenv then
        getgenv().HalloweenHubVerified = true
    end
    pcall(function()
        if makefolder and isfolder and not isfolder(SAVE_FOLDER) then
            makefolder(SAVE_FOLDER)
        end
        if writefile then
            writefile(KEY_FILE, CORRECT_KEY)
        end
    end)
end

local function forgetKey()
    if getgenv then
        getgenv().HalloweenHubVerified = nil
    end
    pcall(function()
        if delfile and isfile and isfile(KEY_FILE) then
            delfile(KEY_FILE)
        end
    end)
end

-------------------------------------------------
-- KEY GATE (runs before Rayfield loads)
-------------------------------------------------
local function runKeyGate()
    if hasSavedKey() then return end

    local accepted = false

    local gui = Instance.new("ScreenGui")
    gui.Name = "KeyGate"
    gui.ResetOnSpawn = false
    gui.Parent = player:WaitForChild("PlayerGui")

    local frame = Instance.new("Frame")
    frame.Size = UDim2.fromOffset(340, 250)
    frame.Position = UDim2.new(0.5, -170, 0.5, -125)
    frame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
    frame.Parent = gui
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -20, 0, 40)
    title.Position = UDim2.new(0, 10, 0, 10)
    title.BackgroundTransparency = 1
    title.Text = "Halloween Hub - Key System"
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.Font = Enum.Font.GothamBold
    title.TextSize = 18
    title.Parent = frame

    local status = Instance.new("TextLabel")
    status.Size = UDim2.new(1, -20, 0, 30)
    status.Position = UDim2.new(0, 10, 0, 50)
    status.BackgroundTransparency = 1
    status.Text = "Enter the key to continue."
    status.TextColor3 = Color3.fromRGB(200, 200, 200)
    status.Font = Enum.Font.Gotham
    status.TextSize = 14
    status.TextWrapped = true
    status.Parent = frame

    local input = Instance.new("TextBox")
    input.Size = UDim2.new(1, -20, 0, 40)
    input.Position = UDim2.new(0, 10, 0, 90)
    input.BackgroundColor3 = Color3.fromRGB(45, 45, 52)
    input.TextColor3 = Color3.fromRGB(255, 255, 255)
    input.PlaceholderText = "Enter key"
    input.Text = ""
    input.ClearTextOnFocus = false
    input.Font = Enum.Font.Gotham
    input.TextSize = 16
    input.Parent = frame
    Instance.new("UICorner", input).CornerRadius = UDim.new(0, 6)

    local function makeButton(text, xScale, xOffset, color)
        local b = Instance.new("TextButton")
        b.Size = UDim2.new(0.5, -15, 0, 40)
        b.Position = UDim2.new(xScale, xOffset, 0, 150)
        b.BackgroundColor3 = color
        b.TextColor3 = Color3.fromRGB(255, 255, 255)
        b.Text = text
        b.Font = Enum.Font.GothamBold
        b.TextSize = 16
        b.Parent = frame
        Instance.new("UICorner", b).CornerRadius = UDim.new(0, 6)
        return b
    end

    local enterBtn = makeButton("Enter key", 0, 10, Color3.fromRGB(230, 120, 30))
    local getKeyBtn = makeButton("Get key", 0.5, 5, Color3.fromRGB(70, 120, 220))

    local function checkKey()
        if trim(input.Text) == CORRECT_KEY then
            accepted = true
            saveKey()
            status.Text = "Key accepted!"
        else
            status.Text = "Wrong key. Try again."
        end
    end

    enterBtn.MouseButton1Click:Connect(checkKey)

    input.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            checkKey()
        end
    end)

    getKeyBtn.MouseButton1Click:Connect(function()
        local copied = pcall(function()
            setclipboard(GET_KEY_LINK)
        end)
        if copied then
            status.Text = "Link copied to clipboard. Paste it in your browser."
        else
            status.Text = "Clipboard not available. Link: " .. GET_KEY_LINK
        end
    end)

    repeat task.wait() until accepted
    task.wait(0.5)
    gui:Destroy()
end

runKeyGate()

-------------------------------------------------
-- LOAD RAYFIELD (only after the key is accepted)
-------------------------------------------------
local loaded, Rayfield = pcall(function()
    return loadstring(game:HttpGet("https://sirius.menu/rayfield"))()
end)
if not loaded or not Rayfield then
    warn("Halloween Hub: Rayfield failed to load.")
    return
end

local function notify(title, content)
    Rayfield:Notify({ Title = title, Content = content, Duration = 3 })
end

-------------------------------------------------
-- WINDOW
-------------------------------------------------
local Window = Rayfield:CreateWindow({
    Name = "Halloween Hub",
    LoadingTitle = "Halloween Hub",
    LoadingSubtitle = "Hallowen update",
    ConfigurationSaving = { Enabled = false },
    Discord = { Enabled = false },
    KeySystem = false,
})

-------------------------------------------------
-- MOVEMENT TAB
-------------------------------------------------
local MoveTab = Window:CreateTab("Movement", 4483362458)

MoveTab:CreateSlider({
    Name = "WalkSpeed",
    Range = { 0, 200 },
    Increment = 1,
    Suffix = "speed",
    CurrentValue = DEFAULT_WALKSPEED,
    Flag = "WalkSpeedSlider",
    Callback = function(Value)
        walkSpeed = Value
        getHumanoid().WalkSpeed = Value
    end,
})

MoveTab:CreateSlider({
    Name = "JumpPower",
    Range = { 0, 300 },
    Increment = 1,
    Suffix = "power",
    CurrentValue = DEFAULT_JUMPPOWER,
    Flag = "JumpPowerSlider",
    Callback = function(Value)
        jumpPower = Value
        local humanoid = getHumanoid()
        humanoid.UseJumpPower = true
        humanoid.JumpPower = Value
    end,
})

MoveTab:CreateSlider({
    Name = "Fly Speed",
    Range = { 10, 300 },
    Increment = 5,
    Suffix = "studs/s",
    CurrentValue = flySpeed,
    Flag = "FlySpeedSlider",
    Callback = function(Value)
        flySpeed = Value
    end,
})

MoveTab:CreateToggle({
    Name = "Fly (WASD + Space/Shift)",
    CurrentValue = false,
    Flag = "FlyToggle",
    Callback = function(Value)
        flying = Value
        if not Value then
            getRoot().AssemblyLinearVelocity = Vector3.zero
        end
    end,
})

MoveTab:CreateToggle({
    Name = "Noclip",
    CurrentValue = false,
    Flag = "NoclipToggle",
    Callback = function(Value)
        noclip = Value
        if not Value then
            for _, part in ipairs(getCharacter():GetDescendants()) do
                if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                    part.CanCollide = true
                end
            end
        end
    end,
})

MoveTab:CreateButton({
    Name = "Reset WalkSpeed and JumpPower",
    Callback = function()
        walkSpeed = DEFAULT_WALKSPEED
        jumpPower = DEFAULT_JUMPPOWER
        applyMovement()
        notify("Reset", "WalkSpeed and JumpPower restored.")
    end,
})

-------------------------------------------------
-- VISUALS TAB
-------------------------------------------------
local VisualTab = Window:CreateTab("Visuals", 4483362458)

VisualTab:CreateSlider({
    Name = "Field of View",
    Range = { 30, 120 },
    Increment = 1,
    Suffix = "deg",
    CurrentValue = DEFAULT_FOV,
    Flag = "FOVSlider",
    Callback = function(Value)
        camera.FieldOfView = Value
    end,
})

VisualTab:CreateToggle({
    Name = "Fullbright",
    CurrentValue = false,
    Flag = "FullbrightToggle",
    Callback = function(Value)
        if Value then
            Lighting.Brightness = 2
            Lighting.Ambient = Color3.fromRGB(178, 178, 178)
            Lighting.OutdoorAmbient = Color3.fromRGB(178, 178, 178)
        else
            Lighting.Brightness = originalLighting.Brightness
            Lighting.Ambient = originalLighting.Ambient
            Lighting.OutdoorAmbient = originalLighting.OutdoorAmbient
        end
    end,
})

VisualTab:CreateSlider({
    Name = "Time of Day",
    Range = { 0, 24 },
    Increment = 0.5,
    Suffix = "hour",
    CurrentValue = originalLighting.ClockTime,
    Flag = "TimeSlider",
    Callback = function(Value)
        Lighting.ClockTime = Value
    end,
})

VisualTab:CreateToggle({
    Name = "Player ESP (highlight others)",
    CurrentValue = false,
    Flag = "ESPToggle",
    Callback = function(Value)
        espEnabled = Value
        if not Value then
            for _, p in ipairs(Players:GetPlayers()) do
                local h = p.Character and p.Character:FindFirstChild("HalloweenESP")
                if h then h:Destroy() end
            end
        end
    end,
})

VisualTab:CreateButton({
    Name = "Reset Camera FOV",
    Callback = function()
        camera.FieldOfView = DEFAULT_FOV
        notify("Camera", "FOV reset to " .. DEFAULT_FOV)
    end,
})

-------------------------------------------------
-- TELEPORT TAB
-------------------------------------------------
local TeleportTab = Window:CreateTab("Teleport", 4483362458)

TeleportTab:CreateInput({
    Name = "Teleport to Player (username)",
    PlaceholderText = "Enter username",
    RemoveTextAfterFocusLost = false,
    Callback = function(Text)
        local target = Players:FindFirstChild(trim(Text))
        if not target then
            notify("Teleport", "Player not found: " .. trim(Text))
            return
        end
        local targetRoot = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
        if not targetRoot then
            notify("Teleport", "That player has no character loaded.")
            return
        end
        getRoot().CFrame = targetRoot.CFrame + Vector3.new(3, 0, 0)
        notify("Teleport", "Teleported to " .. target.Name)
    end,
})

TeleportTab:CreateButton({
    Name = "Teleport to Random Player",
    Callback = function()
        local others = {}
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                table.insert(others, p)
            end
        end
        if #others == 0 then
            notify("Teleport", "No other players found.")
            return
        end
        local target = others[math.random(#others)]
        getRoot().CFrame = target.Character.HumanoidRootPart.CFrame + Vector3.new(3, 0, 0)
        notify("Teleport", "Teleported to " .. target.Name)
    end,
})

TeleportTab:CreateButton({
    Name = "Teleport to Origin",
    Callback = function()
        getRoot().CFrame = CFrame.new(0, 5, 0)
        notify("Teleport", "Teleported to origin.")
    end,
})

-------------------------------------------------
-- CHARACTER TAB
-------------------------------------------------
local CharTab = Window:CreateTab("Character", 4483362458)

CharTab:CreateButton({
    Name = "Heal to Full",
    Callback = function()
        local humanoid = getHumanoid()
        humanoid.Health = humanoid.MaxHealth
        notify("Character", "Health restored.")
    end,
})

CharTab:CreateButton({
    Name = "Respawn",
    Callback = function()
        getHumanoid().Health = 0
    end,
})

CharTab:CreateToggle({
    Name = "Sit",
    CurrentValue = false,
    Flag = "SitToggle",
    Callback = function(Value)
        getHumanoid().Sit = Value
    end,
})

CharTab:CreateSlider({
    Name = "Gravity",
    Range = { 0, 300 },
    Increment = 1,
    Suffix = "studs/s²",
    CurrentValue = DEFAULT_GRAVITY,
    Flag = "GravitySlider",
    Callback = function(Value)
        workspace.Gravity = Value
    end,
})

CharTab:CreateButton({
    Name = "Reset Gravity",
    Callback = function()
        workspace.Gravity = DEFAULT_GRAVITY
        notify("Character", "Gravity reset.")
    end,
})

-------------------------------------------------
-- INFO TAB
-------------------------------------------------
local InfoTab = Window:CreateTab("Info", 4483362458)

local function getGameName()
    local ok, info = pcall(function()
        return MarketplaceService:GetProductInfo(game.PlaceId)
    end)
    return (ok and info and info.Name) or "Unknown"
end

InfoTab:CreateParagraph({
    Title = "Player Info",
    Content = table.concat({
        "Username: " .. player.Name,
        "Display Name: " .. player.DisplayName,
        "User ID: " .. player.UserId,
        "Account Age: " .. player.AccountAge .. " days",
        "Membership: " .. player.MembershipType.Name,
        "Team: " .. (player.Team and player.Team.Name or "None"),
    }, "\n"),
})

InfoTab:CreateParagraph({
    Title = "Game Info",
    Content = table.concat({
        "Game: " .. getGameName(),
        "Place ID: " .. game.PlaceId,
        "Game ID: " .. game.GameId,
        "Job ID: " .. game.JobId,
        "Players in Server: " .. #Players:GetPlayers() .. "/" .. Players.MaxPlayers,
    }, "\n"),
})

-------------------------------------------------
-- UTILITY TAB
-------------------------------------------------
local UtilTab = Window:CreateTab("Utility", 4483362458)

UtilTab:CreateButton({
    Name = "Copy User ID",
    Callback = function()
        local copied = pcall(function()
            setclipboard(tostring(player.UserId))
        end)
        notify("User ID", copied and "Copied to clipboard." or tostring(player.UserId))
    end,
})

UtilTab:CreateButton({
    Name = "Copy Job ID",
    Callback = function()
        local copied = pcall(function()
            setclipboard(game.JobId)
        end)
        notify("Job ID", copied and "Copied to clipboard." or game.JobId)
    end,
})

UtilTab:CreateButton({
    Name = "Rejoin Server",
    Callback = function()
        local rejoined = pcall(function()
            TeleportService:Teleport(game.PlaceId, player)
        end)
        if not rejoined then
            notify("Rejoin", "Rejoin failed. Try again.")
        end
    end,
})

UtilTab:CreateButton({
    Name = "Forget Saved Key",
    Callback = function()
        forgetKey()
        notify("Key", "Saved key removed. You will be asked again next run.")
    end,
})

-------------------------------------------------
-- ESP LOOP (adds highlights to other players)
-------------------------------------------------
task.spawn(function()
    while true do
        if espEnabled then
            for _, p in ipairs(Players:GetPlayers()) do
                local char = p.Character
                if p ~= player and char and not char:FindFirstChild("HalloweenESP") then
                    local h = Instance.new("Highlight")
                    h.Name = "HalloweenESP"
                    h.FillColor = Color3.fromRGB(255, 140, 0)
                    h.OutlineColor = Color3.fromRGB(255, 255, 255)
                    h.Parent = char
                end
            end
        end
        task.wait(1)
    end
end)

-------------------------------------------------
-- FLY AND NOCLIP LOOP
-------------------------------------------------
RunService.Heartbeat:Connect(function()
    local character = player.Character
    if not character then return end

    local humanoid = character:FindFirstChildOfClass("Humanoid")
    local root = character:FindFirstChild("HumanoidRootPart")
    if not humanoid or not root then return end

    if noclip then
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") and part.CanCollide then
                part.CanCollide = false
            end
        end
    end

    if flying then
        humanoid.PlatformStand = true

        local camCF = camera.CFrame
        local direction = Vector3.zero

        if UserInputService:IsKeyDown(Enum.KeyCode.W) then
            direction += camCF.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then
            direction -= camCF.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then
            direction += camCF.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then
            direction -= camCF.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            direction += Vector3.yAxis
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
            direction -= Vector3.yAxis
        end

        -- Mobile fallback
        if direction.Magnitude == 0 and humanoid.MoveDirection.Magnitude > 0 then
            direction = humanoid.MoveDirection
        end

        if direction.Magnitude > 0 then
            direction = direction.Unit
        end

        root.AssemblyLinearVelocity = direction * flySpeed
        root.AssemblyAngularVelocity = Vector3.zero
    elseif humanoid.PlatformStand then
        humanoid.PlatformStand = false
    end
end)

-------------------------------------------------
-- RESPAWN HANDLING
-------------------------------------------------
player.CharacterAdded:Connect(function(character)
    local humanoid = character:WaitForChild("Humanoid")
    humanoid.WalkSpeed = walkSpeed
    humanoid.UseJumpPower = true
    humanoid.JumpPower = jumpPower
end)

applyMovement()
Rayfield:LoadConfiguration()
