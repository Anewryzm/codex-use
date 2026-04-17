# codex-use

Script para gestionar múltiples cuentas de Codex por perfil y promover una cuenta al perfil default (`~/.codex`) que usa Codex App + Codex CLI sin `CODEX_HOME`.

## Instalación (npm)
- `npm i -g @anewryzm/codex-use`

## Comandos
- Principal: `codex-use`
- Alias corto: `cdex`

## Archivo principal
- `tools/codex-use`

## Flujo principal
1. Guardar login por perfil:
- `codex-use login personal`
- `codex-use login trabajo`

2. Cambiar cuenta activa del default:
- `codex-use use personal`
- `codex-use use trabajo`
- `codex-use use trabajo --relaunch` (recomendado para que Codex App refresque sesión automáticamente)
- `codex-use use trabajo --relaunch --from-action` (para Actions: relanza app y cierra la pestaña de Terminal en éxito)

3. Ver usage/rate limits:
- `codex-use usage`
- `codex-use usage --json`
- `codex-use limits trabajo --refresh`
- `codex-use limits --all` o `codex-use limits -a` (vista compacta de límites por perfil)

## Comandos disponibles
- `codex-use login <profile>`
- `codex-use use <profile> [--force] [--relaunch] [--from-action]`
- `codex-use add switch action [--platform <macos|darwin>] [--icon <icon>] [--command-name <cdex|codex-use>] [--dry-run]`
- `codex-use delete switch action [--platform <macos|darwin>] [--dry-run]`
- `codex-use usage [--json]`
- `codex-use backup [--note "..."] [--dir <path>] [--delete]`
- `codex-use backup status [--dir <path>]`
- `codex-use backup init [--dir <path>]`
- `codex-use backup add remote <url> [--name <remote>] [--dir <path>]`
- `codex-use backup push [--remote <name>] [--branch <name>] [--dir <path>]`
- `codex-use backup restore [--dir <path>] [--relaunch] [--delete]`
- `codex-use limits [profile|default] [--all] [--json] [--refresh] [--live-only] [--allow-cache] [--backend <auto|wham|rpc>]`
- `codex-use limits doctor [profile|--all] [--refresh] [--allow-cache] [--backend <auto|wham|rpc>]`
- `codex-use list`
- `codex-use whoami`
- `codex-use status <profile> [--refresh] [--live-only] [--allow-cache] [--backend <auto|wham|rpc>]`
- `codex-use logout <profile>`
- `codex-use logout-default`

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
1. Ejecuta `codex-use list`.
2. Revisa la leyenda:
- `[*]` = el perfil usa exactamente el mismo token que `~/.codex`.
- `[~]` = mismo `account_id`, pero token/identidad distinta.
3. Si haces switch con la app abierta, usa `codex-use use <profile> --relaunch`.
4. Ejecuta `codex-use whoami` para ver `Active profile` además de email/plan/account.

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
- `codex-use add switch action --platform macos`
- Solo vista previa:
- `codex-use add switch action --dry-run`
- Eliminar acciones de switch generadas para macOS:
- `codex-use delete switch action --platform macos`
