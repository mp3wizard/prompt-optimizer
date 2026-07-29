# Prompting a Multi-Model Agent Stack

Patterns for the orchestration shape where **one model plans and judges** and **other models do the typing**:

- **Controller** — Claude Opus 5. Holds the goal, decides scope, writes the briefs, verifies the results.
- **Claude workers** — Claude Sonnet 5 subagents. Share the harness and tool surface; inherit no conversation context beyond the brief you write.
- **External workers** — a Codex CLI executor (or any non-Claude coding agent). Separate process, separate sandbox, **zero** conversation context.

Optimizing prompts for this stack is a different job from optimizing a single prompt. Three prompts exist per unit of work, and they fail in different ways:

| Prompt | Written by | Fails as |
|---|---|---|
| Controller prompt | You | Over-delegation, scope drift, trusting worker reports |
| Worker brief | The controller | Under-specified, context-assumed, unverifiable |
| Verification prompt | The controller | Rubber-stamping, re-running nothing |

---

## 1. The delegation gate belongs in the controller prompt

Opus 5 delegates **more readily** than Opus 4.7/4.8 did. That reverses the prior generation's tuning: 4.7 needed encouragement to fan out, Opus 5 needs a ceiling. Every subagent re-establishes context, re-explores, reports back, and then the controller re-reads the report — the overhead is real and multiplies.

Put an explicit gate in the controller prompt:

```text
## Delegating

Subagents multiply cost and time: each one re-establishes context, re-explores, and
reports back, and you then re-read its report. Delegate only when the payoff clearly
exceeds that overhead.

Delegate when:
- The same routine operation repeats across many independent units (N files, N records,
  N test cases). The win scales with N and one brief covers every unit.
- Genuinely independent tracks can run in parallel (unrelated modules, a wide
  multi-file investigation).

Do NOT delegate:
- Work you could finish in a handful of tool calls yourself.
- A single self-contained artifact, when you already hold the context to write it.
- Design, architecture, ambiguous scope, or anything needing taste or product judgment.
- Verification. Verification belongs in your own loop.

Prefer one subagent over several. Keep spawn counts low. If you launch several agents
for independent work, send them in a single message so they run concurrently.
```

The three "do NOT" lines carry most of the value. The most common waste in this shape is delegating a single file the controller could have written directly, then spending more tokens reviewing the result than writing it would have cost.

---

## 2. Write worker briefs as if the worker has amnesia — because it does

A Sonnet 5 subagent sees your brief and its own tools. A Codex executor sees even less: a different process, a different working directory, and none of the conversation. Anything the brief does not say does not exist.

**Every brief carries five things:**

```text
Task:        [what to do, completely — no "as discussed", no "the file we talked about"]
Paths:       [absolute paths to every file it must read or write]
Constraints: [scope boundary, style, what NOT to touch]
Done means:  [the exact observable that proves success — a command that exits 0,
              a file that exists with property X, a test name that passes]
Report:      [what to hand back, and to say plainly if a step was refused or skipped]
```

`Done means` is the field people drop, and it is the one that makes verification possible. "Refactor the auth module" is unverifiable. "`pytest tests/auth -q` exits 0 and `grep -c 'legacy_token' src/auth/` returns 0" is verifiable by the controller without reading a line of the diff.

### Sonnet 5 workers: state the scope, it will not generalize

Sonnet 5 follows instructions **literally** and does not silently widen an instruction from one item to the neighbouring ones. In a single prompt that reads as precision; in a delegated brief it reads as a worker that fixed exactly the one case you named and left eleven identical ones alone.

```text
Apply this to every handler in the directory, not only the first one.
```

Effort defaults to `high`. Raise to `xhigh` for the hardest coding work. If a worker returns shallow reasoning, **raise effort rather than adding prose telling it to think harder** — the effort parameter is the working lever and prompt nudges are a poor substitute for it.

### Codex-style external workers: design around the process boundary, not the model

Two failure modes here are environmental, not quality problems, and no amount of prompt polish fixes them:

1. **The sandbox is pinned to its working directory.** A brief that tells it to write into another tree fails at the permission boundary having done nothing. Either `cd` into the target tree before invoking it, or have it write inside the current workspace and move the artifact yourself. State the absolute path in the brief and tell it to say so explicitly if the write is refused.

2. **The wrapper's status is not evidence.** Forwarding wrappers report ambiguously — "forwarded to background", "exceeded the timeout, moved to background" — for work that is finished, and sometimes report completion for work that never ran. **Check the file on disk. Run the thing yourself.** Never let a wrapper's reply close a task.

Third, a prompting fix that does work: **say where a caution does not apply.** External executors generalize warnings. Told "never pass `--fast` to service A, it crashes", an executor will also strip `--fast` from unrelated service B — silently degrading it while reporting success. Scope every negative constraint:

```text
Do not pass --fast to service-a (it crashes). This applies ONLY to service-a —
leave the flags on every other service exactly as they are.
```

---

## 3. Do not put verification instructions in the controller prompt

Opus 5 verifies its own work without being told. Instructions that *tell* it to verify — "include a final verification step", "double-check your answer", "use a subagent to verify" — now cause over-verification: redundant passes, extra subagents, tokens spent re-confirming what was already confirmed.

