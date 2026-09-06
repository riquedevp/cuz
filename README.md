local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- IDs dos ícones (no formato rbxassetid://ID)
local ICON_TRADE = "rbxassetid://80446720359667" -- ícone ao lado de "Enviar trade"
local ICON_COIN = "rbxassetid://0000000000" -- troque pelo ID do ícone da moeda quando mandar

--============================================================
-- ScreenGui base
--============================================================
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "TradeUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

local UserInputService = game:GetService("UserInputService")

--============================================================
-- Função genérica para tornar um Frame/TextButton arrastável
--============================================================
local function makeDraggable(guiObject, dragHandle)
    dragHandle = dragHandle or guiObject
    local dragging = false
    local dragStart, startPos
    local moved = false

    dragHandle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            moved = false
            dragStart = input.Position
            startPos = guiObject.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    dragHandle.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            if delta.Magnitude > 3 then
                moved = true
            end
            guiObject.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + delta.X,
                startPos.Y.Scale, startPos.Y.Offset + delta.Y
            )
        end
    end)

    -- retorna uma função pra saber se o último input foi um "arraste" (pra não disparar clique junto)
    return function()
        return moved
    end
end

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

local sendButtonWasMoved = makeDraggable(sendButton)

--============================================================
-- Painel de busca (aparece ao clicar em "Enviar")
--============================================================
local panel = Instance.new("Frame")
panel.Name = "SearchPanel"
panel.Size = UDim2.new(0, 340, 0, 460)
panel.Position = UDim2.new(0.5, -170, 0.5, -230)
panel.BackgroundColor3 = Color3.fromRGB(30, 31, 36)
panel.BorderSizePixel = 0
panel.Visible = false
panel.Parent = screenGui
Instance.new("UICorner", panel).CornerRadius = UDim.new(0, 12)

local panelShadowStroke = Instance.new("UIStroke")
panelShadowStroke.Color = Color3.fromRGB(55, 56, 62)
panelShadowStroke.Thickness = 1
panelShadowStroke.Parent = panel

--============================================================
-- Header: ícone + "Enviar trade" | ícone moeda + valor | X
--============================================================
local header = Instance.new("Frame")
header.Name = "Header"
header.Size = UDim2.new(1, 0, 0, 48)
header.Position = UDim2.new(0, 0, 0, 0)
header.BackgroundTransparency = 1
header.BorderSizePixel = 0
header.Parent = panel

makeDraggable(panel, header)

local headerIcon = Instance.new("ImageLabel")
headerIcon.Size = UDim2.new(0, 20, 0, 20)
headerIcon.Position = UDim2.new(0, 16, 0.5, -10)
headerIcon.BackgroundTransparency = 1
headerIcon.Image = ICON_TRADE
headerIcon.ImageColor3 = Color3.fromRGB(255, 255, 255)
headerIcon.Parent = header

local headerTitle = Instance.new("TextLabel")
headerTitle.Size = UDim2.new(0, 160, 1, 0)
headerTitle.Position = UDim2.new(0, 44, 0, 0)
headerTitle.BackgroundTransparency = 1
headerTitle.Text = "Enviar trade"
headerTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
headerTitle.Font = Enum.Font.GothamBold
headerTitle.TextSize = 17
headerTitle.TextXAlignment = Enum.TextXAlignment.Left
headerTitle.Parent = header

-- Botão de fechar (X)
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 24, 0, 24)
closeBtn.Position = UDim2.new(1, -34, 0.5, -12)
closeBtn.BackgroundTransparency = 1
closeBtn.Text = "X"
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 16
closeBtn.TextColor3 = Color3.fromRGB(180, 180, 185)
closeBtn.ZIndex = 3
closeBtn.BorderSizePixel = 0
closeBtn.Parent = header

-- Botão com ícone de moeda + quantidade (clicável para editar)
local coinButton = Instance.new("TextButton")
coinButton.Size = UDim2.new(0, 76, 0, 26)
coinButton.Position = UDim2.new(1, -118, 0.5, -13)
coinButton.BackgroundColor3 = Color3.fromRGB(45, 46, 52)
coinButton.AutoButtonColor = false
coinButton.Text = ""
coinButton.BorderSizePixel = 0
coinButton.Parent = header
Instance.new("UICorner", coinButton).CornerRadius = UDim.new(0, 8)

