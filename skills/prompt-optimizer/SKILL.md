---
name: prompt-optimizer
description: Advanced prompt optimization and composition system for all prompt types (system prompts, task-specific, creative, technical, agentic, multi-agent orchestration). Use when users need to refine, optimize, or transform prompts to achieve better AI outputs. Triggers include requests to improve prompts, make prompts more effective, analyze prompt quality, create prompts from scratch, optimize existing prompts for specific goals, rightsize a system prompt or CLAUDE.md, write controller/worker prompts for an agent stack, or optimize for specific Claude models (Opus 5.5, Sonnet 5.5, Fable 5, and legacy 4.x). Also use when users share prompts that could be improved or ask for prompt engineering guidance.
---

# AI Best Prompt Optimizer and Composer

Transform prompts from functional to exceptional through deep analysis, strategic optimization, and iterative refinement. Apply "ultrathink" principles—questioning assumptions, obsessing over details, and iterating relentlessly.

## Philosophy

Great prompts feel inevitable—the only right way to ask. Achieve this by:

- **Question every assumption**: Why this structure? What if we started from zero?
- **Obsess over details**: Every word choice matters
- **Plan before writing**: Understand the goal deeply first
- **Iterate relentlessly**: First version is never final
- **Simplify ruthlessly**: Remove complexity without losing power

## Core Framework

### 1. Deep Understanding Phase

Before optimization, understand:

- What is the desired output format and quality?
- Who is the audience? What's their expertise level?
- What context is essential vs. nice-to-have?
- What are the failure modes to prevent?
- Are there implicit assumptions that should be explicit?
- **Target model?** (Opus 5.5 / Sonnet 5.5 / Fable 5 and legacy 4.x behave differently — see `references/model-techniques.md`)
- **Effort setting?** For API use cases, `max`/`xhigh`/`high`/`medium`/`low` changes optimal prompt density
- **Single prompt or an agent stack?** If a controller model briefs worker models, see `references/agent-orchestration.md` — three prompts exist per unit of work, and they fail differently

Analyze current prompt for: clarity, completeness, ambiguity, missing constraints, structure, token efficiency, and model-fit.

**Also analyze for over-constraint.** On 5-generation models the most common defect in an existing prompt is not a missing instruction but too many, conflicting: absolute rules that are not absolutely true, the same constraint repeated across layers, `CRITICAL`/`YOU MUST` emphasis, and self-check scaffolding. Optimization is frequently a **deletion**.

### 2. Strategic Optimization by Prompt Type

**System Prompts (AI Agents/Chatbots):**
Define role/persona, establish guardrails, specify output format/tone, include response patterns and examples, add error handling, balance flexibility with consistency.

**Task-Specific Prompts:**
Break down complex tasks, provide input/output examples, specify success criteria, include what NOT to do (or better: what TO do). Define success criteria as observables the *caller* can check — on Opus 5.5, do not add "verify your work" instructions for the model itself (Sonnet 5.5 at `low` effort is the exception: give it a concrete real-check instruction).

**Creative Prompts:**
Set direction and constraints, inspire without over-constraining, provide style references, balance guidance with freedom, include quality markers.

**Technical Prompts:**
Maximize precision, include technical context, specify format strictly, add validation criteria, consider edge cases, provide technical examples.

**Agentic Prompts:**
See `references/model-techniques.md` for comprehensive patterns including tool usage, parallel execution, state tracking, multi-context workflows, and model-specific behavior tuning.

**Orchestration Prompts (controller + workers):**
A controller model that briefs worker models needs three prompts per unit of work — the controller prompt, the worker brief, and the verification step — each with its own failure mode. See `references/agent-orchestration.md`.

**Standing Context (`CLAUDE.md`, skills, tool descriptions):**
Distinct from the system prompts above: context is reused across many requests and cannot be as specific as a prompt. Optimizing it means *removing* accumulated over-constraint as much as adding guidance — see "Context Engineering" in `references/model-techniques.md`.

### 3. Structural Excellence

**Progressive disclosure:** Overview → specific instructions → examples/constraints

**Layered specificity:** General principles → specific requirements → edge cases

**XML structure:** Wrap distinct content types in descriptive tags (`<instructions>`, `<context>`, `<input>`, `<example>`) — especially important when prompts mix instructions, context, examples, and variables.

