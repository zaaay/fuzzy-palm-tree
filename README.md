--[[
    NeverLose UI Replacement Library
    Drop-in replacement for the NeverLose UI system
    Supports: Window, Tabs, Sections, Labels, Toggles, Sliders, Dropdowns, Keybinds, ColorPickers, Buttons
]]

local NeverLose = {}
getgenv().NeverLose = NeverLose

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer

-- UI Settings
NeverLose.MainColor = Color3.fromRGB(8, 8, 13)
NeverLose.AccentColor = Color3.fromRGB(78, 127, 252)
NeverLose.TextColor = Color3.fromRGB(255, 255, 255)
NeverLose.StrokeColor = Color3.fromRGB(45, 48, 58)
NeverLose.CornerRadius = 8
NeverLose.Font = Enum.Font.GothamMedium
NeverLose.BoldFont = Enum.Font.GothamBold
NeverLose.IconColor = Color3.fromRGB(255, 255, 255)
NeverLose.GlobalLogo = "rbxassetid://1234567890" -- Replace with your logo

NeverLose.Scales = {
    Small = UDim2.fromOffset(400, 350),
    Large = UDim2.fromOffset(600, 500),
    Mobile = UDim2.fromOffset(350, 400)
}

-- Utility functions
local function create(name, parent, props)
    local inst = Instance.new(name)
    for k, v in pairs(props or {}) do
        inst[k] = v
    end
    inst.Parent = parent
    return inst
end

local function addCorner(parent, radius)
    return create("UICorner", parent, {
        CornerRadius = UDim.new(0, radius or NeverLose.CornerRadius)
    })
end

local function addStroke(parent, color, thickness, transparency)
    return create("UIStroke", parent, {
        Color = color or NeverLose.StrokeColor,
        Thickness = thickness or 1,
        Transparency = transparency or 0.65
    })
end

-- Dragging system
local function makeDraggable(frame, dragArea)
    local dragging = false
    local dragStart, startPos
    
    dragArea.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
        end
    end)
    
    dragArea.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
    
    dragArea.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            if dragging then
                local delta = input.Position - dragStart
                frame.Position = UDim2.new(
                    startPos.X.Scale, startPos.X.Offset + delta.X,
                    startPos.Y.Scale, startPos.Y.Offset + delta.Y
                )
            end
        end
    end)
end

-- Keybind parsing
local function parseKey(key)
    if typeof(key) == "EnumItem" then return key end
    if typeof(key) == "string" then
        local lower = key:lower()
        local aliases = {
            m1 = "MouseButton1", m2 = "MouseButton2", m3 = "MouseButton3",
            mb1 = "MouseButton1", mb2 = "MouseButton2", mb3 = "MouseButton3",
            rclick = "MouseButton2", lclick = "MouseButton1"
        }
        if aliases[lower] then
            return Enum.UserInputType[aliases[lower]]
        end
        local keyCode = Enum.KeyCode:GetEnumItems()[key]
        if keyCode then return keyCode end
        local inputType = Enum.UserInputType:GetEnumItems()[key]
        if inputType then return inputType end
    end
    return nil
end
getgenv().parseKey = parseKey

-- Notification system
function NeverLose:CreateNotification()
    local notification = {}
    notification.new = function(title, text, duration)
        -- Simple notification implementation
        print(string.format("[%s] %s", title or "Notification", text or ""))
    end
    return notification
end

-- Logger system
function NeverLose:CreateLogger()
    local logger = {}
    logger.new = function(section, message, level)
        print(string.format("[%s] %s", section or "Log", message or ""))
    end
    return logger
end

-- Indicator system
function NeverLose:CreateIndicator()
    local indicator = {}
    return indicator
end

-- UI Element base
local Element = {}
Element.__index = Element

function Element.new(label)
    local self = setmetatable({}, Element)
    self.Label = label
    self.Elements = {}
    return self
end

