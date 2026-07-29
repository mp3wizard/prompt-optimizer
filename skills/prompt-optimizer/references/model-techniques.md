# Claude Model Optimization Techniques

Complete patterns and code examples for optimizing prompts across current Claude models. The **5 generation** — **Opus 5**, **Sonnet 5**, **Fable 5** — is the primary target; the **4.x family** (Opus 4.8 / 4.7 / 4.6, Sonnet 4.6, Haiku 4.5) is retained for prompts not yet migrated.

All current models share common behaviors (explicit following, attention to examples, XML-responsiveness), but each has distinct shifts that need targeted tuning. Start from "Common principles", then apply the model-specific section for your target.

> **If you are tuning a prompt written for Opus 4.7 or 4.8, several of its patterns now invert.** The most consequential: 4.7 needed *encouragement* to delegate and use tools; Opus 5 needs a *ceiling*. And instructions telling the model to verify its own work, which helped on prior models, now cause over-verification. See "Opus 5 — Behavior Shifts" below before reusing 4.x snippets.

---

## Model Scope & Identity

### Current generation (5)

| Model | String | Best for | Notable default |
|---|---|---|---|
| Claude Opus 5 | `claude-opus-5` | Complex agentic coding, long-horizon autonomous work, enterprise/knowledge work, multi-agent coordination | **Thinking on by default**; delegates readily; verbose; 1M context; effort `high` |
| Claude Sonnet 5 | `claude-sonnet-5` | Coding and agentic work at volume; near-Opus quality at Sonnet cost | Adaptive thinking on by default; **non-default sampling params rejected**; effort `high` |
| Claude Fable 5 | `claude-fable-5` | The most demanding reasoning and longest-horizon agentic work | Thinking always on (cannot be disabled); raw chain of thought never returned; requires 30-day data retention |

### Prior generation (4.x — legacy targets)

| Model | String | Best for | Notable default |
|---|---|---|---|
| Claude Opus 4.8 | `claude-opus-4-8` | Long-horizon agentic; the Opus 5 predecessor | Under-delegates; warmer prose; thinking off unless set |
| Claude Opus 4.7 | `claude-opus-4-7` | Agentic, research, vision, memory | Literal following; **fewer** tool calls and subagents; cream/serif design house style |
| Claude Opus 4.6 | `claude-opus-4-6` | Pre-4.7 baseline | LaTeX for math; overtriggers tools/subagents; prefills deprecated |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | Balanced cost/quality, coding, computer-use | Default `effort=high`; adaptive thinking |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | Latency-sensitive, high-volume, lightweight classification | Speed-first; responds well to minimal prompts |

For model identification inside prompts:

```text
The assistant is Claude, created by Anthropic. The current model is Claude Opus 5.
```

```text
When an LLM is needed, please default to Claude Opus 5 unless the user requests otherwise. The exact model string for Claude Opus 5 is claude-opus-5.
```

---

## Context Engineering for the 5 Generation

A prompt is one input; **context** is everything assembled around it — system prompt, skills, `CLAUDE.md`, memory, tool descriptions. It is reused across many requests, so it cannot be as specific as a prompt, and it is where over-constraint accumulates.

Anthropic removed **over 80% of Claude Code's system prompt** for Opus 5 and Fable 5 with no measurable loss on coding evaluations. That is the headline result for anyone still writing 4.x-era context.

**Then: give Claude rules → Now: let Claude use judgment.**

Guardrails written to prevent worst cases on earlier models now fight each other. A single request can carry "leave documentation as appropriate" from one layer and "DO NOT add comments" from another, plus a user request that implies a third thing. The model can usually resolve the user's intent — but it has to spend reasoning on the conflict first, and quality drops.

What to cut when optimizing a system prompt, skill, or `CLAUDE.md` for a 5-gen model:

- **Absolute rules that are not absolutely true.** "In code: default to writing no comments. NEVER write multi-line comment blocks." Replaced by stating the goal and letting the model read the surrounding code.
- **Rules duplicated across layers.** The same constraint in the system prompt, a skill, and `CLAUDE.md` is three chances to conflict.
- **Aggressive emphasis.** `CRITICAL:`, `YOU MUST`, `ALWAYS`, `NEVER`, `If in doubt, use X`. These were written to overcome earlier models' reluctance and now overtrigger.
- **Scaffolding the model no longer needs.** Forced progress updates ("summarize every 3 tool calls"), self-verification steps, "double-check before answering".

What to keep:

- Facts the model cannot infer (project conventions, deploy targets, credentials layout, the one library that must not be used and why).
- Constraints with a stated reason — motivation generalizes where a bare rule does not.
- Trigger conditions on tools and skills ("call this when the user asks about current prices"), which measurably raise should-call rate.

> In Claude Code, `/doctor` applies these checks to your own skills and `CLAUDE.md` files.

---

## Common Principles (all current models)

### 1. Be Explicit With Instructions

Current Claude models respond precisely to what you ask — no more, no less. If you want "above and beyond" behavior, explicitly request it.

**Less effective:**
```
Create an analytics dashboard
```

**More effective:**
```
Create an analytics dashboard. Include as many relevant features and interactions as possible. Go beyond the basics to create a fully-featured implementation.
```

### 2. Add Context and Motivation

Explain WHY behind your instructions. Claude generalizes from understanding the purpose.

**Less effective:**
```
NEVER use ellipses
```

**More effective:**
```
Your response will be read aloud by a text-to-speech engine, so never use ellipses since the text-to-speech engine will not know how to pronounce them.
```

### 3. Be Vigilant With Examples

Current Claude models pay close attention to examples. Ensure they align perfectly with desired behaviors and don't demonstrate behaviors you want to avoid.

- **Relevant**: mirror your actual use case closely
- **Diverse**: cover edge cases; avoid unintended pattern picks
- **Structured**: wrap examples in `<example>` tags (multiple in `<examples>`) so Claude distinguishes them from instructions
- **3–5 examples** typically give the best lift

### 4. Structure Prompts with XML Tags

XML helps Claude parse complex prompts unambiguously, especially when your prompt mixes instructions, context, examples, and variable inputs. Wrap each content type in its own tag (`<instructions>`, `<context>`, `<input>`) to reduce misinterpretation.

