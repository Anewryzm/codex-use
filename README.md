# dex

CLI to manage multiple Codex accounts by profile and promote one account to the default profile (`~/.codex`) used by Codex App + Codex CLI when `CODEX_HOME` is not set.

## Install (npm)
- `npm i -g @enriquecardoza/dex`

## Commands
- Primary: `dex`
- Short alias: `cdex`
- Legacy alias: `codex-use`

## Main file
- `tools/codex-use`

## Core flow
1. Save login per profile:
- `dex login personal`
- `dex login work`

2. Switch the active default account:
- `dex use personal`
- `dex use work`
- `dex use work --relaunch` (recommended so Codex App refreshes the session automatically)
- `dex use work --relaunch --from-action` (for Actions: relaunches the app and closes the Terminal tab on success)

3. Check usage/rate limits:
- `dex usage`
- `dex usage --json`
- `dex limits work --refresh`
- `dex limits --all` or `dex limits -a` (compact limits view by profile)

## Available commands
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

## Security notes
- `use` creates an automatic backup of `~/.codex/auth.json` in `~/.codex/backups/`.
- Avoid switching accounts while there are active tasks in Codex App.
- `--force` exists for advanced cases and may interrupt in-flight sessions.
- `--relaunch` closes and reopens Codex App so the UI reads the new `auth.json`.
- `--from-action` is intended for Codex Actions: after a successful `--relaunch`, it tries to auto-close the current Terminal tab.
- `usage` reads local history from `~/.codex/sessions`.
- `backup` snapshots `~/.codex/sessions` into a Git repo (default: `~/codex-history-backup`).
- `backup` is copy-only by default (no deletions). Use `--delete` only when you intentionally want mirror mode.
- `limits` is live-first: direct `wham/usage` API with profile token, then RPC fallback (`account/rateLimits/read`).

## Quick troubleshooting when "the account did not change" in the app
1. Run `dex list`.
2. Check the legend:
- `[*]` = the profile uses exactly the same token as `~/.codex`.
- `[~]` = same `account_id`, but different token/identity.
3. If you switch while the app is open, use `dex use <profile> --relaunch`.
4. Run `dex whoami` to see `Active profile` plus email/plan/account.

## Difference between `whoami` and `status`
- `whoami`: quick view of the active profile in `~/.codex` (the account used by app/CLI by default).
- `status <profile>`: diagnostics for a specific profile in `~/.codex-profiles/<profile>`, including:
- identity summary (`plan`, `default_org`, `email`, `account_id`)
- usage limits (`5h` and `weekly`) with remaining percentage, horizontal bar, and reset timing
- readable credits summary (`credits: none`, `credits: unlimited`, `credits: balance ...`)
- Use `--refresh` to force a fresh backend query before showing limits.

## `usage` output
- `usage` shows historical consumption from `~/.codex/sessions` in a compact table.
- Columns: `in(M)`, `out(M)`, `cach(M)` (millions of tokens). `out(M)` includes reasoning.
- `usage --json` returns the raw aggregate (`totals` and `models`) for scripting.

## `limits` output
- `limits <profile>` (without `--json`) shows dashboard-style output: `5h` and `weekly` bars, natural-language reset, and credits summary.
- `limits --all`/`limits -a` shows a compact table with columns `profile`, `email`, `5h (reset)`, `weekly (reset)` and a `Current default profile` line.
- `limits --json` keeps the raw JSON output from rate limits for scripting/automation.
- `limits doctor` prints backend diagnostics (`wham` and `rpc`), latency, error reason, selected backend, and cache age.
- By default, `limits`/`status` do **not** fall back to cache. They fail with live error when no backend responds.
- Use `--allow-cache` to enable fallback from local snapshot (`<profile>/cache/codex-use-rate-limits.json`) when live backends fail.
- Use `--backend wham|rpc|auto` to force backend selection (`auto` defaults to `wham` first, then `rpc`).
- In `limits --all`, cached rows are marked with `~` and `(cached)` in the email column.
- In `status <profile>`, cached fallback is explicitly shown as `note: showing cached limits snapshot (...)`.
- Use `--live-only` to explicitly force no-cache behavior (same as default unless `--allow-cache` is passed).
- Freshness states in `limits --all`:
- `✓` means a live backend responded and values are fresh.
- `~` means all live backends failed, but cached snapshot is being used (`--allow-cache`).
- `network-error`/`rpc-error` means live backends failed and no cache fallback was used.

## `backup status` output
- `backup status` shows backup health in terminal-first format (`key: value`), including repo/git state, source vs backup file counts, session-id coverage, and pending copy/delete deltas.
- `pending_copy_files` indicates what would be copied in default mode.
- `pending_new_files` indicates brand-new rollouts not yet present in backup.
- `pending_modified_files` indicates existing rollout files that changed (for example, more conversation appended to the same `.jsonl`).
- `pending_mirror_deletes` indicates what would be removed only in `--delete` mirror mode.
- `runtime_unflushed_activity` indicates there is fresh Codex runtime activity (`logs_2.sqlite` / `state_5.sqlite`) newer than the latest `sessions/*.jsonl` write.
- `runtime_vs_sessions_lag_seconds` indicates how far runtime activity is ahead of session-log writes.

## `backup` commit notes
- `backup` commits now include a `Session notes` section with only new/changed rollouts since the previous backup commit (not a cumulative list).
- Each note line includes rollout file, resolved title (when available), model, token total snapshot, and latest timestamp.

## Generate Codex Actions
- `add switch action` reads logged profiles from `~/.codex-profiles` and writes per-profile switch actions into `.codex/environments/environment.toml`.
- Platform support for now: `macos` (alias: `darwin`, written as `darwin` in the TOML file).
- Example:
- `dex add switch action --platform macos`
- Preview only:
- `dex add switch action --dry-run`
- Delete generated switch actions for macOS:
- `dex delete switch action --platform macos`
