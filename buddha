-- ============================================================
-- 大佛（Buddha）体型拉满 + 速度強化 v2
-- ・安全にHumanoidを取得（リスポーン対応）
-- ・速度・ジャンプ力を定期的に再適用（Blox Fruitsの上書き対策）
-- ・自分のBuddha変身モデルを検出して巨大化
-- ・GUIでON/OFF（ドラッグ移動可能）
-- ※クライアント側処理のため、自分の画面でのみ反映されます
-- ============================================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local lp = Players.LocalPlayer

-- ===== 設定 =====
local WALK_SPEED = 35     -- 移動速度
local JUMP_POWER = 80     -- ジャンプ力
local BUDDHA_SCALE = 3    -- Buddhaモデルの巨大化倍率（3 = 3倍）
local ON = true           -- 初期ON/OFF
-- ==================

-- キャラ・Humanoidを安全に取得する関数
local function getCharAndHum()
    local char = lp.Character
    if not char then return nil, nil end
    local hum = char:FindFirstChildOfClass("Humanoid")
    return char, hum
end

-- 自分のBuddhaモデルを探す（キャラ配下のモデル名で判定）
local function findBuddhaModel(char)
    for _, obj in pairs(char:GetChildren()) do
        if obj:IsA("Model") and (obj.Name:find("Buddha") or obj.Name:find("Daibutsu")) then
            return obj
        end
    end
    return nil
end

-- Buddhaモデルを巨大化（1回だけ。毎フレームやらない）
local function enlargeBuddha(char)
    local model = findBuddhaModel(char)
    if not model then return end
    local humanoid = model:FindFirstChildOfClass("Humanoid")
    if humanoid then
        -- BodyHeightScale等のNumberValueがあれば引き上げる
        for _, v in pairs(humanoid:GetChildren()) do
            if v:IsA("NumberValue") and v.Name:lower():find("scale") then
                v.Value = BUDDHA_SCALE
            end
        end
    end
    for _, p in pairs(model:GetDescendants()) do
        if p:IsA("BasePart") and not p:GetAttribute("BuddhaScaled") then
            p:SetAttribute("BuddhaScaled", true)
            p.Size = p.Size * BUDDHA_SCALE
            p.CanCollide = false
        end
    end
end

-- メインループ
local lastBuddhaCheck = 0
RunService.Heartbeat:Connect(function()
    if not ON then return end
    local char, hum = getCharAndHum()
    if not char or not hum then return end

    -- 1) 速度系の再適用（別スレッドにしないと重い）
    if hum.WalkSpeed ~= WALK_SPEED then
        hum.WalkSpeed = WALK_SPEED
    end
    if hum.JumpPower ~= JUMP_POWER then
        hum.JumpPower = JUMP_POWER
    end

    -- 2) Buddha巨大化は0.5秒に1回だけチェック（軽量化）
    if tick() - lastBuddhaCheck > 0.5 then
        lastBuddhaCheck = tick()
        pcall(enlargeBuddha, char)
    end
end)

-- ================= GUI（ドラッグ移動対応） =================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "BuddhaGui"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game:GetService("CoreGui")

local Main = Instance.new("Frame")
Main.Size = UDim2.new(0, 160, 0, 75)
Main.Position = UDim2.new(0, 20, 0.35, 0)
Main.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Main.BorderSizePixel = 0
Main.Active = true
Main.Parent = ScreenGui
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 8)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 28)
Title.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Title.Text = " Buddha Max（ドラッグ可）"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 12
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = Main
Instance.new("UICorner", Title).CornerRadius = UDim.new(0, 8)

-- ドラッグ処理
do
    local dragging = false
    local dragStart, startPos
    Title.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = Main.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
            or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            Main.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + delta.X,
                startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

-- ON/OFFボタン
local Toggle = Instance.new("TextButton")
Toggle.Size = UDim2.new(1, -16, 0, 32)
Toggle.Position = UDim2.new(0, 8, 0, 36)
Toggle.BackgroundColor3 = ON and Color3.fromRGB(0, 140, 60) or Color3.fromRGB(140, 40, 40)
Toggle.Text = ON and "ON" or "OFF"
Toggle.TextColor3 = Color3.new(1, 1, 1)
Toggle.Font = Enum.Font.GothamBold
Toggle.TextSize = 14
Toggle.Parent = Main
Instance.new("UICorner", Toggle).CornerRadius = UDim.new(0, 6)

Toggle.MouseButton1Click:Connect(function()
    ON = not ON
    Toggle.Text = ON and "ON" or "OFF"
    Toggle.BackgroundColor3 = ON and Color3.fromRGB(0, 140, 60) or Color3.fromRGB(140, 40, 40)
    if not ON then
        -- OFF時は速度をデフォルトに戻す
        local _, hum = getCharAndHum()
        if hum then
            hum.WalkSpeed = 16
            hum.JumpPower = 50
        end
    end
end)