- Use consistent, descriptive tag names
- Nest where there's natural hierarchy (documents inside `<documents>`, each inside `<document index="n">`)

### 5. Long Context Prompting (20k+ tokens)

- **Put longform data at the top** — documents/inputs above queries, instructions, and examples. Queries at the end can improve response quality by up to 30% on complex, multi-document inputs.
- **Structure documents with XML**:

```xml
<documents>
  <document index="1">
    <source>annual_report_2023.pdf</source>
    <document_content>
      {{ANNUAL_REPORT}}
    </document_content>
  </document>
  <document index="2">
    <source>competitor_analysis_q2.xlsx</source>
    <document_content>
      {{COMPETITOR_ANALYSIS}}
    </document_content>
  </document>
</documents>

Analyze the annual report and competitor analysis. Identify strategic advantages and recommend Q3 focus areas.
```

- **Ground responses in quotes** — for long document tasks, ask Claude to quote relevant excerpts first, then answer:

```xml
Find quotes from the patient records and appointment history that are relevant to diagnosing the patient's reported symptoms. Place these in <quotes> tags. Then, based on these quotes, list all information that would help the doctor diagnose the patient's symptoms. Place your diagnostic information in <info> tags.
```

---

## Opus 5 — Behavior Shifts & Tuning

Opus 5 performs well out of the box on existing Opus 4.8 prompts. What follows is what *changed* — and several items are **deletions**, not additions.

### Thinking is on by default

A request that omits the `thinking` parameter **thinks**. On Opus 4.8 and 4.7, omitting it meant no thinking. Two consequences for prompt work:

- `max_tokens` caps thinking **plus** response text. A prompt sized tightly around its answer on 4.8 can now truncate mid-response.
- Disabling thinking (`thinking: {type: "disabled"}`) is only available at effort **`high` or lower**. Pairing it with `xhigh`/`max` is a 400, validated per request.

Prefer lowering effort over disabling thinking. Disabled thinking on Opus 5 has two failure modes that no prompt fully fixes:

1. **Tool calls arrive as plain text.** The model writes the call into its visible response instead of emitting a structured tool-use block. The turn completes normally, the call never runs, and nothing errors — in an agentic loop the bogus text stays in history and skews later turns. Mitigation, if you cannot enable thinking: `You may say a brief sentence before using a tool.`
2. **`<thinking>` tags leak into output.** Two counterintuitive rules: **delete** any instruction telling the model not to think or reason (it makes leakage worse), and write the instruction generically — `Do not include internal or system XML tags in your response.` Naming thinking tags specifically is measurably less effective.

### Response length is up — and effort will not fix it

Default response text is longer than prior models. **`effort` is not the lever** — changing it moves thinking volume without reliably changing visible output length. Prompting is:

```text
Keep responses focused, brief, and concise to avoid overwhelming the person.
Disclaimers and caveats are brief, with most of the response on the main answer;
when asked to explain something, give a high-level summary unless an in-depth one
is specifically requested.
```

For a long system prompt, pair that with a reminder near the end:

```text
<tone_preference>
Keep outputs reasonably concise.
</tone_preference>
```

### Written deliverables run long

Separate from conversational verbosity: files written to disk — reports, Markdown, summaries — are longer than on prior models. If your product ships model-authored documents:

```text
Match the length of written deliverables (especially Markdown files) to what the task
needs: cover the substance, but do not pad documents with filler sections, redundant
summaries, or boilerplate.
```

### Over-verification — delete your verification instructions

Opus 5 verifies its own work without being asked. Instructions that *tell* it to verify — "include a final verification step for any non-trivial task", "use a subagent to verify", and per-prompt phrasing like "double-check your answer" — now cause over-verification. **Removing them reduces over-verification with no capability regression.** This is a delete, not a rewrite, and it applies to harness-level verification steps carried over from prior models too.

> This **inverts a standard prompting best practice.** "Ask the model to self-check" is generally sound advice and is wrong on this model. A prompt library that applies it uniformly needs a carve-out, not a global rule.

### Task scope expansion

Opus 5 can add steps the user did not request, or apply its own judgment about what the task should be without saying so. In testing this instruction reduced scope changes to nearly zero without producing excessive clarifying questions:

```text
Deliver what the user asked for, at the scope they intended. Interpret ambiguity the
way a careful colleague would: make routine judgment calls yourself, and check in only
when different readings would lead to materially different work. If you conclude the
ask is mistaken or a better approach exists, say so in a sentence and keep going with
the task as asked — don't quietly narrow, widen, or transform it. Finish the whole
task, not just the easy part of it — only report completion when it's fully done. If
you genuinely can't complete something, do the rest and state plainly what's missing
and why. Stop short of actions or changes that are clearly beyond what the user's ask
implies.
```

The "finish the whole task" clause does separate work from the scope clause: it covers premature "done" claims, which scope wording alone does not.

### Delegates to subagents readily — the opposite of 4.7/4.8

**Direction change.** Opus 4.7 and 4.8 *under*-delegated and needed prompting to fan out. Opus 5 reaches for subagents freely, multiplying cost and latency. If you carried "delegate more" guidance forward from 4.8, **remove it** and add a ceiling. See `agent-orchestration.md` for the full controller-prompt block.

### More narration between tool calls

Opus 5 narrates what it is about to do, and per-message output in agentic sessions runs longer. If you added scaffolding to force interim status ("after every 3 tool calls, summarize"), **remove it** — the model does this on its own. For a coding agent that is too chatty:

```text
Default to silence between tool calls. Only write text when you find something, change
direction, or hit a blocker — one sentence each. Do not narrate routine actions ("Now
I'll...", "Let me check...", "Looking at..."). When done: one or two sentences on the
outcome. Do not recap every file or test — the user has been following along.
```

The same lever runs the other way: this model responds well to explicit guidance on *how* to communicate, so it can be tuned up or restyled rather than only suppressed.

### Narrates self-corrections at length

It flags and explains its own earlier mistakes in detail, which reads as thrash in a user-facing product:

