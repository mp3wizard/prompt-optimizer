# prompt-optimizer

Prompt optimization and composition for Claude's 5-generation models, as an installable Claude Code plugin.

Most prompt-engineering advice was written for the 4.x generation and **inverts** on the 5 generation. This plugin encodes what changed.

## What it covers

| Area | What it does |
|---|---|
| **Model-specific tuning** | Opus 5.5, Sonnet 5.5, Fable 5 — and legacy 4.x, marked as legacy |
| **Opus vs Sonnet divergence** | Where Opus 5.5 and Sonnet 5.5 need opposite edits (self-verification, "think" lines, tool eagerness, vision, thinking off) |
| **Inverted patterns** | Flags 4.x advice that now makes prompts *worse* (see below) |
| **Agent orchestration** | Controller prompts, worker briefs, and verification for multi-model stacks — including non-Claude executors |
| **Deconstraint passes** | Rightsizing system prompts, skills, and `CLAUDE.md` — where the correct edit is a deletion |

## The patterns that invert

Reusing a 4.x snippet on a 5-generation model is the most common way to make a prompt worse:

| Pattern | Opus 4.7 / 4.8 | Opus 5.5 |
|---|---|---|
| Subagents | Under-delegates → **encourage** | Over-delegates → **cap** |
| Tool use | Reaches for tools less → **push** | Reaches readily → leave alone |
| Self-verification | Helps | **Causes over-verification → delete** |
| Progress updates | May need forcing | Writes its own (as `thinking` blocks) → **remove scaffolding, render `display: "updates"`** |
| Rules vs judgment | Explicit rules help | Over-constraint hurts → **deconstrain** |

The self-verification row is the counterintuitive one: "ask the model to double-check its work" is standard advice, and it is wrong on Opus 5.5. It is *right* on Sonnet 5.5 at `low` effort, which can report a code change done without running a check — one of several places the two 5.5 models need opposite edits.

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

- "Optimize this system prompt for Opus 5.5"
- "Migrate my Opus 5 agent prompt to Opus 5.5 — it stops partway through unattended runs"
- "My Sonnet 5.5 JSON answers are wrong / don't parse"
- "My CLAUDE.md has grown across three model upgrades — rightsize it"
- "Write the controller prompt and worker brief for an Opus 5.5 → Sonnet 5.5 stack"
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

## Changelog

**3.0.0** — Claude Sonnet 5.5 support; Opus 5 and Sonnet 5 dropped as targets. From Anthropic's *Prompting Claude Sonnet 5.5* guide and the Sonnet 5.5 migration guide:

- New Sonnet 5.5 section: request-level 400s (`between_tools` replaces `disabled`, no forced `tool_choice`, append-only history), recalibrated effort, initiative and scope clauses, JSON reasoning tasks, progress updates, search in chat, mid-turn user messages, real-check verification, tolerant tool calls, vision tools, refusals
- New "Opus 5.5 vs Sonnet 5.5" divergence table
- Opus 5 and Sonnet 5 sections removed; their patterns that Anthropic's 5.5 guides keep as the starting point are folded in as "Inherited from Opus 5" / "Inherited from Sonnet 5". Migration rows from Opus 5, Sonnet 5, Opus 4.8, and Sonnet 4.6 point at the 5.5 models
- SKILL.md playbooks, quick reference, orchestration (Sonnet 5.5 workers), and examples updated

**2.1.0** — Claude Opus 5.5 support, from Anthropic's *Prompting Claude Opus 5.5* guide and the Opus 5.5 migration guide:

- New "Opus 5.5 — Delta over Opus 5" section: request-level 400s (thinking cannot be disabled, no forced `tool_choice`), effort recalibration (default `medium`), migrating thinking-disabled prompts, unattended-run continuation, progress updates in `thinking` blocks, multi-app exploration, time budgets for agent teams, chat thinking instructions, `<pasted_content>` marking, safeguard refusals, vision, and frontend defaults
- Opus 5 advice that changes on 5.5 is now scoped to Opus 5 (the `xhigh` effort start, thinking-disabled repairs, manual CoT); "remove progress scaffolding / default to silence" is qualified with a rendering check first
- Opus 5.5 playbook, quick-reference rows, orchestration guidance (controller effort, `end_turn` handling, time signals), and a worked example of an unattended migration agent

**2.0.0** — 5-generation rewrite: Opus 5, Sonnet 5, Fable 5, agent orchestration, deconstraint passes.

To update an existing install:

```bash
claude plugin marketplace update prompt-optimizer
claude plugin update prompt-optimizer@prompt-optimizer
```

## Sources

Guidance is derived from Anthropic's official prompting documentation for Claude Opus 5.5, Sonnet 5.5, and Fable 5 (plus the Opus 5 and Sonnet 5 guides they build on), plus the published context-engineering guidance for the 5 generation. Model-behaviour claims trace to those sources rather than to folklore.

## License

MIT
