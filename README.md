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
- `tools/codexacct use trabajo --relaunch` (recomendado para que Codex App refresque sesión automáticamente)

3. Ver usage/rate limits por perfil (tabla unificada, vía RPC):
- `tools/codexacct usage default`
- `tools/codexacct usage trabajo`
- `tools/codexacct usage trabajo --json`
- `tools/codexacct limits trabajo --refresh`
- `tools/codexacct usage --all` o `tools/codexacct usage -a` (tabla ASCII de todas las cuentas vinculadas)
- `tools/codexacct limits --all` o `tools/codexacct limits -a` (vista compacta de límites por perfil)

## Comandos disponibles
- `tools/codexacct login <profile>`
- `tools/codexacct use <profile> [--force] [--relaunch]`
- `tools/codexacct usage [profile|default] [--all] [--json] [--refresh]`
- `tools/codexacct limits [profile|default] [--all] [--json] [--refresh]`
- `tools/codexacct list`
- `tools/codexacct whoami`
- `tools/codexacct status <profile> [--refresh]`
- `tools/codexacct logout <profile>`
- `tools/codexacct logout-default`

## Notas de seguridad
- `use` crea backup automático de `~/.codex/auth.json` en `~/.codex/backups/`.
- Evita cambiar cuenta con tareas activas en Codex App.
- `--force` existe para casos avanzados y puede interrumpir sesiones en curso.
- `--relaunch` cierra y vuelve a abrir Codex App para que la UI lea el nuevo `auth.json`.
- `usage/limits` consulta `codex app-server` con JSON-RPC (`initialize`, `account/read`, `account/rateLimits/read`).

## Diagnóstico rápido cuando "no cambia la cuenta" en la app
1. Ejecuta `tools/codexacct list`.
2. Revisa la leyenda:
- `[*]` = el perfil usa exactamente el mismo token que `~/.codex`.
- `[~]` = mismo `account_id`, pero token/identidad distinta.
3. Si haces switch con la app abierta, usa `tools/codexacct use <profile> --relaunch`.
4. Ejecuta `tools/codexacct whoami` para ver `Active profile` además de email/plan/account.

## Diferencia entre `whoami` y `status`
- `whoami`: vista rápida del perfil activo en `~/.codex` (cuenta que usa la app/CLI por defecto).
- `status <profile>`: diagnóstico de un perfil específico en `~/.codex-profiles/<profile>`, incluyendo:
- resumen de identidad (`plan`, `default_org`, `email`, `account_id`)
- límites de uso (`5h` y `weekly`) con porcentaje restante, barra horizontal y tiempo de renovación
- resumen de créditos legible (`credits: none`, `credits: unlimited`, `credits: balance ...`)
- Usa `--refresh` para forzar consulta fresca al backend antes de mostrar límites.

## Salida de `limits`
- `usage` y `limits` comparten la misma salida tabular cuando no usas `--json`.
- `--all`/`-a` muestra tabla compacta por perfiles guardados (sin fila `default`) y una línea `Current default profile: <perfil>*`.
- `usage --json` y `limits --json` conservan salida JSON cruda para scripting/automatización.
