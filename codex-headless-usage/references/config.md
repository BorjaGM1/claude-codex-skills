# Config basics

## Locations
- User config: `~/.codex/config.toml` (Windows: `C:\Users\<user>\.codex\config.toml`)
- Project config: `.codex/config.toml` inside a repo (loads only for trusted projects).
- The CLI and IDE extension share config layers.

## Precedence (highest to lowest)
1. CLI flags and `--config` overrides
2. Selected profile from `--profile`
3. Project config files
4. User config

## Common options
- `model = "gpt-6.1-sol"` (global default; `gpt-6-astra` for "will this fix hold?")
- `model_reasoning_effort = "low"` (global default. **Effort buys no findings** — low through xhigh measured identical on 6.1-sol and astra, so don't pin higher per-call)
- `approval_policy = "on-request"`
- `sandbox_mode = "workspace-write"`
- `web_search = "cached" | "live" | "disabled"`

## Profiles
- A profile is a separate file `$CODEX_HOME/<name>.config.toml`, layered on top of the base user config; select with `--profile <name>` (`-p`). It overrides only the keys it sets.
- Profiles are experimental and may change.
- `--ignore-user-config` bypasses the base config entirely, dropping effort to the **model default** (`low` for 6.1-sol and astra, `medium` for 6-sol/6-luna). Pin `-c model_reasoning_effort=…` when you use it.

## Trusted projects
The CLI tracks trusted directories under `[projects.'<path>']`. Without trust, codex refuses to run unless `--skip-git-repo-check` is passed.
