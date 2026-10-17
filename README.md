-- Carrega a biblioteca Rayfield
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Variável de controle do auto-farm
local farming = false

-- Função para coletar os doces
local function startFarming()
    task.spawn(function()
        while farming do
            local player = game.Players.LocalPlayer
            local character = player.Character or player.CharacterAdded:Wait()
            local hrp = character:FindFirstChild("HumanoidRootPart")

            if hrp then
                -- Procura a pasta Candies no Workspace
                local candiesFolder = workspace:FindFirstChild("Candies")
                if candiesFolder then
                    for _, candy in pairs(candiesFolder:GetChildren()) do
                        if not farming then break end
                        
                        -- Encontra a parte do doce
                        local targetPart = candy:IsA("BasePart") and candy or candy:FindFirstChildWhichIsA("BasePart")
                        if targetPart then
                            -- Teleporta para o doce
                            hrp.CFrame = targetPart.CFrame
                            
                            -- Aumentado para 0.6 segundos (ajuste se quiser mais lento)
                            task.wait(0.6)
                        end
                    end
                end
            end
            task.wait(0.5)
        end
    end)
end

-- Criação da Janela Principal
local Window = Rayfield:CreateWindow({
   Name = "Candy Auto-Farm",
   LoadingTitle = "Carregando Script...",
   LoadingSubtitle = "por Carlão",
   ConfigurationSaving = { Enabled = false },
   KeySystem = false
})

-- Criando a Aba Principal
local FarmTab = Window:CreateTab("Auto Farm", 4483362458)

-- Botão / Toggle para Iniciar/Parar
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

-- Botão para Fechar o Menu
FarmTab:CreateButton({
   Name = "Fechar Menu",
   Callback = function()
      farming = false -- Desativa a coleta ao fechar
      Rayfield:Destroy()
   end,
})
