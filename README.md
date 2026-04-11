# codex-account-switch

Script para gestionar múltiples cuentas de Codex por perfil y promover una cuenta al perfil default (`~/.codex`) que usa Codex App + Codex CLI sin `CODEX_HOME`.

## Archivo principal
- `tools/codexacct`

## Flujo principal
1. Guardar login por perfil:
- `tools/codexacct login personal`
- `tools/codexacct login trabajo`

2. Cambiar cuenta activa del default:
- `tools/codexacct use personal`
- `tools/codexacct use trabajo`

3. Ver usage/rate limits por perfil (estilo CodexBar, vía RPC):
- `tools/codexacct usage default`
- `tools/codexacct usage trabajo`
- `tools/codexacct usage trabajo --json`
- `tools/codexacct limits trabajo --refresh`
- `tools/codexacct usage --all` (tabla ASCII de todas las cuentas vinculadas)

## Comandos disponibles
- `tools/codexacct login <profile>`
- `tools/codexacct use <profile> [--force]`
- `tools/codexacct usage [profile|default] [--all] [--json] [--refresh]`
- `tools/codexacct limits [profile|default] [--all] [--json] [--refresh]`
- `tools/codexacct list`
- `tools/codexacct whoami`
- `tools/codexacct status <profile>`
- `tools/codexacct logout <profile>`
- `tools/codexacct logout-default`

## Notas de seguridad
- `use` crea backup automático de `~/.codex/auth.json` en `~/.codex/backups/`.
- Evita cambiar cuenta con tareas activas en Codex App.
- `--force` existe para casos avanzados y puede interrumpir sesiones en curso.
- `usage/limits` consulta `codex app-server` con JSON-RPC (`initialize`, `account/read`, `account/rateLimits/read`).
