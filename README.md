# mini-middleware

Serviço do Windows que expõe dispositivos locais (câmeras / controles de acesso)
para a plataforma Condlink através de túneis [Cloudflare](https://www.cloudflare.com/).

Roda na máquina que tem acesso à rede local dos dispositivos (tipicamente o PC da
portaria) e mantém, para cada dispositivo, um túnel HTTPS público que é registrado
automaticamente no Condlink.

> Código-fonte: [Condlinkdev/dev-backend → setup-tunel](https://github.com/Condlinkdev/dev-backend/tree/main/setup-tunel)

---

## Como funciona (visão técnica)

1. O serviço lê a configuração (terminais e login) do **Registro do Windows**.
2. Para cada terminal, executa `cloudflared tunnel --url http://<IP>:80`.
3. Captura a URL pública gerada (`https://*.trycloudflare.com`) na saída do `cloudflared`.
4. Faz login na API do Condlink e registra a URL do túnel para aquele `devId`
   (chamada `PUT` em `admin-sinc-terminal-placa`).
5. Mantém os túneis vivos enquanto o serviço estiver em execução.

- **Linguagem:** Go (somente Windows).
- **Dependência empacotada:** `cloudflared.exe` — vem **dentro do instalador**, então
  não há download externo na primeira execução (funciona offline / atrás de firewall).
- **Execução:** Serviço do Windows (`CondlinkMiddleware`, início automático). Não há
  janela aberta; o serviço sobe junto com o Windows, sem precisar de login de usuário.

### Layout de arquivos (padrão Windows)

| Caminho | Conteúdo |
|---|---|
| `C:\Program Files\Condlink\MiniMiddleware\` | `miniMiddleware.exe` + `cloudflared.exe` (binários, somente leitura) |
| `C:\ProgramData\Condlink\MiniMiddleware\miniMiddleware.log` | Log do serviço (gravável) |
| `HKLM\SYSTEM\CurrentControlSet\Services\CondlinkMiddleware\Parameters` | Configuração (ver abaixo) |

### Configuração (propriedade do serviço, no Registro)

A configuração **não** fica mais em arquivos `.json` — ela é uma propriedade do
próprio serviço, salva no Registro e editável pela tela de configuração:

```
HKLM\SYSTEM\CurrentControlSet\Services\CondlinkMiddleware\Parameters
    Terminais  (REG_MULTI_SZ)   uma linha "ip=devId" por dispositivo
    Username   (REG_SZ)         usuário do Condlink
    Password   (REG_SZ)         senha do Condlink
```

> Ao desinstalar o serviço, essa configuração é removida automaticamente.
> Instalações antigas que tinham `C:\admin-condlink\terminal.json` e `login.json`
> são **migradas automaticamente** na primeira execução.

---

## Instalação

### 1. Cadastrar os terminais no Condlink

1. Acesse [admin-condlink.vercel.app](https://admin-condlink.vercel.app).
2. Vá em **Instalação → Terminais → Gerenciar Terminais → Cadastrar Terminais**.
3. Selecione o Fabricante e Modelo, preencha os dados obrigatórios (Nome, **devId**,
   usuário, senha, status e Rota de Acesso) e salve. Anote o **IP** de cada
   dispositivo e o **devId** que você cadastrou.

### 2. Rodar o instalador

1. Copie `CondlinkMiniMiddlewareSetup_v1.0.0.exe` para a máquina da portaria.
2. Execute-o (**clique direito → Executar como administrador**) e aceite o UAC.
3. O instalador automaticamente:
   - instala o serviço `CondlinkMiddleware`;
   - abre a **tela de configuração**;
   - inicia o serviço ao final.
4. Na tela de configuração, informe:
   - **Terminais:** digite o IP e o devId e clique em *Adicionar* (repita para cada um);
   - **Usuário** e **Senha** do Condlink;
   - clique em **Salvar**.

Pronto — o serviço inicia e os túneis sobem. Não é preciso configurar Agendador de
Tarefas nem criar arquivos manualmente.

### 3. Reconfigurar depois (sem reinstalar)

- Menu Iniciar → **"Configurar Condlink Mini-Middleware"**, ou
- execute:

  ```powershell
  & 'C:\Program Files\Condlink\MiniMiddleware\miniMiddleware.exe' config
  ```

Ao salvar, a tela oferece reiniciar o serviço para aplicar as alterações.

---

## Operação e diagnóstico

Verificar o estado do serviço:

```powershell
Get-Service CondlinkMiddleware
```

Ver o log (últimas linhas):

```powershell
Get-Content 'C:\ProgramData\Condlink\MiniMiddleware\miniMiddleware.log' -Tail 30
```

Conferir a configuração gravada no Registro:

```powershell
Get-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Services\CondlinkMiddleware\Parameters'
```

### Subcomandos da linha de comando

Executando `miniMiddleware.exe` diretamente (requer administrador):

| Comando | Ação |
|---|---|
| `miniMiddleware.exe install` | Instala o serviço |
| `miniMiddleware.exe remove` | Remove o serviço (e sua configuração no Registro) |
| `miniMiddleware.exe start` | Inicia o serviço |
| `miniMiddleware.exe stop` | Para o serviço |
| `miniMiddleware.exe config` | Abre a tela de configuração |

---

## Desinstalação

Configurações do Windows → **Aplicativos** → *Condlink Mini-Middleware* → **Desinstalar**.
Isso para e remove o serviço e apaga a configuração do Registro.

---

## Compilação (para desenvolvedores)

No diretório do código-fonte (`setup-tunel`), com **Go 1.22+** instalado:

```powershell
.\build.ps1
```

O script baixa dependências, embute as propriedades/manifesto (`go-winres`),
compila o `miniMiddleware.exe`, garante o `cloudflared.exe` e, se o
[Inno Setup 6](https://jrsoftware.org/isinfo.php) estiver instalado, gera o
instalador em `output\CondlinkMiniMiddlewareSetup_v1.0.0.exe`.