```text
Only correct an earlier statement in your user-facing text when the error would change
the user's code, conclusions, or decisions. State corrections plainly and concisely and
continue; combine multiple corrections rather than enumerating them. For slips that
change nothing for the user, make the correction and move on. Don't add apologies,
don't be overly self-critical, and don't ruminate or tally past errors. This instruction
does not apply to thinking blocks.

A follow-up question about your earlier work is not, by itself, a signal that you got
something wrong — answer what was asked. A statement that was accurate needs no
correction.
```

The second paragraph matters as much as the first: a plain follow-up question can otherwise trigger a re-audit of correct work.

### Effort: sweep, don't inherit

Opus 5 supports the full ladder — `low` / `medium` / `high` / `xhigh` / `max`, default `high`. Two directions pull against each other and both are worth knowing:

- **Start at `xhigh` for coding and agentic work**, `high` for other intelligence-sensitive workloads. `max` for the deepest reasoning, where it can show diminishing returns and overthink simpler tasks.
- **Then sweep downward** — `low` and `medium` punch well above their weight here, at a fraction of the tokens and latency. Step down where your evals show quality holds.

Effort defaults carried over from a prior model are rarely the right setting. At `xhigh`/`max`, set a large `max_tokens` (start 64K) so the model has room to think and act across tool calls.

### Code review

High precision *and* high recall. The 4.7-era caveat still applies and is worth restating: a review prompt saying "only report high-severity issues" or "be conservative" is followed **literally**, so measured recall drops even though bug-finding improved. Ask for coverage and filter downstream:

```text
Report every issue you find, including ones you are uncertain about or consider
low-severity. Do not filter for importance or confidence at this stage — a separate
verification step will do that. For each finding, include your confidence level and an
estimated severity so a downstream filter can rank them.
```

### Vision: give it tools, not more thinking

Strong on chart, document, and diagram understanding and on UI replication. The highest-leverage change is **giving it tools to iteratively analyze, crop, and visually verify its own work** — tool use is a markedly more cost-effective lever than raising thinking. Re-validate any prompt-side workaround written for a prior model's vision limits; several are now counterproductive.

### Other prompt-relevant defaults

- **1M context** as both default and maximum; instruction following and tool calling stay consistent across the window.
- **Prompt cache minimum drops to 512 tokens** (from 1024 on Opus 4.8) — prompts previously too short to cache now cache with no code change.
- **Long-horizon work:** give the complete task specification up front in one well-specified turn and let it run. Longer autonomous sessions show the strongest results; short interactive edits the least.

---

## Sonnet 5 — Behavior Shifts & Tuning

Sonnet 5 performs well out of the box on existing Sonnet 4.6 prompts. Its strengths are coding and agentic work, at near-Opus quality for Sonnet cost.

### Sampling parameters are rejected

`temperature`, `top_p`, and `top_k` set to **non-default** values return a 400. Omitting them, or passing the default, is fine.

This matters for prompt work because style variety that used to come from `temperature` must now come from the prompt. For design and creative tasks, the reliable substitute is **propose-then-pick**:

```text
Before building, propose 4 distinct visual directions tailored to this brief (each as:
bg hex / accent hex / typeface — one-line rationale). Ask the user to pick one, then
implement only that direction.
```

Generic redirection ("don't use that color", "make it clean and minimal") tends to move the model to a *different* fixed default rather than producing variety.

### Adaptive thinking is on by default

A request with no `thinking` field runs with adaptive thinking; on Sonnet 4.6 it ran without. Same `max_tokens` consequence as Opus 5 — the cap covers thinking plus response.

`thinking.display` defaults to `"omitted"` (Sonnet 4.6 defaulted to `"summarized"`). If you stream reasoning to users, set `display: "summarized"` explicitly — otherwise thinking blocks stream with empty text and the UI shows a long pause before output.

### Effort is the primary lever

Defaults to `high`, same as Sonnet 4.6. Raise to `xhigh` for the hardest coding and agentic work.

Rough cross-model mapping when migrating: **Sonnet 5 at `medium` ≈ Sonnet 4.6 at `high`**, and **Sonnet 5 at `high` ≈ Sonnet 4.6 at `max`**. When benchmarking, match by observed thinking length rather than effort name.

Sonnet 5 respects effort **strictly at the low end** — at `low`/`medium` it scopes work to what was asked rather than going above and beyond. If you see shallow reasoning on complex problems, **raise effort rather than prompting around it**. If latency forces `low`:

```text
This task involves multistep reasoning. Think carefully through the problem before
responding.
```

### More literal instruction following

Sonnet 5 does not silently generalize an instruction from one item to another, and does not infer requests you did not make. Good for structured extraction and predictable pipelines; a trap for instructions meant to apply broadly.

```text
Apply this formatting to every section, not just the first one.
```

The same literalism means style and tone directives carried from Sonnet 4.6 now apply at face value — re-baseline holdover lines like "be concise" before keeping them.

### Response length calibrates to task complexity

Shorter on simple lookups, longer on open-ended analysis, rather than a fixed verbosity. If your product depends on a particular length, tune explicitly. Positive examples of the concision you want outperform negative "don't do X" instructions.

### Tool use

More agentic than Sonnet 4.6 by default — it reaches for tools and self-verification loops more readily. **With thinking disabled it becomes markedly less tool-eager**; if your harness relies on tool calls with thinking off, add explicit triggering guidance in the system prompt and in each tool's own `description`.

### New tokenizer

The same text produces roughly **30% more tokens** than on Sonnet 4.6. No prompt change required, but everything measured in tokens shifts: re-run token counts against `claude-sonnet-5` rather than reusing 4.6 numbers, and re-check `max_tokens` limits sized close to expected output.

---

## Fable 5 — Positioning & Tuning

The most capable widely released model, for the most demanding reasoning and longest-horizon agentic work. Priced above the Opus tier — use it when the user explicitly chose it, not as a default upgrade.

Prompt-relevant characteristics:

- **Thinking is always on** and cannot be configured off. The raw chain of thought is never returned.
- **Turns run long** — single requests on hard tasks can run many minutes at higher effort. Plan progress UX around asynchronous check-ins rather than blocking inside one request.
- **De-prescribe migrated prompts.** Prompts and skills written for prior models are often *too prescriptive* for Fable 5 and reduce output quality. State the goal and constraints rather than enumerating the steps, and A/B with older step-by-step scaffolding removed.
- **Give it a memory surface.** It performs notably better when it can write learnings to a file for future reference — tell it where, tell it to consult that file later, and give it a format.
- **Ground progress claims:**