function Element:AddToggle(props)
    props = props or {}
    local toggle = {}
    toggle.Type = "Toggle"
    toggle.Value = props.Default or false
    toggle.Flag = props.Flag
    toggle.Callback = props.Callback
    
    -- Create UI
    local row = create("Frame", self.Label.Container, {
        Size = UDim2.new(1, -20, 0, 30),
        BackgroundTransparency = 1,
        LayoutOrder = #self.Label.Elements + 1
    })
    create("UIPadding", row, {PaddingLeft = UDim.new(0, 10), PaddingTop = UDim.new(0, 5)})
    
    local text = create("TextLabel", row, {
        Size = UDim2.new(0.8, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = props.Name or "Toggle",
        TextColor3 = NeverLose.TextColor,
        TextSize = 13,
        Font = NeverLose.Font,
        TextXAlignment = Enum.TextXAlignment.Left
    })
    
    local toggleFrame = create("Frame", row, {
        Size = UDim2.fromOffset(24, 24),
        AnchorPoint = Vector2.new(1, 0.5),
        Position = UDim2.new(1, -10, 0.5, 0),
        BackgroundColor3 = NeverLose.MainColor,
        BorderSizePixel = 0
    })
    addCorner(toggleFrame, 6)
    addStroke(toggleFrame)
    
    local toggleFill = create("Frame", toggleFrame, {
        Size = UDim2.fromOffset(0, 0),
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.fromScale(0.5, 0.5),
        BackgroundColor3 = NeverLose.AccentColor,
        BorderSizePixel = 0
    })
    addCorner(toggleFill, 4)
    
    local button = create("TextButton", row, {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = ""
    })
    
    local function updateVisual()
        toggleFill:TweenSize(
            toggle.Value and UDim2.fromOffset(20, 20) or UDim2.fromOffset(0, 0),
            Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.15, true
        )
    end
    
    button.MouseButton1Click:Connect(function()
        toggle.Value = not toggle.Value
        updateVisual()
        if toggle.Callback then toggle.Callback(toggle.Value) end
    end)
    
    updateVisual()
    table.insert(self.Label.Elements, toggle)
    return toggle
end

function Element:AddSlider(props)
    props = props or {}
    local slider = {}
    slider.Type = "Slider"
    slider.Value = props.Default or props.Min or 0
    slider.Flag = props.Flag
    slider.Callback = props.Callback
    slider.Min = props.Min or 0
    slider.Max = props.Max or 100
    slider.Rounding = props.Rounding or 0
    
    local row = create("Frame", self.Label.Container, {
        Size = UDim2.new(1, -20, 0, 45),
        BackgroundTransparency = 1,
        LayoutOrder = #self.Label.Elements + 1
    })
    create("UIPadding", row, {PaddingLeft = UDim.new(0, 10), PaddingTop = UDim.new(0, 5)})
    
    local text = create("TextLabel", row, {
        Size = UDim2.new(1, 0, 0, 15),
        BackgroundTransparency = 1,
        Text = string.format("%s: %s", props.Name or "Slider", tostring(slider.Value)),
        TextColor3 = NeverLose.TextColor,
        TextSize = 13,
        Font = NeverLose.Font,
        TextXAlignment = Enum.TextXAlignment.Left
    })
    
    local bar = create("Frame", row, {
        Size = UDim2.new(1, -20, 0, 4),
        Position = UDim2.new(0, 10, 0, 25),
        BackgroundColor3 = NeverLose.MainColor,
        BorderSizePixel = 0
    })
    addCorner(bar, 2)
    
    local fill = create("Frame", bar, {
        Size = UDim2.fromScale(0, 1),
        BackgroundColor3 = NeverLose.AccentColor,
        BorderSizePixel = 0
    })
    addCorner(fill, 2)
    
    local knob = create("Frame", row, {
        Size = UDim2.fromOffset(14, 14),
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.new(0, 10, 0, 27),
        BackgroundColor3 = NeverLose.TextColor,
        BorderSizePixel = 0
    })
    addCorner(knob, 7)
    
    local function updateValue(input)
        local relative = math.clamp((input.Position.X - bar.AbsolutePosition.X) / bar.AbsoluteSize.X, 0, 1)
        local value = slider.Min + (slider.Max - slider.Min) * relative
        value = math.round(value * (10 ^ slider.Rounding)) / (10 ^ slider.Rounding)
        slider.Value = value
        text.Text = string.format("%s: %s", props.Name or "Slider", tostring(value))
        fill.Size = UDim2.fromScale(relative, 1)
        knob.Position = UDim2.new(0, 10 + bar.AbsoluteSize.X * relative, 0, 27)
        if slider.Callback then slider.Callback(value) end
    end
    
    local dragging = false
    knob.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            updateValue(input)
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            updateValue(input)
        end
    end)
    
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    
    -- Initial value
    local initRel = (slider.Value - slider.Min) / (slider.Max - slider.Min)
    fill.Size = UDim2.fromScale(initRel, 1)
    knob.Position = UDim2.new(0, 10 + bar.AbsoluteSize.X * initRel, 0, 27)
    
    table.insert(self.Label.Elements, slider)
    return slider
