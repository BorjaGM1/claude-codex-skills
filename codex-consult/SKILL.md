---
name: codex-consult
description: Multi-round discussion with Codex for complex problems. Pair programming for thinking — iterate until consensus, then present to user.
---

# Consult Codex

## Purpose

When you're stuck or facing something complex, discuss with Codex:
- Architecture decisions
- Implementation approaches
- Trade-off analysis
- Debugging strategies
- Design patterns

This is pair programming for thinking.

> Invocation, flags, model selection, stdin gotcha, background execution — see [[codex-headless-usage]].

## Effort and model

**Don't tune effort between rounds** — it buys no findings (low ≈ … ≈ xhigh,
measured) and the `low` config default is right for every round. Background a
round only when the target is a whole repo.

Tune the **model** instead:

- **Default rounds** → `gpt-6.1-sol` (config default).
- **Stress-testing a proposal** ("would this actually hold?") → `-m gpt-6-astra
  -c model_reasoning_effort=medium`. It is the one that *says* when a fix is incomplete.

## The process

### Round 1: Initial consultation

Prepare a well-designed prompt:
```
I need your help thinking through <problem>.

Context:
<full context — code, constraints, requirements>

Current thinking:
<your current approach or where you're stuck>

Questions:
<specific things you want explored>

Take your time. Think deeply.
```

### Round 2+: Challenge and refine

Review Codex's response critically. Then:
```
Following up on your analysis.

I have questions:
1. <something that doesn't fully click>
2. <alternative you want explored>
3. <edge case he might have missed>

Previous context: <summary of discussion so far>
```

### When to stop

- **Minimum**: 2–3 rounds
- **Stop when**: you reach consensus, or returns are diminishing
- **Don't stop just because**: first answer sounds good

### Present to user

Once you and Codex agree:
```
After discussing with Codex (N rounds), here's our recommendation:

**Summary**: <one paragraph>

**Approach**: <the agreed solution>

**Key considerations**:
- <trade-off 1>
- <trade-off 2>

**Codex's reasoning**: <brief summary of his analysis>

**My assessment**: <your take on his reasoning>
```

## Example prompts

### Architecture decision
```
Deciding between approaches for <feature>.

Option A: <description>
Option B: <description>

Codebase context:
<relevant code snippets or structure>

Constraints:
- <constraint 1>
- <constraint 2>

What are the trade-offs? Which do you recommend and why?
```

### Debugging complex issue
```
Stuck debugging <issue>.

Symptoms:
<what's happening>

What I've tried:
<debugging steps taken>

Relevant code:
<code snippets>

Help me think through possible causes.
```

### Implementation approach
```
Need to implement <feature>.

Requirements:
<requirements>

Current codebase:
<relevant existing code>

How would you approach this? Walk me through your thinking.
```

## Key principles

1. **Prepare good prompts** — garbage in, garbage out
2. **Include full context** — Codex can't see what you don't share
3. **Challenge his thinking** — don't accept first answers blindly
4. **Iterate** — real pair programming takes multiple rounds
5. **Synthesize** — present a unified recommendation to user
