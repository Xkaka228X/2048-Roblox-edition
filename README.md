-- ROBLOX 2048 COMPACT & SMOOTH (MOBILE READY)
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local Player = game.Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")
local FileName = "2048_Compact_Save.txt"

local function LoadBest()
    if isfile and isfile(FileName) then
        local success, result = pcall(function() return HttpService:JSONDecode(readfile(FileName)) end)
        if success and result then return result.Best or 0 end
    end
    return 0
end

local BestScore = LoadBest()
local CurrentScore = 0
local matrix = {{0,0,0,0},{0,0,0,0},{0,0,0,0},{0,0,0,0}}

-- UI
if PlayerGui:FindFirstChild("Game2048Compact") then PlayerGui.Game2048Compact:Destroy() end
local ScreenGui = Instance.new("ScreenGui", PlayerGui); ScreenGui.Name = "Game2048Compact"

local Main = Instance.new("Frame", ScreenGui)
Main.Size = UDim2.new(0, 320, 0, 480); Main.Position = UDim2.new(0.5, -160, 0.5, -240)
Main.BackgroundColor3 = Color3.fromRGB(187, 173, 160); Main.Active = true; Main.Draggable = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 15)

-- Плавность для UI
local function Pop(obj)
    obj.Size = UDim2.new(0, 0, 0, 0)
    TweenService:Create(obj, TweenInfo.new(0.2, Enum.EasingStyle.Back), {Size = UDim2.new(0, 68, 0, 68)}):Play()
end

-- Верхняя панель
local Header = Instance.new("Frame", Main)
Header.Size = UDim2.new(1, -20, 0, 80); Header.Position = UDim2.new(0, 10, 0, 10); Header.BackgroundTransparency = 1

local Title = Instance.new("TextLabel", Header)
Title.Text = "2048"; Title.Size = UDim2.new(0, 80, 1, 0); Title.TextColor3 = Color3.fromRGB(119, 110, 101)
Title.TextSize = 30; Title.Font = "GothamBold"; Title.BackgroundTransparency = 1

local ScoreBox = Instance.new("TextLabel", Header)
ScoreBox.Size = UDim2.new(0, 80, 0, 40); ScoreBox.Position = UDim2.new(0, 90, 0, 10)
ScoreBox.BackgroundColor3 = Color3.fromRGB(143, 130, 119); ScoreBox.TextColor3 = Color3.new(1,1,1)
ScoreBox.Text = "SCORE\n0"; ScoreBox.TextSize = 10; Instance.new("UICorner", ScoreBox)

local BestBox = Instance.new("TextLabel", Header)
BestBox.Size = UDim2.new(0, 80, 0, 40); BestBox.Position = UDim2.new(0, 180, 0, 10)
BestBox.BackgroundColor3 = Color3.fromRGB(143, 130, 119); BestBox.TextColor3 = Color3.new(1,1,1)
BestBox.Text = "BEST\n" .. BestScore; BestBox.TextSize = 10; Instance.new("UICorner", BestBox)

local Close = Instance.new("TextButton", Main)
Close.Size = UDim2.new(0, 25, 0, 25); Close.Position = UDim2.new(1, -30, 0, 5)
Close.Text = "X"; Close.BackgroundColor3 = Color3.fromRGB(200, 0, 0); Close.TextColor3 = Color3.new(1,1,1)
Close.Font = "GothamBold"; Close.MouseButton1Click:Connect(function() ScreenGui:Destroy() end)
Instance.new("UICorner", Close)

-- Игровое поле (компактное)
local Grid = Instance.new("Frame", Main)
Grid.Size = UDim2.new(0, 300, 0, 300); Grid.Position = UDim2.new(0.5, -150, 0, 90)
Grid.BackgroundColor3 = Color3.fromRGB(143, 130, 119)
Instance.new("UICorner", Grid)

local cells = {}
for r = 1, 4 do
    for c = 1, 4 do
        local cell = Instance.new("TextLabel", Grid)
        cell.Size = UDim2.new(0, 68, 0, 68)
        cell.Position = UDim2.new(0, (c-1)*73 + 6, 0, (r-1)*73 + 6)
        cell.BackgroundColor3 = Color3.fromRGB(205, 193, 180)
        cell.Text = ""; cell.Font = "GothamBold"; cell.TextSize = 22; cell.TextColor3 = Color3.new(1,1,1)
        Instance.new("UICorner", cell)
        cells[(r-1)*4 + c] = cell
    end
end

local colors = {[2]=Color3.fromRGB(238,228,218),[4]=Color3.fromRGB(237,224,200),[8]=Color3.fromRGB(242,177,121),[16]=Color3.fromRGB(245,149,99),[32]=Color3.fromRGB(246,124,95),[64]=Color3.fromRGB(246,94,59),[128]=Color3.fromRGB(237,207,114),[256]=Color3.fromRGB(237,204,97),[512]=Color3.fromRGB(237,200,80),[1024]=Color3.fromRGB(237,197,63),[2048]=Color3.fromRGB(237,194,46)}

