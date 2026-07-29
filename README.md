# prompt-optimizer

Prompt optimization and composition for Claude's 5-generation models, as an installable Claude Code plugin.

Most prompt-engineering advice was written for the 4.x generation and **inverts** on the 5 generation. This plugin encodes what changed.

## What it covers

| Area | What it does |
|---|---|
| **Model-specific tuning** | Opus 5, Sonnet 5, Fable 5 — and legacy 4.x, marked as legacy |
| **Inverted patterns** | Flags 4.x advice that now makes prompts *worse* (see below) |
| **Agent orchestration** | Controller prompts, worker briefs, and verification for multi-model stacks — including non-Claude executors |
| **Deconstraint passes** | Rightsizing system prompts, skills, and `CLAUDE.md` — where the correct edit is a deletion |

## The patterns that invert

Reusing a 4.x snippet on a 5-generation model is the most common way to make a prompt worse:

| Pattern | Opus 4.7 / 4.8 | Opus 5 |
|---|---|---|
| Subagents | Under-delegates → **encourage** | Over-delegates → **cap** |
| Tool use | Reaches for tools less → **push** | Reaches readily → leave alone |
| Self-verification | Helps | **Causes over-verification → delete** |
| Progress updates | May need forcing | Narrates on its own → **remove scaffolding** |
| Rules vs judgment | Explicit rules help | Over-constraint hurts → **deconstrain** |

The self-verification row is the counterintuitive one: "ask the model to double-check its work" is standard advice, and it is wrong on Opus 5.

## Install

```bash
claude plugin marketplace add mp3wizard/prompt-optimizer
claude plugin install prompt-optimizer@prompt-optimizer
```

Or from a local checkout:

```bash
claude plugin marketplace add ./prompt-optimizer
claude plugin install prompt-optimizer@prompt-optimizer
```

## Use

The skill triggers on its own when you ask to improve, optimize, analyze, or write a prompt. You can also invoke it directly:

```
/prompt-optimizer
```

Typical asks it handles:

- "Optimize this system prompt for Opus 5"
- "My CLAUDE.md has grown across three model upgrades — rightsize it"
- "Write the controller prompt and worker brief for an Opus 5 → Sonnet 5 stack"
- "Why is my agent spawning so many subagents?"

## Layout

```
skills/prompt-optimizer/
  SKILL.md                          entry point
  references/
    model-techniques.md             per-model patterns + context engineering
    agent-orchestration.md          controller / worker / external executor
    examples.md                     before-after optimization examples
```

## Sources

Guidance is derived from Anthropic's official prompting documentation for Claude Opus 5, Sonnet 5, and Fable 5, plus the published context-engineering guidance for the 5 generation. Model-behaviour claims trace to those sources rather than to folklore.

## License

MIT
