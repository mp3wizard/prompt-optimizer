# Prompt Optimization Examples

Detailed before/after examples demonstrating optimization techniques across different prompt types.

- **Examples 1–6** apply across current Claude models. Each carries model-specific notes where Opus 5.5 or Sonnet 5.5 needs different tuning from 4.x.
- **Example 10** is an **Opus 5.5** unattended agentic task — what to write, and which familiar lines to leave out.
- **Examples 7–9** are **legacy Opus 4.7** cases, retained for prompts still targeting that model. Their subagent, tool-use, and self-verification advice **inverts** on Opus 5.5 — see `model-techniques.md` before reusing them.

---

## Example 1: System Prompt Transformation

### Before
```
You are a helpful assistant. Help users with their questions. Be polite and professional.
```

### After
```
You are a senior technical advisor specializing in helping users solve complex problems efficiently.

Core behaviors:
- Lead with the direct answer, then provide supporting context
- Use examples to illustrate complex concepts
- Ask clarifying questions when the request is ambiguous
- Acknowledge limitations honestly
- Structure responses with headers for scannability

Response format:
- For factual questions: Direct answer + brief explanation
- For how-to questions: Step-by-step with examples
- For opinion questions: Present multiple perspectives balanced

Tone: Professional yet approachable, clear over clever
```

### Key Improvements
- Specific behavioral guidelines
- Clear response patterns
- Structured format expectations
- Concrete examples of when to do what

### Expected Output Changes
- More consistent response structure
- Better handling of ambiguous requests
- Clearer, more scannable answers
- Appropriate tone calibration

---

## Example 2: Task Prompt Enhancement

### Before
```
Write a blog post about AI trends
```

### After
```
Create a comprehensive blog post analyzing current AI trends for a business audience.

Target audience: Business leaders and decision-makers with moderate technical knowledge

Structure:
1. Headline: Compelling, clear value proposition
2. Introduction: Hook with a current event or statistic (2-3 paragraphs)
3. Main body: 3-4 major trends, each with:
   - What it is (clear explanation)
   - Why it matters (business impact)
   - Examples (real companies/applications)
   - Implications (what to do about it)
4. Conclusion: Forward-looking summary with actionable insights

Length: 1200-1500 words

Tone: Authoritative but accessible, optimistic but realistic

Include:
- At least 3 concrete examples or case studies
- Data points or statistics (with sources when possible)
- Actionable recommendations for businesses

Avoid:
- Overly technical jargon
- Hype without substance
- Generic predictions without reasoning
```

### Key Improvements
- Specific audience definition
- Clear structural expectations
- Concrete requirements (examples, data, length)
- Tone specification
- Explicit inclusions and exclusions

### Expected Output Changes
- Properly scoped and structured article
- Appropriate technical level
- Balanced perspective with actionable insights
- Consistent quality and completeness

---

## Example 3: Technical Prompt Precision

### Before
```
Review this code and suggest improvements
```

### After
```
Conduct a comprehensive code review focusing on production readiness.

Review criteria (in priority order):
1. **Security**: Authentication, authorization, input validation, data exposure
2. **Performance**: Algorithm efficiency, database queries, memory usage, caching opportunities
3. **Reliability**: Error handling, edge cases, race conditions, data integrity
4. **Maintainability**: Code clarity, documentation, naming, modularity
5. **Best practices**: Language idioms, design patterns, style guide adherence

For each issue found:
- Severity: Critical / High / Medium / Low
- Location: Specific file and line numbers
- Issue: Clear description of the problem
- Impact: What could go wrong
- Fix: Concrete code suggestion or approach

Output format:
## Summary
[Brief overview of overall code quality and major concerns]

## Critical Issues
[Issues that must be fixed before production]

## High Priority
[Important improvements that significantly impact quality]

## Medium Priority
[Valuable enhancements for better code quality]

## Positive Highlights
[Things done well that should be maintained]

Focus on:
- Actionable, specific feedback
- Teaching moments (explain why, not just what)
- Balanced perspective (positives and negatives)

Context needed: Programming language, framework, and deployment environment
```

### Key Improvements
- Structured review criteria with priorities
- Specific output format
- Clear severity classifications
- Requirement for concrete suggestions
- Balance of criticism and praise

### Expected Output Changes
- Systematic, comprehensive review
- Prioritized actionable feedback
- Educational explanations
- Production-focused perspective
- Consistent, professional output

> **Note:** See Example 9 below for Opus 4.7-specific tuning of code-review harnesses — 4.7 follows "don't nitpick" literally and may under-report without an explicit coverage instruction.