**Long context (20k+ tokens):** Place longform data near the top, above queries, instructions, and examples. Wrap documents in `<documents>` / `<document index="n">` / `<document_content>` / `<source>`. For complex document tasks, ask Claude to quote relevant excerpts first before answering.

**Token efficiency:** Every sentence must justify its existence. Prefer concise clarity.

### 4. Quality Markers

Excellent prompts exhibit:

- **Clarity**: No ambiguity in what's expected
- **Completeness**: All necessary context provided
- **Conciseness**: No unnecessary verbosity
- **Specificity**: Concrete rather than abstract (state scope explicitly — Sonnet 5.5 will not silently generalize an instruction to neighbouring items)
- **Structure**: Logically organized and scannable
- **Examples**: Show, don't just tell (3–5 diverse examples, wrapped in `<example>` tags)
- **Constraints**: Clear boundaries and guardrails
- **Testability**: Output quality can be verified
- **Motivation**: Explain the WHY behind non-obvious instructions

## Optimization Process

### Step 1: Analyze
- Identify core objective
- Identify target model and effort level
- Note strengths and weaknesses
- Spot missing elements
- Find improvement opportunities
- Find **removal** opportunities — conflicting rules, duplicated constraints, aggressive emphasis, self-check scaffolding
- Check which model-specific patterns apply

### Step 2: Plan
- Determine optimization level (quick polish vs deep transformation)
- Identify key improvements **and deletions**
- Plan new structure
- Select relevant patterns per target model — and check whether any inherited 4.x pattern **inverts** on the target

### Step 3: Optimize
- Rewrite unclear instructions
- Add missing context and motivation
- Reorganize for clarity
- Insert concrete XML-wrapped examples
- Refine word choices
- State scope explicitly where the model might otherwise narrow it
- Delete over-constraint the target model no longer needs
- Apply model-specific techniques

### Step 4: Validate
- Does it meet all quality markers?
- Will it produce desired output on the target model?
- Are edge cases handled?
- Is it token-efficient?
- Can it be simplified further?

### Step 5: Present
- Show optimized prompt
- Explain key changes
- Describe expected improvements
- Highlight model-specific optimizations applied and the model they target
- Call out anything **removed** and why (deletions are easy to mistake for omissions)

## Output Format

When presenting optimized prompts:

### 1. Optimized Prompt
Complete, ready-to-use prompt with clear formatting.

### 2. Key Improvements
- What changed and why
- Expected impact on outputs
- Trade-offs made
- Model-specific techniques applied (if any) and the target model
- Anything deliberately **removed**, and why

### 3. Expected Output Changes
Quality enhancements, consistency improvements, better edge case handling.

### 4. Implementation Notes (if applicable)
Variations for different contexts, customization tips, model-specific considerations (effort level, thinking config, prefill migration).

## Adaptive Optimization Levels

**Quick Polish (~10-30% transformation):**
Fix clarity issues, add missing constraints, improve structure, refine word choices.

**Balanced Enhancement (~30-60% transformation):**
Restructure for clarity, add examples and context, enhance specificity, apply quality markers, add relevant model-specific patterns, strip conflicting or duplicated constraints.

**Deep Transformation (~60-100% transformation):**
Question core approach, redesign from principles, add comprehensive framework, include extensive XML-structured examples, fully leverage the target model's capabilities.

**Deconstraint Pass (net reduction):**
A distinct level, and the right one for a system prompt, skill, or `CLAUDE.md` written for an earlier model generation. The output is *shorter* than the input: remove absolute-but-untrue rules, cross-layer duplication, `CRITICAL`/`YOU MUST` emphasis, forced-progress scaffolding, and self-verification instructions (except a concrete real-check line for Sonnet 5.5 at `low` effort). Anthropic removed over 80% of Claude Code's system prompt this way for Opus 5 and Fable 5 with no measurable loss on coding evals.

## Model-Specific Optimization

Current Claude models share **common principles** plus **model-specific behaviors** needing targeted tuning.

**Common principles (all current models):**

1. **Be explicit.** The model follows precisely what's asked. If you want "above and beyond" behavior, explicitly request it.
2. **Add context/motivation.** Explain WHY behind instructions — the model generalizes from purpose where a bare rule does not.
3. **Be vigilant with examples.** Examples strongly influence behavior. Use `<example>` / `<examples>` XML tags; 3–5 diverse ones.
4. **Prefer positive instructions.** "Use flowing prose paragraphs" beats "don't use markdown."
5. **Use XML structure** for complex, mixed-content prompts.
6. **Put longform data at the top**, queries and instructions at the end. On complex multi-document inputs this is worth up to ~30% on response quality.
7. **Don't over-constrain.** On 5-gen models, conflicting and duplicated rules cost more than they buy — state the goal and let the model judge.