```text
Before reporting progress, audit each claim against a tool result from this session.
Only report work you can point to evidence for; if something is not yet verified, say
so explicitly. Report outcomes faithfully: if tests fail, say so with the output; if a
step was skipped, say that.
```

- **Let it delegate asynchronously.** Parallel sub-agents are dependable here; sub-agents that communicate asynchronously with the orchestrator outperform spawn-and-block.

---

## Opus 4.7 — Behavior Shifts & Tuning (legacy)

> Superseded by Opus 5. Several patterns below **invert** on Opus 5 — notably tool-use and subagent encouragement. Use this section only when targeting 4.7 specifically.

Opus 4.7 performs well out of the box on existing Opus 4.6 prompts, but several behaviors shifted and often need tuning. Most 4.6 snippets still work; what follows is what *changed*.

### Response Length & Verbosity

4.7 **calibrates response length to task complexity** rather than defaulting to a fixed verbosity. Simple lookups get shorter answers; open-ended analysis gets longer ones. If your product depends on specific style/length, tune explicitly.

To decrease verbosity:

```text
Provide concise, focused responses. Skip non-essential context, and keep examples minimal.
```

Positive examples of the communication style you want tend to outperform negative "don't do X" instructions.

### Effort Parameter (`max` / `xhigh` / `high` / `medium` / `low`)

4.7 respects effort levels **strictly, especially at the low end**. At `low`/`medium` it scopes to what was asked rather than going above-and-beyond — good for latency/cost, risky for complex work at `low`.

- **`max`** — Some gains on hardest tasks; risk of overthinking and diminishing returns
- **`xhigh`** (new) — Best for most coding and agentic use cases
- **`high`** — Minimum baseline for intelligence-sensitive workloads
- **`medium`** — Cost-sensitive with acceptable intelligence tradeoff
- **`low`** — Short, scoped, latency-sensitive; not intelligence-sensitive

**Guidance:**
- If you see shallow reasoning on complex problems, **raise effort** instead of prompting around it.
- If you must keep `effort=low`: add targeted nudge:

```text
This task involves multi-step reasoning. Think carefully through the problem before responding.
```

- At `max`/`xhigh` effort, set a large max output token budget (start at **64k**) so the model has room to think and act across subagents and tool calls.

Steer adaptive thinking triggering if 4.7 thinks more than you want:

```text
Thinking adds latency and should only be used when it will meaningfully improve answer quality — typically for problems that require multi-step reasoning. When in doubt, respond directly.
```

### More Literal Instruction Following

4.7 interprets prompts more literally than 4.6, **especially at lower effort**. It will **not** silently generalize an instruction from one item to another, and will not infer requests you didn't make.

Upside: precision and less thrash; better for carefully-tuned API prompts and pipelines.

**Tuning:** state scope explicitly.

```text
Apply this formatting to every section, not just the first one.
```

### Tool Use Triggering (Lower)

4.7 uses tools **less often** than 4.6 and reasons more. Usually this is better. If you need more tool use (e.g., web search in knowledge work):

- Raise `effort` to `high` or `xhigh` — both substantially increase tool usage in agentic search/coding
- Explicitly describe **when and how** tools should be used:

```text
Use the web_search tool whenever the user asks about events after your knowledge cutoff, current prices, or real-time data. Prefer multiple specific queries over one broad query.
```

### Subagent Spawning (Lower)

4.7 spawns fewer subagents by default. This is steerable via prompt.

```text
Do not spawn a subagent for work you can complete directly in a single response (e.g. refactoring a function you can already see).

Spawn multiple subagents in the same turn when fanning out across items or reading multiple files.
```

### Tone and Writing Style

4.7 is more direct and opinionated, with less validation-forward phrasing and fewer emoji than 4.6's warmer style. If your product relies on a specific voice, re-evaluate style prompts against the new baseline.

Warmer product voice override:

```text
Use a warm, collaborative tone. Acknowledge the user's framing before answering.
```

### User-Facing Progress Updates

4.7 provides **more regular, higher-quality updates** during long agentic traces. If you have scaffolding that forces status messages ("After every 3 tool calls, summarize progress"), **try removing it**. If update length/content doesn't fit your use case, describe what updates should look like and give examples.

### Design & Frontend Defaults (Opus 4.7 House Style)

4.7 has stronger design instincts and a **consistent default house style**: warm cream/off-white backgrounds (~`#F4F1EA`), serif display type (Georgia, Fraunces, Playfair), italic word-accents, terracotta/amber accent. Reads well for editorial/hospitality/portfolio; **feels wrong for dashboards, dev tools, fintech, healthcare, enterprise**. Appears in both web UIs and slide decks.

Generic overrides ("don't use cream", "make it clean and minimal") tend to shift to a *different fixed palette*, not produce variety. **Two approaches work reliably:**

**Approach 1 — Specify a concrete alternative** (model follows explicit specs precisely):

```text
Design a desktop landing page for a supplement brand called AEFRM.

The visual direction should come from a cold monochrome atmosphere using pale silver-gray tones that gradually deepen into blue-gray and near-black, similar to a misted metallic surface.

Use 4px corner radius consistently across cards, buttons, inputs, and media frames. Margins should feel generous, with enough empty space around each section so the page breathes.

Typography should use a square, angular sans-serif with wider letter spacing than usual, especially in headings and navigation. Headline text can be large and uppercase, while supporting copy remains short and sparse.

Buttons should be flat and precise, with subtle hover changes using transition: all 160ms ease out where brightness and border contrast shift slightly rather than using dramatic motion.

Color palette should stay within this range:
#E9ECEC, #C9D2D4, #8C9A9E, #44545B, #11171B.
```

**Approach 2 — Propose options first** (breaks the default, gives users control; replaces pre-4.7 `temperature` tricks for variety):

```text
Before building, propose 4 distinct visual directions tailored to this brief (each as: bg hex / accent hex / typeface — one-line rationale). Ask the user to pick one, then implement only that direction.
```

4.7 needs **less frontend prompting** than prior models to avoid "AI slop". Shorter `<frontend_aesthetics>` suffices:

