--[[
    ╔══════════════════════════════════════════╗
    ║          XITERPRISON v1.1                ║
    ║    Script para Prison Life - Roblox      ║
    ║                                          ║
    ║    Créditos: Thurzinnn & Thalles         ║
    ╚══════════════════════════════════════════╝
]]

-- Serviços
local Players = game:GetService("Players")
local Teams = game:GetService("Teams")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- ═══════════════════════════════════════════
-- ESTADO GLOBAL DO SCRIPT
-- ═══════════════════════════════════════════
local State = {
    -- Combat
    AimbotEnabled = false,
    TriggerbotEnabled = false,
    FOVEnabled = false,
    FOVSize = 150,
    AimStrength = 0.5,
    TeamCheckCombat = false,
    
    -- Visuals
    ESPEnabled = false,
    BoxfieldEnabled = false,
    LifeStatusEnabled = false,
    TracerEnabled = false,
    ESPRange = 500,
    TeamCheckVisuals = false,
    
    -- Player
    InfiniteAmmoEnabled = false,
    GodModeEnabled = false,
    SpeedhackEnabled = false,
    SpeedValue = 16,
    FlyHackEnabled = false,
    FlySpeedValue = 50,
    NoClipEnabled = false,
    
    -- Anti-Lag
    AntiLagEnabled = false,
    
    -- UI
    CurrentTab = "Home",
    Minimized = false,
    Flying = false,
    FlyBody = nil,
}

-- ═══════════════════════════════════════════
-- CORES DO TEMA PRISIONAL
-- ═══════════════════════════════════════════
local Theme = {
    Background = Color3.fromRGB(25, 25, 28),
    BackgroundDark = Color3.fromRGB(18, 18, 21),
    Sidebar = Color3.fromRGB(30, 30, 34),
    SidebarHover = Color3.fromRGB(40, 40, 45),
    Primary = Color3.fromRGB(230, 126, 34),
    PrimaryDark = Color3.fromRGB(200, 106, 24),
    PrimaryLight = Color3.fromRGB(255, 156, 64),
    Secondary = Color3.fromRGB(85, 85, 95),
    SecondaryLight = Color3.fromRGB(110, 110, 120),
    Text = Color3.fromRGB(235, 235, 240),
    TextDim = Color3.fromRGB(160, 160, 170),
    TextMuted = Color3.fromRGB(110, 110, 120),
    Success = Color3.fromRGB(46, 204, 113),
    Danger = Color3.fromRGB(231, 76, 60),
    CardBg = Color3.fromRGB(35, 35, 40),
    CardBgHover = Color3.fromRGB(42, 42, 48),
    Border = Color3.fromRGB(55, 55, 62),
    Shadow = Color3.fromRGB(0, 0, 0),
    ToggleOff = Color3.fromRGB(60, 60, 68),
    ToggleOn = Color3.fromRGB(230, 126, 34),
    SliderBg = Color3.fromRGB(50, 50, 58),
    SliderFill = Color3.fromRGB(230, 126, 34),
}

-- ═══════════════════════════════════════════
-- FUNÇÕES DE TIME (TEAM CHECK)
-- ═══════════════════════════════════════════
local function GetPlayerTeam(player)
    if player and player.Team then
        return player.Team
    end
    return nil
end

local function GetTeamColor(player)
    local team = GetPlayerTeam(player)
    if team then
        return team.TeamColor.Color
    end
    return Theme.Primary
end

local function GetTeamName(player)
    local team = GetPlayerTeam(player)
    if team then
        return team.Name
    end
    return "Sem Time"
end

local function IsSameTeam(player1, player2)
    local team1 = GetPlayerTeam(player1)
    local team2 = GetPlayerTeam(player2)
    if team1 and team2 then
        return team1 == team2
    end
    return false
end

local function ShouldTargetPlayer(player)
    if State.TeamCheckCombat then
        return not IsSameTeam(LocalPlayer, player)
    end
    return true
end

local function GetESPColor(player)
    if State.TeamCheckVisuals then
        return GetTeamColor(player)
    end
    return Theme.Primary
end

-- ═══════════════════════════════════════════
-- UTILITÁRIOS DE ANIMAÇÃO
-- ═══════════════════════════════════════════
local function Tween(obj, props, duration, style, direction)
    local tweenInfo = TweenInfo.new(
        duration or 0.3,
        style or Enum.EasingStyle.Quart,
        direction or Enum.EasingDirection.Out
    )
    local tween = TweenService:Create(obj, tweenInfo, props)
    tween:Play()
    return tween
end

local function CreateShadow(parent, transparency, size)
    local shadow = Instance.new("ImageLabel")
    shadow.Name = "Shadow"
    shadow.BackgroundTransparency = 1
    shadow.Image = "rbxassetid://6015897843"
    shadow.ImageColor3 = Theme.Shadow
    shadow.ImageTransparency = transparency or 0.5
    shadow.ScaleType = Enum.ScaleType.Slice
    shadow.SliceCenter = Rect.new(49, 49, 450, 450)
    shadow.Size = UDim2.new(1, size or 30, 1, size or 30)
    shadow.Position = UDim2.new(0, -(size or 30) / 2, 0, -(size or 30) / 2)
    shadow.ZIndex = parent.ZIndex - 1
    shadow.Parent = parent
    return shadow
end

local function CreateCorner(parent, radius)
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, radius or 8)
    corner.Parent = parent
    return corner
end

local function CreateStroke(parent, color, thickness, transparency)
    local stroke = Instance.new("UIStroke")
    stroke.Color = color or Theme.Border
    stroke.Thickness = thickness or 1
    stroke.Transparency = transparency or 0.5
    stroke.Parent = parent
    return stroke
end

local function CreatePadding(parent, top, bottom, left, right)
    local padding = Instance.new("UIPadding")
    padding.PaddingTop = UDim.new(0, top or 8)
    padding.PaddingBottom = UDim.new(0, bottom or 8)
    padding.PaddingLeft = UDim.new(0, left or 8)
    padding.PaddingRight = UDim.new(0, right or 8)
    padding.Parent = parent
    return padding
end

-- ═══════════════════════════════════════════
-- DESTRUIR GUI ANTERIOR (SE EXISTIR)
-- ═══════════════════════════════════════════
if game.CoreGui:FindFirstChild("XiterprisonGUI") then
    game.CoreGui:FindFirstChild("XiterprisonGUI"):Destroy()
end

-- ═══════════════════════════════════════════
-- CRIAÇÃO DA GUI PRINCIPAL
-- ═══════════════════════════════════════════
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "XiterprisonGUI"
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game.CoreGui

-- ═══════════════════════════════════════════
-- JANELA PRINCIPAL
-- ═══════════════════════════════════════════
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 720, 0, 480)
MainFrame.Position = UDim2.new(0.5, -360, 0.5, -240)
MainFrame.BackgroundColor3 = Theme.Background
MainFrame.BorderSizePixel = 0
MainFrame.ZIndex = 2
MainFrame.Parent = ScreenGui
CreateCorner(MainFrame, 12)
CreateStroke(MainFrame, Theme.Border, 1, 0.6)
CreateShadow(MainFrame, 0.4, 50)

-- Efeito de abertura
MainFrame.BackgroundTransparency = 1
MainFrame.Size = UDim2.new(0, 680, 0, 440)
Tween(MainFrame, {
    BackgroundTransparency = 0,
    Size = UDim2.new(0, 720, 0, 480)
}, 0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out)

-- ═══════════════════════════════════════════
-- SISTEMA DE ARRASTAR JANELA
-- ═══════════════════════════════════════════
local Dragging = false
local DragInput, DragStart, StartPos

local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 46)
TitleBar.BackgroundColor3 = Theme.BackgroundDark
TitleBar.BorderSizePixel = 0
TitleBar.ZIndex = 3
TitleBar.Parent = MainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 12)
titleCorner.Parent = TitleBar

local TitleFill = Instance.new("Frame")
TitleFill.Name = "TitleFill"
TitleFill.Size = UDim2.new(1, 0, 0, 14)
TitleFill.Position = UDim2.new(0, 0, 1, -14)
TitleFill.BackgroundColor3 = Theme.BackgroundDark
TitleFill.BorderSizePixel = 0
TitleFill.ZIndex = 3
TitleFill.Parent = TitleBar

local AccentLine = Instance.new("Frame")
AccentLine.Name = "AccentLine"
AccentLine.Size = UDim2.new(1, 0, 0, 2)
AccentLine.Position = UDim2.new(0, 0, 1, 0)
AccentLine.BackgroundColor3 = Theme.Primary
AccentLine.BorderSizePixel = 0
AccentLine.ZIndex = 4
AccentLine.Parent = TitleBar

local accentGrad = Instance.new("UIGradient")
accentGrad.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Theme.Primary),
    ColorSequenceKeypoint.new(0.5, Theme.PrimaryLight),
    ColorSequenceKeypoint.new(1, Theme.Primary)
})
accentGrad.Parent = AccentLine