end

function Element:AddDropdown(props)
    props = props or {}
    local dropdown = {}
    dropdown.Type = "Dropdown"
    dropdown.Value = props.Default or props.Values[1]
    dropdown.Flag = props.Flag
    dropdown.Callback = props.Callback
    dropdown.Values = props.Values or {}
    dropdown.Multi = props.Multi or false
    
    local row = create("Frame", self.Label.Container, {
        Size = UDim2.new(1, -20, 0, 45),
        BackgroundTransparency = 1,
        LayoutOrder = #self.Label.Elements + 1
    })
    create("UIPadding", row, {PaddingLeft = UDim.new(0, 10), PaddingTop = UDim.new(0, 5)})
    
    local text = create("TextLabel", row, {
        Size = UDim2.new(1, 0, 0, 15),
        BackgroundTransparency = 1,
        Text = props.Name or "Dropdown",
        TextColor3 = NeverLose.TextColor,
        TextSize = 13,
        Font = NeverLose.Font,
        TextXAlignment = Enum.TextXAlignment.Left
    })
    
    local box = create("Frame", row, {
        Size = UDim2.new(1, -20, 0, 24),
        Position = UDim2.new(0, 10, 0, 18),
        BackgroundColor3 = NeverLose.MainColor,
        BorderSizePixel = 0
    })
    addCorner(box, 4)
    addStroke(box)
    
    local valueLabel = create("TextLabel", box, {
        Size = UDim2.new(0.8, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = tostring(dropdown.Value),
        TextColor3 = NeverLose.TextColor,
        TextSize = 12,
        Font = NeverLose.Font,
        TextXAlignment = Enum.TextXAlignment.Left
    })
    create("UIPadding", valueLabel, {PaddingLeft = UDim.new(0, 8)})
    
    local arrow = create("TextLabel", box, {
        Size = UDim2.new(0.2, 0, 1, 0),
        Position = UDim2.new(0.8, 0, 0, 0),
        BackgroundTransparency = 1,
        Text = "▼",
        TextColor3 = NeverLose.AccentColor,
        TextSize = 10,
        Font = NeverLose.Font
    })
    
    local dropdownList = create("Frame", self.Label.Container, {
        Size = UDim2.new(1, -20, 0, 0),
        Position = UDim2.new(0, 10, 0, 0),
        BackgroundColor3 = NeverLose.MainColor,
        BorderSizePixel = 0,
        Visible = false,
        ZIndex = 50
    })
    addCorner(dropdownList, 4)
    addStroke(dropdownList)
    
    local listLayout = create("UIListLayout", dropdownList, {
        Padding = UDim.new(0, 2),
        SortOrder = Enum.SortOrder.LayoutOrder
    })
    
    local function closeList()
        dropdownList.Visible = false
    end
    
    box.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dropdownList.Visible = not dropdownList.Visible
        end
    end)
    
    -- Populate values
    for i, value in ipairs(dropdown.Values) do
        local item = create("TextButton", dropdownList, {
            Size = UDim2.new(1, 0, 0, 20),
            BackgroundTransparency = 1,
            Text = tostring(value),
            TextColor3 = NeverLose.TextColor,
            TextSize = 12,
            Font = NeverLose.Font,
            TextXAlignment = Enum.TextXAlignment.Left,
            LayoutOrder = i
        })
        create("UIPadding", item, {PaddingLeft = UDim.new(0, 8)})
        
        item.MouseButton1Click:Connect(function()
            if dropdown.Multi then
                if type(dropdown.Value) ~= "table" then dropdown.Value = {} end
                if dropdown.Value[value] then
                    dropdown.Value[value] = nil
                else
                    dropdown.Value[value] = true
                end
                local names = {}
                for k, v in pairs(dropdown.Value) do
                    if v then table.insert(names, k) end
                end
                valueLabel.Text = #names > 0 and table.concat(names, ", ") or "None"
            else
                dropdown.Value = value
                valueLabel.Text = tostring(value)
            end
            if dropdown.Callback then dropdown.Callback(dropdown.Value) end
            if not dropdown.Multi then closeList() end
        end)
    end
    
    -- Close on outside click
    UserInputService.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            local mousePos = UserInputService:GetMouseLocation()
            local absPos = dropdownList.AbsolutePosition
            local absSize = dropdownList.AbsoluteSize
            if not (mousePos.X >= absPos.X and mousePos.X <= absPos.X + absSize.X and
                    mousePos.Y >= absPos.Y and mousePos.Y <= absPos.Y + absSize.Y) then
                closeList()
            end
        end
    end)
    
    table.insert(self.Label.Elements, dropdown)
    return dropdown
