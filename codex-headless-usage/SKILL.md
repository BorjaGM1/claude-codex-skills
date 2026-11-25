---
name: codex-headless-usage
description: Headless Codex CLI usage for deep thinking and analysis. Codex is a thinker, not an implementer — read-only by default. gpt-6.2-sol @ low default (6.2-astra an equivalent second opinion, luna never); effort buys no findings, so don't pay for xhigh; ultra does not delegate. Master reference for all codex-* skills.
---

# Headless Codex (codex exec)

## Philosophy

Codex is a **thinker, not an implementer**. Use him for:
- Deep analysis and reasoning
- Architecture discussions
- Test strategy design
- Code review
- Root cause analysis

Claude Code implements. Codex thinks.

## Model

Default: `gpt-6.2-sol` at `low` — set it in `~/.codex/config.toml`:

```
model = "gpt-6.2-sol"
model_reasoning_effort = "low"
```

User's `~/.codex/config.toml` sets these globally; CLI flags override per-call.
**Needs codex-cli ≥ 0.172** — older binaries don't list the 6.2 models
(`brew upgrade --cask codex`).

**Effort buys no findings.** Measured 2026-11-25: 6.2-sol at `xhigh` returned the
same findings as `low` at 3.7× the latency (same result for the previous models
in October and September). Pin `low`.

**Pick the model by what you're asking for:**

- *Anything* (review, forensics, consult, audit) → **`gpt-6.2-sol` @ low**.
  On real-world regressions (blind-graded, 2026-11-25) it found 56% vs 37% for
  the previous `gpt-6.1-sol` on the same cases, with 0 wrong claims.
- *A second opinion* → **`gpt-6.2-astra` @ medium**. It tied 6.2-sol (60% vs 56%,
  noise) and is the most thorough on secondary edge cases. Neither model says on
  its own when the obvious fix is incomplete: to check a fix holds, *ask that
  explicitly* ("is this fix complete? what does it miss?").
- `gpt-6.1-sol` / `gpt-6-astra` are "previous generation" — ~20 points behind.
- **Never `gpt-6-luna` / `gpt-5.6-luna` for review** — luna called buggy code
  correct, fabricated failing inputs, and found 4.8% of real regressions.
- **Don't use `gpt-6-sol` as a second pass** — 31% per run on real regressions and
  +1 bug over 6.1-sol. A second *Codex* model barely helps; a second *vendor* would.

```
codex exec -m gpt-6.2-astra -c model_reasoning_effort=medium ...   # second opinion
```

**No 6.x model gets plugin/apps instructions injected.** If you actually want
Codex driving plugins (documents, spreadsheets, browser, `image_gen`), use
`gpt-5.6-sol` — but it **retires 2026-12-23** (→ `gpt-6.2-sol`, which has no
plugin instructions).

### The ladder

`low < medium < high < xhigh < max < ultra`, but two entries are not what they look like:

- **`ultra` is client-side only.** The server enum stops at `max`; `ultra` maps to
  `max` plus a delegation toolset. It therefore no longer errors on `luna` — it
  silently degrades to `max`.
- **`ultra` does not currently delegate.** A `--json` trace showed zero
  `spawn_agent` / `followup_task` / `send_message` events, matching
  `multi_agent_v2` being effective-`false` in `codex features list`. One such run
  cost 259s and ~170k input tokens for a single answer. **Don't pay for it.**
- **`max` is model-gated** — check `supported_reasoning_levels` in `codex debug models` (luna has no `ultra`).

These numbers were measured in November 2026 (codex-cli 0.172). When a new model ships, check the
live lineup with `codex debug models` before trusting the names above.

## Git repo requirement

`codex exec` requires the target directory to be a git repository. Without one, it fails with:
```
Not inside a trusted directory and --skip-git-repo-check was not specified.
```

