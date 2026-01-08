Diário de Estudo - Roblox e Metaverso

Link para jogar [O Chão é Lava!](https://www.roblox.com/pt/games/99151708926186/Obby)
--------------------------------------------------------------------------------------
  -- 04/01/2026 - O Começo ✍️
Primeiros passos no Roblox Studio. Entendendo como colocar blocos e onde enfiar os scripts.

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

          		-- 1. SORTEIO 50/50
	          	local chance = math.random(1, 100)
	          	if chance <= 70 then
		          	parte.Color = Color3.fromRGB(100, 100, 255) -- AZUL
		          	parte.Transparency = 0.2
	          	else
	          		parte.Color = Color3.fromRGB(255, 100, 100) -- VERMELHO
		          	parte.Transparency = 0.2
		          	hum.Health = hum.Health - 45
	          	end

          		-- 2. LOOP DE PERMANÊNCIA
	          	-- Ele fica checando se você ainda está perto da peça
	          	-- Enquanto a distância for curta, a cor não muda
	          	task.spawn(function()
		          	while ocupado do
		          		task.wait(0.1)
		          		local distancia = (root.Position - parte.Position).Magnitude

		          		-- Se o jogador se afastar mais que o tamanho da peça (pulou fora)
		            	-- Ou se o jogador morrer
	          			if distancia > (parte.Size.Magnitude / 1.5) or hum.Health <= 0 then
		          			parte.Color = Color3.fromRGB(100, 100, 100)
		          			parte.Transparency = 0.2
          					ocupado = false -- Libera para o próximo sorteio
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

          local linhaDeChegada = script.Parent

          linhaDeChegada.Touched:Connect(function(hit)
	          local char = hit.Parent
          	local player = game.Players:GetPlayerFromCharacter(char)
          	local hum = char:FindFirstChild("Humanoid")

          	-- Só roda se for um player vivo
          	if player and hum and hum.Health > 0 then

	          	-- 1. BUSCAR O SPAWN INICIAL (O que estiver marcado como Neutral)
	          	local spawnOriginal = nil
	          	for _, obj in pairs(game.Workspace:GetDescendants()) do
	          		if obj:IsA("SpawnLocation") and obj.Neutral == true then
          				spawnOriginal = obj
			          	break
	          		end
	          	end

	          	if spawnOriginal then
	          		-- 2. RESETAR O NASCIMENTO E TELEPORTAR
	          		player.RespawnLocation = spawnOriginal
		          	char:MoveTo(spawnOriginal.Position + Vector3.new(0, 3, 0))

	          		-- 3. RESETAR TODOS OS CHECKPOINTS VERDES DO MAPA
	          		-- Eles voltam a ser cinzas para poderem ser pegos de novo
	          		for _, checkpoint in pairs(game.Workspace:GetDescendants()) do
	          			if checkpoint.Name == "Checkpoint" and checkpoint:IsA("SpawnLocation") then
			          		checkpoint.Color = Color3.fromRGB(100, 100, 100) -- Cinza
			          		checkpoint.Material = Enum.Material.Plastic
			          		checkpoint.Transparency = 0.2
	          			end
	          		end

	          		-- 4. EFEITO VISUAL NA CHEGADA (Fica Amarela)
		          	linhaDeChegada.Color = Color3.fromRGB(255, 255, 100) -- Amarelo
		          	linhaDeChegada.Material = Enum.Material.Neon
	          		linhaDeChegada.Transparency = 0.5 -- Fica bem visível

	          		-- 5. ESPERAR 5 SEGUNDOS E VOLTAR AO NORMAL
	          		task.wait(5)

          			linhaDeChegada.Color = Color3.fromRGB(100, 100, 100) -- Volta pra Cinza
	          		linhaDeChegada.Material = Enum.Material.Plastic
	          		linhaDeChegada.Transparency = 0.2

	          		print("Mapa resetado e linha de chegada pronta para o próximo!")
          		end
          	end
          end)