end

function Element:AddKeybind(props)
    props = props or {}
    local keybind = {}
    keybind.Type = "Keybind"
    keybind.Value = parseKey(props.Default) or Enum.KeyCode.F
    keybind.Flag = props.Flag
    keybind.Callback = props.Callback
    
    local row = create("Frame", self.Label.Container, {
        Size = UDim2.new(1, -20, 0, 30),
        BackgroundTransparency = 1,
        LayoutOrder = #self.Label.Elements + 1
    })
    create("UIPadding", row, {PaddingLeft = UDim.new(0, 10), PaddingTop = UDim.new(0, 5)})
    
    local text = create("TextLabel", row, {
        Size = UDim2.new(0.6, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = props.Name or "Keybind",
        TextColor3 = NeverLose.TextColor,
        TextSize = 13,
        Font = NeverLose.Font,
        TextXAlignment = Enum.TextXAlignment.Left
    })
    
    local keyButton = create("TextButton", row, {
        Size = UDim2.new(0.3, 0, 0.8, 0),
        AnchorPoint = Vector2.new(1, 0.5),
        Position = UDim2.new(1, -10, 0.5, 0),
        BackgroundColor3 = NeverLose.MainColor,
        BorderSizePixel = 0,
        Text = keybind.Value.Name or "None",
        TextColor3 = NeverLose.AccentColor,
        TextSize = 12,
        Font = NeverLose.BoldFont
    })
    addCorner(keyButton, 4)
    addStroke(keyButton)
    
    local listening = false
    keyButton.MouseButton1Click:Connect(function()
        listening = true
        keyButton.Text = "..."
    end)
    
    UserInputService.InputBegan:Connect(function(input)
        if listening then
            if input.KeyCode ~= Enum.KeyCode.Unknown and input.KeyCode ~= Enum.KeyCode.None then
                keybind.Value = input.KeyCode
                keyButton.Text = input.KeyCode.Name
                listening = false
                if keybind.Callback then keybind.Callback(input.KeyCode) end
            elseif input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.MouseButton2 then
                keybind.Value = input.UserInputType
                keyButton.Text = input.UserInputType.Name
                listening = false
                if keybind.Callback then keybind.Callback(input.UserInputType) end
            end
        end
    end)
    
    table.insert(self.Label.Elements, keybind)
    return keybind
end

function Element:AddColorPicker(props)
    props = props or {}
    local colorPicker = {}
    colorPicker.Type = "ColorPicker"
    colorPicker.Value = props.Default or Color3.fromRGB(255, 255, 255)
    colorPicker.Flag = props.Flag
    colorPicker.Callback = props.Callback
    
    local row = create("Frame", self.Label.Container, {
        Size = UDim2.new(1, -20, 0, 30),
        BackgroundTransparency = 1,
        LayoutOrder = #self.Label.Elements + 1
    })
    create("UIPadding", row, {PaddingLeft = UDim.new(0, 10), PaddingTop = UDim.new(0, 5)})
    
    local text = create("TextLabel", row, {
        Size = UDim2.new(0.7, 0, 1, 0),
        BackgroundTransparency = 1,
        Text = props.Name or "Color",
        TextColor3 = NeverLose.TextColor,
        TextSize = 13,
        Font = NeverLose.Font,
        TextXAlignment = Enum.TextXAlignment.Left
    })
    
    local colorButton = create("TextButton", row, {
        Size = UDim2.new(0.2, 0, 0.8, 0),
        AnchorPoint = Vector2.new(1, 0.5),
        Position = UDim2.new(1, -10, 0.5, 0),
        BackgroundColor3 = colorPicker.Value,
        BorderSizePixel = 0,
        Text = ""
    })
    addCorner(colorButton, 4)
    addStroke(colorButton)
    
    local popup = create("Frame", self.Label.Container, {
        Size = UDim2.fromOffset(200, 150),
        BackgroundColor3 = NeverLose.MainColor,
        BorderSizePixel = 0,
        Visible = false,
        ZIndex = 100
    })
    addCorner(popup, 8)
    addStroke(popup)
    
    -- Simple color picker with hue slider
    local hueSlider = create("Frame", popup, {
        Size = UDim2.new(1, -20, 0, 15),
        Position = UDim2.new(0, 10, 0, 10),
        BackgroundColor3 = Color3.fromHSV(0, 1, 1),
        BorderSizePixel = 0
    })
    addCorner(hueSlider, 3)
    
    local satVal = create("Frame", popup, {
        Size = UDim2.new(1, -20, 0, 80),
        Position = UDim2.new(0, 10, 0, 35),
        BackgroundColor3 = Color3.fromRGB(255, 0, 0),
        BorderSizePixel = 0,
        ClipsDescendants = true
    })
    addCorner(satVal, 3)
    
    local whiteGrad = create("UIGradient", satVal, {
        Color = ColorSequence.new(Color3.fromRGB(255, 255, 255), Color3.fromRGB(255, 0, 0)),
        Rotation = 90
    })
    
    local blackGrad = create("UIGradient", satVal, {
        Color = ColorSequence.new(Color3.fromRGB(0, 0, 0), Color3.fromRGB(255, 255, 255))
    })
    
    colorButton.MouseButton1Click:Connect(function()
        popup.Visible = not popup.Visible
    end)
    
    -- Simple hue picker
    local hue = 0
    local function updateColor()
        local color = Color3.fromHSV(hue, 1, 1)
        satVal.BackgroundColor3 = color
        whiteGrad.Color = ColorSequence.new(Color3.fromRGB(255, 255, 255), color)
        colorButton.BackgroundColor3 = colorPicker.Value
        if colorPicker.Callback then colorPicker.Callback(colorPicker.Value) end
    end
    
    hueSlider.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            local rel = math.clamp((input.Position.X - hueSlider.AbsolutePosition.X) / hueSlider.AbsoluteSize.X, 0, 1)
            hue = rel
            colorPicker.Value = Color3.fromHSV(hue, 1, 1)
            updateColor()
        end
    end)
    
    satVal.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            local relX = math.clamp((input.Position.X - satVal.AbsolutePosition.X) / satVal.AbsoluteSize.X, 0, 1)
            local relY = math.clamp((input.Position.Y - satVal.AbsolutePosition.Y) / satVal.AbsoluteSize.Y, 0, 1)
            colorPicker.Value = Color3.fromHSV(hue, relX, 1 - relY)
            updateColor()
        end
    end)
    
    table.insert(self.Label.Elements, colorPicker)
    return colorPicker