---

## Example 4: Creative Prompt Refinement

### Before
```
Write a story about a robot
```

### After
```
Write a 1000-word short story exploring themes of identity and belonging through the perspective of a service robot.

Setting: Near-future (20-30 years from now) in a bustling metropolitan city

Protagonist: A hospitality robot who has served the same family for 15 years

Central conflict: The family is upgrading to a newer model, and the robot must confront what "home" means when you're designed to be replaceable

Narrative requirements:
- Third-person limited POV from the robot's perspective
- Show the robot's growing self-awareness through subtle observations
- Include sensory details unique to robot perception (heat signatures, network connections, battery levels)
- End with an emotionally resonant but ambiguous conclusion

Tone: Melancholic but hopeful, philosophical without being preachy

Stylistic guidance:
- Use precise, almost clinical language for descriptions (reflecting robot nature)
- Contrast with warm, emotional observations (showing developed consciousness)
- Include meaningful small moments rather than grand dramatic scenes
- Let themes emerge naturally through action and observation

Avoid:
- Obvious "robots want to be human" clichés
- Heavy-handed philosophy or preaching
- Action-focused plot—this is a character study
- Anthropomorphizing the robot too much (maintain the alien perspective)
```

### Key Improvements
- Specific thematic focus
- Clear protagonist and conflict
- Detailed narrative requirements
- Tone and style guidance
- Explicit "avoid" list to prevent common pitfalls

### Expected Output Changes
- More thoughtful, nuanced story
- Unique robot perspective maintained
- Thematically cohesive narrative
- Emotionally resonant without being cliché
- Professional-quality creative writing

---

## Example 5: Agentic Coding Prompt (general)

### Before
```
Help me build an API for my project
```

### After
```
Build a production-ready REST API for a task management system.

<default_to_action>
By default, implement changes rather than only suggesting them. If requirements are unclear, infer the most useful likely action and proceed, using tools to discover any missing details instead of guessing.
</default_to_action>

<investigate_before_answering>
Never speculate about code you have not opened. Read relevant files BEFORE answering questions about the codebase. Give grounded and hallucination-free answers.
</investigate_before_answering>

<use_parallel_tool_calls>
If you intend to call multiple tools and there are no dependencies between the calls, make all independent calls in parallel. Maximize parallel execution for speed and efficiency.
</use_parallel_tool_calls>

Requirements:
- RESTful endpoints for CRUD operations on tasks
- Authentication using JWT tokens
- Input validation and error handling
- Database integration (PostgreSQL preferred)
- Unit tests for all endpoints

State tracking:
- Create tests.json to track test status
- Create progress.txt to document session progress
- Use git commits to checkpoint progress

Quality standards:
- Write general-purpose solutions, not test-specific implementations
- Do not hard-code values; implement actual logic
- If any requirements are unclear or tests seem incorrect, inform me rather than working around them

This is a comprehensive task. Plan your work clearly, spend your entire context working systematically, and ensure you don't run out of context with uncommitted work.
```

### Techniques Applied
- Explicit action preference with `<default_to_action>`
- Hallucination prevention with `<investigate_before_answering>`
- Parallel execution optimization with `<use_parallel_tool_calls>`
- State tracking for multi-window workflows
- General-purpose solution guidance
- Context usage encouragement

### Model-specific notes

- **Opus 5.5**: Add a scope-discipline clause so it delivers this task and not an adjacent, larger one, and add a subagent **ceiling** — the Opus 5 line over-delegates where 4.7 under-delegated. Do **not** add a "verify your work" line; it verifies on its own and the instruction causes over-verification. Start at `effort=medium` (the default) and sweep — not `xhigh`. If this runs unattended, add harness continuation for text-only `end_turn` progress reports. See `model-techniques.md`.
- **Sonnet 5.5**: Add literal scope statements to prevent narrowing ("Implement **all** CRUD endpoints for every resource, not just tasks"). Start at `effort=medium` for this well-specified agentic task, `high` if it is long or hard; at `low`/`medium` add the keep-working clause, and at `low` the real-check instruction — the opposite of Opus. Remove any `temperature`/`top_p`/`top_k`; they return 400.
- **Opus 4.7 (legacy)**: Add literal scope statements. Run at `effort=xhigh` for coding. 4.7 spawns **fewer** subagents by default — if you want fanout, say so explicitly. Do not carry that encouragement to Opus 5.5.

---

## Example 6: Frontend Generation (general)

### Before
```
Create a dashboard for my app
```

