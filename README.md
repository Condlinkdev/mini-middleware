# mini-middleware

Veja também: [Configuração de Túnel](https://github.com/Condlinkdev/dev-backend/tree/main/setup-tunel)

Para usar esse sistema, é necessário instalar o executável na máquina que tem acesso local às câmeras.

## Inicialização automática:
1. Abra o **Agendador de Tarefas** do Windows.
2. Clique em **Criar tarefa** no painel lateral direito.
3. Na aba **Geral**:
   - Nome: `Iniciar Middleware`
   - Marque: `Executar com privilégios mais altos`
4. Na aba **Disparadores**:
   - Clique em Novo e selecione `Ao fazer logon`.
   - Selecione `Qualquer usuário` e clique em Ok.
   - Adicione outro: selecione `Ao iniciar`.
5. Na aba **Ações**:
   - Tipo: `Iniciar um programa`.
   - Clique em procurar e selecione o arquivo `.exe` instalado.
6. Na aba **Configurações**:
   - Habilite: `Executar a tarefa o quanto antes após um início agendado ser perdido`.
7. Clique em OK e verifique na página inicial se a tarefa foi criada.

---

## Configurar o Hardware
Para comunicar com o Mini-Middleware:

1. Acesse: [admin-condlink.vercel.app](https://admin-condlink.vercel.app)
2. Vá em **Instalação > Terminais > Gerenciar Terminais > Cadastrar Terminais**.
3. Selecione o Fabricante e Modelo específico.
4. Insira os dados obrigatórios (Nome, devId, usuário, senha, status e Rota de Acesso) e salve.
5. Crie um arquivo chamado `terminal.json` no bloco de notas com a estrutura:
   ```json
   {
     "ip_do_dispositivo_1": "dev_id_cadastrado_1",
     "ip_do_dispositivo_2": "dev_id_cadastrado_2"
   }
Salve o arquivo e coloque ele na pasta do admin-condlink caso ela não exista crie a pasta com o nome "admin-condlink" localizada na raiz do computador.
6. Crie outro arquivo chamado 'login.json' no bloco de notas com a estrutura:
	{
		"username": "admin.XXXX",
		"password": "XXXXXXXX"
	}
Salve o arquivo e coloque ele na pasta do admin-condlink, localizada na raiz do computador, criada anteriormente para o arquivo "terminal.json".

Inicialização manual:
- Pressione as teclas Windows + X e selecione "Terminal" para abrir o Terminal.
   - digite o comando ".\miniMiddleware" ou "miniMidleware.exe" para executar o programa.
			- Na pasta C:\Users\Portaria\miniMiddleware.exe
     		- Se der erro, tente em outra pasta
			- Copiar para outra pasta, por exemplo:
			- C:\Users\Public\Downloads\miniMiddleware.exe
			- e executar com o comando:
			```miniMiddleware.exe```
			```.\miniMiddleware.exe ```
			O progrma vai exibir um mensagem como essa:
			<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/416f0dbf-e56d-4afe-9671-411ebd4790f8" />

			- Ele vai ficar minimizado. 
			- Não pode fechar, caso feche o serviço vai fiacar parado.
			- Na barra tem que ficar essa informação que ficou minimizado.