end

function Element:AddButton(props)
    props = props or {}
    local button = {}
    button.Type = "Button"
    button.Callback = props.Callback
    
    local row = create("Frame", self.Label.Container, {
        Size = UDim2.new(1, -20, 0, 30),
        BackgroundTransparency = 1,
        LayoutOrder = #self.Label.Elements + 1
    })
    create("UIPadding", row, {PaddingLeft = UDim.new(0, 10), PaddingTop = UDim.new(0, 5)})
    
    local buttonInst = create("TextButton", row, {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundColor3 = NeverLose.MainColor,
        BorderSizePixel = 0,
        Text = props.Name or "Button",
        TextColor3 = NeverLose.TextColor,
        TextSize = 13,
        Font = NeverLose.Font
    })
    addCorner(buttonInst, 4)
    addStroke(buttonInst)
    
    buttonInst.MouseButton1Click:Connect(function()
        if button.Callback then button.Callback() end
    end)
    
    table.insert(self.Label.Elements, button)
    return button
end

function Element:AddOption()
    return self
end

-- Label class
local Label = {}
Label.__index = Label

function Label.new(container, name)
    local self = setmetatable({}, Label)
    self.Name = name or "Label"
    self.Container = container
    self.Elements = {}
    
    -- Create section frame
    self.Frame = create("Frame", container, {
        Size = UDim2.new(1, -10, 0, 0),
        BackgroundColor3 = NeverLose.MainColor,
        BackgroundTransparency = 0.1,
        BorderSizePixel = 0,
        AutomaticSize = Enum.AutomaticSize.Y
    })
    addCorner(self.Frame, 6)
    addStroke(self.Frame, NeverLose.StrokeColor, 1, 0.5)
    
    self.Inner = create("Frame", self.Frame, {
        Size = UDim2.new(1, -4, 1, -4),
        Position = UDim2.new(0, 2, 0, 2),
        BackgroundTransparency = 1,
        AutomaticSize = Enum.AutomaticSize.Y
    })
    
    local title = create("TextLabel", self.Inner, {
        Size = UDim2.new(1, -20, 0, 20),
        BackgroundTransparency = 1,
        Text = name or "",
        TextColor3 = NeverLose.AccentColor,
        TextSize = 12,
        Font = NeverLose.BoldFont,
        TextXAlignment = Enum.TextXAlignment.Left
    })
    create("UIPadding", title, {PaddingLeft = UDim.new(0, 10), PaddingTop = UDim.new(0, 3)})
    
    local separator = create("Frame", self.Inner, {
        Size = UDim2.new(1, -20, 0, 1),
        Position = UDim2.new(0, 10, 0, 22),
        BackgroundColor3 = NeverLose.StrokeColor,
        BackgroundTransparency = 0.5,
        BorderSizePixel = 0
    })
    
    self.ElementsContainer = create("Frame", self.Inner, {
        Size = UDim2.new(1, 0, 0, 0),
        Position = UDim2.new(0, 0, 0, 25),
        BackgroundTransparency = 1,
        AutomaticSize = Enum.AutomaticSize.Y
    })
    
    self.Layout = create("UIListLayout", self.ElementsContainer, {
        Padding = UDim.new(0, 2),
        SortOrder = Enum.SortOrder.LayoutOrder,
        HorizontalAlignment = Enum.HorizontalAlignment.Center
    })
    
    return self
