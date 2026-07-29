---
name: prompt-optimizer
description: Advanced prompt optimization and composition system for all prompt types (system prompts, task-specific, creative, technical, agentic, multi-agent orchestration). Use when users need to refine, optimize, or transform prompts to achieve better AI outputs. Triggers include requests to improve prompts, make prompts more effective, analyze prompt quality, create prompts from scratch, optimize existing prompts for specific goals, rightsize a system prompt or CLAUDE.md, write controller/worker prompts for an agent stack, or optimize for specific Claude models (Opus 5, Sonnet 5, Fable 5, and legacy 4.x). Also use when users share prompts that could be improved or ask for prompt engineering guidance.
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
- **Target model?** (Opus 5 / Sonnet 5 / Fable 5 and legacy 4.x behave differently — see `references/model-techniques.md`)
- **Effort setting?** For API use cases, `max`/`xhigh`/`high`/`medium`/`low` changes optimal prompt density
- **Single prompt or an agent stack?** If a controller model briefs worker models, see `references/agent-orchestration.md` — three prompts exist per unit of work, and they fail differently

Analyze current prompt for: clarity, completeness, ambiguity, missing constraints, structure, token efficiency, and model-fit.

**Also analyze for over-constraint.** On 5-generation models the most common defect in an existing prompt is not a missing instruction but too many, conflicting: absolute rules that are not absolutely true, the same constraint repeated across layers, `CRITICAL`/`YOU MUST` emphasis, and self-check scaffolding. Optimization is frequently a **deletion**.

### 2. Strategic Optimization by Prompt Type

**System Prompts (AI Agents/Chatbots):**
Define role/persona, establish guardrails, specify output format/tone, include response patterns and examples, add error handling, balance flexibility with consistency.

**Task-Specific Prompts:**
Break down complex tasks, provide input/output examples, specify success criteria, include what NOT to do (or better: what TO do). Define success criteria as observables the *caller* can check — on 5-gen models, do not add "verify your work" instructions for the model itself.

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
- **Specificity**: Concrete rather than abstract (state scope explicitly — Sonnet 5 and Opus 5 will not silently generalize an instruction to neighbouring items)
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
A distinct level, and the right one for a system prompt, skill, or `CLAUDE.md` written for an earlier model generation. The output is *shorter* than the input: remove absolute-but-untrue rules, cross-layer duplication, `CRITICAL`/`YOU MUST` emphasis, forced-progress scaffolding, and self-verification instructions. Anthropic removed over 80% of Claude Code's system prompt this way for Opus 5 and Fable 5 with no measurable loss on coding evals.

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

| Pattern | Opus 4.7 / 4.8 | **Opus 5** |
|---|---|---|
| Subagents | Under-delegates → **encourage** | Over-delegates → **cap** |
| Tool use | Reaches for tools less → **push** | Reaches readily → leave alone |
| Self-verification | Helps | **Causes over-verification → delete** |
| Progress updates | May need forcing | Narrates on its own → **remove scaffolding** |
| Rules vs judgment | Explicit rules help | Over-constraint hurts → **deconstrain** |

### Model-specific behavior shifts

- **Opus 5** — Thinking **on by default** (disabling capped at `high` effort). Verbose — effort does *not* control visible length, prompting does. Narrates more; writes longer deliverables. **Over-verifies** if told to verify. Can expand task scope. **Over-delegates** to subagents. Narrates self-corrections at length. Full effort ladder with `low`/`medium` unusually strong. Code-review severity filters are followed literally → ask for coverage, filter downstream. Vision improves most from crop/analyze/verify *tools*, not more thinking.
- **Sonnet 5** — Adaptive thinking on by default; `thinking.display` defaults to `"omitted"`. **Non-default `temperature`/`top_p`/`top_k` return 400** → style variety must come from the prompt (propose-then-pick). More literal → state scope explicitly. Respects effort strictly at the low end → raise effort rather than prompting around shallow reasoning. New tokenizer, ~30% more tokens. Less tool-eager when thinking is disabled.
- **Fable 5** — Thinking always on, cannot be disabled; raw chain of thought never returned. Long turns (minutes). **De-prescribe migrated prompts** — 4.x step-by-step scaffolding reduces its output quality. Benefits from a memory file and from asynchronous sub-agents.
- **Opus 4.8 / 4.7 (legacy)** — Under-delegate; need tool/subagent encouragement. 4.7 has a strong cream/serif design house style.
- **Opus 4.6 (legacy)** — Overtriggers tools/skills, overengineers, overthinks, overuses subagents. Prefills deprecated. Defaults to LaTeX.
- **Sonnet 4.6 / Haiku 4.5 (legacy)** — Default `effort=high`; Haiku is speed-first and usually needs minimal prompting.

**For complete patterns including:**