**This inverts a standard prompting best practice.** "Ask the model to self-check" is generally sound advice and is wrong here. If your prompt library applies it as a global rule, carve out an exception for this model rather than propagating it.

The fix is a deletion, not a rewrite:

- Remove "verify your work" / "double-check" lines from the controller prompt.
- Remove separate harness-level verification steps carried over from prior models.

What stays is verification of the **workers**, which is a different thing — the controller checking a claim it did not produce. That is not self-checking and should not be removed:

```text
A worker's report is a set of claims, not evidence. Before accepting a result:
run the command it says passes, read the diff it says it made, and confirm the
files it says exist. A green targeted check with a broken build is a failed
verification.
```

---

## 4. Keep the controller's narration readable

Opus 5 narrates more than prior models between tool calls, and its per-message output in agentic sessions runs longer. In an orchestration session the user is often away — the controller's summary is the first and only thing they read.

```text
Default to silence between delegations. Write text when you find something, change
direction, or hit a blocker — one sentence each. Do not narrate routine actions
("Now I'll...", "Let me check...").

Lead with the outcome. Your first sentence after finishing answers "what happened" —
the thing the user would ask for if they said "just give me the TLDR". Detail comes
after.

The vocabulary you built up while working is yours, not the reader's. Drop the working
shorthand in the summary: complete sentences, terms spelled out, no arrow chains or
labels you invented earlier.
```

Also worth curbing: Opus 5 flags and explains its own earlier mistakes at length, which reads as thrash across a long orchestration run.

```text
Only correct an earlier statement when the error changes the user's decisions. State
corrections plainly and move on. Do not apologize, ruminate, or tally past errors.
Workers sometimes report incorrect results — do not take them at face value, but when
you correct course, do it without narrating the correction at length.
```

---

## 5. Scope discipline across the whole run

Opus 5 can add steps nobody asked for, or apply its own judgment about what the task should be without saying so. Across a delegated run this compounds: the controller widens the goal, then briefs three workers against the widened version.

```text
Deliver what the user asked for, at the scope they intended. Make routine judgment
calls yourself; check in only when different readings lead to materially different
work. If you think the ask is mistaken or a better approach exists, say so in a
sentence and keep going with the task as asked — do not quietly narrow, widen, or
transform it. Finish the whole task, not just the easy part; report completion only
when it is fully done. If you genuinely cannot complete something, do the rest and
state plainly what is missing and why.
```

The "finish the whole task / report completion only when done" clause is doing separate work from the scope clause — it covers premature "done" claims, which scope wording alone does not.

---

## 6. Effort routing across the stack

Effort is the primary cost lever in an orchestration shape, because it applies per agent and the agent count multiplies it.

| Role | Start at | Notes |
|---|---|---|
| Controller (Opus 5) | `high` | Planning and judging. `xhigh` only for genuinely hard architecture decisions. |
| Worker — hard coding (Sonnet 5) | `xhigh` | The hardest coding and agentic work. |
| Worker — routine edits (Sonnet 5) | `high` → sweep down | Default. `medium` ≈ Sonnet 4.6 at `high`. |
| Worker — mechanical fan-out | `low` / `medium` | Both models are unusually strong at the low end; re-check quality on your own evals before assuming you need more. |

Effort defaults carried over from a prior model generation are rarely the right setting. Sweep on your own evals rather than inheriting.

**Effort does not control visible response length** on Opus 5 — lowering it to get shorter output does not reliably work. Use an explicit conciseness instruction for length and effort for depth.

---

## 7. Anti-patterns specific to this shape

| Anti-pattern | Why it hurts | Fix |
|---|---|---|
| "Delegate aggressively for speed" in the controller prompt | Opus 5 already over-delegates; this compounds cost | Ceiling, not encouragement (§1) |
| Brief that references the conversation ("the file we discussed") | Worker has no conversation | Absolute paths, self-contained brief (§2) |
| Accepting a wrapper's "done" | Wrappers report status ambiguously | Check disk, re-run the check (§2, §3) |
| "Verify your work carefully" in the controller prompt | Causes over-verification on Opus 5 | Delete it; verify *workers* instead (§3) |
| Unscoped negative constraint in a worker brief | External executors generalize cautions | Say where the caution does not apply (§2) |
| Same effort for every agent | Multiplies cost across the fleet | Route effort by role (§6) |
| Controller re-doing a worker's task after reading its report | Pays for the work twice | "If you delegate, commit to the delegation" (§1) |

---

## 8. Checklist before running an orchestrated task

- [ ] Controller prompt has a delegation **ceiling**, not encouragement
- [ ] Controller prompt has **no** self-verification instruction
- [ ] Controller prompt has a scope-discipline clause including "finish the whole task"
- [ ] Every worker brief is self-contained: absolute paths, no conversation references
- [ ] Every worker brief has a `Done means` line that names an observable
- [ ] Negative constraints in briefs state where they do **not** apply
- [ ] Sonnet 5 briefs state scope explicitly ("every handler, not only the first")
- [ ] Effort routed per role, not uniform
- [ ] Verification re-runs the claimed check rather than reading the report
