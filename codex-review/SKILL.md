---
name: codex-review
description: Deep code review with Codex (the native `codex review` subcommand or `codex exec`) before commits — bugs, security, design. Use when the user wants Codex specifically to review a diff/change ("revísalo con codex", "pásale codex antes de commitear", "codex review this"). For a review that does NOT ask for Codex, use the built-in code-review skill instead.
---

# Code Review with Codex

He thinks. You fix. Two paths — pick by scope.

> Invocation, flags, model/effort selection — see [[codex-headless-usage]].
> Review is *detection*, and detection is saturated: every non-luna run (90, two
> campaigns) found the planted defect at every effort from `low` up. **Don't pin an
> effort here** — the `low` config default is already enough. See the effort-routing table in [[codex-headless-usage]].

## Default: native `codex review` (delta review)

For pre-commit and PR review, let Codex discover and review the diff itself — no
pasting:

```bash
codex review --base develop    # vs branch of record (PR review)
codex review --uncommitted     # staged + unstaged + untracked (pre-commit)
codex review --commit <sha>    # a specific commit
```

It runs git/grep/read on its own (read-only) and returns structured
`[P#] file:line` findings, most-severe first.

**No custom prompt with a scope flag.** On codex-cli 0.160, `--uncommitted`,
`--base` and `--commit` each refuse a `[PROMPT]` ("cannot be used with
'[PROMPT]'"). Close stdin (`< /dev/null`) and run it bare; when you need to steer
the emphasis, use the deep path below with `codex exec`.

Scope: this reviews the **delta only**, not pre-existing code. It shells out
through your login shell, so local warnings (fnm, etc.) may appear mid-stream; the
findings block at the end is clean.

## Deep path: prompt-based review (whole-file / architectural / non-git)

When you need more than a delta — a whole module, an architectural concern, a
*design*, or code that isn't a git diff — drive `codex exec` with an explicit
checklist and point it at files (or paste the code):

```
Review the following thoroughly. Be critical; I want issues before committing.

Code / files:
<paths or code>

Context:
<what this is meant to do>

Check for:
1. Correctness — logic errors, off-by-one, null/undefined handling, type mismatches
2. Security — injection, auth/authz, data exposure, input validation
3. Performance — needless computation, N+1 queries, leaks, blocking ops
4. Design — fits codebase patterns? abstraction level right? coupling concerns?
5. Edge cases — boundaries, empty/null inputs, concurrency, error paths

Be specific. Reference line numbers. Explain why each issue matters.
```

Bump to `xhigh` here **only** when you want Codex to *reason through* a hard design
tradeoff — that buys depth of explanation, not more findings, at ~3× latency. If
the review comes back thin — or the change matters — the productive move is a
second pass with `-c model="gpt-6-astra"`. More effort on the same model buys nothing.

> `codex review` has **no `--model` flag** — steer it with `-c model=…`.

## Interpreting results

For each finding: agree → fix before committing; disagree → challenge him with your
reasoning; unclear → ask. Engage, don't rubber-stamp.

## Pre-commit workflow

1. Make your changes (staged or not).
2. `codex review --uncommitted` (or `--base <branch>` for a PR).
3. Address findings; re-run if the change grew significantly.
4. The commit is yours to make — never commit on the user's behalf unless explicitly asked.