local PrisonIcon = Instance.new("TextLabel")
PrisonIcon.Name = "PrisonIcon"
PrisonIcon.Size = UDim2.new(0, 32, 0, 32)
PrisonIcon.Position = UDim2.new(0, 14, 0.5, -16)
PrisonIcon.BackgroundTransparency = 1
PrisonIcon.Text = "⛓"
PrisonIcon.TextColor3 = Theme.Primary
PrisonIcon.TextSize = 20
PrisonIcon.Font = Enum.Font.GothamBold
PrisonIcon.ZIndex = 4
PrisonIcon.Parent = TitleBar

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Name = "TitleLabel"
TitleLabel.Size = UDim2.new(0, 200, 1, 0)
TitleLabel.Position = UDim2.new(0, 48, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "Xiterprison"
TitleLabel.TextColor3 = Theme.Primary
TitleLabel.TextSize = 20
TitleLabel.Font = Enum.Font.GothamBlack
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.ZIndex = 4
TitleLabel.Parent = TitleBar

local VersionLabel = Instance.new("TextLabel")
VersionLabel.Name = "VersionLabel"
VersionLabel.Size = UDim2.new(0, 40, 0, 18)
VersionLabel.Position = UDim2.new(0, 178, 0.5, -9)
VersionLabel.BackgroundColor3 = Theme.Primary
VersionLabel.BackgroundTransparency = 0.8
VersionLabel.Text = "v1.1"
VersionLabel.TextColor3 = Theme.PrimaryLight
VersionLabel.TextSize = 11
VersionLabel.Font = Enum.Font.GothamBold
VersionLabel.ZIndex = 4
VersionLabel.Parent = TitleBar
CreateCorner(VersionLabel, 4)

-- Botão Minimizar
local MinimizeBtn = Instance.new("TextButton")
MinimizeBtn.Name = "MinimizeBtn"
MinimizeBtn.Size = UDim2.new(0, 32, 0, 32)
MinimizeBtn.Position = UDim2.new(1, -76, 0.5, -16)
MinimizeBtn.BackgroundColor3 = Theme.Secondary
MinimizeBtn.BackgroundTransparency = 0.7
MinimizeBtn.Text = "—"
MinimizeBtn.TextColor3 = Theme.TextDim
MinimizeBtn.TextSize = 16
MinimizeBtn.Font = Enum.Font.GothamBold
MinimizeBtn.ZIndex = 5
MinimizeBtn.Parent = TitleBar
CreateCorner(MinimizeBtn, 6)

MinimizeBtn.MouseEnter:Connect(function()
    Tween(MinimizeBtn, {BackgroundTransparency = 0.4, BackgroundColor3 = Theme.Secondary}, 0.2)
end)
MinimizeBtn.MouseLeave:Connect(function()
    Tween(MinimizeBtn, {BackgroundTransparency = 0.7, BackgroundColor3 = Theme.Secondary}, 0.2)
end)

-- Botão Fechar
local CloseBtn = Instance.new("TextButton")
CloseBtn.Name = "CloseBtn"
CloseBtn.Size = UDim2.new(0, 32, 0, 32)
CloseBtn.Position = UDim2.new(1, -40, 0.5, -16)
CloseBtn.BackgroundColor3 = Theme.Danger
CloseBtn.BackgroundTransparency = 0.7
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Theme.TextDim
CloseBtn.TextSize = 14
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.ZIndex = 5
CloseBtn.Parent = TitleBar
CreateCorner(CloseBtn, 6)

CloseBtn.MouseEnter:Connect(function()
    Tween(CloseBtn, {BackgroundTransparency = 0.2, BackgroundColor3 = Theme.Danger, TextColor3 = Theme.Text}, 0.2)
end)
CloseBtn.MouseLeave:Connect(function()
    Tween(CloseBtn, {BackgroundTransparency = 0.7, BackgroundColor3 = Theme.Danger, TextColor3 = Theme.TextDim}, 0.2)
end)

-- Funcionalidade de arrastar
TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        Dragging = true
        DragStart = input.Position
        StartPos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                Dragging = false
            end
        end)
    end
end)

TitleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        DragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == DragInput and Dragging then
        local delta = input.Position - DragStart
        MainFrame.Position = UDim2.new(
            StartPos.X.Scale, StartPos.X.Offset + delta.X,
            StartPos.Y.Scale, StartPos.Y.Offset + delta.Y
        )
    end
end)

-- Minimizar
local ContentHolder = nil

MinimizeBtn.MouseButton1Click:Connect(function()
    State.Minimized = not State.Minimized
    if State.Minimized then
        Tween(MainFrame, {Size = UDim2.new(0, 720, 0, 48)}, 0.35, Enum.EasingStyle.Quart)
        MinimizeBtn.Text = "+"
    else
        Tween(MainFrame, {Size = UDim2.new(0, 720, 0, 480)}, 0.35, Enum.EasingStyle.Quart)
        MinimizeBtn.Text = "—"
    end
end)

-- Fechar
CloseBtn.MouseButton1Click:Connect(function()
    Tween(MainFrame, {
        Size = UDim2.new(0, 680, 0, 440),
        BackgroundTransparency = 1
    }, 0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.In)
    task.wait(0.35)
    ScreenGui:Destroy()
end)

-- ═══════════════════════════════════════════
-- CORPO PRINCIPAL (SIDEBAR + CONTEÚDO)
-- ═══════════════════════════════════════════
local Body = Instance.new("Frame")
Body.Name = "Body"
Body.Size = UDim2.new(1, 0, 1, -48)
Body.Position = UDim2.new(0, 0, 0, 48)
Body.BackgroundTransparency = 1
Body.ZIndex = 2
Body.ClipsDescendants = true
Body.Parent = MainFrame

-- ═══════════════════════════════════════════
-- SIDEBAR
-- ═══════════════════════════════════════════
local Sidebar = Instance.new("Frame")
Sidebar.Name = "Sidebar"
Sidebar.Size = UDim2.new(0, 170, 1, 0)
Sidebar.BackgroundColor3 = Theme.Sidebar
Sidebar.BorderSizePixel = 0
Sidebar.ZIndex = 3
Sidebar.Parent = Body

local sidebarCorner = Instance.new("UICorner")
sidebarCorner.CornerRadius = UDim.new(0, 12)
sidebarCorner.Parent = Sidebar

local SidebarFillTop = Instance.new("Frame")
SidebarFillTop.Size = UDim2.new(1, 0, 0, 14)
SidebarFillTop.BackgroundColor3 = Theme.Sidebar
SidebarFillTop.BorderSizePixel = 0
SidebarFillTop.ZIndex = 3
SidebarFillTop.Parent = Sidebar

local SidebarFillRight = Instance.new("Frame")
SidebarFillRight.Size = UDim2.new(0, 14, 1, 0)
SidebarFillRight.Position = UDim2.new(1, -14, 0, 0)
SidebarFillRight.BackgroundColor3 = Theme.Sidebar
SidebarFillRight.BorderSizePixel = 0
SidebarFillRight.ZIndex = 3
SidebarFillRight.Parent = Sidebar

local SidebarLine = Instance.new("Frame")
SidebarLine.Name = "SidebarLine"
SidebarLine.Size = UDim2.new(0, 1, 1, 0)
SidebarLine.Position = UDim2.new(1, 0, 0, 0)
SidebarLine.BackgroundColor3 = Theme.Border
SidebarLine.BackgroundTransparency = 0.5
SidebarLine.BorderSizePixel = 0
SidebarLine.ZIndex = 4
SidebarLine.Parent = Sidebar

local TabContainer = Instance.new("Frame")
TabContainer.Name = "TabContainer"
TabContainer.Size = UDim2.new(1, -20, 1, -20)
TabContainer.Position = UDim2.new(0, 10, 0, 10)
TabContainer.BackgroundTransparency = 1
TabContainer.ZIndex = 4
TabContainer.Parent = Sidebar

local TabLayout = Instance.new("UIListLayout")
TabLayout.Padding = UDim.new(0, 4)
TabLayout.SortOrder = Enum.SortOrder.LayoutOrder
TabLayout.Parent = TabContainer

-- ═══════════════════════════════════════════
-- ÁREA DE CONTEÚDO
-- ═══════════════════════════════════════════
ContentHolder = Instance.new("Frame")
ContentHolder.Name = "ContentHolder"
ContentHolder.Size = UDim2.new(1, -172, 1, 0)
ContentHolder.Position = UDim2.new(0, 172, 0, 0)
ContentHolder.BackgroundTransparency = 1
ContentHolder.ZIndex = 2
ContentHolder.ClipsDescendants = true
ContentHolder.Parent = Body

-- ═══════════════════════════════════════════
-- SISTEMA DE ABAS
-- ═══════════════════════════════════════════
local Tabs = {}
local TabButtons = {}
local TabPages = {}

local TabData = {
    {Name = "Home", Icon = "🏠", Order = 1},
    {Name = "Combat", Icon = "🎯", Order = 2},
    {Name = "Visuals", Icon = "👁", Order = 3},
    {Name = "Player", Icon = "🏃", Order = 4},
    {Name = "Anti-Lag", Icon = "⚡", Order = 5},
}