local coinIcon = Instance.new("ImageLabel")
coinIcon.Size = UDim2.new(0, 16, 0, 16)
coinIcon.Position = UDim2.new(0, 8, 0.5, -8)
coinIcon.BackgroundTransparency = 1
coinIcon.Image = ICON_COIN
coinIcon.Parent = coinButton

-- Quantidade de moedas: TextBox para o jogador poder editar/alterar o valor
local coinAmount = Instance.new("TextBox")
coinAmount.Size = UDim2.new(1, -30, 1, 0)
coinAmount.Position = UDim2.new(0, 28, 0, 0)
coinAmount.BackgroundTransparency = 1
coinAmount.Text = "10.169"
coinAmount.TextColor3 = Color3.fromRGB(255, 255, 255)
coinAmount.Font = Enum.Font.GothamBold
coinAmount.TextSize = 14
coinAmount.TextXAlignment = Enum.TextXAlignment.Left
coinAmount.ClearTextOnFocus = false
coinAmount.Parent = coinButton

coinButton.MouseEnter:Connect(function()
    coinButton.BackgroundColor3 = Color3.fromRGB(55, 56, 63)
end)
coinButton.MouseLeave:Connect(function()
    coinButton.BackgroundColor3 = Color3.fromRGB(45, 46, 52)
end)
coinButton.MouseButton1Click:Connect(function()
    coinAmount:CaptureFocus()
end)

-- Caixa de busca
local searchBox = Instance.new("TextBox")
searchBox.Size = UDim2.new(1, -32, 0, 40)
searchBox.Position = UDim2.new(0, 16, 0, 60)
searchBox.BackgroundColor3 = Color3.fromRGB(42, 43, 49)
searchBox.PlaceholderText = "Busca por nome de usuário"
searchBox.PlaceholderColor3 = Color3.fromRGB(140, 140, 145)
searchBox.Text = ""
searchBox.TextColor3 = Color3.fromRGB(230, 230, 230)
searchBox.Font = Enum.Font.Gotham
searchBox.TextSize = 14
searchBox.TextXAlignment = Enum.TextXAlignment.Left
searchBox.ClearTextOnFocus = false
searchBox.BorderSizePixel = 0
searchBox.Parent = panel

local searchPadding = Instance.new("UIPadding")
searchPadding.PaddingLeft = UDim.new(0, 10)
searchPadding.Parent = searchBox

local searchCorner = Instance.new("UICorner")
searchCorner.CornerRadius = UDim.new(0, 8)
searchCorner.Parent = searchBox

local searchStroke = Instance.new("UIStroke")
searchStroke.Color = Color3.fromRGB(60, 61, 68)
searchStroke.Thickness = 1
searchStroke.Parent = searchBox

-- Mensagem de status da busca (só aparece durante/depois de uma busca)
local friendsTitle = Instance.new("TextLabel")
friendsTitle.Size = UDim2.new(1, -32, 0, 20)
friendsTitle.Position = UDim2.new(0, 16, 0, 108)
friendsTitle.BackgroundTransparency = 1
friendsTitle.Text = ""
friendsTitle.Visible = false
friendsTitle.TextColor3 = Color3.fromRGB(150, 150, 155)
friendsTitle.Font = Enum.Font.Gotham
friendsTitle.TextSize = 13
friendsTitle.TextXAlignment = Enum.TextXAlignment.Left
friendsTitle.Parent = panel

-- Lista rolável de amigos
local listFrame = Instance.new("ScrollingFrame")
listFrame.Size = UDim2.new(1, -16, 1, -124)
listFrame.Position = UDim2.new(0, 8, 0, 116)
listFrame.BackgroundTransparency = 1
listFrame.BorderSizePixel = 0
listFrame.ScrollBarThickness = 4
listFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
listFrame.Parent = panel

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 2)
listLayout.Parent = listFrame