**Before running Codex**, check if the `--cd` target is a git repo. If not, either:
1. `git init` the directory first (preferred — you'll likely want version control anyway)
2. Add `--skip-git-repo-check` to the command (use when git init isn't appropriate)

## Default execution

Always use read-only sandbox unless explicitly needed otherwise.

**Short prompts** — pass inline:
```bash
codex exec \
  --sandbox read-only \
  "Short prompt here" < /dev/null
```

**Long/multiline prompts** — pipe via stdin with `-`:
```bash
cat <<'PROMPT' | codex exec \
  --cd /path/to/repo \
  --sandbox read-only \
  --output-last-message /path/to/output.md \
  -
Your long prompt here.
Multiple lines work fine.
PROMPT
```

Long prompts passed as arguments get mangled by the shell. Always use the stdin pipe pattern (`-`) for anything beyond a single line.

## ⚠️ The stdin gotcha (critical)

**Always close stdin.** When run from a non-interactive shell (Bash tool, scripts, CI), codex hangs indefinitely on "Reading additional input from stdin..." waiting for EOF that never comes. Symptom: process alive for 30+ minutes with ~0s CPU time. This is **not** xhigh reasoning — it's a stdin deadlock.

- **Bash tool**: append `< /dev/null`

When using the stdin-piped pattern (`-`), the heredoc closes stdin for you — no extra flag needed.

## Background execution

**xhigh and above can take 5–20+ minutes on a real repo** and produce no output while thinking. Run them as a background task — `run_in_background: true` on the Bash tool call — so the conversation isn't blocked.

Calibration from measured runs (small, 2–5 file targets, 2026-10-03, under
concurrent load — solo runs are faster):

- **6.1-sol @ low**: 20–60s. **astra @ medium**: 25–70s. Foreground is fine;
  `timeout: 180000`.
- **6.1-sol @ xhigh**: 80–210s — 3× low, same findings. Background it if you run it.
- **ultra**: 259s on a 5-file audit (astra, Sept) — no delegation, no better
  answer. Always background if you run it at all.
- Whole-repo work scales far past these numbers. When the target is a real
  codebase rather than a handful of files, background it regardless of effort.

**Never get impatient.** If the process is still running, let it finish. Killing mid-reasoning wastes the entire computation.

## Diagnosing stuck vs. thinking

Run a process check. If elapsed time is 30 min but CPU time is under a few seconds, it's stuck on stdin — kill and re-run with stdin closed.

- `ps -o pid,etime,time -p <PID>`

Real reasoning burns at least some CPU even when the model is sleeping between tokens.

## Multi-round discussions

For complex problems, iterate with Codex:

1. Prepare a well-designed prompt with full context
2. Let Codex think deeply (default model at `low` throughout — effort buys no findings)
3. Review his response critically
4. Question anything that doesn't fully click
5. Repeat until consensus or diminishing returns
6. Minimum 2–3 rounds recommended
7. Present final consensus to user

This is pair programming — challenge his thinking, don't just accept it.

See [[codex-consult]] for the full multi-round workflow.

## Core flags

| Flag | Purpose |
|------|---------|
| `--cd <path>` | Set working directory |
| `--sandbox read-only` | Default — read but no write |
| `--skip-git-repo-check` | Allow running outside a git repo |
| `--json` | JSONL event stream output |
| `--output-last-message <path>` | Clean final response only |
| `--config key=value` | Per-run override (e.g. `model_reasoning_effort=medium`) |
| `--add-dir <path>` | Grant access to additional directories |
| `-i, --image <FILE>...` | Attach images to the initial prompt (space-separated, **not** comma-joined) |
| `--profile <name>` | Layer `$CODEX_HOME/<name>.config.toml` over the base config |
| `--ephemeral` | Run without persisting a session file to disk |

## Useful subcommands

- `codex review [--uncommitted | --base <branch> | --commit <sha>]` — native structured code review that discovers the diff itself and returns `[P#] file:line` findings. Optional context-gatherer for [[codex-review]] when you'd rather not paste a full diff.
- `codex exec resume <SESSION_ID>` — continue a previous exec session, keeping context across rounds. Get the id from the first `--json` event (`thread.started` → `thread_id`). Prefer the id over `--last`, which grabs whatever ran most recently. `resume` has no `--cd`/`--sandbox`: run it from the repo and pin `-c sandbox_mode="read-only"`.
- `codex debug models` — dump the **live** model catalog as JSON. The catalog is server-fetched and `~/.codex/models_cache.json` goes stale; this is the only reliable way to see the current lineup and each model's supported efforts. `--bundled` shows only what ships in the binary.
- `codex features list` — effective state of every feature flag (e.g. `multi_agent_v2`, `image_generation`).

## References

- [CLI flags](references/cli-flags.md)
- [Config](references/config.md)
- [Prompt checklist](references/prompt-checklist.md)

## Specialized skills built on this

- [[peloteo]] — plan → Codex review → implement → Codex review → PR, without the human in the loop
- [[codex-review]] — pre-commit / PR code review
- [[codex-consult]] — multi-round pair-programming-for-thinking

## Minimal example

```bash
codex exec \
  --cd /path/to/repo \
  --sandbox read-only \
  "Analyze the authentication flow. Identify security concerns and suggest improvements." \
  < /dev/null
```
