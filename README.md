local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

--============================================================
-- ScreenGui base
--============================================================
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "TradeUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

--============================================================
-- Botão "Enviar"
--============================================================
local sendButton = Instance.new("TextButton")
sendButton.Size = UDim2.new(0, 90, 0, 32)
sendButton.Position = UDim2.new(0.5, -45, 0.5, -16)
sendButton.BackgroundColor3 = Color3.fromRGB(235, 235, 235)
sendButton.BorderSizePixel = 0
sendButton.Text = ""
sendButton.AutoButtonColor = false
sendButton.ZIndex = 2
sendButton.Parent = screenGui
Instance.new("UICorner", sendButton).CornerRadius = UDim.new(0, 6)

local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(200, 200, 200)
stroke.Thickness = 1
stroke.Parent = sendButton

local icon = Instance.new("TextLabel")
icon.Size = UDim2.new(0, 16, 1, 0)
icon.Position = UDim2.new(0, 12, 0, 0)
icon.BackgroundTransparency = 1
icon.Text = "↑"
icon.TextColor3 = Color3.fromRGB(60, 60, 60)
icon.Font = Enum.Font.GothamBold
icon.TextSize = 16
icon.ZIndex = 2
icon.Parent = sendButton

local label = Instance.new("TextLabel")
label.Size = UDim2.new(0, 60, 1, 0)
label.Position = UDim2.new(0, 30, 0, 0)
label.BackgroundTransparency = 1
label.Text = "Enviar"
label.TextColor3 = Color3.fromRGB(40, 40, 40)
label.Font = Enum.Font.Gotham
label.TextSize = 15
label.TextXAlignment = Enum.TextXAlignment.Left
label.ZIndex = 2
label.Parent = sendButton

sendButton.MouseEnter:Connect(function()
    sendButton.BackgroundColor3 = Color3.fromRGB(225, 225, 225)
end)
sendButton.MouseLeave:Connect(function()
    sendButton.BackgroundColor3 = Color3.fromRGB(235, 235, 235)
end)

--============================================================
-- Painel de busca (aparece ao clicar em "Enviar")
--============================================================
local panel = Instance.new("Frame")
panel.Name = "SearchPanel"
panel.Size = UDim2.new(0, 340, 0, 420)
panel.Position = UDim2.new(0.5, -170, 0.5, -210)
panel.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
panel.BorderSizePixel = 0
panel.Visible = false
panel.Parent = screenGui
Instance.new("UICorner", panel).CornerRadius = UDim.new(0, 12)

local panelShadowStroke = Instance.new("UIStroke")
panelShadowStroke.Color = Color3.fromRGB(225, 225, 225)
panelShadowStroke.Thickness = 1
panelShadowStroke.Parent = panel

-- Caixa de busca
local searchBox = Instance.new("TextBox")
searchBox.Size = UDim2.new(1, -32, 0, 40)
searchBox.Position = UDim2.new(0, 16, 0, 16)
searchBox.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
searchBox.PlaceholderText = "Busca por nome de usuário"
searchBox.PlaceholderColor3 = Color3.fromRGB(150, 150, 150)
searchBox.Text = ""
searchBox.TextColor3 = Color3.fromRGB(30, 30, 30)
searchBox.Font = Enum.Font.Gotham
searchBox.TextSize = 14
searchBox.TextXAlignment = Enum.TextXAlignment.Left
searchBox.ClearTextOnFocus = false
searchBox.Parent = panel

local searchPadding = Instance.new("UIPadding")
searchPadding.PaddingLeft = UDim.new(0, 10)
searchPadding.Parent = searchBox

local searchCorner = Instance.new("UICorner")
searchCorner.CornerRadius = UDim.new(0, 8)
searchCorner.Parent = searchBox

local searchStroke = Instance.new("UIStroke")
searchStroke.Color = Color3.fromRGB(88, 101, 242)
searchStroke.Thickness = 1.5
searchStroke.Parent = searchBox

-- Título "Minhas amizades (N)"
local friendsTitle = Instance.new("TextLabel")
friendsTitle.Size = UDim2.new(1, -32, 0, 24)
friendsTitle.Position = UDim2.new(0, 16, 0, 68)
friendsTitle.BackgroundTransparency = 1
friendsTitle.Text = "Minhas amizades (0)"
friendsTitle.TextColor3 = Color3.fromRGB(30, 30, 30)
friendsTitle.Font = Enum.Font.GothamBold
friendsTitle.TextSize = 15
friendsTitle.TextXAlignment = Enum.TextXAlignment.Left
friendsTitle.Parent = panel

-- Lista rolável de amigos
local listFrame = Instance.new("ScrollingFrame")
listFrame.Size = UDim2.new(1, -16, 1, -104)
listFrame.Position = UDim2.new(0, 8, 0, 96)
listFrame.BackgroundTransparency = 1
listFrame.BorderSizePixel = 0
listFrame.ScrollBarThickness = 4
listFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
listFrame.Parent = panel

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 2)
listLayout.Parent = listFrame

