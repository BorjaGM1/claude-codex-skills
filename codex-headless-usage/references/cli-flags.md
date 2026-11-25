# CLI flags (headless focus)

Verified against **codex-cli 0.153.x**. Check with `codex --help` / `codex exec --help`.

## Global flags
- `--add-dir <path>` — grant additional directories write access alongside the main workspace.
- `-a, --ask-for-approval <on-request|never>` — when Codex pauses for approval.
  **Only these two values.** `untrusted` and `on-failure` no longer exist.
- `--approve-for-me` — route approval requests through automatic review using the workspace-write sandbox.
- `-C, --cd <path>` — set the working directory for the run.
- `-c, --config key=value` — inline override for a single run. Dotted paths (`foo.bar.baz`) reach nested values; the value is parsed as TOML, falling back to a literal string.
- `--enable <FEATURE>` / `--disable <FEATURE>` — repeatable; equivalent to `-c features.<name>=true|false`.
- `--strict-config` — error out on config keys this binary doesn't recognise.
- `--dangerously-bypass-approvals-and-sandbox` — bypass approvals and sandboxing. There is **no `--yolo` and no `--full-auto`** any more.
- `--dangerously-bypass-hook-trust` — run enabled hooks without persisted hook trust.
- `-i, --image <FILE>...` — attach images to the initial prompt. **Space-separated**, not comma-joined. This is *input* — images for Codex to read. Image *generation* is a separate thing (built-in `image_gen` tool / `visualize` plugin, gated by the `image_generation` feature).
- `-m, --model <string>` — override the configured model.
- `--oss` — use the open-source provider; pick the backend with `--local-provider <lmstudio|ollama>`.
- `-p, --profile <name>` — layer `$CODEX_HOME/<name>.config.toml` on top of the base user config.
- `-s, --sandbox <read-only|workspace-write|danger-full-access>` — sandbox policy.
- `--search` — enable live web search (native `web_search` tool, no per-call approval).
- `--no-alt-screen` — inline TUI, preserving scrollback.
- `--remote <ADDR>` / `--remote-auth-token-env <ENV_VAR>` — drive a remote app server.
- `PROMPT` — optional text instruction to start the session.

## codex exec flags
- `PROMPT` — string prompt, or `-` to read from stdin. Piped stdin is appended as a `<stdin>` block when a prompt is also given.
- `--json` — emit newline-delimited JSON events.
- `-o, --output-last-message <path>` — write the final response to a file.
- `--output-schema <path>` — JSON Schema the final response must match.
- `--skip-git-repo-check` — allow running outside a Git repository.
- `--ephemeral` — run without persisting session files to disk.
- `--ignore-user-config` — don't load `$CODEX_HOME/config.toml` (auth still uses `CODEX_HOME`). **This drops effort to the model default** — pin `-c model_reasoning_effort=…` explicitly.
- `--ignore-rules` — don't load user or project execpolicy `.rules` files.
- `--thread-source <SOURCE>` — source classification for new/forked threads.
- `--color <always|never|auto>` — ANSI colour in stdout.

Sub-subcommands: `codex exec resume [--last]`, `codex exec fork`, `codex exec review`.

## codex review flags
`--uncommitted`, `--base <BRANCH>`, `--commit <SHA>`, `--title <TITLE>`, and a
`PROMPT` (or `-` for stdin) for custom emphasis — but on 0.160 the `PROMPT` is
**refused when combined with any scope flag** ("cannot be used with '[PROMPT]'").
Run a scoped review bare, with stdin closed.

**There is no `--model` on `codex review`** — use `-c model=…` and
`-c model_reasoning_effort=…`.

## Other subcommands worth knowing
`codex debug models` (live catalog), `codex features list`, `codex doctor`,
`codex plugin list`, `codex agents`, `codex sandbox`, `codex apply`,
`codex queue`, `codex fork`, `codex archive|unarchive|delete`, `codex cloud`.
