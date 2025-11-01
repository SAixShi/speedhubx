local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local Fusion = require(ReplicatedStorage:WaitForChild("Fusion"))
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

local gui = New("ScreenGui"){
    Name="FluentWindowGui",
    ResetOnSpawn=false,
    IgnoreGuiInset=true,
    [Children] = {
        New("Frame"){
            Name="Window",
            Size=UDim2.new(0,STYLE.WindowSize.X,0,STYLE.WindowSize.Y),
            Position=UDim2.new(0.5,-STYLE.WindowSize.X/2,0.12,0),
            AnchorPoint=Vector2.new(0.5,0),
            BackgroundColor3=STYLE.WindowBg,
            BorderSizePixel=0,
            [Children]={
                New("UICorner"){CornerRadius=STYLE.CornerRadius},
                New("UIGradient"){Rotation=90, Color=ColorSequence.new{{0,STYLE.WindowBg},{1,Color3.fromRGB(32,32,32)}}, Transparency=NumberSequence.new{{0,0.12},{1,0.4}}},
                New("Frame"){
                    Name="Titlebar",
                    Size=UDim2.new(1,0,0,44),
                    BackgroundTransparency=1,
                    [Children]={
                        New("TextLabel"){
                            Name="Title",
                            Position=UDim2.new(0,16,0,10),
                            Size=UDim2.new(0.7,0,0,24),
                            BackgroundTransparency=1,
                            Text="Sen UI — Demo",
                            TextColor3=STYLE.TextColor,
                            Font=Enum.Font.GothamBold,
                            TextSize=18,
                            TextXAlignment=Enum.TextXAlignment.Left
                        },
                        New("TextButton"){
                            Name="Close",
                            Position=UDim2.new(1,-46,0,6),
                            Size=UDim2.new(0,36,0,32),
                            BackgroundTransparency=0.9,
                            Text="✕",
                            Font=Enum.Font.GothamBold,
                            TextSize=16,
                            TextColor3=STYLE.TextColor,
                            AutoButtonColor=true,
                            [OnEvent("Activated")]=function()
                                local win=gui:FindFirstChild("Window")
                                if win then
                                    local tween=TweenService:Create(win,TweenInfo.new(0.22,Enum.EasingStyle.Quad,Enum.EasingDirection.In),{Position=UDim2.new(0.5,-STYLE.WindowSize.X/2,-1,0),BackgroundTransparency=1})
                                    tween:Play()
                                    tween.Completed:Wait()
                                end
                                gui:Destroy()
                            end
                        }
                    }
                },
                New("Frame"){
                    Name="ContentRoot",
                    Position=UDim2.new(0,0,0,44),
                    Size=UDim2.new(1,0,1,-44),
                    BackgroundTransparency=1,
                    [Children]={
                        New("Frame"){
                            Name="Sidebar",
                            Position=UDim2.new(0,0,0,0),
                            Size=UDim2.new(0,150,1,0),
                            BackgroundColor3=STYLE.SidebarBg,
                            BorderSizePixel=0,
                            [Children]={
                                New("UICorner"){CornerRadius=UDim.new(0,10)},
                                New("UIListLayout"){Padding=UDim.new(0,8), SortOrder=Enum.SortOrder.LayoutOrder, FillDirection=Enum.FillDirection.Vertical, HorizontalAlignment=Enum.HorizontalAlignment.Center, VerticalAlignment=Enum.VerticalAlignment.Top}
                            }
                        },
                        New("Frame"){
                            Name="Main",
                            Position=UDim2.new(0,150,0,0),
                            Size=UDim2.new(1,-150,1,0),
                            BackgroundTransparency=1,
                            [Children]={New("UIListLayout"){Padding=UDim.new(0,12), SortOrder=Enum.SortOrder.LayoutOrder, FillDirection=Enum.FillDirection.Vertical}}
                        }
                    }
                }
            }
        }
    }
}
gui.Parent=playerGui

local sidebar=gui.Window.ContentRoot.Sidebar
local main=gui.Window.ContentRoot.Main
local selectedTab=Value("All Elements")

local function clearMain()
    for _,c in ipairs(main:GetChildren()) do
        if c:IsA("Frame") or c:IsA("TextLabel") or c:IsA("TextButton") then c:Destroy() end
    end
end