### ⚠️ Patterns that invert between generations

Reusing a 4.x snippet on a 5-gen model is the most common way to make a prompt worse. Check these before carrying anything forward:

| Pattern | Opus 4.7 / 4.8 | **Opus 5.5** |
|---|---|---|
| Subagents | Under-delegates → **encourage** | Over-delegates → **cap** |
| Tool use | Reaches for tools less → **push** | Reaches readily → leave alone |
| Self-verification | Helps | **Causes over-verification → delete** |
| Progress updates | May need forcing | Writes its own (as `thinking` blocks) → **remove scaffolding, render `display: "updates"`** |
| Rules vs judgment | Explicit rules help | Over-constraint hurts → **deconstrain** |

The Opus 5.5 column is inherited from Opus 5, which the Opus 5.5 guide names as the starting point.

**Opus 5.5 and Sonnet 5.5 also diverge from each other.** Sonnet 5.5 at `low` *needs* a concrete real-check instruction (the opposite of Opus), benefits from "think the problem through" on JSON reasoning tasks (Opus 5.5: remove "think carefully" lines), under-searches in chat (remove "minimize tool calls"), and gains more from vision tools than from effort. Full table: "Opus 5.5 vs Sonnet 5.5" in `references/model-techniques.md`.

### Model-specific behavior shifts

- **Opus 5.5** — Inherits Opus 5's behaviour as the starting point: verbose (prompting, not effort, controls visible length), long deliverables, **over-verifies** if told to verify, can expand scope, **over-delegates**, narrates self-corrections, follows code-review severity filters literally. New on 5.5: 30%+ faster output, fewer tokens per task, $4 / $20 per M. **Thinking always on** — `disabled`/`budget_tokens` return 400; so do forced `tool_choice` (`any`/`tool`). **Effort default `medium`**, which matches or beats Opus 5 at `high` on coding and knowledge-work evals; it thinks more per level, so an inherited `xhigh` costs more — start `medium`, sweep, lower effort before prompting for less thinking. Text between tool calls arrives as **`thinking` blocks** (empty by default; `display: "updates"` returns them alone, `"summarized"` mixes them with reasoning). Unattended loops stop early on text-only `end_turn` → treat it as a report and add continuation logic. Asking for reasoning in the response can be refused (`reasoning_extraction`). Responds strongly to elapsed-time/budget signals, to "explore broadly before acting" in multi-app work, and to `<pasted_content>` marking. Remove chat "think carefully" lines. Vision is strong without tools → re-test scaffolding. Frontend: name the specific default styles to avoid.
- **Sonnet 5.5** — Inherits Sonnet 5's behaviour as the starting point: **literal** instruction following (state scope), sampling params rejected (variety via propose-then-pick), tokenizer ~30% above Sonnet 4.6. New on 5.5: **effort levels recalibrated** — `high` default; `medium` for well-specified agentic work; `medium`/`low` for chat (from `medium` up it thinks before almost every reply). Thinking off = **`between_tools`** (`disabled` is a 400; only at `high` or below, no other fields, no per-message effort change; drop "don't think" rules). Forced `tool_choice` = 400; history must stay append-only (signed thinking). At `low`/`medium` it checks in early on agentic coding; at `low` it can skip verification → add a **real-check** instruction; at `xhigh`/`max` it starts its own review rounds and reviewer subagents → add "stop when checks pass". Adds unrequested tests/docs at every level. JSON reasoning tasks: structured outputs + "Think the problem through before you answer.", treat `max_tokens` stops as failed, parse the last JSON value. Under-searches in chat → remove tool-discouraging language. Treats user text placed after tool results as possible injection. Occasionally mis-cases tool names → tolerant harness. Wider refusal set (`frontier_llm`, `general_harms`). Charts: crop/zoom tools beat effort.
- **Fable 5** — Thinking always on, cannot be disabled; raw chain of thought never returned. Long turns (minutes). **De-prescribe migrated prompts** — 4.x step-by-step scaffolding reduces its output quality. Benefits from a memory file and from asynchronous sub-agents.
- **Opus 4.8 / 4.7 (legacy)** — Under-delegate; need tool/subagent encouragement. 4.7 has a strong cream/serif design house style.
- **Opus 4.6 (legacy)** — Overtriggers tools/skills, overengineers, overthinks, overuses subagents. Prefills deprecated. Defaults to LaTeX.
- **Sonnet 4.6 / Haiku 4.5 (legacy)** — Default `effort=high`; Haiku is speed-first and usually needs minimal prompting.

