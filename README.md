local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local function SideNotify(title, text, duration)
    duration = duration or 3
    
    -- Main Container
    local gui = Instance.new("ScreenGui")
    gui.Name = "NotificationGui"
    gui.ResetOnSpawn = false
    gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

    -- Notification Frame
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 254, 0, 82)
    -- Start position (off-screen to the right)
    frame.Position = UDim2.new(1, 50, 0.75, 0) 
    frame.BackgroundColor3 = Color3.fromRGB(23, 23, 23)
    frame.BorderSizePixel = 0
    frame.Parent = gui

    local corner = Instance.new("UICorner", frame)
    corner.CornerRadius = UDim.new(0, 8)

    -- Icon
    local icon = Instance.new("ImageLabel")
    icon.Size = UDim2.new(0, 50, 0, 50)
    icon.Position = UDim2.new(0, 12, 0, 16)
    icon.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    icon.Image = "rbxassetid://133032272724948" -- Fixed ID
    icon.Parent = frame
    
    local iconCorner = Instance.new("UICorner", icon)
    iconCorner.CornerRadius = UDim.new(1, 0)

    -- Title Label
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(0, 160, 0, 30)
    titleLabel.Position = UDim2.new(0, 75, 0, 10)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = title
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.TextSize = 16
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = frame

    -- Description Label
    local descLabel = Instance.new("TextLabel")
    descLabel.Size = UDim2.new(0, 160, 0, 30)
    descLabel.Position = UDim2.new(0, 75, 0, 35)
    descLabel.BackgroundTransparency = 1
    descLabel.Text = text
    descLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    descLabel.TextSize = 13
    descLabel.Font = Enum.Font.Gotham
    descLabel.TextWrapped = true
    descLabel.TextXAlignment = Enum.TextXAlignment.Left
    descLabel.Parent = frame

    -- Entry Animation
    TweenService:Create(
        frame,
        TweenInfo.new(0.5, Enum.EasingStyle.Quart, Enum.EasingDirection.Out),
        {Position = UDim2.new(1, -270, 0.75, 0)} -- Slides into view
    ):Play()

    -- Exit Animation
    task.delay(duration, function()
        local exit = TweenService:Create(
            frame,
            TweenInfo.new(0.5, Enum.EasingStyle.Quart, Enum.EasingDirection.In),
            {Position = UDim2.new(1, 50, 0.75, 0)}
        )
        exit:Play()
        exit.Completed:Wait()
        gui:Destroy()
    end)
end

-- Example Usage:
SideNotify("Redz Hub V2", "Nova interface e melhorias de desempenho!", 5)


