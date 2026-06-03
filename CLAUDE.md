# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repo distributes the compiled `miniMiddleware.exe` binary for Windows. The **source code** lives at `C:\code\condlink\go\dev-backend\setup-tunel\` (also mirrored at the GitHub URL in `fontes-setup`). Changes to behavior must be made in the source repo and the resulting `.exe` copied here.

## What This Application Does

`miniMiddleware.exe` is a Windows-only Go **service** that:
1. Reads the terminals (IP → devId) and Condlink login from the Windows Registry (service config — see below)
2. Runs `cloudflared.exe` bundled next to the executable (in Program Files); downloads it from GitHub only as a fallback if missing
3. Spawns one `cloudflared tunnel --url http://<IP>:80` goroutine per device
4. Parses the generated `*.trycloudflare.com` URL from cloudflared's stderr
5. Authenticates with the Condlink Cloud Functions API and registers each tunnel URL via PUT to `admin-sinc-terminal-placa`

## Source Layout

- `main.go` — entry point, Windows service (`svc`), service mgmt (install/remove/start/stop), tunnel logic, Condlink API calls
- `config.go` — reads/writes config in the Registry; migrates legacy `terminal.json`/`login.json` on first run
- `gui.go` — `lxn/walk` GUI window for editing config (the `config` subcommand)
- `winres/winres.json` — `go-winres` resource definition: `.exe` properties (version/company) **and** the app manifest (common-controls v6 for the GUI + `requireAdministrator` for UAC/Registry writes)
- `setup.iss` — Inno Setup installer script
- `build.ps1` — full build: deps → go-winres → go build → Inno Setup

## Building

```powershell
# Full build (binary + installer). Requires Go 1.22+, go-winres, and (optional) Inno Setup 6
.\build.ps1
```

Manual binary-only build:
```powershell
go install github.com/tc-hib/go-winres@latest
go-winres make --in winres\winres.json --out rsrc   # generates rsrc_windows_amd64.syso
go build -o miniMiddleware.exe .
```

The build is **console subsystem** (not `-H windowsgui`); the `config` GUI hides the console window itself. This keeps stdout working for the legacy interactive tunnel mode.

## Configuration — stored as a service property in the Registry

Config lives under the service key (so it's removed automatically when the service is uninstalled):

```
HKLM\SYSTEM\CurrentControlSet\Services\CondlinkMiddleware\Parameters
  Terminais  REG_MULTI_SZ   one "ip=devId" line per terminal
  Username   REG_SZ
  Password   REG_SZ
```

Edit it via the GUI: run `miniMiddleware.exe config` (or the Start-menu shortcut "Configurar..."). On first run, if the Registry key is empty, `LoadConfig()` falls back to importing the legacy `C:\admin-condlink\terminal.json` / `login.json`.

## CLI / Service subcommands

```powershell
miniMiddleware.exe install   # cria o serviço (StartAutomatic)
miniMiddleware.exe remove    # remove o serviço (e suas propriedades no Registro)
miniMiddleware.exe start     # inicia
miniMiddleware.exe stop      # para
miniMiddleware.exe config    # abre a GUI de configuração
miniMiddleware.exe 1.2.3.4   # modo legado: roda túneis para IPs passados (debug em console)
```

Service name: `CondlinkMiddleware`. When started by the SCM the binary auto-detects via `svc.IsWindowsService()` and runs as a service; otherwise it's interactive. As a service it logs to `%ProgramData%\Condlink\MiniMiddleware\miniMiddleware.log`.

## Filesystem layout (Windows-standard)

- `%ProgramFiles%\Condlink\MiniMiddleware\` (`{app}`) — `miniMiddleware.exe` + bundled `cloudflared.exe` (read-only binaries). `cloudflaredFullPath()` = `exeDir()` + `cloudflared.exe`.
- `%ProgramData%\Condlink\MiniMiddleware\` (`dataDir()`) — writable runtime data: `miniMiddleware.log`.
- `C:\admin-condlink\` — **legacy only**: read once by `migrateFromJSON()` to import old `terminal.json`/`login.json`. Not created or written anymore.

## Architecture notes

- `runTunnels()` loads config, builds the IP list + `devId` map, then one `startTunnel()` goroutine per IP.
- `startTunnel()` runs cloudflared, scans stderr for the `trycloudflare.com` URL (60s timeout), then `login()` + `UsePutTunel()` register it. A global `stopChan` + tracked `procs` slice let the service stop kill all cloudflared children cleanly.
- API base: `https://us-central1-earnest-cosmos-175020.cloudfunctions.net/`
- `init()` enforces Windows-only.