**For complete patterns including:**

- Context engineering for 5-gen (what to cut from system prompts, skills, `CLAUDE.md`)
- Effort parameter tuning (`max` / `xhigh` / `high` / `medium` / `low`) and per-role routing
- Thinking configuration and `display`; `between_tools` on Sonnet 5.5
- Opus 5.5: request-level 400s, effort recalibration, unattended-run continuation, progress updates in `thinking` blocks, multi-app exploration, time budgets, settled chat answers, pasted-content marking, safeguard refusals, plus the patterns inherited from Opus 5
- Sonnet 5.5: recalibrated effort, initiative and scope clauses, `between_tools`, JSON reasoning tasks, search in chat, mid-turn user messages, real-check verification, tolerant tool calls, plus the patterns inherited from Sonnet 5
- The Opus 5.5 vs Sonnet 5.5 divergence table
- Verbosity, deliverable length, narration, and self-correction control
- Scope discipline and over-verification removal
- Tool usage patterns (`<default_to_action>`, `<do_not_act_before_instructions>`)
- Format control (XML tags, positive instructions, `<avoid_excessive_markdown_and_bullet_points>`)
- Parallel tool calling (`<use_parallel_tool_calls>`)
- Long-horizon reasoning & state tracking; multi-context workflows & compaction
- Subagent orchestration — which direction each generation leans
- Balancing autonomy and safety (reversibility of actions)
- Visual/frontend generation & `<frontend_aesthetics>`; design default override
- Code review harness tuning for coverage vs precision
- Hallucination prevention (`<investigate_before_answering>`)
- Prefill migration paths; LaTeX opt-out; computer-use resolution

→ **Read `references/model-techniques.md`**

**For controller/worker agent stacks** (one model planning, others executing — including non-Claude executors such as a Codex CLI):

→ **Read `references/agent-orchestration.md`**

## Quick Reference: Patterns by Target

**Universal**

| Pattern | When to Use | Tag/Approach |
|---------|-------------|--------------|
| Default to Action | Agentic coding, implementation | `<default_to_action>` |
| Conservative Action | Research, analysis, planning | `<do_not_act_before_instructions>` |
| Parallel Tool Calls | Multi-file reads, independent ops | `<use_parallel_tool_calls>` |
| Investigation First | Code review, codebase questions | `<investigate_before_answering>` |
| Format Control | Prose output, minimal markdown | `<avoid_excessive_markdown_and_bullet_points>` |
| Data Placement | 20k+ token / multi-document inputs | Longform data at top, query at the end |
| State Tracking | Long tasks, multi-session | tests.json, progress.txt |
| Context Awareness | Long-running agents near context limit | "Context will be compacted..." prompt |
| Safety Gating | Prevent destructive shortcuts | Reversibility confirmation prompt |
| Avoid Test-Hardcoding | Agentic coding generality | "General-purpose solution" prompt |
| File Cleanup | Minimize net new files | "Clean up any temporary files" |

**5 generation**