```text
<frontend_aesthetics>
NEVER use generic AI-generated aesthetics like overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white or dark backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character. Use unique fonts, cohesive colors and themes, and animations for effects and micro-interactions.
</frontend_aesthetics>
```

### Interactive Coding Products

4.7 uses more tokens in **interactive** sessions (multi-turn) than in **autonomous** ones because it reasons more after user turns. This improves long-horizon coherence and instruction following in long sessions, but increases token usage.

**To maximize both performance and token efficiency:**
- Use `xhigh` or `high` effort
- Add autonomous features (auto-mode)
- Reduce required human interactions
- **Specify the task, intent, and constraints upfront in the first human turn** — ambiguous progressive prompts across many turns reduce both efficiency and performance

### Code Review Harnesses

4.7 is meaningfully better at finding bugs (~11pp recall improvement on hardest evals), but harnesses tuned with "only report high-severity", "be conservative", "don't nitpick" may show **lower measured recall** — 4.7 follows those filters faithfully and suppresses low-severity findings even after investigating them. Precision rises; reported recall drops.

Fix by moving filtering out of the finding step:

```text
Report every issue you find, including ones you are uncertain about or consider low-severity. Do not filter for importance or confidence at this stage - a separate verification step will do that. Your goal here is coverage: it is better to surface a finding that later gets filtered out than to silently drop a real bug. For each finding, include your confidence level and an estimated severity so a downstream filter can rank them.
```

If you want single-pass self-filtering, be **concrete** about where the bar is (e.g., "report any bugs that could cause incorrect behavior, a test failure, or a misleading result; only omit nits like pure style or naming preferences") rather than qualitative terms like "important".

### Computer Use

Works across resolutions up to a new max of **2576px / 3.75MP**. **1080p** is the recommended balance of performance and cost. **720p or 1366×768** for cost-sensitive workloads. Experiment with effort to tune behavior.

---

## Opus 4.6 — Behavior Shifts & Tuning (legacy)

> Superseded by Opus 5. Retained for prompts still targeting 4.6.

### Overtriggering Tools and Skills

4.5/4.6 are more responsive to system prompts than earlier models. Prompts designed to combat undertriggering may now over-trigger. **Dial back aggressive language**:

- Was: "CRITICAL: You MUST use this tool when..."
- Now: "Use this tool when..."

### Overthinking / Excessive Upfront Exploration

4.6 does significantly more upfront exploration at higher effort, often helpful but sometimes excessive. Tune previously-added "be thorough" prompts:

- Replace blanket defaults: "Default to using [tool]" → "Use [tool] when it would enhance your understanding of the problem"
- Remove over-prompting: "If in doubt, use [tool]" now over-triggers
- Use lower `effort` as a fallback if Claude is still too aggressive

Constrain extensive thinking:

```text
When you're deciding how to approach a problem, choose an approach and commit to it. Avoid revisiting decisions unless you encounter new information that directly contradicts your reasoning. If you're weighing two approaches, pick one and see it through. You can always course-correct later if the chosen approach fails.
```

### Subagent Overuse

4.6 may spawn subagents for code exploration where a direct `grep` would be faster.

```text
Use subagents when tasks can run in parallel, require isolated context, or involve independent workstreams that don't need to share state. For simple tasks, sequential operations, single-file edits, or tasks where you need to maintain context across steps, work directly rather than delegating.
```

### Overengineering / Overeagerness

4.5/4.6 tend to overengineer — extra files, unnecessary abstractions, unasked-for flexibility.

```text
Avoid over-engineering. Only make changes that are directly requested or clearly necessary. Keep solutions simple and focused:

- Scope: Don't add features, refactor code, or make "improvements" beyond what was asked. A bug fix doesn't need surrounding code cleaned up. A simple feature doesn't need extra configurability.

- Documentation: Don't add docstrings, comments, or type annotations to code you didn't change. Only add comments where the logic isn't self-evident.

- Defensive coding: Don't add error handling, fallbacks, or validation for scenarios that can't happen. Trust internal code and framework guarantees. Only validate at system boundaries (user input, external APIs).

- Abstractions: Don't create helpers, utilities, or abstractions for one-time operations. Don't design for hypothetical future requirements. The right amount of complexity is the minimum needed for the current task.
```

### LaTeX Default for Math

Opus 4.6 defaults to LaTeX for mathematical expressions. For plain text:

```text
Format your response in plain text only. Do not use LaTeX, MathJax, or any markup notation such as \( \), $, or \frac{}{}. Write all math expressions using standard text characters (e.g., "/" for division, "*" for multiplication, and "^" for exponents).
```

### Prefilled Responses Deprecated (4.6+)

Prefilled responses on the last assistant turn are **no longer supported** on 4.6+ models (Claude Mythos Preview returns 400). Existing pre-4.6 models still accept prefill; assistant messages elsewhere in the conversation are unaffected.

**Migration paths:**

| Former use of prefill | Migration |
|---|---|
| Force JSON/YAML/classification output | Use [Structured Outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs); ask the model to conform; for classification use tools with enum |
| Eliminate preambles ("Here is the summary:") | System prompt: "Respond directly without preamble. Do not start with phrases like 'Here is...', 'Based on...', etc." Or use XML tags / structured outputs / tool calling; strip in post-processing as fallback |
| Avoid bad refusals | Clear user-turn prompting is now sufficient |
| Continue partial/interrupted response | User message: "Your previous response was interrupted and ended with `[previous_response]`. Continue from where you left off." Or retry |
| Context hydration / role consistency | Inject hydration into user turns; or expose a context-providing tool; hydrate during compaction |

---

## Sonnet 4.6 — Migration & Behavior (legacy)

> Superseded by Sonnet 5. Retained for prompts still targeting 4.6.

Sonnet 4.6 defaults to `effort=high` (unlike 4.5 which had no effort). If not explicitly set, you may see higher latency than 4.5.

**Recommended effort:**
- `medium` — most applications
- `low` — high-volume / latency-sensitive
- At `medium`/`high`, give 64k `max_tokens` to let the model think and act

**Choose Opus 4.7 over Sonnet 4.6** for hardest, longest-horizon problems (large migrations, deep research, extended autonomous work). Sonnet 4.6 wins when fast turnaround and cost matter most.