end

function Label:AddLabel(name)
    local element = Element.new(self)
    local label = create("TextLabel", self.ElementsContainer, {
        Size = UDim2.new(1, -20, 0, 25),
        BackgroundTransparency = 1,
        Text = name or "",
        TextColor3 = NeverLose.TextColor,
        TextSize = 13,
        Font = NeverLose.Font,
        TextXAlignment = Enum.TextXAlignment.Left,
        LayoutOrder = #self.Elements + 1
    })
    create("UIPadding", label, {PaddingLeft = UDim.new(0, 10)})
    self.Elements[label] = element
    return element
end

-- Section class
local Section = {}
Section.__index = Section

function Section.new(container, name)
    local self = setmetatable({}, Section)
    self.Name = name or "Section"
    self.Container = container
    
    self.Frame = create("Frame", container, {
        Size = UDim2.new(1, -10, 0, 0),
        BackgroundColor3 = NeverLose.MainColor,
        BackgroundTransparency = 0.1,
        BorderSizePixel = 0,
        AutomaticSize = Enum.AutomaticSize.Y
    })
    addCorner(self.Frame, 6)
    addStroke(self.Frame, NeverLose.StrokeColor, 1, 0.5)
    
    self.Inner = create("Frame", self.Frame, {
        Size = UDim2.new(1, -4, 1, -4),
        Position = UDim2.new(0, 2, 0, 2),
        BackgroundTransparency = 1,
        AutomaticSize = Enum.AutomaticSize.Y
    })
    
    local title = create("TextLabel", self.Inner, {
        Size = UDim2.new(1, -20, 0, 20),
        BackgroundTransparency = 1,
        Text = name or "",
        TextColor3 = NeverLose.AccentColor,
        TextSize = 12,
        Font = NeverLose.BoldFont,
        TextXAlignment = Enum.TextXAlignment.Left
    })
    create("UIPadding", title, {PaddingLeft = UDim.new(0, 10), PaddingTop = UDim.new(0, 3)})
    
    local separator = create("Frame", self.Inner, {
        Size = UDim2.new(1, -20, 0, 1),
        Position = UDim2.new(0, 10, 0, 22),
        BackgroundColor3 = NeverLose.StrokeColor,
        BackgroundTransparency = 0.5,
        BorderSizePixel = 0
    })
    
    self.ElementsContainer = create("Frame", self.Inner, {
        Size = UDim2.new(1, 0, 0, 0),
        Position = UDim2.new(0, 0, 0, 25),
        BackgroundTransparency = 1,
        AutomaticSize = Enum.AutomaticSize.Y
    })
    
    self.Layout = create("UIListLayout", self.ElementsContainer, {
        Padding = UDim.new(0, 2),
        SortOrder = Enum.SortOrder.LayoutOrder,
        HorizontalAlignment = Enum.HorizontalAlignment.Center
    })
    
    return self