-- Botão de fechar
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 28, 0, 28)
closeBtn.Position = UDim2.new(1, -40, 0, 12)
closeBtn.BackgroundTransparency = 1
closeBtn.Text = "X"
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 16
closeBtn.TextColor3 = Color3.fromRGB(120, 120, 120)
closeBtn.ZIndex = 3
closeBtn.Parent = panel

--============================================================
-- Carrega a lista de amigos reais do jogador (API do Roblox)
--============================================================
local allFriends = {} -- { {Name=..., UserId=..., Thumb=...}, ... }

local function loadFriends()
    local success, friendPages = pcall(function()
        return Players:GetFriendsAsync(player.UserId)
    end)

    if not success or not friendPages then
        friendsTitle.Text = "Minhas amizades (0)"
        return
    end

    local count = 0
    local page = friendPages:GetCurrentPage()
    for _, friendData in ipairs(page) do
        count += 1
        table.insert(allFriends, {
            Name = friendData.Username,
            UserId = friendData.Id,
        })
    end

    while not friendPages.IsFinished and friendPages.IsFinished ~= nil do
        local ok = pcall(function() friendPages:AdvanceToNextPageAsync() end)
        if not ok then break end
        if friendPages.IsFinished then break end
        for _, friendData in ipairs(friendPages:GetCurrentPage()) do
            count += 1
            table.insert(allFriends, {
                Name = friendData.Username,
                UserId = friendData.Id,
            })
        end
    end

    friendsTitle.Text = "Minhas amizades (" .. count .. ")"
end

--============================================================
-- Renderiza a lista (com filtro opcional de busca)
--============================================================
local onFriendSelected -- callback definido mais abaixo

local function clearList()
    for _, child in ipairs(listFrame:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end
end

local function createFriendRow(friendInfo)
    local row = Instance.new("TextButton")
    row.Size = UDim2.new(1, 0, 0, 44)
    row.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    row.AutoButtonColor = false
    row.Text = ""
    row.Parent = listFrame

    row.MouseEnter:Connect(function()
        row.BackgroundColor3 = Color3.fromRGB(245, 245, 245)
    end)
    row.MouseLeave:Connect(function()
        row.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    end)

    local avatar = Instance.new("ImageLabel")
    avatar.Size = UDim2.new(0, 32, 0, 32)
    avatar.Position = UDim2.new(0, 8, 0.5, -16)
    avatar.BackgroundColor3 = Color3.fromRGB(230, 230, 230)
    avatar.Image = "rbxthumb://type=AvatarHeadShot&id=" .. friendInfo.UserId .. "&w=150&h=150"
    avatar.Parent = row
    Instance.new("UICorner", avatar).CornerRadius = UDim.new(1, 0)

    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, -56, 1, 0)
    nameLabel.Position = UDim2.new(0, 48, 0, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = friendInfo.Name
    nameLabel.TextColor3 = Color3.fromRGB(30, 30, 30)
    nameLabel.Font = Enum.Font.Gotham
    nameLabel.TextSize = 14
    nameLabel.TextXAlignment = Enum.TextXAlignment.Left
    nameLabel.Parent = row

    row.MouseButton1Click:Connect(function()
        if onFriendSelected then
            onFriendSelected(friendInfo)
        end
    end)

    return row
end

local function renderList(filterText)
    clearList()
    filterText = (filterText or ""):lower()

    local shown = 0
    for _, friendInfo in ipairs(allFriends) do
        if filterText == "" or friendInfo.Name:lower():find(filterText, 1, true) then
            createFriendRow(friendInfo)
            shown += 1
        end
    end

    listFrame.CanvasSize = UDim2.new(0, 0, 0, shown * 46)
end

searchBox:GetPropertyChangedSignal("Text"):Connect(function()
    renderList(searchBox.Text)
end)

--============================================================
-- Abrir / Fechar painel
--============================================================
sendButton.MouseButton1Click:Connect(function()
    panel.Visible = true
    searchBox.Text = ""
    if #allFriends == 0 then
        loadFriends()
    end
    renderList("")
end)

closeBtn.MouseButton1Click:Connect(function()
    panel.Visible = false
end)

--============================================================
-- O que fazer quando o jogador escolhe um amigo
-- (aqui você define a ação real: abrir janela de trade, etc.)
--============================================================
onFriendSelected = function(friendInfo)
    print("Selecionado para trade: " .. friendInfo.Name .. " (" .. friendInfo.UserId .. ")")
    panel.Visible = false

    -- Exemplo: aqui você chamaria seu sistema real de trade,
    -- por exemplo abrindo uma segunda tela com os itens do jogador
    -- e do amigo selecionado, ou disparando um RemoteEvent para o
    -- servidor iniciar a troca (SEMPRE validando tudo no servidor).
end