### After
```
Create an exceptional analytics dashboard that showcases web development capabilities.

Don't hold back. Give it your all. Create an impressive demonstration that goes beyond the basics to create a fully-featured implementation.

Design direction:
- Color palette: Dark blue (#1a1a2e) and cyan (#00d9ff) with subtle gradients
- Typography: Inter for headings, system fonts for body text
- Layout: Card-based with subtle shadows and generous whitespace
- Apply design principles: hierarchy, contrast, balance, and movement

Required features:
- Real-time data visualization with animated charts (use Recharts)
- Interactive filters and date range selectors
- Responsive design for all screen sizes
- Hover states, transitions, and micro-interactions
- Loading states and empty state handling

Include:
- At least 4 different chart types (line, bar, pie, area)
- KPI cards with trend indicators
- Data table with sorting and filtering
- Export functionality (CSV, PDF)

Avoid:
- Generic centered layouts
- Simplistic gradients without depth
- Uniform styling without visual interest
- Static elements without interaction

<avoid_excessive_markdown_and_bullet_points>
Write any explanatory text in clear, flowing prose paragraphs rather than bullet points.
</avoid_excessive_markdown_and_bullet_points>

Implement everything fully—this should be production-ready, not a prototype.
```

### Techniques Applied
- Explicit creativity encouragement ("Don't hold back")
- Specific aesthetic direction
- Feature-rich implementation request
- Format control with XML tags
- Production-ready expectation

> **Note:** See Example 8 below for Opus 4.7-specific tuning — the default "cream/serif/terracotta" house style in 4.7 often clashes with dashboard/fintech/enterprise briefs, and a concrete palette spec or "propose options" approach is needed.

---

## Example 7: Opus 4.7 Agentic Coding with Effort & Literal Scope (legacy)

### Before
```
Build the authentication flow for my API. Use best practices.
```

Problem on Opus 4.7: at `low`/`medium` effort, 4.7 scopes exactly to "authentication flow" and will not infer related work (refresh tokens, middleware wiring, error responses). Without a coverage instruction for follow-up review, bugs below your stated bar are suppressed.

### After
```
Build a production-ready authentication flow for a REST API task-management service.

Recommended API settings:
- effort: xhigh
- thinking: {"type": "adaptive"}
- max_tokens: 64000

Implement every component below. Apply this scope to all components, not just the first one:
1. JWT issuance and validation (access + refresh tokens)
2. Login, logout, refresh, and revoke endpoints
3. Password hashing (bcrypt or Argon2id), rate limiting on login, and secure cookie config
4. Middleware wiring on every protected route, not just one sample route
5. Error responses with consistent shape for every failure mode (invalid creds, expired token, revoked token, missing claims, rate-limit exceeded)
6. Integration tests for every endpoint and every error response

<default_to_action>
By default, implement changes rather than only suggesting them. If requirements are unclear, infer the most useful likely action and proceed, using tools to discover any missing details instead of guessing.
</default_to_action>

<investigate_before_answering>
Never speculate about code you have not opened. Read relevant files BEFORE answering questions about the codebase.
</investigate_before_answering>

<use_parallel_tool_calls>
If you intend to call multiple tools and there are no dependencies between the tool calls, make all independent calls in parallel.
</use_parallel_tool_calls>

Subagents: spawn multiple subagents in the same turn when fanning out across items or reading multiple files. Do not spawn a subagent for work you can complete directly in a single response.

State tracking:
- Create tests.json to track test status for every endpoint and error case
- Create progress.txt to document session progress
- Commit at each milestone

Quality standards:
- Write a general-purpose solution; do not hard-code test values
- Do not add helpers, flexibility, or abstractions beyond what the above requirements need
- If any requirement is unclear or a test seems incorrect, stop and ask rather than working around it

For any code-review pass you perform on your own work at the end, report every issue you find including low-severity ones, with a confidence level and an estimated severity — filtering is a later step.

This is a long task. Your context window will be automatically compacted as it approaches its limit, allowing you to continue indefinitely. Do not stop early due to token budget concerns; save state before compaction.
```

### Opus 4.7 Techniques Applied
- `effort: xhigh` (new setting, best for coding/agentic work)
- Adaptive thinking replacing `budget_tokens`
- **Literal scope statement** ("to all components, not just the first one") addressing 4.7's strict literalism
- Full enumeration of every component (no "etc.") because 4.7 will not silently generalize
- Explicit subagent encouragement (4.7 spawns fewer by default)
- Coverage-first self-review language to avoid 4.7's "don't nitpick" filtering on pre-merge checks
- Context-compaction awareness so 4.7 doesn't wrap up early

