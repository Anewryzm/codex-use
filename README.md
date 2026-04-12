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
- `tools/codexacct use trabajo --relaunch --from-action` (para Actions: relanza app y cierra la pestaña de Terminal en éxito)

3. Ver usage/rate limits:
- `tools/codexacct usage`
- `tools/codexacct usage --json`
- `tools/codexacct limits trabajo --refresh`
- `tools/codexacct limits --all` o `tools/codexacct limits -a` (vista compacta de límites por perfil)

## Comandos disponibles
- `tools/codexacct login <profile>`
- `tools/codexacct use <profile> [--force] [--relaunch] [--from-action]`
- `tools/codexacct usage [--json]`
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
- `--from-action` está pensado para Codex Actions: tras un `--relaunch` exitoso, intenta cerrar automáticamente la pestaña actual de Terminal.
- `usage` lee histórico local desde `~/.codex/sessions`.
- `limits` consulta `codex app-server` con JSON-RPC (`initialize`, `account/read`, `account/rateLimits/read`).

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

## Salida de `usage`
- `usage` muestra consumo histórico desde `~/.codex/sessions` en tabla compacta.
- Columnas: `in(M)`, `out(M)`, `cach(M)` (millones de tokens). `out(M)` incluye reasoning.
- `usage --json` entrega el agregado crudo (`totals` y `models`) para scripting.

## Salida de `limits`
- `limits <profile>` (sin `--json`) usa salida legible tipo dashboard: barras de `5h` y `weekly`, renovación en lenguaje natural y resumen de créditos.
- `limits --all`/`limits -a` usa tabla compacta con columnas `profile`, `email`, `5h (reset)`, `weekly (reset)` y línea `Current default profile`.
- `limits --json` conserva salida JSON cruda de rate limits para scripting/automatización.