local function UpdateUI(spawning)
    for r=1,4 do for c=1,4 do
        local val = matrix[r][c]
        local cell = cells[(r-1)*4 + c]
        local oldText = cell.Text
        cell.Text = val == 0 and "" or tostring(val)
        cell.BackgroundColor3 = colors[val] or (val>0 and Color3.fromRGB(60,58,50) or Color3.fromRGB(205,193,180))
        cell.TextColor3 = (val <= 4) and Color3.fromRGB(119,110,101) or Color3.new(1,1,1)
        
        if spawning and oldText == "" and cell.Text ~= "" then Pop(cell) end
    end end
    ScoreBox.Text = "SCORE\n"..CurrentScore
    if CurrentScore > BestScore then
        BestScore = CurrentScore
        BestBox.Text = "BEST\n"..BestScore
        if writefile then writefile(FileName, HttpService:JSONEncode({Best = BestScore})) end
    end
end

local function SpawnTile()
    local empty = {}
    for r=1,4 do for c=1,4 do if matrix[r][c] == 0 then table.insert(empty, {r,c}) end end end
    if #empty > 0 then
        local p = empty[math.random(1,#empty)]
        matrix[p[1]][p[2]] = math.random() < 0.9 and 2 or 4
        return true
    end
end

local function Move(dir)
    local old = HttpService:JSONEncode(matrix)
    local function process(line)
        local n = {}
        for i=1,4 do if line[i] ~= 0 then table.insert(n, line[i]) end end
        for i=1,#n-1 do if n[i] == n[i+1] then n[i] *= 2; CurrentScore += n[i]; table.remove(n, i+1) end end
        while #n < 4 do table.insert(n, 0) end
        return n
    end
    for i=1,4 do
        local line = {}
        if dir=="Left" or dir=="Right" then
            for j=1,4 do line[j] = matrix[i][j] end
            if dir=="Right" then line = {line[4],line[3],line[2],line[1]} end
            line = process(line)
            if dir=="Right" then line = {line[4],line[3],line[2],line[1]} end
            for j=1,4 do matrix[i][j] = line[j] end
        else
            for j=1,4 do line[j] = matrix[j][i] end
            if dir=="Down" then line = {line[4],line[3],line[2],line[1]} end
            line = process(line)
            if dir=="Down" then line = {line[4],line[3],line[2],line[1]} end
            for j=1,4 do matrix[j][i] = line[j] end
        end
    end
    if old ~= HttpService:JSONEncode(matrix) then SpawnTile(); UpdateUI(true) end
end

-- Управление
local Controls = Instance.new("Frame", Main)
Controls.Size = UDim2.new(0, 150, 0, 80); Controls.Position = UDim2.new(0.5, -75, 1, -85); Controls.BackgroundTransparency = 1

local function Btn(name, pos, d)
    local b = Instance.new("TextButton", Controls)
    b.Size = UDim2.new(0, 40, 0, 40); b.Position = pos; b.Text = name
    b.BackgroundColor3 = Color3.fromRGB(119, 110, 101); b.TextColor3 = Color3.new(1,1,1)
    b.MouseButton1Click:Connect(function() Move(d) end)
    Instance.new("UICorner", b)
end
Btn("↑", UDim2.new(0.5, -20, 0, 0), "Up"); Btn("↓", UDim2.new(0.5, -20, 0, 42), "Down")
Btn("←", UDim2.new(0, 10, 0, 21), "Left"); Btn("→", UDim2.new(1, -50, 0, 21), "Right")

local Res = Instance.new("TextButton", Main)
Res.Size = UDim2.new(0, 70, 0, 25); Res.Position = UDim2.new(1, -80, 0, 50)
Res.Text = "Restart"; Res.BackgroundColor3 = Color3.fromRGB(119, 110, 101); Res.TextColor3 = Color3.new(1,1,1); Res.TextSize = 10
Res.MouseButton1Click:Connect(function() 
    matrix = {{0,0,0,0},{0,0,0,0},{0,0,0,0},{0,0,0,0}}; CurrentScore = 0; SpawnTile(); SpawnTile(); UpdateUI(true) 
end)
Instance.new("UICorner", Res)

game:GetService("UserInputService").InputBegan:Connect(function(i, g)
    if not g then
        if i.KeyCode == Enum.KeyCode.W or i.KeyCode == Enum.KeyCode.Up then Move("Up")
        elseif i.KeyCode == Enum.KeyCode.S or i.KeyCode == Enum.KeyCode.Down then Move("Down")
        elseif i.KeyCode == Enum.KeyCode.A or i.KeyCode == Enum.KeyCode.Left then Move("Left")
        elseif i.KeyCode == Enum.KeyCode.D or i.KeyCode == Enum.KeyCode.Right then Move("Right") end
    end
end)

SpawnTile(); SpawnTile(); UpdateUI(true)
