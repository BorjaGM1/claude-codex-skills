# Prompt checklist for delegation

Before handing a task to Codex headless, confirm the prompt covers:

- [ ] Working directory and repo root (`--cd`)
- [ ] Files or folders in scope (referenced explicitly in prompt)
- [ ] Read-only vs. edit allowed (`--sandbox read-only` is the default)
- [ ] Tests to run or skip
- [ ] Style or linting requirements
- [ ] Dependency constraints (e.g. "don't add new deps")
- [ ] Output format (plain text, markdown, or JSON via `--output-schema`)
- [ ] Sandbox and approval policy (`--ask-for-approval`)
- [ ] Time or token budget (mention "take your time" or a soft cap)
- [ ] Reasoning level (`low` is the config default and **effort buys no findings** — don't pin higher; `max`/`ultra` add nothing, and `ultra` does not delegate)
- [ ] Stdin closed (`< /dev/null` in bash, `$null |` in PowerShell)
- [ ] Background execution for whole-repo targets or any `xhigh`+ pin (`run_in_background: true` on Bash tool). Small targets at `low` finish in about a minute.

Garbage in, garbage out. Codex can't see what you don't share — include full context.