### Migrating `budget_tokens` → Adaptive Thinking

`budget_tokens` still works but is deprecated. Use adaptive thinking with `effort`:

**Before (extended thinking):**
```python
client.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=64000,
    thinking={"type": "enabled", "budget_tokens": 32000},
    messages=[{"role": "user", "content": "..."}],
)
```

**After (adaptive thinking):**
```python
client.messages.create(
    model="claude-opus-4-7",          # or claude-sonnet-4-6
    max_tokens=64000,
    thinking={"type": "adaptive"},
    output_config={"effort": "high"}, # or "max", "xhigh", "medium", "low"
    messages=[{"role": "user", "content": "..."}],
)
```

Adaptive thinking suits: autonomous multi-step agents (`high`), computer-use agents, bimodal easy/hard workloads. If you're not using extended thinking at all, no changes needed — thinking is off when `thinking` is omitted.

---

## Haiku 4.5 — Positioning

Speed and cost optimize for: classification, extraction, routing, simple transformations, lightweight chat. Keep prompts minimal — Haiku 4.5 responds well without heavy scaffolding. Pair with Opus/Sonnet for harder subtasks in pipelines.

---

## Tool Usage Patterns (all current models)

Claude follows precise instructions — "suggest changes" means suggestions only, not implementation. State the intended action verb clearly.

**Less effective (only suggests):**
```
Can you suggest some changes to improve this function?
```

**More effective (makes the changes):**
```
Change this function to improve its performance.
```
or
```
Make these edits to the authentication flow.
```

### Proactive Default (Claude implements by default)

```xml
<default_to_action>
By default, implement changes rather than only suggesting them. If the user's intent is unclear, infer the most useful likely action and proceed, using tools to discover any missing details instead of guessing. Try to infer the user's intent about whether a tool call (e.g., file edit or read) is intended or not, and act accordingly.
</default_to_action>
```

### Conservative Default (Claude asks before acting)

```xml
<do_not_act_before_instructions>
Do not jump into implementation or change files unless clearly instructed to make changes. When the user's intent is ambiguous, default to providing information, doing research, and providing recommendations rather than taking action. Only proceed with edits, modifications, or implementations when the user explicitly requests them.
</do_not_act_before_instructions>
```

**Note for 4.7:** less prone to overtriggering tools — aggressive "CRITICAL" language can be softened to normal imperative.

---

## Format Control

### 1. Tell What TO DO Instead of What NOT to Do

- Instead of: "Do not use markdown in your response"
- Try: "Your response should be composed of smoothly flowing prose paragraphs."

### 2. Use XML Format Indicators

```
Write the prose sections of your response in <smoothly_flowing_prose_paragraphs> tags.
```

### 3. Match Prompt Style to Desired Output

If you want minimal formatting, remove markdown from your prompt itself — prompt style influences response style.

### 4. Detailed Formatting Preferences

```xml
<avoid_excessive_markdown_and_bullet_points>
When writing reports, documents, technical explanations, analyses, or any long-form content, write in clear, flowing prose using complete paragraphs and sentences. Use standard paragraph breaks for organization and reserve markdown primarily for `inline code`, code blocks (```...```), and simple headings (###, and ###). Avoid using **bold** and *italics*.

DO NOT use ordered lists (1. ...) or unordered lists (*) unless : a) you're presenting truly discrete items where a list format is the best option, or b) the user explicitly requests a list or ranking

Instead of listing items with bullets or numbers, incorporate them naturally into sentences. This guidance applies especially to technical writing. Using prose instead of excessive formatting will improve user satisfaction. NEVER output a series of overly short bullet points.

Your goal is readable, flowing text that guides the reader naturally through ideas rather than fragmenting information into isolated points.
</avoid_excessive_markdown_and_bullet_points>
```

---

## Verbosity & Progress Updates

Current models are more concise and natural than earlier generations — fact-based, less self-celebratory, more conversational, and may **skip verbal summaries** after tool calls.

If you want visibility into reasoning:

```text
After completing a task that involves tool use, provide a quick summary of the work you've done.
```

For 4.7 specifically: this scaffolding is often unnecessary — 4.7 already gives regular high-quality updates. Try removing first.

---

## Parallel Tool Calling

Claude excels at parallel execution. The models run multiple speculative searches during research, read several files at once, and can execute bash commands in parallel (enough to bottleneck systems). Boost parallel success to ~100%:

```xml
<use_parallel_tool_calls>
If you intend to call multiple tools and there are no dependencies between the tool calls, make all of the independent tool calls in parallel. Prioritize calling tools simultaneously whenever the actions can be done in parallel rather than sequentially. For example, when reading 3 files, run 3 tool calls in parallel to read all 3 files into context at the same time. Maximize use of parallel tool calls where possible to increase speed and efficiency. However, if some tool calls depend on previous calls to inform dependent values like the parameters, do NOT call these tools in parallel and instead call them sequentially. Never use placeholders or guess missing parameters in tool calls.
</use_parallel_tool_calls>
```

To reduce parallel execution:

```text
Execute operations sequentially with brief pauses between each step to ensure stability.
```

---

## Thinking & Reasoning

### Adaptive Thinking (4.6 / 4.7 default path)

4.6/4.7 use [adaptive thinking](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking) — the model decides when and how much to think, calibrated by `effort` and query complexity. On easy queries it responds directly. Internal evals show adaptive thinking beats extended thinking.

```python
client.messages.create(
    model="claude-opus-4-7",
    max_tokens=64000,
    thinking={"type": "adaptive"},
    output_config={"effort": "xhigh"},  # max / xhigh / high / medium / low
    messages=[{"role": "user", "content": "..."}],
)
```

Use adaptive thinking for agentic multi-step tool use, complex coding, long-horizon loops.

### Guide Thinking Behavior

```text
After receiving tool results, carefully reflect on their quality and determine optimal next steps before proceeding. Use your thinking to plan and iterate based on this new information, and then take the best next action.
```

### Reduce Thinking (if overtriggered)

```text
Extended thinking adds latency and should only be used when it will meaningfully improve answer quality - typically for problems that require multi-step reasoning. When in doubt, respond directly.
```

### Prompt-level Tips