local function buildAllElements()
    clearMain()
    local card=New("Frame"){Size=UDim2.new(1, -24,0,96), BackgroundColor3=STYLE.CardBg, BorderSizePixel=0, [Children]={New("UICorner"){CornerRadius=UDim.new(0,10)}}}
    card.Parent=main
    local btn=Instance.new("TextButton")
    btn.Size=UDim2.fromOffset(160,36)
    btn.Position=UDim2.new(0,12,0,12)
    btn.Text="Show Notification"
    btn.Font=Enum.Font.Gotham
    btn.TextSize=14
    btn.Parent=card
    btn.Activated:Connect(function() Notify("Hello! This is a Fluent-style notification.",3) end)

    local sliderCard=New("Frame"){Size=UDim2.new(1,-24,0,84), BackgroundColor3=STYLE.CardBg, BorderSizePixel=0, [Children]={New("UICorner"){CornerRadius=UDim.new(0,10)}}}
    sliderCard.Parent=main
    local sliderLabel=Instance.new("TextLabel")
    sliderLabel.BackgroundTransparency=1
    sliderLabel.Position=UDim2.new(0,12,0,36)
    sliderLabel.Size=UDim2.fromOffset(220,22)
    sliderLabel.Text="Value: 50"
    sliderLabel.Font=Enum.Font.Gotham
    sliderLabel.TextSize=14
    sliderLabel.TextColor3=Color3.new(1,1,1)
    sliderLabel.Parent=sliderCard
    local sliderTrack=Instance.new("Frame")
    sliderTrack.Size=UDim2.new(0,220,0,8)
    sliderTrack.Position=UDim2.new(0,12,0,60)
    sliderTrack.BackgroundColor3=Color3.fromRGB(60,60,60)
    sliderTrack.BorderSizePixel=0
    sliderTrack.Parent=sliderCard
    local sliderHandle=Instance.new("TextButton")
    sliderHandle.Size=UDim2.fromOffset(18,18)
    sliderHandle.Position=UDim2.new(0.5,-9,0,-5)
    sliderHandle.AnchorPoint=Vector2.new(0.5,0)
    sliderHandle.BackgroundColor3=Color3.fromRGB(200,200,200)
    sliderHandle.Text=""
    sliderHandle.Parent=sliderTrack
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

    local colorCard=New("Frame"){Size=UDim2.new(1,-24,0,84), BackgroundColor3=STYLE.CardBg, BorderSizePixel=0, [Children]={New("UICorner"){CornerRadius=UDim.new(0,10)}}}
    colorCard.Parent=main
    local colorLabel=Instance.new("TextLabel")
    colorLabel.BackgroundTransparency=1
    colorLabel.Position=UDim2.new(0,12,0,10)
    colorLabel.Size=UDim2.fromOffset(180,22)
    colorLabel.Text="Pick a Color:"
    colorLabel.Font=Enum.Font.Gotham
    colorLabel.TextSize=14
    colorLabel.TextColor3=Color3.new(1,1,1)
    colorLabel.Parent=colorCard
    local preview=Instance.new("Frame")
    preview.Position=UDim2.new(0,12,0,36)
    preview.Size=UDim2.fromOffset(36,36)
    preview.BackgroundColor3=STYLE.Accent
    preview.BorderSizePixel=0
    preview.Parent=colorCard
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

local function buildSettings()
    clearMain()
    local c=New("Frame"){Size=UDim2.new(1,-24,0,96), BackgroundColor3=STYLE.CardBg, BorderSizePixel=0, [Children]={New("UICorner"){CornerRadius=UDim.new(0,10)}}}
    c.Parent=main
    local label=Instance.new("TextLabel")
    label.BackgroundTransparency=1
    label.Position=UDim2.new(0,12,0,36)
    label.Size=UDim2.fromOffset(380,22)
    label.Text="Settings placeholder area."
    label.Font=Enum.Font.Gotham
    label.TextSize=14
    label.TextColor3=Color3.new(1,1,1)
    label.Parent=c
end

buildAllElements()

local function addTab(label, icon, callback)
    local btn=New("TextButton"){Size=UDim2.new(1,-20,0,40), BackgroundTransparency=0.95, Text=icon.."   "..label, Font=Enum.Font.Gotham, TextSize=14, TextColor3=STYLE.TextColor, AutoButtonColor=true, [OnEvent("Activated")]=callback}
    btn.Parent=sidebar
end

addTab("All Elements","☰",function() selectedTab:set("All Elements") buildAllElements() end)
addTab("Settings","⚙",function() selectedTab:set("Settings") buildSettings() end)

local window=gui.Window
window.Position=UDim2.new(0.5,-STYLE.WindowSize.X/2,-1,0)
TweenService:Create(window,TweenInfo.new(0.28,Enum.EasingStyle.Quad,Enum.EasingDirection.Out),{Position=UDim2.new(0.5,-STYLE.WindowSize.X/2,0.12,0)}):Play()