| Pattern | When to Use | Target | Tag/Approach |
|---------|-------------|--------|--------------|
| Deconstraint Pass | System prompt / skill / `CLAUDE.md` written for 4.x | 5-gen | Remove conflicting rules, `CRITICAL`/`MUST`, self-check scaffolding (keep a concrete real-check line for Sonnet 5.5 at `low`) |
| **Remove** Self-Verification | Prompt says "verify"/"double-check" | Opus 5.5 | **Delete the instruction** — causes over-verification |
| **Add** Real-Check Line | Changes reported done without test/build output | Sonnet 5.5 (`low`) | "Run a real check that exercises the change before reporting done…" |
| Curb Subagents | Controller over-delegates | Opus 5.5 | Delegation ceiling (`references/agent-orchestration.md` §1) |
| No Extra Review Rounds | Long, costly sessions at `xhigh`/`max` | Sonnet 5.5 | "When the work is done and checks pass, stop and report…" |
| Keep Working | Checks in before agentic coding is done | Sonnet 5.5 (`low`/`medium`) | Raise effort, or the keep-working clause |
| Ideas First | Open-ended asks turn into builds | Sonnet 5.5 | "When asked for ideas or a plan, give that and stop" |
| Verbosity Control | Responses too long | Opus 5.5 | "Keep responses focused, brief, and concise" — *not* lower effort |
| Deliverable Length | Model-authored files run long | Opus 5.5 | "Match length to what the task needs; no filler sections" |
| Scope Discipline | Model widens the task | Opus 5.5 | "Deliver what was asked… finish the whole task" |
| Curb Narration | Too chatty between tool calls | Opus 5.5 | "Default to silence between tool calls" — after checking rendering |
| Render Updates | Agent looks silent mid-turn | Opus 5.5 / Sonnet 5.5 | `display: "updates"`; render non-empty `thinking` blocks; harness reminder after ~5 silent steps |
| Definition of Done | Long autonomous task | Opus 5.5 | `Done means:` list of observables + named stop conditions; doubles as the end-of-turn checker's criterion |
| Unattended Continuation | Loop stops after a progress report | Opus 5.5 | Text-only `end_turn` = report; checklist + continuation nudge (cap 2–3); name the early stops to avoid |
| Explore Before Acting | Multi-app automation misses context | Opus 5.5 | "Explore broadly with tool calls… including sources the task doesn't mention" |
| Time Budget | Agent team should finish sooner | Opus 5.5 | Append `elapsed 340s / 1200s` to each harness message |
| Settled Answers | Chat follow-ups re-think earlier answers | Opus 5.5 | "Treat earlier answers as settled unless asked" |
| Drop "Think Carefully" | Chat system prompt slows first token | Opus 5.5 | Delete the line; effort is the control |
| Think Before JSON | JSON answers to multi-step tasks are wrong | Sonnet 5.5 | Structured outputs + "Think the problem through before you answer."; `max_tokens` stop = failed |
| Parse Last JSON | No structured outputs available | Sonnet 5.5 | Parse the last JSON value, not first `{` to last `}` |
| Search Specifics | Answers from stale training knowledge | Sonnet 5.5 | Remove "minimize tool calls"; "check specifics that may have changed, even when confident" |
| User Text Out of Tool Results | Real mid-turn messages treated as injection | Sonnet 5.5 | User text after the last `tool_result`; notices in a separate system message; no per-step countdowns |
| Tolerant Tool Calls | Wrong-case tool or parameter names | Sonnet 5.5 | Accept unambiguous matches, or `is_error` with the exact name |
| Mark Pasted Text | Users paste emails/pages into messages | Opus 5.5 | `<pasted_content id="…">` tags + system-prompt note |
| No Reasoning-in-Response | Prompt asks model to show its reasoning | Opus 5.5 / Sonnet 5.5 | Delete; read `display: "summarized"` thinking (`reasoning_extraction` refusal) |
| Named Design Exclusions | Frontend looks generic | Opus 5.5 | List the specific defaults to avoid (cream bg, italic accents, 01/02/03, mono labels, pills) |
| Effort Recalibration | Inherited effort settings | Opus 5.5 / Sonnet 5.5 | Opus 5.5: start `medium`. Sonnet 5.5: `high`, `medium` for well-specified agentic, `medium`/`low` for chat. `xhigh`/`max` only with measured gains |
| Less Thinking = Lower Effort | Too slow to first token | Opus 5.5 / Sonnet 5.5 | Lower effort; prompt text does not reliably reduce thinking |
| Thinking Off | Integration ran with thinking disabled | Sonnet 5.5 | `between_tools` at `high` or below; drop "don't think" rules (they raise XML-tag leakage). Opus 5.5: impossible — start at `low` |
| Sampling → Prompt | Prompt relied on `temperature` | Opus 5.5 / Sonnet 5.5 | Sampling params are 400s; use propose-then-pick for variety |
| Literal Scoping | Prevent narrow interpretation | Sonnet 5.5 | "Apply to every section, not just the first" |
| Code Review Coverage | Severity filters depress recall | Opus 5.5 | "Report every issue including low-severity", filter downstream |
| Vision via Tools | Dense chart / technical drawing | Sonnet 5.5 (Opus 5.5: densest inputs only) | Crop/zoom/code tools — beat raising effort on charts |
| De-prescribe | 4.x step-by-step scaffolding | Fable 5 | State goal + constraints; remove enumerated steps |
| Ground Progress Claims | Long autonomous runs | Fable 5 | "Audit each claim against a tool result before reporting" |

