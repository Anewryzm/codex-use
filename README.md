# codex-use

CLI to manage multiple Codex accounts by profile and promote one account to the default profile (`~/.codex`) used by Codex App + Codex CLI when `CODEX_HOME` is not set.

## Install (npm)
- `npm i -g @anewryzm/codex-use`

## Commands
- Primary: `codex-use`
- Short alias: `cdex`

## Main file
- `tools/codex-use`

## Core flow
1. Save login per profile:
- `codex-use login personal`
- `codex-use login work`

2. Switch the active default account:
- `codex-use use personal`
- `codex-use use work`
- `codex-use use work --relaunch` (recommended so Codex App refreshes the session automatically)
- `codex-use use work --relaunch --from-action` (for Actions: relaunches the app and closes the Terminal tab on success)

3. Check usage/rate limits:
- `codex-use usage`
- `codex-use usage --json`
- `codex-use limits work --refresh`
- `codex-use limits --all` or `codex-use limits -a` (compact limits view by profile)

## Available commands
- `codex-use login <profile>`
- `codex-use use <profile> [--force] [--relaunch] [--from-action]`
- `codex-use add switch action [--platform <macos|darwin>] [--icon <icon>] [--command-name <cdex|codex-use>] [--dry-run]`
- `codex-use delete switch action [--platform <macos|darwin>] [--dry-run]`
- `codex-use usage [--json]`
- `codex-use backup [--note "..."] [--dir <path>] [--delete]`
- `codex-use backup init [--dir <path>]`
- `codex-use backup add remote <url> [--name <remote>] [--dir <path>]`
- `codex-use backup push [--remote <name>] [--branch <name>] [--dir <path>]`
- `codex-use backup restore [--dir <path>] [--relaunch] [--delete]`
- `codex-use limits [profile|default] [--all] [--json] [--refresh]`
- `codex-use list`
- `codex-use whoami`
- `codex-use status <profile> [--refresh]`
- `codex-use logout <profile>`
- `codex-use logout-default`

## Security notes
- `use` creates an automatic backup of `~/.codex/auth.json` in `~/.codex/backups/`.
- Avoid switching accounts while there are active tasks in Codex App.
- `--force` exists for advanced cases and may interrupt in-flight sessions.
- `--relaunch` closes and reopens Codex App so the UI reads the new `auth.json`.
- `--from-action` is intended for Codex Actions: after a successful `--relaunch`, it tries to auto-close the current Terminal tab.
- `usage` reads local history from `~/.codex/sessions`.
- `backup` snapshots `~/.codex/sessions` into a Git repo (default: `~/codex-history-backup`).
- `backup` is copy-only by default (no deletions). Use `--delete` only when you intentionally want mirror mode.
- `limits` queries `codex app-server` using JSON-RPC (`initialize`, `account/read`, `account/rateLimits/read`).

## Quick troubleshooting when "the account did not change" in the app
1. Run `codex-use list`.
2. Check the legend:
- `[*]` = the profile uses exactly the same token as `~/.codex`.
- `[~]` = same `account_id`, but different token/identity.
3. If you switch while the app is open, use `codex-use use <profile> --relaunch`.
4. Run `codex-use whoami` to see `Active profile` plus email/plan/account.

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

## Generate Codex Actions
- `add switch action` reads logged profiles from `~/.codex-profiles` and writes per-profile switch actions into `.codex/environments/environment.toml`.
- Platform support for now: `macos` (alias: `darwin`, written as `darwin` in the TOML file).
- Example:
- `codex-use add switch action --platform macos`
- Preview only:
- `codex-use add switch action --dry-run`
- Delete generated switch actions for macOS:
- `codex-use delete switch action --platform macos`