local function CreateTabButton(data)
    local tabBtn = Instance.new("TextButton")
    tabBtn.Name = "Tab_" .. data.Name
    tabBtn.Size = UDim2.new(1, 0, 0, 42)
    tabBtn.BackgroundColor3 = Theme.Sidebar
    tabBtn.BackgroundTransparency = 1
    tabBtn.Text = ""
    tabBtn.ZIndex = 5
    tabBtn.LayoutOrder = data.Order
    tabBtn.Parent = TabContainer
    CreateCorner(tabBtn, 8)
    
    local indicator = Instance.new("Frame")
    indicator.Name = "Indicator"
    indicator.Size = UDim2.new(0, 3, 0, 0)
    indicator.Position = UDim2.new(0, 0, 0.5, 0)
    indicator.AnchorPoint = Vector2.new(0, 0.5)
    indicator.BackgroundColor3 = Theme.Primary
    indicator.BorderSizePixel = 0
    indicator.ZIndex = 6
    indicator.Parent = tabBtn
    CreateCorner(indicator, 2)
    
    local icon = Instance.new("TextLabel")
    icon.Name = "Icon"
    icon.Size = UDim2.new(0, 24, 0, 24)
    icon.Position = UDim2.new(0, 12, 0.5, -12)
    icon.BackgroundTransparency = 1
    icon.Text = data.Icon
    icon.TextSize = 16
    icon.Font = Enum.Font.GothamBold
    icon.TextColor3 = Theme.TextMuted
    icon.ZIndex = 6
    icon.Parent = tabBtn
    
    local label = Instance.new("TextLabel")
    label.Name = "Label"
    label.Size = UDim2.new(1, -50, 1, 0)
    label.Position = UDim2.new(0, 44, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = data.Name
    label.TextColor3 = Theme.TextMuted
    label.TextSize = 14
    label.Font = Enum.Font.GothamSemibold
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.ZIndex = 6
    label.Parent = tabBtn
    
    tabBtn.MouseEnter:Connect(function()
        if State.CurrentTab ~= data.Name then
            Tween(tabBtn, {BackgroundTransparency = 0.5, BackgroundColor3 = Theme.SidebarHover}, 0.2)
            Tween(label, {TextColor3 = Theme.TextDim}, 0.2)
        end
    end)
    tabBtn.MouseLeave:Connect(function()
        if State.CurrentTab ~= data.Name then
            Tween(tabBtn, {BackgroundTransparency = 1}, 0.2)
            Tween(label, {TextColor3 = Theme.TextMuted}, 0.2)
        end
    end)
    
    TabButtons[data.Name] = {
        Button = tabBtn,
        Indicator = indicator,
        Icon = icon,
        Label = label,
    }
    
    return tabBtn
end

local function CreateTabPage(name)
    local page = Instance.new("ScrollingFrame")
    page.Name = "Page_" .. name
    page.Size = UDim2.new(1, 0, 1, 0)
    page.BackgroundTransparency = 1
    page.BorderSizePixel = 0
    page.ScrollBarThickness = 4
    page.ScrollBarImageColor3 = Theme.Primary
    page.ScrollBarImageTransparency = 0.3
    page.CanvasSize = UDim2.new(0, 0, 0, 0)
    page.AutomaticCanvasSize = Enum.AutomaticSize.Y
    page.Visible = false
    page.ZIndex = 3
    page.Parent = ContentHolder
    
    local pageLayout = Instance.new("UIListLayout")
    pageLayout.Padding = UDim.new(0, 10)
    pageLayout.SortOrder = Enum.SortOrder.LayoutOrder
    pageLayout.Parent = page
    
    CreatePadding(page, 16, 16, 16, 16)
    
    TabPages[name] = page
    return page
end

local function SwitchTab(tabName)
    if State.CurrentTab == tabName then return end
    
    local prevTab = TabButtons[State.CurrentTab]
    if prevTab then
        Tween(prevTab.Button, {BackgroundTransparency = 1}, 0.25)
        Tween(prevTab.Indicator, {Size = UDim2.new(0, 3, 0, 0)}, 0.25)
        Tween(prevTab.Label, {TextColor3 = Theme.TextMuted}, 0.25)
        Tween(prevTab.Icon, {TextColor3 = Theme.TextMuted}, 0.25)
    end
    
    local prevPage = TabPages[State.CurrentTab]
    if prevPage then
        for _, child in ipairs(prevPage:GetChildren()) do
            if child:IsA("GuiObject") then
                Tween(child, {BackgroundTransparency = 1}, 0.15)
            end
        end
        task.wait(0.15)
        prevPage.Visible = false
    end
    
    State.CurrentTab = tabName
    
    local newTab = TabButtons[tabName]
    if newTab then
        Tween(newTab.Button, {BackgroundTransparency = 0.6, BackgroundColor3 = Theme.SidebarHover}, 0.25)
        Tween(newTab.Indicator, {Size = UDim2.new(0, 3, 0, 24)}, 0.3, Enum.EasingStyle.Back)
        Tween(newTab.Label, {TextColor3 = Theme.Primary}, 0.25)
        Tween(newTab.Icon, {TextColor3 = Theme.Primary}, 0.25)
    end
    
    local newPage = TabPages[tabName]
    if newPage then
        newPage.Visible = true
        newPage.CanvasPosition = Vector2.new(0, 0)
        for i, child in ipairs(newPage:GetChildren()) do
            if child:IsA("GuiObject") then
                child.BackgroundTransparency = 1
                local originalTransparency = child:GetAttribute("OriginalTransparency") or 0
                task.delay(i * 0.03, function()
                    Tween(child, {BackgroundTransparency = originalTransparency}, 0.3)
                end)
            end
        end
    end
end

-- ═══════════════════════════════════════════
-- COMPONENTES REUTILIZÁVEIS
-- ═══════════════════════════════════════════

local function CreateCard(parent, title, height, order)
    local card = Instance.new("Frame")
    card.Name = "Card_" .. (title or "NoTitle")
    card.Size = UDim2.new(1, 0, 0, height or 100)
    card.BackgroundColor3 = Theme.CardBg
    card.BorderSizePixel = 0
    card.ZIndex = 4
    card.LayoutOrder = order or 1
    card.AutomaticSize = Enum.AutomaticSize.Y
    card.Parent = parent
    card:SetAttribute("OriginalTransparency", 0)
    CreateCorner(card, 10)
    CreateStroke(card, Theme.Border, 1, 0.7)
    
    if title and title ~= "" then
        local titleLabel = Instance.new("TextLabel")
        titleLabel.Name = "Title"
        titleLabel.Size = UDim2.new(1, -24, 0, 30)
        titleLabel.Position = UDim2.new(0, 12, 0, 8)
        titleLabel.BackgroundTransparency = 1
        titleLabel.Text = title
        titleLabel.TextColor3 = Theme.Text
        titleLabel.TextSize = 16
        titleLabel.Font = Enum.Font.GothamBold
        titleLabel.TextXAlignment = Enum.TextXAlignment.Left
        titleLabel.ZIndex = 5
        titleLabel.Parent = card
    end
    
    local content = Instance.new("Frame")
    content.Name = "Content"
    content.Size = UDim2.new(1, -24, 1, -46)
    content.Position = UDim2.new(0, 12, 0, 42)
    content.BackgroundTransparency = 1
    content.ZIndex = 5
    content.AutomaticSize = Enum.AutomaticSize.Y
    content.Parent = card
    
    local contentLayout = Instance.new("UIListLayout")
    contentLayout.Padding = UDim.new(0, 8)
    contentLayout.SortOrder = Enum.SortOrder.LayoutOrder
    contentLayout.Parent = content
    
    return card, content
end

local function CreateToggle(parent, label, default, order, callback)
    local toggleFrame = Instance.new("Frame")
    toggleFrame.Name = "Toggle_" .. label
    toggleFrame.Size = UDim2.new(1, 0, 0, 36)
    toggleFrame.BackgroundTransparency = 1
    toggleFrame.ZIndex = 6
    toggleFrame.LayoutOrder = order or 1
    toggleFrame.Parent = parent
    
    local toggleLabel = Instance.new("TextLabel")
    toggleLabel.Name = "Label"
    toggleLabel.Size = UDim2.new(1, -60, 1, 0)
    toggleLabel.BackgroundTransparency = 1
    toggleLabel.Text = label
    toggleLabel.TextColor3 = Theme.TextDim
    toggleLabel.TextSize = 14
    toggleLabel.Font = Enum.Font.GothamMedium
    toggleLabel.TextXAlignment = Enum.TextXAlignment.Left
    toggleLabel.ZIndex = 7
    toggleLabel.Parent = toggleFrame
    
    local toggleBg = Instance.new("TextButton")
    toggleBg.Name = "ToggleBg"
    toggleBg.Size = UDim2.new(0, 48, 0, 24)
    toggleBg.Position = UDim2.new(1, -48, 0.5, -12)
    toggleBg.BackgroundColor3 = default and Theme.ToggleOn or Theme.ToggleOff
    toggleBg.Text = ""
    toggleBg.ZIndex = 7
    toggleBg.Parent = toggleFrame
    CreateCorner(toggleBg, 12)
    
    local toggleCircle = Instance.new("Frame")
    toggleCircle.Name = "Circle"
    toggleCircle.Size = UDim2.new(0, 18, 0, 18)
    toggleCircle.Position = default and UDim2.new(1, -21, 0.5, -9) or UDim2.new(0, 3, 0.5, -9)
    toggleCircle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    toggleCircle.ZIndex = 8
    toggleCircle.Parent = toggleBg
    CreateCorner(toggleCircle, 9)
    
    local enabled = default or false
    
    local function UpdateVisual()
        if enabled then
            Tween(toggleBg, {BackgroundColor3 = Theme.ToggleOn}, 0.25)
            Tween(toggleCircle, {Position = UDim2.new(1, -21, 0.5, -9)}, 0.25, Enum.EasingStyle.Back)
            Tween(toggleLabel, {TextColor3 = Theme.Text}, 0.25)
        else
            Tween(toggleBg, {BackgroundColor3 = Theme.ToggleOff}, 0.25)
            Tween(toggleCircle, {Position = UDim2.new(0, 3, 0.5, -9)}, 0.25, Enum.EasingStyle.Back)
            Tween(toggleLabel, {TextColor3 = Theme.TextDim}, 0.25)
        end
    end
    
    toggleBg.MouseButton1Click:Connect(function()
        enabled = not enabled
        UpdateVisual()
        if callback then
            callback(enabled)
        end
    end)
    
    return {
        Frame = toggleFrame,
        SetEnabled = function(val)
            enabled = val
            UpdateVisual()
        end,
        GetEnabled = function()
            return enabled
        end,
        SetVisible = function(vis)
            toggleFrame.Visible = vis
        end,
        SetInteractable = function(inter)
            toggleBg.Active = inter
            toggleBg.AutoButtonColor = inter
            if not inter then
                Tween(toggleLabel, {TextColor3 = Theme.TextMuted}, 0.2)
                Tween(toggleBg, {BackgroundColor3 = Theme.SliderBg}, 0.2)
            else
                UpdateVisual()
            end
        end
    }
end

local function CreateSlider(parent, label, min, max, default, order, callback)
    local sliderFrame = Instance.new("Frame")
    sliderFrame.Name = "Slider_" .. label
    sliderFrame.Size = UDim2.new(1, 0, 0, 52)
    sliderFrame.BackgroundTransparency = 1
    sliderFrame.ZIndex = 6
    sliderFrame.LayoutOrder = order or 1
    sliderFrame.Parent = parent
    
    local sliderLabel = Instance.new("TextLabel")
    sliderLabel.Name = "Label"
    sliderLabel.Size = UDim2.new(1, -60, 0, 20)
    sliderLabel.BackgroundTransparency = 1
    sliderLabel.Text = label
    sliderLabel.TextColor3 = Theme.TextDim
    sliderLabel.TextSize = 13
    sliderLabel.Font = Enum.Font.GothamMedium
    sliderLabel.TextXAlignment = Enum.TextXAlignment.Left
    sliderLabel.ZIndex = 7
    sliderLabel.Parent = sliderFrame
    
    local valueLabel = Instance.new("TextLabel")
    valueLabel.Name = "Value"
    valueLabel.Size = UDim2.new(0, 50, 0, 20)
    valueLabel.Position = UDim2.new(1, -50, 0, 0)
    valueLabel.BackgroundTransparency = 1
    valueLabel.Text = tostring(math.floor(default))
    valueLabel.TextColor3 = Theme.Primary
    valueLabel.TextSize = 13
    valueLabel.Font = Enum.Font.GothamBold
    valueLabel.TextXAlignment = Enum.TextXAlignment.Right
    valueLabel.ZIndex = 7
    valueLabel.Parent = sliderFrame
    
    local sliderBg = Instance.new("Frame")
    sliderBg.Name = "SliderBg"
    sliderBg.Size = UDim2.new(1, 0, 0, 8)
    sliderBg.Position = UDim2.new(0, 0, 0, 28)
    sliderBg.BackgroundColor3 = Theme.SliderBg
    sliderBg.ZIndex = 7
    sliderBg.Parent = sliderFrame
    CreateCorner(sliderBg, 4)
    
    local sliderFill = Instance.new("Frame")
    sliderFill.Name = "Fill"
    local initPercent = (default - min) / (max - min)
    sliderFill.Size = UDim2.new(initPercent, 0, 1, 0)
    sliderFill.BackgroundColor3 = Theme.SliderFill
    sliderFill.ZIndex = 8
    sliderFill.Parent = sliderBg
    CreateCorner(sliderFill, 4)
    
    local fillGrad = Instance.new("UIGradient")
    fillGrad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Theme.PrimaryDark),
        ColorSequenceKeypoint.new(1, Theme.Primary)
    })
    fillGrad.Parent = sliderFill
    
    local knob = Instance.new("Frame")
    knob.Name = "Knob"
    knob.Size = UDim2.new(0, 16, 0, 16)
    knob.Position = UDim2.new(initPercent, -8, 0.5, -8)
    knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    knob.ZIndex = 9
    knob.Parent = sliderBg
    CreateCorner(knob, 8)
    CreateShadow(knob, 0.5, 8)
    
    local sliderBtn = Instance.new("TextButton")
    sliderBtn.Name = "SliderBtn"
    sliderBtn.Size = UDim2.new(1, 0, 0, 24)
    sliderBtn.Position = UDim2.new(0, 0, 0, 22)
    sliderBtn.BackgroundTransparency = 1
    sliderBtn.Text = ""
    sliderBtn.ZIndex = 10
    sliderBtn.Parent = sliderFrame
    
    local draggingSlider = false
    local currentValue = default
    
    local function UpdateSlider(input)
        local absPos = sliderBg.AbsolutePosition.X
        local absSize = sliderBg.AbsoluteSize.X
        local mouseX = input.Position.X
        local percent = math.clamp((mouseX - absPos) / absSize, 0, 1)
        local value = math.floor(min + (max - min) * percent)
        
        currentValue = value
        valueLabel.Text = tostring(value)
        Tween(sliderFill, {Size = UDim2.new(percent, 0, 1, 0)}, 0.1)
        Tween(knob, {Position = UDim2.new(percent, -8, 0.5, -8)}, 0.1)
        
        if callback then
            callback(value)
        end
    end
    
    sliderBtn.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            draggingSlider = true
            UpdateSlider(input)
        end
    end)
    
    sliderBtn.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            draggingSlider = false
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if draggingSlider and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            UpdateSlider(input)
        end
    end)
    
    return {
        Frame = sliderFrame,
        GetValue = function() return currentValue end,
        SetValue = function(val)
            currentValue = val
            local percent = (val - min) / (max - min)
            valueLabel.Text = tostring(math.floor(val))
            Tween(sliderFill, {Size = UDim2.new(percent, 0, 1, 0)}, 0.2)
            Tween(knob, {Position = UDim2.new(percent, -8, 0.5, -8)}, 0.2)
        end,
        SetVisible = function(vis)
            sliderFrame.Visible = vis
        end,
        SetInteractable = function(inter)
            sliderBtn.Active = inter
            if not inter then
                Tween(sliderLabel, {TextColor3 = Theme.TextMuted}, 0.2)
                Tween(sliderFill, {BackgroundColor3 = Theme.SliderBg}, 0.2)
            else
                Tween(sliderLabel, {TextColor3 = Theme.TextDim}, 0.2)
                Tween(sliderFill, {BackgroundColor3 = Theme.SliderFill}, 0.2)
            end
        end
    }