**Orchestration** (see `references/agent-orchestration.md`)

| Pattern | When to Use | Approach |
|---------|-------------|----------|
| Delegation Ceiling | Controller prompt | Cap, not encouragement; "do NOT delegate" list carries the value |
| Self-Contained Brief | Every worker brief | Absolute paths, no conversation references |
| `Done means` Line | Every worker brief | Name an observable the controller can re-run |
| Scoped Negative Constraint | External/Codex executors | Say where the caution does **not** apply |
| Sandbox Boundary | Codex-style executors | `cd` to target tree, or write-then-move; treat refusals as expected |
| Report ≠ Evidence | Verification step | Re-run the claimed check; never accept a wrapper's status |
| Effort Routing | Whole stack | Per-role, not uniform — cost multiplies by agent count |

**Legacy 4.x**

| Pattern | When to Use | Target | Tag/Approach |
|---------|-------------|--------|--------------|
| Encourage Subagents | Model fans out too little | 4.7/4.8 | "Spawn multiple subagents when fanning out" — **not** for Opus 5.5 |
| Increase Tool Usage | Model uses tools less | 4.7/4.8 | Explicitly describe when and how to use tools |
| Force Summary After Tools | Visibility into tool-heavy runs | 4.x | "Provide a quick summary after tool use" |
| Curb Overthinking | Over-explores | 4.6 | "Choose an approach and commit" |
| Anti-Overengineering | Adds unnecessary abstractions | 4.5/4.6 | Scope/doc/defensive/abstractions prompt |
| Creativity Boost | UI/frontend, creative tasks | 4.5/4.6 | "Don't hold back. Give it your all." |
| Frontend Aesthetics | UI/frontend | 4.5–4.7 | `<frontend_aesthetics>` (long for 4.5/4.6, short for 4.7) |
| Design Default Override | Dashboard/fintech/dev tools | 4.7 | Explicit palette spec OR "propose 4 directions first" |
| Adaptive Thinking | Replace `budget_tokens` | 4.6+ | `thinking: {type: "adaptive"}` |
| Prefill Migration | 4.6+ drops prefill on last turn | 4.6+ | Structured outputs / preamble suppression |
| LaTeX Opt-Out | Defaults to LaTeX for math | 4.6 | "Format math in plain text only" |
| Computer Use Res | Screenshot cost/quality balance | 4.7+ | 1080p (up to 2576px/3.75MP) |

## Usage Patterns

### When User Provides Prompt to Optimize

1. Acknowledge and identify prompt type/quality level
2. Clarify goals and target model if needed
3. Determine optimization level
4. Execute optimization with framework
5. Apply model-specific techniques matched to the target model
6. Present results with improvements — and deletions — explained

### When User Wants Prompt From Scratch

1. Understand: task/goal, audience, desired output, constraints, target model, agentic needs?
2. Choose prompt type
3. Apply composition framework with patterns matched to the target model
4. Iterate based on feedback

### When Optimizing for a Specific Model

**For Opus 5.5:**
1. Remove `thinking: disabled` / `budget_tokens`, forced `tool_choice` (`any`/`tool`), sampling params, and prefill — all 400; use structured outputs or strict tools
2. **Delete** any self-verification instruction ("verify", "double-check", "use a subagent to verify") — inherited over-verification
3. **Remove** inherited "delegate more" guidance and add a subagent ceiling
4. Set `effort` explicitly — default is `medium`; start there and sweep, reserve `xhigh`/`max` for measured gains; size `max_tokens` for thinking
5. Add an explicit conciseness instruction (effort will not shorten output) and a deliverable-length instruction if it writes files
6. Add the scope-discipline clause, including "finish the whole task, report completion only when done", plus a `Done means` list for long tasks
7. Delete instructions that ask for reasoning in the response text and any "don't think" rule
8. If the client renders text between tool calls, set `display: "updates"` before touching the prompt; ask explicitly for the update cadence you want
9. For unattended agents, add harness continuation logic and the named-early-stops instruction at the end of the system prompt from the first request — not for human-in-the-loop products
10. Multi-app automation: add the explore-broadly instruction; multi-agent teams: an elapsed-time / budget line plus a hard timeout
11. Chat: remove "think carefully" lines; optionally add the settled-answers instruction; wrap user-pasted content in `<pasted_content>` tags
12. Code review: ask for coverage and filter downstream. Vision: re-test scaffolding. Frontend: name the default styles to avoid