- Context engineering for 5-gen (what to cut from system prompts, skills, `CLAUDE.md`)
- Effort parameter tuning (`max` / `xhigh` / `high` / `medium` / `low`) and per-role routing
- Thinking configuration, `display`, and the thinking-disabled failure modes on Opus 5
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
| Deconstraint Pass | System prompt / skill / `CLAUDE.md` written for 4.x | 5-gen | Remove conflicting rules, `CRITICAL`/`MUST`, self-check scaffolding |
| **Remove** Self-Verification | Prompt says "verify"/"double-check" | Opus 5 | **Delete the instruction** — causes over-verification |
| Curb Subagents | Controller over-delegates | Opus 5 | Delegation ceiling (`references/agent-orchestration.md` §1) |
| Verbosity Control | Responses too long | Opus 5 / Sonnet 5 | "Keep responses focused, brief, and concise" — *not* lower effort |
| Deliverable Length | Model-authored files run long | Opus 5 | "Match length to what the task needs; no filler sections" |
| Scope Discipline | Model widens the task | Opus 5 | "Deliver what was asked… finish the whole task" |
| Curb Narration | Too chatty between tool calls | Opus 5 | "Default to silence between tool calls" |
| Curb Self-Correction | Narrates own mistakes at length | Opus 5 | "Only correct when it changes the user's decisions" |
| Effort Sweep | Inherited effort defaults | Opus 5 / Sonnet 5 | Start `high` (`xhigh` coding), sweep down — `low`/`medium` are strong |
| Thinking Budget Check | Prompt never set `thinking` | Opus 5 / Sonnet 5 | Thinking now on by default — re-check `max_tokens` |
| Thinking-Disabled Repair | Tool calls appear as text / tag leakage | Opus 5 | Prefer low effort + thinking on; else generic "no internal XML tags" |
| Sampling → Prompt | Prompt relied on `temperature` | Sonnet 5 | Non-default sampling = 400; use propose-then-pick for variety |
| Literal Scoping | Prevent narrow interpretation | Sonnet 5 / Opus 5 | "Apply to every section, not just the first" |
| Raise Effort, Not Prose | Shallow reasoning at `low`/`medium` | Sonnet 5 | Raise effort instead of adding "think carefully" |
| Token Re-baseline | Migrating from Sonnet 4.6 | Sonnet 5 | New tokenizer, ~30% more tokens |
| Code Review Coverage | Severity filters depress recall | Opus 5 / Sonnet 5 | "Report every issue including low-severity", filter downstream |
| Vision via Tools | Chart/diagram/UI work | Opus 5 | Give crop/analyze/verify tools rather than raising thinking |
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
| Encourage Subagents | Model fans out too little | 4.7/4.8 | "Spawn multiple subagents when fanning out" — **not** for Opus 5 |
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

**For Opus 5:**
1. **Delete** any self-verification instruction ("verify", "double-check", "use a subagent to verify") — it now causes over-verification
2. **Remove** inherited "delegate more" guidance and add a subagent ceiling instead
3. Add an explicit conciseness instruction — effort will not shorten visible output
4. Add a deliverable-length instruction if the product ships model-authored files
5. Add the scope-discipline clause, including "finish the whole task, report completion only when done"
6. Remove forced progress-update scaffolding; add a silence-default if the agent is too chatty
7. Add the corrections instruction if self-correction narration reads as thrash
8. Re-sweep effort — start `high` (`xhigh` for coding/agentic), then step down; `low`/`medium` are unusually strong
9. If the prompt never set `thinking`, re-check `max_tokens` — thinking is on by default now
10. For code review, ask for coverage and filter downstream
11. For vision work, give crop/analyze/verify tools rather than raising thinking

**For Sonnet 5:**
1. Remove non-default `temperature`/`top_p`/`top_k` — they return 400; move style control into the prompt
2. For design/creative variety, use propose-then-pick rather than generic redirection
3. State scope explicitly — it will not generalize an instruction to neighbouring items
4. Set effort deliberately: `high` default, `xhigh` for the hardest coding; raise effort rather than prompting around shallow reasoning
5. Set `display: "summarized"` if reasoning is streamed to users
6. Re-count tokens against `claude-sonnet-5` (~30% more than Sonnet 4.6)
7. If thinking is disabled, add explicit tool-triggering guidance — it becomes much less tool-eager

**For Fable 5:**
1. **De-prescribe** — remove enumerated step-by-step scaffolding written for earlier models; state goal and constraints
2. Plan for minutes-long turns: streaming, async check-ins, progress UX
3. Give it a memory file and tell it to consult it in later sessions
4. Add the ground-progress-claims instruction for long autonomous runs
5. Let it delegate, asynchronously rather than spawn-and-block

**For legacy 4.x targets** (Opus 4.8 / 4.7 / 4.6, Sonnet 4.6, Haiku 4.5) — see `references/model-techniques.md`. Note that subagent, tool-use, and self-verification guidance for those models **inverts** on Opus 5; do not carry it forward.

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

## Resources

- `references/model-techniques.md` — Complete optimization patterns with code examples: context engineering for 5-gen, common principles, and per-model sections (Opus 5, Sonnet 5, Fable 5, legacy 4.x)
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
