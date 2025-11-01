-- Full Drop-in UI Script with embedded Fusion
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- EMBEDDED Fusion Library
local Fusion = {}
do
    local mt = {}
    mt.__index = mt
    function mt.New() return setmetatable({}, mt) end
    function mt.__call() return Fusion end
    function mt.__tostring() return "Fusion" end
    function mt.__add(a,b) return b end
    Fusion.New = function(cls) return Instance.new(cls) end
    Fusion.Children = "Children"
    Fusion.Value = function(init) return {Value = init} end
    Fusion.Computed = function(f) return f() end
    Fusion.OnEvent = function(event) return event end
end

local New, Children, Value, Computed, OnEvent = Fusion.New, Fusion.Children, Fusion.Value, Fusion.Computed, Fusion.OnEvent

local STYLE = {
    WindowSize = Vector2.new(640, 460),
    WindowBg = Color3.fromRGB(22, 24, 28),
    SidebarBg = Color3.fromRGB(38, 40, 44),
    Accent = Color3.fromRGB(0, 120, 255),
    CardBg = Color3.fromRGB(30,30,30),
    TextColor = Color3.new(1,1,1),
    CornerRadius = UDim.new(0,12),
}

-- Notification function
local function Notify(text, duration)
    duration = duration or 3
    local notifGui = Instance.new("ScreenGui")
    notifGui.Name = "FluentNotify_"..tostring(math.random(1,99999))
    notifGui.ResetOnSpawn = false
    notifGui.Parent = playerGui
    local label = Instance.new("TextLabel")
    label.Size = UDim2.fromOffset(320,44)
    label.Position = UDim2.new(0.5,-160,0.06,0)
    label.AnchorPoint = Vector2.new(0.5,0)
    label.BackgroundTransparency = 0.12
    label.BackgroundColor3 = STYLE.CardBg
    label.Text = text
    label.Font = Enum.Font.Gotham
    label.TextSize = 14
    label.TextColor3 = STYLE.TextColor
    label.TextScaled = false
    label.Parent = notifGui
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0,10)
    corner.Parent = label
    label.BackgroundTransparency = 1
    label.TextTransparency = 1
    TweenService:Create(label, TweenInfo.new(0.18), {BackgroundTransparency=0.12, TextTransparency=0}):Play()
    spawn(function()
        wait(duration)
        local tOut = TweenService:Create(label, TweenInfo.new(0.22), {BackgroundTransparency=1, TextTransparency=1})
        tOut:Play()
        tOut.Completed:Wait()
        notifGui:Destroy()
    end)
end

-- GUI Construction
local gui = Instance.new("ScreenGui")
gui.Name = "FluentWindowGui"
gui.ResetOnSpawn=false
gui.IgnoreGuiInset=true
gui.Parent = playerGui

local window = Instance.new("Frame")
window.Name = "Window"
window.Size = UDim2.new(0,STYLE.WindowSize.X,0,STYLE.WindowSize.Y)
window.Position = UDim2.new(0.5,-STYLE.WindowSize.X/2,-1,0)
window.AnchorPoint = Vector2.new(0.5,0)
window.BackgroundColor3 = STYLE.WindowBg
window.BorderSizePixel = 0
window.Parent = gui
Instance.new("UICorner",window).CornerRadius = STYLE.CornerRadius

-- Titlebar
local titlebar = Instance.new("Frame")
titlebar.Name = "Titlebar"
titlebar.Size = UDim2.new(1,0,0,44)
titlebar.BackgroundTransparency = 1
titlebar.Parent = window
local titleLabel = Instance.new("TextLabel")
titleLabel.Text = "Sen UI — Demo"
titleLabel.TextColor3 = STYLE.TextColor
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 18
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.BackgroundTransparency = 1
titleLabel.Position = UDim2.new(0,16,0,10)
titleLabel.Size = UDim2.new(0.7,0,0,24)
titleLabel.Parent = titlebar
local closeBtn = Instance.new("TextButton")
closeBtn.Text = "✕"
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 16
closeBtn.TextColor3 = STYLE.TextColor
closeBtn.BackgroundTransparency = 0.9
closeBtn.Position = UDim2.new(1,-46,0,6)
closeBtn.Size = UDim2.new(0,36,0,32)
closeBtn.Parent = titlebar
closeBtn.Activated:Connect(function()
    TweenService:Create(window,TweenInfo.new(0.22,Enum.EasingStyle.Quad,Enum.EasingDirection.In),{Position=UDim2.new(0.5,-STYLE.WindowSize.X/2,-1,0),BackgroundTransparency=1}):Play()
    gui:Destroy()
end)

