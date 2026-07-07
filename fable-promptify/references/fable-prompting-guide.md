# Fable 5 prompting guide (distilled)

Sources: Anthropic prompting best practices (platform.claude.com, Claude 4.6/5 era), Fable 5 launch video (Thariq @trq212, Claude Code team, June 2026 — https://x.com/ClaudeDevs/status/2064399512664526853), "Designing loops with Fable 5" (Lance Martin @RLanceMartin, MTS @ Anthropic, June 2026 — https://x.com/RLanceMartin/status/2064397389189071163), "Designing Loops for Claude Code Agents" (@delba_oliveira, Claude Code team, July 2026 — https://x.com/ClaudeDevs/status/2074208949205881033), and the official `/goal` docs (https://code.claude.com/docs/en/goal, fetched July 2026). Distilled June–July 2026 — re-check the docs page if Anthropic releases new Fable guidance. The convention-priors, continuous-checkpointing, and capability-granting points were informed by Theo's launch review (@theo — https://x.com/theo, "Fable is Mythos, and it is really good" — https://www.youtube.com/watch?v=7IewbRdaBWI) and adversarially reviewed before inclusion.

## The mindset shift

With Fable the operator's job moves from checking the model is *doing the work right* to checking it is *doing the right work*. Direction and setup over supervision. Three behaviours follow:

1. **Thought partner** — involve Fable early, give it the context it needs, let it interview you before you commit to a spec.
2. **Goals + ways to verify them** — a well-designed goal or rubric adds feedback to the environment; Fable runs, collects feedback, self-corrects, proceeds until satisfied.
3. **Be ambitious** — Fable rewards being asked for things you have not tried before. If you assumed LLMs couldn't do it, give it a chance.

## The four pillars (what every Fable prompt needs)

### 1. Aim
- State "done" as an observable outcome, not an activity.
- Be clear and direct; Fable follows instructions literally and precisely.
- If you want above-and-beyond, ask for it explicitly: "Include as many relevant features and interactions as possible. Go beyond the basics."
- Action verbs for action: "Implement X", not "Can you suggest improvements to X" — suggestion language produces suggestions.

### 2. Verification
- Give Fable a way to check its own work without the user: tests to write first, a rubric of checkable criteria, observable behaviours, a self-check instruction ("Before you finish, verify your answer against [criteria]").
- For long or high-stakes runs, prefer a **verifier subagent** (independent context window) over self-critique — models grade their own outputs poorly.
- A rubric file with discrete checkable criteria (e.g. "run a baseline; run 20 experiments") outperforms vague quality language.
- Tests verify correctness; they do not define the solution. If the prompt includes tests, also say: implement the general solution, don't hard-code to the test inputs, and report incorrect tests rather than working around them.
- Grant the means, not only the criteria: where verification matters, give tool access the default environment lacks (browser/computer use for UI checks) and explicit permission for throwaway harnesses ("you may write fuzzers or temporary test scripts to validate this; clean them up when done"). Feature builds and long-horizon runs only — never quick tasks.

### 3. Context, not constraints
- Every constraint should carry its motivation. "Keep it simple" → "This feature is an experiment; there's a real chance we delete it in a month, so don't build anything painful to throw away." Fable generalises from the why and catches things you didn't think to forbid.
- Same for format rules: "Never use ellipses" → "The output is read by a text-to-speech engine that can't pronounce ellipses."
- Give Fable a role when it focuses behaviour ("You are reviewing this as a security auditor").
- Pre-empt strong priors. Fable trusts baked-in conventions over unstated reality — a repo whose `main` branch deploys to staging will be "diagnosed" as broken unless the prompt says otherwise, even after in-session correction. State any way the setup deviates from convention (branch→environment mapping, unusual auth flows, intentional architecture quirks). These are the facts users rarely think to state, because the contrary prior is invisible to them.

### 4. Scope & ambition
- Explicit boundaries: what NOT to touch, what is out of scope.
- Counter overengineering with scope context, not bare prohibition: say whether this is an MVP, an experiment, or a production feature, and what flexibility is actually needed.
- State the autonomy level: what it may do freely (edit, test, commit locally) vs what needs confirmation (push, deploy, delete, send, spend). Frame as reversibility: local reversible actions free; hard-to-reverse or outward-facing actions confirm first.

## Style rules for the drafted prompt

- **Calm prose.** No CAPS, "MUST", "CRITICAL", "IMPORTANT". Fable and the 4.6-era models are highly responsive to the system/user prompt; aggressive emphasis that fixed undertriggering in old models now causes overtriggering. "Use this tool when..." beats "CRITICAL: You MUST use this tool when...".
- **Positive instructions.** Say what to do, not what to avoid: "Write flowing prose paragraphs" beats "Do not use markdown".
- **XML tags when mixing content types.** `<instructions>`, `<context>`, `<input>`, `<example>` — only needed when the prompt combines instructions with pasted data, documents, or examples. Long documents go at the top, the query/instructions at the end (up to ~30% quality gain on long inputs). For long-document tasks, ask for relevant quotes first, grounded in `<quotes>` tags.
- **Examples steer format best.** 3–5 relevant, diverse examples in `<example>` tags when output format/tone matters.
- **General over prescriptive for reasoning.** "Think thoroughly about edge cases" outperforms a hand-written step-by-step reasoning plan; Fable's own reasoning usually exceeds what a human would prescribe.
- **Match prompt style to desired output style.** A markdown-heavy prompt begets markdown-heavy output.
- **Every line earns its place.** Padding dilutes. A crystal 10-line prompt beats a 60-line template instantiation.

## Per-shape additions

### Quick task (bounded fix/edit)
Aim + verification, nothing else. No session mechanics, no state files. Often 3–8 lines.

### Feature build
Full four pillars. Consider: interview-first ("interview me about the implementation before writing the spec") if the user is still unsure; HTML mockups of 2–3 directions if it's visual.

### Long-horizon autonomous run (hours, multi-context-window)
Add to the prompt:
- **Goal + completion discipline**: deliver as spec file + compact `/goal` (pattern 2 in "Loops and /goal"): the prompt becomes the handover file, the goal is the completion contract. In the prompt itself: "Continue working systematically until complete; don't stop early due to token budget — save progress and state before the context refreshes."
- **State files**: structured state in JSON (`tests.json` with pass/fail status), freeform progress in `progress.txt`, git commits as checkpoints after each completed component — runs can be cut off without warning (usage caps, crashes), so checkpoint continuously, not only when context runs low. "It is unacceptable to remove or edit tests."
- **Fresh-context restart instructions**: "Review progress.txt, tests.json, and the git log. Run the fundamental integration test before implementing anything new."
- **Setup scripts**: encourage an `init.sh` so servers/tests/linters restart gracefully in a new window.
- **Verification loop**: a Workflow or verifier-subagent pass over each part of the plan, producing a report of what was implemented and what differed.
- **Memory progression** (if memory is available): fail → investigate → verify → distill into a general rule → consult the rule instead of re-deriving. Prompt Fable to distill verified learnings into general rules, not just failure notes.

### Research / analysis
- Define what a successful answer looks like (success criteria).
- Ask for multi-source verification of claims.
- For complex research: competing hypotheses, confidence levels in progress notes, regular self-critique, a hypothesis tree or research-notes file for persistence and transparency.

### Writing / creative
- Voice, audience, length, format — with an example of each if the user has one.
- For frontend/visual work: ask for distinctive, creative output explicitly; name the aesthetic direction. Generic prompts produce generic "AI slop" design.

## Loops and /goal (session primitives)

A loop is an agent repeating cycles of work until a stop condition is met. Pick the primitive at delivery time — the loop type is a design decision, not a default:

| Loop | You hand off | Use when | Primitive |
|---|---|---|---|
| **Turn-based** | the check | exploring or deciding; short tasks | a normal session + verification skills |
| **Goal-based** | the stop condition | "done" is verifiable from Claude's own output | `/goal` |
| **Time-based** | the trigger | recurring work, or watching an external system (CI, a queue) | `/loop` (local) / `/schedule` (cloud) |
| **Proactive** | the prompt itself | a recurring stream of well-defined work | `/schedule` + `/goal` + workflows + auto mode |

### /goal mechanics (what a drafted goal must respect)

- `/goal <condition>` sets a **session-scoped stop check**: after every turn, a small fast evaluator model (Haiku by default) reads the condition **plus the transcript** and answers met / not met. A "not met" reason is injected as next-turn steering; "met" ends the loop.
- The evaluator **runs no tools and reads no files**. It judges only what Fable has surfaced in the transcript — so every criterion must name the check whose output will appear there ("`npm test` exits 0", "`git status` is clean"), not a state of the world.
- The condition is capped at **4,000 characters**. There are no flags: turn/time caps go inside the text ("stop after 20 turns").
- Setting a goal **immediately starts a turn** with the condition as the directive. A new `/goal` replaces the old; `/goal clear` cancels; bare `/goal` shows status (turns evaluated, token spend, the evaluator's last reason). An active goal survives `--resume` (counters reset).
- Headless: `claude -p "/goal <condition>"` runs the whole loop in one invocation.
- A durable condition has three parts: **one measurable end state**, **the stated check that proves it**, and **the constraints that matter** (what must not change) — plus the turn cap.

### The three delivery patterns

1. **Goal-only** — quick verifiable task. The condition carries the task, end state + check, and turn cap in one `/goal` invocation. No separate prompt: setting the goal fires the first turn.
2. **Spec file + compact goal** — feature builds and long-horizon runs (the default). The full compiled prompt lives in a handover/plan file; the goal argument is a completion contract: "Implement the spec in `<file>` fully" + criteria + constraints + turn cap. One paste; every turn including the first is goal-evaluated. The file also survives context refreshes, which the goal argument alone does not help with.
3. **Prompt-then-goal** — mid-session. Paste the compiled prompt, set the compact goal after work is underway. Keep for interactive sessions; in a fresh session pattern 2 is strictly better.

Never compress the compiled prompt into the goal argument: the 4,000-char cap destroys context-with-why, and the evaluator re-reads the condition every turn — it needs the contract, not the story.

### Goal condition template

```
Implement the spec in <file> fully. Done when:
1. <end state> — verified by <check Claude runs, output shown>
2. <end state> — verified by <check>
Constraints: <what must not change>.
Stop after <N> turns if not complete; report remaining gaps.
```

### Managing the loop's token budget (from the loops guidance)

- Deterministic criteria + explicit turn caps let the evaluator stop the loop at the earliest correct moment.
- Scripts for deterministic work — a skill that ships a script beats re-deriving steps each iteration.
- Time-based loops: match the interval to how often the watched thing actually changes; prefer reacting to events over polling.
- Pilot before a large run; check `/goal` (bare) and `/usage` to see turns and token spend.

## Other session mechanics to recommend (delivery advice, not always in the prompt)

- **Workflows** — for verification fan-out: "use a workflow to verify each part of the plan and prepare a report on what was implemented and if anything differed."
- **Fresh session vs current** — a fresh session when the current conversation has accumulated unrelated context; current when the conversation context IS the context.
- **Verifier subagent over self-critique** — grading in an independent context window outperforms self-grading. Use this (not `/goal`) for taste-judged outcomes: a small evaluator model cannot judge design quality or prose voice.
- **Loops over steering** — rather than directly steering Fable step-by-step, design the loop: goal/rubric for feedback, memory for cross-session learning, and let it self-correct. When a result misses the bar, fix the system (the skill, the rubric, the goal), not just the instance.

## Anti-patterns (never put these in a drafted prompt)

- Aggressive emphasis (CAPS/MUST/CRITICAL) — causes overtriggering.
- "If in doubt, use [tool]" / blanket thoroughness nudges — Fable already explores; these cause overtriggering. Targeted "use X when it would enhance understanding" only if needed.
- Bare constraints without motivation.
- Suggestion language when action is wanted.
- Prescriptive step-by-step reasoning plans for problems Fable can reason through itself.
- Hard-coding/test-gaming incentives — pair any test mention with "general solution, not test-specific".
- Boilerplate from this guide that the task shape doesn't need.