end

function Section:AddLabel(name)
    return Label.new(self.ElementsContainer, name)
end

-- Tab class
local Tab = {}
Tab.__index = Tab

function Tab.new(window, props)
    local self = setmetatable({}, Tab)
    self.Name = props.Name or "Tab"
    self.Icon = props.Icon or ""
    self.Window = window
    self.Sections = {}
    
    self.Button = create("TextButton", window.TabBar, {
        Size = UDim2.new(0, 80, 0, 30),
        BackgroundColor3 = NeverLose.MainColor,
        BackgroundTransparency = 0.5,
        BorderSizePixel = 0,
        Text = (self.Icon ~= "" and self.Icon .. " " or "") .. self.Name,
        TextColor3 = NeverLose.TextColor,
        TextSize = 12,
        Font = NeverLose.BoldFont
    })
    addCorner(self.Button, 4)
    
    self.Page = create("ScrollingFrame", window.Content, {
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 4,
        Visible = false,
        AutomaticCanvasSize = Enum.AutomaticSize.Y
    })
    
    self.Layout = create("UIListLayout", self.Page, {
        Padding = UDim.new(0, 8),
        SortOrder = Enum.SortOrder.LayoutOrder,
        HorizontalAlignment = Enum.HorizontalAlignment.Center
    })
    
    self.Button.MouseButton1Click:Connect(function()
        window:SelectTab(self)
    end)
    
    table.insert(window.Tabs, self)
    return self
end

function Tab:AddSection(props)
    props = props or {}
    local section = Section.new(self.Page, props.Name or "Section")
    table.insert(self.Sections, section)
    return section
end

function Tab:AddSubTab(props)
    -- Simplified - just return a tab-like object
    return self
end

-- Window class
local Window = {}
Window.__index = Window