end

local function CreateSeparator(parent, order)
    local sep = Instance.new("Frame")
    sep.Name = "Separator"
    sep.Size = UDim2.new(1, 0, 0, 1)
    sep.BackgroundColor3 = Theme.Border
    sep.BackgroundTransparency = 0.6
    sep.BorderSizePixel = 0
    sep.ZIndex = 6
    sep.LayoutOrder = order or 1
    sep.Parent = parent
    return sep
end

-- ═══════════════════════════════════════════
-- CRIAR ABAS
-- ═══════════════════════════════════════════
for _, data in ipairs(TabData) do
    CreateTabButton(data)
    CreateTabPage(data.Name)
end

-- ═══════════════════════════════════════════
-- ABA HOME
-- ═══════════════════════════════════════════
local homePage = TabPages["Home"]

local homeBanner = Instance.new("Frame")
homeBanner.Name = "Banner"
homeBanner.Size = UDim2.new(1, 0, 0, 120)
homeBanner.BackgroundColor3 = Theme.CardBg
homeBanner.ZIndex = 4
homeBanner.LayoutOrder = 1
homeBanner.Parent = homePage
homeBanner:SetAttribute("OriginalTransparency", 0)
CreateCorner(homeBanner, 12)
CreateStroke(homeBanner, Theme.Primary, 1, 0.7)

local bannerGrad = Instance.new("UIGradient")
bannerGrad.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Theme.Primary),
    ColorSequenceKeypoint.new(1, Theme.BackgroundDark)
})
bannerGrad.Transparency = NumberSequence.new({
    NumberSequenceKeypoint.new(0, 0.85),
    NumberSequenceKeypoint.new(1, 0.95)
})
bannerGrad.Rotation = 25
bannerGrad.Parent = homeBanner

local homeTitle = Instance.new("TextLabel")
homeTitle.Name = "HomeTitle"
homeTitle.Size = UDim2.new(1, -30, 0, 45)
homeTitle.Position = UDim2.new(0, 15, 0, 15)
homeTitle.BackgroundTransparency = 1
homeTitle.Text = "⛓ XITERPRISON ⛓"
homeTitle.TextColor3 = Theme.Primary
homeTitle.TextSize = 28
homeTitle.Font = Enum.Font.GothamBlack
homeTitle.TextXAlignment = Enum.TextXAlignment.Center
homeTitle.ZIndex = 5
homeTitle.Parent = homeBanner

local homeSubtitle = Instance.new("TextLabel")
homeSubtitle.Name = "Subtitle"
homeSubtitle.Size = UDim2.new(1, -30, 0, 20)
homeSubtitle.Position = UDim2.new(0, 15, 0, 58)
homeSubtitle.BackgroundTransparency = 1
homeSubtitle.Text = "Prison Life Script Hub"
homeSubtitle.TextColor3 = Theme.TextDim
homeSubtitle.TextSize = 14
homeSubtitle.Font = Enum.Font.GothamMedium
homeSubtitle.TextXAlignment = Enum.TextXAlignment.Center
homeSubtitle.ZIndex = 5
homeSubtitle.Parent = homeBanner

local decBar = Instance.new("Frame")
decBar.Name = "DecBar"
decBar.Size = UDim2.new(0.4, 0, 0, 3)
decBar.Position = UDim2.new(0.3, 0, 0, 85)
decBar.BackgroundColor3 = Theme.Primary
decBar.BackgroundTransparency = 0.3
decBar.ZIndex = 5
decBar.Parent = homeBanner
CreateCorner(decBar, 2)

local descCard, descContent = CreateCard(homePage, "📋 Bem-vindo", 0, 2)

local descText = Instance.new("TextLabel")
descText.Name = "Description"
descText.Size = UDim2.new(1, 0, 0, 0)
descText.AutomaticSize = Enum.AutomaticSize.Y
descText.BackgroundTransparency = 1
descText.Text = "Olá, se você está usando este script é graças a mim, eu peço que você se divirta utilizando deste script, aproveite a interface e seus recursos visuais."
descText.TextColor3 = Theme.TextDim
descText.TextSize = 14
descText.Font = Enum.Font.GothamMedium
descText.TextWrapped = true
descText.TextXAlignment = Enum.TextXAlignment.Left
descText.ZIndex = 7
descText.LayoutOrder = 1
descText.Parent = descContent

local credCard, credContent = CreateCard(homePage, "👥 Créditos", 0, 3)

