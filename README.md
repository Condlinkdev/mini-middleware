# mini-middleware

Veja também:
https://github.com/Condlinkdev/dev-backend/tree/main/setup-tunel

Para usar esse sistema tem que instalar o executável na máquina que tem acesso local às câmeras.

Inicialização automática:
			     - Abra o Agendador de Tarefas do Windows.
			
			     - Com o agendador aberto, clique em Criar tarefa no painel lateral direito.

			     - Ao clicar em Criar tarefa, na aba Geral coloque o nome de Iniciar Middleware, habilite o Executar com privilégios mais altos e selecione a aba Disparadores.
			
			     - Na aba de disparadores, selecione Ao fazer logon, coloque Qualquer usuário, deixe Habilitado e clique em Ok.
				
			     - Ainda na aba de disparadores vamos adicionar mais uma item, repita o mesmo passo a passo só troque a ação para Ao iniciar, depois de adicionar clique na aba Ações.

			     - Na aba de ações, vamos adicionar uma nova ação. Selecione o tipo da ação como Iniciar um programa, clique em procurar e ache o .exe que você instalou, ao selecionar verifique se o caminho está correto e clique em Ok.

			     - Clique na aba Configurações e habilite a opção Executar a tarefa o quanto antes após um início agendado ser perdido. E depois clique em Ok para finalizar o processo.
			
			     - Verifique se está tudo ok na página inicial.



Configurar o Hardware para se comunicar com o Mini-Middleware:
								- Acesse o admin-condlink.vercel.app
					
								- Acesse Instalação > Terminais > Gerenciar Terminais > Cadastrar Terminais > Selecione o Fabricante Referente e o Modelo Especifico > Insira os dados obrigatorios (nome do dispositivo, coloque um devId, digite o usuario do terminal, senha referido, status e Rota de Acesso) > Salve
								
								- Faça isso para todos os terminais em seguida vá no bloco de notas e crie um arquivo chamado terminal.json e utilize a seguinte estrutura para cada terminal cadastrado:
								
								{
									"ip do dispositivo": "dev id cadastrado no condlink",
									"ip do dispositivo": "dev id cadastrado no condlink"								
								}

								- Salve o arquivo e coloque ele na pasta do admin-condlink caso ela não exista crie a pasta com o nome "admin-condlink" localizada na raiz do computador.

								- Faça tambem um arquivo chamado login.json com a seguinte estrutura:

								{
									"username": "admin.XXXX",
									"password": "XXXXXXXX"
								}					

								- Salve o arquivo e coloque ele na pasta do admin-condlink caso ela não exista crie a pasta com o nome "admin-condlink" localizada na raiz do computador.


								
Inicialização manual:
				- Pressione as teclas Windows + X e selecione "Terminal" para abrir o Terminal.
				
				- digite o comando ".\miniMiddleware" para executar o programa.
				- Na pasta C:\Users\Portaria\miniMiddleware.exe
				- Copiar para outra pasta, por exemplo:
				- C:\Users\Public\Downloads\miniMiddleware.exe
				- e executar com o comando:
				```.\miniMiddleware.exe ```
				O progrma vai exibir um mensagem como essa:
				<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/416f0dbf-e56d-4afe-9671-411ebd4790f8" />

				- Ele vai ficar minimizado. 
				- Não pode fechar, caso feche o serviço vai fiacar parado.
				- Na barra tem que ficar essa informação
				