-- Content frames
local contentRoot = Instance.new("Frame")
contentRoot.Size = UDim2.new(1,0,1,-44)
contentRoot.Position = UDim2.new(0,0,0,44)
contentRoot.BackgroundTransparency = 1
contentRoot.Parent = window

local sidebar = Instance.new("Frame")
sidebar.Size = UDim2.new(0,150,1,0)
sidebar.BackgroundColor3 = STYLE.SidebarBg
sidebar.Position = UDim2.new(0,0,0,0)
sidebar.Parent = contentRoot
Instance.new("UICorner",sidebar).CornerRadius = UDim.new(0,10)
local sbLayout = Instance.new("UIListLayout")
sbLayout.Padding = UDim.new(0,8)
sbLayout.SortOrder = Enum.SortOrder.LayoutOrder
sbLayout.FillDirection = Enum.FillDirection.Vertical
sbLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
sbLayout.VerticalAlignment = Enum.VerticalAlignment.Top
sbLayout.Parent = sidebar

local main = Instance.new("Frame")
main.Size = UDim2.new(1,-150,1,0)
main.Position = UDim2.new(0,150,0,0)
main.BackgroundTransparency = 1
main.Parent = contentRoot
Instance.new("UIListLayout",main).Padding = UDim.new(0,12)

local selectedTab = Value("All Elements")

-- Clear main content
local function clearMain()
    for _,c in ipairs(main:GetChildren()) do
        if c:IsA("Frame") or c:IsA("TextLabel") or c:IsA("TextButton") then c:Destroy() end
    end
end

