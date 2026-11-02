# main.lua.
-- GAME DUMP MAX ULTRA
-- Delta / Roblox - somente leitura

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local MarketplaceService = game:GetService("MarketplaceService")
local HttpService = game:GetService("HttpService")

local player = Players.LocalPlayer
local output = {}

local function add(s)
    output[#output + 1] = tostring(s)
end

local function vec(v)
    return string.format(
        "X=%.2f Y=%.2f Z=%.2f",
        v.X, v.Y, v.Z
    )
end

local function getPath(obj)
    local parts = {}
    local current = obj

    while current and current ~= game do
        table.insert(parts, 1, current.Name)
        current = current.Parent
    end

    return table.concat(parts, ".")
end

add("========== GAME DUMP MAX ULTRA ==========")
add("Jogo: " .. game.Name)
add("PlaceId: " .. game.PlaceId)
add("JobId: " .. game.JobId)
add("Jogador: " .. player.Name)
add("UserId: " .. player.UserId)
add("========================================")

-- JOGADORES
add("\n========== JOGADORES ==========")

for _, p in ipairs(Players:GetPlayers()) do
    add("Nome: " .. p.Name)
    add("DisplayName: " .. p.DisplayName)
    add("UserId: " .. p.UserId)

    local char = p.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")

    if root then
        add("Posição: " .. vec(root.Position))
    end
end

-- OBJETOS E COORDENADAS
local function scan(root)
    for _, obj in ipairs(root:GetDescendants()) do
        local path = getPath(obj)

        add("\nTipo: " .. obj.ClassName)
        add("Nome: " .. obj.Name)
        add("Caminho: " .. path)

        if obj:IsA("BasePart") then
            add("Posição: " .. vec(obj.Position))
            add("Tamanho: " .. vec(obj.Size))
            add("Rotação: " .. vec(obj.Orientation))
        elseif obj:IsA("Model") then
            local ok, cf = pcall(function()
                return obj:GetPivot()
            end)

            if ok then
                add("Pivot: " .. vec(cf.Position))
            end
        elseif obj:IsA("ValueBase") then
            local ok, value = pcall(function()
                return obj.Value
            end)

            if ok and typeof(value) ~= "Instance" then
                add("Valor: " .. tostring(value))
            end
        end

        if obj:IsA("RemoteEvent")
        or obj:IsA("RemoteFunction")
        or obj:IsA("BindableEvent")
        or obj:IsA("BindableFunction") then
            add("TIPO DE COMUNICAÇÃO: " .. obj.ClassName)
        end
    end
end

add("\n========== WORKSPACE ==========")
scan(Workspace)

add("\n========== REPLICATED STORAGE ==========")
scan(ReplicatedStorage)

add("\n========== PLAYER GUI ==========")
pcall(function()
    scan(player:FindFirstChildOfClass("PlayerGui"))
end)

add("\n========== BACKPACK ==========")
pcall(function()
    scan(player:FindFirstChildOfClass("Backpack"))
end)

-- PERSONAGEM
add("\n========== PERSONAGEM ==========")
if player.Character then
    scan(player.Character)
end

-- GAME PASSES: identificação do jogo
add("\n========== GAME PASSES ==========")
add("PlaceId: " .. game.PlaceId)
add("Consulte os passes públicos associados ao jogo.")

add("\n========== FIM DO RELATÓRIO ==========")

local report = table.concat(output, "\n")

-- INTERFACE
local gui = Instance.new("ScreenGui")
gui.Name = "GameDumpMax"
gui.ResetOnSpawn = false

pcall(function()
    gui.Parent = game:GetService("CoreGui")
end)

if not gui.Parent then
    gui.Parent = player:WaitForChild("PlayerGui")
end

local open = Instance.new("TextButton")
open.Size = UDim2.fromOffset(48, 48)
open.Position = UDim2.new(0, 15, 0.4, 0)
open.Text = "DUMP"
open.TextColor3 = Color3.new(1,1,1)
open.BackgroundColor3 = Color3.fromRGB(20, 10, 35)
open.BorderSizePixel = 0
open.Parent = gui

local frame = Instance.new("Frame")
frame.Size = UDim2.fromOffset(245, 145)
frame.Position = UDim2.new(0, 70, 0.4, 0)
frame.BackgroundColor3 = Color3.fromRGB(15, 12, 25)
frame.BorderSizePixel = 0
frame.Visible = false
frame.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10)
corner.Parent = frame

local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(160, 60, 255)
stroke.Thickness = 2
stroke.Parent = frame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 38)
title.BackgroundTransparency = 1
title.Text = "GAME DUMP MAX"
title.TextColor3 = Color3.fromRGB(190, 110, 255)
title.TextSize = 17
title.Parent = frame

local copy = Instance.new("TextButton")
copy.Size = UDim2.new(1, -20, 0, 42)
copy.Position = UDim2.fromOffset(10, 55)
copy.Text = "COPIAR TUDO"
copy.TextColor3 = Color3.new(1,1,1)
copy.BackgroundColor3 = Color3.fromRGB(100, 35, 180)
copy.Parent = frame

local status = Instance.new("TextLabel")
status.Size = UDim2.new(1, -10, 0, 25)
status.Position = UDim2.fromOffset(5, 105)
status.BackgroundTransparency = 1
status.Text = "Objetos: " .. #output
status.TextColor3 = Color3.new(1,1,1)
status.TextSize = 12
status.Parent = frame

open.MouseButton1Click:Connect(function()
    frame.Visible = not frame.Visible
end)

copy.MouseButton1Click:Connect(function()
    if setclipboard then
        setclipboard(report)
        status.Text = "Relatório copiado!"
    else
        status.Text = "Clipboard não disponível."
    end
end)
