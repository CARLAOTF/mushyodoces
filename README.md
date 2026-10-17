-- Carrega a biblioteca Rayfield
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Variáveis de Controle
local farming = false
local antiAfkEnabled = true

-- Configuração da Posição do Teleporte (1m acima)
local offset = Vector3.new(0, 3.5, 0) 

---------------------------------------------------------
-- SISTEMA ANTI-AFK COMPLETO (PC + MOBILE)
---------------------------------------------------------
local env = (getgenv and getgenv()) or _G

if not env.AntiAFK_Loaded then 
    env.AntiAFK_Loaded = true

    local Players = game:GetService("Players")
    local VirtualInputManager = game:GetService("VirtualInputManager")
    local UserInputService = game:GetService("UserInputService")
    local VirtualUser = game:GetService("VirtualUser")

    local function antiAfkPulse()
        if not antiAfkEnabled then return end

        -- Método 1: Envio de KeyEvent neutro (PC)
        pcall(function()
            VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.Unknown, false, game)
            VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.Unknown, false, game)
        end)
        
        -- Método 2: Movimento do Mouse / Touch Virtual (PC & Mobile)
        pcall(function()
            local position = UserInputService:GetMouseLocation()
            VirtualInputManager:SendMouseMoveEvent(position.X, position.Y, game)
        end)

        -- Método 3: VirtualUser Click (Perfeito para Mobile / Celulares)
        pcall(function()
            VirtualUser:CaptureController()
            VirtualUser:ClickButton2(Vector2.new(0, 0))
        end)

        -- Método 4: Simulação do evento WindowFocused
        local fire = env.firesignal or env.fireSignal
        if type(fire) == "function" then
            pcall(fire, UserInputService.WindowFocused)
        end
    end

    -- Evento Nativo do Roblox quando entra em estado AFK
    Players.LocalPlayer.Idled:Connect(function()
        antiAfkPulse()
    end)

    -- Loop contínuo de suporte a cada 5 minutos (evita desconexões silenciosas)
    task.spawn(function()
        while true do
            task.wait(300) -- Roda a cada 5 minutos
            if antiAfkEnabled then
                antiAfkPulse()
                
                -- Método 5 (Pulo suave do Personagem)
                pcall(function()
                    local char = Players.LocalPlayer.Character
                    if char and char:FindFirstChildOfClass("Humanoid") then
                        char:FindFirstChildOfClass("Humanoid"):ChangeState(Enum.HumanoidStateType.Jumping)
                    end
                end)
            end
        end
    end)
end

---------------------------------------------------------
-- LÓGICA DE AUTO-FARM DOS CANDIES
---------------------------------------------------------
local function startFarming()
    task.spawn(function()
        while farming do
            local player = game.Players.LocalPlayer
            local character = player.Character or player.CharacterAdded:Wait()
            local hrp = character:FindFirstChild("HumanoidRootPart")

            if hrp then
                local candiesFolder = workspace:FindFirstChild("Candies")
                if candiesFolder then
                    for _, candy in pairs(candiesFolder:GetChildren()) do
                        if not farming then break end
                        
                        local targetPart = candy:IsA("BasePart") and candy or candy:FindFirstChildWhichIsA("BasePart")
                        if targetPart then
                            -- Teleporta 1m acima do doce
                            hrp.CFrame = CFrame.new(targetPart.Position + offset)
                            
                            -- Tempo de espera para registrar o contato
                            task.wait(0.6)
                        end
                    end
                end
            end
            task.wait(0.5)
        end
    end)
end

---------------------------------------------------------
-- INTERFACE GRÁFICA (RAYFIELD MENU)
---------------------------------------------------------
local Window = Rayfield:CreateWindow({
   Name = "Candy Auto-Farm + Anti-AFK",
   LoadingTitle = "Carregando Script...",
   LoadingSubtitle = "por Carlão",
   ConfigurationSaving = { Enabled = false },
   KeySystem = false
})

local FarmTab = Window:CreateTab("Auto Farm", 4483362458)

-- Toggle do Auto Farm
FarmTab:CreateToggle({
   Name = "Coletar Candies",
   CurrentValue = false,
   Flag = "CandyFarmToggle",
   Callback = function(Value)
      farming = Value
      if farming then
          startFarming()
      end
   end,
})

-- Toggle do Anti-AFK
FarmTab:CreateToggle({
   Name = "Anti-AFK Ativo",
   CurrentValue = true,
   Flag = "AntiAFKToggle",
   Callback = function(Value)
      antiAfkEnabled = Value
   end,
})

-- Botão para Fechar o Menu
FarmTab:CreateButton({
   Name = "Fechar Menu",
   Callback = function()
      farming = false
      Rayfield:Destroy()
   end,
})