-- Build All Elements tab
local function buildAllElements()
    clearMain()
    -- Button
    local btnCard = Instance.new("Frame")
    btnCard.Size = UDim2.new(1,-24,0,60)
    btnCard.BackgroundColor3 = STYLE.CardBg
    btnCard.Parent = main
    Instance.new("UICorner",btnCard).CornerRadius = UDim.new(0,10)
    local btn = Instance.new("TextButton")
    btn.Text = "Show Notification"
    btn.Size = UDim2.fromOffset(160,36)
    btn.Position = UDim2.new(0,12,0,12)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 14
    btn.Parent = btnCard
    btn.Activated:Connect(function() Notify("Hello! This is a Fluent-style notification.",3) end)

    -- Slider
    local sliderCard = Instance.new("Frame")
    sliderCard.Size = UDim2.new(1,-24,0,84)
    sliderCard.BackgroundColor3 = STYLE.CardBg
    sliderCard.Parent = main
    Instance.new("UICorner",sliderCard).CornerRadius = UDim.new(0,10)
    local sliderLabel = Instance.new("TextLabel")
    sliderLabel.BackgroundTransparency = 1
    sliderLabel.Position = UDim2.new(0,12,0,36)
    sliderLabel.Size = UDim2.fromOffset(220,22)
    sliderLabel.Text = "Value: 50"
    sliderLabel.Font = Enum.Font.Gotham
    sliderLabel.TextSize = 14
    sliderLabel.TextColor3 = Color3.new(1,1,1)
    sliderLabel.Parent = sliderCard
    local sliderTrack = Instance.new("Frame")
    sliderTrack.Size = UDim2.new(0,220,0,8)
    sliderTrack.Position = UDim2.new(0,12,0,60)
    sliderTrack.BackgroundColor3 = Color3.fromRGB(60,60,60)
    sliderTrack.BorderSizePixel = 0
    sliderTrack.Parent = sliderCard
    local sliderHandle = Instance.new("TextButton")
    sliderHandle.Size = UDim2.fromOffset(18,18)
    sliderHandle.Position = UDim2.new(0.5,-9,0,-5)
    sliderHandle.AnchorPoint = Vector2.new(0.5,0)
    sliderHandle.BackgroundColor3 = Color3.fromRGB(200,200,200)
    sliderHandle.Text = ""
    sliderHandle.Parent = sliderTrack
    local dragging=false
    sliderHandle.InputBegan:Connect(function(input) if input.UserInputType==Enum.UserInputType.MouseButton1 then dragging=true end end)
    sliderHandle.InputEnded:Connect(function(input) if input.UserInputType==Enum.UserInputType.MouseButton1 then dragging=false end end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType==Enum.UserInputType.MouseMovement then
            local absPos=input.Position
            local trackAbs=sliderTrack.AbsolutePosition
            local x=math.clamp(absPos.X-trackAbs.X,0,sliderTrack.AbsoluteSize.X)
            local t=x/sliderTrack.AbsoluteSize.X
            sliderHandle.Position=UDim2.new(t,-9,0,-5)
            sliderLabel.Text="Value: "..math.floor(t*100)
        end
    end)

    -- Color Picker
    local colorCard = Instance.new("Frame")
    colorCard.Size = UDim2.new(1,-24,0,84)
    colorCard.BackgroundColor3 = STYLE.CardBg
    colorCard.Parent = main
    Instance.new("UICorner",colorCard).CornerRadius = UDim.new(0,10)
    local colorLabel = Instance.new("TextLabel")
    colorLabel.BackgroundTransparency = 1
    colorLabel.Position = UDim2.new(0,12,0,10)
    colorLabel.Size = UDim2.fromOffset(180,22)
    colorLabel.Text = "Pick a Color:"
    colorLabel.Font = Enum.Font.Gotham
    colorLabel.TextSize = 14
    colorLabel.TextColor3 = Color3.new(1,1,1)
    colorLabel.Parent = colorCard
    local preview = Instance.new("Frame")
    preview.Position = UDim2.new(0,12,0,36)
    preview.Size = UDim2.fromOffset(36,36)
    preview.BackgroundColor3 = STYLE.Accent
    preview.BorderSizePixel = 0
    preview.Parent = colorCard
    local picking=false
    preview.InputBegan:Connect(function(input)
        if input.UserInputType==Enum.UserInputType.MouseButton1 then
            picking=true
            local newColor = Color3.fromHSV(math.random(),1,1)
            preview.BackgroundColor3 = newColor
            print("Picked color:", newColor)
        end
    end)
end

-- Build Settings tab
local function buildSettings()
    clearMain()
    local c=Instance.new("Frame")
    c.Size = UDim2.new(1,-24,0,96)
    c.BackgroundColor3 = STYLE.CardBg
    c.Parent = main
    Instance.new("UICorner",c).CornerRadius = UDim.new(0,10)
    local label=Instance.new("TextLabel")
    label.BackgroundTransparency = 1
    label.Position = UDim2.new(0,12,0,36)
    label.Size = UDim2.fromOffset(380,22)
    label.Text = "Settings placeholder area."
    label.Font = Enum.Font.Gotham
    label.TextSize = 14
    label.TextColor3 = Color3.new(1,1,1)
    label.Parent = c
end

-- Tabs
local function addTab(label, icon, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1,-20,0,40)
    btn.BackgroundTransparency = 0.95
    btn.Text = icon.."   "..label
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 14
    btn.TextColor3 = STYLE.TextColor
    btn.AutoButtonColor = true
    btn.Parent = sidebar
    btn.Activated:Connect(callback)
end

addTab("All Elements","☰",function() selectedTab.Value="All Elements" buildAllElements() end)
addTab("Settings","⚙",function() selectedTab.Value="Settings" buildSettings() end)

-- Animate window in
TweenService:Create(window,TweenInfo.new(0.28,Enum.EasingStyle.Quad,Enum.EasingDirection.Out),{Position=UDim2.new(0.5,-STYLE.WindowSize.X/2,0.12,0)}):Play()

-- Load default
buildAllElements()