--============================================================
-- Carrega a lista de amigos reais do jogador (API do Roblox)
--============================================================
local allFriends = {} -- { {Name=..., UserId=..., Thumb=...}, ... }

local function loadFriends()
    local success, friendPages = pcall(function()
        return Players:GetFriendsAsync(player.UserId)
    end)

    if not success or not friendPages then
        return
    end

    local page = friendPages:GetCurrentPage()
    for _, friendData in ipairs(page) do
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
            table.insert(allFriends, {
                Name = friendData.Username,
                UserId = friendData.Id,
            })
        end
    end
end

--============================================================
-- Renderiza a lista (com filtro opcional de busca)
--============================================================
local onFriendSelected -- callback definido mais abaixo

local function clearList()
    for _, child in ipairs(listFrame:GetChildren()) do
        if child:IsA("TextButton") or (child:IsA("TextLabel") and child.Name == "SectionLabel") then
            child:Destroy()
        end
    end
end

local function createSectionLabel(text)
    local lbl = Instance.new("TextLabel")
    lbl.Name = "SectionLabel"
    lbl.Size = UDim2.new(1, 0, 0, 22)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.TextColor3 = Color3.fromRGB(20, 20, 20)
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 14
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = listFrame
    return lbl
end

local function createFriendRow(friendInfo)
    local row = Instance.new("TextButton")
    row.Size = UDim2.new(1, 0, 0, 44)
    row.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    row.BorderSizePixel = 0
    row.AutoButtonColor = false
    row.Text = ""
    row.Parent = listFrame

    row.MouseEnter:Connect(function()
        row.BackgroundColor3 = Color3.fromRGB(247, 247, 250)
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

local function renderFriendsList()
    clearList()
    local rows = 0

    createSectionLabel("Minhas amizades (" .. #allFriends .. ")")
    rows += 1
    for _, friendInfo in ipairs(allFriends) do
        createFriendRow(friendInfo)
        rows += 1
    end

    listFrame.CanvasSize = UDim2.new(0, 0, 0, rows * 45)
end

-- Busca um usuário real do Roblox pelo nome EXATO digitado
local function searchUserByName(name)
    clearList()
    friendsTitle.Visible = true
    friendsTitle.Text = "Buscando..."

    local userId
    local ok = pcall(function()
        userId = Players:GetUserIdFromNameAsync(name)
    end)

    if not ok or not userId then
        friendsTitle.Text = "Nenhum usuário encontrado"
        listFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
        return
    end

    friendsTitle.Visible = false
    createFriendRow({ Name = name, UserId = userId })
    listFrame.CanvasSize = UDim2.new(0, 0, 0, 44)
end

-- Debounce: espera parar de digitar antes de buscar (evita spam de requests)
local searchToken = 0
searchBox:GetPropertyChangedSignal("Text"):Connect(function()
    local text = searchBox.Text

    if text == "" then
        friendsTitle.Visible = false
        renderFriendsList()
        return
    end

    searchToken += 1
    local myToken = searchToken
    task.wait(0.4) -- espera meio segundo de pausa na digitação
    if myToken ~= searchToken then return end -- usuário continuou digitando, cancela essa busca antiga
    if searchBox.Text ~= text then return end

    searchUserByName(text)
end)

--============================================================
-- Abrir / Fechar painel
--============================================================
sendButton.MouseButton1Click:Connect(function()
    if sendButtonWasMoved() then return end -- não abre o painel se o clique foi na verdade um arraste
    panel.Visible = true
    searchBox.Text = ""
    if #allFriends == 0 then
        loadFriends()
    end
    renderFriendsList()
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

    searchBox.Text = ""
    friendsTitle.Visible = false
    renderFriendsList()

    panel.Visible = false

    -- Exemplo: aqui você chamaria seu sistema real de trade,
    -- por exemplo abrindo uma segunda tela com os itens do jogador
    -- e do amigo selecionado, ou disparando um RemoteEvent para o
    -- servidor iniciar a troca (SEMPRE validando tudo no servidor).
end