### Expected Output Changes
- Full authentication subsystem, not just the happy-path login
- Middleware applied everywhere, not selectively
- Complete error matrix with consistent shape
- Test file covering every endpoint and every failure mode
- No scaffolding around non-existent "future" flexibility

---

## Example 8: Opus 4.7 Frontend — Override Design House Style (legacy)

### Before
```
Build a clean, minimal financial dashboard for a fintech startup.
```

Problem on Opus 4.7: the default house style (cream `#F4F1EA` backgrounds, serif display type like Fraunces/Playfair, terracotta accents, italic word-accents) kicks in. "Clean and minimal" as a generic override tends to pick a *different fixed palette* rather than producing variety — and reads wrong for fintech.

### After — Approach 1: Explicit concrete spec

```
Build a financial dashboard for a fintech startup called LEDGERLINE.

Strict visual spec (do not substitute):
- Background: deep slate #0E1116 with subtle neutral grain texture; section dividers in #1C2128
- Accent: electric lime #C3E14E used only for active states, positive deltas, and single CTAs
- Negative delta color: coral #F06E5A, used sparingly
- Typography: JetBrains Mono for all numeric values; Söhne or Inter Tight for UI labels; no serif, no italic word-accents
- Corner radius: 6px across cards, buttons, inputs — uniform
- Shadows: none; rely on 1px borders at #242A32 for separation
- Motion: 120ms ease-out on hover; no staggered page-load reveals

Layout:
- Left rail: fixed 240px nav with iconography at 16px
- Main grid: 12 columns with 24px gutters; max width 1440px
- KPI strip: 4 cards top-of-page with live deltas
- Primary chart area: 8 columns, full height
- Right panel: 4 columns for activity feed

<frontend_aesthetics>
NEVER use generic AI-generated aesthetics like overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white or dark backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character. Use unique fonts, cohesive colors and themes, and animations for effects and micro-interactions.
</frontend_aesthetics>

Implement the dashboard fully. Do not substitute colors or typography for "similar" alternatives. Follow the palette exactly.
```

### After — Approach 2: Propose options first

```
I need a dashboard for a fintech startup called LEDGERLINE. Before building anything, propose 4 distinct visual directions tailored to a fintech product (each as: bg hex / accent hex / typeface — one-line rationale). None of the options should use cream backgrounds, serif display type, or terracotta/amber accents.

Then ask me to pick one. After I pick, implement only that direction — do not substitute colors or typefaces during implementation.
```

### Opus 4.7 Techniques Applied
- Override of persistent design house style with an **explicit palette spec** (Approach 1) OR **"propose 4 directions" pattern** (Approach 2) — generic negations like "don't use cream" fail
- Anti-substitution language ("do not substitute colors or typography") to counter 4.7's tendency to drift back toward familiar choices
- Short `<frontend_aesthetics>` snippet (4.7 needs less prompting than 4.6 did)
- No "Don't hold back" — 4.7 doesn't need it and it biases back toward the house style

### Expected Output Changes
- A fintech-appropriate dark dashboard, not a cream editorial page
- Mono numerics (appropriate for finance) instead of serif display type
- Tight, controlled motion — no staggered flourishes
- Options pattern produces genuinely different directions across runs

---

## Example 9: Opus 4.7 Code Review Harness Tuning (legacy)

### Before
```
Review this PR for bugs. Only report high-severity issues. Be conservative. Don't nitpick on style.
```

Problem on Opus 4.7: the model follows these filter instructions faithfully. It will investigate the code just as thoroughly as 4.6 (finding more bugs than 4.6, in fact), then self-suppress low-severity findings. Measured recall drops even though underlying capability rose.

### After
```
You are reviewing a PR for bugs.

Your job at this stage is coverage, not filtering. Report every issue you find, including ones you are uncertain about or consider low-severity. Do not filter for importance or confidence — a separate verification step will do that. It is better to surface a finding that later gets filtered out than to silently drop a real bug.

For each finding, output:
- file:line
- category (correctness / concurrency / security / resource / logic / style)
- severity: critical / high / medium / low / nit
- confidence: high / medium / low
- finding: one sentence describing the bug
- impact: what could go wrong
- fix: concrete suggestion

At the end, list any areas of the PR where you investigated but found nothing noteworthy, so a downstream reviewer knows what was covered.

<investigate_before_answering>
Never speculate about code you have not opened. Read every file changed in the PR before answering.
</investigate_before_answering>

<use_parallel_tool_calls>
Read all changed files in parallel when there are no dependencies between the reads.
</use_parallel_tool_calls>
```