function Window.new(props)
    props = props or {}
    local self = setmetatable({}, Window)
    self.Name = props.Name or "UI"
    self.Size = props.Size or NeverLose.Scales.Large
    self.Keybind = parseKey(props.Keybind) or Enum.KeyCode.RightControl
    self.Tabs = {}
    self.CurrentTab = nil
    self.Visible = true
    
    -- Create ScreenGui
    self.ScreenGui = create("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"), {
        Name = "NeverLoseUI",
        ResetOnSpawn = false,
        ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    })
    NeverLose.ScreenGui = self.ScreenGui
    
    -- Main frame
    self.MainFrame = create("Frame", self.ScreenGui, {
        Size = self.Size,
        Position = UDim2.new(0.5, -self.Size.X.Offset / 2, 0.5, -self.Size.Y.Offset / 2),
        BackgroundColor3 = NeverLose.MainColor,
        BackgroundTransparency = 0.1,
        BorderSizePixel = 0
    })
    addCorner(self.MainFrame, 10)
    addStroke(self.MainFrame, NeverLose.StrokeColor, 1.5, 0.5)
    
    -- Title bar
    self.TitleBar = create("Frame", self.MainFrame, {
        Size = UDim2.new(1, 0, 0, 35),
        BackgroundColor3 = NeverLose.MainColor,
        BackgroundTransparency = 0.2,
        BorderSizePixel = 0
    })
    addCorner(self.TitleBar, UDim.new(0, 10))
    
    self.Title = create("TextLabel", self.TitleBar, {
        Size = UDim2.new(1, -40, 1, 0),
        BackgroundTransparency = 1,
        Text = self.Name,
        TextColor3 = NeverLose.TextColor,
        TextSize = 16,
        Font = NeverLose.BoldFont,
        TextXAlignment = Enum.TextXAlignment.Center
    })
    
    -- Close button
    self.CloseButton = create("TextButton", self.TitleBar, {
        Size = UDim2.fromOffset(25, 25),
        AnchorPoint = Vector2.new(1, 0.5),
        Position = UDim2.new(1, -5, 0.5, 0),
        BackgroundColor3 = Color3.fromRGB(255, 50, 50),
        BorderSizePixel = 0,
        Text = "X",
        TextColor3 = Color3.fromRGB(255, 255, 255),
        TextSize = 14,
        Font = NeverLose.BoldFont
    })
    addCorner(self.CloseButton, 6)
    
    self.CloseButton.MouseButton1Click:Connect(function()
        self:ToggleInterface()
    end)
    
    -- Tab bar
    self.TabBar = create("Frame", self.MainFrame, {
        Size = UDim2.new(1, -20, 0, 35),
        Position = UDim2.new(0, 10, 0, 40),
        BackgroundTransparency = 1,
        BorderSizePixel = 0
    })
    
    self.TabLayout = create("UIListLayout", self.TabBar, {
        Padding = UDim.new(0, 4),
        FillDirection = Enum.FillDirection.Horizontal,
        SortOrder = Enum.SortOrder.LayoutOrder
    })
    
    -- Content
    self.Content = create("Frame", self.MainFrame, {
        Size = UDim2.new(1, -20, 1, -85),
        Position = UDim2.new(0, 10, 0, 80),
        BackgroundTransparency = 1,
        BorderSizePixel = 0
    })
    
    -- Make draggable
    makeDraggable(self.MainFrame, self.TitleBar)
    
    -- Keybind toggle
    UserInputService.InputBegan:Connect(function(input)
        if input.KeyCode == self.Keybind then
            self:ToggleInterface()
        end
    end)
    
    return self
end

function Window:AddTab(props)
    return Tab.new(self, props)
end

function Window:SelectTab(tab)
    if self.CurrentTab then
        self.CurrentTab.Button.BackgroundColor3 = NeverLose.MainColor
        self.CurrentTab.Button.BackgroundTransparency = 0.5
        self.CurrentTab.Page.Visible = false
    end
    self.CurrentTab = tab
    tab.Button.BackgroundColor3 = NeverLose.AccentColor
    tab.Button.BackgroundTransparency = 0.3
    tab.Page.Visible = true
end

function Window:ToggleInterface()
    self.Visible = not self.Visible
    self.MainFrame.Visible = self.Visible
end

function Window:SetSize(size)
    self.Size = size
    self.MainFrame.Size = size
    self.MainFrame.Position = UDim2.new(0.5, -size.X.Offset / 2, 0.5, -size.Y.Offset / 2)
end

function Window:Destroy()
    self.ScreenGui:Destroy()
end

function Window:Watermark()
    local watermark = {}
    watermark.AddBlock = function()
        local block = {}
        block.Input = function(callback)
            -- Keybind input for watermark
            return block
        end
        return block
    end
    watermark.Destroy = function() end
    return watermark
end

-- UserSettings property
Window.__index.UserSettings = nil

-- CreateWindow function
function NeverLose:CreateWindow(props)
    local window = Window.new(props)
    
    -- Add UserSettings tab
    window.UserSettings = window:AddTab({Name = "Settings", Icon = "⚙"})
    
    return window
end

-- Export the library
getgenv().NeverLose = NeverLose

print("[NeverLose] UI Library loaded successfully!")
