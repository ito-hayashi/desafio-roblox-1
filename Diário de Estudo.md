Diário de Estudo - Roblox e Metaverso

Link para jogar [O Chão é Lava!](https://www.roblox.com/pt/games/99151708926186/Obby)
--------------------------------------------------------------------------------------
  -- 04/01/2026 - O Começo ✍️
Primeiros passos no Roblox Studio. Entendendo como colocar blocos e onde enfiar os scripts.

	print ("Hello, World")

  -- 06/01/2026 - Fazendo a Lava 💀
Aprendi a matar o jogador ou tirar vida.
O tal do debouncing serve para o script não rodar mil vezes por segundo e bugar tudo.
          
          local parent:Part = script.Parent
          local debouncing = {}

          parent.Touched:Connect(function(hit)
          	local playerName = hit.Parent.Name
          	if debouncing [playerName] then
      	    	return
	          end
	
	          -- Inicia o debouncing
	          debouncing[playerName] = true
	
	          local character:Model = hit.Parent
	          if not character then
	          	return
	          end
	
          	local humanoid:Humanoid = character:FindFirstChild("Humanoid")
          	if not humanoid then
	          	return
          	end	
	
	          -- Mata o jogador
          	humanoid.Health = 0
	
          	-- Remove o debouncing após 1 segundo
          	task.delay(0.1, function()
	          	debouncing[playerName] = nil
	          end)
          end)


  -- 06/01/2026 (Noite) - Piso de Sorteio 🎰
Fiz um sistema que escolhe se o piso é bom ou ruim na hora que você pisa.
Usei math.random(1, 100) para definir as chances (70% de ser azul e ficar de boa).
Coloquei uma checagem de distância (Magnitude) pro piso não resetar enquanto eu ainda estou em cima dele.
          
         local parte = script.Parent
	local ocupado = false

	parte.Touched:Connect(function(hit)
	local char = hit.Parent
	local hum = char:FindFirstChild("Humanoid")
	local root = char:FindFirstChild("HumanoidRootPart")

	-- Só inicia se for um jogador vivo e o piso não estiver ocupado
	if hum and root and hum.Health > 0 and not ocupado then
		ocupado = true 

		-- 1. SORTEIO (1/1 sempre dará azul para teste)
		local chance = math.random(1, 1)
		if chance <= 1 then
			parte.Color = Color3.fromRGB(100, 100, 255) -- AZUL
			parte.Transparency = 0.2

			-- LÓGICA DE CURA: Soma 50 de vida, limitando ao máximo de 100
			hum.Health = math.min(hum.Health + 50, 100)
		else
			parte.Color = Color3.fromRGB(255, 100, 100) -- VERMELHO
			parte.Transparency = 0.2
			hum.Health = hum.Health - 50
		end

		-- 2. LOOP DE PERMANÊNCIA
		task.spawn(function()
			while ocupado do
				task.wait(0.1)
				-- Verifica se o objeto ainda existe antes de calcular distância
				if not root or not root.Parent then break end

				local distancia = (root.Position - parte.Position).Magnitude

				if distancia > (parte.Size.Magnitude / 1.5) or hum.Health <= 0 then
					parte.Color = Color3.fromRGB(100, 100, 100)
					parte.Transparency = 0.2
					ocupado = false 
				end
			end
		end)
	end
	end)

  -- 07/01/2026 - Checkpoints 📍
Fiz o piso virar um ponto de nascimento.
O truque foi trocar a Part comum por um SpawnLocation e mudar a cor pra verde quando o jogador encosta, pra avisar que salvou.

          local spawnPoint = script.Parent

          spawnPoint.Touched:Connect(function(hit)
          	local char = hit.Parent
          	local player = game.Players:GetPlayerFromCharacter(char)

          	-- A MUDANÇA ESTÁ AQUI: 
          	-- Só roda se for um player e se a cor NÃO for o verde de ativado
          	if player and spawnPoint.Color ~= Color3.fromRGB(100, 255, 100) then

	          	-- 1. Mudança Visual
	          	spawnPoint.Color = Color3.fromRGB(100, 255, 100)
	          	spawnPoint.Material = Enum.Material.Neon
	          	spawnPoint.Transparency = 0.5

          		-- 2. Define o local de nascimento
	          	player.RespawnLocation = spawnPoint

	          	print("Checkpoint salvo: " .. spawnPoint.Name)
          	end
          end)

  -- 08/01/2026 - Linha de Chegada 🏁
O maior desafio: fazer o jogador ganhar, voltar pro começo e o mapa "esquecer" os checkpoints.
O script agora varre o mapa todo procurando os blocos chamados "Checkpoint" e pinta eles de cinza de novo.
Coloquei um tempo de 5 segundos para o piso amarelo voltar ao normal depois da vitória.
-edit: Agora ele não só reseta o mapa, como também conversa com o Placar de Líderes para dar o prêmio ao vencedor.

          local linhaDeChegada = script.Parent

	linhaDeChegada.Touched:Connect(function(hit)
    local char = hit.Parent
    local player = game.Players:GetPlayerFromCharacter(char)
    local hum = char:FindFirstChild("Humanoid")

    -- Só roda se for um player vivo
    if player and hum and hum.Health > 0 then

        -- 1. 🏆 DAR O PONTO NO PLACAR
        local stats = player:FindFirstChild("leaderstats")
        if stats then
            local score = stats:FindFirstChild("Triunfos")
            if score then
                score.Value = score.Value + 1 -- Soma uma vitória
            end
        end

        -- 2. 📍 VOLTAR PARA O INÍCIO
        local spawnOriginal = nil
        for _, obj in pairs(game.Workspace:GetDescendants()) do
            if obj:IsA("SpawnLocation") and obj.Neutral == true then
                spawnOriginal = obj
                break
            end
        end

        if spawnOriginal then
            player.RespawnLocation = spawnOriginal
            char:MoveTo(spawnOriginal.Position + Vector3.new(0, 3, 0))

            -- 3. 🧹 RESETAR CORES DOS CHECKPOINTS
            for _, checkpoint in pairs(game.Workspace:GetDescendants()) do
                if checkpoint.Name == "Checkpoint" and checkpoint:IsA("SpawnLocation") then
                    checkpoint.Color = Color3.fromRGB(100, 100, 100) -- Cinza
                    checkpoint.Material = Enum.Material.Plastic
                    checkpoint.Transparency = 0.2
                end
            end

            -- 4. ✨ EFEITO VISUAL DE VITÓRIA (5 Segundos)
            linhaDeChegada.Color = Color3.fromRGB(255, 255, 100)
            linhaDeChegada.Material = Enum.Material.Neon
            task.wait(5)
            linhaDeChegada.Color = Color3.fromRGB(100, 100, 100) -- Volta ao normal
            linhaDeChegada.Material = Enum.Material.Plastic
        end
    end
	end)

		   
-- 08/01/2026 - 📊 5. Placar de Líderes (Leaderboard)
Este script fica no ServerScriptService. Ele cria o placar que todos os jogadores veem no canto da tela e prepara a variável "Triunfos" para salvar as vitórias.

	local Players = game:GetService("Players")

	local function onPlayerAdded(player)
	-- Cria a pasta que o Roblox reconhece como Placar
	local leaderstats = Instance.new("Folder")
	leaderstats.Name = "leaderstats"
	leaderstats.Parent = player

	-- Cria o valor de Triunfos começando em 0
	local score = Instance.new("IntValue")
	score.Name = "Triunfos"
	score.Value = 0
	score.Parent = leaderstats
	end

	-- Avisa o jogo para rodar a função toda vez que alguém entrar
	Players.PlayerAdded:Connect(onPlayerAdded)
