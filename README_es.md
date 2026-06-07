# dex

Script para gestionar múltiples cuentas de Codex por perfil y promover una cuenta al perfil default (`~/.codex`) que usa Codex App + Codex CLI sin `CODEX_HOME`.

## Instalación (npm)
- `npm i -g @enriquecardoza/dex`

## Comandos
- Principal: `dex`
- Alias corto: `cdex`
- Alias legacy: `codex-use`

## Archivo principal
- `tools/codex-use`

## Flujo principal
1. Guardar login por perfil:
- `dex login personal`
- `dex login trabajo`

2. Cambiar cuenta activa del default:
- `dex use personal`
- `dex use trabajo`
- `dex use trabajo --relaunch` (recomendado para que Codex App refresque sesión automáticamente)
- `dex use trabajo --relaunch --from-action` (para Actions: relanza app y cierra la pestaña de Terminal en éxito)

3. Ver usage/rate limits:
- `dex usage`
- `dex usage --json`
- `dex limits trabajo --refresh`
- `dex limits --all` o `dex limits -a` (vista compacta de límites por perfil)

## Comandos disponibles
- `dex login <profile>`
- `dex use <profile> [--force] [--relaunch] [--from-action]`
- `dex add switch action [--platform <macos|darwin>] [--icon <icon>] [--command-name <dex|cdex|codex-use>] [--dry-run]`
- `dex delete switch action [--platform <macos|darwin>] [--dry-run]`
- `dex usage [--json]`
- `dex backup [--note "..."] [--dir <path>] [--delete]`
- `dex backup status [--dir <path>]`
- `dex backup init [--dir <path>]`
- `dex backup add remote <url> [--name <remote>] [--dir <path>]`
- `dex backup push [--remote <name>] [--branch <name>] [--dir <path>]`
- `dex backup restore [--dir <path>] [--relaunch] [--delete]`
- `dex limits [profile|default] [--all] [--json] [--refresh] [--live-only] [--allow-cache] [--backend <auto|wham|rpc>]`
- `dex limits doctor [profile|--all] [--refresh] [--allow-cache] [--backend <auto|wham|rpc>]`
- `dex list`
- `dex whoami`
- `dex status <profile> [--refresh] [--live-only] [--allow-cache] [--backend <auto|wham|rpc>]`
- `dex logout <profile>`
- `dex logout-default`

## Notas de seguridad
- `use` crea backup automático de `~/.codex/auth.json` en `~/.codex/backups/`.
- Evita cambiar cuenta con tareas activas en Codex App.
- `--force` existe para casos avanzados y puede interrumpir sesiones en curso.
- `--relaunch` cierra y vuelve a abrir Codex App para que la UI lea el nuevo `auth.json`.
- `--from-action` está pensado para Codex Actions: tras un `--relaunch` exitoso, intenta cerrar automáticamente la pestaña actual de Terminal.
- `usage` lee histórico local desde `~/.codex/sessions`.
- `backup` crea snapshots de `~/.codex/sessions` en un repo Git (default: `~/codex-history-backup`).
- `backup` por defecto solo copia (no borra). Usa `--delete` solo si quieres modo espejo.
- `limits` es live-first: API directa `wham/usage` con token del perfil y fallback a RPC (`account/rateLimits/read`).

## Diagnóstico rápido cuando "no cambia la cuenta" en la app
1. Ejecuta `dex list`.
2. Revisa la leyenda:
- `[*]` = el perfil usa exactamente el mismo token que `~/.codex`.
- `[~]` = mismo `account_id`, pero token/identidad distinta.
3. Si haces switch con la app abierta, usa `dex use <profile> --relaunch`.
4. Ejecuta `dex whoami` para ver `Active profile` además de email/plan/account.

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
- `limits doctor` imprime diagnóstico de backend (`wham` y `rpc`), latencia, motivo de error, backend elegido y edad de caché.
- Por default, `limits`/`status` **no** hacen fallback a caché. Fallan con error live si ningún backend responde.
- Usa `--allow-cache` para habilitar fallback desde snapshot local (`<profile>/cache/codex-use-rate-limits.json`) cuando fallen los backends live.
- Usa `--backend wham|rpc|auto` para forzar backend (`auto` prueba `wham` primero y luego `rpc`).
- En `limits --all`, las filas en caché se marcan con `~` y `(cached)` en la columna de email.
- En `status <profile>`, el fallback de caché se muestra explícitamente como `note: showing cached limits snapshot (...)`.
- Usa `--live-only` para forzar explícitamente modo sin caché (igual que el default, salvo que pases `--allow-cache`).
- Estados de frescura en `limits --all`:
- `✓` significa que un backend live respondió y los valores son frescos.
- `~` significa que fallaron los backends live, pero se está usando snapshot en caché (`--allow-cache`).
- `network-error`/`rpc-error` significa que fallaron los backends live y no se usó fallback de caché.

## Salida de `backup status`
- `backup status` muestra salud del respaldo con formato orientado a terminal (`key: value`): estado del repo/git, conteo de archivos origen vs backup, cobertura por `session_id` y deltas pendientes de copia/borrado.
- `pending_copy_files` indica lo que se copiaría en modo default.
- `pending_new_files` indica rollouts nuevos que aún no existen en backup.
- `pending_modified_files` indica rollouts existentes que cambiaron (por ejemplo, cuando se añade más conversación al mismo `.jsonl`).
- `pending_mirror_deletes` indica lo que se borraría solo en modo espejo con `--delete`.
- `runtime_unflushed_activity` indica que hay actividad fresca de runtime (`logs_2.sqlite` / `state_5.sqlite`) más nueva que el último write en `sessions/*.jsonl`.
- `runtime_vs_sessions_lag_seconds` indica cuántos segundos de desfase hay entre runtime y escritura en session logs.

## Notas del commit de `backup`
- Los commits de `backup` ahora incluyen una sección `Session notes` con solo rollouts nuevos/modificados desde el commit anterior (no lista acumulativa infinita).
- Cada línea incluye archivo de rollout, título resuelto (si existe), modelo, snapshot de tokens totales y timestamp más reciente.

## Generar Actions de Codex
- `add switch action` lee perfiles logueados desde `~/.codex-profiles` y escribe acciones de cambio por perfil en `.codex/environments/environment.toml`.
- Soporte de plataforma por ahora: `macos` (alias: `darwin`, se escribe como `darwin` en el TOML).
- Ejemplo:
- `dex add switch action --platform macos`
- Solo vista previa:
- `dex add switch action --dry-run`
- Eliminar acciones de switch generadas para macOS:
- `dex delete switch action --platform macos`