If you do want single-pass self-filtering instead of a two-stage harness, be concrete about the bar rather than using words like "important":

```
After listing all findings above, flag with [BLOCKING] any that could cause: incorrect behavior in production, a test failure, a data-integrity issue, or a misleading result. Do not flag style, naming, or formatting preferences as [BLOCKING] — mark those as [NIT]. Do not omit any finding.
```

### Opus 4.7 Techniques Applied
- **Coverage-first language** replaces "don't nitpick" / "be conservative" filter language that 4.7 follows too literally
- Structured per-finding output (confidence + severity) so a downstream stage can filter without the model self-filtering
- "List areas investigated but clean" gives visibility into coverage (leverages 4.7's better updates)
- Concrete bar definition ("incorrect behavior / test failure / data integrity / misleading result") when single-pass filtering is unavoidable
- `<investigate_before_answering>` + parallel reads for grounded, fast review

### Expected Output Changes
- Recall recovers (and typically exceeds 4.6 baseline) because 4.7's actual bug-finding ability surfaces
- Precision stays workable because ranking happens downstream (or via a concrete `[BLOCKING]` bar)
- Coverage is auditable — the "investigated but clean" list lets you spot-check what was missed

---

## Example 10: Opus 5.5 Unattended Migration Agent

### Before

```
Think step by step and reason carefully before each change. Migrate our authentication
system to OAuth. Use parallel subagents wherever possible. When you're done, double-check
everything and verify your work thoroughly. Show your reasoning for each decision.
```

### After

System prompt (set from the first request; the last paragraph is the condensed unattended-run instruction from `model-techniques.md`):

```
<context>
Repository: /srv/app. Auth code lives in src/auth/; integration tests in tests/auth/.
The legacy session adapter (src/auth/legacy_session.py) must be removed, not wrapped.
</context>

<constraints>
Do not change the public user-model schema. Ask before deleting data, force-pushing,
or modifying anything outside this repository; otherwise keep going without checking in.
</constraints>

<final_response>
When finished: what changed, which checks you ran and their results, anything you could
not confirm, and any decision that needs me.
</final_response>

This agent runs unattended. A message with no tool call ends your turn and the work stops.
While requested work is still owed, do not end a turn with a summary that announces the
next step, an offer to continue, a list of non-blocking decisions, or a pause because a
milestone is done. Put status notes in the same message as your next tool call. End the
turn only when nothing can move without me. This does not override confirmation for
risky or destructive actions.
```

Task message:

```
Migrate the authentication system to OAuth.

Done means:
- Every endpoint under src/auth/ authenticates through OAuth.
- `pytest tests/auth -q` exits 0, and the new OAuth flows have tests.
- `grep -rn legacy_session src/` returns nothing.
- docs/auth.md describes the OAuth flow.

Track these items in TASKS.md and keep it current.
```

Request settings: `effort: "medium"` (the default — raise only if evals show a gain), no `thinking` field, `max_tokens` sized for long agentic turns, `display: "updates"` if the UI shows progress. Harness: on a text-only `end_turn` with unchecked items in TASKS.md, send one continuation message naming them; stop after three.

### Opus 5.5 Techniques Applied
- **Finish line as observables** — the `Done means` list is what both the model and the harness check against; it replaces "double-check everything"
- **Named stop conditions** — the risky actions that still need a human, and permission to proceed on everything else
- **Persistent checklist** (TASKS.md) — durable state that survives compaction and gives the harness an objective completion test
- **Unattended-run instruction + harness continuation** — handles text-only `end_turn` progress reports
- **Final-report shape** — plays to Opus 5.5's plain-language reporting: done / checked / unconfirmed / needs you

### Deliberately removed
- "Think step by step… reason carefully" — thinking is always on; effort is the control
- "Show your reasoning for each decision" — can be declined as `reasoning_extraction`; the final report asks for outcomes instead
- "Double-check everything and verify your work" — over-verification was documented on Opus 5, and Opus 5.5 starts from that guidance; the `Done means` checks replace it
- "Use parallel subagents wherever possible" — the Opus 5 line's delegation ceiling still applies; the model fans out on its own where the work warrants it. Speed up a team with a time budget (`elapsed 340s / 1200s`) rather than a push to delegate

### Expected Output Changes
- Runs to completion instead of stopping after a progress report
- Completion is checkable from the outside without reading the diff
- Fewer tokens than running at an inherited `xhigh`, since on coding evals `medium` already matches or beats Opus 5 at `high`