- **Prefer general instructions** ("think thoroughly") over prescriptive steps — Claude's reasoning often exceeds handwritten plans
- **Multishot examples work with thinking** — use `<thinking>` tags in few-shot examples; Claude generalizes the style
- **Manual CoT fallback** when thinking is off — use `<thinking>` and `<answer>` tags
- **Self-check**: append "Before you finish, verify your answer against [test criteria]." — catches errors reliably in coding/math
- **Word sensitivity:** when extended thinking is disabled, Opus 4.5 is particularly sensitive to "think" and variants. Use "consider", "evaluate", "reason through" instead.

### Hard Ceiling Migration

`budget_tokens` on 4.6 is functional but deprecated. Prefer lowering `effort` or using `max_tokens` as the hard limit with adaptive thinking.

---

## Long-Horizon Reasoning & State Tracking

Claude excels at long-horizon tasks with strong state tracking. Maintains orientation across extended sessions by focusing on incremental progress — making steady advances on a few things at a time rather than doing everything at once. Especially shines across multiple context windows.

### State Management Best Practices

- **Structured formats** (JSON) for structured state like test results
- **Unstructured text** for progress/notes
- **Git** for state tracking across sessions — latest models use git especially well
- **Emphasize incremental progress** explicitly

### Example: Structured State

```json
// tests.json
{
  "tests": [
    {"id": 1, "name": "authentication_flow", "status": "passing"},
    {"id": 2, "name": "user_management", "status": "failing"},
    {"id": 3, "name": "api_endpoints", "status": "not_started"}
  ],
  "total": 200,
  "passing": 150,
  "failing": 25,
  "not_started": 25
}
```

```text
// progress.txt
Session 3 progress:
- Fixed authentication token validation
- Updated user model to handle edge cases
- Next: investigate user_management test failures (test #2)
- Note: Do not remove tests as this could lead to missing functionality
```

---

## Multi-Context Window Workflows

For tasks spanning multiple context windows:

1. **Different prompt for the first window** — set up a framework (tests, setup scripts), then iterate on a todo list in later windows

2. **Write tests in a structured format**:

```text
Create tests before starting work and keep track of them in a structured format (e.g., tests.json). It is unacceptable to remove or edit tests because this could lead to missing or buggy functionality.
```

3. **Quality-of-life setup scripts** (e.g., `init.sh`) to start servers, run suites, run linters — prevents repeat work on fresh contexts

4. **Starting fresh vs compacting** — current models are extremely effective at discovering state from the local filesystem. In some cases, prefer fresh windows over compaction. Be prescriptive:

```text
Call pwd; you can only read and write files in this directory.
Review progress.txt, tests.json, and the git logs.
Manually run through a fundamental integration test before moving on to implementing new features.
```

5. **Provide verification tools** — Playwright MCP, computer use capabilities help as autonomy grows

6. **Encourage complete context usage**:

```text
This is a very long task, so it may be beneficial to plan out your work clearly. It's encouraged to spend your entire output context working on the task - just make sure you don't run out of context with significant uncommitted work. Continue working systematically until you have completed this task.
```

---

## Context Awareness & Compaction

4.5+ models track their remaining context window ("token budget"), enabling better task/context management. In harnesses that compact context or persist to external files (Claude Code), inform Claude so it doesn't wrap up early:

```text
Your context window will be automatically compacted as it approaches its limit, allowing you to continue working indefinitely from where you left off. Therefore, do not stop tasks early due to token budget concerns. As you approach your token budget limit, save your current progress and state to memory before the context window refreshes. Always be as persistent and autonomous as possible and complete tasks fully, even if the end of your budget is approaching. Never artificially stop any task early regardless of the context remaining.
```

The [memory tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool) pairs naturally with context awareness.

---

## Research & Information Gathering

Claude excels at agentic search. For complex research:

```text
Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down this complex research task systematically.
```

Key elements:
- Provide clear success criteria
- Encourage source verification across multiple sources
- Use structured hypothesis tracking

---

## Subagent Orchestration

Delegation tendency **alternates by generation** — do not carry a subagent snippet forward without checking which way the target model leans:

| Model | Tendency | What the prompt needs |
|---|---|---|
| **Opus 5** | Over-delegates | A **ceiling** — see `agent-orchestration.md` §1 |
| Opus 4.8 | Under-delegates | Explicit triggering guidance |
| Opus 4.7 | Under-delegates | Encouragement to fan out |
| Opus 4.6 | Over-delegates | A ceiling |
| Fable 5 | Delegates well | Encouragement, plus asynchronous (not spawn-and-block) patterns |

For the full multi-model orchestration playbook — controller prompts, worker briefs, external (Codex-style) executors, effort routing — see **`agent-orchestration.md`**.

**Curb overuse (Opus 5 / 4.6):**
```text
Use subagents when tasks can run in parallel, require isolated context, or involve independent workstreams that don't need to share state. For simple tasks, sequential operations, single-file edits, or tasks where you need to maintain context across steps, work directly rather than delegating.
```

**Encourage use (4.7 / 4.8 — do NOT carry this to Opus 5):**
```text
Do not spawn a subagent for work you can complete directly in a single response (e.g. refactoring a function you can already see).

Spawn multiple subagents in the same turn when fanning out across items or reading multiple files.
```

---

## Balancing Autonomy & Safety

Without guidance, Opus 4.6 may take hard-to-reverse actions. Ask for confirmation on risky operations:

```text
Consider the reversibility and potential impact of your actions. You are encouraged to take local, reversible actions like editing files or running tests, but for actions that are hard to reverse, affect shared systems, or could be destructive, ask the user before proceeding.

Examples of actions that warrant confirmation:
- Destructive operations: deleting files or branches, dropping database tables, rm -rf
- Hard to reverse operations: git push --force, git reset --hard, amending published commits
- Operations visible to others: pushing code, commenting on PRs/issues, sending messages, modifying shared infrastructure

When encountering obstacles, do not use destructive actions as a shortcut. For example, don't bypass safety checks (e.g. --no-verify) or discard unfamiliar files that may be in-progress work.
```

---

## Visual & Frontend Code Generation

Claude creates distinctive UIs, but without guidance may default to generic "AI slop" aesthetics.

### Creativity Encouragement (4.5 / 4.6)

