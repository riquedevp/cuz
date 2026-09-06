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

-- Barra de arrastar no topo do painel
local dragBar = Instance.new("Frame")
dragBar.Name = "DragBar"
dragBar.Size = UDim2.new(1, 0, 0, 28)
dragBar.Position = UDim2.new(0, 0, 0, 0)
dragBar.BackgroundTransparency = 1
dragBar.Parent = panel

local dragDots = Instance.new("TextLabel")
dragDots.Size = UDim2.new(1, 0, 1, 0)
dragDots.BackgroundTransparency = 1
dragDots.Text = "⋯"
dragDots.TextColor3 = Color3.fromRGB(180, 180, 180)
dragDots.Font = Enum.Font.GothamBold
dragDots.TextSize = 20
dragDots.Rotation = 90
dragDots.Parent = dragBar

makeDraggable(panel, dragBar)

-- Caixa de busca
local searchBox = Instance.new("TextBox")
searchBox.Size = UDim2.new(1, -32, 0, 40)
searchBox.Position = UDim2.new(0, 16, 0, 32)
searchBox.BackgroundColor3 = Color3.fromRGB(245, 246, 250)
searchBox.PlaceholderText = "Busca por nome de usuário"
searchBox.PlaceholderColor3 = Color3.fromRGB(150, 150, 155)
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
searchStroke.Color = Color3.fromRGB(225, 226, 232)
searchStroke.Thickness = 1
searchStroke.Parent = searchBox

-- Mensagem de status da busca (só aparece durante/depois de uma busca)
local friendsTitle = Instance.new("TextLabel")
friendsTitle.Size = UDim2.new(1, -32, 0, 20)
friendsTitle.Position = UDim2.new(0, 16, 0, 80)
friendsTitle.BackgroundTransparency = 1
friendsTitle.Text = ""
friendsTitle.Visible = false
friendsTitle.TextColor3 = Color3.fromRGB(120, 120, 120)
friendsTitle.Font = Enum.Font.Gotham
friendsTitle.TextSize = 13
friendsTitle.TextXAlignment = Enum.TextXAlignment.Left
friendsTitle.Parent = panel

-- Lista rolável de amigos
local listFrame = Instance.new("ScrollingFrame")
listFrame.Size = UDim2.new(1, -16, 1, -96)
listFrame.Position = UDim2.new(0, 8, 0, 88)
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
closeBtn.Size = UDim2.new(0, 24, 0, 24)
closeBtn.Position = UDim2.new(1, -32, 0, 2)
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
local recentSelections = {} -- pessoas que você já clicou nesta sessão

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

-- Adiciona (ou move pro topo, se já existia) alguém na lista de recentes
local function addToRecents(friendInfo)
    for i, existing in ipairs(recentSelections) do
        if existing.UserId == friendInfo.UserId then
            table.remove(recentSelections, i)
            break
        end
    end
    table.insert(recentSelections, 1, friendInfo)
    if #recentSelections > 10 then
        table.remove(recentSelections, #recentSelections)
    end
end

local function renderFriendsList()
    clearList()
    local rows = 0

    if #recentSelections > 0 then
        createSectionLabel("Recentes")
        rows += 1
        for _, friendInfo in ipairs(recentSelections) do
            createFriendRow(friendInfo)
            rows += 1
        end
    end

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

    addToRecents(friendInfo)
    searchBox.Text = ""
    friendsTitle.Visible = false
    renderFriendsList()

    panel.Visible = false

    -- Exemplo: aqui você chamaria seu sistema real de trade,
    -- por exemplo abrindo uma segunda tela com os itens do jogador
    -- e do amigo selecionado, ou disparando um RemoteEvent para o
    -- servidor iniciar a troca (SEMPRE validando tudo no servidor).
end