local function CreateCreditEntry(name, role, order)
    local entry = Instance.new("Frame")
    entry.Name = "Credit_" .. name
    entry.Size = UDim2.new(1, 0, 0, 40)
    entry.BackgroundColor3 = Theme.Background
    entry.BackgroundTransparency = 0.5
    entry.ZIndex = 7
    entry.LayoutOrder = order
    entry.Parent = credContent
    CreateCorner(entry, 8)
    
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(0.5, -10, 1, 0)
    nameLabel.Position = UDim2.new(0, 12, 0, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = "⭐ " .. name
    nameLabel.TextColor3 = Theme.Primary
    nameLabel.TextSize = 14
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextXAlignment = Enum.TextXAlignment.Left
    nameLabel.ZIndex = 8
    nameLabel.Parent = entry
    
    local roleLabel = Instance.new("TextLabel")
    roleLabel.Size = UDim2.new(0.5, -10, 1, 0)
    roleLabel.Position = UDim2.new(0.5, 0, 0, 0)
    roleLabel.BackgroundTransparency = 1
    roleLabel.Text = role
    roleLabel.TextColor3 = Theme.TextMuted
    roleLabel.TextSize = 12
    roleLabel.Font = Enum.Font.GothamMedium
    roleLabel.TextXAlignment = Enum.TextXAlignment.Right
    roleLabel.ZIndex = 8
    roleLabel.Parent = entry
    
    return entry
end

CreateCreditEntry("Thurzinnn", "Desenvolvedor Principal", 1)
CreateCreditEntry("Thalles", "Co-Desenvolvedor", 2)

local statusCard, statusContent = CreateCard(homePage, "📊 Status", 0, 4)

local statusText = Instance.new("TextLabel")
statusText.Name = "Status"
statusText.Size = UDim2.new(1, 0, 0, 30)
statusText.BackgroundTransparency = 1
statusText.Text = "✅ Script carregado com sucesso!"
statusText.TextColor3 = Theme.Success
statusText.TextSize = 14
statusText.Font = Enum.Font.GothamBold
statusText.TextXAlignment = Enum.TextXAlignment.Left
statusText.ZIndex = 7
statusText.LayoutOrder = 1
statusText.Parent = statusContent

local playerInfo = Instance.new("TextLabel")
playerInfo.Name = "PlayerInfo"
playerInfo.Size = UDim2.new(1, 0, 0, 20)
playerInfo.BackgroundTransparency = 1
playerInfo.Text = "Jogador: " .. LocalPlayer.Name
playerInfo.TextColor3 = Theme.TextDim
playerInfo.TextSize = 12
playerInfo.Font = Enum.Font.GothamMedium
playerInfo.TextXAlignment = Enum.TextXAlignment.Left
playerInfo.ZIndex = 7
playerInfo.LayoutOrder = 2
playerInfo.Parent = statusContent

-- ═══════════════════════════════════════════
-- ABA COMBAT (COM TEAM CHECK)
-- ═══════════════════════════════════════════
local combatPage = TabPages["Combat"]

local aimbotCard, aimbotContent = CreateCard(combatPage, "🎯 Aimbot", 0, 1)

local combatSubToggles = {}
local combatSubSliders = {}

local aimbotToggle = CreateToggle(aimbotContent, "Ativar Aimbot", false, 1, function(enabled)
    State.AimbotEnabled = enabled
    for _, sub in pairs(combatSubToggles) do
        if sub.SetInteractable then
            sub.SetInteractable(enabled)
        end
    end
    for _, sub in pairs(combatSubSliders) do
        if sub.SetInteractable then
            sub.SetInteractable(enabled)
        end
    end
end)

CreateSeparator(aimbotContent, 2)

local subTitle = Instance.new("TextLabel")
subTitle.Name = "SubTitle"
subTitle.Size = UDim2.new(1, 0, 0, 24)
subTitle.BackgroundTransparency = 1
subTitle.Text = "⚙ Configurações do Aimbot"
subTitle.TextColor3 = Theme.TextMuted
subTitle.TextSize = 12
subTitle.Font = Enum.Font.GothamBold
subTitle.TextXAlignment = Enum.TextXAlignment.Left
subTitle.ZIndex = 7
subTitle.LayoutOrder = 3
subTitle.Parent = aimbotContent

-- Team Check Combat
combatSubToggles.TeamCheck = CreateToggle(aimbotContent, "🛡 Team Check (Ignorar Aliados)", false, 4, function(enabled)
    State.TeamCheckCombat = enabled
end)
combatSubToggles.TeamCheck.SetInteractable(false)

-- Info box do Team Check
local teamCheckInfo = Instance.new("Frame")
teamCheckInfo.Name = "TeamCheckInfo"
teamCheckInfo.Size = UDim2.new(1, 0, 0, 36)
teamCheckInfo.BackgroundColor3 = Theme.Background
teamCheckInfo.BackgroundTransparency = 0.5
teamCheckInfo.ZIndex = 7
teamCheckInfo.LayoutOrder = 5
teamCheckInfo.Parent = aimbotContent
CreateCorner(teamCheckInfo, 6)

local teamCheckInfoIcon = Instance.new("TextLabel")
teamCheckInfoIcon.Size = UDim2.new(0, 24, 1, 0)
teamCheckInfoIcon.Position = UDim2.new(0, 8, 0, 0)
teamCheckInfoIcon.BackgroundTransparency = 1
teamCheckInfoIcon.Text = "ℹ"
teamCheckInfoIcon.TextColor3 = Theme.Primary
teamCheckInfoIcon.TextSize = 14
teamCheckInfoIcon.Font = Enum.Font.GothamBold
teamCheckInfoIcon.ZIndex = 8
teamCheckInfoIcon.Parent = teamCheckInfo

local teamCheckInfoText = Instance.new("TextLabel")
teamCheckInfoText.Size = UDim2.new(1, -40, 1, 0)
teamCheckInfoText.Position = UDim2.new(0, 36, 0, 0)
teamCheckInfoText.BackgroundTransparency = 1
teamCheckInfoText.Text = "O aimbot não mirará em jogadores do seu time"
teamCheckInfoText.TextColor3 = Theme.TextMuted
teamCheckInfoText.TextSize = 11
teamCheckInfoText.Font = Enum.Font.GothamMedium
teamCheckInfoText.TextXAlignment = Enum.TextXAlignment.Left
teamCheckInfoText.TextWrapped = true
teamCheckInfoText.ZIndex = 8
teamCheckInfoText.Parent = teamCheckInfo

CreateSeparator(aimbotContent, 6)

-- Triggerbot
combatSubToggles.Triggerbot = CreateToggle(aimbotContent, "Triggerbot", false, 7, function(enabled)
    State.TriggerbotEnabled = enabled
end)
combatSubToggles.Triggerbot.SetInteractable(false)

-- FOV
combatSubToggles.FOV = CreateToggle(aimbotContent, "FOV Circle", false, 8, function(enabled)
    State.FOVEnabled = enabled
end)
combatSubToggles.FOV.SetInteractable(false)

-- FOV Size Slider
combatSubSliders.FOVSize = CreateSlider(aimbotContent, "FOV Size", 30, 500, 150, 9, function(value)
    State.FOVSize = value
end)
combatSubSliders.FOVSize.SetInteractable(false)

CreateSeparator(aimbotContent, 10)

-- Aim Strength
combatSubSliders.AimStrength = CreateSlider(aimbotContent, "Aim Strength", 1, 100, 50, 11, function(value)
    State.AimStrength = value / 100
end)
combatSubSliders.AimStrength.SetInteractable(false)

-- ═══════════════════════════════════════════
-- ABA VISUALS (COM TEAM CHECK)
-- ═══════════════════════════════════════════
local visualsPage = TabPages["Visuals"]

local espCard, espContent = CreateCard(visualsPage, "👁 ESP", 0, 1)

local visualSubToggles = {}
local visualSubSliders = {}

local espToggle = CreateToggle(espContent, "Ativar ESP", false, 1, function(enabled)
    State.ESPEnabled = enabled
    for _, sub in pairs(visualSubToggles) do
        if sub.SetInteractable then
            sub.SetInteractable(enabled)
        end
    end
    for _, sub in pairs(visualSubSliders) do
        if sub.SetInteractable then
            sub.SetInteractable(enabled)
        end
    end
    if not enabled then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                local espFolder = player.Character:FindFirstChild("ESP_Folder")
                if espFolder then espFolder:Destroy() end
            end
        end
    end
end)

CreateSeparator(espContent, 2)

local espSubTitle = Instance.new("TextLabel")
espSubTitle.Name = "SubTitle"
espSubTitle.Size = UDim2.new(1, 0, 0, 24)
espSubTitle.BackgroundTransparency = 1
espSubTitle.Text = "⚙ Configurações do ESP"
espSubTitle.TextColor3 = Theme.TextMuted
espSubTitle.TextSize = 12
espSubTitle.Font = Enum.Font.GothamBold
espSubTitle.TextXAlignment = Enum.TextXAlignment.Left
espSubTitle.ZIndex = 7
espSubTitle.LayoutOrder = 3
espSubTitle.Parent = espContent

-- Team Check Visuals
visualSubToggles.TeamCheck = CreateToggle(espContent, "🏷 Team Check (Nome e Cor do Time)", false, 4, function(enabled)
    State.TeamCheckVisuals = enabled
end)
visualSubToggles.TeamCheck.SetInteractable(false)

-- Info box do Team Check Visuals
local teamCheckVisInfo = Instance.new("Frame")
teamCheckVisInfo.Name = "TeamCheckVisInfo"
teamCheckVisInfo.Size = UDim2.new(1, 0, 0, 48)
teamCheckVisInfo.BackgroundColor3 = Theme.Background
teamCheckVisInfo.BackgroundTransparency = 0.5
teamCheckVisInfo.ZIndex = 7
teamCheckVisInfo.LayoutOrder = 5
teamCheckVisInfo.Parent = espContent
CreateCorner(teamCheckVisInfo, 6)

local teamVisInfoIcon = Instance.new("TextLabel")
teamVisInfoIcon.Size = UDim2.new(0, 24, 1, 0)
teamVisInfoIcon.Position = UDim2.new(0, 8, 0, 0)
teamVisInfoIcon.BackgroundTransparency = 1
teamVisInfoIcon.Text = "🎨"
teamVisInfoIcon.TextColor3 = Theme.Primary
teamVisInfoIcon.TextSize = 14
teamVisInfoIcon.Font = Enum.Font.GothamBold
teamVisInfoIcon.ZIndex = 8
teamVisInfoIcon.Parent = teamCheckVisInfo

local teamVisInfoText = Instance.new("TextLabel")
teamVisInfoText.Size = UDim2.new(1, -40, 1, -8)
teamVisInfoText.Position = UDim2.new(0, 36, 0, 4)
teamVisInfoText.BackgroundTransparency = 1
teamVisInfoText.Text = "Exibe o nome do time acima da cabeça e colore o Box/Tracer com a cor do time de cada jogador"
teamVisInfoText.TextColor3 = Theme.TextMuted
teamVisInfoText.TextSize = 11
teamVisInfoText.Font = Enum.Font.GothamMedium
teamVisInfoText.TextXAlignment = Enum.TextXAlignment.Left
teamVisInfoText.TextWrapped = true
teamVisInfoText.ZIndex = 8
teamVisInfoText.Parent = teamCheckVisInfo

CreateSeparator(espContent, 6)

visualSubToggles.Boxfield = CreateToggle(espContent, "Boxfield (Caixa)", false, 7, function(enabled)
    State.BoxfieldEnabled = enabled
end)
visualSubToggles.Boxfield.SetInteractable(false)

visualSubToggles.LifeStatus = CreateToggle(espContent, "Life Status (Vida)", false, 8, function(enabled)
    State.LifeStatusEnabled = enabled
end)
visualSubToggles.LifeStatus.SetInteractable(false)

visualSubToggles.Tracer = CreateToggle(espContent, "Tracer (Linhas)", false, 9, function(enabled)
    State.TracerEnabled = enabled
end)
visualSubToggles.Tracer.SetInteractable(false)

CreateSeparator(espContent, 10)

visualSubSliders.ESPRange = CreateSlider(espContent, "ESP Range", 50, 2000, 500, 11, function(value)
    State.ESPRange = value
end)
visualSubSliders.ESPRange.SetInteractable(false)

-- ═══════════════════════════════════════════
-- ABA PLAYER
-- ═══════════════════════════════════════════
local playerPage = TabPages["Player"]

local utilCard, utilContent = CreateCard(playerPage, "🛡 Utilidades", 0, 1)

CreateToggle(utilContent, "Infinite Ammo", false, 1, function(enabled)
    State.InfiniteAmmoEnabled = enabled
end)

CreateToggle(utilContent, "God Mode", false, 2, function(enabled)
    State.GodModeEnabled = enabled
end)

CreateToggle(utilContent, "No Clip", false, 3, function(enabled)
    State.NoClipEnabled = enabled
end)

local speedCard, speedContent = CreateCard(playerPage, "🏃 Speedhack", 0, 2)

CreateToggle(speedContent, "Ativar Speedhack", false, 1, function(enabled)
    State.SpeedhackEnabled = enabled
    if enabled then
        local char = LocalPlayer.Character
        if char and char:FindFirstChild("Humanoid") then
            char.Humanoid.WalkSpeed = State.SpeedValue
        end
    else
        local char = LocalPlayer.Character
        if char and char:FindFirstChild("Humanoid") then
            char.Humanoid.WalkSpeed = 16
        end
    end
end)

CreateSlider(speedContent, "Velocidade", 16, 200, 16, 2, function(value)
    State.SpeedValue = value
    if State.SpeedhackEnabled then
        local char = LocalPlayer.Character
        if char and char:FindFirstChild("Humanoid") then
            char.Humanoid.WalkSpeed = value
        end
    end
end)

local flyCard, flyContent = CreateCard(playerPage, "✈ Fly Hack", 0, 3)

CreateToggle(flyContent, "Ativar Fly", false, 1, function(enabled)
    State.FlyHackEnabled = enabled
    if enabled then
        local char = LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            State.Flying = true
            local hrp = char.HumanoidRootPart
            
            if State.FlyBody then
                State.FlyBody:Destroy()
            end
            
            local bv = Instance.new("BodyVelocity")
            bv.Name = "FlyVelocity"
            bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bv.Velocity = Vector3.new(0, 0, 0)
            bv.Parent = hrp
            
            local bg = Instance.new("BodyGyro")
            bg.Name = "FlyGyro"
            bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
            bg.D = 100
            bg.P = 10000
            bg.Parent = hrp
            
            State.FlyBody = bv
            State.FlyGyro = bg
            
            if char:FindFirstChild("Humanoid") then
                char.Humanoid.PlatformStand = true
            end
        end
    else
        State.Flying = false
        local char = LocalPlayer.Character
        if char then
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hrp then
                local bv = hrp:FindFirstChild("FlyVelocity")
                if bv then bv:Destroy() end
                local bg = hrp:FindFirstChild("FlyGyro")
                if bg then bg:Destroy() end
            end
            if char:FindFirstChild("Humanoid") then
                char.Humanoid.PlatformStand = false
            end
        end
        State.FlyBody = nil
        State.FlyGyro = nil
    end
end)

CreateSlider(flyContent, "Velocidade de Voo", 10, 300, 50, 2, function(value)
    State.FlySpeedValue = value
end)

-- ═══════════════════════════════════════════
-- ABA ANTI-LAG
-- ═══════════════════════════════════════════
local antilagPage = TabPages["Anti-Lag"]

local lagCard, lagContent = CreateCard(antilagPage, "⚡ Anti-Lag", 0, 1)

local lagDesc = Instance.new("TextLabel")
lagDesc.Name = "Description"
lagDesc.Size = UDim2.new(1, 0, 0, 0)
lagDesc.AutomaticSize = Enum.AutomaticSize.Y
lagDesc.BackgroundTransparency = 1
lagDesc.Text = "O sistema Anti-Lag reduz elementos visuais pesados do jogo para melhorar significativamente o desempenho. Isso inclui remoção de partículas, texturas desnecessárias e efeitos de iluminação complexos."
lagDesc.TextColor3 = Theme.TextDim
lagDesc.TextSize = 13
lagDesc.Font = Enum.Font.GothamMedium
lagDesc.TextWrapped = true
lagDesc.TextXAlignment = Enum.TextXAlignment.Left
lagDesc.ZIndex = 7
lagDesc.LayoutOrder = 1
lagDesc.Parent = lagContent

CreateSeparator(lagContent, 2)

CreateToggle(lagContent, "Ativar Anti-Lag", false, 3, function(enabled)
    State.AntiLagEnabled = enabled
    if enabled then
        local lighting = game:GetService("Lighting")
        lighting.GlobalShadows = false
        lighting.FogEnd = 9e9
        
        for _, v in ipairs(Workspace:GetDescendants()) do
            if v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Smoke") or v:IsA("Fire") or v:IsA("Sparkles") then
                v.Enabled = false
            elseif v:IsA("Decal") or v:IsA("Texture") then
                v.Transparency = 1
            elseif v:IsA("MeshPart") or v:IsA("Part") then
                v.Material = Enum.Material.SmoothPlastic
                v.Reflectance = 0
            end
        end
        
        for _, v in ipairs(lighting:GetDescendants()) do
            if v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("BloomEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("ColorCorrectionEffect") then
                v.Enabled = false
            end
        end
        
        pcall(function()
            settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
        end)
    else
        local lighting = game:GetService("Lighting")
        lighting.GlobalShadows = true
        
        for _, v in ipairs(Workspace:GetDescendants()) do
            if v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Smoke") or v:IsA("Fire") or v:IsA("Sparkles") then
                v.Enabled = true
            elseif v:IsA("Decal") or v:IsA("Texture") then
                v.Transparency = 0
            end
        end
        
        for _, v in ipairs(lighting:GetDescendants()) do
            if v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("BloomEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("ColorCorrectionEffect") then
                v.Enabled = true
            end
        end
        
        pcall(function()
            settings().Rendering.QualityLevel = Enum.QualityLevel.Automatic
        end)
    end
end)

local lagStatus = Instance.new("Frame")
lagStatus.Name = "StatusBox"
lagStatus.Size = UDim2.new(1, 0, 0, 60)
lagStatus.BackgroundColor3 = Theme.Background
lagStatus.BackgroundTransparency = 0.5
lagStatus.ZIndex = 7
lagStatus.LayoutOrder = 4
lagStatus.Parent = lagContent
CreateCorner(lagStatus, 8)

local lagStatusIcon = Instance.new("TextLabel")
lagStatusIcon.Size = UDim2.new(0, 30, 1, 0)
lagStatusIcon.Position = UDim2.new(0, 10, 0, 0)
lagStatusIcon.BackgroundTransparency = 1
lagStatusIcon.Text = "💡"
lagStatusIcon.TextSize = 18
lagStatusIcon.ZIndex = 8
lagStatusIcon.Parent = lagStatus

local lagStatusText = Instance.new("TextLabel")
lagStatusText.Size = UDim2.new(1, -50, 1, -10)
lagStatusText.Position = UDim2.new(0, 45, 0, 5)
lagStatusText.BackgroundTransparency = 1
lagStatusText.Text = "Ative o Anti-Lag para melhorar o FPS. O sistema irá desativar sombras, partículas, efeitos de pós-processamento e reduzir a qualidade dos materiais."
lagStatusText.TextColor3 = Theme.TextMuted
lagStatusText.TextSize = 11
lagStatusText.Font = Enum.Font.GothamMedium
lagStatusText.TextWrapped = true
lagStatusText.TextXAlignment = Enum.TextXAlignment.Left
lagStatusText.ZIndex = 8
lagStatusText.Parent = lagStatus

-- ═══════════════════════════════════════════
-- FOV CIRCLE (Drawing API)
-- ═══════════════════════════════════════════
local FOVCircle = nil
pcall(function()
    FOVCircle = Drawing.new("Circle")
    FOVCircle.Color = Theme.Primary
    FOVCircle.Thickness = 2
    FOVCircle.NumSides = 64
    FOVCircle.Radius = State.FOVSize
    FOVCircle.Filled = false
    FOVCircle.Visible = false
    FOVCircle.Transparency = 0.7
end)

-- ═══════════════════════════════════════════
-- ESP DRAWINGS
-- ═══════════════════════════════════════════
local ESPDrawings = {}

local function ClearESP(player)
    if ESPDrawings[player] then
        for _, drawing in pairs(ESPDrawings[player]) do
            pcall(function() drawing:Remove() end)
        end
        ESPDrawings[player] = nil
    end
end

local function ClearAllESP()
    for player, _ in pairs(ESPDrawings) do
        ClearESP(player)
    end
end

-- ═══════════════════════════════════════════
-- LOOP PRINCIPAL DE RENDERIZAÇÃO
-- ═══════════════════════════════════════════
RunService.RenderStepped:Connect(function()
    -- ═══ FOV Circle ═══
    if FOVCircle then
        FOVCircle.Visible = State.AimbotEnabled and State.FOVEnabled
        if FOVCircle.Visible then
            FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
            FOVCircle.Radius = State.FOVSize
        end
    end
    
    -- ═══ Fly ═══
    if State.Flying and State.FlyBody and State.FlyGyro then
        local char = LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            local hrp = char.HumanoidRootPart
            local moveDirection = Vector3.new(0, 0, 0)
            
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                moveDirection = moveDirection + Camera.CFrame.LookVector
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                moveDirection = moveDirection - Camera.CFrame.LookVector
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                moveDirection = moveDirection - Camera.CFrame.RightVector
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                moveDirection = moveDirection + Camera.CFrame.RightVector
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                moveDirection = moveDirection + Vector3.new(0, 1, 0)
            end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
                moveDirection = moveDirection - Vector3.new(0, 1, 0)
            end
            
            if moveDirection.Magnitude > 0 then
                moveDirection = moveDirection.Unit
            end
            
            State.FlyBody.Velocity = moveDirection * State.FlySpeedValue
            State.FlyGyro.CFrame = Camera.CFrame
        end
    end
    
    -- ═══ No Clip ═══
    if State.NoClipEnabled then
        local char = LocalPlayer.Character
        if char then
            for _, part in ipairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
    end
    
    -- ═══ God Mode ═══
    if State.GodModeEnabled then
        local char = LocalPlayer.Character
        if char and char:FindFirstChild("Humanoid") then
            char.Humanoid.Health = char.Humanoid.MaxHealth
        end
    end
    
    -- ═══ Speed Hack ═══
    if State.SpeedhackEnabled then
        local char = LocalPlayer.Character
        if char and char:FindFirstChild("Humanoid") then
            char.Humanoid.WalkSpeed = State.SpeedValue
        end
    end
    
    -- ═══ Infinite Ammo ═══
    if State.InfiniteAmmoEnabled then
        pcall(function()
            local char = LocalPlayer.Character
            if char then
                for _, tool in ipairs(char:GetChildren()) do
                    if tool:IsA("Tool") then
                        local ammo = tool:FindFirstChild("Ammo") or tool:FindFirstChild("ammo")
                        if ammo and (ammo:IsA("NumberValue") or ammo:IsA("IntValue")) then
                            ammo.Value = 999
                        end
                    end
                end
            end
        end)
    end
    
    -- ═══════════════════════════════════════════
    -- AIMBOT COM TEAM CHECK
    -- ═══════════════════════════════════════════
    if State.AimbotEnabled then
        pcall(function()
            if UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
                local closestPlayer = nil
                local closestDistance = State.FOVEnabled and State.FOVSize or math.huge
                local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
                
                for _, player in ipairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
                        -- ★ TEAM CHECK: Pula jogadores do mesmo time ★
                        if not ShouldTargetPlayer(player) then
                            continue
                        end
                        
                        local head = player.Character.Head
                        local humanoid = player.Character:FindFirstChild("Humanoid")
                        
                        -- Não mirar em jogadores mortos
                        if humanoid and humanoid.Health <= 0 then
                            continue
                        end
                        
                        local screenPos, onScreen = Camera:WorldToScreenPoint(head.Position)
                        
                        if onScreen then
                            local dist = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude
                            if dist < closestDistance then
                                closestDistance = dist
                                closestPlayer = player
                            end
                        end
                    end
                end
                
                if closestPlayer and closestPlayer.Character and closestPlayer.Character:FindFirstChild("Head") then
                    local targetPos = closestPlayer.Character.Head.Position
                    local currentCFrame = Camera.CFrame
                    local targetCFrame = CFrame.lookAt(currentCFrame.Position, targetPos)
                    Camera.CFrame = currentCFrame:Lerp(targetCFrame, State.AimStrength)
                end
            end
            
            -- Triggerbot com Team Check
            if State.TriggerbotEnabled then
                local target = Mouse.Target
                if target then
                    local targetPlayer = Players:GetPlayerFromCharacter(target.Parent)
                    if targetPlayer and targetPlayer ~= LocalPlayer then
                        -- ★ TEAM CHECK no Triggerbot também ★
                        if ShouldTargetPlayer(targetPlayer) then
                            pcall(function()
                                mouse1click()
                            end)
                        end
                    end
                end
            end
        end)
    end
    
    -- ═══════════════════════════════════════════
    -- ESP COM TEAM CHECK (COR E NOME DO TIME)
    -- ═══════════════════════════════════════════
    if State.ESPEnabled then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then
                pcall(function()
                    local char = player.Character
                    if char and char:FindFirstChild("HumanoidRootPart") and char:FindFirstChild("Humanoid") and char:FindFirstChild("Head") then
                        local hrp = char.HumanoidRootPart
                        local head = char.Head
                        local humanoid = char.Humanoid
                        local myHRP = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                        
                        if not myHRP then return end
                        
                        local distance = (hrp.Position - myHRP.Position).Magnitude
                        
                        if distance > State.ESPRange then
                            ClearESP(player)
                            return
                        end
                        
                        -- Não mostrar ESP em jogadores mortos
                        if humanoid.Health <= 0 then
                            ClearESP(player)
                            return
                        end
                        
                        local screenPos, onScreen = Camera:WorldToScreenPoint(hrp.Position)
                        local headPos = Camera:WorldToScreenPoint(head.Position + Vector3.new(0, 0.5, 0))
                        local footPos = Camera:WorldToScreenPoint(hrp.Position - Vector3.new(0, 3, 0))
                        
                        if not onScreen then
                            ClearESP(player)
                            return
                        end
                        
                        if not ESPDrawings[player] then
                            ESPDrawings[player] = {}
                        end
                        
                        -- ★ OBTER COR COM BASE NO TIME ★
                        local espColor = GetESPColor(player)
                        local teamName = GetTeamName(player)
                        local isSameTeam = IsSameTeam(LocalPlayer, player)
                        
                        -- ═══ BOXFIELD COM COR DO TIME ═══
                        if State.BoxfieldEnabled then
                            if not ESPDrawings[player].Box then
                                ESPDrawings[player].Box = Drawing.new("Quad")
                                ESPDrawings[player].Box.Thickness = 1.5
                                ESPDrawings[player].Box.Filled = false
                                ESPDrawings[player].Box.Transparency = 0.8
                            end
                            
                            -- ★ Cor muda conforme o time quando Team Check está ativo ★
                            ESPDrawings[player].Box.Color = espColor
                            
                            local height = math.abs(headPos.Y - footPos.Y)
                            local width = height / 2
                            
                            ESPDrawings[player].Box.PointA = Vector2.new(screenPos.X - width / 2, headPos.Y)
                            ESPDrawings[player].Box.PointB = Vector2.new(screenPos.X + width / 2, headPos.Y)
                            ESPDrawings[player].Box.PointC = Vector2.new(screenPos.X + width / 2, footPos.Y)
                            ESPDrawings[player].Box.PointD = Vector2.new(screenPos.X - width / 2, footPos.Y)
                            ESPDrawings[player].Box.Visible = true
                        else
                            if ESPDrawings[player].Box then
                                ESPDrawings[player].Box.Visible = false
                            end
                        end
                        
                        -- ═══ NOME DO JOGADOR + VIDA ═══
                        if State.LifeStatusEnabled then
                            if not ESPDrawings[player].Name then
                                ESPDrawings[player].Name = Drawing.new("Text")
                                ESPDrawings[player].Name.Size = 14
                                ESPDrawings[player].Name.Center = true
                                ESPDrawings[player].Name.Outline = true
                                ESPDrawings[player].Name.OutlineColor = Color3.fromRGB(0, 0, 0)
                                ESPDrawings[player].Name.Font = 2
                            end
                            
                            local health = math.floor(humanoid.Health)
                            local maxHealth = math.floor(humanoid.MaxHealth)
                            ESPDrawings[player].Name.Text = player.Name .. " [" .. health .. "/" .. maxHealth .. "]"
                            ESPDrawings[player].Name.Position = Vector2.new(screenPos.X, headPos.Y - 18)
                            ESPDrawings[player].Name.Visible = true
                            
                            -- Cor baseada na vida
                            local healthPercent = humanoid.Health / humanoid.MaxHealth
                            if healthPercent > 0.6 then
                                ESPDrawings[player].Name.Color = Color3.fromRGB(46, 204, 113)
                            elseif healthPercent > 0.3 then
                                ESPDrawings[player].Name.Color = Color3.fromRGB(241, 196, 15)
                            else
                                ESPDrawings[player].Name.Color = Color3.fromRGB(231, 76, 60)
                            end
                        else
                            if ESPDrawings[player].Name then
                                ESPDrawings[player].Name.Visible = false
                            end
                        end
                        
                        -- ═══ ★ NOME DO TIME ACIMA DA CABEÇA ★ ═══
                        if State.TeamCheckVisuals then
                            if not ESPDrawings[player].TeamName then
                                ESPDrawings[player].TeamName = Drawing.new("Text")
                                ESPDrawings[player].TeamName.Size = 13
                                ESPDrawings[player].TeamName.Center = true
                                ESPDrawings[player].TeamName.Outline = true
                                ESPDrawings[player].TeamName.OutlineColor = Color3.fromRGB(0, 0, 0)
                                ESPDrawings[player].TeamName.Font = 2
                            end
                            
                            -- ★ Cor do texto = cor do time ★
                            ESPDrawings[player].TeamName.Color = espColor
                            
                            -- Indicador se é aliado ou inimigo
                            local teamPrefix = ""
                            if isSameTeam then
                                teamPrefix = "👥 "
                            else
                                teamPrefix = "⚔ "
                            end
                            
                            ESPDrawings[player].TeamName.Text = teamPrefix .. "[" .. teamName .. "]"
                            
                            -- Posicionar acima do nome/vida
                            local yOffset = -18
                            if State.LifeStatusEnabled then
                                yOffset = -36
                            end
                            ESPDrawings[player].TeamName.Position = Vector2.new(screenPos.X, headPos.Y + yOffset)
                            ESPDrawings[player].TeamName.Visible = true
                        else
                            if ESPDrawings[player].TeamName then
                                ESPDrawings[player].TeamName.Visible = false
                            end
                        end
                        
                        -- ═══ DISTÂNCIA ═══
                        if State.TeamCheckVisuals or State.LifeStatusEnabled then
                            if not ESPDrawings[player].Distance then
                                ESPDrawings[player].Distance = Drawing.new("Text")
                                ESPDrawings[player].Distance.Size = 12
                                ESPDrawings[player].Distance.Center = true
                                ESPDrawings[player].Distance.Outline = true
                                ESPDrawings[player].Distance.OutlineColor = Color3.fromRGB(0, 0, 0)
                                ESPDrawings[player].Distance.Font = 2
                                ESPDrawings[player].Distance.Color = Color3.fromRGB(180, 180, 180)
                            end
                            
                            ESPDrawings[player].Distance.Text = "[" .. math.floor(distance) .. "m]"
                            ESPDrawings[player].Distance.Position = Vector2.new(screenPos.X, footPos.Y + 4)
                            ESPDrawings[player].Distance.Visible = true
                        else
                            if ESPDrawings[player].Distance then
                                ESPDrawings[player].Distance.Visible = false
                            end
                        end
                        
                        -- ═══ TRACER COM COR DO TIME ═══
                        if State.TracerEnabled then
                            if not ESPDrawings[player].Tracer then
                                ESPDrawings[player].Tracer = Drawing.new("Line")
                                ESPDrawings[player].Tracer.Thickness = 1.5
                                ESPDrawings[player].Tracer.Transparency = 0.6
                            end
                            
                            -- ★ Cor muda conforme o time quando Team Check está ativo ★
                            ESPDrawings[player].Tracer.Color = espColor
                            
                            ESPDrawings[player].Tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                            ESPDrawings[player].Tracer.To = Vector2.new(screenPos.X, screenPos.Y)
                            ESPDrawings[player].Tracer.Visible = true
                        else
                            if ESPDrawings[player].Tracer then
                                ESPDrawings[player].Tracer.Visible = false
                            end
                        end
                        
                    else
                        ClearESP(player)
                    end
                end)
            end
        end
        
        -- Limpar ESP de jogadores que não existem mais
        for player, _ in pairs(ESPDrawings) do
            if not player.Parent or not player.Character then
                ClearESP(player)
            end
        end
    else
        ClearAllESP()
    end
end)

-- Limpar ESP quando jogador sai
Players.PlayerRemoving:Connect(function(player)
    ClearESP(player)
end)

-- ═══════════════════════════════════════════
-- INICIALIZAÇÃO
-- ═══════════════════════════════════════════
State.CurrentTab = ""
task.wait(0.5)
SwitchTab("Home")

for _, data in ipairs(TabData) do
    local btn = TabButtons[data.Name].Button
    btn.MouseButton1Click:Connect(function()
        SwitchTab(data.Name)
    end)
end

-- ═══════════════════════════════════════════
-- BOTÃO DE TOGGLE UI
-- ═══════════════════════════════════════════
local ToggleUIBtn = Instance.new("TextButton")
ToggleUIBtn.Name = "ToggleUI"
ToggleUIBtn.Size = UDim2.new(0, 44, 0, 44)
ToggleUIBtn.Position = UDim2.new(0, 10, 0.5, -22)
ToggleUIBtn.BackgroundColor3 = Theme.Primary
ToggleUIBtn.Text = "⛓"
ToggleUIBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleUIBtn.TextSize = 20
ToggleUIBtn.Font = Enum.Font.GothamBold
ToggleUIBtn.Visible = false
ToggleUIBtn.ZIndex = 10
ToggleUIBtn.Parent = ScreenGui
CreateCorner(ToggleUIBtn, 22)
CreateShadow(ToggleUIBtn, 0.4, 20)

ToggleUIBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = true
    ToggleUIBtn.Visible = false
    Tween(MainFrame, {BackgroundTransparency = 0, Size = UDim2.new(0, 720, 0, 480)}, 0.4, Enum.EasingStyle.Back)
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.Home or input.KeyCode == Enum.KeyCode.Insert then
        if MainFrame.Visible then
            Tween(MainFrame, {BackgroundTransparency = 1, Size = UDim2.new(0, 680, 0, 440)}, 0.3)
            task.wait(0.3)
            MainFrame.Visible = false
            ToggleUIBtn.Visible = true
        else
            MainFrame.Visible = true
            ToggleUIBtn.Visible = false
            Tween(MainFrame, {BackgroundTransparency = 0, Size = UDim2.new(0, 720, 0, 480)}, 0.4, Enum.EasingStyle.Back)
        end
    end
end)

-- ═══════════════════════════════════════════
-- NOTIFICAÇÃO DE CARREGAMENTO
-- ═══════════════════════════════════════════
local Notification = Instance.new("Frame")
Notification.Name = "Notification"
Notification.Size = UDim2.new(0, 320, 0, 70)
Notification.Position = UDim2.new(0.5, -160, 1, 10)
Notification.BackgroundColor3 = Theme.CardBg
Notification.ZIndex = 20
Notification.Parent = ScreenGui
CreateCorner(Notification, 10)
CreateStroke(Notification, Theme.Primary, 1, 0.5)
CreateShadow(Notification, 0.3, 30)

local notifAccent = Instance.new("Frame")
notifAccent.Size = UDim2.new(0, 4, 0.7, 0)
notifAccent.Position = UDim2.new(0, 10, 0.15, 0)
notifAccent.BackgroundColor3 = Theme.Primary
notifAccent.ZIndex = 21
notifAccent.Parent = Notification
CreateCorner(notifAccent, 2)

local notifText = Instance.new("TextLabel")
notifText.Size = UDim2.new(1, -35, 1, 0)
notifText.Position = UDim2.new(0, 25, 0, 0)
notifText.BackgroundTransparency = 1
notifText.Text = "⛓ Xiterprison v1.1 carregado!\n🛡 Team Check disponível em Combat & Visuals\n⌨ Pressione Home/Insert para toggle"
notifText.TextColor3 = Theme.Text
notifText.TextSize = 12
notifText.Font = Enum.Font.GothamBold
notifText.TextWrapped = true
notifText.TextXAlignment = Enum.TextXAlignment.Left
notifText.ZIndex = 21
notifText.Parent = Notification

Tween(Notification, {Position = UDim2.new(0.5, -160, 1, -90)}, 0.6, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
task.delay(5, function()
    Tween(Notification, {Position = UDim2.new(0.5, -160, 1, 80)}, 0.5, Enum.EasingStyle.Quart, Enum.EasingDirection.In)
    task.wait(0.6)
    Notification:Destroy()
end)

-- ═══════════════════════════════════════════
-- LIMPEZA AO DESTRUIR
-- ═══════════════════════════════════════════
ScreenGui.Destroying:Connect(function()
    ClearAllESP()
    
    if FOVCircle then
        pcall(function() FOVCircle:Remove() end)
    end
    
    pcall(function()
        local char = LocalPlayer.Character
        if char then
            if char:FindFirstChild("Humanoid") then
                char.Humanoid.WalkSpeed = 16
                char.Humanoid.PlatformStand = false
            end
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hrp then
                local bv = hrp:FindFirstChild("FlyVelocity")
                if bv then bv:Destroy() end
                local bg = hrp:FindFirstChild("FlyGyro")
                if bg then bg:Destroy() end
            end
        end
    end)
    
    pcall(function()
        local lighting = game:GetService("Lighting")
        lighting.GlobalShadows = true
        settings().Rendering.QualityLevel = Enum.QualityLevel.Automatic
    end)
end)

print("═══════════════════════════════════════")
print("  ⛓ XITERPRISON v1.1 LOADED ⛓")
print("  Créditos: Thurzinnn & Thalles")
print("  ★ NOVO: Team Check (Combat & Visuals)")
print("  Toggle: Home/Insert")
print("═══════════════════════════════════════")