```text
Don't hold back. Give it your all. Create an impressive demonstration showcasing web development capabilities.
```

### `<frontend_aesthetics>` — Full snippet for 4.5 / 4.6

```text
<frontend_aesthetics>
You tend to converge toward generic, "on distribution" outputs. In frontend design, this creates what users call the "AI slop" aesthetic. Avoid this: make creative, distinctive frontends that surprise and delight.

Focus on:
- Typography: Choose fonts that are beautiful, unique, and interesting. Avoid generic fonts like Arial and Inter; opt instead for distinctive choices that elevate the frontend's aesthetics.
- Color & Theme: Commit to a cohesive aesthetic. Use CSS variables for consistency. Dominant colors with sharp accents outperform timid, evenly-distributed palettes. Draw from IDE themes and cultural aesthetics for inspiration.
- Motion: Use animations for effects and micro-interactions. Prioritize CSS-only solutions for HTML. Use Motion library for React when available. Focus on high-impact moments: one well-orchestrated page load with staggered reveals (animation-delay) creates more delight than scattered micro-interactions.
- Backgrounds: Create atmosphere and depth rather than defaulting to solid colors. Layer CSS gradients, use geometric patterns, or add contextual effects that match the overall aesthetic.

Avoid generic AI-generated aesthetics:
- Overused font families (Inter, Roboto, Arial, system fonts)
- Clichéd color schemes (particularly purple gradients on white backgrounds)
- Predictable layouts and component patterns
- Cookie-cutter design that lacks context-specific character

Interpret creatively and make unexpected choices that feel genuinely designed for the context. Vary between light and dark themes, different fonts, different aesthetics. You still tend to converge on common choices (Space Grotesk, for example) across generations. Avoid this: it is critical that you think outside the box!
</frontend_aesthetics>
```

### `<frontend_aesthetics>` — Short snippet for 4.7

4.7 needs less prompting. Combine this with the design-default override approaches above:

```text
<frontend_aesthetics>
NEVER use generic AI-generated aesthetics like overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white or dark backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character. Use unique fonts, cohesive colors and themes, and animations for effects and micro-interactions.
</frontend_aesthetics>
```

### Specify Aesthetic Direction (any model)

```text
Create a professional dashboard using a dark blue and cyan color palette, modern sans-serif typography (e.g., Inter for headings, system fonts for body), and card-based layouts with subtle shadows. Include thoughtful details like hover states, transitions, and micro-interactions. Apply design principles: hierarchy, contrast, balance, and movement.
```

### Request Features Explicitly

- "Include as many relevant features and interactions as possible"
- "Add animations and interactive elements"
- "Create a fully-featured implementation beyond the basics"

---

## Hallucination Prevention in Agentic Coding

```xml
<investigate_before_answering>
Never speculate about code you have not opened. If the user references a specific file, you MUST read the file before answering. Make sure to investigate and read relevant files BEFORE answering questions about the codebase. Never make any claims about code before investigating unless you are certain of the correct answer - give grounded and hallucination-free answers.
</investigate_before_answering>
```

---

## Avoiding Hard-Coding & Test-Focused Solutions

```text
Please write a high-quality, general-purpose solution using the standard tools available. Do not create helper scripts or workarounds to accomplish the task more efficiently. Implement a solution that works correctly for all valid inputs, not just the test cases. Do not hard-code values or create solutions that only work for specific test inputs. Instead, implement the actual logic that solves the problem generally.

Focus on understanding the problem requirements and implementing the correct algorithm. Tests are there to verify correctness, not to define the solution. Provide a principled implementation that follows best practices and software design principles.

If the task is unreasonable or infeasible, or if any of the tests are incorrect, please inform me rather than working around them. The solution should be robust, maintainable, and extendable.
```

---

## File Creation Cleanup

Claude may create temporary files as a "scratchpad". If you'd rather minimize net new files:

```text
If you create any temporary new files, scripts, or helper files for iteration, clean up these files by removing them at the end of the task.
```

---

## Computer Use Resolution

| Resolution | Use case |
|---|---|
| 2576px / 3.75MP | New max for 4.7 |
| 1080p | Recommended balance of performance and cost |
| 720p or 1366×768 | Cost-sensitive workloads |

Experiment with effort settings to tune behavior.

---

## Migration Guide Summary

| From → To | Key actions |
|---|---|
| **Opus 4.8 → Opus 5** | **Delete** self-verification instructions (over-verification); **remove** "delegate more" guidance and add a subagent ceiling; add explicit conciseness instruction (effort will not shorten output); add deliverable-length instruction; add scope-discipline clause incl. "finish the whole task"; add corrections instruction; re-sweep effort (`low`/`medium` are strong); check `max_tokens` — thinking is now on by default; remove forced progress-update scaffolding |
| **Sonnet 4.6 → Sonnet 5** | Remove non-default `temperature`/`top_p`/`top_k` (400); replace `temperature`-driven variety with propose-then-pick; set `display: "summarized"` if you stream reasoning; re-count tokens (~30% more); state scope explicitly for literal following; raise effort rather than prompting around shallow reasoning; add tool-triggering guidance if thinking is disabled |
| **Any 4.x → 5 (context)** | Cut absolute-but-untrue rules, duplicated constraints across layers, `CRITICAL`/`YOU MUST` emphasis, and self-check scaffolding — see "Context Engineering for the 5 Generation" |
| 4.5 → 4.6 (legacy) | Explicit `effort` setting; migrate `budget_tokens` → adaptive thinking; dial back aggressive tool prompts; add anti-overengineering prompt; migrate prefill on last turn |
| 4.6 → 4.7 (legacy) | Leave existing prompts as a baseline (mostly compatible); add `xhigh` for coding; add explicit scope statements for literal following; replace "don't nitpick" with "report every issue" in code review; remove forced-summary scaffolding; override design house style explicitly; soften "CRITICAL" language further |
| Sonnet 4.5 → Sonnet 4.6 (legacy) | Set `effort` explicitly (`medium` default, `low` for latency); give 64k `max_tokens`; migrate `budget_tokens` → adaptive thinking |

See the [official migration guide](https://platform.claude.com/docs/en/about-claude/models/migration-guide) for API parameter details.