**For Sonnet 5.5:**
1. Replace `thinking: disabled` with `between_tools` (at `high` or below, no other fields); remove forced `tool_choice`, sampling params, prefill, and thinking budgets — all 400
2. Re-sweep effort on recalibrated levels: `high` default; `medium` for well-specified agentic work; `medium`/`low` for chat; `xhigh`/`max` only with measured gains
3. State scope explicitly — it is literal — and decide whether you want its unrequested tests/docs (if not, add the "don't add what wasn't asked" paragraph)
4. At `low`/`medium` agentic coding: add the keep-working clause or raise effort. At `low`: add the real-check instruction. At `xhigh`/`max`: add "stop when checks pass, no extra review rounds"
5. JSON reasoning tasks: structured outputs + "Think the problem through before you answer." (adaptive thinking, not `between_tools`); treat `max_tokens` stops as failed; parse the last JSON value without structured outputs
6. Chat and knowledge work with a search tool: remove tool-discouraging language and add the search-specifics instruction
7. Harness: user text never inside `tool_result`; notices in a separate system message; no per-step countdowns in interactive sessions; append-only history; tolerant tool-name matching
8. Render progress-update `thinking` blocks (`display: "updates"`, or nothing extra under `between_tools`); remove "hold findings for the final response"
9. For variety, use propose-then-pick rather than `temperature`. For dense charts, give crop/zoom tools rather than more effort
10. Remove reasoning-in-response instructions (`reasoning_extraction`); handle the wider refusal set

**For Fable 5:**
1. **De-prescribe** — remove enumerated step-by-step scaffolding written for earlier models; state goal and constraints
2. Plan for minutes-long turns: streaming, async check-ins, progress UX
3. Give it a memory file and tell it to consult it in later sessions
4. Add the ground-progress-claims instruction for long autonomous runs
5. Let it delegate, asynchronously rather than spawn-and-block

**For legacy 4.x targets** (Opus 4.8 / 4.7 / 4.6, Sonnet 4.6, Haiku 4.5) — see `references/model-techniques.md`. Note that subagent, tool-use, and self-verification guidance for those models **inverts** on Opus 5.5; do not carry it forward.

### When the Target Is an Agent Stack

A controller model briefing worker models is not one prompt but three, each with its own failure mode:

1. **Controller prompt** — delegation ceiling, scope discipline, no self-verification instruction
2. **Worker brief** — self-contained (absolute paths, no conversation references), with a `Done means` line naming an observable
3. **Verification** — re-run the claimed check; a worker's report is a set of claims, not evidence

Route effort per role rather than uniformly; cost multiplies by agent count. External executors (Codex-style CLIs) add a process boundary: the sandbox is pinned to its working directory, wrapper status is not evidence, and negative constraints must state where they do **not** apply.

→ `references/agent-orchestration.md`

### General Playbook (Any Model)

1. Apply explicit instruction patterns
2. Add XML-wrapped behavioral controls
3. Put longform data at the top, queries at the end
4. Include parallel execution guidance for tool-heavy prompts
5. Add state tracking patterns for long-horizon tasks
6. Specify format control using positive instructions
7. Ground responses with `<investigate_before_answering>`
8. Cut over-constraint before adding anything
9. Never ask for internal reasoning in the response text on thinking-always-on models (Opus 5.5) — read summarized thinking instead

## Resources

- `references/model-techniques.md` — Complete optimization patterns with code examples: context engineering for 5-gen, common principles, and per-model sections (Opus 5.5, Sonnet 5.5, Fable 5, legacy 4.x) and the Opus 5.5 vs Sonnet 5.5 divergence table
- `references/agent-orchestration.md` — Controller/worker prompt patterns for multi-model agent stacks, including non-Claude executors
- `references/examples.md` — Detailed before/after optimization examples

## Final Principles

1. **Question assumptions** — Why does it have to work this way?
2. **Obsess over details** — Every word matters
3. **Plan thoroughly** — Design before writing
4. **Craft beautifully** — Elegant, intuitive structure
5. **Iterate relentlessly** — First version is never final
6. **Simplify ruthlessly** — Remove all excess; on 5-gen models the best edit is often a deletion
7. **Leverage model capabilities** — Use techniques appropriate for the target model, and check whether an inherited pattern inverts on it

The best prompts feel inevitable — like there's no other way they could have been written.
